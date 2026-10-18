# 复杂生产环境 AI 工程化落地全链路知识库

> 配套规划：[构建规划（完整版）](复杂生产环境%20AI%20工程化落地全链路知识库构建规划.md)
> 构建原则：**核心理论归一化沉淀**、**生产实践分 Python / Java 双栈并行**。
> 定位：面向企业级生产落地的实战型工程知识库，覆盖 RAG / Agent / AIGC 从需求设计到规模化运维的全生命周期。
> 本文件是**全库总导航（站点地图）**：从本页可抵达库内每一个 md/code 文件。

---

## 一、架构总览与当前状态（四层 + 基础支撑底座）

> 结构：**1 理论底座（01）+ 2 生产实践栈（02 Python / 03 Java）+ 1 基础支撑底座（05 计算机基础）+ 4 横向支撑域（04）**。05 是基础地基，支撑 阶段4 部署 / 阶段5 运维 与双栈工程实现。

| 层 | 定位（对齐规划"三、整体架构"） | 核心来源 | 状态 |
|----|---------------------------|---------|------|
| `01-通用理论原理层/` | 归一化理论底座：Agent 本质/范式、上下文与记忆、RAG、工具协议、多 Agent 编排、评估、治理 | xiaolinnote.ai / xiaolinnote.agent / javaguide.ai | **[已完成]** 7 模块封口 |
| `02-Python生态生产实践栈/` | AI 原生应用、算法侧落地（K8s + GPU 推理集群） | hello-agents / ai-agent-book | **[已建]** 全六阶段骨架 + 阶段2 三件套 |
| `03-Java生态生产实践栈/` | 企业级集成、业务侧落地（Java 微服务） | javaguide / nageoffer.ragent / javaup.nexus / RuoYi AI / JEECG AI | **[已建]** 全六阶段 + 阶段2 企业范例 + 平台视角补强 |
| `04-横向支撑域/` | 平台建设 / 组织流程 / 成本 ROI / 安全合规（全链路通用） | 全部资源横向整合 | **[已建]** 4 域映射 |
| `05-基础支撑域/` | 计算机基础底座：网络 / 系统(OS) / MySQL / Redis，支撑部署与运维 | xiaolincoding.com 图解系列 | **[已建]** 4 主题文档 |
| `素材收集/` | 原始采集素材（保留溯源，勿删） | 6 站点入口 + 核心长文 + 平台/基础补采 + 参考仓库规划 | **[已完成]** 9 份（00–09） |

**双栈对照**：理论层为 Python / Java 双栈共享底座；同一知识点在两个实践栈保持"原理一致、实践差异化"（以 `阶段2-开发工程实现` 的对照表为核心载体）。

---

## 二、全库文件导航

### 📌 根级

- [README.md](README.md) — 本导航（站点地图）
- [构建规划（完整版）](复杂生产环境%20AI%20工程化落地全链路知识库构建规划.md) — 知识库的原始规划与架构定义
- [战略洞察：Java 工程能力移植与企业级 AI 应用形态](战略洞察-Java工程能力移植与企业级AI应用形态.md) — 北极星论述：Java 工程能力移植性（70~80% 可复用）+ 企业级 AI 形态（Web 为核心+多端），附全库落位映射

### 📚 01-通用理论原理层（归一化底座，7 模块已封口）

| 文件 | 核心内容 |
|------|---------|
| [README](01-通用理论原理层/README.md) | 理论层总览、模块覆盖、来源映射、待补清单 |
| [1.1 Agent 本质与核心范式](01-通用理论原理层/1.1-Agent本质与核心范式.md) | Agent 定义、四大组件、ReAct/Plan-Execute/Reflection、Workflow vs Agent、提示词工程起点 |
| [1.2 上下文与记忆工程](01-通用理论原理层/1.2-上下文与记忆工程.md) | Prompt→Context→Memory→Harness 链路、上下文压缩、记忆分层与生命周期、Harness 六层 |
| [1.3 RAG 技术体系](01-通用理论原理层/1.3-RAG技术体系.md) | RAG 流程、切分/Embedding/向量库/重排、GraphRAG/LightRAG、vs 微调 |
| [1.4 工具体系与通信协议](01-通用理论原理层/1.4-工具体系与通信协议.md) | Function Call / MCP / Skills / A2A 分层与关系、MCP 深解 |
| [1.5 多 Agent 协作与编排](01-通用理论原理层/1.5-多Agent协作与编排.md) | Graph Engineering、Workflow/Graph/Loop 对比、9 种编排模式、Loop Engineering |
| [1.6 评估体系](01-通用理论原理层/1.6-评估体系.md) | 评测 vs 监控、四类评测、Golden Set、LLM-as-Judge、Java 后端落地 |
| [1.7 治理与安全原则](01-通用理论原理层/1.7-治理与安全原则.md) | Prompt 注入、越权/泄露防护、数据安全、大模型网关 |

### 🐍 02-Python 生态生产实践栈（AI 原生，全六阶段）

| 文件 | 核心内容 |
|------|---------|
| [README](02-Python生态生产实践栈/README.md) | Python 栈总览、技术栈地图、阶段↔理论层↔Java 栈映射 |
| [阶段1 需求与架构设计](02-Python生态生产实践栈/阶段1-需求与架构设计/Python-AI工程化需求与架构设计.md) | AI Native vs 流程驱动、框架选型对比、架构落地、低代码原型 |
| [阶段2·生产范式](02-Python生态生产实践栈/阶段2-开发工程实现/Python企业级Agent工程实践.md) | ai-agent-book 生产范式：上下文/KV Cache、RAG/RRF、工具 ACI/MCP、Coding Agent + **双栈对照表** |
| [阶段2·从零框架+案例](02-Python生态生产实践栈/阶段2-开发工程实现/Python自研框架与实战案例.md) | hello-agents 自研框架（Ch7）、记忆检索（Ch8）、上下文（Ch9）、3 个端到端案例（Ch13-15） |
| [阶段2·真实代码](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/README.md) | hello-agents `code/` 逐字落盘（67 文件 / 10400+ 行：Ch7-9 框架基础 + Ch10-12 进阶范式 + Ch13-15 端到端案例）索引 |
| ├─ [chapter7 自研框架](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/chapter7-自研框架/) | my_llm / my_simple_agent / my_react_agent / my_calculator_tool / my_main / my_advanced_search |
| ├─ [chapter8 记忆与检索](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/chapter8-记忆与检索/) | MemoryTool / RAGTool_MarkItDown / QA_Assistant |
| ├─ [chapter9 上下文工程](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/chapter9-上下文工程/) | context_builder_basic / context_builder_with_agent |
| ├─ [chapter10 通信协议](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/chapter10-通信协议/) | MCP(connect/transport/server) / A2A(server/client/tool) / ANP(init/distribute/balance) |
| ├─ [chapter11 Agentic-RL](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/chapter11-agentic-rl/) | dataset / reward / grpo_training / pipeline / eval / distributed |
| ├─ [chapter12 评估](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/chapter12-评估/) | bfcl / gaia / llm_judge / win_rate 数据生成 |
| ├─ [chapter13 旅行助手](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/chapter13-旅行助手/) | trip_planner_agent / amap_service(MCP) / api_main / schemas / config |
| ├─ [chapter14 DeepResearch](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/chapter14-deepresearch/) | agent(编排) / services_planner·search·summarizer·reporter / prompts |
| └─ [chapter15 赛博小镇](02-Python生态生产实践栈/阶段2-开发工程实现/code-examples/chapter15-赛博小镇/) | agents(多Agent) / relationship_manager(好感度) / state_manager / batch_generator |
| [阶段3 测试质量保障](02-Python生态生产实践栈/阶段3-测试质量保障/Python-AI测试与质量保障.md) | RAGAS/DeepEval、GAIA/OSWorld/SWE-bench、CI 回归 |
| [阶段4 部署基础设施](02-Python生态生产实践栈/阶段4-部署基础设施/Python-AI部署与基础设施.md) | FastAPI/Uvicorn、多阶段 CUDA 镜像、K8s GPU、vLLM/TensorRT-LLM |
| [阶段5 运维观测](02-Python生态生产实践栈/阶段5-运维观测/Python-AI运维观测.md) | 可观测三支柱、告警分级、Bad Case 闭环、持续进化 |
| [阶段6 治理合规](02-Python生态生产实践栈/阶段6-治理合规/Python-AI治理与合规.md) | 安全护栏、数据隔离、合规审计、双栈归一化结论 |

### ☕ 03-Java 生态生产实践栈（企业集成，已建 + 平台视角）

| 文件 | 核心内容 |
|------|---------|
| [README](03-Java生态生产实践栈/README.md) | Java 栈总览、双极对照（自研 Ragent/Nexus + 平台 RuoYi/JEECG）、阶段进度 |
| [平台级 AI 解决方案（RuoYi AI / JEECG AI）](03-Java生态生产实践栈/Java平台级AI解决方案-RuoYiAI与JEECG.md) | **平台视角补强**：两平台总览/功能矩阵/技术栈/能力对照、理论层映射、"平台 vs 自研"选型决策框架 |
| [平台源码级模块拆解 · 对照 Ragent/Nexus](03-Java生态生产实践栈/Java平台源码级模块拆解对照Ragent与Nexus.md) | **平台视角·源码层**：实拉取 RuoYi `ruoyi-chat` 与 JEECG `airag` 真实包结构，逐包映射 Ragent/Nexus 模块，点明平台产品化与自研护城河 |
| [Spring AI 生产级应用范式](03-Java生态生产实践栈/SpringAI生产级应用范式-llm-apps-java-spring-ai.md) | **组件级补强（G4）**：以 ThomasVitale/llm-apps-java-spring-ai 为参照，沉淀 Spring AI 多模型/结构化输出/工具调用/Memory/Guardrails/RAG/Observability/MCP 的生产代码范式 |
| [阶段1 需求与架构设计](03-Java生态生产实践栈/阶段1-需求与架构设计/Java-AI工程化需求与架构设计.md) | AI vs 流程驱动、框架选型（Spring AI/LangChain4j/AgentScope）、模块化分层、原型验证 |
| [阶段2·企业级范例](03-Java生态生产实践栈/阶段2-开发工程实现/Java企业级Agent工程实践.md) | 融合 Ragent + Nexus + JavaGuide：分层架构、双链路/三层执行器、检索融合、上下文治理、模型容错、11 种设计模式 |
| [阶段3 测试质量保障](03-Java生态生产实践栈/阶段3-测试质量保障/Java-AI测试与质量保障.md) | 非确定性断言、Golden Set、RAGAS/DeepEval、关键路径测试、限流压测 |
| [阶段4 部署基础设施](03-Java生态生产实践栈/阶段4-部署基础设施/Java-AI部署与基础设施.md) | Spring Boot/JVM、容器/K8s、Java×Python 协作、模型网关、Kafka/RocketMQ |
| [阶段5 运维观测](03-Java生态生产实践栈/阶段5-运维观测/Java-AI运维观测.md) | Micrometer/Actuator/SkyWalking/LangFuse、链路质量、影子路由观测 |
| [阶段6 治理合规](03-Java生态生产实践栈/阶段6-治理合规/Java-AI治理与合规风控.md) | Sa-Token/Spring Security、Prompt 注入防护、审计、数据隔离、限流治理 |

### 🧩 04-横向支撑域（全链路通用）

| 文件 | 核心内容 |
|------|---------|
| [README](04-横向支撑域/README.md) | 横向域总览、4 大域与全链路阶段映射 |
| [平台建设](04-横向支撑域/平台建设.md) | AI 中间件/网关/评测平台/可观测基建 |
| [组织与流程](04-横向支撑域/组织流程.md) | 团队角色、开发流程、人机协作、上线规范 |
| [成本与 ROI](04-横向支撑域/成本ROI.md) | Token/推理成本、ROI 度量、降本策略 |
| [安全合规治理](04-横向支撑域/安全合规治理.md) | 安全红线、合规要求、审计追溯 |

### 🧱 05-基础支撑域（计算机基础底座，支撑部署/运维/双栈）

| 文件 | 核心内容 |
|------|---------|
| [README](05-基础支撑域/README.md) | 基础支撑域总览、4 主题↔AI 工程化映射、与阶段4/5/双栈交叉索引 |
| [01-Redis 基础支撑](05-基础支撑域/01-Redis基础支撑.md) | ZSet→Agent 公平排队、Hash→记忆、分布式锁→并发去重、缓存防击穿→向量检索、高可用、**§五 原理精读(SDS/分布式锁 setnx)** |
| [02-MySQL 基础支撑](05-基础支撑域/02-MySQL基础支撑.md) | 索引/B+树→对话与向量元数据表、事务/MVCC→一致写入、锁→并发、日志→CDC 触发向量更新、**§五 原理精读(B+树/MVCC)** |
| [03-网络基础支撑](05-基础支撑域/03-网络基础支撑.md) | SSE/WebSocket→流式对话、HTTP/2 多路复用→网关降延迟、TCP 队列→高并发、gRPC→K8s 内部、**§五 原理精读(HTTP/2/WebSocket)** |
| [04-系统(OS)基础支撑](05-基础支撑域/04-系统基础支撑.md) | 线程模型→JVM Agent 执行器、epoll/Reactor→网关、零拷贝、一致性哈希→分片、伪共享、**§五 原理精读(epoll/Reactor)** |

### 🗂️ 素材收集（原始溯源，勿删）

| 文件 | 来源站点 | 内容 |
|------|---------|------|
| [00-索引与处理计划](素材收集/00-索引与处理计划.md) | — | 采集索引、处理顺序、八阶段进度记录 |
| [01-xiaolinnote.ai](素材收集/01-xiaolinnote-ai-大模型面试题.md) | xiaolinnote.com/ai/ | 六大专题题库（Agent30/RAG23/工具22/LLM23/LangChain/AI编程）+ 理论体系 |
| [02-xiaolinnote.agent](素材收集/02-xiaolinnote-agent-图解Agent.md) | xiaolinnote.com/agent/ | Agent 是什么、Harness、Graph、Loop、RAG、GraphRAG/LightRAG 长文 |
| [03-nageoffer.ragent](素材收集/03-nageoffer-ragent.md) | nageoffer.com/ragent/ | 企业级 Java Agentic RAG：7 模块、双链路、11 模式 |
| [04-javaup.nexus](素材收集/04-javaup-nexus-agent.md) | javaup.chat/super-agent/ | 企业级 Java Nexus Agent：三层执行器、双通道检索、Neo4j |
| [05-javaguide.ai](素材收集/05-javaguide-ai.md) | javaguide.cn/ai/ | Java 后端 AI 体系：RAG/MCP/Skills/记忆/上下文/Harness/评测/安全/网关 子页 |
| [06-xiaolincoding](素材收集/06-xiaolincoding.md) | xiaolincoding.com | 图解网络/系统/MySQL/Redis 基础支撑域索引 |
| [07-ruoyi-jeecg-platform](素材收集/07-ruoyi-jeecg-platform.md) | doc.ruoyiai.chat / jeecg.com / 公网解析 | Java 平台视角素材：RuoYi AI + JEECG AI 定位/功能/技术栈/对照（溯源） |
| [08-xiaolincoding-base](素材收集/08-xiaolincoding-base.md) | xiaolincoding.com 四大图解系列 | 基础支撑域素材：网络/系统/MySQL/Redis 栏目地图 + AI 工程化映射点（溯源） |
| [09-参考仓库待采集索引](素材收集/09-参考仓库待采集索引.md) | GitHub: awesome-llm-apps / AutoGPT / Dify / llm-apps-java-spring-ai | 四仓库落位规划 + 六大缺失项（G1–G6）分析；**已于 2026-10-18 采集并落地 G1–G6** |

---

## 三、全链路阶段划分（规划"四、分模块"）

每个实践栈均按 6 阶段组织子目录：
`阶段1-需求与架构设计` → `阶段2-开发工程实现` → `阶段3-测试质量保障` → `阶段4-部署基础设施` → `阶段5-运维观测` → `阶段6-治理合规`

通用理论层按知识域组织：`1.1 Agent本质` / `1.2 上下文与记忆` / `1.3 RAG` / `1.4 工具协议` / `1.5 多Agent编排` / `1.6 评估` / `1.7 治理`。

---

## 四、使用与维护约定

1. **内容分级**：每个知识点在文中标注 `理论原理` / `生产最佳实践` / `待验证方案` 三级，并注明适用版本/场景。
2. **资源溯源**：所有沉淀内容在文末 `> 来源` 标注原始素材文件，便于回溯（见上方"素材收集"）。
3. **双栈对齐**：同一知识模块在 Python / Java 栈保持原理一致、实践差异化（见阶段2 对照表）。
4. **持续迭代**：`素材收集/` 为原始素材库，正式文件为归一化成果；补采后回灌对应层级。

---

## 五、当前进度与下一步

**已完成（十余阶段）**：理论层封口 → 双栈骨架 → 理论层补采回灌 → javaguide 子页封口 → Python 栈填充 → Python 阶段2 深化 → hello-agents 逐章拆解 → code 真实落盘（Ch7-9）→ 收口总导航 → Java 栈补全（全六阶段）→ Java 平台视角补采（RuoYi AI / JEECG AI）→ 基础支撑域实采（→ 05-基础支撑域）→ hello-agents Ch13-15 端到端案例原码落盘（40 文件）→ 两平台源码级模块拆解对照 Ragent/Nexus → **Ch10/11/12 Python 代码范式落盘（通信协议 MCP/A2A/ANP、Agentic-RL、评估；27 文件，code-examples 增至 67 文件 / 10400+ 行）** → **③ 05 基础支撑域原理精读回灌（Redis/MySQL/网络/OS 四篇各新增「五、原理精读」段）** → **④ 跨文件质量打磨（修复 11 处占位路径、命名一致性核查，完成）** → **战略洞察文档落盘 + 四参考仓库采集规划（G1–G6）** → **四仓库采集落地 G1–G6**：① G1 自主智能体范式（1.1 第五范式 + AutoGPT 平台）、② G2 LLMOps 平台对照（Dify + AutoGPT，平台建设.md）、③ G3 案例/应用模式索引（02-Python README 第七节 + 04 README 应用模式库）、④ G4 Spring AI 生产范式（新文档）、⑤ G5 Agent 六形态部署（03 阶段4）、⑥ G6 成本与配额工程（04 成本ROI）。

**建议下一步（按用户给定顺序）**：
- ② ✅ 两平台源码级模块拆解对照 Ragent/Nexus（已完成）；
- ① ✅ Ch10/11/12 的 Python 代码范式落盘（已完成）；
- ③ ✅ 05 基础支撑域原理精读回灌（已完成）；
- ④ ✅ 质量打磨：跨文件术语统一、去重检查、命名一致性核查、占位路径修复（已完成）；
- ⑤ ✅ 战略文章落盘 + 四仓库规划与采集（已完成）：落盘 `战略洞察-Java工程能力移植与企业级AI应用形态.md` 与 `素材收集/09-参考仓库待采集索引.md`；四仓库（awesome-llm-apps / AutoGPT / Dify / llm-apps-java-spring-ai）已直连采集并落地 G1–G6（见上方进度线）。
