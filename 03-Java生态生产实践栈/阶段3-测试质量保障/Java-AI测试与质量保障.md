# 阶段3 · Java AI 测试与质量保障

> 分级：🟡 生产最佳实践 ｜ 来源：`素材收集/03-nageoffer-ragent.md`、`素材收集/04-javaup-nexus-agent.md`、`素材收集/05-javaguide-ai.md`（评测体系）
> 定位：规划"阶段3 测试质量保障·Java 生态"。LLM 应用的非确定性是测试核心挑战——断言策略与效果评估必须区别于传统 Java 单测。

## 一、测试栈基础（传统 + AI）

| 类型 | 工具 | 说明 |
|------|------|------|
| 单元测试 | JUnit 5 / TestNG + Mockito | 工具函数、分块、改写、融合等非 LLM 逻辑 |
| 集成测试 | Spring Boot Test / Testcontainers | 起 PGVector/ES/Redis 容器验证检索链路 |
| Mock 模型 | 本地 stub `ChatClient` | 固定返回的假模型，跑通编排不烧钱 |
| 契约测试 | JSON Schema 校验 | MCP 工具输入输出、模型方言适配器 |

> **关键路径测试**（Ragent 强调）：对"分块→改写→混合检索→Rerank→生成"主链路做端到端断言，比散点单测更能兜住质量。

## 二、LLM 输出的非确定性断言

模型输出不固定，不能逐字 `assertEquals`：

- **结构化校验**：断言返回 JSON 字段存在、类型正确、枚举合法（用 JSON Schema / 注解校验）。
- **相似度阈值**：对关键文本用语义相似度（如 `>=0.85`）替代精确匹配。
- **规则断言**：断言"必须包含引用来源""不得出现禁词""调用次数 <= 上限"。
- **快照 + 人工复核**：首版存快照，模型升级后 diff 由人确认回归。

## 三、RAG 效果评估（Golden Set + 指标）

对齐理论层 `1.6-评估体系`，Java 侧落地：

- **Golden Set**：业务沉淀的"问题→标准答案/标准召回文档"集，随版本回归。
- **离线指标**：忠实度（Faithfulness）、答案相关度（Answer Relevance）、上下文召回率（Context Recall）、噪声敏感度。可用 RAGAS/DeepEval 的 Java 绑定或自研打分（LLM-as-Judge）。
- **回归门禁**：CI 中跑 Golden Set，关键指标低于阈值则阻断合并。

## 四、Agent / 工具调用测试

- **工具契约**：每个 MCP 工具定义 Schema，测试用非法入参验证校验拦截。
- **防死循环**：Nexus 用 `Hook` 限制调用次数；测试制造"工具持续失败"场景断言循环被截断。
- **ReAct 链路**：Mock 模型返回固定 Thought/Action，断言最终 Observation 收敛、Checkpoint 持久化正确。
- **人工确认路径**：写操作工具测试"未确认不执行"的分支。

## 五、性能与稳定性压测

- **并发压测**：JMeter / Gatling 模拟并发问答，验证 Redis 公平排队（Ragent：ZSET + Lua 原子抢占）与限流生效。
- **模型故障注入**：随机让候选模型超时/空响应，验证三态熔断降级、首包探测切换。
- **长链路超时**：SSE 全局超时 + 客户端断开取消（框架层 `framework` 模块能力）需专项验证。

## 六、与 Python 栈阶段3 对照

| 测试点 | Python 栈 | Java 栈 |
|--------|----------|--------|
| 评测框架 | RAGAS/DeepEval 原生 | 同指标，Java 绑定/自研 + CI 回归 |
| 链路测试 | pytest + Testcontainers | JUnit 5 + Testcontainers |
| 压测 | Locust | JMeter / Gatling |
| 关键路径 | 强调 RAG Pipeline 端到端 | 同 + 强调"关键路径测试"与限流验证 |
