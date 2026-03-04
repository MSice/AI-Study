# 阶段 1：LLM 基础认知（第 1-2 周）

**学习目标**：理解大语言模型的基本原理，能说清楚 LLM 是什么、怎么工作的、能做什么和不能做什么。

**前置要求**：无（本阶段是起点）

> 📚 本阶段配套阅读：[推荐资源深度总结](./resources-digest.md) — 将本阶段所有推荐资源的核心内容提炼为一篇通俗易懂的中文解读，帮助你快速建立全局认知。

---

## 知识模块

### 1.1 什么是大语言模型（LLM）

- **是什么**：LLM 就像一个读过互联网上几乎所有文章的超级学霸——它不是真的"理解"内容，而是学会了"下一个词最可能是什么"的统计规律。你给它开个头，它就能接着往下写。

- **为什么重要**：LLM 是当前 AI 应用的核心引擎。理解它的本质，才能知道什么任务适合交给它，什么任务不适合，避免在开发中踩坑。

- **核心概念**：
  - **Token（词元）**：把文字切成小积木块。LLM 不是一个字一个字读的，而是把文本切成 Token 来处理。"你好世界" 可能被切成 ["你好", "世界"] 两个 Token。
  - **参数量**：模型的"脑容量"。GPT-4 有上万亿参数，参数越多通常越聪明，但也越慢越贵。
  - **上下文窗口（Context Window）**：模型一次能"看到"的文本长度。就像工作记忆——窗口越大，能处理的信息越多。
  - **推理（Inference）**：模型根据输入生成输出的过程。就像考试时学生根据题目写答案。
  - **幻觉（Hallucination）**：模型一本正经地胡说八道。因为它是基于概率生成文本，有时会编造看起来合理但实际错误的内容。

- **动手练习**：
  1. 打开 [OpenAI Tokenizer](https://platform.openai.com/tokenizer)，输入中英文句子，观察 Token 切分结果
  2. 用同一个问题分别问 GPT-4o、Claude、Llama 3，对比回答差异
  3. 故意问一个冷门事实，观察模型是否出现幻觉

- **推荐资源**：
  - 📖 必读：[What Is ChatGPT Doing … and Why Does It Work?](https://writings.stephenwolfram.com/2023/02/what-is-chatgpt-doing-and-why-does-it-work/) — Stephen Wolfram 的经典长文，用直觉解释 LLM 原理（英文）
  - 🎬 推荐：[3Blue1Brown - But what is a GPT?](https://www.youtube.com/watch?v=wjZofJX0v4M) — 可视化讲解 Transformer 架构（英文，有中文字幕）
  - 🛠️ 工具：OpenAI Tokenizer、ChatGPT / Claude 对话界面
  - 💡 **中文解读**：[推荐资源深度总结 - 1.1 节](./resources-digest.md#11-stephen-wolfram-长文chatgpt-到底在做什么)

- **检验标准**：
  - LLM 生成文本的基本原理是什么？（提示：下一个 Token 预测）
  - 为什么 LLM 会产生幻觉？如何缓解？
  - Token 数量和费用/速度之间是什么关系？

---

### 1.2 主流 LLM 模型对比

- **是什么**：就像汽车有不同品牌和型号，LLM 也有很多"厂商"和"车型"。选对模型就像选对工具——杀鸡不用牛刀，但也别拿小刀砍大树。

- **为什么重要**：不同模型在能力、价格、速度、隐私方面差异巨大。实际开发中需要根据场景选择最合适的模型。

- **核心概念**：
  - **闭源模型 vs 开源模型**：闭源（GPT-4o、Claude）通常更强但要付费且数据上云；开源（Llama、Qwen）可本地部署，数据不出门。
  - **模型系列**：OpenAI（GPT 系列）、Anthropic（Claude 系列）、Meta（Llama 系列）、Google（Gemini 系列）、阿里（Qwen/通义千问系列）、DeepSeek 等。
  - **模型大小与选择**：同一系列通常有大/中/小多个版本。小模型快且便宜，大模型强但贵。
  - **API 调用 vs 本地部署**：API 调用简单但依赖网络和费用；本地部署需要 GPU 但数据安全。

- **动手练习**：
  1. 安装 [Ollama](https://ollama.com/)，在本地运行 Qwen2.5（7B）或 Llama 3（8B）
  2. 用同一组测试问题（翻译、代码生成、逻辑推理、创意写作）对比本地模型和 GPT-4o 的表现
  3. 记录每个模型的响应速度和质量差异

- **推荐资源**：
  - 📖 必读：[LMSYS Chatbot Arena Leaderboard](https://chat.lmsys.org/?leaderboard) — 实时更新的 LLM 排行榜，基于人类投票（英文）
  - 🎬 推荐：[Ollama 官方文档](https://github.com/ollama/ollama) — 本地运行开源模型的最简方式
  - 🛠️ 工具：Ollama、LM Studio
  - 💡 **中文解读**：[推荐资源深度总结 - 1.2 节](./resources-digest.md#12-lmsys-chatbot-arena模型排行榜怎么看)

- **检验标准**：（✅ 已整理回答 → [1.2 检验标准回答](./笔记/1.2-检验标准回答.md)）
  - 能说出至少 5 个主流 LLM 及其特点
  - 什么场景该用闭源 API？什么场景该用本地开源模型？
  - 如何在本地用 Ollama 运行一个开源模型？

---

### 1.3 LLM API 基础调用

- **是什么**：API 就像餐厅的点菜窗口——你把"菜单"（提示词）递进去，厨房（模型）做好后把"菜"（回答）递出来。学会调 API 是用代码驱动 LLM 的第一步。

- **为什么重要**：所有 LLM 应用的底层都是 API 调用。掌握 API 调用是从"聊天用户"进化为"AI 开发者"的关键一步。

- **核心概念**：
  - **Chat Completions API**：最常用的 API 格式，发送一组对话消息，返回模型的回复。
  - **System / User / Assistant 消息角色**：System 设定模型人设，User 是用户输入，Assistant 是模型回复。三者构成完整对话。
  - **Temperature（温度）**：控制输出的"创造力"。温度低（0）= 确定性高，适合事实类任务；温度高（1）= 更随机，适合创意写作。
  - **Streaming（流式输出）**：让模型一个字一个字地返回，而不是等全部生成完。用户体验更好，就像打字机效果。

- **动手练习**：
  1. 用 Python 的 `openai` 库调用 GPT-4o-mini 的 Chat Completions API
  2. 尝试修改 `temperature`、`max_tokens`、`system` 消息，观察输出变化
  3. 实现一个流式输出的命令行聊天机器人（20 行代码以内）

- **推荐资源**：
  - 📖 必读：[OpenAI API 官方文档 - Chat Completions](https://platform.openai.com/docs/guides/text-generation) — 最权威的 API 使用指南（英文）
  - 🎬 推荐：[Anthropic Claude API 文档](https://docs.anthropic.com/en/docs/build-with-claude/text-generation) — Claude 的 API 调用方式（英文）
  - 🛠️ 工具：Python `openai` 库、`anthropic` 库、Postman / curl
  - 💡 **中文解读**：[推荐资源深度总结 - 1.3 节](./resources-digest.md#13-openai-api-文档chat-completions-核心要点)

- **检验标准**：
  - 能用代码调用 OpenAI 或 Claude 的 API 并获取回复
  - Temperature 参数对输出有什么影响？
  - System 消息的作用是什么？给一个实际使用场景

---

## 阶段项目：LLM 能力探测器

**项目描述**：开发一个 Python 脚本，自动用一组标准化测试题（翻译、摘要、代码生成、数学推理、创意写作 5 个类别，每类 2 题）测试不同 LLM，并生成对比报告。

**技术要求**：
- 使用 OpenAI API 和 Ollama 本地 API
- 至少对比 2 个模型（如 GPT-4o-mini vs Qwen2.5-7B）
- 记录每次调用的响应时间和 Token 用量
- 输出 Markdown 格式的对比报告

**预期产出**：
- `llm_benchmark.py` — 测试脚本
- `report.md` — 自动生成的对比报告

**预计耗时**：3-4 小时

---

## 阶段检查点

- [ ] 能用自己的话解释 LLM 的工作原理（下一个 Token 预测）
- [ ] 能说出 5 个主流 LLM 模型及各自特点
- [ ] 已在本地用 Ollama 成功运行一个开源模型
- [ ] 能用 Python 调用 OpenAI / Claude API 并获取回复
- [ ] 理解 Temperature、Token、Context Window 等核心概念
- [ ] 完成阶段项目：LLM 能力探测器

---

**下一步** → [阶段 2：提示词工程](../02-prompt-engineering/index.md)
