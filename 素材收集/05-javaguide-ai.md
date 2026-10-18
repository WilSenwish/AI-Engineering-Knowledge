# javaguide.cn/ai/ · JavaGuide AI 应用开发知识体系素材

> 来源：https://javaguide.cn/ai/（GitHub 156K+ Star 的 JavaGuide 新增 AI 应用开发专栏）
> 开源配套：https://github.com/Snailclimb/AIGuide
> 定位：面向**后端/Java 开发者**的 AI 应用开发体系，串联大模型基础、Agent、RAG、MCP、Prompt、系统设计，强调"把 Demo 放进真实后端系统"。
> 落位：知识库**Java 生态生产实践栈 + 部分通用理论**，对应规划"阶段1 架构设计 / 阶段2 开发实现 / 阶段3 测试 / 阶段4 部署 / 阶段6 治理"。
> 采集：已抓取专栏入口（完整文章地图 + 建议阅读顺序）与《RAG 基础概念》深解；其余子页（Agent/MCP/Skills/Harness/安全/网关/评测等）待补。

---

## 一、专栏核心观点（踩坑预警）

- 大模型不能只当黑盒 API：Token 截断、采样参数漂移、JSON 输出失稳，需在调用链路做格式校验/重试/兜底/异常处理。
- Agent 难点在 Memory 与 Context Engineering：长任务易偏题、循环绕死，靠"执行侧驾驭"而非只调 API。
- RAG 答非所问多在**召回阶段**：Chunk 切粗、Query 未改写、关键词与向量未结合、重排缺失——逐项排查比换贵模型有用。
- MCP/Function/Tool Calling 解决"怎么接工具"，生产难点在后面：谁能调、能操作哪些数据、调用审计、失败回滚。
- 上线后才暴露稳定性/可观测/成本/质量回归问题，Demo 阶段感受不到。

---

## 二、建议阅读顺序（知识库可复用为"AI 应用开发学习路径"）

1. AI 核心概念总览（LLM/Token/Agent/RAG/MCP/Skills/ReAct 串到一条链路）
2. AI 应用开发面试指南（高频追问清单）
3. LLM 运行机制 / 大模型 API 调用工程实践（调用链路、上下文、结构化返回）
4. AI Agent 核心概念 / 提示词工程 / 上下文工程
5. 多 Agent 协作系统设计（任务拆分/状态共享/冲突处理/失败恢复）
6. RAG 基础概念 / 文档处理与切分 / 检索优化
7. AI 应用系统设计 / LLM·Agent 安全实战 / 大模型网关 / AI 应用评测体系

---

## 三、核心文章地图（按专题）

### 面试与路线
- Java/Go 开发者 AI 应用开发与 Agent 学习路线：https://javaguide.cn/roadmap/java-to-ai-roadmap.html
- 后端转型 AI Agent 学习建议：https://javaguide.cn/roadmap/backend-to-ai-agent-roadmap.html
- AI 核心概念总览：https://javaguide.cn/ai/ai-core-concepts.html
- AI 应用开发面试指南：https://javaguide.cn/ai/interview-questions/ai-interview-guide.html
- 大模型/Agent/RAG/AI 系统设计 面试题总结：https://javaguide.cn/ai/interview-questions/

### 大模型基础
- 专题：https://javaguide.cn/ai/llm-basis/
- LLM 运行机制（Token/上下文窗口/采样参数）：https://javaguide.cn/ai/llm-basis/llm-operation-mechanism.html
- 大模型 API 调用工程实践：https://javaguide.cn/ai/llm-basis/llm-api-engineering.html
- 结构化输出（JSON Schema/Function Calling/Tool Calling/MCP）：https://javaguide.cn/ai/llm-basis/structured-output-function-calling.html
- AI 应用评测体系（Golden Set/LLM-as-Judge/Trace 回放/灰度）：https://javaguide.cn/ai/llm-basis/llm-evaluation.html

### AI Agent
- 专题：https://javaguide.cn/ai/agent/
- Agent 核心概念：https://javaguide.cn/ai/agent/agent-basis.html
- Agent 记忆系统：https://javaguide.cn/ai/agent/agent-memory.html
- 多 Agent 协作系统设计：https://javaguide.cn/ai/agent/multi-agent.html
- 提示词工程：https://javaguide.cn/ai/agent/prompt-engineering.html
- 上下文工程：https://javaguide.cn/ai/agent/context-engineering.html
- 万字拆解 MCP：https://javaguide.cn/ai/agent/mcp.html
- 万字详解 Agent Skills：https://javaguide.cn/ai/agent/skills.html
- Harness Engineering（六层框架）：https://javaguide.cn/ai/agent/harness-engineering.html
- Workflow/Graph/Loop：https://javaguide.cn/ai/agent/workflow-graph-loop.html
- Loop Engineering：https://javaguide.cn/ai/agent/loop-engineering.html

### RAG 检索增强
- 专题：https://javaguide.cn/ai/rag/
- RAG 基础概念：https://javaguide.cn/ai/rag/rag-basis.html ✅ 已深解（见下）
- 文档处理与切分：https://javaguide.cn/ai/rag/rag-document-processing.html
- 向量索引与向量库：https://javaguide.cn/ai/rag/rag-vector-store.html
- 检索优化：https://javaguide.cn/ai/rag/rag-optimization.html
- GraphRAG：https://javaguide.cn/ai/rag/graphrag.html
- 知识库更新策略：https://javaguide.cn/ai/rag/rag-knowledge-update.html

### AI 系统设计
- 专题：https://javaguide.cn/ai/system-design/
- AI 应用系统设计：https://javaguide.cn/ai/system-design/ai-application-architecture.html
- LLM/Agent 安全实战：https://javaguide.cn/ai/system-design/llm-security.html
- 大模型网关：https://javaguide.cn/ai/system-design/llm-gateway.html
- AI 语音技术：https://javaguide.cn/ai/system-design/ai-voice.html

### 关联专题
- AI 编程实战：https://javaguide.cn/ai-coding/
- 系统设计 / 高可用 / 高性能 / 分布式：https://javaguide.cn/system-design/ 等

---

## 四、长文深解 ①：《RAG 基础概念》

### 1. 什么是 RAG
检索增强生成：先从知识库检索相关片段，再与问题一起喂给 LLM 基于证据回答，而非只靠训练记忆。

### 2. 为什么需要 RAG（补 LLM 三短板）
- **知识时效性**：训练截止后新事件/政策/文档，靠动态检索注入。
- **私有数据访问**：只提相关片段给 LLM，不暴露全量企业数据。
- **幻觉控制**：提供明确参考文本降低编造概率（非彻底消除，需配引用校验/拒答/反馈闭环）。

### 3. 传统搜索 vs RAG（选型）
- 用户目标是"找到材料"→搜索框更直接、低成本、可审计；"读完给结论"→RAG。
- 企业常**两套入口并存**：简单查找走搜索，复杂问答走 RAG。

### 4. 工作原理（两阶段）
- **离线索引**：输入→清理→增强元数据→Chunking（兼顾语义完整/Embedding 长度/上下文窗口/召回粒度）→Embedding→存入向量库（Milvus/pgvector/ES/Faiss）。
- **在线检索**：请求→(Query 改写)→查询向量化→相似性检索→上下文增强(A)→LLM 生成(G)→(反馈)。

### 5. Embedding 与相似度
- 把文本映射高维稠密向量，语义近→空间近。维度 768/1024/1536/3072，非"越高越好"（存储/算力成本）。
- 闭源 API（OpenAI/Cohere/Jina）vs 开源（BGE/GTE/E5）按出网/私有化需求选；MTEB 仅参考，需业务集评测。
- 相似度：余弦（RAG 最常用，对长度不敏感）/ 内积（L2 归一化后≈余弦）/ 欧氏（对幅度敏感）。

### 6. RAG vs 微调
- RAG 解决"不知新知识/私有知识"；微调解决"不会按你的方式说/做"。
- 结合：微调懂领域术语/格式，RAG 供实时可追溯证据；资源有限先稳 RAG 再考虑微调。

### 7. 长上下文能否取代 RAG → 不能
- 长上下文贵、慢、易噪声；百万级文档塞不下、成本延迟扛不住；Lost in the Middle（中间信息被忽略）；权限隔离仍需应用层；可追溯性 RAG 更优。

### 8. RAG 演进三阶段
- Naive RAG（切块→Embedding→TopK→生成，Demo 级）
- Advanced RAG（Query Rewrite/HyDE→混合检索→Rerank→上下文压缩）
- Modular RAG（检索器/重排器/压缩器/路由器/生成器可插拔，适合生产/Agent）

### 9. 优势与局限
- 优势：知识更新成本低、减幻觉可溯源、数据隔离易做、换领域成本低。
- 局限：检索质量决定上限（GIGO）、上下文非越长越好、延迟高、工程复杂（增量索引/权限/溯源/评测）、Token 成本高。

> 实战项目：interview-guide（Spring Boot 4.0 + Java 21 + Spring AI + PostgreSQL + pgvector + RustFS + Redis），https://github.com/Snailclimb/interview-guide

---

## 五、对知识库的价值

- **Java 后端视角的 AI 工程化认知框架**：把大模型基础/Agent/RAG/MCP/系统设计串成一条链路，可直接作为规划"阶段1 架构设计"的 Java 侧理论素材。
- **与 xiaolinnote 互补**：xiaolinnote 偏原理图解，JavaGuide 偏"放进真实后端系统"的工程取舍（网关/安全/评测/可观测），正好补 Java 实践栈。
- **待补重点**：Agent 记忆/上下文工程/MCP/Skills/Harness/Workflow-Graph-Loop、RAG 切分与优化、安全实战、大模型网关、评测体系——均为规划各阶段 Java 栈核心，建议优先补采。

## 六、补采子页（MCP / Skills / Workflow-Graph-Loop / 安全 / 网关 已抓 ✅）

### ①《万字拆解 MCP》（ai/agent/mcp.html）
- **定位**：MCP = MCP Client 与 Server 之间的**通信协议**（Host 承载交互+模型调用，Client 连 Server，Server 暴露能力）；解决"同一工具交付多个 Host 时重复集成"，把 N×M 降为 N+M。
- **与 FC/Agent 关系**：FC 产生命令（结构化调用意图）→ MCP 传递命令并连接工具 → Agent 决定链路何时继续/结束。
- **能力类型**：Tools（执行动作）/ Resources（只读上下文，如文件/日志/Schema）/ Prompts（可复用提示模板）；进阶 Client→Server 能力：Roots（目录范围）、Sampling（Server 请求 LLM 生成）、Elicitation（向用户补充询问）。
- **传输**：JSON-RPC 2.0（偏方法调用，贴合工具调用）；stdio（本地子进程，stdout 不能打日志）+ Streamable HTTP（远程，可走网关鉴权限流）。
- **生产接入 checklist**：Schema/版本（字段单位、时间格式、枚举写清，工具带版本号+灰度回滚）；权限安全（目录/库/API 边界，读写分离，高危人工确认，脱敏，防路径遍历/SQL/命令注入，三方 Server 源码依赖审核）；可观测（Trace ID + 结构化日志 + 调用链）；成本归因（关联用户/业务线/工具）；依赖治理（SDK/三方 Server 维护者与更新）。
- **Server 设计**：避免 `execute_sql`/`file_operation` 万能工具，按业务动作拆分（动词+名词 + description 写清适用/禁用场景）；大文件按 chunk（~100KB）读取；安全不靠模型，靠目录校验+沙箱+限流熔断。

### ②《万字详解 Agent Skills》（ai/agent/skills.html）
- **定义**：Skill = 写在 `SKILL.md` 中、可被发现按需读取的**任务说明**（规定"怎么做"，非工具能力）；解决跨会话规则复用 + 上下文腐化（Context Rot）治理。
- **SKILL.md 结构**：YAML 元数据（name≤64 小写连字符；description 写清①做什么②何时用+触发词）+ 正文指令（默认方案/项目约束/IO 格式/失败处理，建议≤500 行，长内容拆 references/）。
- **与 MCP 区别**：Prompt(说要做什么) → FC(发动作) → MCP(接外部能力) → Skill(规定流程与约束)。协同：宿主把 Skill 简介放进上下文→模型路由命中→延迟加载完整 SKILL.md→按流程调工具。
- **三层渐进式披露**：广告层（仅元数据）→ 指令层（命中读正文）→ 资源层（按需读 references/scripts）。
- **企业价值**：团队经验资产化（评审/慢SQL/报表固化）；Checklist+验证节点防跳步；按任务风险分配自由度（迁移/部署低自由度+回滚，评审高自由度）；精准加载缓解窗口退化。

### ③《Workflow / Graph / Loop》（ai/agent/workflow-graph-loop.html）
- **三者关系**：Workflow 是目标与过程，Graph 是结构与载体（Node/Edge/State），Loop 是图上的控制模式（回边）。
- **Agent Loop vs Graph Loop**：Agent Loop = 顶层运行引擎（推理→行动→观察 直到完成）；Graph Loop = 图内部特定节点子集通过回边迭代修正（可嵌套在 Agent Loop 某节点内）。
- **Loop 防死循环三要素**：继续条件 + 退出条件 + 安全边界（最大轮次+超时+Token 预算）；常叠加固定上限（条件负责继续，轮次/超时/预算强制退出）；支持嵌套循环（外层质量迭代/内层工具重试，各自独立退出）。
- **Spring AI Alibaba Graph 落地**：`OverAllState`+`KeyStrategyFactory`（Replace/Append 策略）、`NodeAction`、`addEdge`/`addConditionalEdges`、条件边回指形成回边、`MemorySaver`/`RedisSaver` 持久化、`interruptBefore`+`updateState` 人机协同；与 LangGraph 概念一一对应。
- **弹性模式**：瞬时错误指数退避重试；LLM 可恢复错误塞回 State 循环；用户可修复错误 `interruptBefore` 转人工；意外错误冒泡；熔断器（连续格式校验失败降级）；舱壁隔离（Semaphore/Resilience4j）；补偿事务 Saga。
- **坑**：State 太粗/太细都不好（按业务含义分块）；Loop 放大 Token/延迟（先粗筛再精修、够好即停）；节点间数据传早期统一 schema。

### ④《LLM/Agent 安全实战》（ai/system-design/llm-security.html）
- **主要风险**：Prompt Injection（直接/间接/ jailbreak）、工具越权与过度代理（万能工具）、敏感数据泄露/跨租户、供应链（恶意 MCP/Skill/知识库投毒）、代码执行隔离失效、内容违规、多 Agent 级联。
- **防护（Java/Spring 落地）**：
  - 鉴权与身份分离：`ExecutionContext(tenantId,userId,authorities)` 来自登录态不进模型可改参数；执行前服务端用 `tenantId+orderId` 重算权限，不信任模型租户字段。
  - 四层校验：`validateProposal`(结构)→`refundPolicy.check`(语义)→`authorizationService`(资源)→风险策略；`IdempotencyKey` 服务端生成绑定 `commandHash`，审批 Token 绑定参数。
  - 审计：`ToolAuditEvent` 记请求/租户/用户/工具/参数Hash/风险级/决策/幂等键/状态，PII 不全文入库。
  - Spring AI 安全：仅传当前身份允许的 `ToolCallback`；`ToolCallingManager` 归一化+鉴权+幂等；`ToolCallingAdvisor` 控预算/阻断。
- **设计原则**：模型输出不可信（视为待校验建议）；后端兜底（鉴权/最小权限/审批/隔离阻止未授权）；全面不可信输入；最小权限精细拆分；分层防御；安全 Badcase 随模型/Prompt/Schema/MCP/Skill/权限版本化回归。

### ⑤《大模型网关》（ai/system-design/llm-gateway.html）
- **定位**：API 网关能力 + 模型调用控制面，位于应用层与模型供应商之间的**统一治理入口**（未必是独立服务，可应用内轻量模块）。
- **Router vs Gateway**：Router 只选模型；Gateway 管整次调用全生命周期（鉴权/限流/路由/Fallback/观测/成本）。
- **核心价值（直连痛点）**：模型名写死→注册表配置化；Key 分散→统一密钥；供应商限流→限流/Fallback/熔断；成本不可见→usage+归因；日志缺失→Trace+Prompt 版本；SDK 分散→Provider Adapter 统一封装。
- **能力**：多模型统一接入（OpenAI 兼容入口 + Provider Adapter 转各家协议/工具/流式/usage）；模型路由（按 scene/输入 Token/套餐/风险/模型状态，绑定 tier 而非具体模型名，Model Registry 映射）；限流配额（用户/租户/模型/供应商/Token 五维，estimate→reserve→usage→reconcile 预算）；观测审计（request_id+route_reason+usage 脱敏留存）。
- **自研组件**：API Adapter / Auth-Tenant / Prompt Renderer / Token Budget Estimator / Model Registry / Router / Provider Adapter / Retry-Fallback / RateLimiter / Cost Tracker / Observability / Audit Log；第一版先统一 API+Adapter+日志，再补路由/Fallback/预算。
- **选型**：起步轻量自研（先收口模型调用）；多供应商快速接入看 LiteLLM；已有 Kong 看 Kong AI Gateway；已在 Cloudflare 看 Cloudflare AI Gateway；智能路由等评测集+Trace 具备后再引。顺序：**先工程治理，再智能路由**。

## 七、补采子页（Agent记忆 / 上下文工程 / Harness / 评测 / 提示词 / 多Agent 已抓 ✅）

### ⑥《Agent 记忆系统》（ai/agent/agent-memory.html）
- **二分**：短期记忆（Session 级，服务当前任务）vs 长期记忆（跨 Session，沉淀偏好/历史决策），物理逻辑都该分开。存储形式三分类：Token级（向量库文本块/JSON）/参数化（预训练/LoRA/SFT）/潜在（KV Cache/激活值）；MemOS「记忆立方体」支持纯文本↔激活↔参数动态流转。
- **功能分类**：事实/经验/工作记忆；再细分情景/语义/程序记忆。生命周期：编码→存储→提取→巩固→反思→遗忘。
- **短期控制三法**：上下文缩减（滑动窗口/压缩）、卸载（重型结果放外部存储留 UUID/路径引用）、隔离（主 Agent 只交子任务片段）。
- **长期 = Record & Retrieve**：写入异步 LLM 语义提纯（Best-Effort，幂等 Key 用 sourceId+batchId 非文本哈希）；检索向量化 Query 取回 prepend，P99 吃 TTFT（Redis 预热/浅层画像全量预载）。
- **长期记忆 vs RAG**：技术像但服务对象不同——RAG 服务共享知识源，长期记忆管理特定用户个性化经验，可分开召回统一排序。
- **存储三层**：VectorStore / GraphStore(Neo4j) / Reranker；产品 Mem0/LETTA/MemGPT/ZEP/MemOS/MIRIX。
- **高级演化**：自我反思、Reflect Loop、聚类合并；遗忘 `score=relevance×importance×decay(t)`，冲突标废弃定期 Vacuum。
- **检索优化**：混合检索（BM25+Dense），RRF/Linear/Cross-encoder 融合；多租户硬过滤，高活跃用独立 Collection；**先修检索链路再看写入**。
- **Markdown 记忆**：少量可审计信息写 CLAUDE.md/AGENTS.md/rules，Git 版本化零迁移；与向量库边界分明、可并存。

### ⑦《上下文工程》（ai/agent/context-engineering.html）
- **定义**：*the art of providing all the context for the task to be plausibly solvable*；比 PE 管得更大（窗口里放什么/怎么放/何时进出）。PE 是告诉厨师怎么做菜，CE 是备好厨房；类比 LLM 内存管理（页面置换/LRU）。
- **Context Assembler**：约束→目标→证据(RAG)→记忆→工具→历史摘要，再 `rank`+`fit_token_budget`。静态规则拆角色/目标/约束/执行流/输出格式存 rules/AGENTS.md；工具描述要答「何时该调/不该调」；动态上下文别一股脑塞，排障必留原始引用。
- **运行时加载**：预检索（FAQ/稳定文档）/ 按需加载 JIT+Progressive Disclosure（代码库分析/排查）/ 混合（生产主流）。
- **长任务三法**：Compaction（压缩历史摘要）、Structured Note-taking（NOTES.md 外部化）、Sub-agent（独立上下文，主 Agent 只收 1-2K 摘要）。
- **Token 预算优先级**：低(早期历史→摘要)/中(背景/旧结果→裁剪)/高(系统约束/目标/安全→固定区)/阶段(当前工具/Schema)。
- **失效根因**：Context Rot、Lost in the Middle；对策删重复+约束放明确位置+长文档先切分/检索/摘要+保留可回查引用。
- **评估 5 指标**：任务成功率/工具质量/上下文成本/延迟/结果质量；先建 20-50 条轨迹基线，每次只改一个变量。黄金法则：*do the simplest thing that works*。

### ⑧《Harness Engineering 六层框架》（ai/agent/harness-engineering.html）
- **等式**：`Agent = Model + Harness`；三层递进 PE→CE→Harness（长链路低容错商业场景 Harness 成主要矛盾）。
- **六层**：L1 信息边界 / L2 工具系统 / L3 执行编排 / L4 记忆与状态 / L5 评估与观测 / L6 约束校验恢复；起步先补 L1+L6。
- **瓶颈常在模型外**：Can.ac 仅换文件编辑接口得分 6.7%→68.3%；LangChain 优化后 Terminal Bench 2.0 第30→第5（模型不变）。model-harness 耦合，选型以任务为准。
- **Context 越多越蠢**：168K 用到 ~40% 后质量降（Smart/Dumb Zone，非普适阈值）；Anthropic「上下文焦虑」→ context resets。
- **建设优先级**：P0（AGENTS.md≤100行当目录、自定义 Linter+修复、知识进仓库）/ P1（分层上下文、进度 JSON、端到端验证、控利用率）/ P2（专业化分工、垃圾回收、可观测性）。
- **成熟度**：L0 无→L1 基础约束→L2 反馈回路→L3 专业化 Agent→L4 自治循环。
- **案例**：OpenAI(3人5月100万行0手写, 机械化约束)/ Anthropic(16并行Opus写C编译器、GAN三智能体)/ Stripe Minions(周1300+PR无人值守)/ Hashimoto(单Agent每行失败案例)/ Böckeler(Context/Constraints/GC 三类)。

### ⑨《AI 应用评测体系》（ai/llm-basis/llm-evaluation.html）
- **评测 vs 监控**：评测开发阶段主动发现+发布前阻断回归；监控上线后持续观察漂移，是延伸非替代。公开 benchmark 不能覆盖业务分布/脏输入/低频致命失败。
- **四类评测**：离线(冻结 Golden Set)/ Trace 回放 / 线上灰度 / 回归 CI(能力集+回归集)。
- **指标分层**：检索(Recall@k/Hit Rate/MRR)、生成(Faithfulness/Relevance/Completeness/Noise)、Agent(完成率/工具精确召回/一致性 pass@k)、结构化(JSON/Schema)、安全成本(越权/幻觉/TTFT/Token)、代理(满意/追问/转人工)。
- **Golden Set**：来源(生产分层采样/人工构造/失败回填/冷启动RAGAS)；蜕变测试留保留集；分层比总量关键，高权重失败单独门禁；接 CI 一票否决。
- **LLM-as-Judge**：Reference-based / Reference-free；规则+Judge+人工三层；防位置/冗长/自我强化/有限推理四类偏差；需人工校准集验一致率。
- **Trace 回放与灰度**：回放须存正文快照/哈希，否则页面更新后失效；灰度看效应量+置信区间非死板百分比；持续监控双轨(快照回归+能力探针)。
- **Java 落地**：EvalRecord(EvalRun 单条全字段) + EvalRunSummary；CI 阈值为绝对(Faithfulness<0.75)+相对(完成率降≤5%)；PR 阶段只跑核心回归集。

### ⑩《提示词工程》（ai/agent/prompt-engineering.html）
- **四要素**：Role/Task/Context/Format；不看长度看是否说清。角色与格式放输入两端规避 Lost in the Middle。
- **技巧**：角色扮演、CoT(数学/逻辑/reasoning model 不假设拿原始推理)、少样本(1-3)、任务分解、结构化输出(先定 Schema, JSON/XML/YAML/Markdown 各有利弊, 须降级重试)、XML标签+预填充、链式提示。
- **降幻觉**：允许说不知道、引用原文、多次采样(Best-of-N/一致性)、迭代验证。
- **安全**：Prompt 注入(直接/间接)三层防护(权限隔离/分隔符/高危审批)；越狱与注入区别。
- **工程**：提示词路由(识别类型选链路)、RAG 混合检索(BM25+语义+Rerank+HyDE)、工具系统设计(名称友好/原子/读写分离)、回归集维护(固定基线每次只改一变量)。

### ⑪《多 Agent 协作系统设计》（ai/agent/multi-agent.html）
- **优先简单架构**：先单 Agent/固定 Workflow；拆分信号=不同工具权限上下文/可独立子方向/能产出结构化交付物。
- **编排模式**：Sequential/Parallel/Router/Supervisor/Handoff/Evaluator-Optimizer/Group Chat/Debate/Event-driven，各有权衡（见 1.5 表格）。
- **任务拆分**：`AgentTask` 契约(子目标/依赖/allowedTools/outputSchema/maxIters/timeout)；并行判定与副作用单一执行者/幂等。
- **状态共享**：进程内参数/黑板、跨服务消息/A2A；只留必要信息，原始材料存 Artifact 库只交摘要+引用+URI。
- **冲突处理**：状态写分槽/CAS、观点带 scope 交归并器、所有权 Lease+Fencing、副作用幂等+唯一约束。
- **失败恢复**：分类处理+状态机+Checkpoint+补偿(Saga)+HITL；通信显式传 TaskContext 忌 ThreadLocal。
