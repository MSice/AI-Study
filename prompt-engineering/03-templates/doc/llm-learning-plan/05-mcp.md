# 阶段 5：MCP 协议（第 11-12 周）

**学习目标**：理解 MCP（Model Context Protocol）协议的设计理念，能开发 MCP Server 和 Client，让 AI 应用通过标准化接口连接外部工具和数据源。

**前置要求**：完成阶段 3（AI Agent）和阶段 4（RAG）

---

## 知识模块

### 5.1 MCP 协议概述

- **是什么**：MCP（Model Context Protocol，模型上下文协议）就像 AI 世界的 USB 接口标准。以前每个 AI 应用要连接不同的工具（数据库、搜索引擎、文件系统等），都得自己写一套对接代码，就像早期每种设备都有自己的充电线。MCP 定义了一个统一的协议，让任何 AI 应用都能通过同一个"接口"连接任何工具。

- **为什么重要**：随着 AI Agent 越来越复杂，需要连接的工具越来越多。没有标准协议，每个工具都要单独对接，维护成本爆炸。MCP 正在成为 AI 工具连接的事实标准，Anthropic 主导推出，已被 Cursor、Claude Desktop 等主流产品采用。

- **核心概念**：
  - **Server（服务端）**：提供工具能力的一方。比如一个"天气查询 MCP Server"，它把天气 API 包装成 MCP 协议格式，任何支持 MCP 的 AI 应用都能调用。
  - **Client（客户端）**：使用工具能力的一方。比如 Claude Desktop、Cursor IDE，它们通过 MCP 协议发现和调用各种 Server 提供的工具。
  - **Tools（工具）**：Server 暴露给 Client 的具体能力，类似于 API 的 endpoint。每个工具有名称、描述和参数定义。
  - **Resources（资源）**：Server 提供的数据源，Client 可以读取。比如数据库中的表、文件系统中的文件。
  - **Prompts（提示词模板）**：Server 提供的预定义提示词模板，Client 可以直接使用。
  - **Transport（传输层）**：MCP 消息的传输方式，支持 stdio（标准输入输出，本地进程通信）和 SSE（Server-Sent Events，HTTP 远程通信）。

- **动手练习**：
  1. 安装 Claude Desktop，配置一个官方的 MCP Server（如 filesystem server），体验 MCP 的实际效果
  2. 阅读 [MCP 协议规范](https://modelcontextprotocol.io/specification)，画出 Client-Server 交互的时序图
  3. 列出 5 个你觉得适合做成 MCP Server 的工具/服务

- **推荐资源**：
  - 📖 必读：[MCP 官方文档](https://modelcontextprotocol.io/introduction) — 协议的权威说明（英文）
  - 📖 补充：[MCP 协议规范](https://modelcontextprotocol.io/specification) — 技术细节参考（英文）
  - 🛠️ 工具：Claude Desktop、Cursor IDE、`@modelcontextprotocol/sdk`

- **检验标准**：
  - MCP 解决了什么问题？和直接用 Function Calling 有什么区别？
  - MCP 的 Server 和 Client 分别是什么角色？
  - Tools、Resources、Prompts 三种能力类型有什么区别？

---

### 5.2 MCP Server 开发

- **是什么**：开发 MCP Server 就像"开一家标准化的外卖店"——你把自己的服务（工具能力）按照 MCP 的"菜单格式"包装好，任何支持 MCP 的"外卖平台"（AI 应用）都能下单调用。

- **为什么重要**：MCP Server 是连接 AI 和外部世界的桥梁。掌握 Server 开发，你就能把任何现有的 API、数据库、服务包装成 AI 可用的工具。

- **核心概念**：
  - **SDK 选择**：官方提供 TypeScript SDK（`@modelcontextprotocol/sdk`）和 Python SDK（`mcp`），选择你熟悉的语言即可。
  - **工具定义**：用 JSON Schema 描述工具的名称、功能说明、输入参数。描述要写得清楚，因为 LLM 会根据描述决定何时调用。
  - **资源暴露**：定义 Server 提供的数据资源，支持 URI 模板和动态资源列表。
  - **错误处理**：工具执行失败时返回清晰的错误信息，帮助 LLM 理解问题并重试。
  - **stdio vs SSE**：本地开发用 stdio（简单直接），远程部署用 SSE（支持网络访问）。

- **动手练习**：
  1. 用 TypeScript 或 Python SDK 开发一个最简 MCP Server，提供一个"echo"工具（输入什么返回什么）
  2. 在 Claude Desktop 中配置并测试你的 Server
  3. 扩展 Server，添加一个实用工具（如：查询 GitHub 仓库信息、读取本地 JSON 文件）

- **推荐资源**：
  - 📖 必读：[MCP Server 开发快速入门](https://modelcontextprotocol.io/quickstart/server) — 官方 Server 开发教程（英文）
  - 📖 补充：[MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) / [Python SDK](https://github.com/modelcontextprotocol/python-sdk) — SDK 源码和示例
  - 🛠️ 工具：Node.js 18+、Python 3.10+、Claude Desktop

- **检验标准**：
  - 一个 MCP Server 的基本代码结构是怎样的？
  - 如何定义一个工具的 Schema？好的工具描述有什么特征？
  - stdio 和 SSE 两种传输方式各适合什么场景？

---

### 5.3 MCP Client 开发与集成

- **是什么**：MCP Client 就像"万能遥控器"——它能发现和连接各种 MCP Server，把它们提供的工具整合到 AI 应用中。开发 Client 意味着你能构建自己的 AI 平台，让用户通过你的应用使用各种 MCP 工具。

- **为什么重要**：虽然 Claude Desktop 和 Cursor 已经是 MCP Client，但在实际项目中你可能需要构建自定义的 Client——比如在自己的 Web 应用、CLI 工具或后端服务中集成 MCP 能力。

- **核心概念**：
  - **Server 发现与连接**：Client 需要知道有哪些 Server 可用，并建立连接。可以通过配置文件指定，也可以动态发现。
  - **能力协商**：连接建立后，Client 和 Server 会交换各自支持的能力（工具列表、资源列表等）。
  - **工具调用流程**：用户输入 → LLM 决定调用哪个工具 → Client 向 Server 发送调用请求 → Server 执行并返回结果 → 结果反馈给 LLM → LLM 生成最终回答。
  - **多 Server 管理**：一个 Client 可以同时连接多个 Server，把所有工具汇聚在一起供 LLM 使用。

- **动手练习**：
  1. 用 SDK 开发一个最简 MCP Client，连接你在 5.2 中开发的 Server
  2. 在 Client 中集成 LLM（OpenAI / Claude API），实现完整的"用户提问 → LLM 选择工具 → 调用 MCP Server → 返回结果"流程
  3. 让 Client 同时连接 2 个不同的 MCP Server，测试多工具协作

- **推荐资源**：
  - 📖 必读：[MCP Client 开发快速入门](https://modelcontextprotocol.io/quickstart/client) — 官方 Client 开发教程（英文）
  - 📖 补充：[MCP 官方示例项目](https://github.com/modelcontextprotocol/servers) — 各种 MCP Server 实现参考
  - 🛠️ 工具：MCP SDK、MCP Inspector（调试工具）

- **检验标准**：
  - MCP Client 的核心职责是什么？
  - 一个完整的 MCP 工具调用流程是怎样的？
  - 如何让一个 Client 同时管理多个 Server？

---

## 阶段项目：MCP 工具生态

**项目描述**：开发一个 MCP 工具生态系统，包含 2-3 个 MCP Server 和一个自定义 MCP Client：

**Server 1：笔记管理 Server**
- 工具：创建笔记、搜索笔记、列出所有笔记
- 资源：暴露笔记列表作为 Resource

**Server 2：网页摘要 Server**
- 工具：给定 URL，抓取网页内容并生成摘要
- 工具：提取网页中的关键信息（标题、作者、日期等）

**Client：智能助手**
- 连接上述 Server，集成 LLM
- 用户可以用自然语言操作：如"帮我把这个网页的摘要保存为笔记"

**技术要求**：
- Server 使用 TypeScript 或 Python SDK
- Client 集成 OpenAI 或 Claude API
- 支持 stdio 传输
- 有完整的错误处理

**预期产出**：
- `mcp-ecosystem/` — 项目目录
  - `servers/note-server/` — 笔记管理 Server
  - `servers/web-summary-server/` — 网页摘要 Server
  - `client/` — 自定义 Client
- `README.md` — 使用说明和架构图

**预计耗时**：8-10 小时

---

## 阶段检查点

- [ ] 能解释 MCP 协议的设计理念和核心概念
- [ ] 能开发一个功能完整的 MCP Server
- [ ] 能开发一个 MCP Client 并连接 Server
- [ ] 理解 Tools、Resources、Prompts 的区别和用法
- [ ] 能在 Claude Desktop 中配置和使用自定义 MCP Server
- [ ] 完成阶段项目：MCP 工具生态

---

**下一步** → [阶段 6：LLM 应用开发实战](./06-llm-app-dev.md)
