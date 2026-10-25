# Consensus-AI

<!-- md-i18n:switcher -->
**In anderen Sprachen:** [English](README.md) · Deutsch · [中文](README.zh.md) · [हिन्दी](README.hi.md) · [Español](README.es.md) · [Русский](README.ru.md)

> **Autor:** Mikhail Deynekin – https://Deynekin.com – Mikhail@Deynekin.com  
> **Version:** 1.0.0 (2026-04-27)  
> **Lizenz:** PolyForm Noncommercial 1.0.0

## Überblick

Consensus-AI ist eine Webanwendung auf PHP 8.4, die einen **KI-Rat im Stil von OpenCorum** für deliberative Intelligenz umsetzt. Sie nutzt mehrere LLMs gleichzeitig über die API von [BotHub.chat](https://bothub.chat), führt deren Ausgaben in einer strukturierten Konsens-Engine zusammen und macht den gesamten Ablauf über ein interaktives **„Details“**-Diagramm zu jeder Antwort transparent.

Inspiriert von [OpenCorum](https://opencorum.com/) und der [CAIS API](https://github.com/paulmann/CAIS).

## Zentrale Funktionen

- **Ratssitzungen** mit mehreren Modellen und Rollen über BotHub.chat (OpenAI-kompatibel)
- **SKILLS**-System: deklarative JSON-Workflows, die Agentenrollen, Prompts und Konsensregeln festlegen
- Automatische Auswahl oder automatische Erstellung des passendsten SKILL je Anfrage
- **Vektorspeicher**: Ablage und Abruf von Chunks für kurz- und langfristigen Kontext
- Vollständiger **Audit-Trail**: jede Modellantwort, jeder Schritt, jeder Konsenswert in MySQL gespeichert
- Interaktive **„Details“**-Schaltfläche: anklickbarer DAG, der den gesamten Ablauf des Rats zeigt
- PHP 8.4 mit strikten Typen, readonly-Properties und Enums durchgängig
- MySQL 8.x über PDO mit Prepared Statements und expliziten Transaktionen

## Repository-Struktur

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

## Architektur (Council mit vier Ebenen)

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

## Integrierte SKILLS

| Slug | Zweck | Rollen |
|------|---------|-------|
| `webapp_architect` | Architektur für PHP + MySQL + Vektorspeicher | Lösungsarchitekt, Skalierungsexperte, Sicherheitsingenieur |
| `db_performance_optimizer` | Optimierung von MySQL-Abfragen und -Schema | Abfrage-Profiler, Storage-Designer, Migrationsplaner |
| `php_code_refiner` | Refactoring von PHP-8.4-Code | Senior-PHP-Reviewer, Spezialist für Fehlerbehandlung, Reviewer für Zukunftssicherheit |
| `ux_frontend_collaborator` | Modernes UI/UX-Design 2026 | UX-Designer, UI-Engineer, Interaction-Designer |

## Schnellstart

1. Das Repository klonen
2. `.env` mit dem eigenen BotHub-API-Schlüssel und den MySQL-Zugangsdaten konfigurieren
3. `db/schema.sql` in der eigenen MySQL-8.x-Instanz ausführen
4. Einen Vektorspeicher konfigurieren (empfohlen: Qdrant, alternativ der MySQL-inline-Fallback)
5. Den integrierten PHP-Server starten oder auf dem eigenen Webserver ausrollen

## Dokumentation

Alle Entwurfsentscheidungen, Schemata und Modulaufteilungen sind in [`docs/`](docs/) dokumentiert.

## Links

- [BotHub.chat API](https://bothub.chat)
- [OpenCorum](https://opencorum.com/)
- [CAIS API Reference](https://github.com/paulmann/CAIS)
