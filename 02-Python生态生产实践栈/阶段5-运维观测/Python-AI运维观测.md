# 阶段5 · Python AI 生产运行与运维观测

> 对齐理论层 [`1.7-治理与安全原则`](../../01-通用理论原理层/1.7-治理与安全原则.md) + 横向支撑「可观测/持续优化」；Java 栈对照 `../../03-Java生态生产实践栈/阶段5-运维观测`。
> 素材：ai-agent-book 第8-9章（持续进化）、hello-agents 运维实践。

## 1. 可观测性三支柱（AI 扩展）

- **指标（Metrics）**：请求量、延迟(P50/P95/P99)、Token 消耗、错误率、模型调用成功率、工具调用耗时。Prometheus 抓取 + Grafana 看板。
- **链路（Tracing）**：每次 Agent 运行的全链路——节点耗时、工具调用、检索召回、LLM 调用、Token。用 **LangSmith / LangFuse / OpenTelemetry** 自动埋点。
- **日志（Logging）**：结构化日志（JSON），含 `trace_id`/`user_id`/`session_id`，便于关联排查。

## 2. 告警分级

| 级别 | 触发 | 动作 |
|------|------|------|
| 业务告警 | 答案faithfulness 跌破阈值 / 工具调用失败率升高 | 告警 + 自动回滚评测集 |
| 资源告警 | GPU 显存/利用率异常、队列堆积 | 扩容 / 限流 |
| 质量告警 | 线上 Bad Case 聚类 | 触发优化闭环 |
| 成本告警 | Token 成本超预算 | 路由降级 / 缓存命中率检查 |

## 3. 持续优化闭环（理论 1.7 + ai-agent-book Ch9 持续进化）

- **Bad Case 闭环**：线上 Trace 标记差 → 进入 Bad Case 库 → 分析根因（检索/Prompt/工具）→ 迭代 → Golden Set 回归。
- **Prompt 迭代**：版本化管理 Prompt 模板（如 PromptLayer / 本地 YAML），A/B 对比效果。
- **检索调优**：分析低相关召回，调整切分/重写/重排参数。
- **持续进化（ai-agent-book Ch9）**：从运行轨迹提取信号，更新知识/指令/程序/参数——形成 Agent 自进化闭环。

## 4. 容量管理与故障恢复

- **流量预估**：基于历史 QPS + 活动预测，提前扩容 GPU/HPA。
- **瓶颈定位**：Trace 火焰图定位慢节点（重排？大模型？工具？）。
- **故障恢复**：无状态服务滚动重启；有状态（向量库）用副本 + 健康检查；Celery 任务重试 + 死信队列。

## 5. 与 Java 栈对照

| 维度 | Python | Java |
|------|--------|------|
| 链路 | LangSmith/LangFuse/OTel | Skywalking / OTel |
| 指标 | Prometheus + Grafana | Micrometer + Prometheus |
| 日志 | structlog / JSON | Logback/Log4j2 + ELK |
| 熔断降级 | tenacity + 自定义 | Spring Cloud 熔断/限流 |

> 双栈观测指标定义与告警模型一致，采集/展示工具因生态而异。
