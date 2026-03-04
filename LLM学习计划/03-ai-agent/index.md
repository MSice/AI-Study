# 阶段 3：AI Agent（第 5-7 周）

**学习目标**：理解 AI Agent 的核心架构，能从零开发一个具备工具调用能力的 Agent 应用。

**前置要求**：完成阶段 2（提示词工程）

> 📚 本阶段配套阅读：[推荐资源深度总结](./resources-digest.md) — 将本阶段所有推荐资源的核心内容提炼为一篇通俗易懂的中文解读，帮助你快速建立全局认知。

---

## 知识模块

### 3.1 Agent 概念与架构

- **是什么**：AI Agent（智能体）就像一个能自己想、自己干的 AI 助手。普通的 LLM 只能"你问我答"，而 Agent 能自己制定计划、调用工具、检查结果，直到完成任务。就像你雇了一个能独立干活的助理，而不只是一个只会回答问题的客服。

- **为什么重要**：Agent 是 LLM 从"聊天工具"进化为"生产力工具"的关键。它让 AI 能真正帮你做事——搜索信息、操作数据库、调用 API、写文件——而不只是说说而已。

- **核心概念**：
  - **感知-思考-行动循环（Perceive-Think-Act）**：Agent 的基本工作模式。感知环境信息 → 思考下一步做什么 → 执行动作 → 观察结果 → 继续思考……循环往复直到任务完成。
  - **ReAct 模式**：Reasoning + Acting 的结合。模型先"推理"（我应该做什么），再"行动"（调用工具），然后"观察"（看看结果），如此循环。这是目前最主流的 Agent 架构。
  - **工具调用（Tool Use / Function Calling）**：让 LLM 能调用外部工具（搜索引擎、计算器、API 等）。模型不直接执行，而是输出"我要调用 XX 工具"的指令，由程序代为执行。
  - **记忆（Memory）**：Agent 的"笔记本"。短期记忆是当前对话上下文，长期记忆可以用数据库存储历史信息。
  - **规划（Planning）**：Agent 把复杂任务拆解为多个子任务的能力。好的 Agent 会先想清楚步骤再动手。

- **动手练习**：
  1. 在纸上画出一个 ReAct Agent 处理"帮我查今天北京天气并推荐穿搭"这个任务的完整流程图
  2. 用 ChatGPT / Claude 的 Function Calling 功能，体验工具调用的效果
  3. 阅读一个开源 Agent 项目（如 AutoGPT）的 README，理解其架构

- **推荐资源**：
  - 📖 必读：[LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) — Lilian Weng 的经典 Agent 综述（英文）
  - 🎬 推荐：[ReAct 原始论文](https://arxiv.org/abs/2210.03629) — 理解 ReAct 模式的学术来源（英文）
  - 🛠️ 工具：OpenAI Function Calling、LangChain Agents
  - 💡 **中文解读**：[推荐资源深度总结 - 3.1 & 3.2 节](./resources-digest.md#31-lilian-wengllm-驱动的自主-agent)

- **检验标准**：
  - Agent 和普通的 LLM 对话有什么本质区别？
  - ReAct 模式的三个步骤是什么？
  - 为什么 Agent 需要"记忆"？短期记忆和长期记忆有什么区别？

---

### 3.2 Function Calling 实战

- **是什么**：Function Calling（函数调用）就像给 LLM 一个工具箱。你告诉模型"你有这些工具可以用"，模型在需要时会说"我要用 XX 工具，参数是 YY"，然后你的程序真正执行这个工具并把结果告诉模型。就像老板说"帮我查一下某某数据"，秘书去查了告诉老板结果。

- **为什么重要**：Function Calling 是 Agent 的"手和脚"。没有它，LLM 只能纸上谈兵；有了它，LLM 能真正操作外部世界。

- **核心概念**：
  - **函数定义（Function Schema）**：用 JSON Schema 描述工具的名称、功能、参数。模型根据这个描述决定何时调用什么工具。
  - **工具选择（Tool Selection）**：模型根据用户请求和可用工具列表，自动选择最合适的工具。
  - **参数提取（Parameter Extraction）**：模型从用户的自然语言中提取出工具需要的结构化参数。"帮我查北京明天的天气" → `{"city": "北京", "date": "明天"}`。
  - **多轮工具调用**：复杂任务可能需要连续调用多个工具，每次调用的结果会影响下一步决策。

- **动手练习**：
  1. 用 OpenAI 的 Function Calling API 实现一个能"查天气 + 做计算"的简单 Agent
  2. 定义 3 个工具函数（如：搜索、计算、时间查询），让模型自动选择调用
  3. 实现一个多轮对话，模型根据上一步结果决定下一步操作

- **推荐资源**：
  - 📖 必读：[OpenAI Function Calling 文档](https://platform.openai.com/docs/guides/function-calling) — 官方 Function Calling 使用指南（英文）
  - 📖 补充：[Anthropic Tool Use 文档](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview) — Claude 的工具调用方式（英文）
  - 🛠️ 工具：Python `openai` 库、`anthropic` 库
  - 💡 **中文解读**：[推荐资源深度总结 - 3.3 & 3.4 节](./resources-digest.md#33-openai-function-calling-文档)

- **检验标准**：
  - Function Calling 的工作流程是怎样的？（定义工具 → 模型选择 → 执行 → 返回结果）
  - 如何用 JSON Schema 定义一个工具？
  - 模型如何决定是否需要调用工具？

---

### 3.3 Agent 框架实战

- **是什么**：Agent 框架就像"Agent 的脚手架"——它帮你处理了工具注册、对话管理、循环控制等通用逻辑，你只需要专注于定义工具和业务逻辑。就像用 Web 框架（Express/Flask）开发网站，不用自己写 HTTP 解析。

- **为什么重要**：从零写 Agent 很繁琐（要处理循环、错误、记忆、并发等），框架能大幅提升开发效率。掌握主流框架是 Agent 开发的必备技能。

- **核心概念**：
  - **LangChain**：最流行的 LLM 应用开发框架，提供了 Agent、Chain、Tool 等丰富的抽象。生态最大，但有时过度抽象。
  - **LangGraph**：LangChain 团队推出的 Agent 编排框架，用图（Graph）的方式定义 Agent 的工作流，更灵活可控。
  - **CrewAI**：多 Agent 协作框架，适合需要多个 Agent 分工合作的场景。
  - **自定义 Agent**：理解框架原理后，用原生 API 自己实现 Agent 循环，更轻量更可控。

- **动手练习**：
  1. 用 LangChain 的 `create_react_agent` 创建一个带搜索工具的 Agent
  2. 用 LangGraph 实现同样的 Agent，对比两种方式的代码结构差异
  3. 不用任何框架，纯用 OpenAI API + 循环实现一个最简 ReAct Agent（约 50 行代码）

- **推荐资源**：
  - 📖 必读：[LangGraph 官方教程](https://langchain-ai.github.io/langgraph/tutorials/introduction/) — 从零构建 Agent 的最佳教程（英文）
  - 📖 补充：[LangChain 官方文档](https://python.langchain.com/docs/introduction/) — LangChain 全面参考（英文）
  - 🛠️ 工具：LangChain、LangGraph、LangSmith（调试追踪）
  - 💡 **中文解读**：[推荐资源深度总结 - 3.5 & 3.6 节](./resources-digest.md#35-langgraph-官方教程)

- **检验标准**：
  - LangChain 和 LangGraph 的核心区别是什么？
  - 什么时候该用框架，什么时候该自己写 Agent？
  - 一个最简 ReAct Agent 的代码结构是怎样的？

---

## 阶段项目：个人助理 Agent

**项目描述**：开发一个命令行个人助理 Agent，能根据用户的自然语言指令自动完成以下任务：
- 搜索网页信息（调用搜索 API）
- 读写本地文件
- 执行简单的数据计算
- 发送消息通知（如调用 Webhook）

**技术要求**：
- 使用 LangGraph 或纯 OpenAI API 实现 ReAct 循环
- 至少注册 4 个工具
- 支持多轮对话，Agent 能记住上下文
- 有清晰的日志输出，能看到 Agent 的"思考过程"
- 有基本的错误处理（工具调用失败时的重试/降级）

**预期产出**：
- `personal_agent/` — 项目目录
  - `agent.py` — Agent 核心逻辑
  - `tools/` — 工具定义
  - `config.py` — 配置管理
- `README.md` — 使用说明和架构图

**预计耗时**：6-8 小时

---

## 阶段检查点

- [ ] 能画出 ReAct Agent 的工作流程图
- [ ] 能用 OpenAI Function Calling 实现工具调用
- [ ] 能用 LangGraph 或 LangChain 构建 Agent
- [ ] 能不依赖框架，用原生 API 实现最简 Agent
- [ ] 理解 Agent 的记忆和规划机制
- [ ] 完成阶段项目：个人助理 Agent

---

**下一步** → [阶段 4：RAG 检索增强生成](../04-rag/index.md)（如果还没学的话）或 [阶段 5：MCP 协议](../05-mcp/index.md)
