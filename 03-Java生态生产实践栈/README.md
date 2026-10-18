# 03-Java 生态生产实践栈

> 定位：企业级集成、业务侧落地（Java 微服务）。与 Python 栈保持原理一致、实践差异化。
> 来源素材：`素材收集/05-javaguide-ai.md`（框架）、`素材收集/03-nageoffer-ragent.md` + `素材收集/04-javaup-nexus-agent.md`（自研范例）、`素材收集/07-ruoyi-jeecg-platform.md`（平台视角）。
> 本栈含"双极对照"：从零自研工程化（Ragent/Nexus，见阶段2）+ 平台/低代码交付（RuoYi AI/JEECG AI，见根文件 `Java平台级AI解决方案-RuoYiAI与JEECG.md`）。

## 本栈阶段地图

| 阶段 | 核心内容 | 已有素材 | 状态 |
|------|---------|---------|------|
| `阶段1-需求与架构设计` | 框架选型（Spring AI/LangChain4j/AgentScope）、AI vs 流程驱动、模块化分层、原型验证 | javaguide 架构 + Ragent/Nexus 架构 | **[已补]** |
| `阶段2-开发工程实现` | 企业级 Agent/RAG 工程化（分层、双链路、检索融合、上下文/记忆、容错、可观测、11 模式） | Ragent + Nexus + javaguide | **[企业级范例]** |
| `阶段3-测试质量保障` | 非确定性断言、Golden Set、RAGAS/DeepEval、关键路径测试、限流压测 | Ragent 关键路径测试 + javaguide 评测 | **[已补]** |
| `阶段4-部署基础设施` | Spring Boot 打包/JVM 调优、容器/K8s、Java×Python 协作、模型网关、Kafka/RocketMQ | Ragent/Nexus 部署 + javaguide 网关 | **[已补]** |
| `阶段5-运维观测` | Micrometer/Actuator/SkyWalking/LangFuse、链路质量、影子路由观测 | Ragent LangFuse + Nexus 影子路由 | **[已补]** |
| `阶段6-治理合规` | Sa-Token/Spring Security、Prompt 注入防护、审计、数据隔离、限流治理 | Ragent/Nexus 安全 + javaguide 安全实战 | **[已补]** |

## 两个企业级 Java 范例（对齐规划"双栈并行"）

- **Ragent**（nageoffer）：7 Maven 模块、v1 Workflow + v2 Agentic 双链路、11 种设计模式实战。
- **Nexus Agent**（javaup）：三层执行器、双通道检索、Neo4j 结构图谱、影子路由、Skills 声明式。
- 两者共同印证：**企业级 Java AI = 分层架构 + 检索工程 + 上下文/记忆治理 + 模型容错 + 全链路追踪 + 安全护栏**。

> 详细对照见 `阶段2-开发工程实现/Java企业级Agent工程实践.md`。

## 平台 / 低代码交付视角（补强，横跨阶段 1/2/4/6）

- **Java 平台级 AI 解决方案：RuoYi AI 与 JEECG AI**（`Java平台级AI解决方案-RuoYiAI与JEECG.md`）：补齐"平台视角"——两个代表性 Java 开源 AI 平台（RuoYi AI 企业级助手、JEECG AI 低代码+AIGC），含双平台总览/功能矩阵/技术栈/能力对照、"平台内部如何呼应理论层与已建 Java 栈"映射表、"平台 vs 自研"选型决策框架。
- **Java 平台源码级模块拆解 · 对照 Ragent / Nexus**（`Java平台源码级模块拆解对照Ragent与Nexus.md`）：把"平台视角"从功能层推进到**源码模块层**——实拉取 RuoYi AI（`ruoyi-chat` 的 `agent/mcp/service/observability/argtrace/websocket/factory/mapper` 等包）与 JEECG（`jeecg-boot-module-airag` 的 `llm/`+`app/`）真实目录树，逐包映射回 Ragent/Nexus 模块，并点明"平台产品化了什么、自研仍强在哪"。
- 价值：与 `阶段2` 自研范式形成"产品化对照"，让平台"按钮背后的工程"一目了然；并给出 MVP 用平台、核心能力自研的**混合落地策略**。
