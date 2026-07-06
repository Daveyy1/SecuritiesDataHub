# Project Proposal: "Securities Data Hub" – Mini Platform for Securities Data

**Goal:** A portfolio project that specifically covers the core requirements of the SNB "Expert Application Developer" role and can serve as concrete evidence in the job interview.

---

## 1. Why this project?

Direct mapping to the job posting:

| Requirement from the posting | How the project covers it |
|---|---|
| In-house development in Java (Spring Boot) and Python | Backend in Spring Boot, analysis layer in Python |
| Design of data-driven applications | End-to-end pipeline: collection → processing → analysis |
| Integration of third-party software | Integrating an existing tool (e.g. Kafka or Airflow) instead of building it yourself |
| Handling granular data (e.g. securities data) | Data model for securities/portfolios as a central theme |
| Stable operations & maintenance | Containerization, CI/CD, logging/monitoring |
| Business requirements → technical specification | Included fictional requirements document + architecture decision |
| Explaining things clearly to non-technical business staff | Non-technical companion document ("business unit guide") |

---

## 2. Scenario (fictional but realistic)

You take on the role of the application development team at a (fictional) supervisory authority that periodically receives securities reports from banks (position data: ISIN, holdings, valuation, reporting date, reporting bank), validates and stores them, and makes them available to data scientists/economists for analysis.

This almost literally covers the introductory text of the posting: *"collection, processing, and analysis of extensive data from banks and companies."*

---

## 3. Architecture

```
Banks (simulated)
   │  CSV/JSON reports
   ▼
[Ingestion Service, Python]
   - Validation (schema, plausibility, duplicates)
   - Normalization
   │
   ▼
[Message Queue / Third-party software: Kafka or simpler: Airflow batch job]
   │
   ▼
[Core Backend, Spring Boot]
   - REST API (reports, queries, aggregations)
   - Persistence via Spring Data JPA → PostgreSQL
   - Business validation rules (e.g. ISIN format, valuation plausibility)
   │
   ▼
[Analysis Service, Python: pandas/numpy]
   - Retrieves data from the core backend via REST
   - Computes metrics (portfolio concentration, value changes over time, outlier detection)
   - Returns results as JSON or writes to its own analysis table
   │
   ▼
[Dashboard, simple: Grafana/Metabase OR minimal React UI]
   - Intended for "business units": metrics, no SQL knowledge required
```

**Deliberate architecture decision you can explain in the interview:**
Why separate ingestion/core/analysis? → Separation of concerns, independent scalability, different teams (data engineers vs. data scientists) can work independently — exactly the target picture of "application development works closely with data scientists and economists."

---

## 4. Concrete implementation steps

### Phase 1 – Data model & core backend (Spring Boot)
- Entities: `Bank`, `Security` (ISIN, issuer, type), `Report` (holdings, valuation, reporting date, bank FK, security FK)
- Spring Data JPA repositories, PostgreSQL
- REST endpoints: create report, filter reports by bank/period, aggregation (total holdings per ISIN)
- Business validation: ISIN checksum (Luhn-like), required fields, valuation plausibility (e.g. no negative holdings)
- Bonus: simple role concept (bank user can only see their own data, analyst sees everything)

### Phase 2 – Ingestion & third-party software integration
- Deliberately choose **one** integration and document the selection process like a mini procurement:
  - Compare options (e.g. Kafka vs. RabbitMQ vs. a simple scheduler with Airflow)
  - Define criteria: operational effort, scalability, learning curve, fit to the use case
  - Justify the decision and record it in a short (1-page) "evaluation document"
- Actually build the integration: e.g. a Python script reads CSV reports, validates the schema, publishes to a Kafka topic, a Spring Boot consumer writes to the DB

### Phase 3 – Analysis layer (Python)
- Python service (FastAPI or a simple scheduled script) that accesses the core backend via REST
- Calculations: portfolio concentration per bank (Herfindahl index), time-series analysis of holdings, simple outlier detection (e.g. Z-score) for unusual reports
- Return results to the backend or write to a dedicated table

### Phase 4 – Operations & maintenance
- Docker Compose for all components (backend, DB, Kafka, analysis service)
- GitHub Actions: build, tests, linting on every push
- Logging (structured, e.g. with Logback) and a simple health-check endpoint
- Short runbook: "What to do if the ingestion service fails"

### Phase 5 – Communication for business units
- Write a **non-technical** 1-2 page document explaining:
  - What the application does (no code, no jargon like "REST" or "JPA")
  - How an economist uses the analysis results
  - What data quality rules apply and why
- This simulates exactly the requirement "target-group-appropriate advice and support for business units"

---

## 5. Data source

Since real bank reports aren't available, two options:
1. **Generate synthetic data** (recommended, fastest to start): a Python script generates plausible ISIN codes, banks, and holdings across multiple reporting periods
2. **Public reference data** as a basis: e.g. the SNB statistics portal (time series on securities holdings) for realistic magnitudes — this also shows a connection to the SNB

---

## 6. Timeline (proposal)

| Week | Focus |
|---|---|
| 1 | Data model, Spring Boot core backend, first REST endpoints |
| 2 | Validation, third-party software integration (Kafka/Airflow) |
| 3 | Python analysis layer, first metrics |
| 4 | Docker/CI-CD, dashboard, non-technical companion document, polish |

---

## 7. What you can show afterward in the interview

- GitHub repo with a clean README, architecture diagram, setup instructions
- Well-justified technology decisions (procurement mindset)
- Both code (Java + Python) and a "business-friendly" document
- Awareness of operations, not just prototype thinking

---

*Next step: I can set up the data model for Phase 1 directly with you (entities + initial Spring Boot project structure), if you'd like.*
