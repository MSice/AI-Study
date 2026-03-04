---
name: git-workflow
description: 管理 AI-Study 仓库的 Git 工作流，包括提交规范、分支管理、中文路径处理和完整的 add/commit/push/PR 流程。当用户需要提交代码、创建分支、推送变更、处理中文文件名的 Git 操作时使用此技能。
---

# Git Workflow - AI-Study

## 仓库信息

- **远程仓库**: `https://github.com/MSice/AI-Study.git`
- **主分支**: `main`
- **特点**: 包含大量中文目录和文件名（如 `LLM学习计划/`、`笔记/`）

## 提交规范

采用 Conventional Commits 格式，commit message 使用中文描述：

```
<type>(<scope>): <简要描述>

<详细说明（可选）>
```

### Type 类型

| Type | 用途 |
|------|------|
| `docs` | 文档/笔记变更（最常用） |
| `feat` | 新增学习模块或功能 |
| `fix` | 修正内容错误 |
| `refactor` | 重组目录结构或内容 |
| `chore` | 配置文件、工具变更 |

### Scope 范围

使用模块简称作为 scope：

| Scope | 对应目录 |
|-------|---------|
| `llm-fundamentals` | `LLM学习计划/01-llm-fundamentals/` |
| `prompt` | `LLM学习计划/02-prompt-engineering/` |
| `agent` | `LLM学习计划/03-ai-agent/` |
| `rag` | `LLM学习计划/04-rag/` |
| `mcp` | `LLM学习计划/05-mcp/` |
| `app-dev` | `LLM学习计划/06-llm-app-dev/` |
| `advanced` | `LLM学习计划/07-advanced/` |
| `resources` | `LLM学习计划/08-resources-and-graduation/` |
| `plan` | `LLM学习计划/` 根目录 |
| `cursor` | `.cursor/` 配置 |

### 提交示例

```
docs(llm-fundamentals): 添加本地部署模型选择笔记

整理了 Ollama 与 LM Studio 的对比分析
```

```
refactor(plan): 重组学习计划目录结构

将各模块拆分为独立目录，便于管理笔记和资源
```

```
chore(cursor): 添加 Git 工作流技能配置
```

## 分支管理

### 命名规范

```
<type>/<简短描述>
```

示例：
- `docs/add-llm-notes` — 添加笔记
- `feat/add-rag-module` — 新增模块
- `refactor/restructure-plan` — 重组结构

### 工作流程

1. 日常笔记和小改动：直接在 `main` 分支提交
2. 较大的结构调整或新模块：创建功能分支 → PR → 合并

## 中文路径处理

此仓库包含中文目录名，Git 操作时需注意：

### 关键配置

```bash
# 确保 Git 正确显示中文文件名（不转义为八进制）
git config core.quotepath false
```

### 操作要点

- `git add` 中文路径时使用引号：`git add "LLM学习计划/"`
- `git status` 显示乱码时先执行上述配置
- 使用通配符时注意 shell 对中文的处理

## 完整工作流

### 日常提交流程

```bash
# 1. 查看变更
git status

# 2. 添加文件（中文路径加引号）
git add "LLM学习计划/01-llm-fundamentals/笔记/新笔记.md"
# 或添加整个目录
git add "LLM学习计划/"

# 3. 提交（遵循提交规范）
git commit -m "docs(llm-fundamentals): 添加新笔记"

# 4. 推送
git push origin main
```

### 功能分支流程

```bash
# 1. 创建并切换分支
git checkout -b docs/add-new-notes

# 2. 完成修改并提交（可多次提交）
git add .
git commit -m "docs(prompt): 添加 Prompt Engineering 学习笔记"

# 3. 推送分支
git push -u origin docs/add-new-notes

# 4. 创建 PR
gh pr create --title "docs(prompt): 添加 Prompt Engineering 学习笔记" --body "添加了 Prompt Engineering 模块的学习笔记和资源整理"
```

### 批量提交（多个模块变更）

当多个模块同时有变更时，按模块分别提交：

```bash
git add "LLM学习计划/01-llm-fundamentals/"
git commit -m "docs(llm-fundamentals): 更新基础模块笔记"

git add "LLM学习计划/02-prompt-engineering/"
git commit -m "docs(prompt): 添加提示工程笔记"

git push origin main
```

## 注意事项

- 不要提交 `.env`、密钥等敏感文件
- `.cursor/rules/` 和 `.cursor/skills/` 可以提交（项目配置）
- 大文件（模型权重等）不应直接提交，考虑使用 Git LFS 或 `.gitignore`
- 建议为此仓库添加 `.gitignore` 文件排除不需要的文件
