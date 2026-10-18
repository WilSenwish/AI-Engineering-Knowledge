# Python 企业级 Agent 工程实践

> 分级：🟡 生产最佳实践 ｜ 来源：[hello-agents](https://github.com/datawhalechina/hello-agents)（Ch4-10）、[ai-agent-book](https://github.com/bojieli/ai-agent-book)（Ch2-5,9-10，共 109 实验）
> 定位：规划"阶段2 开发工程实现·Python 生态"的核心沉淀。与 Java 栈 [`Java企业级Agent工程实践.md`](../../03-Java生态生产实践栈/阶段2-开发工程实现/Java企业级Agent工程实践.md) 形成**双栈对照**（原理一致、实践差异化）。
> 本章已深化：ai-agent-book Ch2（上下文/KV Cache/Skills）、Ch3（记忆/RAG/RRF）、Ch4（工具ACI/MCP/主动发现）、Ch5（Coding Agent 元能力）的具体范式已落位，附实验地图。

## 一、架构分层（Python 共识）

| 层 | Python 实现 | 职责 |
|----|------------|------|
| 入口/编排 | `FastAPI` + `LangGraph` StateGraph / 自研 HelloAgents Loop | 请求入口、状态编排、流式响应（SSE/流式 generator） |
| AI 供应商隔离 | `langchain-openai` / `openai` 客户端 + 模型路由封装 | Chat/Embedding/Rerank 客户端、档位/降级/多模型切换 |
| 业务编排 | `Agent` / `ToolNode` / `Retriever` | ReAct 循环、工具调用、RAG、记忆、Skills |
| 系统支撑 | `tenacity`（重试/熔断）、`langfuse`（观测）、`pydantic`（契约） | 容错、可观测、I/O 校验、并发控制 |
| 工具 | `mcp` Python SDK（stdio/HTTP）、`@tool` 装饰器 | 基于 MCP 协议的外部工具即插即用 |

> **分层意图**：业务编排、模型供应商差异、基础设施相互隔离 → 切换模型/向量库时核心流程不重写（接口 + 注册表 + 依赖注入）。

## 二、执行范式（Workflow vs Agent 的 Python 落地）

- **Workflow（确定性编排）**：用 `LangGraph` 写死节点顺序（rewrite → intent → retrieve → generate），更快可控，适合知识助手。对应 hello-agents Ch4 范式 + Ch7 框架应用。
- **Agentic（ReAct 循环）**：`agent_executor` 边推理边决策，检索/MCP 工具/Skills 皆为可调工具。核心公式 `Agent = LLM + 上下文 + 工具`（ai-agent-book Ch1）。
- **混合（Supervisor）**：LangGraph `START → supervisor → {worker} → supervisor`，确定主干 + 复杂环节起 Agent（理论 1.5）。
- **Coding Agent 特例**：代码即"创造工具的工具"（ai-agent-book Ch5）——让 Agent 自举工具/脚本，扩宽动作空间（见第六节）。

### ReAct 核心循环（ai-agent-book Ch2，消息列表管理范式）
```python
# 无状态 API：每轮必须重传完整上下文；assistant 原样回放（含 tool_calls）
while True:
    resp = client.chat.completions.create(model="Qwen3-0.6B", messages=messages, tools=tools)
    msg = resp.choices[0].message
    messages.append(msg)                      # 原样回放 assistant
    if not msg.tool_calls:
        print(msg.content); break
    for tc in msg.tool_calls:
        result = execute_tool(tc.function.name, tc.function.arguments)
        messages.append({"role": "tool", "tool_call_id": tc.id, "content": result})
```

## 三、上下文与记忆工程（理论 [`1.2`](../../01-通用理论原理层/1.2-上下文与记忆工程.md)）

### 3.1 上下文结构：静态前缀 + 动态轨迹
- API 层由四种消息角色 `system/user/assistant/tool` + 独立字段 `tools` 构成。
- **静态前缀**（`system` + `tools` 定义）全程不变；**轨迹（Trajectory）**（用户/模型/工具结果）随交互增长。
- 优化全部基于该结构展开。

### 3.2 KV Cache 复用铁律（降本关键，ai-agent-book Ch2）
- 系统提示词与工具定义一旦确定，保持**字节级稳定**（改一个空格即失效后续 KV 缓存）。
- 动态信息（时间/用户状态）作为**新消息 append 到末尾**，绝不修改已有 `system`。
- `tools` 数组顺序固定，不按使用频率动态重排。
- **禁用滑动窗口**：截断历史会破坏前缀一致性并丢失工具结果，导致循环调用。
- 子 Agent 提示词须与父 Agent 字节级对齐以命中 Prompt Cache。

```python
# ❌ 错误：修改 system 前缀（破坏缓存）
# system = f"You are a helper. Time: {datetime.now()}"
# ✅ 正确：动态数据追加到末尾
messages = [{"role": "system", "content": "You are a helpful assistant."}]  # 恒定
messages.append({"role": "user", "content": f"[Status] Time: {datetime.now().isoformat()}"})
```

### 3.3 上下文压缩与状态栏
- 接近 token 预算时**成批压缩**旧轨迹，保留 `decisions / constraints / failures / citations`。
- 末尾注入**状态栏**（如"已呼叫 3 次"）弥补模型不擅长主动归纳隐式状态的缺陷（缓解上下文腐化）。

```python
stable_prefix = system_message            # 字节级稳定
stable_tools  = core_tool_schemas        # 固定顺序
trajectory    = load_message_history(session)
status_msg    = make_status_message(derive_current_state(trajectory))
if estimated_tokens(stable_prefix, trajectory, status_msg) > budget:
    trajectory = compress_old_evidence(trajectory,
                  preserve=[decisions, constraints, failures, citations])
request.messages = [stable_prefix] + trajectory + [status_msg]
request.tools = stable_tools
```

### 3.4 提示工程与注入防御
- SOP 流程驱动（Step1→Step2↓）替代无序规则（消融显示乱序成功率降 30%+）。
- XML 标签承载机器可解析语义，Markdown 标题承载层次。
- **注入防御**：外部内容用 `<external_content source="webpage">` 包裹标记来源，区分"指令"与"数据"。

### 3.5 Agent Skills（三层渐进式披露，ai-agent-book Ch2）
- **元数据**（YAML `name`/`description`，路由条件写"何时用/何时不用/反例"，常驻低 token）；
- **核心流程**（完整指令正文，触发时作为 `user` message 注入末尾）；
- **细则**（文件引用按需深入加载）。
- 原则：少量目录常驻，完整正文仅显式触发时**追加到末尾**——"只增不改"天然兼容 KV Cache。

### 3.6 记忆系统（ai-agent-book Ch3）
- **分层**：短期/轨迹（`append-only` 工作记忆）、长期（跨会话、用户 ID 绑定键值对）、用户画像（主题—子主题两级槽位，配合事件记忆）。
- **存储格式**：Simple Notes（最小事实）→ Enhanced Notes（段落）→ JSON Cards（三层嵌套）→ Advanced JSON Cards（加 backstory/person/relationship 消歧）。
- **更新机制**：`Proposer Agent` 提交 PR（工作分支 diff）+ `Reviewer Agent`（异源模型）审核 + Git 版本化；索引作为派生物可重建。
- **User as Code（进阶）**：带类型可执行对象 + 规则函数，将聚合/冲突/约束交给确定性代码。

```python
# 用户状态约束用代码真值校验（而非信任模型自报）
def check_passport_validity(state):
    for trip in state["trips"]:
        if trip["is_international"]:
            days = (parse(state["passport"]["expiry"]) - parse(trip["departure_date"])).days
            if days < 180: alert(f"passport expires too soon: {days}d")
```

## 四、RAG 检索工程（理论 [`1.3`](../../01-通用理论原理层/1.3-RAG技术体系.md)）

| 环节 | Python 实现 | 工程要点（ai-agent-book Ch3） |
|------|------------|------------------------------|
| 解析切分 | `unstructured`/`pymupdf`/`docling` + `RecursiveCharacterTextSplitter` | 256–1024 token/块，重叠 10%–20%；结构感知/语义切分 |
| 嵌入 | `bge-m3`（稠密，上下文模型）/ BM25（稀疏） | 稠密余弦 + 稀疏倒排并行召回 |
| 多路召回 | 向量（HNSW/ANNOY）+ BM25 + 知识图谱（Neo4j）并行 | 单通道失败不影响主链 |
| 融合 | **RRF（k=60）** + Cross-Encoder 重排（`bge-reranker-v2-m3`） | 多路得分融合、重排提质 |
| 上下文感知 | 索引前用 LLM 为块生成前缀摘要（如 `[节选自ACME 2025 Q2财报]`） | 失败率降 49%–67% |
| 防幻觉 | 回答溯源（引用 chunk id）+ 无证据短路 | 架构层杜绝编造 |

```python
def reciprocal_rank_fusion(rankings, k=60):
    """rankings: List[List[doc_id]]，每种检索器返回的排序列表"""
    fused = {}
    for ranking in rankings:
        for rank, doc_id in enumerate(ranking):
            fused[doc_id] = fused.get(doc_id, 0.0) + 1.0 / (k + rank + 1)
    return sorted(fused.items(), key=lambda x: x[1], reverse=True)
# dense = ["doc_1","doc_7","doc_5"]; sparse = ["doc_1","doc_3","doc_5"]
# final = reciprocal_rank_fusion([dense, sparse])  → 取 top_n 送重排
```

- **知识组织**：RAPTOR（自下而上聚类→父节点摘要，形成"细节→宏观"树）、GraphRAG（实体+关系三元组，支持多跳推理与实体消歧，适合医疗/法律垂直）。
- 指标：`recall@k` / `MRR` / `nDCG`。
- 经典坑（与 Java 栈一致）：Chunk 切粗、Query 未改写、关键词与向量未结合、重排缺失 → 逐项排查比换贵模型有用。

## 五、工具体系与 MCP（理论 [`1.4`](../../01-通用理论原理层/1.4-工具体系与通信协议.md)）

### 5.1 五类工具（ai-agent-book Ch4）
| 类型 | 调用方向 | 作用对象 | 代表 |
|------|---------|---------|------|
| 感知 | Agent 主动 | 获取信息 | web_search / read_file / fetch_url |
| 执行 | Agent 主动 | 改变世界 | shell_exec / write_file / send_email |
| 协作 | Agent 主动 | 驱动其他 Agent/人 | spawn_subagent / request_human_approval |
| 用户沟通 | Agent 主动 | 传递信息 | reply_to_user / send_card |
| 事件触发 | 注册+外部触发 | 唤醒 Agent | set_timer / monitor_shell |

### 5.2 ACI 原则：通用优于专用（除非安全/权限/性能理由）
- 与其提供四则运算计算器，不如提供通用 `code_interpreter`（沙盒装 sympy/numpy/pandas），让 Agent 通过代码完成任意计算——利用 LLM 的代码能力而非限制它。
- 一项能力做成专用工具还是 Skill，与"一次暴露多少条"是**两个独立决策**（形态定常驻 token，披露定同时暴露量）。

### 5.3 感知工具接口设计
- 搜索返回**结构化候选列表**（标题/位置/摘要）+ 分页/游标，让 Agent 先浏览再深入。
- 读取支持 `offset/limit`，**显式截断**（注明省略量与续读方式）；静默截断危险。
- 只读性红利：结果可缓存（键含 user_id + 授权范围 + TTL）、可并行。

### 5.4 MCP 协议（模型上下文协议）
- Server：`from mcp.server import Server`，`@server.list_tools()` / `@server.call_tool()` 暴露能力。
- Client：`mcp` SDK 的 `stdio_client` / `streamable_http_client` 连接；LangChain 有 `MCPToolAdapter` 桥接。
- 三类主动工具（感知/执行/协作）即 MCP Tools；Resources/Prompts 分别对应数据与外部提示模板。

### 5.5 主动工具发现（应对成百上千工具，ai-agent-book Ch4 实验 4-1）
- 对照：把 120+ 工具 schema 一次性注入（>50K tokens）→ 4B 模型指令遵循退化、错选/遗忘工具。
- 方案：system 仅保留 `web_search`/`code_interpreter`/`discover_tools` 元工具；`discover_tools` 接受自然语言需求，经**嵌入向量相似度**返回 3-5 个候选 + schema，追加到对话历史（user message），状态栏更新工具列表。
- 让小模型在上百工具场景下保持可用，大模型应对成千上万工具。

### 5.6 工具安全
- 执行类工具错误代价高，安全约束是核心：提议者-审核者（事前审批 + 事后验证）、Sidecar 机制。
- HITL：关键决策请求人类确认，设超时默认行为（"5 分钟无响应则保守策略"）+ 多渠道通知。
- Schema 校验入参；沙箱隔离代码执行。

## 六、Coding Agent：代码即"创造工具的工具"（ai-agent-book Ch5）

- **元能力**：普通能力做具体事；元能力"能创造其他能力"。代码生成即元能力——精确、可执行、可组合，当场写新工具/新约束/新表达，无需预预制全部能力。
- **7 个核心工具箱**：Code Interpreter（沙盒 Python）/ Bash / Read / Write / Edit / Glob / Grep。
- **通用 Agent = Coding Agent + 文件系统**：以工作空间为中枢；长期记忆存 `MEMORY.md`（Markdown 而非向量库，便于人改 + Git 版本）；Deep Research / Computer Use / Coding 三融合，**Coding 是核心**（PPT/Word 本质是 OOXML 代码，PDF 由 MD/HTML/LaTeX 生成，数据分析由 Python 完成）。
- **Harness 工程原则（让 Agent 不做错事）**：约束优先于指导（Linter/CI 强制 > 文档建议）、验证自动化（测试/类型检查）、故障边界控制（并行失败不波及父级）、回退可靠（Git/沙盒快照）。
- **自举/动态扩展**：遇无 SDK 系统现场生成适配代码；Computer Use 成功序列固化为 RPA；日志格式变化自动生成解析脚本。

```python
# 元能力应用：前端解析失败 → Agent 现场写适配代码并热更新
def adaptive_parser_on_fail(raw_sample, error):
    prompt = f"写 parse(log_str)->dict，样本:{raw_sample}, 错:{error}"
    gen_code = llm_generate(prompt)          # 动态创造工具
    if sandbox_test(gen_code): hot_reload_frontend(gen_code)
    return "parser updated"
```

## 七、模型容错与流量保护

- **模型容错**：封装 `LLMClient` 支持多候选（DeepSeek/GLM/OpenAI）+ `tenacity` 重试 + 超时降级；首包超时/空响应自动切换。
- **限流**：`asyncio.Semaphore` / `redis` 令牌桶控制并发；batched 请求降成本。
- **成本**：开启 `streaming` 提升体感；缓存相同 Query 的 Embedding/答案（语义缓存，键含租户+TTL）。

## 八、可观测与安全护栏

- **可观测**：LangSmith / LangFuse 追踪每次 Run 的 Node 耗时、输入输出、工具调用、Token 消耗；OpenTelemetry 接入 Prometheus/Grafana。
- **安全**（理论 [`1.7`](../../01-通用理论原理层/1.7-治理与安全原则.md)）：
  - Prompt 注入检测：输入过规则 + LLM 判别，隔离系统指令与用户内容（`<external_content>` 标记来源）。
  - 输出审核：敏感词/合规校验后再返回。
  - 数据隔离：多租户记忆按 `user_id` 分区，检索带租户过滤；缓存键含授权范围。
  - 审计：记录工具调用与决策链路备查。
- **流式**：FastAPI `StreamingResponse` 分事件输出（思考/正文/来源）；客户端断开即取消上游请求。

## 九、设计模式实战（Python 提炼）

- 策略（检索通道/后处理替换）、工厂（`Tool`/`Retriever` 创建）、模板方法（并行检索/模型请求）、**注册表（`TOOL_REGISTRY` 装饰器注册 / discover_tools 元工具）**、装饰器（重试/缓存/观测）、责任链（后处理/降级）、中间件（记忆注入/上下文压缩/工具批处理）、适配器（模型方言/MCP）、状态机（LangGraph StateGraph）、事件回调（流式/SSE）、AOP（`@trace`/`@audit` 装饰器）。

## 十、双栈对照表（规划"双栈对齐"核心交付）

| 能力 | Python 实现 | Java 实现（03 栈） |
|------|------------|-------------------|
| 编排框架 | LangGraph StateGraph / AutoGen / 自研 HelloAgents | Spring AI / Ragent framework / Nexus 执行器 |
| 执行范式 | ReAct / Workflow / Supervisor | v1 Workflow / v2 Agentic / 三层执行器 |
| 上下文压缩 | KV Cache 字节级稳定 + 成批压缩 + 状态栏 | 三层上下文（控 Token） |
| 记忆 | Chroma/Redis/向量库 + JSON Cards/User as Code | Redis/MySQL/向量库 + 持久化摘要 |
| RAG 多路 | 向量+BM25+Neo4j，RRF(k=60)+重排 | 向量+关键词+图谱+联网，RRF+Rerank |
| 工具/MCP | `mcp` SDK / `@tool` / ACI 通用优先 / 主动发现 | Spring Bean 工具 / mcp-server |
| Coding Agent | 7 工具箱 + 元能力自举 + MEMORY.md | （Java 侧以业务集成工具为主） |
| 容错 | tenacity 重试 + 多模型切换 | 档位 + 三态熔断降级 |
| 观测 | LangSmith/LangFuse/OTel | LangFuse/管理端 Trace |
| 流式 | FastAPI StreamingResponse | Spring WebFlux SSE |
| 安全 | 注入检测 + 输出审核 + 租户隔离 | Sa-Token + 数据归属 + Schema 校验 |

> 结论：两条栈的**工程本质相同**——分层隔离、检索工程、上下文/记忆治理、模型容错、全链路观测、安全护栏；差异仅在语言生态与框架。学习任一栈后，另一栈可快速迁移。

## 十一、ai-agent-book 实验地图（Ch2-5 已落位，按章映射阶段）

| 章节 | 主题 | 实验数 | 映射阶段 |
|------|------|-------|---------|
| Ch2 上下文工程 | KV Cache、提示消融、注入攻防、Skills | 2-1~2-7（7） | 阶段2（三） |
| Ch3 记忆与知识库 | 记忆策略对比、BM25、混合检索、RAPTOR/GraphRAG、上下文感知检索 | 3-1~3-12（12） | 阶段2（三/四） |
| Ch4 工具 | 主动工具发现、感知/执行/协作 MCP 服务器 | 4-1~4-5（5） | 阶段2（五） |
| Ch5 Coding Agent | 轨迹接管、代码化规则、PPT/视频生成、自适应日志解析 | 5-1~5-13（13） | 阶段2（六） |
| Ch7 评估 | 评估环境/指标/统计显著性 | 7-1~7-14（14） | 阶段3 |
| Ch8 后训练 | SFT/RL、工具调用内化 | 8-1~8-19（19） | 阶段2 进阶 / 横向 |
| Ch9 持续进化 | 从轨迹更新知识/指令/程序/参数 | 9-1~9-9（9） | 阶段5 |
| Ch10 多 Agent | 协作框架、上下文共享/隔离 | 10-1~10-6（6） | 阶段1 / 理论 1.5 |

> 全 109 实验按章映射后，Python 栈各阶段均有可运行实验支撑；建议逐章把对应实验代码沉淀进 `code/` 并链接至本文件。

## 十二、对知识库的沉淀建议

- 将上述"分层架构 / 执行范式 / KV Cache / RRF / ACI / MCP 主动发现 / Coding Agent 元能力 / 设计模式"提炼为 `阶段2` Python 子主题，与 Java 栈逐条对照。
- 下一步可补：hello-agents Ch7 自研框架源码拆解、ai-agent-book Ch8 后训练 / Ch9 持续进化 / Ch10 多 Agent 实验逐章落地、LangGraph/AutoGen/LlamaIndex 最新版本代码范式。
