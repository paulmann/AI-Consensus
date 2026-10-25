# Consensus-AI

<!-- md-i18n:switcher -->
**其他语言：** [English](README.md) · [Deutsch](README.de.md) · 中文 · [हिन्दी](README.hi.md) · [Español](README.es.md) · [Русский](README.ru.md)

> **作者:** Mikhail Deynekin – https://Deynekin.com – Mikhail@Deynekin.com  
> **版本:** 1.0.0 (2026-04-27)  
> **许可证:** PolyForm Noncommercial 1.0.0

## 概述

Consensus-AI 是一个 PHP 8.4 Web 应用，实现用于协商式智能的 **OpenCorum 风格 AI 议事会**。它通过 [BotHub.chat](https://bothub.chat) API 同时调用多个 LLM，由结构化共识引擎合并各模型的输出，并为每个答案提供可交互的 **"Details"** 示意图，使整个过程完全透明。

灵感来自 [OpenCorum](https://opencorum.com/) 与 [CAIS API](https://github.com/paulmann/CAIS)。

## 核心特性

- 通过 BotHub.chat（兼容 OpenAI）进行多模型、多角色的**议事会会话**
- **SKILLS** 系统：以声明式 JSON 工作流定义智能体角色、提示词与共识规则
- 为每次查询自动选择或自动创建最相关的 SKILL
- **向量记忆**：分块存储与检索，覆盖短期和长期上下文
- 完整**审计轨迹**：每次模型响应、每个步骤、每个共识评分都持久化到 MySQL
- 全面采用 PHP 8.4 严格类型、readonly 属性与枚举
- 通过 PDO 使用 MySQL 8.x，配合预处理语句与显式事务

## 仓库结构

```
Consensus-AI/
├── README.md
├── docs/
│   ├── Part1-Architecture.md         # Overall concept, functional requirements, tech stack
│   ├── Part2-Skills.md               # SKILL JSON schema + 4 concrete SKILLS
│   ├── Part3-Database-Interfaces.md  # MySQL DDL + PHP interfaces
│   ├── Part4-CouncilEngine-API.md    # CouncilEngine layers + API endpoints
│   ├── Part5-Controllers-Flow.md     # HTTP controllers + downgrade mitigation
│   └── Part6-CouncilEngine.md        # Full CouncilEngine PHP 8.4 module
├── src/
│   ├── Application/
│   │   ├── Council/
│   │   │   └── CouncilEngine.php
│   │   ├── Skill/
│   │   └── Memory/
│   ├── Domain/
│   │   ├── Council/
│   │   ├── Skill/
│   │   └── Memory/
│   ├── Infrastructure/
│   │   └── LLM/
│   └── Http/
│       └── Controller/
└── db/
    └── schema.sql
```

## 架构（四层议事会）

```
User Query
    │
    ▼
[L0] Skill Router ──► Auto-create SKILL if needed
    │
    ▼
[L1] Query Distribution ──► Parallel calls to multiple roles + models
    │
    ▼
[L2] Response Normalization ──► JSON schema enforcement per role
    │
    ▼
[L3] Consensus Analysis ──► Agreement scores, conflict detection
    │
    ▼
[L4] Decision Synthesis ──► Final answer + audit trail
    │
    ▼
Final Answer + [Details] button
```

## 内置 SKILLS

| 标识 | 用途 | 角色 |
|------|---------|-------|
| `webapp_architect` | PHP+MySQL+向量存储架构 | 解决方案架构师、可扩展性专家、安全工程师 |
| `db_performance_optimizer` | MySQL 查询与 schema 优化 | 查询剖析师、存储设计师、迁移规划师 |
| `php_code_refiner` | PHP 8.4 代码重构 | 资深 PHP 评审员、错误处理专家、前瞻性评审员 |
| `ux_frontend_collaborator` | 面向 2026 的现代 UI/UX 设计 | UX 设计师、UI 工程师、交互设计师 |

## 快速开始

1. 克隆仓库
2. 配置 `.env`，填入 BotHub API 密钥与 MySQL 凭据
3. 对 MySQL 8.x 实例执行 `db/schema.sql`
4. 配置向量存储（推荐 Qdrant，或使用 MySQL 内联回退方案）
5. 启动 PHP 内置服务器，或部署到你的 Web 服务器

## 文档

所有设计决策、schema 与模块划分都记录在 [`docs/`](docs/) 中。

## 链接

- [BotHub.chat API](https://bothub.chat)
- [OpenCorum](https://opencorum.com/)
- [CAIS API Reference](https://github.com/paulmann/CAIS)
