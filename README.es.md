# Consensus-AI

<!-- md-i18n:switcher -->
**En otros idiomas:** [English](README.md) · [Deutsch](README.de.md) · [中文](README.zh.md) · [हिन्दी](README.hi.md) · Español · [Русский](README.ru.md)

> **Autor:** Mikhail Deynekin – https://Deynekin.com – Mikhail@Deynekin.com  
> **Versión:** 1.0.0 (2026-04-27)  
> **Licencia:** PolyForm Noncommercial 1.0.0

## Descripción general

Consensus-AI es una aplicación web en PHP 8.4 que implementa un **consejo de IA al estilo de OpenCorum** para inteligencia deliberativa. Utiliza varios LLM simultáneamente a través de la API de [BotHub.chat](https://bothub.chat), combina sus salidas mediante un motor de consenso estructurado y ofrece transparencia total a través de un diagrama interactivo de **«Detalles»** para cada respuesta.

Inspirado en [OpenCorum](https://opencorum.com/) y en la [API CAIS](https://github.com/paulmann/CAIS).

## Características principales

- **Sesiones de consejo** multimodelo y multirrol a través de BotHub.chat (compatible con OpenAI)
- Sistema **SKILLS**: flujos de trabajo JSON declarativos que definen roles de agente, prompts y reglas de consenso
- Selección automática o creación automática del SKILL más relevante para cada consulta
- **Memoria vectorial**: almacenamiento de fragmentos y recuperación para contexto a corto y largo plazo
- **Registro de auditoría** completo: cada respuesta de modelo, cada paso y cada puntuación de consenso se guardan en MySQL
- **Botón «Detalles»** interactivo: DAG sobre el que se puede hacer clic y que muestra todo el flujo del consejo
- Tipos estrictos de PHP 8.4, propiedades readonly y enums en todo el código
- MySQL 8.x a través de PDO con consultas preparadas y transacciones explícitas

## Estructura del repositorio

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

## Arquitectura (consejo de 4 capas)

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

## SKILLS integrados

| Slug | Propósito | Roles |
|------|---------|-------|
| `webapp_architect` | Arquitectura de PHP + MySQL + almacén vectorial | Arquitecto de soluciones, Experto en escalabilidad, Ingeniero de seguridad |
| `db_performance_optimizer` | Optimización de consultas y esquemas de MySQL | Perfilador de consultas, Diseñador de almacenamiento, Planificador de migraciones |
| `php_code_refiner` | Refactorización de código PHP 8.4 | Revisor sénior de PHP, Especialista en manejo de errores, Revisor de preparación para el futuro |
| `ux_frontend_collaborator` | Diseño UI/UX moderno para 2026 | Diseñador UX, Ingeniero UI, Diseñador de interacción |

## Inicio rápido

1. Clone el repositorio
2. Configure `.env` con su clave de API de BotHub y sus credenciales de MySQL
3. Ejecute `db/schema.sql` en su instancia de MySQL 8.x
4. Configure un almacén vectorial (se recomienda Qdrant, o el respaldo integrado en MySQL)
5. Inicie el servidor integrado de PHP o despliegue en su servidor web

## Documentación

Todas las decisiones de diseño, los esquemas y el desglose de módulos están documentados en [`docs/`](docs/).

## Enlaces

- [API de BotHub.chat](https://bothub.chat)
- [OpenCorum](https://opencorum.com/)
- [Referencia de la API CAIS](https://github.com/paulmann/CAIS)
