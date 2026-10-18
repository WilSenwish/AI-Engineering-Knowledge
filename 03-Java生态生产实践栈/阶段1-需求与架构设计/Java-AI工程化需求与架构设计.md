# 阶段1 · Java AI 工程化需求与架构设计

> 分级：🟡 生产最佳实践 ｜ 来源：`素材收集/05-javaguide-ai.md`、`素材收集/03-nageoffer-ragent.md`、`素材收集/04-javaup-nexus-agent.md`
> 定位：规划"阶段1 需求与架构设计·Java 生态"。与 `02-Python生态生产实践栈/阶段1` 原理一致、技术栈差异化——Java 侧强调**企业系统集成、既有微服务复用、权限/事务/中间件融合**。

## 一、需求形态判定（先做正确的事）

Java 业务侧落地的首要决策是：**该用 Workflow 还是 Agentic**？直接决定架构复杂度。

| 判定维度 | 倾向 Workflow（Ragent v1） | 倾向 Agentic（Ragent v2 / Nexus） |
|---------|--------------------------|----------------------------------|
| 任务边界 | 明确、可枚举（知识问答、固定流程） | 开放、需规划（多步操作、工具调用） |
| 可控性要求 | 高（金融/合规场景） | 中（允许模型自定路径） |
| 可解释性 | 路径可审计 | 链路长、需全链路 Trace |
| 典型场景 | 企业知识助手、工单分类 | 自动化运营、跨系统操作 Agent |

> **Nexus 经验**：把"路由判定/问题改写/子问题拆分/知识域收缩/歧义检测"等**确定性决策前置到编排中心**，模型只做生成，显著降低不确定性。这是 Java 工程化控制风险的利器。

## 二、框架选型（Java 生态）

| 框架 | 定位 | 适用 | 注意 |
|------|------|------|------|
| **Spring AI** | Spring 生态官方 AI 抽象（ChatClient/VectorStore/ToolCallback） | 新项目、Spring Boot 团队首选 | 版本演进快，注意 1.x→2.x API 变化 |
| **LangChain4j** | 借鉴 LangChain 的 Java 实现 | 习惯 LCEL 风格、链式编排 | 生态较 Python 版小 |
| **AgentScope（Java）** | 阿里多智能体框架，ReAct 执行引擎 | 多 Agent 协作、MCP 工具发现 | Ragent 的 `agent` 模块即基于此 |
| **自研编排** | 基于模板方法/责任链/状态机自搭 | 极致可控、复杂双链路 | 成本高，需沉淀设计模式 |

> **选型原则**：先用 Spring AI 收口模型调用；多供应商快速接入看 LiteLLM 网关；已有 Kong 看 Kong AI Gateway。**顺序：先工程治理，再智能路由**。

## 三、架构落地（模块化 + 分层）

Java 企业级 AI 普遍采用**模块化单体**（非微服务拆分 AI 能力），以隔离"业务编排 / AI 供应商差异 / 通用基础设施"：

- **Ragent 7 模块**：`bootstrap`（装配）/ `framework`（统一响应·幂等·MQ·SSE）/ `infra-ai`（模型客户端·档位·降级）/ `system`（认证·审计）/ `rag`（RAG 问答·入库 Pipeline）/ `agent`（AgentScope ReAct·MCP·技能·记忆）/ `mcp-server`（独立工具服务）。
- **Nexus 6 模块**：用户入口 / 前置编排中心 / 三层执行器 / 知识底座 / 混合检索 / 工程化护栏。

> **分层意图**：切换模型（ChatClient/EmbeddingClient/RerankClient 候选路由）、向量库（VectorStore 配置切换）、对象存储时，**核心问答流程不重写**——靠接口 + 注册表 + 配置隔离实现。

## 四、原型验证（最小可跑）

Java 侧推荐"内存起步"原型，避免过早引入重量依赖：

1. Spring Boot 3.x + Spring AI（或 LangChain4j）+ 内存 `VectorStore`（如 `SimpleVectorStore`）。
2. 一条 RAG 链路跑通：文档→切分→Embedding→检索→Prompt 拼装→流式返回（SSE）。
3. 引入 `mcp-server` 一个本地工具验证工具调用闭环。
4. 验证通过后，再替换向量库为 PGVector/ES、模型为生产档位、加 Trace 与限流。

## 五、与 Python 栈阶段1 对照

| 决策点 | Python 栈 | Java 栈 |
|--------|----------|--------|
| 框架 | LangGraph/AutoGen/LlamaIndex/自研 | Spring AI/LangChain4j/AgentScope/自研 |
| 架构风格 | 多服务、算法侧优先 | 模块化单体、业务侧优先 |
| 核心价值 | 迭代快、AI 原生 | 权限/事务/中间件融合、可控可审计 |
| 原型速度 | 快（脚本即跑） | 略慢（需 Spring 装配）但更贴近生产 |

> 理论底座复用 `01-通用理论原理层`（1.1 Agent 本质、1.5 多 Agent 编排）；详细工程范式见同栈 `阶段2-开发工程实现/Java企业级Agent工程实践.md`。
