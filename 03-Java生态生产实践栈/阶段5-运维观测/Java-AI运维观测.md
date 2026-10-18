# 阶段5 · Java AI 生产运行与运维观测

> 分级：🟡 生产最佳实践 ｜ 来源：`素材收集/03-nageoffer-ragent.md`、`素材收集/04-javaup-nexus-agent.md`
> 定位：规划"阶段5 运维观测·Java 生态"。AI 应用的可观测重点是**链路质量**——模型回答好不好、工具调没调对、检索是否召回。

## 一、可观测三支柱（Java 落地）

| 支柱 | 工具 | 采集内容 |
|------|------|---------|
| Metrics | Micrometer + Prometheus + Grafana | JVM（GC/线程池——Ragent 10 线程池 + TTL）、QPS、模型首包/总耗时、限流排队长度、熔断次数 |
| Logging | SLF4J + Logback + ELK/Loki | 结构化日志：问题改写结果、召回文档 ID、工具调用入参出参、异常 |
| Tracing | OpenTelemetry + SkyWalking / **LangFuse** | Run/Node 级链路：每步耗时、输入输出、Token 消耗、异常节点 |

> **Actuator**：Spring Boot Actuator 暴露 `/health`/`/metrics`/`/prometheus`，与 K8s 探针、Prometheus scrape 集成。

## 二、AI 链路追踪要点

- **Run/Node 级 Trace**（Ragent 基于 LangFuse）：一次问答拆成"改写→召回→Rerank→生成"节点，记录每节点输入/输出/耗时/Token，管理端可看趋势与配置 Diff。
- **工具调用追踪**：MCP 工具调用记录来源、参数、结果、耗时，定位"答非所问"在召回还是生成。
- **影子路由观测**（Nexus）：用户手动选文档时后台静默跑完整路由，对比命中率/置信度/候选排名，形成**持续优化闭环**——这是 Java 工程化特有的"用真实流量反哺路由质量"手段。

## 三、告警与稳定性运营

- **告警分级**：模型熔断（P0）、限流排队溢出（P1）、检索召回率跌至阈值（P2）。
- **Bad Case 闭环**：Trace + 用户反馈 + 回答溯源，沉淀为 Golden Set（回灌阶段3 评测）。
- **模型容错观测**：档位切换、首包超时、三态熔断触发次数监控，验证降级策略真实生效。
- **流量保护观测**：Redis ZSET 公平排队深度、Lua 抢占成功率、分布式信号量占用，防突发压垮模型。

## 四、与 Python 栈阶段5 对照

| 观测点 | Python 栈 | Java 栈 |
|--------|----------|--------|
| Trace 平台 | LangSmith/LangFuse | LangFuse 同样适用 + SkyWalking（Java 生态原生） |
| Metrics | Prometheus + 应用埋点 | Micrometer + Actuator + JVM/线程池指标 |
| 链路质量 | 同 Run/Node 级 | 同 + 影子路由观测（Nexus 特色） |
| 稳定性 | 限流/降级 | Redis 排队 + 熔断 + 事务消息幂等（更重基础设施保障） |
