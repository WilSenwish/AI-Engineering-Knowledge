# xiaolinnote.com/agent/ · 图解 Agent 素材

> 来源：https://xiaolinnote.com/agent/
> 定位：小林"图解 Agent"专栏，以万字长文 + 手绘图解方式深度拆解 Agent / RAG / 工程方法论（Harness / Loop / Graph Engineering）。区别于 /ai/ 的逐题拆解，本栏目是**体系化长文**。
> 落位：知识库**通用理论层 + 工程方法论**，对应规划"阶段1 架构设计 / 阶段2 开发实现 / 阶段5 运维观测"。
> 采集：已抓取 `agent.html`(概念)、`harness-engineering.html`、`graph-engineering.html` 三篇长文；`rag.html`、`openclaw.html`、`loop-engineering.html`、`loop_engineering_handbook.html`、`graphrag-lightrag.html` 待补。

---

## 一、栏目结构

- **Agent 概念与生态**：`concept/agent.html`(Agent 是什么) · `concept/openclaw.html`(OpenClaw 生态)
- **RAG 检索增强**：`rag/rag.html`(RAG 是什么) · `rag/graphrag-lightrag.html`(GraphRAG/LightRAG)
- **Agent 工程方法论**：`engineering/harness-engineering.html` · `engineering/loop-engineering.html` · `engineering/loop_engineering_handbook.html` · `engineering/graph-engineering.html`

---

## 二、长文提炼 ①：《AI Agent 是什么》

### 1. LLM 的四大弊端 → Agent 的解法
- 弊端：只会说不会做、没有记忆、不会用工具、不会规划。
- Agent 引入：工具调用（做事）、记忆系统（连贯）、MCP 等标准化协议（接工具）、任务拆解规划（自主）。

### 2. Agent 四大核心组件
大脑(LLM) + 规划模块 + 记忆模块 + 工具模块。"LLM 告诉怎么做，Agent 直接做完"。

### 3. Agent vs Workflow（核心区别：谁控制流程）
- Workflow：LLM+工具经预定义代码路径编排，控制权在代码，确定/可复现/省 token（约 Agent 的 1/4），调试易。
- Agent：LLM 动态主导流程与工具调用，灵活但不确定、调试难。
- Anthropic 建议：从最简单方案起步，明确需要才加复杂度；生产多为**混合架构**（如客服：Workflow 控流程，复杂环节起 Agent）。

### 4. 工作模式
- ReAct（边想边做）、Plan-and-Execute（先想后做）、Reflection（自我反思 / 双 Agent Code Review）、Multi-Agent（协作）。

### 5. 底层能力分层（极重要，面试高频）
| 概念 | 角色 | 比喻 |
|------|------|------|
| Function Call | 基础调用能力 | "打电话的能力" |
| MCP | 统一工具连接标准 | "公司通讯录/电话系统"（AI 界 USB-C） |
| Skills | 做事方法/知识层 | "岗位培训手册/菜谱" |
| A2A | Agent 间协作协议 | 横向通信（MCP 是竖向） |

- **Function Call**：一次性单步调用，是 Agent 的"原子操作"；Agent 是 Function Call 的"高级编排"（一个任务可能十几次 FC）。
- **MCP**：Host(宿主) / Client(翻译官) / Server(暴露工具) 三角色；把 N×M 集成降为 N+M；工具可发现、运行时动态接入。
- **Skills**：`SKILL.md` 声明式定义（YAML 元数据 + 自然语言指令），教 Agent "怎么做"。MCP=厨房(能做什么)，Skills=菜谱(该怎么做)。
- **A2A**（Google 2025.4）：Agent Card(名片) / Task(任务生命周期) / Message&Artifact；基于 HTTP+JSON+SSE，支持异步长任务；与 MCP 互补（竖向接工具、横向 Agent 协作）。

---

## 三、长文提炼 ②：《Harness Engineering》

### 1. 定义与职责
- Harness = 包裹模型、驾驭其"执行任务"的整套运行环境（Mitchell Hashimoto 提出，OpenAI 背书）。
- 关注点从"输入侧"转移到"执行侧"：监督、约束、拉回跑偏、出错爬起。
- 实践定义：Agent 每次犯错，不是手改，而是**工程化一个永久方案**（改 AGENTS.md、加 linter、补测试、Git Hook），沉到环境里不再犯。

### 2. 为什么需要（越用越聪明的秘密）
- Prompt ⊂ Context ⊂ Harness：三代工程视角层层放大（听懂 → 知道 → 执行侧驾驭）。
- **复利效应**：每次犯错修复沉到环境 → 环境变强 → 更少犯错 → 改进更快。OpenAI Codex 固化"Golden Principles"，后台 Agent 每天自动扫偏离开修复 PR。

### 3. 六层架构
1. 上下文管理：just-in-time retrieval 防 context rot；规则文件做渐进式披露（控 100 行索引，细节懒加载）。
2. 工具调度：可控接入（MCP），明确"给什么工具/何时用/结果如何回喂"。
3. 任务编排：ReAct / Plan-and-Execute 给定"工作轨道"。
4. 记忆与状态：状态外化到文件系统（claude-progress.txt、git history），分层（任务/会话/长期偏好 user-preferences.md）。
5. 可观测与评估：独立 Eval 集作"尺子"；Trace+日志+指标（LangSmith）把调试从"猜"变"看"。
6. 安全/约束与恢复：约束硬编码（最多分析 20 PR / 超 token 停）；每步硬校验；失败预案（限流重试、本地队列）。

### 4. 与 Agent 内核的关系
- 核心等式：`Agent = Model + Harness`（或 `Harness = Agent - Model`）。
- 模型决定能力天花板；Harness 决定能否落地生产。未来工程师主战场是"设计让 Agent 高质量产出的环境"。

---

## 四、长文提炼 ③：《Graph Engineering 多 Agent 图式编排》

### 1. 是什么
- 按任务依赖关系用 **graph（节点+边）** 编排流程：该串行串行、该并行并行，每块交给职责明确的节点。
- `loop ⊂ graph`：一个 loop 是最小 graph（单节点自环）；graph 管"节点之间"，loop 管"节点内部"。

### 2. 三要素
- **Node（节点）**：干活的单元——agent / 固定代码 / DB 查询 / 人（审批节点）。好节点：只干一件事、可单测、可替换。
- **Edge（边）**：决定下一节点。确定性边（代码写死，如测试通过→部署）/ 模型决策边（如工单分类）/ 条件边（可代码或模型判断，如测试通过→审批、失败→返工）。原则：能写死尽量写死，模型判断力留给少数语义分支。
- **State（状态）**：字段固定的 JSON 对象，记录进度/产物/token。节点完成后存档 → 中断=读档重试单节点，而非从头跑。

### 3. 价值与适用
- 解决串行流：慢、无存档、难暂停审批、难表达复杂分工。
- 适用：可拆独立子任务且需并行缩短耗时；不同环节需不同模型/工具/权限；需人工确认环节；长时多分支需追踪进度。
- 不适用：小任务顺序执行即可（"一个人一下午能干完，别开项目启动会"）。前提：先确保每个节点稳定，再谈编排。

### 4. 实现
- LangGraph：`StateGraph` + `add_node` / `add_edge` / `add_conditional_edges` / `compile()`。graph engineering 是设计方法，LangGraph 是实现框架之一（也可用 AutoGen/ADK/Claude Code workflow/自写调度）。
- Claude Code：subagent=节点、hook=固定检查、workflow=调度脚本（自然语言派 agent），人类节点=最终确认；`/deep-research` 即已落地的 graph 工作流。

---

## 五、补采长文（rag / graphrag-lightrag / loop-engineering 已抓 ✅）

### ③《RAG 是什么》（rag.html，约 19679 字）
- **定位**：RAG = 给大模型"开卷考试"——先检索外部知识库相关片段，再与问题一起喂给 LLM 基于证据回答；解决知识截止、私有数据、幻觉三困境。
- **两阶段流程**：索引（离线：文档加载→切割→Embedding→向量库）/ 查询（在线：问题向量化→相似度检索→构造增强 Prompt→生成）。
- **关键工程点**：
  - 切分：固定大小→语义切分，建议递归字符切分起步；避免一句话被切断。
  - Re-rank：Cross-Encoder 精排，准确率提升 30-50%；**先检索 50-100 候选，重排到 Top-5/10** 性价比最高；简单事实查询可设阈值触发。
  - Embedding 演进：Word2Vec→BERT→BGE-M3→**第四代 LLM-based（LLM2Vec / Qwen3-Embedding / Seed1.5-Embedding，2025 中英文 SOTA）**。
  - 向量库：HNSW/IVF/PQ 索引做 ANN 近似最近邻；Milvus/Pinecone/Weaviate 等按场景选型。
  - 多路召回：向量 + BM25 混合，效果比单路好 10-30%。
- **评估**：RAGAS（GPT-4 当裁判，算 Faithfulness/Relevance 等）、DeepEval、TruLens、ARES。
- **降幻觉**：RAG 是核心手段但不能完全消除，需组合策略；复杂推理可用 GraphRAG/LightRAG。

### ④《GraphRAG 与 LightRAG》（graphrag-lightrag.html）
- **GraphRAG（微软 2024.4）**：LLM 抽实体/关系建知识图谱；解决传统 RAG 三大痛点——跨文档多跳推理失效、全局归纳无解、切块语义断裂；Local Search（图遍历邻居）+ Global Search（Leiden 社区检测+社区摘要，Map-Reduce）。代价：索引贵（"Token 焚烧炉"）、增量级联失效、延迟高。
- **LightRAG（港大 2024.10）**：GraphRAG 轻量改良——去社区层（省 80%+ LLM 调用）、**双层检索（低层实体+高层主题并行）**、追加式增量（新文档 upsert）。Token 成本降 ~99%（百万 token 约 $0.15 vs $20-50），延迟 1-3s。
- **选型**：GraphRAG=重型武器（深度全局洞察、跨文档高价值推理、静态少更新、强可解释）；LightRAG=轻巧刀具（频繁更新、成本敏感、C 端实时、10万-500万 token 甜蜜区）。务实路径：传统 RAG→遇跨文档/全局问题升 LightRAG→精度不足再上 GraphRAG。
- **关系**：LightRAG 保留 Naive 模式兼容传统向量 RAG，是 GraphRAG 去社区层的轻量子集。

### ⑤《Loop Engineering》（loop-engineering.html，约 5997 字）
- **四代 engineering 演进（瓶颈迁移）**：Prompt（怎么说）→ Context（喂什么）→ Harness（驾驶舱/环境）→ **Loop（最后一环：坐在键盘前的你）**。模型每强一截，瓶颈外移一层。
- **定义**：把"亲自写 prompt 的你"替换成"设计代替你做这件事的系统"；loop = 递归式目标，AI 持续迭代直到完成。杠杆支点从"说一句话"移到"设计一套制度"。
- **一个 loop = 五大件 + 一份磁盘记忆**：
  1. 自动化（心跳：定时/事件触发）
  2. worktree（并行隔离，防 agent 改同一文件打架）
  3. skill（固化项目知识，治"金鱼记忆"，知识复利）
  4. connector（基于 MCP 接外部系统：issue/监控/PR）
  5. sub-agent（**写代码与检查代码分离**，B 用干净上下文挑刺）
  6. 记忆（磁盘上的状态文件："agent 会忘，但 repo 不会"）
- **真实 loop 示例**：晨间 triage——自动触发→分诊 skill 写状态文件→每问题开 worktree 派 sub-agent 起草→第二个 sub-agent 审→connector 开 PR→搞不定的进收件箱→写回状态文件（明天接着跑）。
- **三盆冷水（代价）**：① 验证仍归你（检查 agent 的"done"只是声明非证明）；② 理解债 Comprehension Debt（产出越快、你越不懂）；③ 认知投降 Cognitive Surrender（带判断力设计是解药，逃避思考是助燃剂）。
- **工具无关**：Codex / Claude Code 部件一一对应，loop 图纸是资产，放在哪家都能转。

> 待补：`concept/openclaw.html`（OpenClaw 生态）、`engineering/loop_engineering_handbook.html`（落地手册）。
