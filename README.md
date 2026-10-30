# Runbook-Following Agent (PS-03)

A Spring Boot application that troubleshoots operational incidents by
automatically identifying and following predefined runbooks, step by
step, until the incident is **RESOLVED** or **ESCALATED**.

## Tech Stack

- Java 21
- Spring Boot 3.3.4 (Spring Web, Spring Data JPA, Thymeleaf, Validation)
- Maven
- MySQL 8.0
- Thymeleaf + HTML + CSS + vanilla JavaScript
- JUnit 5 (with an in-memory H2 database for tests only)

## How Runbook Matching Works (AI Requirement)

The system works **completely without any external AI API**:

- `ai.mode=mock` (the default) uses `MockRunbookMatcher`, a deterministic,
  keyword-based matcher. Each runbook has a `matchKeywords` field
  (comma-separated phrases); the matcher scores each runbook against the
  incoming problem description and picks the best match.
- `ai.mode=real` activates `AiRunbookMatcher`, which calls an
  OpenAI-compatible chat completions API using an API key from the
  `AI_API_KEY` environment variable. If the key is missing or the call
  fails, it safely falls back to "no match" — it never crashes the app.
- No API key is hardcoded anywhere. The app is 100% usable in Mock Mode.

## Sample Runbooks (auto-loaded on first run)

`DataSeeder` automatically inserts these 3 runbooks the first time the
app starts against an empty database:

1. **Application Down** — keywords: "application is not responding", "application down", "app crash", ...
2. **Database Connection Failure** — keywords: "database connection failed", "db down", ...
3. **High CPU Usage** — keywords: "cpu is very high", "high cpu", "cpu spike", ...

You can also run `database/data.sql` manually for the same effect.

## Project Structure

```
runbook-following-agent/
├── pom.xml
├── README.md
├── database/
│   ├── schema.sql      (run this first in MySQL Workbench)
│   └── data.sql        (optional — sample data, also auto-seeded by the app)
└── src/
    ├── main/java/com/runbookagent/
    │   ├── RunbookAgentApplication.java
    │   ├── config/       (AppConfig, DataSeeder)
    │   ├── entity/       (Runbook, RunbookStep, Incident, ExecutionLog, enums)
    │   ├── repository/   (Spring Data JPA repositories)
    │   ├── dto/          (request/response objects)
    │   ├── exception/    (custom exceptions + global handler)
    │   ├── ai/           (RunbookMatcher, MockRunbookMatcher, AiRunbookMatcher, ActionSimulator)
    │   ├── service/      (RunbookService, IncidentService, DashboardService)
    │   └── controller/   (REST controllers + Thymeleaf ViewController)
    ├── main/resources/
    │   ├── application.properties
    │   ├── templates/    (dashboard, runbooks, runbook-form, runbook-details,
    │   │                  incidents, incident-create, incident-execute, incident-history)
    │   └── static/css, static/js
    └── test/java/com/runbookagent/  (JUnit 5 tests, run against in-memory H2)
```

## REST API

| Method | Endpoint                         | Purpose                                  |
|--------|-----------------------------------|-------------------------------------------|
| POST   | /api/runbooks                    | Create a runbook                          |
| GET    | /api/runbooks                    | List all runbooks                         |
| GET    | /api/runbooks/{id}                | Get one runbook (with steps)             |
| PUT    | /api/runbooks/{id}                | Update a runbook                          |
| DELETE | /api/runbooks/{id}                | Delete a runbook                          |
| POST   | /api/incidents                    | Create an incident (auto-matches runbook) |
| GET    | /api/incidents                    | List all incidents                        |
| GET    | /api/incidents/{id}                | Get one incident                         |
| POST   | /api/incidents/{id}/start          | Start/resume execution                   |
| POST   | /api/incidents/{id}/result         | Record SUCCESS/FAILED for current step   |
| GET    | /api/incidents/{id}/history        | Get the execution log timeline           |
| GET    | /api/dashboard                    | Get dashboard counts                      |

## Web Pages

- `/` — Dashboard
- `/runbooks` — Runbook list
- `/runbooks/new` — Create runbook
- `/runbooks/{id}` — Runbook details
- `/runbooks/{id}/edit` — Edit runbook
- `/incidents` — Incident list
- `/incidents/new` — Report a new incident
- `/incidents/{id}/execute` — Step-by-step execution screen
- `/incidents/{id}/history` — Execution history timeline

See the setup guide provided alongside this project for exact
installation, configuration, and demo instructions.
