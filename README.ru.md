# Consensus-AI

<!-- md-i18n:switcher -->
**На других языках:** [English](README.md) · [Deutsch](README.de.md) · [中文](README.zh.md) · [हिन्दी](README.hi.md) · [Español](README.es.md) · Русский

> **Автор:** Mikhail Deynekin – https://Deynekin.com – Mikhail@Deynekin.com  
> **Версия:** 1.0.0 (2026-04-27)  
> **Лицензия:** PolyForm Noncommercial 1.0.0

## Обзор

Consensus-AI — веб-приложение на PHP 8.4, реализующее **ИИ-совет в стиле OpenCorum** для совещательного интеллекта. Оно использует несколько LLM одновременно через API [BotHub.chat](https://bothub.chat), объединяет их выводы структурированным движком консенсуса и раскрывает весь ход работы через интерактивную диаграмму **«Details»** для каждого ответа.

Вдохновлено [OpenCorum](https://opencorum.com/) и [CAIS API](https://github.com/paulmann/CAIS).

## Ключевые возможности

- **Сессии совета** с множеством моделей и ролей через BotHub.chat (совместим с OpenAI)
- Система **SKILLS**: декларативные процессы на JSON, задающие роли агентов, промпты и правила консенсуса
- Автовыбор или автосоздание наиболее подходящего SKILL под запрос
- **Векторная память**: хранение чанков и их извлечение для краткосрочного и долгосрочного контекста
- Полный **журнал аудита**: каждый ответ модели, каждый шаг, каждая оценка консенсуса сохраняются в MySQL
- Интерактивная **кнопка «Details»**: кликабельный DAG, показывающий весь ход работы совета
- PHP 8.4: strict types, readonly-свойства, повсеместное использование enum
- MySQL 8.x через PDO с подготовленными выражениями и явными транзакциями

## Структура репозитория

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

## Архитектура (совет из 4 уровней)

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

## Встроенные SKILLS

| Slug | Назначение | Роли |
|------|---------|-------|
| `webapp_architect` | Архитектура PHP+MySQL+векторного хранилища | Solution Architect, Scalability Expert, Security Engineer |
| `db_performance_optimizer` | Оптимизация запросов и схемы MySQL | Query Profiler, Storage Designer, Migration Planner |
| `php_code_refiner` | Рефакторинг кода PHP 8.4 | Senior PHP Reviewer, Error-Handling Specialist, Future-Proofing Reviewer |
| `ux_frontend_collaborator` | Современный дизайн интерфейсов и UX 2026 года | UX Designer, UI Engineer, Interaction Designer |

## Быстрый старт

1. Клонируйте репозиторий
2. Настройте `.env`: ключ API BotHub и учётные данные MySQL
3. Выполните `db/schema.sql` на своём экземпляре MySQL 8.x
4. Настройте векторное хранилище (рекомендуется Qdrant, либо запасной вариант — внутри MySQL)
5. Запустите встроенный сервер PHP или разверните приложение на своём веб-сервере

## Документация

Все проектные решения, схемы и разбивка на модули описаны в [`docs/`](docs/).

## Ссылки

- [BotHub.chat API](https://bothub.chat)
- [OpenCorum](https://opencorum.com/)
- [Справочник по API CAIS](https://github.com/paulmann/CAIS)
