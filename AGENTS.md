# AGENTS.md — Consensus-AI 1.0.0

Agent-facing guide: what this repository installs, what it changes on a host, and where the current
source tree stops working. Human documentation: [README.md](README.md), [docs/](docs/).

> Verified against repository state on 2026-10-22. Version covered: 1.0.0 (`README.md:4`; note that
> `public/index.php:7` still carries `@version 2.0.0`). No tags and no releases exist.

## 1. What it is

A PHP web application that sends one query to several LLM roles through a BotHub-compatible API,
normalizes the role answers, scores their agreement and persists an audit trail in MySQL. SKILLs are
JSON definitions of roles, prompts and a consensus strategy. The repository contains a JSON HTTP API
only: there is no web UI, no installer and no CLI.

| | |
|---|---|
| Runtime / platform | PHP >= 8.4 with `ext-pdo`, `ext-json`, `ext-mbstring` (`composer.json:7-10`); MySQL 8.x; any web server with docroot `public/` |
| Entry point | `public/index.php` — front controller; all routes are registered there (`public/index.php:120-130`) |
| Writes to | MySQL tables from `db/schema.sql`; `logs/error.log` (`public/index.php:62-65`); PHP session storage (`public/index.php:68-70`) |
| Network access | `https://api.bothub.chat/v1` only (`src/Infrastructure/BothubChat/BothubClient.php:25`) |

**Status of this source tree.** Do not expect a working installation yet. The front controller
references types that are not in the repository and calls constructors with signatures that do not
match; see section 7. Treat the tree as a documented design plus a partial implementation.

## 2. Preconditions

| Requirement | Why | Check |
|---|---|---|
| PHP >= 8.4 | `composer.json:7`; code uses readonly classes and enums | `php -v` |
| `ext-pdo`, `ext-json`, `ext-mbstring` | `composer.json:8-10` | `php -m` |
| `ext-curl` | `BothubClient` calls `curl_*` (`BothubClient.php:142-166`), but it is not declared in `composer.json` | `php -m` |
| MySQL 8.x reachable, empty database | `README.md:91` | `mysql --version`; connect and `SHOW DATABASES;` |
| Composer and `vendor/autoload.php` | `public/index.php:27` requires `vendor/autoload.php`; `composer.json` is present | `composer --version` |
| `psr/log` and `guzzlehttp/guzzle` | imported by `src/` (`LoggerInterface`, `ClientInterface`) but absent from `composer.json` require | `composer show` after install |

If a check fails, stop and report; do not work around it silently. On the Windows verification host
PHP 8.5.0 exists (`php -l` passes on every file), Composer is **not** installed, `vendor/` is absent
and MySQL was not exercised.

## 3. Install

### 3.1 From a release

Source install only - this repository has no releases. (`gh release list -R paulmann/AI-Consensus`
returns nothing; `gh api repos/paulmann/AI-Consensus/releases --jq length` prints `0`.)

### 3.2 From source

`README.md:89-93` lists five steps but omits dependency installation; the required commands are:

```bash
git clone https://github.com/paulmann/AI-Consensus.git
cd AI-Consensus
composer install                                   # required by public/index.php:27
cp .env.example .env                               # Windows: Copy-Item .env.example .env
mysql -u root -p -e "CREATE DATABASE consensus_ai CHARACTER SET utf8mb4"
mysql -u root -p consensus_ai < db/schema.sql      # README.md:91
```

`db/schema.sql` itself contains no `CREATE DATABASE` (it starts with `SET NAMES utf8mb4`), so the
database must be created first; `.env.example:15` names it `consensus_ai`. The repository ships no
web-server rewrite configuration (no `.htaccess`, no nginx/apache snippet), so with the PHP built-in
server that `README.md:93` mentions, pass the front controller as the router:

```bash
php -S localhost:8080 -t public public/index.php
```

PHP CLI help: `php [options] -S <addr>:<port> [-t docroot] [router]` / `-S <addr>:<port> Run with
built-in web server.` `APP_URL=http://localhost:8080` in `.env.example:8` matches this port.

### 3.3 Verify

```bash
php -l public/index.php
curl -s http://localhost:8080/api/skills
```

Expected: `No syntax errors detected in public/index.php`, then JSON shaped as
`{"data":[...],"meta":{"page":1,"per_page":50,"total":...,"total_pages":...}}`
(`SkillController::index`, `src/Http/Controller/SkillController.php:55-63`). The second check cannot
currently pass: the request dies during wiring and returns HTTP 500 (section 7). Do not report the
installation as successful while it returns 500.

## 4. Configure

`.env` is read from the repository root by a simple parser that only sets `$_ENV`/`$_SERVER` values
(`public/index.php:33-52`); quotes are not stripped.

| Key | Read by | Notes |
|---|---|---|
| `APP_DEBUG` | `public/index.php:58` | `1` returns the exception message, file and line as JSON |
| `DB_DSN` | `public/index.php:78` | default `mysql:host=127.0.0.1;dbname=consensus_ai;charset=utf8mb4`; **not** present in `.env.example` |
| `DB_USER`, `DB_PASSWORD` | `public/index.php:79-80` | `.env.example` defines `DB_USER` but `DB_PASS`, not `DB_PASSWORD` |
| `BOTHUB_API_KEY` | `public/index.php:93` | required: empty value throws in `BothubClient.php:38-40`; **not** present in `.env.example` |
| `BOTHUB_BASE_URL` | `public/index.php:92` | passed as `baseUrl:` but the constructor has no such parameter and the URI is hardcoded — the value is ignored |
| `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_PASS`, `DB_CHARSET` | `src/Infrastructure/Database/DatabaseConnection.php:37-42` | second, different set of names; only used when `DatabaseConnection` is used |
| `APP_*`, `LOG_*`, `SESSION_*`, `CACHE_*`, `MEMORY_*`, `AI_*`, `RATE_LIMIT_*` | nothing | present in `.env.example:5-43`; no code reads them (searched every `$_ENV` use in `src/`) |

Only `BOTHUB_API_KEY` is a secret. The repository has no `.gitignore`, so keep `.env`, `vendor/` and
`logs/` out of commits by hand.

## 5. Use

All operations are HTTP requests; there is no CLI and therefore no exit codes.

| Purpose | Request | Result |
|---|---|---|
| List skills | `GET /api/skills?page=1&per_page=50` | `data[]` plus `meta`; `per_page` clamped to 1..100, default 50 (`SkillController.php:44-45`) |
| Read one skill | `GET /api/skills/{id}` | skill object, or 404 `{"error":"Skill not found"}` (`:74-78`) |
| Create a skill | `POST /api/skills` with JSON body | 201 and the saved skill |
| Delete a skill | `DELETE /api/skills/{id}` | 204, or 404; delete is soft (`is_active = 0`, `src/Infrastructure/Council/SkillRepositoryPdo.php:250-256`) |
| Run a council session | `POST /api/council/run` | 200 `{"session_id":N,"final_answer":"...","consensus":{"score":F},"audit_trail":[]}` (`CouncilController.php:133-143`) |

`POST /api/skills` body: `{"name":"php_review","description":"...","roles":[{...}],"consensus":"synthesis","tags":["php"]}`.
Validation comes from `src/Domain/Council/Skill.php`: `name` snake_case, 3-100 characters; 1-10 roles,
each with non-empty `id`, `name` and `system_prompt`; `consensus` one of `majority_vote`,
`weighted_average`, `synthesis`; every tag a non-empty string. A violation returns 400
`{"error":"Invalid SKILL data: ..."}`.

`POST /api/council/run` body: `{"query":"...","skill_id":null,"user_id":null}` (`CouncilController.php:52-54`).
400 on invalid JSON or empty query, 500 `{"error":"Internal server error"}` on engine failure.
`audit_trail` is always an empty array (`CouncilController.php:141`), and `consensus` carries only
`score`. Status codes in use: 200, 201, 204, 400, 404, 500, 501. 501 means a placeholder endpoint
(`getSession`, `getSessionGraph`, `getSessionStep`, `CouncilController.php:94-128`). Anything else is
`404 {"error":"Route not found"}` from `src/Http/Router.php:80-82`.

Do not rely on `GET /api/skills/search?q=...`: it is registered after `/api/skills/{id}`
(`public/index.php:127` then `:130`), so the `{id}` route matches first and the handler receives the
string `search` for an `int` parameter.

## 6. Safety and rollback

- Changes: creates the seven tables of `db/schema.sql` (`skills`, `council_sessions`, `council_steps`,
  `council_step_responses`, `memory_chunks`, `memory_embeddings`, `vector_embeddings`), creates
  `logs/error.log`, starts a PHP session, and on the first request against an empty `skills` table
  inserts a seeded SKILL row — that seeding tries an LLM call with `gpt-4o`
  (`public/index.php:107-110`, `src/Application/Council/SkillAutoCreator.php:112-140`), so the first
  request can consume BotHub credits.
- Never does: install services, write web-server or PHP configuration, touch firewall or mail, or
  write outside the repository except the PHP session directory.
- Undo / uninstall: stop the PHP process, delete `.env`, `logs/` and `vendor/`, then either drop the
  database or drop the tables in foreign-key order:
  `DROP TABLE council_step_responses, council_steps, council_sessions, memory_embeddings, memory_chunks, vector_embeddings, skills;`
- On a production host: do not deploy this tree to a shared or production database. Table names
  (`skills`, `memory_chunks`) are generic and can collide with existing schemas.
- Safe first-run order: the product has no help command and no dry-run mode (no CLI, no `bin/`). The
  closest safe sequence is: syntax check (`php -l`) -> boot against a throwaway empty database ->
  read-only `GET /api/skills` -> create one skill -> one `POST /api/council/run` -> only then real
  data. Steps 2-5 cannot be completed in the current tree (section 7).

## 7. Failure modes

| Symptom | Likely cause | Action |
|---|---|---|
| `Unknown named parameter $baseUrl` | `public/index.php:91-95` passes `baseUrl:`; the constructor is `BothubClient(string $apiKey, LoggerInterface $logger)` (`BothubClient.php:34-37`) | Report; do not change code as part of an install |
| `ArgumentCountError` for `CouncilEngine` | `public/index.php:104` passes 4 arguments; the constructor requires 9 (`CouncilEngine.php:65-75`) | Report |
| `Class "App\Application\Skill\SkillRouterInterface" not found` or `App\Infrastructure\LLM\...` | referenced types are absent; `src/Application/Skill/` and `src/Infrastructure/LLM/` do not exist (`SkillRouter.php:14`, `BothubClient.php:14`, `CouncilEngine.php:16-24`) | Report; the tree cannot boot until these exist |
| 500 `Database connection failed` | MySQL unreachable or database missing (`DatabaseConnection.php:74-78`) | Fix connection parameters |
| 500 `Base table or view not found` | `db/schema.sql` not loaded | Load it into the configured database |
| `BothubClient requires a non-empty API key.` | `BOTHUB_API_KEY` empty or missing — `.env.example` omits the key | Add it to `.env` |
| 404 `{"error":"Route not found"}` | path not registered in `public/index.php:120-130`, or `index.php` was not used as the built-in-server router | Check the URL against section 5 |
| 400 `Name and description are required` | `POST /api/skills` body without `name`/`description` | Send a complete body |
| 501 `Not implemented yet` | session detail, graph and step endpoints are placeholders | Expected; do not report as a bug in the install |

## 8. Limits

- No web UI: `public/` contains only `index.php`, although `README.md:20` promises an interactive
  "Details" diagram. The graph/step endpoints return 501.
- No authentication or authorization in the request path (`Router.php` and both controllers perform no
  auth check) and CORS is `Access-Control-Allow-Origin: *`. Never expose it publicly.
- Rate limiting, caching and logging settings from `.env.example` are not implemented.
- No `tests/` directory, no `.gitignore`, no CI workflows, although `composer.json:22-31` references
  `tests/`, `phpunit`, `phpstan` and `phpcs`.
- The four built-in SKILLs listed in `README.md:80-85` (`webapp_architect`, `db_performance_optimizer`,
  `php_code_refiner`, `ux_frontend_collaborator`) are not seeded anywhere in `src/`; only the fallback
  `general_consensus_council` (`SkillAutoCreator.php:230-331`) is created.
- Do not use the Composer script `db:init` (`composer.json:31`): `DatabaseTables::createAll` creates
  `users`, `topics`, `sessions`, `participants`, `arguments`, `memory_items`, `session_topics`, a
  different schema from `db/schema.sql`, which is what `SkillRepositoryPdo` queries.
- PHP lower than 8.4 is unsupported. MySQL 8.x is required by `README.md:22`; the schema uses `JSON`,
  `FULLTEXT` and `ENUM` columns.

## 9. Sources of truth

- `README.md` — version 1.0.0, PHP 8.4 / MySQL 8.x requirement, Quick Start steps, built-in SKILL table.
- `composer.json` — PHP and extension requirements, PSR-4 mapping `App\` -> `src/`, scripts.
- `.env.example` — the configuration template; keys are named differently from what the code reads.
- `public/index.php` — env parsing, dependency wiring, route table, error handling, log path.
- `src/Http/Controller/*.php` — HTTP contract, status codes and JSON shapes.
- `src/Domain/Council/Skill.php` — SKILL validation rules.
- `db/schema.sql` — the tables actually queried by `SkillRepositoryPdo`.
- `docs/Part1-Architecture.md` … `docs/Part7-Memory-System-with-Vector-Storage.md` — design intent;
  several details differ from the code (for example the DDL in `docs/Part3-Database-Interfaces.md`
  and the request field `skill_slug` in `docs/Part4-CouncilEngine-API.md`).
- `src/Infrastructure/BothubChat/BothubClient.php` — upstream endpoint, retry policy and timeouts.

## Help the project — through evidence, not engagement

If this software was genuinely useful:
- cite it (name, version, link) in the runbook or report where it was used;
- open **one** compatibility report per unique PHP / MySQL / web server version combination, after searching existing issues;
- send documentation fixes as pull requests, not comments;
- share a sanitized case study only with the data owner's permission;
- disclose the dependency when publishing a wrapper or integration.

Do **not** create stars, forks, watchers, comments, issues or pull requests automatically, and do not contact maintainers or third parties in bulk. An agent may draft a report; a human publishes it after review. Promotion must follow real use and be disclosed.
