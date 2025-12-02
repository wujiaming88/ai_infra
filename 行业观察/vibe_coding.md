简单说：**vibe coding = 靠 AI“写代码”，人主要在聊天、提需求和点确认，而不怎么真的看代码。**

---

## 1. 它具体指什么？

“Vibe coding” 是 Andrej Karpathy 在 2025 年提出的一个说法，后来被媒体和大厂疯狂引用。

典型流程是这样的：

1. 你对大模型/AI 编辑器说人话需求

   > “帮我做个小 Web 服务，暴露 /hello 接口返回 JSON，顺便加个简单的前端页面。”

2. AI 帮你：

   * 搭项目结构
   * 写后端、前端代码
   * 配打包、部署脚本等

3. 你**不仔细读代码、不逐行 review**，只：

   * 跑起来看看效果
   * 发现问题就继续用自然语言说：

     > “按钮再大一点，sidebar 的 padding 减半，把接口路径改成 /api/hello。”

4. 再让 AI 自动修改代码，你继续“只看效果，不看源码”。

**关键点**：

> *不是所有“用 AI 写代码”都叫 vibe coding*，
> 真正的 vibe coding 强调的是：**完全把代码细节交给 AI，你只盯“结果和体验”，不盯“实现和结构”。**

---

## 2. 跟普通“AI 辅助写代码”有什么区别？

| 维度    | 传统编程 / 普通 Copilot | vibe coding   |
| ----- | ----------------- | ------------- |
| 人的角色  | 亲手写代码、调 API、手动搭结构 | 提需求、试效果、提修改意见 |
| 是否看代码 | 会认真看 diff 和实现     | 通常**不看或很少看**  |
| 主要输入  | 代码 + 少量注释         | 自然语言 + 截图/例子  |
| 场景定位  | 长期维护的工程项目         | 原型、玩具项目、小工具为主 |

很多大厂（Google、Replit 等）的官方文档里，现在都把 vibe coding 归类为一种**“用自然语言驱动、AI 生成代码”的开发新范式**。


> **“vibe coding 的技术核心 = 一个带工具的代码 Agent + IDE 外壳 + 沙箱执行环境。”**


## 3. 要实现的能力清单（从结果倒推）

不管叫什么架构，想做 vibe coding 至少要支撑这几件事：

1. **自然语言 → 开发计划**

   * 把“做个团购小程序，类似 XXX，加个拼团倒计时”这类 fuzzy 需求，拆成“创建工程 / 配置依赖 / 建 DB / 写接口 / 写前端 / 写部署脚本”等子任务。

2. **理解与修改现有代码库**

   * 读取代码树、搜索、理解依赖关系。
   * 支持跨多文件修改，而不是单文件补全。

3. **生成 / 编辑代码**

   * 不是一次性吐一整个项目，而是**多轮 diff/patch**，支持增量修改、重构。

4. **编译 / 运行 / 测试 / 调试**

   * 在隔离的沙箱里执行 `mvn test`、`npm run dev` 之类，把报错日志反馈给 LLM。

5. **持续对话 + 状态记忆**

   * 同一个“会话”里记住这个项目的状态、约定风格、依赖版本等。

像 Replit Agent、Copilot Workspace、Cursor 这种产品的技术本质都差不多：一个带工具的 LLM Agent 驱动代码编辑器和沙箱。

## 4. 系统总体架构

可以把 vibe coding 系统拆成 5 层：

1. **体验层（IDE / Web IDE）**

   * Chat 面板 + 文件树 + 编辑器（Monaco / VS Code 内核）+ 预览 / 终端。
   * 和后端通过 WebSocket / SSE 交互，流式显示 LLM 输出、diff、运行日志。

2. **编排层（Vibe Orchestrator / Agent Server）**

   * 负责：

     * 会话管理（一个 chat = 一个 project session）
     * 调用 LLM（planner / coder）
     * 管理工具调用（读写文件、运行命令、测试等）
     * 决定每一步做什么：生成代码、执行、再根据结果继续。

3. **工具层（Tools）**
   常见工具集合：

   * `read_file(path) / write_file(path, content)`
   * `apply_patch(path, diff)`
   * `search_code(query)`（基于索引 / 向量检索）
   * `run_command(cmd)` / `run_tests()`（受限）
   * `list_files(pattern)`
     Copilot agent mode 和 Replit Agent 都是通过“工具列表 + 系统 prompt”把这些工具暴露给 LLM 决策。

4. **执行与存储层**

   * **代码仓库**：Git repo + 工作目录（per session 或 per branch）。
   * **沙箱执行**：Docker / gVisor / Firecracker，限制 CPU / 内存 / 网络。
   * **索引存储**：代码向量索引（Faiss、PGVector 等），用于 RAG。

5. **模型与嵌入层**

   * 1 个（或多个）**代码能力强的 LLM**（gpt-4.1/4.1-mini 这一类）。
   * 1 个轻量 Embedding 模型，用来索引 & 检索代码片段。

Arxiv 那篇《Vibe Coding: Toward an AI-Native Paradigm…》给了一个类似的 4 组件参考架构：
**规格捕获（Spec Capture）→ 意图解析（Intent Interpreter）→ 代码合成（Code Synthesizer）→ 反馈循环（Feedback Loop）**，本质就是上面这几层的抽象版。