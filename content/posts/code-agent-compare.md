---
author: "wdeng"
date: 2026-05-11
linktitle: 不同code agent的系统提示符和skills实现
title: 不同code agent的系统提示符和skills实现
weight: 15
---

目前主流的 AI Code Agent 正在从“纯 Prompt 指令”转向“**指令 + 技能 (Agent Skills)**”的混合架构。

以下是各主流 Agent（Gemini CLI, Cursor, Claude Code, Cline, Trae 等）在系统提示符（System Prompt）与技能（Skills）实现上的对比，以及如何实现跨平台兼容。

---

## 1. 系统提示符 (System Prompt) 的实现对比

系统提示符决定了 Agent 的“人设”和基本行为准则。

| Agent | 实现方式 | 作用范围 | 特色 |
| --- | --- | --- | --- |
| **Cursor** | `.cursorrules` 文件 | 全局或文件夹级 | 支持 frontmatter 声明 `alwaysApply` 或根据文件后缀（globs）自动挂载。 |
| **Claude Code** | `CLAUDE.md` 文件 | 项目根目录 | 纯 Markdown 格式，Claude 在初始化时会自动读取该文件作为行为准则。 |
| **Cline** | `Custom Instructions` | 全局配置 | 在设置中填写的全局 Prompt，所有对话都会携带。 |
| **Trae** | `.traerules` 文件 | 项目根目录 | 类似于 Cursor，通过特定的规则文件定义项目特定的编码标准。 |
| **Gemini CLI** | `settings.json` / `mission.md` | 项目或全局 | 通过 `/memory` 指令或 `mission.md` 注入长期上下文。 |
| **GitHub Copilot** | `copilot.customizations.json` / `Copilot Studio` | 全局或工作区 | 通过 Copilot Studio 配置代码风格、注释偏好和任务提示。 |

---

## 2. 技能 (Skills) 的实现对比

技能赋予 Agent “执行”能力（读写、搜索、运行脚本）。

| Agent | 实现方式 | 技术标准 | 核心逻辑 |
| --- | --- | --- | --- |
| **Claude Code** | `.claude/skills/` | **Agent Skills 标准** | 目录包含 `SKILL.md`（说明书）和脚本。支持运行时动态执行。 |
| **Cursor** | `/skill` 命令 | 私有实现 | 可以是保存的复杂 Prompt 或结合外部文档的 Workflow。 |
| **Cline** | `.cline/skills/` | **MCP (Model Context Protocol)** | 深度集成 MCP，允许 Agent 发现并调用外部 MCP Server 提供的 Tool。 |
| **Gemini CLI** | MCP Server | **MCP** | 通过 `settings.json` 连接 MCP 服务器（如 Gitea, Google Cloud）。 |
| **GitHub Copilot** | `Copilot Studio` / `copilot.customizations.json` | 全局或工作区 | 以“建议补全”为核心，支持基于代码上下文的实时提示和配置化行为。 |
| **OpenDevin (OpenCode)** | Python Sandbox | 自定义 Agent | 运行在 Docker 中的沙箱环境，通过 Python 函数封装 Skill。 |

---

## 3. 核心差异总结

1. **静态 vs 动态**：Cursor/Trae 偏向**静态规则**（Rules），告诉 AI 怎么写代码；Claude Code/Cline 偏向**动态技能**（Skills），告诉 AI 怎么完成一个具体的任务流。
2. **协议化：** **MCP (Model Context Protocol)** 正在成为技能实现的行业标准。Cline 和 Gemini CLI 已经完全转向 MCP，而 Claude Code 也通过其 Agent Skills 标准与 MCP 互通。GitHub Copilot 则主要在 Studio 配置层面进行编码建议优化，不以技能调用为核心，但在未来也可能通过 Copilot for Business 与更多工具链集成。
3. **按需加载**：现代 Agent 不再一次性加载所有指令，而是先读取 `SKILL.md` 的描述，当 Agent 认为需要时才加载全文，节省 Token。

---

## 4. 如何实现兼容多个 Agent 的系统提示符和技能？

如果你希望一份配置在 Cursor, Claude Code 和 Cline 之间通用，可以采用以下“三位一体”方案：

### A. 指令兼容：建立 `AGENT.md` 规范

不要只写 `.cursorrules`。在根目录建立一个核心文档，然后让各 Agent 引用它：

1. 创建 `DEVELOPMENT_STANDARDS.md` 存储核心编码规范。
2. 在 `.cursorrules` 中写：`Include standards from DEVELOPMENT_STANDARDS.md`。
3. 在 `CLAUDE.md` 中写：`Follow the guidelines in DEVELOPMENT_STANDARDS.md`。

### B. 技能兼容：采用 Agent Skills 开放标准

Claude Code 推广的 [Agent Skills](https://agentskills.io/) 标准是跨平台的最佳实践：

* **结构**：创建一个目录 `.agent/skills/your-skill/`。
* **SKILL.md**：包含标准 YAML Frontmatter（描述技能何时触发）。
* **脚本层**：将具体逻辑写成独立脚本（Python/Node），由不同 Agent 通过 Shell 调用。

### C. 接口兼容：使用 MCP (Model Context Protocol)

MCP 是终极解决方案。

1. **开发一个 MCP Server**，将你的所有自定义技能（如“查询公司数据库”、“部署至预发环境”）封装为 MCP Tools。
2. **Cline / Gemini CLI**：直接在配置中指向该 MCP Server 地址。
3. **Claude Code**：支持连接 MCP。
4. **Cursor**：目前主要通过 Custom Rules 模拟，但也可以通过运行一个本地脚本间接调用 MCP 接口。

---

### 推荐的工作流

> **1. 定义规则：** 维护一份 `RULES.md`，手动链接到 `.cursorrules`, `CLAUDE.md`, `.traerules`。
> **2. 定义技能：** 遵循 **Agent Skills** 目录结构，并在 `SKILL.md` 中提供清晰的触发描述。
> **3. 扩展能力：** 复杂的外部交互优先写成 **MCP Server**。