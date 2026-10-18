# 02 · Python 生态生产实践栈

> 定位：**AI Native Agent 的 Python 原生工程实践路径**。与 `03-Java生态生产实践栈`（企业级集成、业务侧落地）形成规划要求的「同一知识点双栈实现对照」。
> 理论底座统一复用 `01-通用理论原理层`；本栈只沉淀 **Python 视角的框架选型、代码实现、工程规范与部署运维**。

## 一、核心素材（规划"阶段二"主来源）

| 来源 | 定位 | 落位阶段 | 关键产出 |
|------|------|---------|---------|
| [datawhalechina/hello-agents](https://github.com/datawhalechina/hello-agents) | 系统性智能体教程，从原理到自研框架 HelloAgents | 全阶段 | 经典范式手撕、框架应用、记忆/RAG/上下文/协议/评估、综合案例 |
| [bojieli/ai-agent-book](https://github.com/bojieli/ai-agent-book) | 生产级 Agent 工程化指南（核心公式 `Agent = LLM + 上下文 + 工具`），109 实验 | 全阶段 | 上下文工程、MCP、Coding Agent、评估、后训练、持续进化、多 Agent |

## 二、Python 技术栈地图

- **LLM 接入**：OpenAI / DeepSeek / 智谱 GLM / Kimi / Claude 原生 SDK；`openai` 兼容客户端；本地 `ollama`
- **Agent 框架**：LangChain + LangGraph（图式编排）、AutoGen（群聊多 Agent）、AgentScope（阿里）、LlamaIndex（RAG 优先）、CrewAI（角色扮演）、自研 HelloAgents
- **协议与工具**：MCP Python SDK（`mcp`）、A2A、ANP；Function Calling / Tool Calling
- **记忆与检索**：Chroma / FAISS / Milvus / Qdrant（向量库）、Neo4j（图谱）、Redis（短期记忆）
- **评估**：RAGAS、DeepEval、LLM-as-Judge、GAIA / OSWorld / SWE-bench 基准
- **服务与部署**：FastAPI / Uvicorn / Gunicorn、vLLM / TensorRT-LLM（推理）、Docker、K8s（GPU 调度）
- **可观测**：LangSmith / LangFuse（链路追踪）、Prometheus + Grafana、OpenTelemetry
- **工程化**：`uv`（包管理）、`pytest`、结构化日志、Pydantic（I/O 契约）

## 三、阶段 ↔ 理论层 ↔ Java 栈 对齐映射

| 阶段 | 本栈文件 | 复用理论层 | Java 栈对照 |
|------|---------|-----------|------------|
| 阶段1 需求与架构设计 | `阶段1-需求与架构设计/Python-AI工程化需求与架构设计.md` | 1.1 Agent本质、1.5 多Agent | `03-Java生态生产实践栈/阶段1` |
| 阶段2 开发工程实现 | `阶段2-开发工程实现/Python企业级Agent工程实践.md`（ai-agent-book 生产范式 + 双栈对照）<br>`阶段2-开发工程实现/Python自研框架与实战案例.md`（hello-agents 从零框架 + 可运行代码 + 3 案例）<br>`阶段2-开发工程实现/code-examples/`（hello-agents `code/` 逐字落盘原码，11 文件/1860 行，含 README 索引） | 1.2 上下文记忆、1.3 RAG、1.4 工具、1.5 | `03-Java生态生产实践栈/阶段2/Java企业级Agent工程实践.md` |
| 阶段3 测试质量保障 | `阶段3-测试质量保障/Python-AI测试与质量保障.md` | 1.6 评估体系 | `03-Java生态生产实践栈/阶段3` |
| 阶段4 部署基础设施 | `阶段4-部署基础设施/Python-AI部署与基础设施.md` | （横向支撑·基础设施） | `03-Java生态生产实践栈/阶段4` |
| 阶段5 运维观测 | `阶段5-运维观测/Python-AI运维观测.md` | 1.7 治理、横向·可观测 | `03-Java生态生产实践栈/阶段5` |
| 阶段6 治理合规 | `阶段6-治理合规/Python-AI治理与合规.md` | 1.7 治理与安全 | `03-Java生态生产实践栈/阶段6` |

## 四、双栈对照要点（规划"双栈对齐"质量要求）

- **同一能力，两种实现**：上下文压缩、分层记忆、混合检索、MCP 接入、流式响应、评估管线、容器化——均需在 Python / Java 两栈分别沉淀，原理一致、实践差异化。
- **选型差异**：Python 重 AI 原生（框架多、迭代快、适合算法侧）；Java 重企业集成（权限/事务/中间件融合、适合业务侧）。
- **溯源**：所有内容标注来源仓库章节（如 `hello-agents Ch8`、`ai-agent-book Ch4`）。

## 五、内容分级
- 理论原理（复用 01 层）
- 生产最佳实践（本栈主沉淀，含可运行范式 + 真实代码）
- 待验证方案（新框架/实验性，标注适用场景与版本）

## 六、待补 / 深化
- hello-agents Ch13-15 案例 `code/` 真实落盘（旅行助手 / DeepResearch / 赛博小镇；Ch7-9 的 code 已落盘于 `code-examples/`）
- hello-agents Ch10 通信协议（MCP/A2A/ANP）、Ch11 Agentic-RL、Ch12 评估 的 Python 代码范式（对齐理论层 1.4/1.6）
- `Extra-Chapter` 社区精选：AgentSkills 解读、GUI/Web Agent 实战、Agent 自进化、踩坑经验
- ai-agent-book 109 实验按章映射进各阶段（Ch2 上下文、Ch3 记忆/RAG、Ch4 工具/MCP、Ch5 Coding Agent、Ch7 评估、Ch8 后训练、Ch9 持续进化、Ch10 多 Agent）
- 补充 LangGraph / AutoGen / LlamaIndex 最新版本的代码范式与踩坑

## 七、应用模式索引（awesome-llm-apps 参照）

> 来源：`github.com/Shubhamsaboo/awesome-llm-apps`（100+ Apache-2.0 开箱即用 app / agent / skills）。补足"真实应用模式索引"，与 阶段2 的逐章代码范式形成"范式 + 模式"互补（G3）。

### 7.1 应用模式分类（taxonomy）

| 模式 | 特征 | 代表示例 |
|------|------|----------|
| **Starter Agents** | 单文件、单 API key 即跑 | travel / data-analysis / medical-imaging / multimodal |
| **Advanced Agents** | 生产级：tools + memory + 多步推理 | deep-research / fraud-investigation / system-architect |
| **Multi-Agent Teams** | 多角色协作 | finance / legal / recruitment / real-estate / travel-planner |
| **Always-on Agents** | 定时/事件驱动后台 agent | HN briefing / release radar |
| **Voice Agents** | 实时语音输入输出 | audio tour / customer support / insurance claim |
| **Generative UI** | 渲染交互 UI（表单/图表/看板） | dashboard canvas / MCP app builder |
| **MCP Agents** | 经 Model Context Protocol 接工具 | browser / github / notion / travel-planner MCP |
| **RAG 应用** | 朴素/代理式/纠正式(CRAG)/混合检索/知识图谱/视觉/本地 RAG（22+ 变体） | agentic-rag / corrective-rag / hybrid-search / knowledge-graph-rag |
| **Memory 应用** | 跨会话记忆 | arxiv-agent-memory / travel-agent-memory |
| **Chat-with-X** | 任意数据源对话 | GitHub / Gmail / PDF / YouTube |
| **Optimization Tools** | 降 token/成本 | prompt-cache / compression |
| **Agent Skills** | 给编码 agent 装能力（一键安装） | project-graveyard / dependency-doctor / self-improving |

### 7.2 对知识库的价值
- 把"能做什么 app"做成**模式清单**，反向指导 阶段2 范式选择（先定模式，再选 ReAct / Workflow / Multi-Agent）。
- 与 `../04-横向支撑域/README.md` 的"应用模式库"互为表里（此处偏 Python 生态，04 偏通用库）。
