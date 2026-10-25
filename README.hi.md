# Consensus-AI

<!-- md-i18n:switcher -->
**अन्य भाषाओं में:** [English](README.md) · [Deutsch](README.de.md) · [中文](README.zh.md) · हिन्दी · [Español](README.es.md) · [Русский](README.ru.md)

> **लेखक:** Mikhail Deynekin – https://Deynekin.com – Mikhail@Deynekin.com  
> **संस्करण:** 1.0.0 (2026-04-27)  
> **लाइसेंस:** PolyForm Noncommercial 1.0.0

## अवलोकन

Consensus-AI एक PHP 8.4 वेब एप्लिकेशन है जो deliberative intelligence के लिए **OpenCorum-शैली की AI council** लागू करता है। यह [BotHub.chat](https://bothub.chat) API के ज़रिए एक साथ कई LLMs का उपयोग करता है, उनके आउटपुट को एक संरचित consensus engine से जोड़ता है, और हर उत्तर के लिए इंटरैक्टिव **"Details"** डायग्राम के ज़रिए पूरी पारदर्शिता दिखाता है।

[OpenCorum](https://opencorum.com/) और [CAIS API](https://github.com/paulmann/CAIS) से प्रेरित।

## मुख्य विशेषताएँ

- Multi-model, multi-role **council sessions**, BotHub.chat (OpenAI-compatible) के ज़रिए
- **SKILLS** सिस्टम: declarative JSON workflows जो agent roles, prompts और consensus rules परिभाषित करते हैं
- हर query के लिए सबसे प्रासंगिक SKILL का auto-selection या auto-creation
- **Vector memory**: short-term और long-term context के लिए chunk storage + retrieval
- पूरा **audit trail**: हर model response, हर step, हर consensus score MySQL में सुरक्षित रहता है
- इंटरैक्टिव **"Details" button**: क्लिक करने योग्य DAG जो पूरा council flow दिखाता है
- PHP 8.4 strict types, readonly properties, enums — हर जगह
- PDO के ज़रिए MySQL 8.x, prepared statements और explicit transactions के साथ

## रिपॉज़िटरी संरचना

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

## आर्किटेक्चर (4-Layer Council)

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

## अंतर्निहित SKILLS

| Slug | उद्देश्य | Roles |
|------|---------|-------|
| `webapp_architect` | PHP+MySQL+vector store आर्किटेक्चर | Solution Architect, Scalability Expert, Security Engineer |
| `db_performance_optimizer` | MySQL query/schema अनुकूलन | Query Profiler, Storage Designer, Migration Planner |
| `php_code_refiner` | PHP 8.4 कोड रीफ़ैक्टरिंग | Senior PHP Reviewer, Error-Handling Specialist, Future-Proofing Reviewer |
| `ux_frontend_collaborator` | आधुनिक 2026 UI/UX डिज़ाइन | UX Designer, UI Engineer, Interaction Designer |

## त्वरित शुरुआत

1. रिपॉज़िटरी क्लोन करें
2. `.env` में अपनी BotHub API key और MySQL credentials कॉन्फ़िगर करें
3. अपने MySQL 8.x instance पर `db/schema.sql` चलाएँ
4. एक vector store कॉन्फ़िगर करें (Qdrant अनुशंसित, या MySQL-inline fallback)
5. PHP का built-in server शुरू करें या अपने web server पर deploy करें

## दस्तावेज़ीकरण

सभी design decisions, schemas और module breakdowns [`docs/`](docs/) में दस्तावेज़ित हैं।

## लिंक

- [BotHub.chat API](https://bothub.chat)
- [OpenCorum](https://opencorum.com/)
- [CAIS API Reference](https://github.com/paulmann/CAIS)
