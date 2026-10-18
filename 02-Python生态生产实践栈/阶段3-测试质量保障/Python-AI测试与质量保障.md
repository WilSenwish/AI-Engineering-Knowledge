# 阶段3 · Python AI 测试与质量保障

> 对齐理论层 [`1.6-评估体系`](../../01-通用理论原理层/1.6-评估体系.md)；Java 栈对照 `../../03-Java生态生产实践栈/阶段3-测试质量保障`。
> 素材：hello-agents 第12章（性能评估）、ai-agent-book 第7章（评估，14 实验）。

## 1. 评估类型与指标（理论 1.6）

- **离线评估**：Golden Set（黄金集）固定问答对，回归测试防退化。Python 用 `pytest` + 评测脚本批量跑。
- **在线评估**：生产 Trace 回放、灰度/AB。用 LangFuse 导出轨迹做回放。
- **核心指标**：任务完成率、工具调用准确率、召回@K、答案忠实度（faithfulness）、相关性（relevancy）、延迟、Token 消耗。

## 2. 自动化评估实现

- **RAGAS / DeepEval**：一行接入评估 RAG 的 faithfulness / context_precision / answer_relevancy。
  ```python
  from ragas import evaluate
  from ragas.metrics import faithfulness, answer_relevancy
  # dataset: 含 question / answer / contexts / ground_truth
  result = evaluate(dataset, metrics=[faithfulness, answer_relevancy])
  ```
- **LLM-as-Judge**：用强模型当裁判打分（理论 1.6 四类偏差防护：位置/长度/自我偏好/格式）。Judge Prompt 固定 + 多裁判取均值 + 人工抽检校准。
- **Agent 行为用例**：`pytest` 写场景——给定输入，断言工具调用序列/最终答案。工具可 mock（`monkeypatch`）。

## 3. 基准测试接入（ai-agent-book Ch7 / hello Ch12）

| 基准 | 度量 | Python 接入 |
|------|------|------------|
| **GAIA** | 通用 Agent 综合推理 | 官方 eval 脚本 + 自实现工具集 |
| **OSWorld** | 计算机操作（GUI） | 虚拟机/容器环境 + `browser-use` 类 |
| **SWE-bench** | 代码修复 | 仓库复现 + 补丁验证 |

- 实验轨道：ai-agent-book 第6/7/8/10章映射 22 个外部仓库（固定 SHA 克隆），如 `browser-use`、`OSWorld`、`SWE-bench`、`verl`。

## 4. 性能压测与成本统计

- 接口压测：`locust` / `pytest-benchmark` 模拟并发，统计 P50/P95 延迟、吞吐。
- Token 统计：LLM 客户端 wrapper 累计 `usage`，按模型/路由维度聚合成本。

## 5. CI 回归与阈值（理论 1.6 Java 落地思路对齐）

- GitHub Actions：`uv run pytest tests/eval --benchmark` 跑 Golden Set，faithfulness < 阈值则失败。
- 评测集随版本演进（新增 Bad Case 即补入），防止"优化一个、退化一片"。

## 6. 与 Java 栈对照

| 维度 | Python | Java |
|------|--------|------|
| 测试框架 | pytest + RAGAS/DeepEval | JUnit/TestNG + 自研 EvalRecord |
| 基准 | GAIA/OSWorld/SWE-bench 原生 Python | 对接评估指标服务 |
| 压测 | locust | JMeter |

> 双栈共用同一套评估方法论（理论 1.6），指标定义一致，工具实现差异。
