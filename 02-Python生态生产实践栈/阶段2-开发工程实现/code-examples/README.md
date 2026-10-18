# code-examples · hello-agents 实战脚本真实落盘

> **本目录定位**：将 [datawhalechina/hello-agents](https://github.com/datawhalechina/hello-agents) 仓库 `code/` 下的**真实可运行脚本**逐字落盘，作为 `Python自研框架与实战案例.md`（范式提炼）的**配套原码**。
> 范式文档讲"为什么、怎么做（提炼）"，本目录给"原项目真实代码（可跑）"——两者结合，Python 栈阶段2 从范式到代码完整闭环。
>
> **来源声明**：以下文件均逐字取自 hello-agents 仓库 `code/chapter7|8|9|10|11|12|13|14|15/`，仅调整了个别含 `&` 的文件名（如 `11_Q&A_Assistant.py` → `11_QA_Assistant.py`）以适配文件系统。其中 chapter13-15 为**完整端到端应用**（旅行助手 / DeepResearch / 赛博小镇），本目录只落盘其 **backend Agent 核心逻辑**，前端（Vue/Godot）样板不纳入；chapter10-12 为**进阶主题范式脚本**（通信协议 / Agentic-RL / 评估），只取最具代表性的 .py。版权归原作者/Datawhale 所有，本目录仅作知识库学习用途归档。

## 一、运行前提

```bash
pip install "hello-agents[all]==0.2.0"   # 或更新到 0.2.9（issue#320 相关）
# 配置 .env：OPENAI_API_KEY / DASHSCOPE_API_KEY(百炼) / QDRANT / NEO4J 等
cp .env.example .env  # 各章 code 目录均提供 .env.example，按需复制
```

> 注：部分脚本依赖 Qdrant / Neo4j / MarkItDown 等外部服务，需按 `.env.example` 与本机环境准备；纯 LLM/Agent 逻辑的脚本（chapter7、chapter9 基础示例）仅需 API Key 即可运行。

## 二、文件索引（按章 + 对应范式文档章节）

### chapter7-自研框架（对应范式文档「一、自研 Agent 框架范式」）
| 文件 | 行数 | 作用 | 范式文档对应 |
|------|------|------|------------|
| `my_llm.py` | 40 | 继承 `HelloAgentsLLM` 接入自定义 Provider（如 ModelScope） | 1.3 核心抽象·HelloAgentsLLM |
| `my_simple_agent.py` | 250 | `SimpleAgent` 完整实现：历史管理、工具正则解析、对话循环 | 1.3/1.4/1.5 短期记忆与工具调用 |
| `my_react_agent.py` | 99 | ReAct 主循环：Thought/Action/Finish 解析 + 工具执行 | 1.4 ReAct 主循环 |
| `my_calculator_tool.py` | 61 | 自定义 `Tool` 子类（计算器）示例 | 1.3 工具抽象·Tool/Registry |
| `my_advanced_search.py` | 131 | 进阶搜索工具（注册到 Registry、OpenAI schema 转换） | 1.6 扩展点·工具注册 |
| `my_main.py` | 21 | 入口：装配 LLM/Agent/Tool 并运行 | 1.2 分层架构 |

### chapter8-记忆与检索（对应范式文档「二、记忆与检索工程实现」）
| 文件 | 行数 | 作用 | 范式文档对应 |
|------|------|------|------------|
| `01_MemoryTool_Basic_Operations.py` | 198 | 记忆读写基本操作（语义/工作记忆） | 2.1 记忆四分类 |
| `04_RAGTool_MarkItDown_Pipeline.py` | 407 | RAG 全链路：MarkItDown→分块→Embedding→Qdrant | 2.2 RAG Pipeline |
| `11_QA_Assistant.py` | 448 | **完整案例**：PDF 问答助手，集成 Memory+RAG+Gradio | 2.1/2.2/2.4 综合应用 |

### chapter9-上下文工程（对应范式文档「三、上下文工程实现」）
| 文件 | 行数 | 作用 | 范式文档对应 |
|------|------|------|------------|
| `01_context_builder_basic.py` | 103 | ContextBuilder 基础：分区组装 `[Role][Task][Evidence][Context][Output]` | 3.1/3.2 上下文构成与分区 |
| `02_context_builder_with_agent.py` | 102 | 将 ContextBuilder 接入 Agent 推理流程 | 3.2/3.4 选择压缩与记忆协作 |

### chapter13-旅行助手（对应范式文档「四、端到端案例 · 13 旅行助手」）
> 基于 `SimpleAgent` + 高德地图 **MCP** 工具的个性化旅行规划应用（FastAPI 后端）。
| 文件 | 行数 | 作用 |
|------|------|------|
| `trip_planner_agent.py` | 430 | 多智能体旅行规划核心：SimpleAgent 装配高德 MCP 工具，生成多日行程 |
| `amap_service.py` | 269 | 高德地图 MCP 服务封装（POI 搜索 / 天气 / 步行驾车公交路线规划） |
| `schemas.py` | 206 | Pydantic 数据模型（行程/景点/偏好） |
| `api_main.py` | 99 | FastAPI 路由：`POST /api/trip/plan`、`GET /api/map/*` |
| `config.py` | 111 | 配置管理（LLM/MCP/高德 Key） |
| `llm_service.py` | 37 | LLM 服务封装 |
| `run.py` | 16 | 启动入口（`uvicorn app.api.main:app`） |

### chapter14-DeepResearch（对应范式文档「四、端到端案例 · 14 深度研究」）
> 自动化深度研究 Agent：规划 → 多轮搜索 → 摘要 → 报告生成（FastAPI 后端，服务化拆分）。
| 文件 | 行数 | 作用 |
|------|------|------|
| `agent.py` | 551 | 编排主逻辑：串联 planner/search/summarizer/reporter 的 DeepResearch 流程 |
| `services_tool_events.py` | 215 | 工具事件流（搜索/读取进度回调） |
| `services_planner.py` | 160 | 研究计划生成（拆解子问题、制定搜索策略） |
| `services_summarizer.py` | 125 | 网页内容摘要与要点抽取 |
| `services_search.py` | 97 | 网络搜索工具封装 |
| `services_reporter.py` | 77 | 最终研究报告生成 |
| `services_notes.py` | 59 | 研究笔记/中间结论管理 |
| `services_text_processing.py` | 16 | 文本清洗工具 |
| `prompts.py` | 110 | 各阶段 Prompt 模板（规划/搜索/总结/报告） |
| `main.py` | 190 | FastAPI 入口与流式响应 |
| `config.py` | 143 | 配置（LLM/搜索 API/并发） |
| `models.py` | 51 | 数据模型 |
| `utils.py` | 84 | 通用工具（解析/限流/去重） |

### chapter15-赛博小镇（对应范式文档「四、端到端案例 · 15 多 Agent 模拟」）
> 多 Agent 模拟（AI-Town）：3 个 NPC 自主对话、记忆系统、好感度系统（FastAPI 后端 + Godot 前端）。
| 文件 | 行数 | 作用 |
|------|------|------|
| `agents.py` | 483 | 多 Agent 核心：NPC 人格/对话/自主行为编排 |
| `relationship_manager.py` | 324 | 好感度/关系系统（5 级亲密度建模与更新） |
| `main.py` | 393 | FastAPI + 对话主循环、日志落盘 |
| `batch_generator.py` | 211 | 批量对话/剧情生成 |
| `state_manager.py` | 134 | NPC 状态管理（位置/行为/记忆指针） |
| `logger.py` | 115 | 对话日志系统 |
| `view_logs.py` | 113 | 日志查看与回放 |
| `models.py` | 69 | 数据模型（NPC/对话/记忆） |
| `config.py` | 45 | 配置（LLM/NPC 数量/日志路径） |

> 记忆系统、好感度系统、对话日志的设计说明另见上游 `code/chapter15/Helloagents-AI-Town/` 下的 `MEMORY_SYSTEM_GUIDE.md` / `AFFINITY_SYSTEM_GUIDE.md` / `DIALOGUE_LOG_GUIDE.md`。

### chapter10-通信协议（对应范式文档「五、进阶主题 · 10 通信协议 MCP/A2A/ANP」）
> 智能体对外通信三大协议范式：MCP（工具接入）、A2A（Agent 间协作）、ANP（Agent 网络协议）。
| 文件 | 行数 | 作用 | 协议 |
|------|------|------|------|
| `02_Connect2MCP.py` | 101 | MCP 连接与初始化（客户端接入） | MCP |
| `03_GitHubMCP.py` | 33 | 接入 GitHub MCP 服务实战 | MCP |
| `04_MCPTransport.py` | 83 | MCP 传输层（STDIO / HTTP） | MCP |
| `05_UseMCPToolInAgent.py` | 48 | 在 Agent 中调用 MCP 工具 | MCP |
| `my_mcp_server.py` | 239 | 自建 MCP Server（工具暴露） | MCP |
| `06_MultiAgentDocumentAssist.py` | 134 | 多 Agent 文档协作编排 | Multi-Agent |
| `07_SimpleA2AAgent.py` | 84 | A2A 基础 Agent | A2A |
| `09_A2A_Server.py` | 49 | A2A 服务端（暴露能力） | A2A |
| `09_A2A_Client.py` | 21 | A2A 客户端（调用远端 Agent） | A2A |
| `10_A2ATool_Simple.py` | 26 | 把 A2A Agent 包装为本地工具 | A2A |
| `11_ANPInit.py` | 51 | ANP 节点初始化与握手 | ANP |
| `12_ANPTaskDistribution.py` | 80 | ANP 任务分发 | ANP |
| `13_ANPLoadBalancing.py` | 34 | ANP 负载均衡 | ANP |
| `14_weather_mcp_server.py` | 77 | MCP Server 完整示例（天气） | MCP |

### chapter11-Agentic-RL（对应范式文档「五、进阶主题 · 11 Agentic-RL」）
> 用强化学习训练 Agent 行为策略（数据集 / 奖励函数 / GRPO / 分布式）。
| 文件 | 行数 | 作用 |
|------|------|------|
| `01_dataset_loading.py` | 285 | 训练数据集加载与预处理 |
| `02_reward_functions.py` | 335 | 奖励函数设计（过程/结果奖励） |
| `05_grpo_training.py` | 334 | **GRPO 训练核心**（群组相对策略优化） |
| `06_complete_pipeline.py` | 262 | 完整训练流水线（SFT→RL→评估） |
| `07_model_evaluation.py` | 335 | 训练后模型评估 |
| `08_distributed_training.py` | 104 | 分布式训练（accelerate / 多卡） |

### chapter12-评估（对应范式文档「五、进阶主题 · 12 评估体系」）
> Agent / 工具调用评测：BFCL（工具调用）、GAIA（通用智能体）、LLM-as-Judge 数据生成。
| 文件 | 行数 | 作用 |
|------|------|------|
| `01_basic_agent_example.py` | 38 | 基础 Agent 评估示例 |
| `02_bfcl_quick_start.py` | 49 | BFCL 工具调用评测快速开始 |
| `04_run_bfcl_evaluation.py` | 293 | 运行 BFCL 完整评测 |
| `05_gaia_quick_start.py` | 85 | GAIA 智能体评测快速开始 |
| `07_data_generation_complete_flow.py` | 118 | 评测数据生成全流程 |
| `08_data_generation_llm_judge.py` | 167 | **LLM-as-Judge** 数据生成 |
| `09_data_generation_win_rate.py` | 170 | 胜率对比（Win Rate）数据生成 |

> 三章均为**进阶主题范式脚本**，聚焦"协议 / 训练 / 评测"三大工程化能力；完整 40+ 文件见上游 `code/chapter10|11|12/`，本目录只取最具代表性的 27 个。

## 三、与范式文档、Java 栈的关联

- **范式提炼**：`../Python自研框架与实战案例.md`（同一阶段2 目录下）
- **双栈对照**：`../Python企业级Agent工程实践.md`（ai-agent-book 生产范式 + 与 Java 栈对照表）
- **Java 侧同类能力**：`../../../03-Java生态生产实践栈/阶段2-开发工程实现/Java企业级Agent工程实践.md`（Ragent/Nexus 的企业级实现）

## 四、使用建议

1. **先读范式文档**，理解设计哲学（万物皆为工具、ContextBuilder 分区、记忆四分类）；
2. **再读本目录对应脚本**，对照范式看真实实现细节；
3. **本地跑通 chapter7 + chapter9 基础示例**（仅需 API Key），建立"从零手撕框架"的体感；
4. 需要检索/记忆能力时，再引入 chapter8 的 RAG/Memory 脚本并准备 Qdrant/Neo4j。

> 本目录为**原码归档**，不修改上游逻辑；如需在知识库内做注释版/简化版，建议在范式文档中另起"代码精读"小节，避免与上游更新脱节。
