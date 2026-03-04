# MCP Server 快速搭建详解 — TypeScript 版

> 本文档手把手带你从零搭建一个 MCP Server，每一步都有详细注释说明**为什么要这么做**。  
> 完成后你将拥有一个可在 Claude Desktop / Cursor 中使用的自定义 MCP 工具。

---

## 目录

1. [前置知识](#1-前置知识)
2. [环境准备](#2-环境准备)
3. [项目初始化](#3-项目初始化)
4. [编写 MCP Server](#4-编写-mcp-server)
5. [构建与运行](#5-构建与运行)
6. [接入 Claude Desktop 测试](#6-接入-claude-desktop-测试)
7. [进阶：添加 Resource 和 Prompt](#7-进阶添加-resource-和-prompt)
8. [常见问题排查](#8-常见问题排查)

---

## 1. 前置知识

### MCP 是什么？

MCP（Model Context Protocol）是 Anthropic 推出的开放协议，定义了 AI 应用（Client）与外部工具/数据源（Server）之间的通信标准。

**核心架构**：

```
┌──────────────┐    MCP 协议     ┌──────────────┐
│  MCP Client  │ ◄────────────► │  MCP Server  │
│ (Claude/Cursor)│   JSON-RPC    │  (你开发的)   │
└──────────────┘                └──────────────┘
```

### Server 能提供什么？

| 能力类型 | 说明 | 类比 |
|---------|------|------|
| **Tools** | LLM 可调用的函数 | API endpoint |
| **Resources** | 可读取的数据源 | 文件/数据库 |
| **Prompts** | 预定义的提示词模板 | 快捷指令 |

本教程以 **Tools** 为主线，最后补充 Resource 和 Prompt。

---

## 2. 环境准备

### 2.1 安装 Node.js

```bash
# 检查是否已安装（需要 v18+）
node --version
npm --version
```

> **为什么要 Node.js 18+？**  
> MCP TypeScript SDK 使用了原生 `fetch` API 和 ES Module 等现代特性，这些在 Node.js 18 之后才稳定支持。

如果未安装，前往 [nodejs.org](https://nodejs.org/) 下载 LTS 版本。

### 2.2 确认 TypeScript 环境

```bash
# 不需要全局安装 TypeScript，后面会作为项目依赖安装
# 但如果你想全局使用 tsc 命令：
npm install -g typescript
```

---

## 3. 项目初始化

### 3.1 创建项目目录

```bash
mkdir my-mcp-server
cd my-mcp-server
```

### 3.2 初始化 npm 项目

```bash
npm init -y
```

> **为什么用 `npm init -y`？**  
> `-y` 跳过交互式问答，直接用默认值生成 `package.json`。后面我们会手动修改关键配置。

### 3.3 安装依赖

```bash
# 运行时依赖
npm install @modelcontextprotocol/sdk zod@3

# 开发依赖
npm install -D @types/node typescript
```

**各依赖的作用**：

| 包名 | 类型 | 作用 |
|------|------|------|
| `@modelcontextprotocol/sdk` | 运行时 | MCP 官方 SDK，提供 Server/Client 类、传输层、协议实现 |
| `zod@3` | 运行时 | Schema 验证库，用于定义工具的输入参数类型。SDK 内部用 Zod 来生成 JSON Schema 给 LLM 看 |
| `@types/node` | 开发 | Node.js 的 TypeScript 类型定义 |
| `typescript` | 开发 | TypeScript 编译器，将 `.ts` 编译为 `.js` |

> **为什么用 Zod 而不是手写 JSON Schema？**  
> Zod 提供了 TypeScript 原生的类型推导 + 运行时验证。你写一次 Zod schema，同时得到：  
> 1. TypeScript 类型检查（编译时）  
> 2. JSON Schema（给 LLM 看，让它知道参数格式）  
> 3. 运行时输入验证（防止 LLM 传错参数）

### 3.4 配置 package.json

打开 `package.json`，添加/修改以下字段：

```json
{
  "name": "my-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "my-mcp-server": "./build/index.js"
  },
  "scripts": {
    "build": "tsc && chmod 755 build/index.js",
    "dev": "tsc --watch"
  },
  "files": ["build"]
}
```

**逐字段解释**：

| 字段 | 值 | 为什么 |
|------|---|--------|
| `"type": "module"` | — | 启用 ES Module（`import/export`）。MCP SDK 是 ESM 包，不设置这个会报 `ERR_REQUIRE_ESM` |
| `"bin"` | 指向编译产物 | 让这个包可以作为 CLI 工具运行。Claude Desktop 会通过 `node build/index.js` 启动你的 Server |
| `"build"` 脚本 | `tsc && chmod 755` | 先编译 TS → JS，再给产物加可执行权限（macOS/Linux 需要） |
| `"files"` | `["build"]` | 如果发布到 npm，只包含编译后的文件，不包含源码 |

### 3.5 配置 tsconfig.json

在项目根目录创建 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./build",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

**关键配置解释**：

| 配置项 | 值 | 为什么 |
|-------|---|--------|
| `target: "ES2022"` | — | 编译目标。ES2022 支持 top-level await、`Array.at()` 等，Node 18+ 完全支持 |
| `module: "Node16"` | — | 模块系统。`Node16` 让 TS 正确处理 `.js` 扩展名的 ESM 导入（MCP SDK 的导入路径带 `.js`） |
| `moduleResolution: "Node16"` | — | 模块解析策略，与 `module` 配套。处理 `package.json` 的 `exports` 字段 |
| `outDir: "./build"` | — | 编译输出目录 |
| `rootDir: "./src"` | — | 源码目录，编译时保持目录结构 |
| `strict: true` | — | 开启所有严格类型检查，减少运行时错误 |

### 3.6 创建源码目录

```bash
mkdir src
touch src/index.ts
```

此时项目结构：

```
my-mcp-server/
├── node_modules/
├── src/
│   └── index.ts        ← 主文件，我们的 Server 代码写在这里
├── package.json
├── package-lock.json
└── tsconfig.json
```

---

## 4. 编写 MCP Server

下面我们在 `src/index.ts` 中编写完整的 MCP Server。我会把代码拆成几个部分，逐段解释。

### 4.1 导入与常量

```typescript
#!/usr/bin/env node

// ---- 导入 MCP SDK 核心模块 ----

// McpServer：MCP 服务端的核心类，负责注册工具、处理请求
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";

// StdioServerTransport：基于标准输入/输出的传输层
// 为什么用 stdio？因为 Claude Desktop 通过启动子进程 + stdin/stdout 与 Server 通信
// 这是最简单的本地部署方式，不需要网络配置
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

// Zod：用于定义工具参数的 Schema
// LLM 需要知道每个工具接受什么参数、什么类型、什么约束
// Zod schema 会被自动转换为 JSON Schema 传给 LLM
import { z } from "zod";

// ---- 常量定义 ----
// 这里以一个"笔记管理"工具为例，你可以替换成任何你想包装的服务
const NOTES_STORE: Map<string, string> = new Map();
```

> **为什么第一行是 `#!/usr/bin/env node`？**  
> 这是 Unix shebang 行，让系统知道用 `node` 来执行这个文件。当 Claude Desktop 直接运行编译后的 `.js` 文件时，系统需要知道用什么解释器。

> **为什么导入路径带 `.js` 后缀？**  
> 这是 Node.js ESM 的要求。即使源码是 `.ts`，导入时必须写 `.js`，因为 TypeScript 编译后文件扩展名是 `.js`。这是 `module: "Node16"` 的行为。

### 4.2 创建 Server 实例

```typescript
// 创建 MCP Server 实例
// name 和 version 会在 Client 连接时通过"初始化握手"发送给 Client
// Client 可以据此判断连接的是什么 Server、什么版本
const server = new McpServer({
  name: "my-notes-server",   // Server 名称，会显示在 Claude Desktop 的工具列表中
  version: "1.0.0",          // 语义化版本号，方便 Client 做兼容性判断
});
```

> **为什么需要 name 和 version？**  
> MCP 协议的初始化阶段，Client 和 Server 会交换身份信息和能力声明。name 帮助用户识别工具来源，version 用于兼容性管理。

### 4.3 注册工具（Tool）

这是 MCP Server 的核心——定义 LLM 可以调用的工具。

```typescript
// ---- 工具 1：添加笔记 ----
server.registerTool(
  // 参数 1：工具名称（唯一标识符）
  // LLM 会根据这个名称来调用工具，所以要语义清晰
  "add_note",

  // 参数 2：工具元数据
  {
    // description：工具的功能描述
    // 这段文字非常重要！LLM 根据 description 来判断什么时候该调用这个工具
    // 写得越清晰，LLM 的工具选择就越准确
    description: "Add a new note with a title and content. Use this when the user wants to save or remember information.",

    // inputSchema：定义工具接受的参数
    // 用 Zod 定义，SDK 会自动转换为 JSON Schema
    // LLM 看到 JSON Schema 后，会按照格式生成参数
    inputSchema: {
      // .describe() 里的文字会作为参数说明传给 LLM
      // 帮助 LLM 理解每个参数的含义和格式要求
      title: z.string().describe("The title of the note, used as unique identifier"),
      content: z.string().describe("The content/body of the note"),
    },
  },

  // 参数 3：工具执行函数
  // 当 LLM 决定调用这个工具时，这个函数会被执行
  // 参数会被 Zod 自动验证，类型安全
  async ({ title, content }) => {
    NOTES_STORE.set(title, content);

    // 返回值格式是 MCP 协议规定的
    // content 是一个数组，支持多种类型（text、image、resource 等）
    // 这里返回纯文本结果，LLM 会读取这个结果来生成最终回答
    return {
      content: [
        {
          type: "text" as const,
          text: `Note "${title}" has been saved successfully.`,
        },
      ],
    };
  }
);

// ---- 工具 2：获取笔记 ----
server.registerTool(
  "get_note",
  {
    description: "Retrieve a note by its title. Use this when the user asks about a previously saved note.",
    inputSchema: {
      title: z.string().describe("The title of the note to retrieve"),
    },
  },
  async ({ title }) => {
    const content = NOTES_STORE.get(title);

    if (!content) {
      // 工具执行"失败"时，不要抛异常
      // 而是返回清晰的错误信息，让 LLM 理解发生了什么并告知用户
      return {
        content: [
          {
            type: "text" as const,
            text: `Note "${title}" not found. Available notes: ${
              NOTES_STORE.size > 0
                ? Array.from(NOTES_STORE.keys()).join(", ")
                : "none"
            }`,
          },
        ],
      };
    }

    return {
      content: [
        {
          type: "text" as const,
          text: `# ${title}\n\n${content}`,
        },
      ],
    };
  }
);

// ---- 工具 3：列出所有笔记 ----
server.registerTool(
  "list_notes",
  {
    description: "List all saved notes. Use this when the user wants to see what notes are available.",
    // 无参数的工具：inputSchema 传空对象
    inputSchema: {},
  },
  async () => {
    if (NOTES_STORE.size === 0) {
      return {
        content: [
          {
            type: "text" as const,
            text: "No notes saved yet.",
          },
        ],
      };
    }

    const notesList = Array.from(NOTES_STORE.entries())
      .map(([title, content]) => `- **${title}**: ${content.substring(0, 50)}...`)
      .join("\n");

    return {
      content: [
        {
          type: "text" as const,
          text: `Saved notes:\n\n${notesList}`,
        },
      ],
    };
  }
);
```

> **为什么 description 这么重要？**  
> LLM 不会看你的代码实现，它只能看到工具的 `name`、`description` 和 `inputSchema`。description 写得好不好，直接决定了 LLM 能否在正确的时机调用正确的工具。

> **为什么返回值是 `{ content: [{ type: "text", text: "..." }] }` 这种格式？**  
> 这是 MCP 协议规定的标准返回格式。`content` 数组支持多种内容类型（文本、图片、嵌入资源等），方便扩展。即使只返回文本，也要遵循这个格式。

### 4.4 启动 Server

```typescript
// ---- 启动 Server ----
async function main() {
  // 创建 stdio 传输层
  // stdio 传输通过进程的 stdin/stdout 通信
  // 这是 MCP 最常用的本地传输方式：
  //   - Client（如 Claude Desktop）启动 Server 作为子进程
  //   - Client 通过子进程的 stdin 发送请求（JSON-RPC 格式）
  //   - Server 通过 stdout 返回响应
  //   - 简单、安全、不需要网络配置
  const transport = new StdioServerTransport();

  // 将 Server 连接到传输层
  // 连接后 Server 开始监听 stdin 上的 JSON-RPC 消息
  await server.connect(transport);

  // 日志输出到 stderr，不是 stdout！
  // 这一点至关重要：stdout 是 MCP 的通信通道（JSON-RPC 消息）
  // 如果你用 console.log()（写入 stdout），会破坏 JSON-RPC 消息流，导致 Server 崩溃
  // console.error() 写入 stderr，不会干扰通信
  console.error("My Notes MCP Server is running on stdio");
}

main().catch((error) => {
  console.error("Fatal error in main():", error);
  process.exit(1);
});
```

> **为什么绝对不能用 `console.log()`？**  
> 这是 MCP stdio 传输的头号陷阱。`console.log()` 写入 stdout，而 stdout 是 Server 和 Client 之间的 JSON-RPC 通信通道。任何非 JSON-RPC 的内容出现在 stdout 上，都会被 Client 当作损坏的消息，导致连接断开。**所有调试日志必须用 `console.error()`（写入 stderr）**。

---

## 5. 构建与运行

### 5.1 编译 TypeScript

```bash
npm run build
```

> **为什么要编译？**  
> Node.js 不能直接运行 `.ts` 文件（除非用 `tsx` 等工具）。`tsc` 将 TypeScript 编译为 JavaScript，输出到 `build/` 目录。Claude Desktop 运行的是编译后的 `.js` 文件。

编译后的项目结构：

```
my-mcp-server/
├── build/
│   └── index.js        ← 编译产物，Claude Desktop 运行这个
├── src/
│   └── index.ts        ← 源码
├── package.json
├── tsconfig.json
└── ...
```

### 5.2 本地验证

```bash
# 直接运行，看是否报错
node build/index.js
```

如果看到 stderr 输出 `My Notes MCP Server is running on stdio`，说明 Server 启动成功。按 `Ctrl+C` 退出。

> **为什么运行后看起来"卡住了"？**  
> 因为 Server 在等待 stdin 上的 JSON-RPC 消息。这是正常的——在实际使用中，Client 会通过 stdin 发送请求。

---

## 6. 接入 Claude Desktop 测试

### 6.1 获取项目绝对路径

```bash
# macOS / Linux
pwd
# 输出类似：/Users/yourname/projects/my-mcp-server
```

> **为什么必须用绝对路径？**  
> Claude Desktop 启动 Server 子进程时，工作目录不确定。相对路径可能解析到错误的位置，导致找不到文件。

### 6.2 配置 Claude Desktop

打开配置文件：

```bash
# macOS
code ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

添加你的 Server 配置：

```json
{
  "mcpServers": {
    "my-notes": {
      "command": "node",
      "args": ["/Users/yourname/projects/my-mcp-server/build/index.js"]
    }
  }
}
```

**配置字段说明**：

| 字段 | 说明 |
|------|------|
| `"my-notes"` | Server 的显示名称，可以自定义 |
| `"command"` | 启动命令，这里是 `node` |
| `"args"` | 命令参数，传入编译后的 JS 文件的**绝对路径** |

> **为什么 command 是 `node` 而不是直接运行 JS 文件？**  
> 虽然我们加了 shebang 行，但 Windows 上不支持 shebang。显式指定 `node` 作为命令更跨平台。

### 6.3 重启 Claude Desktop

**必须完全退出再重新打开**（不是关窗口）：
- macOS：`Cmd + Q` 或菜单栏 → Quit Claude
- Windows：系统托盘右键 → Quit

> **为什么要完全退出？**  
> Claude Desktop 只在启动时读取配置文件。关闭窗口不会退出进程（它会驻留在系统托盘），所以配置不会生效。

### 6.4 验证

打开 Claude Desktop，在输入框旁边找到工具图标（"+"号），应该能看到你的 `my-notes` Server 和它提供的 3 个工具。

试试对话：
- "帮我保存一条笔记，标题是'MCP学习'，内容是'今天学会了搭建MCP Server'"
- "我保存了哪些笔记？"
- "查看'MCP学习'这条笔记"

---

## 7. 进阶：添加 Resource 和 Prompt

### 7.1 添加 Resource

Resource 让 Client 可以读取 Server 提供的数据：

```typescript
// 注册一个 Resource：暴露所有笔记的列表
// Resource 用 URI 标识，Client 通过 URI 来读取
server.resource(
  // Resource 名称
  "notes-list",
  // URI：使用自定义协议前缀，格式自定义
  "notes://all",
  // 元数据
  {
    description: "A list of all saved notes",
    mimeType: "text/plain",
  },
  // 读取函数：Client 请求这个 URI 时执行
  async () => {
    const allNotes = Array.from(NOTES_STORE.entries())
      .map(([title, content]) => `${title}: ${content}`)
      .join("\n\n");

    return {
      contents: [
        {
          uri: "notes://all",
          text: allNotes || "No notes yet.",
          mimeType: "text/plain",
        },
      ],
    };
  }
);
```

> **Tool 和 Resource 的区别？**  
> - **Tool**：LLM 主动调用的"动作"，有输入参数，会产生副作用（如写数据）  
> - **Resource**：Client 主动读取的"数据"，类似 GET 请求，只读不写

### 7.2 添加 Prompt

Prompt 是预定义的提示词模板，Client 可以直接使用：

```typescript
// 注册一个 Prompt 模板
server.prompt(
  // Prompt 名称
  "summarize-notes",
  // 元数据
  {
    description: "Generate a summary of all saved notes",
  },
  // 生成函数：返回一组消息，Client 会把这些消息发给 LLM
  async () => {
    const allNotes = Array.from(NOTES_STORE.entries())
      .map(([title, content]) => `## ${title}\n${content}`)
      .join("\n\n");

    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Please summarize the following notes:\n\n${allNotes || "No notes available."}`,
          },
        },
      ],
    };
  }
);
```

> **为什么需要 Prompt？**  
> 有些常用的交互模式可以封装成模板，用户一键触发，不需要每次手动输入。比如"总结所有笔记"、"根据笔记生成周报"等。

---

## 8. 常见问题排查

### Server 没有出现在 Claude Desktop 中

1. **检查配置文件语法**：JSON 格式错误（多余逗号、缺少引号）会导致整个配置失效
2. **检查路径**：必须是绝对路径，不能用 `~` 或相对路径
3. **检查编译**：确保执行过 `npm run build`，`build/index.js` 存在
4. **完全重启**：必须 `Cmd+Q` 退出 Claude Desktop，不是关窗口

### 工具调用失败

1. **查看日志**：
   ```bash
   # macOS
   tail -f ~/Library/Logs/Claude/mcp-server-my-notes.log
   tail -f ~/Library/Logs/Claude/mcp.log
   ```
2. **检查 console.log**：确保代码中没有 `console.log()`，全部改为 `console.error()`
3. **手动测试**：直接运行 `node build/index.js`，看是否有报错

### TypeScript 编译报错

| 错误 | 原因 | 解决 |
|------|------|------|
| `Cannot find module '...'` | 导入路径缺少 `.js` 后缀 | 加上 `.js`，如 `from "./utils.js"` |
| `ERR_REQUIRE_ESM` | package.json 缺少 `"type": "module"` | 添加该字段 |
| `Cannot use import statement` | 同上 | 同上 |

---

## 完整代码参考

最终的 `src/index.ts` 完整代码：

```typescript
#!/usr/bin/env node

import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const NOTES_STORE: Map<string, string> = new Map();

const server = new McpServer({
  name: "my-notes-server",
  version: "1.0.0",
});

server.registerTool(
  "add_note",
  {
    description: "Add a new note with a title and content. Use this when the user wants to save or remember information.",
    inputSchema: {
      title: z.string().describe("The title of the note, used as unique identifier"),
      content: z.string().describe("The content/body of the note"),
    },
  },
  async ({ title, content }) => {
    NOTES_STORE.set(title, content);
    return {
      content: [{ type: "text" as const, text: `Note "${title}" has been saved successfully.` }],
    };
  }
);

server.registerTool(
  "get_note",
  {
    description: "Retrieve a note by its title. Use this when the user asks about a previously saved note.",
    inputSchema: {
      title: z.string().describe("The title of the note to retrieve"),
    },
  },
  async ({ title }) => {
    const content = NOTES_STORE.get(title);
    if (!content) {
      return {
        content: [{
          type: "text" as const,
          text: `Note "${title}" not found. Available notes: ${
            NOTES_STORE.size > 0 ? Array.from(NOTES_STORE.keys()).join(", ") : "none"
          }`,
        }],
      };
    }
    return {
      content: [{ type: "text" as const, text: `# ${title}\n\n${content}` }],
    };
  }
);

server.registerTool(
  "list_notes",
  {
    description: "List all saved notes. Use this when the user wants to see what notes are available.",
    inputSchema: {},
  },
  async () => {
    if (NOTES_STORE.size === 0) {
      return {
        content: [{ type: "text" as const, text: "No notes saved yet." }],
      };
    }
    const notesList = Array.from(NOTES_STORE.entries())
      .map(([title, content]) => `- **${title}**: ${content.substring(0, 50)}...`)
      .join("\n");
    return {
      content: [{ type: "text" as const, text: `Saved notes:\n\n${notesList}` }],
    };
  }
);

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("My Notes MCP Server is running on stdio");
}

main().catch((error) => {
  console.error("Fatal error in main():", error);
  process.exit(1);
});
```

---

## 下一步

- 把内存中的 `Map` 替换为真实的数据库（SQLite / PostgreSQL）
- 添加更多工具（搜索笔记、删除笔记、导出笔记）
- 尝试 SSE 传输，让 Server 可以远程访问
- 阅读 [MCP 官方文档](https://modelcontextprotocol.io/) 了解更多协议细节
