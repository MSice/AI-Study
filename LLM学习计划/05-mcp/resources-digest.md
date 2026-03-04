# 阶段 5 推荐资源深度总结

> 本文将 [阶段 5：MCP 协议](./index.md) 中推荐的核心资源提炼为一篇通俗易懂的中文解读。
> 目标：即使你不阅读原始英文资源，也能建立对 MCP 协议的完整理解。

---

## 目录

- [5.1 MCP 协议概述](#51-mcp-协议概述)
  - [5.1.1 MCP 官方文档](#511-mcp-官方文档)
  - [5.1.2 MCP 协议规范](#512-mcp-协议规范)
- [5.2 MCP Server 开发](#52-mcp-server-开发)
  - [5.2.1 MCP Server 开发快速入门](#521-mcp-server-开发快速入门)
  - [5.2.2 MCP TypeScript SDK / Python SDK](#522-mcp-typescript-sdk--python-sdk)
- [5.3 MCP Client 开发与集成](#53-mcp-client-开发与集成)
  - [5.3.1 MCP Client 开发快速入门](#531-mcp-client-开发快速入门)
  - [5.3.2 MCP 官方示例项目](#532-mcp-官方示例项目)
- [总结：串联所有知识点](#总结串联所有知识点)

---

## 5.1 MCP 协议概述

### 5.1.1 MCP 官方文档

> 原文：[MCP Introduction](https://modelcontextprotocol.io/introduction)

MCP 官方介绍页用最简洁的语言解释了协议是什么、能做什么、为什么重要。

#### 核心定义：MCP 是什么？

**MCP（Model Context Protocol，模型上下文协议）** 是一个开源的、标准化的协议，用来连接 AI 应用和外部系统。

官方用了一个很好的类比：**MCP 就像 AI 应用的 USB-C 接口**。就像 USB-C 让各种电子设备能统一连接一样，MCP 让各种 AI 应用（如 Claude、ChatGPT）能统一连接到数据源、工具和工作流。

#### MCP 能实现什么？

通过 MCP，AI 应用可以连接：

| 类型 | 举例 |
|------|------|
| **数据源** | 本地文件、数据库 |
| **工具** | 搜索引擎、计算器 |
| **工作流** | 预定义的提示词模板、 specialized prompts（针对特定任务的提示词模板） |

有了这些连接，AI 就能获取关键信息并执行任务。官方举了几个生动的例子：

- AI 模型可以在 Blender 里创建 3D 设计，并通过 3D 打印机打印出来
- 企业级聊天机器人可以连接组织内的多个数据库，让用户用对话方式分析数据
- Claude Code 可以根据 Figma 设计稿生成完整的 Web 应用
- AI 助手可以访问你的 Google 日历和 Notion，成为更个性化的助手

#### 为什么 MCP 重要？不同角色的收益

| 角色 | 收益 |
|------|------|
| **最终用户** | AI 应用更强大，能访问你的数据、在必要时替你执行操作 |
| **AI 应用 / Agent 开发者** | 能接入丰富的数据源、工具和 App 生态，提升能力、改善用户体验 |
| **工具 / 数据源开发者** | 开发一次，就能被所有支持 MCP 的 AI 应用使用，降低开发和对接成本 |

#### 官方提供的下一步

文档末尾指向了三个方向：开发 Server、开发 Client、学习核心概念。这是 MCP 生态的完整闭环。

---

### 5.1.2 MCP 协议规范

> 原文：[MCP Specification](https://modelcontextprotocol.io/specification)

MCP 协议规范是技术实现的权威参考，定义了协议的基础架构、消息格式和核心能力。

#### 协议定位与设计灵感

MCP 是一个**开放协议**，让大语言模型（LLM）应用能无缝集成外部数据源和工具。无论你是做 AI 驱动的 IDE、增强聊天界面，还是自定义 AI 工作流，MCP 都提供标准化的连接方式。

规范明确提到，MCP 的灵感来自 **LSP（Language Server Protocol，语言服务器协议）**。LSP 统一了编程语言在各种开发工具中的支持方式；MCP 则统一了「额外上下文和工具」在 AI 应用生态中的集成方式。

#### 核心架构：三个角色

| 角色 | 含义 |
|------|------|
| **Server（服务端）** | 提供上下文和能力的一方 |
| **Client（客户端）** | 宿主应用中的连接器 |
| **Host（宿主）** | 发起连接的 LLM 应用 |

三者通过 **JSON-RPC 2.0** 消息进行通信。

#### 协议基础能力

- **Server 与 Client 的能力协商**：连接时互相声明支持哪些功能
- **有状态连接**：保持会话状态，支持多轮交互
- **JSON-RPC 消息格式**：统一的请求/响应结构

#### Server 提供的三种能力

| 能力 | 说明 |
|------|------|
| **Tools（工具）** | 供 AI 模型调用的函数 |
| **Prompts（提示词）** | 模板化的消息和工作流 |
| **Resources（资源）** | 供用户或模型使用的上下文和数据 |

#### Client 可提供的能力

| 能力 | 说明 |
|------|------|
| **Elicitation（征询）** | Server 主动向用户请求额外信息 |
| **Roots（根路径）** | Server 主动查询 URI 或文件系统边界 |
| **Sampling（采样）** | Server 发起的 Agent 行为和递归 LLM 调用 |

#### 版本与协商

- 版本号格式：`YYYY-MM-DD`，表示最后一次不兼容变更的日期
- 修订状态：Final（已定稿）、Current（当前可用）、Draft（草案）
- 版本协商在**初始化阶段**完成；若协商失败，Client 应优雅断开连接

#### 安全与信任原则（非常重要）

规范强调：MCP 通过任意数据访问和代码执行路径赋予强大能力，因此**安全与信任**是所有实现者必须认真对待的问题。

**核心原则：**

1. **用户知情与可控**
   - 提供清晰的界面，让用户审核和授权操作
   - 用户必须能控制哪些数据被共享、哪些操作被执行
   - 所有数据访问和操作都需要用户明确同意

2. **数据隐私**
   - 用户数据应有适当的访问控制
   - Host 不得在未经用户同意的情况下将资源数据传输到别处
   - 在将用户数据暴露给 Server 之前，Host 必须获得用户明确同意

3. **工具安全**
   - 用户应在授权前了解每个工具的作用
   - Host 在调用任何工具前必须获得用户明确同意
   - 工具描述（如注释）应视为不可信，除非来自可信 Server
   - 工具代表任意代码执行，必须谨慎对待

4. **LLM 采样控制**
   - 协议有意限制 Server 对以下内容的可见性：实际发送的提示词、采样是否发生、采样结果
   - 用户应控制这些行为
   - 任何 LLM 采样请求都必须经用户明确批准

**实现建议：** 虽然 MCP 本身无法在协议层强制这些原则，但实现者应当：在功能设计中考虑隐私影响、遵循安全最佳实践、实现适当的访问控制、清晰记录安全影响、在应用中构建完善的同意和授权流程。

---

## 5.2 MCP Server 开发

### 5.2.1 MCP Server 开发快速入门

> 原文：[MCP Server Quickstart](https://modelcontextprotocol.io/quickstart/server)

官方 Server 快速入门教程通过一个**天气 MCP Server** 的完整示例，教你从零搭建 Server 并接入 Claude Desktop。

#### 教程目标

构建一个提供两个工具的 Server：`get_alerts`（获取天气警报）和 `get_forecast`（获取天气预报），然后将其连接到 Claude Desktop。

#### MCP Server 的三种核心能力

在动手之前，教程先介绍了 MCP Server 可以提供的三种能力：

| 能力 | 说明 | 控制方 |
|------|------|--------|
| **Prompts（提示词）** | 预写模板，帮助用户完成特定任务 | 用户 |
| **Tools（工具）** | 可被 LLM 调用的函数（需用户批准） | 模型 |
| **Resources（资源）** | 类似文件的数据，Client 可读取（如 API 响应、文件内容） | 应用 |

本教程主要聚焦于 **Tools**。

#### 重要：STDIO 模式下的日志禁忌

使用 **stdio（标准输入输出）** 传输时，**绝对不能向 stdout（标准输出）写任何内容**，否则会破坏 JSON-RPC 消息，导致 Server 崩溃。

| 语言 | ❌ 错误 | ✅ 正确 |
|------|---------|---------|
| Python | `print("xxx")` | `print("xxx", file=sys.stderr)` 或 `logging.info()` |
| TypeScript | `console.log()` | `console.error()` |
| Java | `System.out.println()` | 使用写入 stderr 的日志库 |
| C# | `Console.WriteLine()` | 使用写入 stderr 的日志库 |
| Rust | `println!()` | `eprintln!()` |

HTTP 模式的 Server 不受此限制。

#### Python 版快速实现要点

**环境要求：** Python 3.10+，MCP Python SDK 1.2.0+

**核心步骤：**

1. 使用 `FastMCP` 创建 Server 实例
2. 用 `@mcp.tool()` 装饰器定义工具，利用类型注解和 docstring 自动生成工具定义
3. 工具函数接收参数、调用外部 API（如美国国家气象局 API）、返回格式化的字符串
4. 用 `mcp.run(transport="stdio")` 启动 Server

**Claude Desktop 配置示例：**

```json
{
  "mcpServers": {
    "weather": {
      "command": "uv",
      "args": ["--directory", "/绝对路径/weather", "run", "weather.py"]
    }
  }
}
```

#### TypeScript 版快速实现要点

**环境要求：** Node.js 16+

**核心步骤：**

1. 使用 `McpServer` 创建 Server 实例
2. 用 `server.registerTool()` 注册工具，传入名称、描述、Zod 定义的 inputSchema、以及执行函数
3. 工具执行函数返回 `{ content: [{ type: "text", text: "..." }] }` 格式
4. 使用 `StdioServerTransport` 连接并启动

**Claude Desktop 配置示例：**

```json
{
  "mcpServers": {
    "weather": {
      "command": "node",
      "args": ["/绝对路径/weather/build/index.js"]
    }
  }
}
```

#### 工具调用流程（幕后发生了什么）

当用户提问时，大致流程是：

1. Client 将问题发给 Claude
2. Claude 分析可用工具，决定调用哪些
3. Client 通过 MCP Server 执行选中的工具
4. 结果返回给 Claude
5. Claude 生成自然语言回复
6. 回复展示给用户

#### 故障排查要点

- **Server 不显示：** 完全退出 Claude Desktop 后重启；确保路径是绝对路径；检查 JSON 语法
- **工具调用失败：** 重启 Claude；验证 Server 能独立运行；查看 `~/Library/Logs/Claude/mcp*.log`
- **天气 API 错误：** 检查是否被限流、坐标是否在美国境内（NWS API 仅支持美国）

#### 其他语言支持

官方快速入门还提供了 Java（Spring AI）、Kotlin、C#、Rust 等语言的完整示例，结构类似，都是：创建 Server → 注册工具 → 配置传输层 → 在 Claude Desktop 中配置。

---

### 5.2.2 MCP TypeScript SDK / Python SDK

> 原文：[TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) / [Python SDK](https://github.com/modelcontextprotocol/python-sdk)

MCP 官方提供 TypeScript 和 Python 两种主流语言的 SDK，用于开发 MCP Server 和 Client。

#### TypeScript SDK

- **仓库：** `@modelcontextprotocol/sdk`（npm 包名）
- **用途：** 开发 MCP Server 和 Client
- **特点：** 与 Node.js 生态集成好，支持 stdio 和 SSE 传输
- **典型用法：**
  - Server：`McpServer` + `StdioServerTransport` 或 HTTP 传输
  - 工具注册：`server.registerTool(name, schema, handler)`
  - 参数校验：常配合 Zod 定义 inputSchema

#### Python SDK

- **仓库：** `mcp`（PyPI 包名）
- **用途：** 开发 MCP Server 和 Client
- **特点：** 支持 `FastMCP` 等高级封装，用装饰器和类型注解即可定义工具
- **典型用法：**
  - Server：`FastMCP("server-name")` + `@mcp.tool()` 装饰器
  - 工具定义：函数签名和 docstring 自动生成 Schema
  - 传输：`mcp.run(transport="stdio")` 或 HTTP

#### 选择建议

| 场景 | 推荐 |
|------|------|
| 前端 / Node 生态、已有 TypeScript 项目 | TypeScript SDK |
| 数据科学、后端服务、快速原型 | Python SDK |
| 需要同时开发 Server 和 Client | 两种 SDK 都支持，选你熟悉的语言即可 |

#### 学习建议

- 直接看官方 quickstart 的完整代码（见 5.2.1）
- 在 GitHub 仓库中查看 `examples/` 或 `samples/` 目录
- 遇到问题可查阅 Issues 和官方文档

---

## 5.3 MCP Client 开发与集成

### 5.3.1 MCP Client 开发快速入门

> 原文：[MCP Client Quickstart](https://modelcontextprotocol.io/quickstart/client)

官方 Client 快速入门教你构建一个**能连接 MCP Server、并集成 LLM 的聊天客户端**，实现完整的「用户提问 → LLM 选工具 → 调用 MCP → 返回结果」流程。

#### 教程目标

开发一个基于 LLM 的聊天客户端，能连接任意 MCP Server，在对话中自动发现和调用 Server 提供的工具。

#### 核心工作流程（7 步）

当用户提交一个问题时：

1. Client 从 MCP Server 获取可用工具列表
2. Client 将用户问题和工具描述一起发给 LLM（如 Claude）
3. LLM 分析后决定是否调用工具、调用哪些
4. Client 通过 MCP Server 执行工具调用
5. 工具结果返回给 Client
6. Client 将结果反馈给 LLM
7. LLM 生成最终回复，Client 展示给用户

#### Python 版实现要点

**环境要求：** `uv`、Python、macOS 或 Windows

**核心组件：**

1. **连接 Server：** 使用 `StdioServerParameters` 指定启动命令（如 `python` 或 `node`）和脚本路径
2. **获取工具列表：** `session.list_tools()` 得到工具名称、描述、inputSchema
3. **调用 Claude API：** 将 `messages` 和 `tools` 传给 Anthropic API
4. **处理 tool_use：** 若 Claude 返回 `tool_use`，用 `session.call_tool()` 执行，再把结果作为 `tool_result` 加入消息，继续请求 Claude

**运行方式：**

```bash
uv run client.py path/to/server.py   # Python Server
uv run client.py path/to/build/index.js  # Node Server
```

#### TypeScript 版实现要点

**环境要求：** Node.js 17+、Anthropic API Key

**核心组件：**

1. **连接：** `StdioClientTransport` 指定 command 和 args
2. **工具列表：** `mcp.listTools()` 后转换为 Claude 所需的 `Tool` 格式
3. **对话循环：** 调用 `anthropic.messages.create()`，遍历 `response.content`，遇到 `tool_use` 则 `mcp.callTool()`，将结果加入 messages 后再次请求

**运行方式：**

```bash
npm run build
node build/index.js path/to/server.py
node build/index.js path/to/build/index.js
```

#### 最佳实践

| 方面 | 建议 |
|------|------|
| **错误处理** | 优雅处理连接失败、超时；工具调用用 try-catch 包裹 |
| **资源管理** | 使用 `AsyncExitStack`（Python）或 `finally` 确保关闭连接 |
| **安全** | API Key 放 `.env`，不提交到版本库；谨慎授权工具权限 |
| **工具名称** | 遵循规范格式，避免 Client 校验失败 |

#### 常见问题

- **路径问题：** 使用绝对路径；Windows 下可用 `/` 或 `\\`
- **首次响应慢：** Server 启动、LLM 推理、工具执行都需要时间，首次可能需 30 秒
- **常见错误：** `Connection refused`（检查 Server 路径）、`ANTHROPIC_API_KEY is not set`（检查 .env）、`Tool execution failed`（检查工具所需环境变量）

#### 其他语言

教程还包含 Java（Spring AI + Brave Search 示例）、Kotlin、C# 等实现，结构类似：建立 MCP 连接 → 获取工具 → 集成 LLM → 处理 tool_use 循环。

---

### 5.3.2 MCP 官方示例项目

> 原文：[MCP Servers Repository](https://github.com/modelcontextprotocol/servers)

MCP 官方示例仓库汇集了各种 MCP Server 的实现，是学习「如何把真实能力包装成 MCP Server」的绝佳参考。

#### 仓库定位

这是 MCP 生态的**官方 Server 示例集合**，包含文件系统、数据库、搜索引擎、日历、Slack 等常见场景的 Server 实现。

#### 典型 Server 类型（结合文档中的概念）

| 类型 | 可能包含的 Server | 主要能力 |
|------|-------------------|----------|
| **文件与文档** | filesystem、Google Drive | Resources、Tools |
| **数据与搜索** | Brave Search、数据库类 | Tools |
| **协作与日历** | Slack、Google Calendar | Tools、Resources |
| **开发工具** | GitHub、Figma | Tools、Resources |

#### 学习建议

1. **按需选读：** 挑和你业务相近的 Server 看实现
2. **关注结构：** 工具如何定义、Resources 如何暴露、错误如何返回
3. **对照规范：** 结合 MCP 规范理解每个 Server 的协议用法
4. **直接复用：** 很多 Server 可通过 `npx` 等方式直接运行，如 `npx -y @modelcontextprotocol/server-brave-search`

#### 与 Server 概念的对应关系

结合 [Understanding MCP servers](https://modelcontextprotocol.io/docs/learn/server-concepts) 文档，可以这样理解：

- **Tools：** 模型主动调用，如搜索航班、发邮件、创建日历事件
- **Resources：** 应用按需读取的被动数据，如文件内容、数据库 schema、API 文档
- **Prompts：** 用户主动选择的模板，如「规划假期」「总结会议」

多 Server 协作时，一个 Client 可同时连接多个 Server，把工具、资源和提示词汇聚在一起，供 LLM 统一使用。

---

## 总结：串联所有知识点

学完以上资源，你可以建立起这样的认知框架：

```
MCP 协议全景
├── 是什么？→ AI 应用的「USB 接口」标准，连接数据源、工具、工作流（官方文档）
├── 怎么设计？→ JSON-RPC、Server/Client/Host 角色、Tools/Resources/Prompts 三种能力（协议规范）
├── 安全原则？→ 用户知情可控、数据隐私、工具安全、LLM 采样控制（协议规范）
├── 开发 Server？
│   ├── 选 SDK：TypeScript 或 Python（或 Java/Kotlin/C#/Rust）
│   ├── 定义工具：名称、描述、inputSchema、执行逻辑
│   ├── 注意 STDIO 下不能写 stdout
│   └── 在 Claude Desktop 中配置 command 和 args（Server 快速入门）
├── 开发 Client？
│   ├── 连接 Server → 获取工具列表
│   ├── 将工具描述传给 LLM
│   ├── 处理 tool_use → 调用 MCP Server → 将结果反馈给 LLM
│   └── 循环直到 LLM 生成最终回复（Client 快速入门）
└── 参考实现？→ 官方 servers 仓库中的各种 Server 示例
```

**关键认知：**

1. **MCP 解决的是「对接」问题。** 不是替代 Function Calling，而是让「工具」和「数据」以统一协议暴露，任何支持 MCP 的 AI 应用都能复用，无需为每个应用单独写对接代码。

2. **Server 三种能力各司其职。** Tools 由模型决定何时调用；Resources 由应用决定何时读取；Prompts 由用户决定何时使用。理解这一点能帮你设计更清晰的 Server。

3. **安全与信任是实现的底线。** 规范中的安全原则不是可选项，而是所有实现者都应考虑的设计约束。

4. **动手比看书重要。** 建议按顺序：先配置一个官方 Server（如 filesystem）到 Claude Desktop 体验 → 跟着教程写一个最简单的 echo Server → 再写一个 Client 连接它 → 最后尝试多 Server 协作。

---

**返回** → [阶段 5：MCP 协议](./index.md)
