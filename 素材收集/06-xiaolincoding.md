# xiaolincoding.com · 图解基础与面试素材（基础支撑域）

> 来源：https://xiaolincoding.com/（作者：小林coding，1500 图 + 100 万字）
> 定位：图解计算机网络/操作系统/MySQL/Redis + 后端面试八股 + AI Agent 面试入口。偏**计算机基础与后端底层原理**，是知识库"基础支撑域"的素材来源。
> 落位：知识库**基础支撑域（部署与基础设施 / 运维观测的底层逻辑）**，对应规划"阶段4 部署 / 阶段5 运维观测 / 横向支撑域·组织流程"。
> 采集优先级：🟢 低（量大、偏基础，按需定向采集）。本文件为索引 + 落位标注。

---

## 一、核心栏目与 URL

### 必看图解系列（基础支撑域主体）
| 栏目 | URL | 知识库落位 |
|------|-----|-----------|
| 图解网络 | https://xiaolincoding.com/network/ | 阶段4 部署（HTTP/HTTPS/TCP/IP 底层） |
| 图解系统（OS） | https://xiaolincoding.com/os/ | 阶段4/5（进程/内存/文件系统/网络系统） |
| 图解 MySQL | https://xiaolincoding.com/mysql/ | 阶段2（存储）/ 阶段5（索引/事务/MVCC/锁/BP） |
| 图解 Redis | https://xiaolincoding.com/redis/ | 阶段2（缓存）/ 阶段5（持久化/高可用/一致性） |

### AI Agent 面试入口（指向 xiaolinnote，已在 01/02 采集）
- Agent 面试题：https://xiaolinnote.com/ai/agent/agent_info.html
- RAG 面试题：https://xiaolinnote.com/ai/rag/rag_info.html
- LLM 工具调用：https://xiaolinnote.com/ai/tools/tools_info.html
- 大模型工程：https://xiaolinnote.com/ai/llm/llm_info.html
- LangChain：https://xiaolinnote.com/ai/langchain/langchain_info.html
- AI 编程：https://xiaolinnote.com/ai/coding/coding_info.html

### 后端面试八股系列（Java 栈基础）
- Java 后端面试题：https://xiaolincoding.com/interview/
- Golang：https://xiaolincoding.com/interview/golang.html
- C++：https://xiaolincoding.com/interview/cpp.html
- Python：https://xiaolincoding.com/interview/python.html
- 测试开发：https://xiaolincoding.com/interview/test_dev.html

### 面试项目（AI 方向，可作案例素材）
- DevFlow AI 项目（多 Agent 协作）：https://xiaolincoding.com/project/devflow.html
- MewHelp 智能客服：https://xiaolincoding.com/project/mewhelp.html
- MewCode Agent 项目：https://xiaolincoding.com/project/mewcode.html
- 智能 OnCall Agent：https://xiaolincoding.com/project/aioncallagent.html

---

## 二、与知识库的关联点（定向采集建议）

仅当知识库以下模块缺底层原理时，再从对应图解栏目定向抓取：

1. **阶段4 部署与基础设施**：TCP/HTTP/HTTPS 握手与性能、操作系统 I/O 与并发模型、容器化底层。
2. **阶段2 开发与工程实现（Java 栈）**：MySQL 索引/事务/MVCC/锁 用于 RAG 元数据与业务表设计；Redis 数据结构/持久化/高可用 用于会话记忆、限流、缓存。
3. **阶段5 运维观测**：Redis 缓存一致性、MySQL 慢查询与调优、操作系统监控指标。
4. **横向支撑域·组织流程**：面试项目（DevFlow/MewHelp 等）可作为"双栈典型生产案例"的补充案例库。

---

## 三、采集说明

- 本源**不逐篇全采**：内容体量极大且以计算机基础为主，与 AI 工程化直接关联度低于 01~05。
- 建议策略：在知识库正式搭建"基础支撑域"时，按上述关联点**定向抓取单篇**（如 Redis 持久化与高可用、MySQL 事务与 MVCC），而非整站搬运。
- AI 相关面试内容已归并至 01（xiaolinnote/ai）与 02（xiaolinnote/agent），避免重复。
