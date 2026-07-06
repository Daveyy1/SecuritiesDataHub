# CLAUDE.md

This file governs how Claude Code works on this project. It describes **how** work is done — **what** is being built lives in `docs/PROJECT_PLAN.md`.

## Project context

Portfolio project "Securities Data Hub": a platform for collecting, validating, storing, and analyzing securities reports (banks → supervisory authority). Goal: demonstrate the skills needed for a role as an application developer in a data-intensive, central-bank-like environment.

Before starting any new task: read `docs/PROJECT_PLAN.md` to understand the overall scope and current phase. For the core backend specifically, also read `docs/ARCHITECTURE.md` — it defines the hexagonal architecture (ports & adapters), the domain model (`Bank`, `Security`, `Report` and their invariants), and the reasoning behind key type choices (e.g. `BigDecimal` wrappers over raw primitives). In case of conflicts between this file and either document: ask briefly rather than guessing.

## Current status

> This section is updated continuously. After each completed step: record the current status here (active phase, last completed milestone, open items).

- **Active phase:** Phase 1 – Data model & core backend
- **Last milestone:** –
- **Open items:** –

## Tech stack & conventions

- **Backend:** Java 21, Spring Boot 4.1.0, Spring Data JPA, PostgreSQL
- **Analysis layer:** Python 3.14, pandas, numpy, FastAPI (if run as a service, not a script)
- **Build:** Maven for the Java parts
- **Java style:** Records for DTOs where possible, no Lombok, constructor injection instead of field injection
- **Python style:** Type hints mandatory, black formatting
- **Tests:** JUnit 5 + Mockito for Java, pytest for Python — at least one test case for every new business rule (validation, calculation)
- **Database:** Migrations via Flyway, no manual schema changes

## Workflow rules

1. **Before any implementation:** briefly state which phase/section of `docs/PROJECT_PLAN.md` the change relates to.
2. **Tests first** for business logic (validation rules, calculations). Plain CRUD endpoints don't need strict TDD.
3. **After each completed step:**
   - Update the "Current status" section above
   - Create a meaningful commit message for my review, only I will commit changes
4. **Architecture decisions** (new library, new pattern, deviation from the plan): justify before implementing — don't deviate silently.
5. **Don't do without asking:**
   - Add new dependencies
   - Change established conventions (package structure, naming scheme)
   - Expand scope beyond the current phase
   - Commit anything
6. When business requirements are unclear (e.g. what "plausible holdings" means): make a sensible assumption, document it explicitly, don't ask endlessly.

## Project structure (target)

```
project/
├── CLAUDE.md
├── docs/
│   ├── PROJECT_PLAN.md
│   └── ARCHITECTURE.md
├── core-backend/        (Spring Boot)
├── ingestion/            (Python, validation + publishing)
├── analysis-service/     (Python, metrics)
└── docker-compose.yml
```

## Don't forget

- Always comment business validation rules with their rationale (interview-worthy!)
- For decisions between multiple options (e.g. Kafka vs. Airflow): record a brief comparison as a comment or in `docs/decisions/` — this simulates the procurement process from the job posting
