# 阶段1 · Python AI 工程化需求与架构设计

> 对齐理论层 [`1.1-Agent本质与核心范式`](../../01-通用理论原理层/1.1-Agent本质与核心范式.md)、[`1.5-多Agent协作与编排`](../../01-通用理论原理层/1.5-多Agent协作与编排.md)；Java 栈对照 `../../03-Java生态生产实践栈/阶段1-需求与架构设计`。
> 素材：hello-agents 第1-7章（基础/范式/低代码/框架/自研）、ai-agent-book 第1章（入门）。

## 1. 需求分析：先定"是不是 Agent 问题"

- **AI Native vs 流程驱动**：hello-agents 明确分两派——Dify/Coze/n8n 属"软件工程类 Agent"（LLM 作数据处理后端，本质是流程编排）；真正 AI Native Agent 以 LLM 为决策核心。业务若路径固定、确定性高 → 优先工作流；路径开放、需动态决策 → 上 Agent。
- **非功能需求量化**（理论 1.1）：延迟（首 token / 总耗时）、吞吐（并发 QPS）、成本（Token/请求、$/千次）、可靠性（成功率、降级）、安全性。先用指标框定选型边界。
- **原型验证优先**：用 Coze/Dify 低代码平台快速验证需求是否成立，再决定自研投入（hello-agents 第5章）。

## 2. 框架选型对比（Python 生态）

| 框架 | 核心范式 | 适用场景 | 多 Agent | 学习成本 | 生产成熟度 |
|------|---------|---------|---------|---------|-----------|
| **LangChain + LangGraph** | Chain / Graph（状态机） | 复杂可控编排、RAG 流水线 | ✅（Graph/State） | 中 | 高（生态最全） |
| **AutoGen** | 群聊/对话驱动 | 多 Agent 协作、人机协同 | ✅（GroupChat） | 中 | 高 |
| **AgentScope** | 消息驱动、易调试 | 阿里系、对话式多 Agent | ✅ | 低 | 中 |
| **LlamaIndex** | 数据/RAG 优先 | 知识库问答、检索增强 | 弱（偏单 Agent） | 低 | 高（RAG 强） |
| **CrewAI** | 角色扮演协作 | 类"团队"分工任务 | ✅ | 低 | 中 |
| **自研 HelloAgents** | 极简、可解释 | 学习原理、轻量定制 | 可扩展 | 高（需自写） | 低（教学向） |

> 选型建议：RAG 重 → LlamaIndex；复杂状态编排 → LangGraph；多 Agent 群聊 → AutoGen/AgentScope；需要完全可控且懂原理 → 基于 OpenAI 原生 API 自研（hello-agents Ch7）。

## 3. 架构落地实践

### 3.1 单 Agent 服务架构
```
用户请求 → API 层(FastAPI) → Agent 编排层(LangGraph/自研) → LLM + 工具(Tools/MCP) + 记忆(Memory)
                                                  ↓
                                          可观测层(LangSmith/Trace)
```
- 用 **Pydantic** 定义 Agent I/O 契约，用 **依赖注入** 管理 LLM/工具/记忆实例。
- 上下文窗口有限 → 早期引入 Context Assembler（理论 1.2）。

### 3.2 多 Agent 编排架构（对应理论 1.5）
- **主管-下属（Supervisor-Subagent）**：LangGraph `START → supervisor → {worker} → supervisor` 状态图。
- **群聊（GroupChat）**：AutoGen `GroupChatManager` 轮转发言。
- **流水线（Pipeline）**：固定顺序串联，每个 Agent 产出作为下个输入。
- 选型依据：任务耦合度低、可并行 → 群聊/流水线；需强一致协调 → 主管-下属。

### 3.3 RAG + Agent 混合架构
- Agent 内置 `retrieve` 工具，按需检索而非每次全量注入（理论 1.3）。
- 图谱增强用 Neo4j + 向量库双通道（参考 Java 栈 Nexus 双通道检索思路，Python 侧用 `langchain-neo4j`）。

## 4. 原型快速验证（低代码平台）

| 平台 | 类型 | 适用 | 与代码栈衔接 |
|------|------|------|------------|
| Coze | 低代码 Agent | 对话Bot、快速 Demo | 导出工作流后可代码化重构 |
| Dify | 低代码 + 工作流 | RAG 应用、Agent 编排 | 提供 API，可嵌入自有服务 |
| n8n | 自动化工作流 | 流程驱动集成 | Webhook + 节点编排 |

> 原则：低代码验证需求 → 代码栈承载生产（权限、可观测、成本可控）。不要长期把核心逻辑锁死在低代码平台。

## 5. Python 项目骨架（工程化起点）

```
project/
├── pyproject.toml        # uv 管理，[project.optional-dependencies] 按章分 extras
├── src/agent/
│   ├── core/             # Agent 编排（graph / loop）
│   ├── tools/            # 工具与 MCP client
│   ├── memory/           # 短期/长期记忆
│   ├── rag/              # 检索管线
│   └── api/              # FastAPI 入口
├── tests/                # pytest 用例
└── deployment/           # Dockerfile / k8s yaml
```
- 包管理用 **uv**（`uv sync --extra ch2` 装对应章依赖，ai-agent-book 模式）；Python 3.11–3.13 兼容。
- 配置用 `.env`（API Key）+ Pydantic Settings，避免硬编码密钥。

## 6. 与 Java 栈对照（双栈对齐）

| 维度 | Python（本栈） | Java（03 栈） |
|------|---------------|--------------|
| 框架 | LangGraph/AutoGen/LlamaIndex | Spring AI / LangChain4j / AgentScope-Java |
| 服务 | FastAPI + Uvicorn | Spring Boot + WebFlux(SSE) |
| 记忆 | Chroma/Redis/Neo4j | Redis/MySQL/向量库 |
| 优势 | 框架多、迭代快、算法侧友好 | 权限/事务/中间件融合、企业集成成熟 |
| 适用 | AI 原生应用、算法侧落地 | 企业级业务系统集成 |

> 下一步：进入 [`阶段2-开发工程实现`](../阶段2-开发工程实现/Python企业级Agent工程实践.md)，落地上下文/记忆/RAG/工具/MCP/Coding Agent 的 Python 实现。
