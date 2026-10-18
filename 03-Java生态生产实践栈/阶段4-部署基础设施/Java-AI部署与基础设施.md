# 阶段4 · Java AI 部署与基础设施

> 分级：🟡 生产最佳实践 ｜ 来源：`素材收集/03-nageoffer-ragent.md`、`素材收集/04-javaup-nexus-agent.md`、`素材收集/05-javaguide-ai.md`（大模型网关）
> 定位：规划"阶段4 部署基础设施·Java 生态"。Java AI 服务多为**常驻微服务**，与 Python 推理服务协作是典型部署形态。

## 一、应用打包与运行

- **打包**：Spring Boot fat jar（或 Spring Boot 3 的 `native-image` GraalVM 编译，启动快但 AI 生态兼容性需验证）。
- **JVM 调优**：AI 场景关注堆外内存（Netty/PgVector 连接池）、GC 停顿；Ragent 用 **10 个专用线程池 + TTL 上下文透传** 隔离检索/模型/入库任务，部署时需保证线程池参数经压测。
- **配置防错**：启动期一致性校验（如必填模型档位、向量库连通），早失败优于运行期异常。

## 二、容器化与编排

- **镜像**：多阶段 Dockerfile（build 阶段 Maven/Gradle，runtime 用 `eclipse-temurin` 精简 JRE）。
- **K8s**：Deployment + Service；HPA 按 CPU/QPS 扩缩（注意 JVM 堆随副本线性增长，设合理 request/limit）。
- **有状态依赖**：PGVector（`PostgreSQL` + `pgvector`）、Elasticsearch、Neo4j（Nexus 文档结构图谱）、Redis（限流/记忆）以 StatefulSet 或独立托管（RDS/云厂商）部署，与应用 Deployment 分离。

## 三、Java 与 Python 推理服务协作

企业常见**混合部署**：Java 负责业务编排/权限/事务，Python（vLLM/TensorRT-LLM）负责 GPU 推理。

- **架构**：Java 微服务 + Python 推理 Sidecar/独立服务，通过 HTTP/gRPC 调用。
- **模型网关**：大模型网关（统一鉴权/限流/路由/审计/降级/多模型管理）部署在 Java 服务与模型之间（详见 `05-javaguide-ai.md` 网关子页）。**顺序：先工程治理，再智能路由**。
- **MCP Server**：`mcp-server` 模块可独立部署为远程 MCP 服务，Java Agent 通过网络发现调用（非硬编码）。

## 四、异步与消息基础设施

- **文档入库异步流水线**：Nexus 用 **Kafka** 驱动分步任务（解析→切块→向量化→入库），每步任务日志可追；Ragent 用 **RocketMQ 事务消息**保障分块可靠执行。
- **流式**：SSE 长连接需网关/Ingress 配置超时放大，避免被反向代理截断。

## 五、与 Python 栈阶段4 对照

| 部署点 | Python 栈 | Java 栈 |
|--------|----------|--------|
| 服务形态 | FastAPI/Uvicorn/Gunicorn（ASGI） | Spring Boot（Servlet/WebFlux） |
| GPU 推理 | 同进程 vLLM/TensorRT-LLM 常见 | 多侧车/独立推理服务 |
| 网关 | 同 | 大模型网关位置一致，Java 侧更强调与既有 Kong/Spring Cloud 网关融合 |
| 有状态依赖 | 向量库/Redis 同 | PGVector/ES/Neo4j/Redis 同 + 消息中间件（Kafka/RocketMQ） |
| 可观测基建 | 同 Prometheus/Grafana | 同 + Actuator/JVM 指标 |

## 六、Agent 部署形态系统性梳理

> 源自《战略洞察-Java工程能力移植与企业级AI应用形态》（"二、企业级 AI 应用/Agent 是不是基本都偏 Web 服务端"一节）。**核心判断**：企业级 AI 的核心服务层绝大多数是 Web 服务形态，但完整形态是「Web 服务为核心 + 多端嵌入 + 离线调度」的组合，并非只有 Web 一种形态。

### 6.1 六种部署形态对照

| 形态 | 交付方式 | 技术载体 | Java 工程能力复用度 | 典型场景 |
|------|----------|----------|------------------|----------|
| ① Web 服务为核心 | HTTP/HTTPS + SSE/WebSocket | Spring Boot / 大模型网关 / API 网关 | 极高（95%+） | 对话、流式输出、多轮交互、中心化管控 |
| ② 嵌入式 Agent | SDK / 插件 / 本地轻量逻辑 + 云端重算 | 客户端 SDK + 云端 API | 中（云端部分高） | 内部代码助手、办公助手、IDE/浏览器插件 |
| ③ 离线批处理 Agent | Worker 节点 / 定时任务 / 消息消费 | 定时调度 + 消息队列 Worker | 极高（与传统异步处理一致） | 批量文档分析、数据巡检、报表生成 |
| ④ 边缘私有化 Agent | 轻量容器 / 独立服务 | 私有化部署、数据不出域 | 高（工程治理能力通用） | 客户现场、边缘服务器、信创/合规环境 |
| ⑤ 工作流组件 | 流程引擎能力节点 | Camunda / 工作流引擎节点 | 高（BPM 集成经验复用） | 嵌入企业既有工作流 |
| ⑥ 分布式调度平台 | 管理后台(Web) + 独立 Worker | 调度集群 + Worker 池 | 高（调度治理通用） | 大量 Agent 任务分布式调度 |

### 6.2 各形态工程要点

- **Web 为核心（①）**：标准化对接（HTTP/WebSocket/SSE）、交互天然适配、中心化管控（权限/审计/多租户/计费/监控）、框架默认形态（Spring AI/LangChain/AutoGen 服务化最终都是 Web API）。这是 Java 企业级工程能力降维打击的主战场。
- **嵌入 / 离线 / 边缘（②③④）**：需要"有状态 Agent"管理（会话快照、中断续跑、状态机），区别于无状态 Web；离线/边缘不对外暴露 Web 接口，只消费消息与输出结果。Java 的线程池/调度/容灾经验在此完全通用。
- **工作流 / 调度（⑤⑥）**：把 Agent 能力封装为流程节点或调度任务，复用既有 BPM、定时调度、分布式调度（XXL-JOB / 云调度）经验；执行节点是独立 Worker，不直接处理 Web 请求。
- **选型原则**：核心管控放 Web 中心服务（保证企业级治理），重计算/私有数据走边缘或云端，批量与定时走 Worker——既保住企业级治理能力，又覆盖非 Web 形态。

### 6.3 衔接
- → `阶段2-开发工程实现`：长会话状态管理、工具调用编排（对应 ②⑤ 的有状态需求）。
- → `阶段5-运维观测`：六种形态的监控/链路差异。
- → 根目录《战略洞察-Java工程能力移植与企业级AI应用形态》：本节的原始论述与全库落位映射。
