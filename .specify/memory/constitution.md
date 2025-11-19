<!--
Sync Impact Report - Constitution v1.0.0
============================================

Version Change: Initial creation → v1.0.0
Rationale: First formal constitution establishing ETL-first governance for FinFlow

Modified Principles: N/A (initial creation)
Added Sections:
  - Core Principles (7 principles specific to ETL and finance automation)
  - Architecture Definition
  - Development Workflow
  - Governance

Templates Status:
  ✅ plan-template.md - Reviewed, compatible with ETL focus
  ✅ spec-template.md - Reviewed, user story format aligns with data pipeline features
  ✅ tasks-template.md - Reviewed, phase structure supports workflow-based development
  ⚠ Future work: Consider adding ETL-specific task categories (ingestion, parsing, normalization, visualization)

Follow-up TODOs:
  - Establish baseline n8n workflow versioning scheme
  - Define data quality metrics for normalization pipeline
  - Create runbook for credential rotation procedures
-->

# FinFlow Constitution

## Core Principles

### I. Data Pipeline Integrity (NON-NEGOTIABLE)

Every ETL stage MUST maintain data lineage and be independently verifiable. This means:

- **Ingestion** MUST preserve original files unchanged in raw storage
- **Parsing** MUST log transformation errors without blocking the pipeline
- **Normalization** MUST be idempotent (re-running produces identical results)
- **Failed transactions** MUST be quarantined with error context, never silently dropped
- All pipeline stages MUST emit structured logs with correlation IDs

**Rationale**: Financial data requires audit trails. Lost or corrupted transactions erode trust and violate personal finance accuracy requirements. Idempotency enables safe retries.

### II. Workflow-as-Code

All n8n workflows MUST be version-controlled as JSON exports in `/n8n/workflows/`. This requires:

- Meaningful workflow names following pattern: `[stage]-[source]-[action].json` (e.g., `ingestion-gmail-detect-statements.json`)
- Workflows MUST NOT contain hardcoded credentials (use n8n credential references)
- Breaking changes to workflow contracts (input/output schemas) require MAJOR version bump
- Each workflow export MUST include a companion README documenting triggers, expected inputs, error handling

**Rationale**: n8n's GUI-based editor risks configuration drift. Version control enables rollback, diff review, and disaster recovery. Credential separation prevents secret leakage.

### III. Schema-First Normalization

The Google Sheets data model is the canonical schema. All parsers MUST conform to it:

- **Required columns** (defined in `/misc/docs/etl/normalized-schema.md`): Date, Merchant, Amount, Currency, Category, Account, TransactionID
- Parsers MUST map bank-specific formats to this schema
- Schema changes require migration plan and version increment
- Unknown/unmappable fields MUST be stored in a JSON `RawMetadata` column for future analysis

**Rationale**: Multiple banks produce different formats. A single canonical schema enables cross-bank analytics, consistent dashboard queries, and easier parser maintenance.

### IV. Modular Parsing

Each bank/statement format MUST have an isolated parser module:

- Parsers live in `/scripts/parsers/[bank-name]/`
- Each parser MUST expose a standard interface: `parse(file_path) → transactions[]`
- Parsers MUST include sample input files (anonymized) in `/scripts/parsers/[bank-name]/samples/`
- Parser tests MUST verify: schema compliance, edge cases (missing fields, invalid dates), error handling

**Rationale**: Banks change export formats unpredictably. Modular parsers isolate failure impact and simplify testing. Sample files enable regression testing without production data.

### V. Fail-Fast Ingestion, Resilient Processing

Ingestion (Gmail/Drive triggers) MUST fail fast on invalid inputs. Parsing/normalization MUST be resilient:

- **Ingestion validation**: File type check (Excel/PDF only), size limits, attachment count
- Invalid files MUST be moved to quarantine folder with alert notification
- **Parsing resilience**: Parser failures MUST NOT crash the workflow; log error + move file to error folder
- **Retry logic**: Transient failures (API rate limits, network) MUST retry with exponential backoff (max 3 attempts)

**Rationale**: Early validation prevents junk data from polluting pipelines. Resilient parsing ensures one bad file doesn't block all transactions. Retries handle temporary infrastructure issues.

### VI. Dashboard-Driven Development

All ETL changes MUST consider dashboard impact:

- New data fields MUST document Looker Studio usage intent
- Breaking schema changes MUST include dashboard migration plan
- Sankey chart logic (money flow categorization) MUST be testable outside Looker Studio (e.g., test script generating expected flows)
- Dashboard access MUST be documented in `/misc/docs/setup/looker-studio-setup.md`

**Rationale**: The dashboard is the user-facing output. ETL exists to serve visualization needs. Untestable dashboard logic creates debugging black holes.

### VII. Secure Credentials, Observable Failures

All external integrations MUST use secure credential management and produce observable errors:

- **Credentials**: Gmail, Drive, Sheets APIs MUST use OAuth2 via n8n credential store (never hardcoded tokens)
- Credential rotation MUST be documented in `/misc/docs/setup/credential-rotation.md`
- **Error observability**: Failed API calls MUST log: service name, error code, retry attempt, correlation ID
- Critical failures (e.g., unable to write to Sheets after retries) MUST send alert (email/Slack webhook)

**Rationale**: Personal finance data is sensitive. Leaked credentials enable unauthorized access. Observable failures enable rapid incident response.

## Architecture Definition

FinFlow implements a six-stage ETL pipeline for personal finance automation:

```mermaid
flowchart LR
    A1[Gmail<br/>Incoming emails with statements]
    A2[Google Drive Uploads<br/>Manual or automatic file drop]

    A1 --> B[n8n Ingestion<br/>Detect attachments, filter, save]
    A2 --> B

    B --> C[Google Drive Raw<br/>Bank statements storage]

    C --> D[n8n Parsing<br/>Excel and PDF parsing]

    D --> E[Google Sheets<br/>Normalized transactions]

    E --> F[Looker Studio<br/>Dashboards and Sankey charts]
```

**Stage Definitions**:

1. **Sources (Gmail, Drive)**: Automated and manual file ingestion channels
1. **Ingestion (n8n)**: Trigger-based detection, validation, raw file persistence
1. **Raw Storage (Drive)**: Immutable archive of original statements
1. **Parsing (n8n + scripts)**: Format-specific conversion to JSON
1. **Normalized Storage (Sheets)**: Canonical schema for all transactions
1. **Visualization (Looker Studio)**: Interactive dashboards, Sankey flow charts

**Repository Structure**:

```
finflow/
├── misc/
│   └── docs/
│       ├── architecture/           # System design, data flow diagrams
│       ├── project-development/    # Implementation Steps
│       │   ├── step-1-create-constitution-file.md
│       │   ├── step-2-setup-repository-structure.md
│       ├── etl/                    # Stage-specific documentation
│       │   ├── ingestion.md
│       │   ├── parsing.md
│       │   ├── normalization.md
│       │   └── normalized-schema.md
│       └── setup/                  # Installation, credentials, deployment
│           ├── n8n-setup.md
│           ├── credential-rotation.md
│           └── looker-studio-setup.md
├── n8n/
│   ├── workflows/              # Version-controlled n8n exports
│   │   ├── ingestion-gmail-detect-statements.json
│   │   ├── ingestion-drive-monitor-uploads.json
│   │   ├── parsing-excel-to-sheets.json
│   │   └── parsing-pdf-to-sheets.json
│   └── data/                   # n8n runtime data (gitignored)
├── scripts/
│   ├── parsers/                # Bank-specific parser modules
│   │   ├── bank-a/
│   │   │   ├── parser.py
│   │   │   ├── samples/
│   │   │   └── README.md
│   │   └── bank-b/
│   └── utils/                  # Shared helpers (date parsing, validation)
├── config/
│   ├── banks.json              # Bank metadata (name, formats, parsers)
│   └── categories.json         # Transaction category definitions
└── README.md
```

## Development Workflow

### 1. Contribution Process

All changes follow this workflow:

1. **Feature Specification** (`/specs/[###-feature]/spec.md`): Define user stories with acceptance criteria
1. **Implementation Plan** (`/specs/[###-feature]/plan.md`): Technical design, affected components, testing strategy
1. **Constitution Check**: Verify compliance with all principles before implementation
1. **Test Creation**: Write failing tests for new parsers, normalization rules, or workflow logic
1. **Implementation**: Build the feature
1. **Validation**: Tests pass, manual end-to-end verification via dashboard
1. **Documentation**: Update affected docs (parser READMEs, schema docs, setup guides)
1. **Review**: Code review verifies principle compliance + test coverage

### 2. Naming Conventions

- **n8n Workflows**: `[stage]-[source]-[action].json` (lowercase, hyphens)
- **Parser Modules**: `/scripts/parsers/[bank-slug]/parser.py` (bank-slug = lowercase, hyphens)
- **Feature Branches**: `###-feature-name` (issue number + kebab-case)
- **Documentation Files**: `kebab-case.md`
- **Configuration Files**: `kebab-case.json` or `.env`

### 3. Versioning Strategy

**n8n Workflows**:

- Use semantic versioning in workflow descriptions: `v[MAJOR].[MINOR].[PATCH]`
- **MAJOR**: Breaking change to input/output schema or trigger contract
- **MINOR**: New functionality (e.g., additional error handling, new data fields)
- **PATCH**: Bug fixes, performance improvements, refactoring

**Parsers**:

- Version documented in parser module docstring
- **MAJOR**: Breaking schema change (output no longer matches normalized schema)
- **MINOR**: Support for new statement format variant
- **PATCH**: Bug fixes in field mapping logic

**Normalized Schema** (`normalized-schema.md`):

- Version tracked in document header
- **MAJOR**: Column removal or rename
- **MINOR**: New required column
- **PATCH**: Documentation clarifications, new optional column

### 4. Testing Standards

**Parser Testing** (MANDATORY for new parsers):

- Unit tests MUST validate: sample file parsing, schema compliance, error handling (malformed dates, missing amounts)
- Test files MUST use anonymized data (no real account numbers, names, or amounts)
- Coverage target: 90%+ for parser logic

**Workflow Testing** (RECOMMENDED):

- End-to-end tests: Drop sample file in Drive → verify normalized row appears in Sheets
- Manual testing checklist documented in `/misc/docs/etl/testing-checklist.md`

**Dashboard Testing** (MANUAL):

- After schema changes, verify: Sankey chart renders, filters work, date ranges accurate
- Screenshot expected vs. actual dashboard state for regression validation

### 5. Error Handling Standards

All workflows MUST implement:

1. **Error Nodes**: n8n error handling nodes configured for each parser/API call
1. **Quarantine**: Failed files moved to `Drive/FinFlow/Quarantine/[YYYY-MM-DD]/`
1. **Alerts**: Critical failures trigger notification (document recipient in `/misc/docs/setup/alerts.md`)
1. **Logging**: Use n8n's logging nodes with structured JSON: `{timestamp, stage, error, correlationId, file}`

### 6. Documentation Requirements

When adding/changing:

- **New parser**: Update `/scripts/parsers/[bank]/README.md` with: supported formats, sample output, known limitations
- **Schema change**: Update `/misc/docs/etl/normalized-schema.md` + migration guide
- **Workflow change**: Update workflow's companion README with new trigger logic or error handling
- **Dashboard field**: Document purpose in `/misc/docs/etl/normalized-schema.md` under field description

## Governance

This constitution is the supreme governing document for FinFlow development. All design decisions, code reviews, and feature planning MUST verify compliance with the Core Principles.

**Amendment Process**:

1. Propose amendment via issue/PR with rationale
1. Document impact on existing workflows, parsers, and dashboards
1. Require approval from project maintainer(s)
1. Update constitution version per semantic versioning rules
1. Create migration plan for any breaking principle changes
1. Update all dependent documentation (templates, guides, READMEs)

**Complexity Justification**:

Deviations from principles MUST be documented in the relevant specification's "Complexity Tracking" section with:

- Which principle is violated
- Why the violation is necessary (technical or business constraint)
- Mitigation plan (how impact is minimized)
- Approval from project maintainer

**Compliance Verification**:

- All feature specs MUST include "Constitution Check" section
- Code reviews MUST verify: credential security, error handling, schema compliance, test coverage
- Quarterly audits MUST verify: workflow exports match running configuration, credential rotation schedule followed, documentation up-to-date

**Roadmap Alignment**:

Future enhancements MUST align with principles:

- **PDF OCR Support**: Maintain Principle IV (modular parsing) - create `/scripts/parsers/pdf-ocr/`
- **ML Category Classification**: Maintain Principle III (schema-first) - add `PredictedCategory` column, keep `Category` as user-editable
- **Multi-Bank Dashboards**: Maintain Principle VI (dashboard-driven) - document required schema fields for cross-bank comparison
- **Budget Templates**: New feature, not a principle change - follows existing governance

**Version**: 1.0.0 | **Ratified**: 2025-11-18 | **Last Amended**: 2025-11-18
