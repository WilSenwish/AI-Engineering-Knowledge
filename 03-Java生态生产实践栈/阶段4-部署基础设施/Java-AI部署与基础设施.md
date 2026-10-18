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
