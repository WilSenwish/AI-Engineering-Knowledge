# Java 平台源码级模块拆解 · 对照 Ragent / Nexus

> 定位：**把"平台视角"从功能层推进到源码模块层**。
> 前篇 `Java平台级AI解决方案-RuoYiAI与JEECG.md` 已在**功能/特性**维度把两平台与理论层、自研范式做了对照；本文进一步**拆解两个开源平台的真实源码包结构**，把每个模块/包映射回本栈已沉淀的 **Ragent / Nexus** 自研模块（见 `阶段2-开发工程实现/Java企业级Agent工程实践.md`），形成"平台按钮背后的工程 ↔ 自研模块 ↔ 理论层"的逐包对照。
> 价值：看到平台 Maven 包名就能对应到 Ragent/Nexus 的哪一块能力，知道"平台省了什么、自研强在哪"。

> **源码实证**：以下目录树通过 GitHub API（`api.github.com/repos/.../contents/`）于 2026-10-18 实拉取，为当前 `main` / `springboot3_sas` 分支真实结构；版本演进以官方为准。

---

## 0. 为什么要做"源码级"拆解

- 功能层对照回答"平台能做什么"；**源码层对照回答"平台的每一块能力，对应自研架构的哪一环"**。
- Ragent / Nexus 是"从零手搓"的参照系（分层 / 双链路 / 三层执行器 / 模型容错 / 全链路观测）；两个平台是"开箱即用"的成品。**二者在同一套理论层概念下收敛**——拆开源码即可印证。
- 本文所有映射都指向 `Java企业级Agent工程实践.md` 的分层表（入口/编排、AI 供应商隔离、业务编排、系统支撑、工具）与 Ragent/Nexus 的具体实现（Redis ZSET 公平排队、RocketMQ 事务、LangFuse、Sa-Token、三层执行器、影子路由）。

---

## 1. RuoYi AI 源码模块拆解（`ageerle/ruoyi-ai`，MIT）

### 1.1 Maven 多模块总览
```
ruoyi-ai/
├── pom.xml                 # 父 POM，统一依赖（Spring Boot 3.4/3.5 + Spring AI + LangChain4j）
├── ruoyi-admin/            # 管理后台（启动入口，对应 入口/编排 bootstrap）
├── ruoyi-common/           # 通用工具、基础组件
├── ruoyi-extend/           # 扩展模块
│   ├── ruoyi-monitor-admin/    # 监控 Admin（对应 阶段5 观测）
│   └── ruoyi-snailjob-server/  # SnailJob 任务调度/重试（对应 Ragent RocketMQ 事务/幂等/重试护栏）
└── ruoyi-modules/          # 业务模块
    ├── ruoyi-chat/         # ★ 核心 AI 对话/智能体模块（见 1.2）
    ├── ruoyi-aiflow/       # AI 流程（包 org.ruoyi.workflow，对应 Ragent v1 Workflow）
    ├── ruoyi-workflow/     # 工作流引擎
    ├── ruoyi-system/       # 系统管理（用户/角色/权限，对应 阶段6 RBAC）
    └── ruoyi-generator/    # 代码生成器
```

### 1.2 核心模块 `ruoyi-chat` 的 Java 包（`org.ruoyi`）
```
ruoyi-modules/ruoyi-chat/src/main/java/org/ruoyi/
├── agent/          # 智能体编排（Supervisor / ReAct）—— 对应 Ragent agent 模块 / Nexus 智能体引擎
├── mcp/            # MCP 工具即插即用 —— 对应 Ragent mcp-server 模块
├── service/        # 业务编排服务（RAG 问答、对话编排）—— 对应 Ragent rag/agent 业务编排层
├── controller/     # HTTP / SSE 入口 —— 对应 入口/编排 bootstrap + framework
├── websocket/     # WebSocket 流式输出 —— 对应 阶段2 流式（SSE/WS）
├── factory/        # 工厂创建（意图/分块/回调对象）—— 对应 Ragent 工厂模式
├── observability/  # 可观测性埋点 —— 对应 Ragent LangFuse / 阶段5 观测
├── argtrace/       # 参数/调用链路追踪 —— 对应 全链路 Trace（Run/Node 耗时/IO/异常）
├── domain/         # 领域模型（对话/知识库/模型实体）—— 对应 状态外化载体
├── mapper/         # MyBatis 持久化 —— 对应 长期记忆/状态外化存储
├── config/         # 配置中心
├── constant/       # 常量
├── common/         # 通用
└── enums/          # 枚举
```
> 一眼可见：`agent / mcp / service / controller / observability / factory / mapper` 七个包，与本栈"分层架构表"的入口/编排、业务编排、工具、可观测、设计模式、状态外化**逐一对齐**——平台只是把自研范式"包名化"了。

---

## 2. JEECG AI 源码模块拆解（`jeecgboot/jeecg-boot` 的 `jeecg-boot-module-airag`，Apache-2.0）

> JEECG 把 AI 能力建在成熟低代码底座上，AI 核心集中在独立 Maven 模块 `jeecg-boot-module-airag`，采用"能力层-应用层"双层架构。

```
jeecg-boot-module-airag/
├── llm/                       # 能力层（AI 核心，对应 Ragent "AI 供应商隔离 + 业务编排"）
│   ├── controller/            # 模型管理接口层 —— 对应 入口/编排 bootstrap
│   ├── service/              # 模型交互服务层 —— 对应 Ragent infra-ai + rag/agent
│   ├── handler/              # 核心处理逻辑层
│   │   ├── AIChatHandler     #   模型适配引擎（封装 GPT/通义/文心差异）—— 对应 Ragent 适配器（模型方言）
│   │   └── EmbeddingHandler  #   文档向量化/语义检索 —— 对应 Ragent infra-ai EmbeddingClient
│   └── document/             # 文档处理工具类（PDF/Word/TXT 高保真解析分块）—— 对应 RAG 解析/分块
└── app/                       # 应用层（前端交互，对应 入口/编排）
    └── controller/           # 聊天服务 / 知识库查询接口 —— 对应 用户入口
```
> 交互模式：REST + **SSE**（打字机流式）—— 对应 阶段2 流式。对比 Dify 的独有能力（高保真文档、低代码流程引擎、iframe 嵌入）在 `document/` 与低代码内核中落地。

---

## 3. 模块对照矩阵（平台包 ↔ Ragent ↔ Nexus ↔ 理论/阶段）

| 平台源码位置 | 职责 | Ragent 对应模块 | Nexus 对应模块 | 理论层 / 阶段 |
|------------|------|----------------|----------------|--------------|
| **RuoYi** `ruoyi-chat/agent` | 智能体编排（Supervisor/ReAct） | `agent` | 智能体引擎 | 1.5 多 Agent 编排 |
| **RuoYi** `ruoyi-chat/mcp` | MCP 工具即插即用 | `mcp-server` | MCP/Skills | 1.4 工具/MCP/Skills |
| **RuoYi** `ruoyi-chat/service` | 业务编排（RAG/对话） | `rag` / `agent` | 三层执行器 | 阶段2 业务编排 |
| **RuoYi** `ruoyi-chat/controller` + `websocket` | HTTP/SSE/WS 入口与流式 | `bootstrap` + `framework` | 用户入口 + 前置编排 | 阶段2 入口/流式 |
| **RuoYi** `ruoyi-chat/factory` | 工厂创建（意图/分块/回调） | 工厂模式 | — | 阶段2 设计模式 |
| **RuoYi** `ruoyi-chat/observability` + `argtrace` | 可观测 / 调用追踪 | LangFuse 可观测 | 影子路由观测 | 阶段5 运维观测 |
| **RuoYi** `ruoyi-chat/domain` + `mapper` | 状态外化 / 长期记忆存储 | 持久化摘要 | 滑动窗口/摘要压缩 | 1.2 上下文与记忆 |
| **RuoYi** `ruoyi-aiflow`(`workflow`) | 可视化流程编排 | v1 Workflow | — | 1.5 编排（Workflow/Graph） |
| **RuoYi** `ruoyi-snailjob-server` | 任务重试 / 调度 | RocketMQ 事务/幂等/重试 | 工程化护栏 | 阶段4/5 护栏 |
| **RuoYi** `ruoyi-monitor-admin` | 监控面板 | 管理端趋势/详情 | SkyWalking | 阶段5 观测 |
| **RuoYi** `ruoyi-system` | RBAC / 权限 | Sa-Token 认证 | 安全基础 | 阶段6 治理合规 |
| **JEECG** `llm/handler/AIChatHandler` | 模型适配引擎 | 适配器（模型方言） | 知识底座路由 | 1.3/1.4 供应商隔离 |
| **JEECG** `llm/document` | 文档解析/向量化（高保真） | `infra-ai` Embedding | 检索引擎 | 1.3 RAG 技术体系 |
| **JEECG** `llm/service` | 模型交互 / 编排 | `infra-ai` + `rag` | 三层执行器 | 阶段2 业务编排 |
| **JEECG** `llm/controller` + `app/controller` | 接口层（REST+SSE） | `bootstrap` | 用户入口 | 阶段2 入口/流式 |

> 结论：**平台包结构 ≈ 自研分层表的产品化**。Ragent 用 `agent/rag/mcp/system/bootstrap` 五个 Maven 模块表达的边界，RuoYi 用 `ruoyi-chat` 内的 `agent/mcp/service/controller/observability` 子包同样表达；JEECG 用 `llm/`（能力层）+ `app/`（应用层）双层表达同样的"AI 供应商隔离 vs 业务编排"。

---

## 4. 平台"产品化"了什么，自研仍强在哪

### 4.1 平台已开箱封装（对应自研范式，无需手搓）
- **多智能体**：RuoYi 用 LangChain4j **Supervisor** 原生编排搜索/SQL/图表/闲聊角色——Ragent 的 Supervisor 等价物被框架直接提供。
- **MCP 即插即用 + 工具市场**：RuoYi `mcp/` 包 + 工具市场，对应 Ragent `mcp-server`；平台额外提供 LOCAL/REMOTE/ModelScope 多源与可视化配置。
- **可视化流程编排**：RuoYi `ruoyi-aiflow`(workflow) / JEECG 流程设计器引擎，对应 Ragent v1 Workflow 路径写死范式——但平台用拖拽画布替代手搓 DAG。
- **可观测**：RuoYi `observability/argtrace` 包对应 Ragent LangFuse；JEECG 微服务接 SkyWalking——与 `阶段5` 技术栈重合。
- **安全治理**：RuoYi `ruoyi-system` + Sa-Token/JWT（与 `阶段6` 一致）；JEECG 信创六库（达梦/金仓）。

### 4.2 自研（Ragent/Nexus）仍明显领先的深度工程
| 深度能力 | 自研实现 | 平台现状 | 为何自研更强 |
|---------|---------|---------|------------|
| **模型流量保护** | Redis ZSET 公平排队 + Lua 原子抢占 + 过期信号量 + Pub/Sub 唤醒 | 一般靠网关限流，少有"按会话公平排队" | 突发流量下避免模型被单会话打爆，公平性保障 |
| **分块事务一致性** | RocketMQ 事务消息 + 幂等 + 分布式锁状态机 | 用 SnailJob/调度兜底重试 | 海量文档分块"恰好一次"执行，平台深度不及 |
| **前置编排 + 证据短路** | Nexus 三层执行器（歧义追问→知识问答→开放式 Agent）+ 无证据短路 | 多为单一对话链路 | 架构层杜绝幻觉，降低模型不确定性 |
| **影子路由优化闭环** | Nexus 影子路由静默对比命中率/置信度/候选排名 | 平台少有 A/B 影子对比 | 形成"上线即回归"的持续优化闭环 |
| **设计模式显式化** | Ragent 11 种模式提炼为 Java AI 工程规范 | 平台隐式使用，无显式范式文档 | 自研可把经验沉淀为可复用规范 |
| **检索融合深度** | Ragent 向量+关键词+知识图谱+联网并行 + RRF + Rerank | 平台以向量+关键词 + Rerank 为主 | 多通道并行 + 图谱召回在复杂知识域更准 |

### 4.3 选型启示（呼应前篇决策框架）
- **平台 = 广度 / 速度（0→1）**：Maven 包一拉、可视化一配即可上线；RuoYi 2 核 2G 可跑，JEECG 低代码直接进业务系统。
- **自研 = 深度 / 可控（1→N）**：Nexus 三层执行器、Ragent 公平排队/MQ 事务等是"平台做不到或做不深"的护城河。
- **混合最常见**：平台做 MVP/内部工具/知识库问答快速验证；验证过的核心差异化能力逐步自研迁移到 Ragent/Nexus 式架构。平台负责"广"，自研负责"深"。

---

## 5. 落位与回引

- 本篇是 `Java平台级AI解决方案-RuoYiAI与JEECG.md`（功能层）的**源码层深化**，二者共同构成 Java 栈"平台视角"双文档。
- 交叉阅读：`阶段2-Java企业级Agent工程实践.md`（自研模块基准）、`阶段5-Java-AI运维观测.md`（SkyWalking/LangFuse）、`阶段6-Java-AI治理与合规风控.md`（Sa-Token/RBAC/信创）。
- 与 Python 栈对照：Python 侧"平台"是 Dify/Coze/FastGPT（RuoYi/JEECG 也能接入），可在 `02-...-README.md` 平台视角章节补充同源对照。
- **完成前篇"仍可补强"项**：RuoYi AI 与 JEECG 的源码级模块拆解（对照 Ragent/Nexus 多模块结构）已基本完成；平台侧"阶段3 测试"工程实践仍可作为后续补强（平台多把评测做成"知识库问答效果"配置，而非 CI 回归）。

---

> 来源：`素材收集/07-ruoyi-jeecg-platform.md`（功能层素材）；RuoYi AI 目录树取自 `github.com/ageerle/ruoyi-ai` GitHub API、JEECG AI 取自 `jeecg-boot-module-airag` 公开结构（采集于 2026-10-18）。具体包名/版本以官方仓库实时为准。
