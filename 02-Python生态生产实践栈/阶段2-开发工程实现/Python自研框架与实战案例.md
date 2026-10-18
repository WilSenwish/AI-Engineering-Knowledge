# Python 自研框架与实战案例（hello-agents / HelloAgents）

> **本文件定位**：与同目录 `Python企业级Agent工程实践.md`（基于 ai-agent-book 的生产范式）**互补**——
> 本文聚焦 **「从零手撕框架 + 可运行代码 + 端到端案例」**，把理论范式落到具体代码。
> - 理论范式（上下文/KV Cache、记忆/RAG、工具/MCP、Coding Agent）→ 见 `Python企业级Agent工程实践.md`
> - 本文提供：自研框架骨架、记忆与检索的具体实现、上下文工程管线、3 个完整可运行案例
>
> **来源**：[datawhalechina/hello-agents](https://github.com/datawhalechina/hello-agents) 第 7/8/9/13/14/15 章（含 `Hello-Agents` 仓库代码）。
> **配套原码**：同目录 `code-examples/`（hello-agents `code/` 逐字落盘的**真实可运行脚本**，11 个文件 / 1860 行，含 README 索引）。本范式文档讲"提炼"，`code-examples/` 给"原码"，两者结合形成完整闭环。

---

## 一、自研 Agent 框架范式（Ch7 构建你的 Agent 框架）

### 1.1 设计哲学：万物皆为工具

HelloAgents 的核心简化思想——**「万物皆为工具」**：不做 Memory / RAG / MCP 的独立模块，而是把这些都**抽象成 `Tool`**。这样框架只有 `Agent / LLM / Tool` 三个核心概念，认知负担最低，扩展最灵活（记忆后端、检索、外部协议都只是某种 Tool 的实现）。

### 1.2 分层架构（core / agents / tools）

```
core/        核心抽象层
  agent.py       Agent(ABC) 顶层抽象
  llm.py         HelloAgentsLLM 统一模型接口（多厂商切换）
  message.py     Message 标准化消息
  config.py      Config 配置中心（from_env）
  exceptions.py  异常
agents/       实现层（均继承 core.Agent）
  simple_agent.py / react_agent.py / reflection_agent.py / plan_solve_agent.py
tools/        工具层
  base.py        Tool(ABC) 统一抽象
  registry.py    ToolRegistry 注册/调度/转 OpenAI schema
  chain.py       ToolChain 工具链
  async_executor.py  异步并行
  builtin/       calculator / search 等内置工具
```

### 1.3 核心抽象与接口

| 基类 / 组件 | 职责 | 关键接口 |
|------------|------|---------|
| `Agent(ABC)` | 智能体顶层抽象 | `@abstractmethod run()`；`add_message()`、`get_history()`（管理 `_history: list[Message]`） |
| `HelloAgentsLLM` | 模型调用中枢 | `invoke()`、`think()`(流式)、`stream_invoke()`；`provider` 多厂商切换 |
| `Tool(ABC)` | 工具统一抽象 | `@abstractmethod run(parameters: Dict)->str`、`get_parameters()` |
| `ToolRegistry` | 工具管理中枢 | `register_tool()`、`register_function()`、`execute_tool()`、`to_openai_schema()` |
| `Message(BaseModel)` | 标准化消息 | `content, role, timestamp, metadata`；`to_dict()` 转 OpenAI 格式 |
| `Config(BaseModel)` | 配置中心 | `from_env()` 读环境变量 |

### 1.4 ReAct 主循环（核心可运行代码）

```python
# core/agent.py —— Agent 抽象基类
from abc import ABC, abstractmethod

class Agent(ABC):
    def __init__(self, name, llm, system_prompt=None, config=None):
        self.name = name; self.llm = llm
        self.system_prompt = system_prompt
        self.config = config or Config(); self._history = []

    @abstractmethod
    def run(self, input_text: str, **kwargs) -> str: ...
    def add_message(self, message): self._history.append(message)
    def get_history(self): return self._history.copy()
```

```python
# agents/react_agent.py —— ReAct 主循环
def run(self, input_text: str, **kwargs) -> str:
    self.current_history = []; current_step = 0
    while current_step < self.max_steps:
        current_step += 1
        prompt = self.prompt_template.format(
            tools=self.tool_registry.get_tools_description(),
            question=input_text, history="\n".join(self.current_history))
        resp = self.llm.invoke([{"role": "user", "content": prompt}], **kwargs)
        thought, action = self._parse_output(resp)
        if action and action.startswith("Finish"):
            final = self._parse_action_input(action)
            self.add_message(Message(input_text, "user"))
            self.add_message(Message(final, "assistant"))
            return final
        if action:
            t_name, t_input = self._parse_action(action)
            obs = self.tool_registry.execute_tool(t_name, t_input)
            self.current_history.append(f"Action: {action}")
            self.current_history.append(f"Observation: {obs}")
    return "抱歉，无法在限定步数内完成"
```

> 提示词约束 LLM 输出 `Thought:` 与 `Action: tool[input]` 或 `Finish[answer]`；循环内调用 LLM → 解析 → 执行工具 → 把 Action/Observation 追加进历史 → 直至 Finish 或超 `max_steps`。

### 1.5 记忆与工具调用机制

- **短期记忆**：Agent 内置 `self._history: list[Message]`，`run` 时遍历拼入 messages。
- **长期记忆**：按"万物皆为工具"理念，写成 `memory` 工具（如 `[TOOL_CALL:memory:recall=用户信息]`），由 Registry 执行——**无独立 Memory 基类**。
- **工具调用三种范式**：
  - `SimpleAgent`：正则解析 `[TOOL_CALL:name:params]` → `registry.execute_tool`
  - `ReActAgent`：解析 `name[input]` → `registry.execute_tool`
  - `FunctionCallAgent`：`Tool.to_openai_schema()` 转原生 OpenAI function calling

```python
# tools/base.py + registry.py —— 工具基类与注册
class Tool(ABC):
    def __init__(self, name, description): self.name = name; self.description = description
    @abstractmethod
    def run(self, parameters: Dict[str, Any]) -> str: pass
    @abstractmethod
    def get_parameters(self): pass

class ToolRegistry:
    def __init__(self): self._tools = {}; self._functions = {}
    def register_tool(self, tool: Tool): self._tools[tool.name] = tool
    def register_function(self, name, description, func):
        self._functions[name] = {"description": description, "func": func}
    def execute_tool(self, name, param):
        if name in self._tools: return self._tools[name].run(param)
        if name in self._functions: return self._functions[name]["func"](param)
```

### 1.6 框架扩展点

1. **新 LLM**：继承 `HelloAgentsLLM` 重写 `__init__`，拦截 `provider="xxx"` 设专属 `base_url/api_key`；或在 `_auto_detect_provider` 中增加环境变量识别。
2. **新工具 / 新记忆后端**：继承 `Tool` 实现 `run()`（记忆后端可在 `run` 内接向量库），或 `registry.register_function` 快注册。
3. **新 Agent 范式**：继承 `Agent` 实现 `run()`（如 Tree-of-Thought）。
4. **高级**：`ToolChain` 串联多工具；`AsyncToolExecutor` 线程池并行；`FunctionCallAgent` 原生 schema 转换。

> **与 Java 栈对照**：HelloAgents 的「Agent/LLM/Tool 三抽象 + 万物皆为工具」与 Ragent/Nexus 的「分层执行器 + 工具即组件」思路一致；Java 侧用 Spring Bean 注册工具，Python 侧用 `ToolRegistry` 注册，本质都是**统一工具抽象 + 注册中心**。

---

## 二、记忆与检索工程实现（Ch8 记忆与检索）

### 2.1 记忆四分类与存储映射

借鉴人类认知模型，框架将记忆分为四类，映射到不同存储后端：

| 记忆类型 | 定位 | 生命周期 | 存储方案 | 用户画像归属 |
|---------|------|---------|---------|------------|
| 工作记忆 Working | 短期/临时上下文 | TTL 自动清理（默认 60min），≤50 条 | 纯内存（Python list） | 临时会话状态 |
| 情景记忆 Episodic | 长期具体事件 | 持久化 | SQLite（结构化）+ Qdrant（向量） | 记录"用户某日完成某项目" |
| 语义记忆 Semantic | 长期抽象知识/规则 | 持久化 | Neo4j（图）+ Qdrant（向量） | **用户画像首选**（importance=0.8） |
| 感知记忆 Perceptual | 多模态数据 | 动态 | SQLite + Qdrant（按模态分集合） | 图像/音频元数据 |

> 用户画像与偏好作为抽象知识写入 `semantic` 类型；跨会话持久化依赖外部数据库，而非临时 `_history`。

### 2.2 RAG 处理 Pipeline

```
任意格式文档 → MarkItDown 转换 → Markdown → 智能分块 → Embedding → 向量库 → 检索/重排
```
1. **文档处理**：`MarkItDown` 统一转 Markdown；PDF 走增强解析，失败有 `_fallback_text_reader` 兜底。
2. **切分策略**：先按 Markdown 标题层次（`#/##`）语义分割，再按 Token 估算（支持中英文混合）切块，带 overlap 重叠。
3. **Embedding**：统一接口 `get_text_embedder()`，优先百炼 API → 本地 `all-MiniLM-L6-v2` → TF-IDF 兜底；维度异常自动补零/截断。
4. **向量检索**：Qdrant 存储，基础向量检索。
5. **重排 / 高级检索**：
   - **MQE 多查询扩展**：LLM 生成同义问法，提升召回。
   - **HyDE 假设文档嵌入**：LLM 先写假答案，用答案向量搜真文档，缩小语义鸿沟。
   - **融合**：扩展查询结果去重、按 `score` 排序，候选池 `candidate_pool_multiplier=4`。

### 2.3 工程化要点（与理论书的互补视角）

- **工作记忆非语义检索**：用 TF-IDF 词法相似度 + 关键词匹配（非稠密向量），避免误用嵌入导致内存浪费。
- **维度不匹配防护**：文本/图像/音频建立**分离 Qdrant 集合**，逃避维度冲突。
- **遗忘曲线工程化**：时间近因性采用指数衰减 `exp(-decay_factor * age_hours / 24)`，最低保底 0.1 分。
- **评分公式透明**：语义记忆混合分 = `(向量相似度×0.7 + 图相似度×0.3) × (0.8 + 重要性×0.4)`，可直接调参。

### 2.4 可复用代码

```python
# 记忆读写（语义/工作记忆）
from hello_agents.tools import MemoryTool
memory_tool = MemoryTool(user_id="user123")
memory_tool.run("add", content="用户张三是一名Python开发者",
                memory_type="semantic", importance=0.8)
res = memory_tool.run("search", query="Python开发者", limit=3, memory_types=["semantic"])
```

```python
# RAG 知识库构建与基础检索
from hello_agents.tools import RAGTool
rag = RAGTool(knowledge_base_path="./kb", rag_namespace="test")
rag.execute("add_text", text="Python由Guido于1991年发布", document_id="py_intro")
hits = rag.execute("search", query="Python历史", limit=3, min_score=0.1)
```

```python
# 高级扩展检索（MQE + HyDE 融合）
def retrieve_expanded(store, query, top_k=8, enable_mqe=True, enable_hyde=True):
    expansions = [query]
    if enable_mqe: expansions += _prompt_mqe(query, 2)     # LLM 生成同义查询
    if enable_hyde:
        h = _prompt_hyde(query)                            # LLM 生成假设文档
        if h: expansions.append(h)
    agg = {}
    for q in set(expansions):
        qv = embed_query(q)
        for hit in store.search_similar(qv, limit=top_k*4,
                where={"memory_type": "rag_chunk"}):
            mid = hit["metadata"]["memory_id"]
            agg[mid] = hit if mid not in agg else max(agg[mid], hit, key=lambda x: x["score"])
    return sorted(agg.values(), key=lambda x: x["score"], reverse=True)[:top_k]
```

> **配套实战**：`code/chapter8/11_Q&A_Assistant.py`（基于 Gradio 的 PDF 问答助手，集成 Memory+RAG）；安装 `pip install "hello-agents[all]==0.2.0"` + 配置 `.env`（Qdrant/Neo4j/Embedding）。

---

## 三、上下文工程实现（Ch9 上下文工程）

> 与 ai-agent-book 第2章（KV Cache / 提示工程理论）互补：本节给出**可运行的 ContextBuilder 管线代码**。

### 3.1 上下文构成与分区

上下文由以下类别组成，在 `ContextBuilder` 中固化为分区模板：

| 类别 | 说明 | 框架分区 |
|------|------|---------|
| 系统提示/指令 | 角色、行为准则、工具指引 | `[Role & Policies]`（relevance=1.0，最高优先级） |
| 用户消息 | 当前查询 | `[Task]` |
| 知识/证据 | RAG 检索、few-shot 示例 | `[Evidence]`（type: rag_result/knowledge/example） |
| 工具说明 | 最小可行工具集 MVTS | 系统提示内描述 |
| 状态/历史 | 对话历史（默认 5 条）、记忆检索 | `[Context]` |
| 示例 Few-shot | 精选样例 | 动态 `custom_packets` 或并入 `[Evidence]` |
| 输出要求 | 输出格式约束 | `[Output]` |

骨架输出顺序：`[Role & Policies]` → `[Task]` → `[Evidence]` → `[Context]` → `[Output]`。

### 3.2 选择 / 压缩 / 结构化 / 按需加载

- **Gather 汇集**：容错多源拉取——系统指令、MemoryTool 检索、RAGTool 检索、最近对话（默认 5 条）、自定义包。
- **Select 选择**：综合评分 = `relevance_weight*相关性 + recency_weight*新近性`；过滤 `min_relevance`；贪心填充至 `available_tokens`（系统指令预留 `reserve_ratio`）。相关性默认 Jaccard 相似度。
- **Structure 结构化**：按类型分组写入固定 Markdown 分区，便于调试。
- **Compress 压缩**：超限时按分区截断，保持结构完整。
- **JIT 按需加载**：`TerminalTool` 执行 `grep/find/head` 沙箱命令即时探索文件；`NoteTool` 持久化笔记，每轮前 `search/list` 动态注入，避免预加载全部数据。

### 3.3 可复用代码

```python
# 上下文结构化组装器
def structure_context(selected_packets, user_query):
    system_instructions, evidence, context = [], [], []
    for packet in selected_packets:
        ptype = packet.metadata.get("type", "general")
        if ptype == "system_instruction":
            system_instructions.append(packet.content)
        elif ptype in ["rag_result", "knowledge", "example"]:
            evidence.append(packet.content)
        else:
            context.append(packet.content)
    sections = []
    if system_instructions:
        sections.append("[Role & Policies]\n" + "\n".join(system_instructions))
    sections.append(f"[Task]\n{user_query}")
    if evidence:
        sections.append("[Evidence]\n" + "\n---\n".join(evidence))
    if context:
        sections.append("[Context]\n" + "\n".join(context))
    sections.append("[Output]\n请基于以上信息,提供准确、有据的回答。")
    return "\n\n".join(sections)
```

```python
# 上下文压缩函数（分区截断，保持结构完整）
def compress_context(context: str, max_tokens: int) -> str:
    def count_tokens(text):
        chinese = sum(1 for ch in text if '\u4e00' <= ch <= '\u9fff')
        english = len(text.split())
        return int(chinese + english * 1.3)
    def truncate(text, max_tok):
        char_per_tok = len(text) / count_tokens(text) if count_tokens(text) > 0 else 4
        return text[:int(max_tok * char_per_tok)]
    if count_tokens(context) <= max_tokens:
        return context
    sections = context.split("\n\n")
    compressed, total = [], 0
    for sec in sections:
        sec_tok = count_tokens(sec)
        if total + sec_tok <= max_tokens:
            compressed.append(sec); total += sec_tok
        else:
            rem = max_tokens - total
            if rem > 50:
                compressed.append(truncate(sec, rem) + "\n[... 内容已压缩 ...]")
            break
    return "\n\n".join(compressed)
```

```python
# 动态 Few-shot / 笔记转 ContextPacket
from dataclasses import dataclass
from datetime import datetime
from typing import Optional, Dict, Any

@dataclass
class ContextPacket:
    content: str
    timestamp: datetime
    token_count: int
    relevance_score: float = 0.5
    metadata: Optional[Dict[str, Any]] = None

def build_fewshot_packets(examples: list, query: str, limit=2):
    packets = []
    for ex in sorted(examples,
                     key=lambda e: len(set(e['q'].split()) & set(query.split())),
                     reverse=True)[:limit]:
        content = f"示例输入: {ex['q']}\n示例输出: {ex['a']}"
        packets.append(ContextPacket(
            content=content, timestamp=datetime.now(),
            token_count=len(content)//4, relevance_score=0.85,
            metadata={"type": "example", "source": "fewshot"}))
    return packets
# 用法：builder.build(user_query=q, custom_packets=build_fewshot_packets(examples, q))
```

### 3.4 与记忆模块的协作回路

```
记忆提供候选 → ContextBuilder 筛选结构化 → LLM 推理 → 新记忆/笔记写回
```
- `MemoryTool`：`_gather` 中 `memory_tool.search` 获取相关记忆 → 转 `ContextPacket` 注入 `[Context]`；推理后 `memory_tool.add` 写回摘要。
- `NoteTool`：长时程任务每轮前 `note_tool.search/list` 检索笔记，按类型设权重（blocker=0.9, action=0.8）转包传入。
- `TerminalTool`：探索文件所得可存记忆或直接转包，完成"JIT 检索 → 记忆 → 上下文"闭环。

---

## 四、综合实战案例（Ch13 / 14 / 15）

### 4.1 智能旅行助手（Ch13）—— 多 Agent 流水线 + MCP

**架构**：四层（Vue3 前端 / FastAPI 后端 / HelloAgents 智能体层 / 高德 MCP + Unsplash + LLM 外部服务）。
**多 Agent 协作**：4 个单一职责 Agent 顺序流水线——
- `AttractionSearchAgent`（景点搜索）、`WeatherQueryAgent`（天气）、`HotelAgent`（酒店）、`PlannerAgent`（行程规划与预算，纯 LLM 不调工具）。

**关键工程决策**：
- 通过 **MCP** 集成 `amap-mcp-server`（`MCPTool(auto_expand=True)` 自动暴露 16 个工具如 `amap_maps_text_search`/`amap_maps_weather`）。
- **所有子 Agent 共享同一个 MCPTool 实例**——避免每 Agent 各启进程耗资源、超 API 速率限制。
- 预算计算在 Planner 的 Prompt 中要求 LLM 估算四大类（门票/酒店/餐饮/交通）并汇总。

```python
class TripPlannerAgent:
    def __init__(self):
        settings = get_settings()
        self.llm = HelloAgentsLLM()
        self.mcp_tool = MCPTool(
            name="amap_mcp", command="npx",
            args=["-y", "@sugarforever/amap-mcp-server"],
            env={"AMAP_API_KEY": settings.amap_api_key},
            auto_expand=True)  # 单进程、16 工具共享
        self.attraction_agent = SimpleAgent(name="AttractionSearchAgent", llm=self.llm, system_prompt=ATTRACTION_AGENT_PROMPT)
        self.attraction_agent.add_tool(self.mcp_tool)
        self.weather_agent = SimpleAgent(name="WeatherQueryAgent", llm=self.llm, system_prompt=WEATHER_AGENT_PROMPT)
        self.weather_agent.add_tool(self.mcp_tool)
        self.hotel_agent = SimpleAgent(name="HotelAgent", llm=self.llm, system_prompt=HOTEL_AGENT_PROMPT)
        self.hotel_agent.add_tool(self.mcp_tool)

    def plan_trip(self, request: TripPlanRequest) -> TripPlan:
        attr_resp = self.attraction_agent.run(f"请搜索{request.city}的{request.preferences}景点")
        weather_resp = self.weather_agent.run(f"请查询{request.city}的天气")
        hotel_resp = self.hotel_agent.run(f"请搜索{request.city}的{request.accommodation}酒店")
        planner_query = self._build_planner_query(request, attr_resp, weather_resp, hotel_resp)
        planner_resp = self.planner_agent.run(planner_query)  # 纯 LLM 生成 JSON
        return self._parse_trip_plan(planner_resp)
```

**难点与方案**：单 Agent 提示词臃肿 → 拆 4 专用 Agent；直连高德 API 解析乱 → 用 MCP 封装屏蔽 JSON-RPC；多 Agent 各启 MCP 超限额 → 共享实例；前后端字段不一致 → Pydantic `field_validator` 校验；规划耗时 10–30s 无反馈 → 前端 2min 超时 + 模拟进度条。

> **配套原码**：`code-examples/chapter13-旅行助手/`（7 个后端核心文件，430 行的 `trip_planner_agent.py` + 高德 MCP 封装 `amap_service.py` + FastAPI 入口 `api_main.py` 等）。

### 4.2 自动化深度研究智能体 DeepResearch（Ch14）—— TODO 驱动范式

**架构**：四层（Vue3 + SSE 前端 / FastAPI `/research/stream` / HelloAgents 智能体层 / 搜索引擎 + LLM 外部服务）。
**核心工作流**：问题拆解 → 检索 → 阅读 → 综合 → 报告（三 Agent 顺序协作）：

1. **规划**：`TODO Planner` 将主题拆为 3–5 个 TODO 子任务（含 `title/intent/query`）。
2. **执行**：逐任务 `SearchTool` 检索 → `Task Summarizer` 阅读提炼（**强制为每个观点加来源标记 `[1][2]`**）。
3. **报告**：`Report Writer` 整合所有总结，生成结构化带引用报告。

```python
class DeepResearchAgent:
    def run(self, research_topic: str) -> str:
        todo_list = self.planner.plan_todo_list(research_topic)          # 1. 拆解
        task_summaries = []
        for task in todo_list:
            search_results = self.search_service.search(task.query)       # 2a. 检索
            summary, urls = self.summarizer.summarize_task(task, search_results)  # 2b. 阅读
            task_summaries.append((task, summary, urls))
        report = self.reporter.generate_report(research_topic, task_summaries)  # 3. 报告
        return report
```

```python
# 检索后处理：去重 + 单源 Token 限制（防上下文溢出）
def _deduplicate_sources(self, sources):
    seen, unique = set(), []
    for s in sources:
        if s["url"] not in seen:
            seen.add(s["url"]); unique.append(s)
    return unique

def _limit_source_tokens(self, sources, max_tokens=2000):
    max_chars = max_tokens * 4
    for s in sources:
        if len(s["snippet"]) > max_chars:
            s["snippet"] = s["snippet"][:max_chars] + "..."
    return sources
```

**难点与方案**：

| 难点 | 方案 |
|------|------|
| 长上下文/信息过载 | URL 去重 + 单源 ~2000 Token 截断 + 只投喂精简片段给 Summarizer |
| 成本/效率 | `SearchService` MD5 哈希缓存；`.env` 配廉价引擎 |
| LLM 输出可靠性 | Planner 返回 JSON 正则 `r'\[.*\]'` 提取 + 字段校验 + NoteTool 持久化中间态 |
| 过程黑盒/调试难 | `tool_call_listener` 回调监听所有工具调用，经 SSE 实时推送前端 |
| 流式体验 | FastAPI `StreamingResponse` 发 `text/event-stream`，分 `progress/plan/task_summary/report` 事件 |

> **配套原码**：`code-examples/chapter14-deepresearch/`（13 个后端文件，551 行的 `agent.py` 编排主逻辑 + `services_*` 服务化拆分 + `prompts.py` 各阶段模板）。

### 4.3 赛博小镇（Ch15）—— 多 Agent 社会模拟

**架构**：四层（Godot 4.5 前端 / FastAPI 后端 `StateManager`+`RelationshipManager` / HelloAgents 智能体层 / LLM + Qdrant + SQLite 外部服务）。
**核心机制**：
- 每个 NPC 是独立 `SimpleAgent`（独立记忆 + 角色 Prompt）。
- **记忆流**：`WorkingMemory`（短期，容量 10、TTL 120min）+ `EpisodicMemory`（SQLite+Qdrant 向量，`top_k=3` 语义召回）。
- **轻量社会模拟**：`NPCBatchGenerator` 按早/中/晚推断场景上下文，一次 LLM 调用产出所有 NPC 背景对话（成本降至 1/3）。
- **好感度驱动**：LLM 分析对话情感（-3~+5）动态改写 `system_prompt`，五级关系（陌生 0-20 → 挚友 81-100）。

```python
def create_npc_agent(npc_id, name, role, personality):
    system_prompt = f"你是{name},一位{role}。性格:{personality}..."
    llm = HelloAgentsLLM()
    memory_manager = MemoryManager(
        working_memory=WorkingMemory(capacity=10, ttl_minutes=120),
        episodic_memory=EpisodicMemory(
            db_path=f"memory_data/{npc_id}_episodic.db",
            collection_name=f"{npc_id}_memories"))
    return SimpleAgent(name=name, llm=llm, system_prompt=system_prompt, memory_manager=memory_manager)

def process_dialogue(agent, player_message):
    recent = agent.memory_manager.working_memory.get_recent_messages(5)        # 短期
    relevant = agent.memory_manager.episodic_memory.search(query=player_message, top_k=3)  # 长期
    reply = agent.run(player_message, context={"recent": recent, "relevant": relevant})
    agent.memory_manager.add_interaction(player_message, reply)              # 回写
    return reply

async def background_dialogue_update():  # 环境时钟驱动，每 5 分钟
    while True:
        dialogues = get_batch_generator().generate_batch_dialogues()
        for npc_name, dialogue in dialogues.items():
            state_manager.update_npc_background_dialogue(npc_name, dialogue)
        await asyncio.sleep(300)
```

**难点与方案**：并发 LLM 成本高 → `NPCBatchGenerator` 批量生成（成本 1/3）；NPC 呆板 → 双 memory + 好感度改写 Prompt；游戏卡顿 → Godot `HTTPRequest` 异步 + 信号；状态冲突 → `StateManager` 中心化 `is_busy` 校验。

> **配套原码**：`code-examples/chapter15-赛博小镇/`（9 个后端文件，483 行的 `agents.py` 多 Agent 编排 + `relationship_manager.py` 好感度系统 + `state_manager.py` 状态机 + `batch_generator.py` 批量生成）。记忆/好感度/日志的设计说明另见上游 `MEMORY_SYSTEM_GUIDE.md` / `AFFINITY_SYSTEM_GUIDE.md` / `DIALOGUE_LOG_GUIDE.md`。

---

## 五、进阶主题：通信协议 / Agentic-RL / 评估（Ch10 / 11 / 12）

> 三大工程化进阶能力，补全"框架 → 案例 → 协议/训练/评测"的能力闭环。范式讲解见下文，**配套原码**均已落盘至 `code-examples/chapter10-通信协议`、`chapter11-agentic-rl`、`chapter12-评估`。

### 5.1 通信协议：MCP / A2A / ANP（Ch10）
- **MCP（Model Context Protocol）**：把外部能力（高德地图、GitHub、文件系统）封装为标准化工具服务器。关键范式：`MCPTool` 接入（`05_UseMCPToolInAgent.py`）+ 自建 Server（`my_mcp_server.py` / `14_weather_mcp_server.py`），传输层支持 STDIO / HTTP（`04_MCPTransport.py`）。
- **A2A（Agent-to-Agent）**：Agent 之间互调。范式：`09_A2A_Server.py` 暴露能力 + `09_A2A_Client.py` 远程调用 + `10_A2ATool_Simple.py` 把远端 Agent 包装为本地工具；多 Agent 文档协作见 `06_MultiAgentDocumentAssist.py`。
- **ANP（Agent Network Protocol）**：Agent 网络的节点发现/任务分发/负载均衡，`11_ANPInit.py` → `12_ANPTaskDistribution.py` → `13_ANPLoadBalancing.py` 一条龙。
- **与理论层对齐**：MCP/A2A/ANP 即理论层 `1.4 工具体系与通信协议` 的"可运行实现"；Java 侧对应 Ragent `mcp-server`、RuoYi `ruoyi-chat/mcp` 包。

### 5.2 Agentic-RL：用强化学习训练 Agent 策略（Ch11）
- 数据：`01_dataset_loading.py` 加载轨迹数据集。
- 奖励：`02_reward_functions.py` 设计过程/结果奖励（含格式、工具调用正确性、最终答案）。
- 训练：`05_grpo_training.py` **GRPO**（群组相对策略优化，免去独立 critic）；`06_complete_pipeline.py` 串联 SFT→RL→评估；`08_distributed_training.py` 多卡加速。
- 评估：`07_model_evaluation.py` 训练后评测。
- **与理论层对齐**：呼应理论层 `1.7 治理与安全原则` 中"可训练/可对齐"的能力边界，以及评估体系。

### 5.3 评估体系（Ch12）
- **工具调用评测**：BFCL（`02_bfcl_quick_start.py` 快速开始 + `04_run_bfcl_evaluation.py` 完整运行）。
- **通用智能体评测**：GAIA（`05_gaia_quick_start.py`）。
- **数据生成 / LLM-as-Judge**：`07_data_generation_complete_flow.py` 全流程 + `08_data_generation_llm_judge.py` 用 LLM 当裁判 + `09_data_generation_win_rate.py` 胜率对比。
- **与理论层对齐**：即理论层 `1.6 评估体系`（评测 vs 监控、Golden Set、LLM-as-Judge）的"可运行实现"；Java 侧对应 `阶段3-测试质量保障` 的 RAGAS/DeepEval。

> 三章代码均已落盘；完整 40+ 文件见上游 `code/chapter10|11|12/`，本目录只取最具代表性的 27 个（详见 `code-examples/README.md`）。

## 六、与 ai-agent-book 范式对照（速查）

| 主题 | ai-agent-book（生产范式） | hello-agents（本文代码） |
|------|------------------------|------------------------|
| 框架抽象 | `LLM + 上下文 + 工具` 公式 | `Agent / LLM / Tool` 三抽象 + 万物皆为工具 |
| 上下文工程 | KV Cache 字节级稳定、成批压缩、状态栏 | `ContextBuilder` + `ContextPacket` + 分区模板 + 压缩代码 |
| 记忆 | 短期/长期/用户画像分层 | 工作/情景/语义/感知四类 + 存储映射 + 遗忘曲线 |
| RAG | 256–1024 块 + RRF + Cross-Encoder | MarkItDown + MQE/HyDE 融合 + Qdrant |
| 工具/MCP | ACI 原则、主动工具发现 | `MCPTool(auto_expand)` 共享实例 + ToolRegistry |
| Coding Agent | 代码即"创造工具的工具" | ReAct 循环 + `TerminalTool` JIT 探索 |
| 多 Agent | 编排模式对比 | 旅行助手流水线 / 赛博小镇社会模拟 |

> **结论**：ai-agent-book 给出"为什么这么做"的范式与原理，hello-agents 给出"怎么写代码"的骨架与案例。两者结合，Python 栈阶段2 从**范式**落到**可运行代码**，与 Java 栈的 Ragent/Nexus 范例形成完整的「双栈对照」。

---

## 七、待补 / 延伸

- ~~把 Ch7–Ch9、Ch13–Ch15 的 `code/` 实战脚本真实落盘进对应阶段（目前为范式摘录）。~~ ✅ **已完成**：Ch7-9（11 文件）+ Ch13-15（29 文件）已全部落盘至 `code-examples/`，本文「四」三个案例的「配套原码」已分别指向对应目录。
- ~~Ch10 通信协议（MCP/A2A/ANP）、Ch11 Agentic-RL、Ch12 评估 的 Python 代码范式（与理论层 1.4/1.6 对齐）。~~ ✅ **已完成**：27 个代表性文件已落盘至 `code-examples/chapter10-通信协议`、`chapter11-agentic-rl`、`chapter12-评估`，本文「五」三节各指向对应原码。
- `Extra-Chapter` 社区精选：AgentSkills 解读、GUI/Web Agent 实战、Agent 自进化、踩坑经验。
