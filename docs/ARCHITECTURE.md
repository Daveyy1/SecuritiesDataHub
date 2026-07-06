# Architecture: Core Backend

## Style

Hexagonal Architecture (Ports & Adapters).

- Full domain/persistence separation applies where business logic exists (`Report`).
- Plain CRUD without business rules (`Bank`) uses the JPA entity directly — no full separation needed there.

## Package structure

```
com.securitiesdatahub.core
├── domain
│   ├── model          (Bank, Security, Report – plain Java, no JPA annotations)
│   └── service         (validation rules: IsinValidator, ReportPlausibilityChecker)
├── application
│   ├── port
│   │   ├── in           (CreateReportUseCase, FindReportsQuery)
│   │   └── out          (SaveReportPort, LoadBankPort, LoadSecurityPort)
│   └── service          (CreateReportService implements CreateReportUseCase)
└── adapter
    ├── in
    │   ├── rest          (ReportController)
    │   └── messaging      (KafkaReportConsumer) [Phase 2]
    └── out
        └── persistence    (JpaReportRepository, ReportMapper)
```

## Domain model

- `Bank` — id, name, bic. No business rules.
- `Security` — isin, issuerName, type. `Isin` is a validated value type (checksum enforced on construction).
- `Report` — id, reportingBank (`BankId`), security (`Isin`), holdings, valuation, reportingDate. References other entities by ID, not by object.
  - Structural invariants enforced on construction: non-negative holdings, reporting date not in the future.
  - Context-dependent plausibility checks (e.g. large deviation from prior period) live in `ReportPlausibilityChecker`, separate from construction-time validation.
- ID types (`BankId`, `ReportId`, etc.) are distinct value types, not raw `UUID`/`Long`.
- `Holdings` and `Valuation` are `BigDecimal`-based wrapper types, not raw `BigDecimal` fields.

## Open decisions / next steps

- [ ] Define ports: `SaveReportPort`, `LoadReportPort`, `CreateReportUseCase`, `FindReportsQuery`
- [ ] `ReportPlausibilityChecker` interface shape
- [ ] REST adapter DTOs (kept separate from domain records)
- [ ] Persistence adapter: JPA entities + `ReportMapper`
