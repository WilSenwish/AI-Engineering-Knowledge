# xiaolinnote.com/ai/ · 大模型面试题素材

> 来源：https://xiaolinnote.com/ai/
> 定位：小林面试笔记出品的 AI 大模型面试题合集，覆盖 Agent / RAG / 工具调用 / 大模型工程 / LangChain / AI 编程 六大专题，题目来自大厂真实面经，配手绘图解与"面试现场复盘"。
> 落位：知识库**通用理论层（核心素材）**，对应规划"阶段1 架构设计 / 阶段2 开发实现 / 阶段3 测试评估"。
> 采集方式：入口页已抓全文章地图与专题结构；逐题正文为待补深度素材。

---

## 一、专题总览（六大专题）

| 专题 | 规模 | 信息页（完整目录） |
|------|------|-------------------|
| Agent 面试专题 | 30 题 | https://xiaolinnote.com/ai/agent/agent_info.html |
| RAG 面试专题 | 23 题 | https://xiaolinnote.com/ai/rag/rag_info.html |
| LLM 工具调用面试专题 | 22+ 题 | https://xiaolinnote.com/ai/tools/tools_info.html |
| 大模型工程面试专题 | 23 题 | https://xiaolinnote.com/ai/llm/llm_info.html |
| LangChain 面试专题 | — | https://xiaolinnote.com/ai/langchain/langchain_info.html |
| AI 编程面试专题 | — | https://xiaolinnote.com/ai/coding/coding_info.html |

---

## 二、Agent 面试专题（30 题，URL 全集）

基础与架构：`1_whatisagent`(什么是Agent/与大模型区别) · `2_components`(核心组件) · `3_workflow_tools`(Workflow/Agent/Tools 区别) · `4_patterns`(其他范式) · `5_react`(ReAct 实现) · `6_three_patterns`(ReAct/Plan-and-Execute/Reflection 选型) · `7_tasksplit`(任务拆分) · `8_memory`(记忆机制设计) · `9_memory_storage`(长短期记忆存储) · `10_multiagent`(Multi-Agent) · `11_single_multi`(单/多 Agent 方案)

工程实践：`12_memcompress`(记忆压缩) · `13_handcode`(为何手搓 Agent) · `14_planning`(规划能力) · `15_reflection`(反思机制) · `16_collab`(多 Agent 协作/动态切换) · `17_context_engineering`(上下文工程) · `18_session_state`(多轮状态/中断恢复) · `19_evaluation`(效果评估/指标) · `20_loop_stability`(路径震荡/死循环治理)

生产治理：`21_agent_latency_trace`(Trace 定位延迟) · `22_multi_agent_resilience`(超时/失联/并发冲突) · `23_task_hallucination`(任务幻觉) · `24_database_security`(DB 越权/泄露) · `25_harness`(Harness 是什么) · `26_harness_design`(Harness 设计) · `27_self_evolution`(自进化) · `28_human_confirmation`(人工确认) · `29_context_compression_threshold`(压缩阈值) · `30_execution_variance`(执行结果方差)

> 基址：`https://xiaolinnote.com/ai/agent/<id>.html`

---

## 三、RAG 面试专题（23 题，URL 全集）

基础：`1_whatisrag`(是什么/流程) · `2_rag_problems`(解决什么) · `3_rag_vs_finetune`(vs 微调) · `4_chunking`(切分策略) · `5_semantic_cuts`(语义切断) · `6_embedding`(Embedding 选型) · `7_embedding_algos`(算法) · `8_vectordb`(向量库选型) · `9_vectordb_practice`(实践)

检索优化：`10_online_workflow`(在线流程) · `11_retrieval_types`(向量vs关键词) · `12_query_rewrite`(查询改写) · `13_multi_retrieval`(多路召回) · `14_retrieval_opt`(检索优化) · `15_advanced_paradigms`(Self-RAG/Corrective RAG) · `16_graph_db`(图库增强) · `17_hallucination`(幻觉规避) · `18_evaluation`(效果量化) · `19_dynamic_update`(动态更新) · `20_hardest_parts`(最难处) · `21_knowledge_conflict`(知识冲突) · `22_rag_long_context`(大窗口为何还需RAG) · `23_rag_vs_file_search`(vs Agent 文件检索)

> 基址：`https://xiaolinnote.com/ai/rag/<id>.html`

---

## 四、大模型工程面试专题（23 题，URL 全集）

底层原理：`what_is_llm` · `transformer_architecture` · `mha_mqa_gqa_flash_attention` · `position_encoding` · `tokenizer`
训练与对齐：`llm_training` · `scaling_law_emergence` · `finetuning` · `lora` · `post_training`(RLHF/DPO/GRPO) · `dpo_vs_ppo`
推理与生成：`decoding_strategies` · `temperature_top_p_top_k` · `kv_cache_prompt_caching` · `quantization`(INT8/INT4/AWQ/GPTQ)
应用与 Prompt：`prompt_engineering` · `cot` · `hallucination` · `moe` · `deployment_frameworks`(vLLM/TGI/llama.cpp/SGLang)
评测与选型：`evaluation_metrics` · `model_selection` · `23_long_context_lost_middle`(Lost in the Middle)

> 基址：`https://xiaolinnote.com/ai/llm/<id>.html`

---

## 五、工具调用 / LangChain / AI 编程（待补正文）

- **工具调用** 信息页：https://xiaolinnote.com/ai/tools/tools_info.html — 覆盖 Function Calling、Tools、MCP、A2A、Skills、SSE 传输协议、LLM 网关；已见 `21_execution_sandbox`(代码沙箱)、`22_skill_loading`(Skill 加载)。
- **LangChain** 信息页：https://xiaolinnote.com/ai/langchain/langchain_info.html — Chain、Tools、Memory、LangGraph、LlamaIndex、框架选型、DeepResearch。
- **AI 编程** 信息页：https://xiaolinnote.com/ai/coding/coding_info.html — AI 编程方式、需求与验收、Spec 驱动、上下文管理、任务拆分、代码审查、调试测试。

---

## 六、核心理论体系提炼（来自专题导言，可作为知识库"理论底座"骨架）

1. **Agent 本质**：`Agent = LLM 在循环中自主使用工具的系统`，四大组件 = 大脑(LLM) + 规划 + 记忆 + 工具。
2. **设计范式**：ReAct（边想边做）/ Plan-and-Execute（先想后做）/ Reflection（做完自查）。面试高频，需讲清代价与适用场景。
3. **Workflow vs Agent**：控制权在代码 vs 模型；成本约为 1:4；生产多为两者混合。
4. **记忆**：短期（上下文）+ 长期（外部存储跨会话）；需设计存储粒度、压缩、跨会话保留。
5. **上下文工程**：动态组装、压缩阈值、执行结果差异控制；直接决定长任务稳定性。
6. **RAG 链路**：文档解析 → 切分 → Embedding → 向量库 → 检索（多路/改写/重排）→ 生成；答非所问多因召回阶段。
7. **生产治理**：Trace 定位延迟、死循环治理、数据库越权防护、人工确认、自进化、执行方差控制。
8. **LLM 工程**：KV Cache/Prompt Caching 省 90% 输入 token；量化降本；vLLM 等推理框架选型；评测集自建。

> 下一步：逐题抓取正文，按规划"阶段1~6"归位为"理论原理 / 生产最佳实践 / 待验证方案"三级。
