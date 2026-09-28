---
author: "wdeng"
date: 2026-05-11
linktitle: Code Agent 的通用规范与技能体系
title: Code Agent 的通用规范与技能体系
weight: 15
---

主流 AI Code Agent（Cursor, Claude Code, Cline, Trae, Gemini CLI 等）虽然各有特色，但在"指令规范"和"技能扩展"上正在形成一套**跨平台通用的标准**。

本文说明所有主流 Agent 都共同支持的三大能力：统一的规则文件 `AGENTS.md`、开放的 Agent Skills 标准，以及 MCP 协议。

---

## 1. 统一的规则文件：`AGENTS.md`

所有主流 Agent 都支持通过项目根目录下的**规则文件**来注入长期行为准则，且正朝着统一的 `AGENTS.md` 命名对齐。

### 核心约定

| 约定 | 说明 |
| --- | --- |
| **文件位置** | 项目根目录（`./AGENTS.md`） |
| **格式** | 纯 Markdown，可包含 YAML Frontmatter |
| **加载时机** | Agent 初始化时自动读取，作为全局行为约束 |
| **作用范围** | 该项目下的所有对话 |

### 可以写什么

- **编码规范**：代码风格、命名约定、目录结构偏好
- **技术栈说明**：项目使用的框架、语言版本、构建工具
- **开发工作流**：提交规范、测试要求、Code Review 流程
- **安全约束**：禁止的操作、敏感信息处理方式
- **角色设定**：Agent 在本项目中的定位（如"你是一个资深 Go 工程师"）

### 示例

```markdown
# AGENTS.md

## 项目概述
- 这是一个 Go 微服务项目，使用 Go 1.22 和 gRPC。

## 编码规范
- 遵循 Uber Go Style Guide
- 所有公开函数必须写 godoc 注释
- 错误处理使用 `fmt.Errorf("context: %w", err)` 包装

## 工作流
- 修改代码后必须运行 `make test`
- 新功能需要补充单元测试，覆盖率不低于 70%
```

### 跨版本兼容

早期的 Agent 可能使用不同的文件名（如 `.cursorrules`、`CLAUDE.md`），你可以在这些文件中各写一句引用：

```markdown
# .cursorrules
Always follow the guidelines in AGENTS.md
```

这样一份 `AGENTS.md` 就能服务所有 Agent。

---

## 2. 开放的技能标准：Agent Skills

所有主流 Agent 都支持**技能（Skills）**机制——让 Agent 在需要时执行具体的任务流（读写文件、运行脚本、调用外部接口）。

[Agent Skills](https://agentskills.io/) 是一个跨平台的开放标准，被 Claude Code、Cline、Trae 等广泛采用。

### 目录结构

```
.agent/skills/
└── your-skill/
    ├── SKILL.md          # 技能说明书（触发条件 + 执行步骤）
    ├── script.py         # 可选：脚本实现
    └── data.json         # 可选：数据文件
```

### SKILL.md 格式

```markdown
---
name: database-migration
description: 当用户需要创建数据库迁移脚本或执行迁移时触发
triggers:
  - "migrate"
  - "migration"
  - "数据库迁移"
---

# 数据库迁移技能

## 何时使用
当用户请求创建或执行数据库迁移时。

## 执行步骤
1. 在 `migrations/` 目录下创建新文件，命名格式 `YYYYMMDDHHMMSS_description.sql`
2. 编写 SQL，同时包含 UP 和 DOWN 部分
3. 运行 `make migrate-up` 验证
4. 输出迁移结果
```

### 核心设计理念

| 特性 | 说明 |
| --- | --- |
| **按需加载** | Agent 先读取 `description` 判断是否需要，仅在触发时才加载全文，节省 Token |
| **Prompt + 脚本结合** | 简单逻辑写在 SKILL.md 中，复杂逻辑用独立脚本实现 |
| **纯 Shell 可执行** | 脚本层用 Python/Node/Bash，任何 Agent 都能通过 Shell 调用 |

---

## 3. 终极扩展：MCP (Model Context Protocol)

MCP 是所有主流 Agent 都支持的**外部工具接入协议**，由 Anthropic 发起，已成为行业标准。

### 什么是 MCP

MCP 定义了一套 Client-Server 协议：

```
┌──────────────┐    MCP 协议     ┌──────────────┐
│  Code Agent  │ ◄──────────────► │  MCP Server  │
│  (Client)    │   stdio/HTTP     │  (Tool 提供方)│
└──────────────┘                  └──────────────┘
```

- **MCP Server**：把能力封装成标准的 Tool（如"查询数据库"、"部署服务"、"操作飞书文档"）
- **Code Agent**：作为 MCP Client，发现并调用 Server 提供的 Tool

### 为什么 MCP 是终极方案

| 优势 | 说明 |
| --- | --- |
| **一次开发，全平台可用** | 写一个 MCP Server，Cursor、Claude Code、Cline、Gemini CLI 都能直接连接 |
| **类型安全** | Tool 的入参和返回值有 JSON Schema 定义，Agent 知道传什么、会收到什么 |
| **无需 Shell** | 比起让 Agent 拼命令行，MCP 提供结构化调用，更稳定、更安全 |
| **官方生态** | 已有数百个现成的 MCP Server（GitHub、PostgreSQL、Slack、飞书等） |

### 典型应用

- **内部系统对接**：把公司的 OA、数据库、部署平台封装成 MCP Server
- **云资源操作**：通过 MCP Tool 创建 EC2 实例、查询 S3 文件
- **文档自动化**：读取/写入飞书文档、Confluence 页面
- **CLI 工具增强**：给 Agent 一个 Tool 而不是让它猜命令行参数

---

## 推荐工作流

> **1. 规则层**：维护一份 `AGENTS.md`，作为所有 Agent 的行为准则
> **2. 技能层**：遵循 Agent Skills 标准，把可复用的任务流写成 `.agent/skills/` 下的技能
> **3. 扩展层**：需要对接外部系统（数据库、API、第三方服务）时，开发 MCP Server

三层递进，按需组合，一份配置服务所有主流 Code Agent。
