# MCP 核心概念与调用方式

> 整理自对 MCP（Model Context Protocol）的学习讨论，涵盖核心概念、调用原理和实际设计决策。

---

## 一、MCP Server vs 普通 HTTP Server

| 维度 | MCP Server | 普通 HTTP Server |
|------|-----------|-----------------|
| **目标消费者** | AI Agent / LLM 应用 | 浏览器、移动端、其他服务 |
| **通信协议** | JSON-RPC 2.0（stdio 或 SSE） | HTTP REST / GraphQL |
| **暴露的是什么** | 语义化能力（Tool / Resource / Prompt） | API 端点（GET / POST ...） |
| **自描述性** | 内置，AI 连接后自动发现所有能力 | 需要额外文档（Swagger/OpenAPI） |
| **交互模式** | 有生命周期管理、双向通信、能力协商 | 请求-响应，无状态 |

**本质**：MCP Server 是一个"面向 AI 的标准化能力提供层"，不替代 HTTP Server，而是在其之上加了一层 AI 友好的协议。

---

## 二、MCP Server 的调用方式

### 调用链路

```
用户提问 → MCP Client（Claude/Cursor） → MCP Server（你的工具）
```

完整流程：
1. `initialize`（握手，交换能力信息）
2. `initialized`（通知，准备完毕）
3. `tools/list`（发现可用工具）
4. `tools/call`（调用工具，LLM 自主决定）
5. 返回结果 → LLM 组织答案 → 回复用户

### 两种传输方式

| 方式 | 原理 | 适用场景 |
|------|------|----------|
| **stdio** | Client 把 Server 作为子进程启动，通过 stdin/stdout 传 JSON | 本地开发，Claude Desktop / Cursor 默认方式 |
| **SSE + HTTP** | Server 监听 HTTP 端口，Client 通过网络连接 | 远程部署，多 Client 共享 |

### 本地模拟调用（3 种方式）

- **echo + 管道**：直接向 stdin 发 JSON-RPC 消息，理解底层协议格式
- **subprocess + JSON-RPC**：用 Python 脚本启动子进程，手动发送完整的握手和调用流程
- **MCP SDK Client API**：用 `ClientSession` + `stdio_client`，SDK 封装好了所有细节，推荐生产使用

---

## 三、Tool / Resource / Prompt 三种能力

### 一句话区分

- **Tool** = AI 的"手"，LLM 自主决定调用，执行动作
- **Resource** = AI 的"背景资料"，用户/Client 提前准备，注入上下文
- **Prompt** = AI 的"剧本"，用户手动触发的预设工作流模板

### 对比表

| 维度 | Tool | Resource | Prompt |
|------|------|----------|--------|
| 谁发起 | LLM 自主发起 | 用户在 UI 上手动选择 | 用户手动触发 |
| 谁决策 | LLM 判断要不要调 | 用户/Client 直接指定 | 用户选择模板 |
| 做什么 | 执行动作（可能有副作用） | 读取数据（只读） | 填充提示词模板 |
| 何时发生 | 对话过程中，按需 | 对话开始前或旁路加载 | 对话开始时触发 |
| 类比 | 对助理说"帮我查个东西" | 开会前发的"背景材料" | 给助理一份"工作手册" |

### Tool vs Resource 的判断标准

> 核心问题：**这个数据是谁决定要获取的？**

- **LLM 在对话中自己决定要获取** → Tool
- **用户/Client 在 UI 上直接指定** → Resource

示例：
- 用户说"帮我查一下 zhangsan 的信息" → **Tool**（LLM 推理后决定调什么接口）
- 用户在 Cursor 里输入 `@某个文件` → **Resource**（用户直接指定，LLM 不参与决策）

**实际建议**：拿不准的时候先做 Tool。Tool 能覆盖 Resource 的功能，反过来不行。

---

## 四、平台 OpenAPI 转 MCP Server 的设计建议

1. **按业务域拆分**，每个域一个 MCP Server，每个 Server 控制在 10-20 个 Tool
2. **写操作和需要判断的查询** → Tool
3. **静态/半静态数据（文档、配置、报表）** → Resource
4. **高频复合操作流程（排查、分析、生成）** → Prompt
5. **不是所有 API 都要暴露**，只暴露 AI 对话中真正需要的
6. **Tool 的描述比实现更重要**，LLM 根据描述决定是否调用
