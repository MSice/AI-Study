# 阶段 3 推荐资源深度总结

> 本文将 [阶段 3：AI Agent](./index.md) 中推荐的核心资源提炼为一篇通俗易懂的中文解读。
> 目标：即使你不阅读原始英文资源，也能建立对 AI Agent 技术的完整理解。

---

## 目录

- [3.1 Lilian Weng：LLM 驱动的自主 Agent](#31-lilian-wengllm-驱动的自主-agent)
- [3.2 ReAct 原始论文：推理与行动的协同](#32-react-原始论文推理与行动的协同)
- [3.3 OpenAI Function Calling 文档](#33-openai-function-calling-文档)
- [3.4 Anthropic Tool Use 文档](#34-anthropic-tool-use-文档)
- [3.5 LangGraph 官方教程](#35-langgraph-官方教程)
- [3.6 LangChain 官方文档](#36-langchain-官方文档)
- [总结：串联所有知识点](#总结串联所有知识点)

---

## 3.1 Lilian Weng：LLM 驱动的自主 Agent

> 原文：[LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)（Lilian Weng，2023）

Lilian Weng 是 OpenAI 的研究科学家，这篇博客是理解 AI Agent 架构最经典的综述之一。她用清晰的框架把 Agent 系统的核心组件拆解开来，并介绍了大量前沿研究和实践案例。

### 核心观点一：Agent 系统的整体架构

一个由大语言模型（LLM）驱动的自主 Agent 系统，可以理解为：**LLM 是 Agent 的"大脑"，再配上几个关键组件**：

| 组件 | 作用 | 通俗理解 |
|------|------|---------|
| **规划（Planning）** | 把大任务拆成小步骤，自我反思纠错 | 像项目经理做任务分解和复盘 |
| **记忆（Memory）** | 短期记住当前对话，长期存储历史信息 | 像人的工作记忆和长期记忆 |
| **工具使用（Tool Use）** | 调用外部 API 获取模型不知道的信息 | 像人用计算器、搜索引擎、数据库 |

这三个组件缺一不可，它们让 LLM 从"只会聊天"进化为"能独立干活"的 Agent。

### 核心观点二：规划——任务分解与自我反思

**任务分解**：复杂任务通常需要很多步骤，Agent 需要提前想清楚。常见做法包括：

- **思维链（Chain of Thought，CoT）**：让模型"一步一步想"，把大任务变成多个可管理的小任务
- **思维树（Tree of Thoughts）**：在每一步探索多种推理可能性，形成树状结构，用广度优先或深度优先搜索
- **LLM+P**：把规划外包给传统规划器，用 PDDL（规划领域定义语言）作为中间接口

**自我反思**：Agent 能从过去的错误中学习，改进后续行动。相关技术包括：

- **ReAct**：把"推理"和"行动"交织在一起，模型先想再干，干了再看结果，再想下一步
- **Reflexion**：给 Agent 动态记忆和反思能力，当发现轨迹低效或有幻觉时，可以重置环境重新尝试
- ** hindsight 链（Chain of Hindsight）**：给模型展示一系列带反馈的过去输出，训练它根据反馈逐步改进

### 核心观点三：记忆——短期、长期与向量检索

文章用人类记忆的分类来类比 Agent 的记忆：

| 人类记忆类型 | Agent 对应 | 说明 |
|-------------|-----------|------|
| **感觉记忆** | 输入的嵌入表示 | 对原始文本、图像等的编码 |
| **短期记忆** | 上下文学习（In-Context Learning） | 受 Transformer 上下文窗口长度限制，通常几千到几万 Token |
| **长期记忆** | 外部向量数据库 | 可存储海量信息，通过检索在需要时召回 |

**向量检索（MIPS）**：长期记忆通常用向量数据库实现，支持"最大内积搜索"（Maximum Inner Product Search），即给定一个查询向量，快速找到最相似的 top-k 个向量。常用算法包括：

- **ScaNN**：Google 的向量量化算法
- **FAISS**：Facebook 的相似度搜索库
- **HNSW**：层次化可导航小世界图，灵感来自"六度分隔"理论
- **ANNOY**：基于随机投影树的近似最近邻
- **LSH**：局部敏感哈希

### 核心观点四：工具使用——Agent 的"手和脚"

工具使用让 LLM 能调用外部 API，获取模型权重中不存在的信息，例如：实时天气、代码执行、专有数据库等。

**MRKL 架构**：把系统设计成"专家模块 + LLM 路由器"，LLM 负责把请求路由到最合适的专家（可以是计算器、天气 API、货币转换器等）。

**实践案例**：
- **ChatGPT 插件** 和 **OpenAI Function Calling**：工具调用在实际产品中的典型应用
- **HuggingGPT**：用 ChatGPT 作为任务规划器，从 HuggingFace 选模型、执行任务、汇总结果
- **API-Bank**：评估工具增强型 LLM 的基准，包含 53 个常用 API、264 个标注对话、568 次 API 调用

### 核心观点五：案例研究——科学发现与生成式 Agent

**ChemCrow**：化学领域的 Agent，配备 13 个专家设计的工具，能完成有机合成、药物发现、材料设计等任务。遵循 ReAct 格式：Thought → Action → Action Input → Observation。有趣的是：GPT-4 自评认为和 ChemCrow 差不多，但人类专家评估显示 ChemCrow 明显更好——说明 LLM 在专业领域可能无法准确评估自己的表现。

**生成式 Agent（Generative Agents）**：25 个由 LLM 控制的虚拟角色生活在一个沙盒环境中（类似《模拟人生》）。每个 Agent 有记忆流、检索模型、反思机制和规划能力，能产生 emergent 社交行为，如信息传播、关系记忆、协调社交活动。

### 核心观点六：Agent 面临的挑战

1. **有限的上下文长度**：历史信息、详细指令、API 调用上下文都会占用空间，设计必须在这个限制下工作
2. **长期规划和任务分解的困难**：在长历史中做规划、有效探索解空间仍然很难；遇到意外错误时，LLM 调整计划的能力不如人类
3. **自然语言接口的可靠性**：模型输出可能有格式错误，偶尔"不听话"，所以很多 Agent 演示代码都在做输出解析

---

## 3.2 ReAct 原始论文：推理与行动的协同

> 原文：[ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)（Yao et al.，ICLR 2023）

ReAct 是当前最主流的 Agent 架构模式，这篇论文是它的学术来源。核心思想很简单：**让模型交替进行"推理"和"行动"，而不是只推理或只行动**。

### 核心观点一：人类智能的启示

论文指出，人类智能的一个独特之处是能无缝结合"任务导向的行动"和"语言推理"（内心独白）。比如在厨房做菜时，我们会在两个动作之间进行推理：
- 追踪进度："切好了，该烧水了"
- 处理意外："没盐了，用酱油和胡椒代替"
- 意识到需要外部信息："面团怎么和？搜一下"

这种"行动"与"推理"的紧密协同，让人能快速学习新任务，在未见过的情境下也能稳健决策。

### 核心观点二：ReAct 的提示模板

ReAct 的提示模板让模型按固定格式输出，大致如下：

```
Thought: （推理：我接下来要做什么，为什么）
Action: （行动：调用什么工具）
Observation: （观察：工具返回的结果）
... （重复多次，直到任务完成）
```

例如在 ALFWorld 环境中找胡椒瓶：
- Thought: 我需要找胡椒瓶，更可能出现在橱柜、柜台……
- Action: 去橱柜 1
- Observation: 在橱柜 1 你看到一个花瓶
- Thought: 橱柜 1 没有，去柜台 1、2、3，然后柜台 4、5
- Action: 去柜台 3
- Observation: 在柜台 3 你看到苹果、面包、胡椒瓶、花瓶
- Action: 从柜台 3 拿胡椒瓶 1
- Observation: 现在你找到了胡椒瓶 1。接下来需要把它放进抽屉 1
- ……

### 核心观点三：ReAct 相比其他方法的优势

**在知识密集型任务上**（如 HotpotQA 问答、FEVER 事实核查）：
- 纯思维链（Chain-of-Thought）容易产生幻觉和错误传播
- ReAct 通过调用 Wikipedia API 获取真实信息，克服了这些问题
- 生成的轨迹更像人类解题过程，可解释性更好

**在交互式决策任务上**（ALFWorld、WebShop）：
- ReAct 比模仿学习和强化学习方法分别高出 **34%** 和 **10%** 的绝对成功率
- 只需要 1～2 个上下文示例就能达到这样的效果

**对比"只行动不推理"（Act-only）**：在知识任务和决策任务上，去掉 Thought 步骤后效果都会变差，说明推理痕迹对模型诱导、跟踪、更新行动计划以及处理异常都很重要。

### 核心观点四：ReAct 的本质

ReAct 把模型的"行动空间"扩展为：**任务相关的离散动作 + 语言空间**。前者让 LLM 能与环境交互（如调用 Wikipedia 搜索 API），后者让 LLM 用自然语言生成推理痕迹。两者交织，形成"推理—行动—观察"的循环，直到任务完成。

---

## 3.3 OpenAI Function Calling 文档

> 原文：[OpenAI Function Calling 文档](https://platform.openai.com/docs/guides/function-calling)（OpenAI 官方）

Function Calling（函数调用，也叫工具调用）让 OpenAI 模型能够连接外部系统和数据，是构建 Agent 的必备能力。

### 核心概念：三个关键术语

| 术语 | 含义 | 举例 |
|------|------|------|
| **工具（Tools）** | 你提供给模型的功能 | 查天气、查账户、办理退款 |
| **工具调用（Tool calls）** | 模型决定要调用哪个工具、传什么参数 | 用户问"巴黎天气如何"，模型返回 `get_weather(location="Paris")` |
| **工具调用输出（Tool call outputs）** | 你的程序执行工具后返回给模型的结果 | `{"temperature": "25", "unit": "C"}` |

完整流程：用户提问 → 模型返回工具调用 → 你的程序执行工具 → 把结果返回模型 → 模型生成最终回答。

### 工具调用的五步流程

1. **向模型发起请求**，并附带可用工具列表
2. **收到模型的工具调用**（可能零个、一个或多个）
3. **在你的程序中执行**对应工具逻辑
4. **把执行结果**作为 `function_call_output` 发回模型
5. **收到模型的最终回复**（或更多工具调用，继续循环）

### 如何定义函数（Function Schema）

函数用 JSON Schema 描述，主要字段：

| 字段 | 说明 |
|------|------|
| `type` | 固定为 `"function"` |
| `name` | 函数名，如 `get_weather` |
| `description` | 详细说明何时、如何用这个函数 |
| `parameters` | JSON Schema，定义参数类型、是否必填等 |
| `strict` | 是否启用严格模式，保证输出符合 Schema |

示例：

```json
{
  "type": "function",
  "name": "get_weather",
  "description": "获取指定地点的当前天气",
  "parameters": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "城市和国家，如：北京，中国"
      },
      "units": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "温度单位"
      }
    },
    "required": ["location", "units"],
    "additionalProperties": false
  },
  "strict": true
}
```

### 定义函数的最佳实践

1. **写清楚**：函数名、参数描述、使用场景要具体，包含示例和边界情况
2. **用软件工程思维**：让新人只看描述就能正确用——"实习生测试"
3. **减轻模型负担**：总是成对调用的函数可以合并；已知的参数用代码传，别让模型填
4. **控制函数数量**：建议同时少于 20 个，数量太多会影响准确率
5. **善用工具**：在 Playground 里迭代 Schema；函数多或任务难时，可考虑微调

### 工具选择（tool_choice）参数

| 值 | 含义 |
|----|------|
| `"auto"` | 默认，模型自己决定是否调用、调用多少 |
| `"required"` | 必须调用至少一个工具 |
| `{"type": "function", "name": "get_weather"}` | 强制调用指定函数 |
| `"none"` | 不传工具时的等效行为 |

### 并行工具调用与严格模式

- 模型可能在一次回复中调用多个工具，可通过 `parallel_tool_calls: false` 限制为最多一个
- **严格模式**：`strict: true` 时，模型输出会严格符合 Schema；需要所有属性都标记 `required`，且 `additionalProperties: false`

### 流式输出

流式模式下，可以实时看到模型在调用哪个工具、参数如何逐步填充，适合做交互式体验。

### 自定义工具（Custom Tools）

除了 JSON Schema 定义的函数，还可以用**自定义工具**：模型传入任意字符串输入，你的工具返回任意格式。适合不想用 JSON 包裹、或需要自定义语法的场景。还支持用**上下文无关文法（CFG）**（Lark 或正则）约束模型的输入格式。

### Token 使用

函数定义会注入到系统消息中，占用上下文并计入输入 Token。函数多或描述长时，可能触及上下文上限，可考虑减少函数数量、缩短描述，或使用微调。

---

## 3.4 Anthropic Tool Use 文档

> 原文：[Anthropic Tool Use 文档](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview)（Anthropic 官方）

Claude 的 Tool Use（工具使用）让模型能调用你定义的工具，扩展其能力。与 OpenAI 的 Function Calling 概念类似，但实现细节和部分能力有差异。

### 核心定义

你定义工具及其参数，Claude 决定**何时**以及**如何**调用它们。工具调用在 LAB-Bench FigQA、SWE-bench 等基准上表现突出，甚至超过人类专家基线。

### 工具定义的三个核心属性

| 属性 | 说明 |
|------|------|
| **name** | 工具标识符，符合正则 `^[a-zA-Z0-9_-]{1,64}$` |
| **description** | 详细说明工具做什么、何时使用 |
| **input_schema** | JSON Schema 对象，定义参数 |

### 严格工具使用（Strict Tool Use）

在工具定义中加 `strict: true`，可以保证 Claude 的调用严格符合 Schema，适合生产环境。与 OpenAI 的 `strict` 模式类似。

### 程序化工具调用（Programmatic Tool Calling）

Claude 可以在代码执行环境中执行代码，由代码去调用工具。这样**多工具串联**时可以减少与 API 的往返次数，降低延迟和 Token 消耗。

### 工具搜索（Tool Search）

当工具数量达到几百或几千个时，把所有工具都放在上下文中会占用大量 Token。**工具搜索**可以按需动态发现和加载工具，在保持高选择准确率的同时，显著减少上下文占用（据称可超过 85%）。

### 最佳实践

- 复杂任务、模糊查询时，优先使用 **Claude Opus 4.6**
- 工具返回内容应精简，只包含必要信息
- 工具命名要有意义、可区分
- 把相关操作合并成更少、更强大的工具

---

## 3.5 LangGraph 官方教程

> 原文：[LangGraph 官方教程](https://langchain-ai.github.io/langgraph/tutorials/introduction/)（LangChain 团队）

LangGraph 是 LangChain 团队推出的**低层 Agent 编排框架**，用图（Graph）的方式定义 Agent 的工作流，适合需要精细控制、长期运行、有状态的 Agent。

### LangGraph 是什么？

LangGraph 专注于** Agent 编排**，不抽象提示词或架构，而是提供基础设施，支持：

- **长期运行**：Agent 可以持续运行很久
- **有状态**：能记住对话和中间状态
- **可调试**：与 LangSmith 集成，可视化执行路径和状态变化

### 安装与 Hello World

```bash
pip install -U langgraph
```

```python
from langgraph.graph import StateGraph, MessagesState, START, END

def mock_llm(state: MessagesState):
    return {"messages": [{"role": "ai", "content": "hello world"}]}

graph = StateGraph(MessagesState)
graph.add_node(mock_llm)
graph.add_edge(START, "mock_llm")
graph.add_edge("mock_llm", END)
graph = graph.compile()

graph.invoke({"messages": [{"role": "user", "content": "hi!"}]})
```

### 核心概念

- **StateGraph**：用节点和边定义工作流
- **MessagesState**：内置的消息状态类型，适合对话
- **START / END**：图的入口和出口
- **add_node**：添加节点（如 LLM、工具调用）
- **add_edge**：定义节点之间的连接

### 核心能力

| 能力 | 说明 |
|------|------|
| **生产级部署** | 支持长期、有状态工作流的扩展基础设施 |
| **LangSmith 调试** | 追踪执行路径、状态变化、运行时指标 |
| **记忆** | 短期工作记忆 + 跨会话的长期记忆 |
| **人机协作** | 在任意节点暂停，让人检查或修改状态 |
| **持久执行** | 失败后可恢复，从断点继续 |

### 与 LangChain 的关系

- LangGraph 可以**单独使用**，不依赖 LangChain
- 文档中常用 LangChain 的模型、工具等组件
- 若你刚接触 Agent 或想要更高层抽象，推荐先用 **LangChain 的 Agent**
- 若需要**确定性 + 智能体**混合工作流、或重度自定义，用 LangGraph

### 设计灵感

LangGraph 的图结构借鉴了 Pregel、Apache Beam，接口风格参考了 NetworkX。

---

## 3.6 LangChain 官方文档

> 原文：[LangChain 官方文档](https://python.langchain.com/docs/introduction/)（LangChain 团队）

LangChain 是当前最流行的 LLM 应用开发框架，提供预置的 Agent 架构和丰富的模型、工具集成，适合快速搭建 Agent 和应用。

### LangChain 是什么？

LangChain 是开源框架，让你用很少的代码就能构建自定义 Agent 和 LLM 应用。核心特点：

- **统一模型接口**：不同厂商（OpenAI、Anthropic、Google 等）的 API 格式不同，LangChain 统一封装，方便切换
- **开箱即用的 Agent**：内置 Agent 架构，几行代码就能跑起来
- **基于 LangGraph**：LangChain 的 Agent 底层是 LangGraph，因此具备持久执行、人机协作、持久化等能力

### 快速创建 Agent

```python
from langchain.agents import create_agent

def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

agent = create_agent(
    model="claude-sonnet-4-6",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)

agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
)
```

只需定义工具函数、选择模型和系统提示，即可构建一个带工具调用的 Agent。

### LangChain vs LangGraph vs Deep Agents

| 框架 | 适用场景 |
|------|---------|
| **Deep Agents** | 需要"开箱即用"：长对话压缩、虚拟文件系统、子 Agent 等 |
| **LangChain** | 快速搭建 Agent，兼顾易用和一定自定义 |
| **LangGraph** | 需要深度控制：确定性 + 智能体混合、可持久化、人机协作等 |

### 核心优势

- **标准模型接口**：统一调用方式，降低厂商锁定
- **易用且灵活**：入门简单，也支持深度定制
- **基于 LangGraph**：享受持久执行、人机协作、持久化等能力
- **LangSmith 调试**：追踪请求、评估输出、监控部署

---

## 总结：串联所有知识点

学完以上 6 个资源，你应该能理解这样一个完整的 Agent 认知框架：

```
AI Agent 技术栈
├── 概念与架构（Lilian Weng + ReAct）
│   ├── 三大组件：规划、记忆、工具使用
│   ├── ReAct 模式：Thought → Action → Observation 循环
│   └── 挑战：上下文限制、规划可靠性、自然语言接口
├── 工具调用实战（OpenAI + Anthropic）
│   ├── 定义工具：name、description、parameters（JSON Schema）
│   ├── 流程：用户请求 → 模型选择工具 → 执行 → 返回结果 → 模型生成回答
│   └── 差异：OpenAI 与 Anthropic 的 API 格式略有不同，但思路一致
└── 框架实战（LangGraph + LangChain）
    ├── LangChain：高层抽象，快速搭建 Agent
    ├── LangGraph：低层编排，精细控制工作流
    └── 选型：快速原型用 LangChain，复杂流程用 LangGraph
```

**关键认知**：

1. **Agent = LLM + 规划 + 记忆 + 工具**。LLM 是大脑，规划负责拆任务和反思，记忆负责短期和长期信息，工具负责与外部世界交互。

2. **ReAct 是当前最主流的 Agent 模式**。推理—行动—观察的循环，让模型既能"想"又能"干"，还能根据结果调整计划，比纯推理或纯行动都更有效。

3. **Function Calling / Tool Use 是 Agent 的"手"**。定义好工具 Schema，模型会自动选择、调用，你的程序负责执行并返回结果。这是连接 LLM 与外部系统的标准方式。

4. **框架是效率工具**。从零写 Agent 要处理循环、错误、记忆、并发等，LangChain 和 LangGraph 能大幅减少重复工作。但理解底层原理后，用原生 API 写一个最简 ReAct Agent（约 50 行）也是很好的练习。

5. **动手比看书重要**。建议顺序：先用 OpenAI 或 Anthropic 的 API 实现一个带 2～3 个工具的简单 Agent，再用 LangGraph 或 LangChain 构建同样的 Agent，对比两种方式的差异。

---

**返回** → [阶段 3：AI Agent](./index.md)
