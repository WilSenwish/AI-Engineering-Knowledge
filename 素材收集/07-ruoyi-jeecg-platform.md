# 07 · Java 平台级 AI 解决方案素材（RuoYi AI / JEECG AI）

> 本文件为《Java 平台视角补采》的原始素材沉淀，作为 `03-Java生态生产实践栈/Java平台级AI解决方案-RuoYiAI与JEECG.md` 的溯源依据。
> 采集时间：2026-10-18。来源为官方文档站 + 公网检索（GitHub/Gitee/微信/搜狐）。

---

## 一、采集目的

此前 Java 栈（03 层）以 **Ragent / Nexus** 为样本，代表"**从零自研工程化**"视角（手搭分层、双链路/三层执行器、模型容错、全链路追踪与治理）。
但企业真实落地还有另一极：**平台 / 低代码交付**——直接采用成熟 Java 底座，把 AI 能力以"开箱即用 + 私有化 + 可嵌入业务系统"的方式落地。
本批补采聚焦两个代表性开源平台：**RuoYi AI**（企业级 AI 助手平台）与 **JEECG AI**（JeecgBoot 体系下的 Java 版 AI 应用平台 / 企业级 Dify 替代），补齐"平台视角"。

---

## 二、RuoYi AI 原始素材

### 来源
- 官方文档：`https://doc.ruoyiai.chat/`（功能总览 + 各模块子页）
- GitHub（Soul-debug 镜像）：`https://github.com/Soul-debug/ruoyi-ai`
- Gitee 主仓：`https://gitee.com/ageerle/ruoyi-ai`
- 第三方解析（微信公众号「开源向导」，2026）：`http://mp.weixin.qq.com/s?__biz=MzIwOTA3ODA4Mg==&mid=2458356188&idx=2&sn=e82f58c7d9148ea6c69dae7cbc9c97f3`
- 其他镜像：`https://gitee.com/lllwkk/ruoyi-ai`、`https://xmsumi.com/detail/1950`

### 定位
- 企业级 AI 助手平台 / 一站式 AI 应用开发框架；开箱即用、简单高效、私有化部署。
- MIT 协议，GitHub Trending 当日 #1，约 4.8k stars / 1.2k forks。
- 创始维护者 ageerle，团队含林大侠、evo、龙猫等。

### 功能矩阵（按官方文档"功能总览"逐条）
- **模型管理**：统一管理 OpenAI、DeepSeek、通义千问、智谱、Ollama、自定义 OpenAI-compatible API 等多厂商模型。
- **平台接入**：接入 FastGPT、RAGFlow 及 OpenAI/Anthropic 兼容服务；Dify、Coze 提供适配代码（需补齐厂商选项与凭据）。
- **多模态接口**：图片、语音、视频、预测接口的 Provider 路由与调用；普通聊天附件边界说明。
- **知识管理**：文档解析 → 知识分块 → 向量化 → 向量库操作 → 召回 → 重排 → 混合检索，完整 RAG 链路。
- **工具管理**：Function Calling、内置工具、本地 MCP、远程 MCP、工具市场。
- **MCP 管理**：协议概念 → LOCAL / REMOTE / 魔搭 ModelScope 实战，结合后端源码做智能体工具调用、排障、扩展。
- **技能管理**：docx/pdf/xlsx 等技能资源；区分"普通聊天选择器"与"Coding Harness 真实执行链路"。
- **上下文管理**：系统提示词 + 会话历史 + 知识片段组装；验证最近 20 条历史记忆；通过 LangChain4j 示例扩展窗口、持久化、长期记忆。
- **智能体管理**：基于 LangChain4j Supervisor 编排网页搜索 / SQL / 图表 / ECharts / 闲聊角色；可预检索知识库并绑定 MCP。
- **流程编排**：可视化节点 + SSE 运行模型、知识检索、条件分支、邮件、HTTP、网络搜索、图片生成流程。

### 技术栈
- 后端：Spring Boot 3.4 / 3.5.x + Spring AI + LangChain4j（部分来源称已升级 Spring AI 1.0.0 正式版）。
- 数据：MySQL 8.0 + Redis + 向量数据库（Milvus / Weaviate / Qdrant）。
- 前端：Vue 3 + Vben Admin + Naive UI / element-plus-x（用户端 ruoyi-web、管理端 ruoyi-admin 分离）。
- 安全：Sa-Token + JWT（RBAC）。
- 实时：WebSocket + SSE 流式响应（"打字机"式）。
- 文档处理：PDF / Word / Excel 解析 + 图像智能分析。
- 本地推理：兼容 Ollama、vLLM，无 GPU 亦可跑 Qwen / Llama 等开源模型。
- 向量模型：BGE-large-zh-v1.5 中文向量模型（中文语义检索）。

### 架构特点
- 模型接入层基于抽象接口，新增模型供应商只需实现标准适配器。
- RAG 引擎与向量库解耦，按规模/成本选存储方案。
- 工作流编排拖拽式，降低流程配置技术门槛。
- 模块划分：ruoyi-admin（管理后台）、ruoyi-web（用户端）、ruoyi-modules（业务模块）、ruoyi-extend（AI 扩展）清晰分离。
- 设计理念："避免大而全陷阱"，每个模块职责清晰，支持开箱即用与深度定制双路径。

### 部署与运维
- 两种 Docker 方案：快速镜像启动（2 核 2G 轻量服务器即可）、一键脚本部署（自定义构建调优）。
- 端口规划：用户前端 8081、管理后台 8082、API 服务 6039、向量库 Weaviate 6038（不同来源端口表述略有差异）。
- 默认演示账号 admin / admin123；在线演示 web.pandarobot.chat（用户端）、admin.pandarobot.chat（管理端）。
- 提供环境变量配置、日志管理、数据备份、性能优化文档。

---

## 三、JEECG AI 原始素材

### 来源
- 官网：`https://jeecg.com`、AI 平台入口 `https://jeecg.com/aigc/index`
- 搜狐深度解析（企业级 Dify 替代）：`https://www.sohu.com/a/1025942599_100285483`
- Dify vs JEECG AI 对比：`http://mp.weixin.qq.com/s?__biz=MzIwMDE0OTgzNA==&mid=2650414004&idx=1&sn=aee1bef62fa00938d2c30f6c4a742ef9`
- JeecgBoot v3.9.3 发布（AI 低代码 2.0 / Spring Boot 4）：`https://news.sohu.com/a/1065484091_100285483`
- 开源仓：`https://github.com/jeecgboot/JeecgBoot`（或 `zhangdaiscott/jeecg-boot`）

### 定位
- 业内唯一 **Java 版开源 AI 应用平台**；企业级 Dify 替代方案。
- 与 Dify 差异：Dify 是"独立 LLM 应用开发平台"（Python + React）；JEECG AI 是"**低代码 + AIGC 应用深度融合平台**"（Java + Vue3），重视业务流程自动化与系统集成。
- 基于 JeecgBoot 低代码底座，集成 LLM + 知识库问答（RAG），支持从设计到生产完整流程。

### 功能矩阵
- **应用与对话层**：AI 应用管理（普通应用 / 高级流程应用）；AI 助手对话（ChatGPT / DeepSeek / 智谱 / 私有模型，结合企业知识库）；iframe 一键嵌入任意第三方系统。
- **知识库与 RAG**：AI 知识库（文档 / 问答对训练）；RAG 覆盖 PDF / PPT 等格式文本提取。
- **模型与编排层**：模型管理（本地私有 / 国内公共 / 国外）；AI 流程设计器引擎（画布拖拽、实时运行、查看执行情况）；**AI 流程即服务**（流程封装成功能性 API 对外调用）。
- **工具与多模态**：内置 MCP 协议支持 + 开箱即用 AI 工具箱。

### 对比 Dify 三大独有能力
1. **高保真文档处理**：支持含图片的 Markdown 文档库导入（批量）；PDF 格式与图片高保真还原（Dify 在格式丢失/图片消失上暴露短板）。
2. **低代码流程编排引擎**：AI 逻辑 + 业务系统集成（调用系统接口、业务逻辑），不仅是 AI 逻辑。
3. **强业务集成**：基于 Java 技术栈与低代码流程引擎，方便调用 ERP / CRM 等既有系统接口与业务逻辑。

### 技术栈
- JeecgBoot 底座：Spring Boot 3 + MyBatis-Plus + Vue 3 + TypeScript + Vite 6 + Ant Design Vue 4。
- 微服务：Spring Cloud Alibaba（Nacos / Gateway / Sentinel / SkyWalking）；支持单体与微服务自由切换。
- 信创适配：MySQL / Oracle / SQL Server / PostgreSQL / 达梦 / 人大金仓。
- 最新 v3.9.3：Spring Boot 4 + Vite 8 全栈升级（AI 低代码进入 2.0 时代）。
- AI 能力：AI 对话助手、AI 建表、AI 写文章、AI 流程编排；支持 ChatGPT / DeepSeek / Ollama 本地模型。
- 低代码：OnlineCoding → 代码生成器 → 手工 MERGE；在线表单、报表、大屏、流程自动化；代码生成器解决 Java 项目约 70% 重复工作。

### 文档处理优势（企业知识管理场景）
- 完整导入含图片的 Markdown 文档库。
- PDF 格式转换与图片保持大量优化，导入内容完整、视觉一致。
- 对话界面支持图片渲染，回复格式美观。

---

## 四、两平台公认同构点（提炼，供主文档引用）
- 均为 **Java + Spring Boot 3.x + Vue3** 全栈，私有化友好，适合把 AI 嵌进既有 Java 业务系统。
- 均覆盖"模型管理 → 知识库 RAG → 工具/MCP → 智能体/Agent → 流程编排"全链路。
- 均用成熟 Java 安全框架（Sa-Token / Spring Security 体系）做权限与数据隔离。
- 均强调"开箱即用 + 可二次开发"，与 Ragent/Nexus"从零自研"形成对照。
- 在 AI 流程编排上走"可视化拖拽"路线，降低非算法工程师的参与门槛。

---

## 五、待补 / 注意事项
- RuoYi AI 不同镜像仓（ageerle/lllwkk/Soul-debug）版本号与端口略有差异，主文档以官方 doc.ruoyiai.chat 为主、检索信息为辅。
- JEECG AI 与 JeecgBoot 版本演进快（v3.7.x → v3.9.3，部分已到 Spring Boot 4），主文档标注"版本敏感"。
- 两平台均为"平台/产品"而非"工程范式教材"，与已建 Java 栈的"手把手工程实践"互补，不宜混为一"教学章节"，主文档以"视角补齐 + 决策框架"组织。
