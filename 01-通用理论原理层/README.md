# 01-通用理论原理层（归一化沉淀）

> 定位：知识库的理论底座，所有生产实践共享的"归一化"认知。本层不绑定具体语言/框架，提炼跨栈通用的本质、范式、架构模式、评估与治理原则。
> 来源素材：`素材收集/01-xiaolinnote-ai-大模型面试题.md`、`素材收集/02-xiaolinnote-agent-图解Agent.md`、`素材收集/05-javaguide-ai.md`（理论部分）。

## 本层模块地图

| 模块 | 覆盖主题 | 主要来源 |
|------|---------|---------|
| `1.1-Agent本质与核心范式.md` | Agent 定义、四大组件、ReAct/Plan-Execute/Reflection 范式、Workflow vs Agent、提示词工程基础 | xiaolinnote.ai(Agent30题) / xiaolinnote.agent(Agent是什么) / javaguide.ai(prompt-engineering) |
| `1.2-上下文与记忆工程.md` | 提示词工程、上下文工程(Context Assembler/加载策略/长任务三法)、记忆系统(分层/生命周期/存储三层/产品/遗忘)、Harness 六层框架与案例 | xiaolinnote.ai / xiaolinnote.agent(Harness) / javaguide.ai(agent-memory/context-engineering/harness-engineering/prompt-engineering) |
| `1.3-RAG技术体系.md` | RAG 流程、切分/Embedding/向量库/重排、GraphRAG/LightRAG、vs 微调、演进三阶段 | xiaolinnote.ai(RAG23题) / javaguide.ai(RAG基础+graphrag) / xiaolinnote.agent(rag) |
| `1.4-工具体系与通信协议.md` | Function Call / MCP / Skills / A2A 分层与关系 | xiaolinnote.agent(concept) / javaguide.ai(mcp/skills) |
| `1.5-多Agent协作与编排.md` | 多Agent协作模式(9种)、Graph Engineering、Workflow/Graph/Loop 对比、任务拆分/状态/冲突/失败恢复 | xiaolinnote.agent(graph-engineering) / javaguide.ai(workflow-graph-loop/loop-engineering/multi-agent) |
| `1.6-评估体系.md` | 评测vs监控、四类评测、核心指标、Golden Set 构建、LLM-as-Judge 防偏、Trace回放/灰度、Java 落地(EvalRecord) | xiaolinnote.ai(评估) / javaguide.ai(llm-evaluation) |
| `1.7-治理与安全原则.md` | Prompt 注入、越权/泄露防护、数据合规、可解释性、大模型网关 | javaguide.ai(安全实战/网关) / xiaolinnote.ai(治理) |

## 素材覆盖状态（截至当前·全库 8 阶段完成）
- **[已完成] 理论层 7 模块均已完成首轮填充 + 多轮补采回灌**：
  - 第一轮：1.1–1.7 骨架（来源 01/02/05 理论部分）。
  - 第二/三轮补采：回灌 GraphRAG/LightRAG、MCP/Skills、Loop Engineering、安全/网关、Agent 记忆/上下文工程/Harness/提示词/多 Agent/评测体系——**javaguide.ai 理论子页已全量纳入，1.2 与 1.6 正式封口**。
  - 本层作为全库底座，已被 `02-Python栈`、`03-Java栈`、`04-横向支撑域` 共享引用。
- **[可深化] 高价值来源（非阻塞，按需补充）**：
  - xiaolinnote.ai 逐题正文（Agent30/RAG23/工具22/LLM23/LangChain/AI编程）——增厚 1.1/1.3/1.4 实例。
  - xiaolinnote.agent `openclaw.html`、`loop_engineering_handbook.html`——补 Loop Engineering 实操。
  - javaguide.ai RAG 子页（rag-document-processing / rag-vector-store / rag-optimization / graphrag / rag-knowledge-update）——补强 1.3 工程细节。

> 本层内容分级：理论原理 / 生产最佳实践 / 待验证方案（文中按需标注）。
