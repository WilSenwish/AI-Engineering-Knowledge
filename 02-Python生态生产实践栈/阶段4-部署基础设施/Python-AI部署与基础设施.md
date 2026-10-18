# 阶段4 · Python AI 部署与基础设施

> 对齐横向支撑域「平台建设/基础设施」；Java 栈对照 `../../03-Java生态生产实践栈/阶段4-部署基础设施`。
> 素材：ai-agent-book 部署类实验、hello-agents 工程化部署实践。

## 1. 服务封装（API 调用模式）

- **FastAPI + Uvicorn**：异步入口，`StreamingResponse` 支持流式（SSE / 原生 stream）。
  ```python
  @app.post("/chat")
  async def chat(req: ChatReq):
      return StreamingResponse(stream_generator(req), media_type="text/event-stream")
  ```
- **Gunicorn + Uvicorn workers**：多进程生产部署，`-k uvicorn.workers.UvicornWorker -w N`。
- **批量/后台任务**：Celery / ARQ 处理长时 Agent 任务，避免阻塞请求。

## 2. 容器化

- **多阶段构建**瘦身：builder 装依赖 → runtime 仅拷贝 `site-packages` + 代码。
- **CUDA 镜像**：`FROM nvidia/cuda:12.x-runtime` + 安装 torch（GPU 推理）；或 CPU 镜像 + `onnxruntime` 降本。
- **Python 专属**：用 `uv` 导出 `requirements.txt` 或 `uv lock` 保证可复现；非 root 用户运行。

## 3. K8s 编排（GPU 推理服务集群）

- **GPU 资源调度**：`resources.limits.nvidia.com/gpu: 1` + NodeSelector/taint；`nvidia-device-plugin` 暴露 GPU。
- **弹性伸缩**：HPA 按 QPS/RT/自定义指标（如队列长度）扩缩；推理服务用 KEDA 基于队列深度。
- **滚动发布 + 灰度**：新版本 canary + 流量切分；配合 LangFuse 影子对比。
- **健康检查**：`/healthz`（存活）、`/readyz`（就绪，检查模型/向量库连接）。

## 4. 推理加速（私有化部署）

- **vLLM**：OpenAI 兼容推理服务，PagedAttention 提吞吐；`vllm serve qwen2.5-7b --gpu-memory-utilization 0.9`。
- **TensorRT-LLM / SGLang**：更低延迟；量化（AWQ/GPTQ）降显存。
- **模型路由**：按任务复杂度路由小/大模型（理论 1.1 选型 + 成本），Python 侧用 `litellm` 统一网关。

## 5. 缓存与降本

- **语义缓存**：相同/近似 Query 直接返回缓存答案（如 `gptcache`）。
- **Embedding 缓存**：文档入库后 Embedding 持久化，避免重复计算。
- **批处理**：离线批量推理用 batch endpoint 降单价。

## 6. 与 Java 栈对照

| 维度 | Python | Java |
|------|--------|------|
| 服务 | FastAPI/Uvicorn/Gunicorn | Spring Boot 内嵌 Tomcat/Netty |
| 镜像 | CUDA 镜像 + uv 锁 | JRE 瘦身 + 分层构建 |
| 推理 | vLLM/TensorRT-LLM（GPU） | 对接第三方 API 网关 / 私有化客户端 |
| 网关 | litellm 统一路由 | 大模型网关（Kong/Spring Cloud Gateway） |

> 差异点：Python 侧常自带推理服务（vLLM）跑 GPU 集群；Java 侧多对接外部模型服务/网关。底层 K8s / 可观测思路一致。
