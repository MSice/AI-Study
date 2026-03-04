# MCP Server 快速搭建详解 — Python 版

> 本文档手把手带你从零搭建一个 MCP Server，每一步都有详细注释说明**为什么要这么做**。  
> 完成后你将拥有一个可在 Claude Desktop / Cursor 中使用的自定义 MCP 工具。

---

## 目录

1. [前置知识](#1-前置知识)
2. [环境准备](#2-环境准备)
3. [项目初始化](#3-项目初始化)
4. [编写 MCP Server](#4-编写-mcp-server)
5. [运行与测试](#5-运行与测试)
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

### 2.1 安装 Python

```bash
# 检查是否已安装（需要 3.10+）
python3 --version
```

> **为什么要 Python 3.10+？**  
> MCP Python SDK 使用了 `match` 语句、`TypeAlias`、`ParamSpec` 等 3.10+ 的语法特性。低版本会直接报语法错误。

### 2.2 安装 uv（推荐的包管理器）

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# 安装后重启终端，验证安装
uv --version
```

> **为什么用 uv 而不是 pip？**  
> `uv` 是 Rust 编写的 Python 包管理器，速度比 pip 快 10-100 倍，且自带虚拟环境管理。MCP 官方文档推荐使用 uv。当然你也可以用 pip + venv，后面会给出替代命令。

---

## 3. 项目初始化

### 3.1 创建项目

**方式 A：使用 uv（推荐）**

```bash
# 创建项目目录并初始化
uv init my-mcp-server
cd my-mcp-server

# 创建虚拟环境并激活
uv venv
source .venv/bin/activate    # macOS/Linux
# .venv\Scripts\activate     # Windows

# 安装依赖
uv add "mcp[cli]"
```

**方式 B：使用 pip + venv**

```bash
mkdir my-mcp-server
cd my-mcp-server

# 创建虚拟环境
python3 -m venv .venv
source .venv/bin/activate

# 安装依赖
pip install "mcp[cli]"
```

**依赖说明**：

| 包名 | 作用 |
|------|------|
| `mcp` | MCP Python SDK 核心包，提供 FastMCP 类、传输层、协议实现 |
| `mcp[cli]` | 额外安装 CLI 工具（`mcp dev` 调试命令、`mcp install` 安装命令） |

> **为什么用 `mcp[cli]` 而不是 `mcp`？**  
> `[cli]` 是 Python 的 extras 语法，会额外安装 CLI 调试工具。开发阶段非常有用——可以用 `mcp dev` 命令在浏览器中测试你的 Server，不需要每次都重启 Claude Desktop。

### 3.2 创建主文件

```bash
touch server.py
```

此时项目结构：

```
my-mcp-server/
├── .venv/              ← 虚拟环境（不要提交到 git）
├── server.py           ← 主文件，我们的 Server 代码写在这里
├── pyproject.toml      ← 项目配置（uv init 生成）
└── ...
```

---

## 4. 编写 MCP Server

下面我们在 `server.py` 中编写完整的 MCP Server。代码拆成几个部分，逐段解释。

### 4.1 导入与初始化

```python
# ---- 导入 MCP SDK ----

# FastMCP 是 Python SDK 提供的高级 API
# 它利用 Python 的类型注解和 docstring 自动生成工具定义
# 相比底层 API，FastMCP 让你写更少的代码
from mcp.server.fastmcp import FastMCP

# ---- 创建 Server 实例 ----

# name 参数：Server 名称，会在 Client 连接时显示
# 这个名称会出现在 Claude Desktop 的工具列表中
mcp = FastMCP("my-notes-server")

# ---- 数据存储 ----
# 这里用字典模拟数据库，实际项目中替换为真实的数据库
notes_store: dict[str, str] = {}
```

> **为什么用 FastMCP 而不是底层的 Server 类？**  
> FastMCP 是 Python SDK 的"语法糖"。底层 Server 类需要你手动定义 JSON Schema、注册 handler、处理序列化。FastMCP 通过装饰器 + 类型注解自动完成这些工作，代码量减少 60-70%。这是 Python SDK 相比 TypeScript SDK 的一大优势。

### 4.2 注册工具（Tool）

这是 MCP Server 的核心——定义 LLM 可以调用的工具。

```python
# ---- 工具 1：添加笔记 ----

# @mcp.tool() 装饰器将一个普通函数注册为 MCP 工具
# FastMCP 会自动做以下事情：
#   1. 从函数名生成工具名称（add_note）
#   2. 从 docstring 生成工具描述（LLM 根据描述决定何时调用）
#   3. 从类型注解生成参数的 JSON Schema（LLM 根据 Schema 生成参数）
#   4. 运行时自动验证参数类型
@mcp.tool()
def add_note(title: str, content: str) -> str:
    """Add a new note with a title and content.

    Use this when the user wants to save or remember information.

    Args:
        title: The title of the note, used as unique identifier.
        content: The content/body of the note.
    """
    # 函数体就是工具的执行逻辑
    # 直接返回字符串即可，FastMCP 会自动包装成 MCP 协议格式
    notes_store[title] = content
    return f'Note "{title}" has been saved successfully.'
```

> **为什么 docstring 这么重要？**  
> FastMCP 会把 docstring 作为工具的 `description` 传给 LLM。LLM 不会看你的代码实现，它只能看到工具名、描述和参数定义。docstring 写得好不好，直接决定了 LLM 能否在正确的时机调用正确的工具。
>
> **最佳实践**：
> - 第一行写功能概述
> - 第二行写使用场景（"Use this when..."），帮助 LLM 判断何时调用
> - Args 部分描述每个参数的含义

> **为什么类型注解是必须的？**  
> FastMCP 依赖类型注解来生成 JSON Schema。没有类型注解，LLM 就不知道参数应该是什么类型，可能传入错误的值。`title: str` 会被转换为 `{"type": "string"}`。

```python
# ---- 工具 2：获取笔记 ----

@mcp.tool()
def get_note(title: str) -> str:
    """Retrieve a note by its title.

    Use this when the user asks about a previously saved note.

    Args:
        title: The title of the note to retrieve.
    """
    content = notes_store.get(title)

    if content is None:
        # 工具执行"失败"时，不要抛异常
        # 而是返回清晰的错误信息，让 LLM 理解发生了什么并告知用户
        available = ", ".join(notes_store.keys()) if notes_store else "none"
        return f'Note "{title}" not found. Available notes: {available}'

    return f"# {title}\n\n{content}"
```

> **为什么不抛异常？**  
> 如果工具函数抛出未捕获的异常，MCP SDK 会返回一个通用错误消息给 LLM，LLM 无法理解具体发生了什么。返回描述性的错误信息，LLM 可以据此给用户有用的反馈（比如"没找到这个笔记，你是不是想找 XXX？"）。

```python
# ---- 工具 3：列出所有笔记 ----

@mcp.tool()
def list_notes() -> str:
    """List all saved notes.

    Use this when the user wants to see what notes are available.
    """
    # 无参数的工具也完全可以
    # FastMCP 会生成一个空的 inputSchema
    if not notes_store:
        return "No notes saved yet."

    lines = [
        f"- **{title}**: {content[:50]}..."
        for title, content in notes_store.items()
    ]
    return f"Saved notes:\n\n" + "\n".join(lines)
```

### 4.3 启动 Server

```python
# ---- 启动入口 ----

if __name__ == "__main__":
    # mcp.run() 启动 Server 并开始监听
    #
    # transport="stdio" 表示使用标准输入/输出通信：
    #   - Client（如 Claude Desktop）启动 Server 作为子进程
    #   - Client 通过子进程的 stdin 发送请求（JSON-RPC 格式）
    #   - Server 通过 stdout 返回响应
    #   - 简单、安全、不需要网络配置
    #
    # 为什么用 stdio 而不是 HTTP？
    #   - 本地开发最简单的方式，不需要配置端口和网络
    #   - Claude Desktop 原生支持 stdio 方式启动 MCP Server
    #   - 安全：不暴露网络端口，只有启动它的 Client 能访问
    mcp.run(transport="stdio")
```

> **关于日志的重要警告**：
>
> ```python
> import sys
>
> # ❌ 错误：print() 默认写入 stdout，会破坏 JSON-RPC 通信
> print("Processing request")
>
> # ✅ 正确：写入 stderr
> print("Processing request", file=sys.stderr)
>
> # ✅ 正确：使用 logging 模块（默认写入 stderr）
> import logging
> logging.info("Processing request")
> ```
>
> **为什么 `print()` 会导致 Server 崩溃？**  
> stdio 传输模式下，stdout 是 MCP 的 JSON-RPC 通信通道。`print()` 默认写入 stdout，会在 JSON-RPC 消息流中插入非法内容，导致 Client 解析失败、连接断开。**所有调试输出必须写入 stderr**。

---

## 5. 运行与测试

### 5.1 直接运行

```bash
# 使用 uv
uv run server.py

# 或者激活虚拟环境后直接运行
python server.py
```

Server 启动后会等待 stdin 输入，看起来像"卡住了"——这是正常的。

> **为什么看起来卡住了？**  
> Server 在等待 Client 通过 stdin 发送 JSON-RPC 消息。在没有 Client 连接的情况下，它就静静地等着。按 `Ctrl+C` 退出。

### 5.2 使用 MCP Inspector 调试（推荐）

```bash
# MCP Inspector 是一个浏览器端的调试工具
# 它会启动你的 Server 并提供一个 Web 界面来测试工具
mcp dev server.py
```

> **为什么推荐用 Inspector？**  
> 每次修改代码后重启 Claude Desktop 太慢了。MCP Inspector 提供了一个独立的 Web 界面，你可以：
> - 查看 Server 注册了哪些工具/资源/提示词
> - 手动调用工具并查看返回结果
> - 查看 JSON-RPC 通信日志
> - 快速迭代开发

Inspector 启动后会打印一个 URL（通常是 `http://localhost:5173`），在浏览器中打开即可。

### 5.3 使用 MCP CLI 安装到 Claude Desktop

```bash
# 自动配置 Claude Desktop 的 config 文件
mcp install server.py
```

> **为什么有这个命令？**  
> 手动编辑 `claude_desktop_config.json` 容易出错（路径写错、JSON 格式错误等）。`mcp install` 自动检测项目路径并写入正确的配置。

---

## 6. 接入 Claude Desktop 测试

### 6.1 手动配置（如果没用 `mcp install`）

获取项目绝对路径：

```bash
pwd
# 输出类似：/Users/yourname/projects/my-mcp-server
```

> **为什么必须用绝对路径？**  
> Claude Desktop 启动 Server 子进程时，工作目录不确定。相对路径可能解析到错误的位置。

打开 Claude Desktop 配置文件：

```bash
# macOS
code ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

添加配置：

```json
{
  "mcpServers": {
    "my-notes": {
      "command": "uv",
      "args": [
        "--directory",
        "/Users/yourname/projects/my-mcp-server",
        "run",
        "server.py"
      ]
    }
  }
}
```

**配置字段说明**：

| 字段 | 说明 |
|------|------|
| `"my-notes"` | Server 的显示名称 |
| `"command": "uv"` | 启动命令。用 `uv` 而不是 `python`，因为 `uv run` 会自动处理虚拟环境 |
| `"--directory"` | 指定项目目录，`uv` 会在这个目录下找 `pyproject.toml` 和虚拟环境 |
| `"run", "server.py"` | 在项目虚拟环境中运行 `server.py` |

> **为什么 command 是 `uv` 而不是 `python`？**  
> 如果用 `python`，你需要指定虚拟环境中 python 的绝对路径（如 `/Users/.../my-mcp-server/.venv/bin/python`），很容易出错。`uv --directory <path> run` 会自动找到项目的虚拟环境并在其中运行，更可靠。
>
> 如果你用 pip 而不是 uv，配置改为：
> ```json
> {
>   "command": "/Users/yourname/projects/my-mcp-server/.venv/bin/python",
>   "args": ["/Users/yourname/projects/my-mcp-server/server.py"]
> }
> ```

### 6.2 重启 Claude Desktop

**必须完全退出再重新打开**：
- macOS：`Cmd + Q` 或菜单栏 → Quit Claude
- Windows：系统托盘右键 → Quit

> **为什么要完全退出？**  
> Claude Desktop 只在启动时读取配置文件。关闭窗口不会退出进程（它驻留在系统托盘），配置不会生效。

### 6.3 验证

打开 Claude Desktop，在输入框旁找到工具图标，应该能看到 `my-notes` Server。

试试对话：
- "帮我保存一条笔记，标题是'MCP学习'，内容是'今天学会了用 Python 搭建 MCP Server'"
- "我保存了哪些笔记？"
- "查看'MCP学习'这条笔记"

---

## 7. 进阶：添加 Resource 和 Prompt

### 7.1 添加 Resource

Resource 让 Client 可以读取 Server 提供的数据：

```python
# @mcp.resource() 装饰器注册一个资源
# 参数是资源的 URI，Client 通过这个 URI 来读取
# URI 格式自定义，建议用 "协议://路径" 的形式
@mcp.resource("notes://all")
def get_all_notes() -> str:
    """A list of all saved notes.

    Returns all notes in a readable format.
    """
    # 直接返回字符串，FastMCP 自动包装成 MCP 协议格式
    if not notes_store:
        return "No notes yet."

    return "\n\n".join(
        f"{title}: {content}"
        for title, content in notes_store.items()
    )
```

> **Tool 和 Resource 的区别？**  
> - **Tool**：LLM 主动调用的"动作"，有输入参数，可能产生副作用（写数据）  
> - **Resource**：Client 主动读取的"数据"，类似 GET 请求，只读不写  
> - 实际场景：Tool 用于"添加笔记"、"删除笔记"；Resource 用于"查看笔记列表"

### 7.2 添加动态 Resource（带参数）

```python
# URI 中的 {title} 是动态参数
# Client 请求 notes://note/MCP学习 时，title 参数会被自动提取为 "MCP学习"
@mcp.resource("notes://note/{title}")
def get_note_resource(title: str) -> str:
    """Get a specific note by title.

    Args:
        title: The title of the note.
    """
    content = notes_store.get(title)
    if content is None:
        return f'Note "{title}" not found.'
    return f"# {title}\n\n{content}"
```

> **为什么需要动态 Resource？**  
> 静态 URI（如 `notes://all`）只能返回固定内容。动态 URI 让一个 Resource 定义服务多个具体资源，就像 REST API 中的 `/users/:id`。

### 7.3 添加 Prompt

```python
# @mcp.prompt() 装饰器注册一个提示词模板
# Prompt 是预定义的消息序列，Client 可以一键触发
@mcp.prompt()
def summarize_notes() -> str:
    """Generate a summary of all saved notes.

    Creates a prompt that asks the LLM to summarize all notes.
    """
    all_notes = "\n\n".join(
        f"## {title}\n{content}"
        for title, content in notes_store.items()
    )

    if not all_notes:
        all_notes = "No notes available."

    return f"Please summarize the following notes:\n\n{all_notes}"
```

> **为什么需要 Prompt？**  
> 有些常用的交互模式可以封装成模板，用户一键触发，不需要每次手动输入。比如"总结所有笔记"、"根据笔记生成周报"等。

### 7.4 带参数的 Prompt

```python
@mcp.prompt()
def write_about(topic: str) -> str:
    """Write content about a specific topic using saved notes as reference.

    Args:
        topic: The topic to write about.
    """
    relevant_notes = "\n\n".join(
        f"## {title}\n{content}"
        for title, content in notes_store.items()
        if topic.lower() in title.lower() or topic.lower() in content.lower()
    )

    if not relevant_notes:
        relevant_notes = "No relevant notes found."

    return (
        f"Based on the following reference notes, write a comprehensive article about '{topic}':\n\n"
        f"{relevant_notes}"
    )
```

---

## 8. 常见问题排查

### Server 没有出现在 Claude Desktop 中

1. **检查配置文件语法**：JSON 格式错误（多余逗号、缺少引号）会导致整个配置失效
2. **检查路径**：`--directory` 后面必须是绝对路径
3. **检查 uv 路径**：运行 `which uv` 获取 uv 的完整路径，必要时在 `command` 中使用完整路径
4. **完全重启**：必须 `Cmd+Q` 退出 Claude Desktop

### 工具调用失败

1. **查看日志**：
   ```bash
   # macOS
   tail -f ~/Library/Logs/Claude/mcp-server-my-notes.log
   tail -f ~/Library/Logs/Claude/mcp.log
   ```
2. **检查 print()**：确保代码中没有 `print()` 写入 stdout，改用 `print(..., file=sys.stderr)` 或 `logging`
3. **用 Inspector 测试**：`mcp dev server.py`，在浏览器中手动调用工具看返回值

### Python 环境问题

| 问题 | 原因 | 解决 |
|------|------|------|
| `ModuleNotFoundError: No module named 'mcp'` | 没在虚拟环境中运行 | 激活虚拟环境或用 `uv run` |
| `SyntaxError` | Python 版本低于 3.10 | 升级 Python |
| `uv: command not found` | uv 未安装或未加入 PATH | 重新安装 uv 并重启终端 |

### 调试技巧

```python
import sys
import logging

# 配置日志输出到 stderr（不会干扰 MCP 通信）
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    stream=sys.stderr,  # 关键：输出到 stderr
)

logger = logging.getLogger("my-mcp-server")

@mcp.tool()
def add_note(title: str, content: str) -> str:
    """Add a new note."""
    logger.debug(f"Adding note: title={title}, content_length={len(content)}")
    notes_store[title] = content
    logger.info(f"Note '{title}' saved successfully")
    return f'Note "{title}" has been saved successfully.'
```

---

## 完整代码参考

最终的 `server.py` 完整代码：

```python
import sys
import logging
from mcp.server.fastmcp import FastMCP

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    stream=sys.stderr,
)
logger = logging.getLogger("my-notes-server")

mcp = FastMCP("my-notes-server")

notes_store: dict[str, str] = {}


@mcp.tool()
def add_note(title: str, content: str) -> str:
    """Add a new note with a title and content.

    Use this when the user wants to save or remember information.

    Args:
        title: The title of the note, used as unique identifier.
        content: The content/body of the note.
    """
    notes_store[title] = content
    logger.info(f"Note '{title}' saved")
    return f'Note "{title}" has been saved successfully.'


@mcp.tool()
def get_note(title: str) -> str:
    """Retrieve a note by its title.

    Use this when the user asks about a previously saved note.

    Args:
        title: The title of the note to retrieve.
    """
    content = notes_store.get(title)
    if content is None:
        available = ", ".join(notes_store.keys()) if notes_store else "none"
        return f'Note "{title}" not found. Available notes: {available}'
    return f"# {title}\n\n{content}"


@mcp.tool()
def list_notes() -> str:
    """List all saved notes.

    Use this when the user wants to see what notes are available.
    """
    if not notes_store:
        return "No notes saved yet."
    lines = [
        f"- **{title}**: {content[:50]}..."
        for title, content in notes_store.items()
    ]
    return "Saved notes:\n\n" + "\n".join(lines)


@mcp.resource("notes://all")
def get_all_notes() -> str:
    """A list of all saved notes."""
    if not notes_store:
        return "No notes yet."
    return "\n\n".join(
        f"{title}: {content}" for title, content in notes_store.items()
    )


@mcp.resource("notes://note/{title}")
def get_note_resource(title: str) -> str:
    """Get a specific note by title."""
    content = notes_store.get(title)
    if content is None:
        return f'Note "{title}" not found.'
    return f"# {title}\n\n{content}"


@mcp.prompt()
def summarize_notes() -> str:
    """Generate a summary of all saved notes."""
    all_notes = "\n\n".join(
        f"## {title}\n{content}" for title, content in notes_store.items()
    )
    if not all_notes:
        all_notes = "No notes available."
    return f"Please summarize the following notes:\n\n{all_notes}"


if __name__ == "__main__":
    mcp.run(transport="stdio")
```

---

## Python vs TypeScript 对比

| 维度 | Python (FastMCP) | TypeScript (McpServer) |
|------|-----------------|----------------------|
| 工具定义方式 | `@mcp.tool()` 装饰器 + 类型注解 + docstring | `server.registerTool()` + Zod Schema |
| 代码量 | 更少（装饰器自动推导） | 稍多（需手写 Schema） |
| 类型安全 | 运行时验证 | 编译时 + 运行时双重验证 |
| 调试工具 | `mcp dev`（内置） | 需手动配置 |
| 生态 | 数据科学/AI 生态更丰富 | Web/全栈生态更丰富 |
| 适合场景 | 数据处理、AI 管线、快速原型 | Web 服务、前端工具链、生产部署 |

---

## 下一步

- 把内存中的字典替换为真实的数据库（SQLite / PostgreSQL）
- 添加更多工具（搜索笔记、删除笔记、导出笔记）
- 尝试 SSE 传输，让 Server 可以远程访问
- 用 `mcp dev` 调试更复杂的工具交互
- 阅读 [MCP 官方文档](https://modelcontextprotocol.io/) 了解更多协议细节
