# Spring AI 生产级应用范式（llm-apps-java-spring-ai 参考）

> 层级：🔴 待验证方案 → 🟡 生产最佳实践（Spring AI 官方参考示例集，Apache-2.0）｜ 来源：`github.com/ThomasVitale/llm-apps-java-spring-ai`
> 定位：本库 Java 栈差异化补强（G4）。与 `阶段2` 的 Ragent/Nexus 自研范式、`Java平台级AI解决方案-RuoYiAI与JEECG.md` 的平台视角形成"组件级生产范式 / 企业级自研 / 平台产品化"三角互补。

## 一、定位与价值

- 以 **Spring AI + Spring Boot** 为核心，提供"从原型到生产"的 Java GenAI 应用**组件级**代码范式；技术基线为 Java 25 + Podman/Docker（PGVector 等）。
- 与既有 Java 栈的关系：Ragent/Nexus 讲"企业级分层架构与双链路"，RuoYi AI/JEECG AI 讲"平台产品化"，本仓讲"Spring AI 每个能力点怎么写生产代码"——三者互补，不重复。
- 直接印证战略洞察《Java工程能力移植与企业级AI应用形态》：Java 企业级工程能力 70%~80% 可直接复用，Spring AI 把"AI 特有范式"也接进了 Java 体系。

## 二、能力全景（按官方 README 模块归类）

| 模块 | 覆盖能力 | 对应本库理论/实践 |
|------|----------|------------------|
| **Use Cases** | Chatbot / Question Answering(RAG) / Semantic Search / Structured Data Extraction / Text Classification | 阶段2 Agent/RAG 工程化 |
| **Models** | Chat（Mistral/Ollama/OpenAI/多供应商路由）/ Embedding（Mistral/Ollama/OpenAI/ONNX）/ Image（OpenAI）/ Audio（STT·TTS） | 理论 1.3 模型档位路由 |
| **Patterns** | Prompts（基础/消息/模板）/ Structured Output / Multimodality / Tool Calling / Memory（Basic·JDBC·Spring Security·Vector Store）/ Guardrails（输入·输出） | 理论 1.2/1.4/1.7 |
| **Data Ingestion** | Document Readers（JSON/Markdown/PDF/Text/Tika）/ Transformers（Metadata 富化 / Splitters 分块） | 理论 1.3 RAG 文档处理 |
| **RAG** | Sequential（Naive→Advanced）/ Branching（条件分支检索）/ Conditional（按需检索） | 理论 1.3 进阶 RAG |
| **Observability** | LLM（Mistral/Ollama/OpenAI）/ Vector Store（PGVector）Micrometer 指标与追踪 | 阶段5 运维观测 |
| **MCP** | Brave Search Server 接入（Model Context Protocol） | 理论 1.4 通信协议 |

> 注：README 中 Vector Stores / Evaluation / Agents 标注 "Coming soon"，说明该仓仍在演进；落地时以 Spring AI 官方文档为准。

## 三、关键生产范式拆解

### 3.1 多模型供应商抽象与路由（`chat-multiple-providers`）
Spring AI 的 `ChatClient` 抽象屏蔽供应商差异，可运行时按**档位/成本/可用性**切换模型。直接呼应理论 1.3 模型档位路由，以及 Ragent `infra-ai` 的首包探测/熔断。

### 3.2 结构化输出（Structured Output）
把 LLM 自由文本转为 JSON / Java 对象（`BeanOutputConverter` + 结构化提示）。这是 **Agent 工具调用、数据抽取** 的工程基础——把"非结构化生成"收敛为"可校验 I/O 契约"。

### 3.3 记忆的多种持久化（`memory-*`）
| 记忆实现 | 适用 | 企业级含义 |
|----------|------|-----------|
| Basic | 内存会话 | 原型/单实例 |
| **JDBC** | 关系库持久化 | 跨重启会话、对接既有 DB（呼应战略"长会话与状态管理"） |
| **Spring Security** | 多租户隔离记忆 | 租户级数据隔离，对齐 OAuth2/JWT 多租户体系 |
| **Vector Store** | 语义召回记忆 | 记忆即检索，呼应理论 1.2 记忆工程 |

### 3.4 工具调用（Tool Calling）
把 Java 方法暴露为 LLM 可调用工具（Mistral/Ollama/OpenAI 三套实现），是 Agent 执行层基础；与理论 1.4 工具协议、MCP 形成"函数调用 → 标准化协议"的递进。

### 3.5 RAG 流变
- **Sequential Naive→Advanced**：从"检索-拼接-生成"到带重排/压缩的进阶管线。
- **Branching / Conditional**：按问题类型分支检索、按需才检索——对应理论 1.3「进阶 RAG」。
- 文档处理 `splitters`（切块）+ `metadata`（摘要/关键词富化）共同决定召回质量，呼应 05-基础支撑域「RAG 与向量工程」。

### 3.6 可观测性
LLM 调用与 Vector Store 的 Micrometer 指标/追踪，对齐 03-Java `阶段5-运维观测`（LangFuse / Actuator / JVM 指标）。

### 3.7 Guardrails（输入/输出护栏）
敏感词/格式/安全护栏，对齐理论 1.7 治理与安全、阶段6 治理合规。

## 四、与 Java 企业级工程能力映射（呼应战略洞察）

| 维度 | 复用度 | 说明 |
|------|--------|------|
| Spring Boot 打包 / JVM 调优 / 容器 K8s | 极高 | 与既有 Java 部署完全一致 |
| 连接池 / 事务 / 安全（Spring Security·Sa-Token） | 极高 | 企业级基本功直接平移 |
| 可观测（Micrometer / Actuator） | 极高 | 阶段5 已沉淀 |
| 流式（SSE/WebSocket） | 中 | 需补生产级流式处理（异常续传/保活/中断恢复） |
| 模型容错（重试/降级/多模型路由） | 高 | Spring AI 抽象 + Ragent 熔断范式 |
| 成本（Token 统计/按租户计费） | 中 | 需在拦截层补充（见 04-成本ROI） |
| RAG 向量工程 / Prompt 版本化 | 高 | Spring AI + 本仓范式已覆盖 |

**结论**：Spring AI 让"AI 特有范式"也长在 Java 工程体系内，是把战略洞察"用 Java 做核心服务编排"落到代码的关键抓手。

## 五、落位与交叉引用
- → `阶段2-开发工程实现/Java企业级Agent工程实践.md`：企业级 Agent/RAG 工程化（本仓是组件级范式补充）。
- → `阶段4-部署基础设施/Java-AI部署与基础设施.md`：Spring Boot / JVM / K8s 部署。
- → `阶段5-运维观测/Java-AI运维观测.md`：Micrometer / Actuator 可观测。
- → `../01-通用理论原理层/1.3-RAG技术体系.md`、`1.2-上下文与记忆工程.md`、`1.4-工具体系与通信协议.md`：理论底座。
- → `Java平台级AI解决方案-RuoYiAI与JEECG.md`：平台产品化对照（本仓是"自研 Spring AI 代码范式"）。
- → 根目录《战略洞察-Java工程能力移植与企业级AI应用形态》：本仓是战略结论的代码级印证。
