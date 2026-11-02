# Pinwheel — Software Requirements Specification

![status](https://img.shields.io/badge/status-draft%20for%20review-orange?style=flat-square)
![standard](https://img.shields.io/badge/based%20on-ISO%2FIEC%2FIEEE%2029148-informational?style=flat-square)

> **This is a strawman.** Every requirement below is a proposal to be challenged, rewritten or deleted.
> Values marked **TBD** are assumptions that must be confirmed before the requirement is baselined.

| Field | Value |
|---|---|
| Document version | 0.1 (draft) |
| Date | 2026-11-02 |
| Owner | Eduardo Bizzi Martins |
| Related docs | `docs/vision.md` (pending), `docs/adr/` (pending), `docs/v2-possible-features.md` |

---

## 1. Introduction

### 1.1 Purpose

This document lists the functional and non-functional requirements of **Pinwheel**, an open-source web application that ingests Playwright test reports and lets users analyze test execution history. It is the input for the backlog (user stories), the architecture (Phase 4) and the test plan (Phase 7).

### 1.2 Scope

Pinwheel receives Playwright JSON reports (manually or from CI), stores them, and shows run-level and test-level execution history in a data table. Later releases add insights (flakiness, failure clustering, duration anomalies), charts and CI integrations.

### 1.3 Definitions

| Term | Meaning |
|---|---|
| **Report** | The JSON produced by Playwright's `json` reporter for one execution of a suite. |
| **Run** | One ingested report plus its run metadata (commit, branch, CI run…). |
| **Run metadata** | Context not present in the Playwright report: repository, commit SHA, branch, CI event, CI run ID, run attempt, CI run URL. |
| **Test** | One `spec` executed in one Playwright project (browser). |
| **Attempt** | One execution of a test inside a run (`retry` 0, 1, 2…). |
| **Outcome** | Playwright's overall status of a test in a run: `expected`, `unexpected`, `flaky`, `skipped`. |
| **Stable test ID** | An identifier that matches the same test across runs (candidate: file + title path + project name). |
| **Raw report** | The report stored exactly as received, without transformation. |

### 1.4 References

- ISO/IEC/IEEE 29148:2018 — *Systems and software engineering — Life cycle processes — Requirements engineering*
- ISO/IEC 25010:2023 — *Product quality model* (used to classify non-functional requirements)
- Playwright JSON reporter documentation
- Pinwheel project context, V2 possible features, the-internet-playwright data source context

---

## 2. How to read this document

### 2.1 Release badges

| Badge | Meaning |
|---|---|
| ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | **MVP / university deliverable.** Upload or POST a report, store it, browse run and test history in a table. |
| ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | **Insights.** Charts and trends, cross-run flakiness, failure clustering, duration anomalies, automatic reporter. |
| ![V3](https://img.shields.io/badge/V3-8250df?style=flat-square) | **Integrations and scale.** CI/chat integrations, user accounts, multi-team use, test prioritization. |

### 2.2 Requirement attributes

Each requirement has the attributes recommended by ISO/IEC/IEEE 29148 (§5.2.8), trimmed for a solo project:

| Attribute | Values |
|---|---|
| **ID** | `FR-<AREA>-NN` (functional) or `NFR-<CHARACTERISTIC>-NN` (non-functional). IDs are never reused. |
| **Release** | V1, V2, V3 (badge). |
| **Priority** | MoSCoW **within its release**: Must, Should, Could. Won't-this-time items move to a later release. |
| **Verification** | **T** = Test, **D** = Demonstration, **I** = Inspection, **A** = Analysis. |
| **Traces to** | Stakeholder need(s) from §3. Later: user story and test case IDs (see §8). |

### 2.3 Wording conventions (29148 §5.2.4)

- **shall** = binding requirement. **should** = goal. **may** = option. **will** = statement of fact.
- One requirement per statement (*singular*). Avoid vague terms: *fast*, *easy*, *user-friendly*, *etc.*, *and/or*, *support* without saying how.
- Requirements say **what**, not **how**. Technology choices live in §6 (Constraints) and in ADRs.

### 2.4 Quality checklist for each requirement (29148 §5.2.5)

Before baselining, each requirement should be: **necessary**, **appropriate** (right level of detail), **unambiguous**, **complete**, **singular**, **feasible**, **verifiable**, **correct**, and **conforming** to the template. The set as a whole should be **complete**, **consistent**, **feasible**, **comprehensible** and **able to be validated** (§5.2.6).

---

## 3. Stakeholders and needs

| ID    | Stakeholder          | Need                                                                                         |
| ----- | -------------------- | -------------------------------------------------------------------------------------------- |
| SN-01 | QA engineer          | Know whether a failure is new, recurring or intermittent without opening reports one by one. |
| SN-02 | Developer            | Find which tests failed on a given branch or commit, and why.                                |
| SN-03 | Tech lead            | See the health of the suite over time (pass rate, duration, flakiness).                      |
| SN-04 | CI pipeline (system) | Send a report without manual steps, and without breaking the pipeline if Pinwheel is down.   |
| SN-05 | Forker / maintainer  | Deploy, configure and extend Pinwheel without editing core code.                             |
| SN-06 | University evaluator | Traceable engineering artifacts (requirements → design → tests).                             |

---

## 4. Functional requirements

### 4.1 Report ingestion (ING)

| ID        | Release                                                         | Requirement                                                                                                                                                                                                | Priority | Verif. | Traces to           |
| --------- | --------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------ | ------------------- |
| FR-ING-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall accept a Playwright JSON report through an HTTP API.                                                                                                                                      | Must     | T      | SN-04               |
| FR-ING-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall allow a user to upload a Playwright JSON report file through the web UI.                                                                                                                  | Must     | D      | SN-01, SN-02        |
| FR-ING-03 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall accept optional run metadata (repository, commit SHA, branch, CI event, CI run ID, run attempt, CI run URL) together with a report.                                                       | Must     | T      | SN-02, SN-03        |
| FR-ING-04 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall reject a report that is not valid Playwright JSON reporter output, and the response shall identify the reason for rejection.                                                              | Must     | T      | SN-04               |
| FR-ING-05 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall store every accepted report unmodified (raw report), together with its run metadata and the time it was received. *Rationale: the data model can be rebuilt from raw data if it changes.* | Must     | T      | SN-05               |
| FR-ING-06 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall detect a report whose CI run ID and run attempt match an already stored run, and shall not create a duplicate run.                                                                        | Should   | T      | SN-03               |
| FR-ING-07 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall import the reports already stored by `the-internet-playwright` (`pinwheel.raw_reports`) as runs.                                                                                          | Should   | D      | SN-06               |
| FR-ING-08 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The project shall provide a Playwright reporter that sends the report and run metadata to the system at the end of a test execution.                                                                       | Must     | D      | SN-04               |
| FR-ING-09 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The project shall modularize ingestion to allow for multiple projects.                                                                                                                                     | Must     | D      | SN-01, SN-02, SN-03 |
| FR-ING-10 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The project shall limit each project to one report source.                                                                                                                                                 | Must     | T      | SN-01, SN-02, SN-03 |
| FR-ING-11 | ![V3](https://img.shields.io/badge/V3-8250df?style=flat-square) | The project shall provide a reusable GitHub Actions step that sends the report to the system.                                                                                                              | Could    | D      | SN-04               |

### 4.2 Report processing (PRC)

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| FR-PRC-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall extract, from each accepted report, one record per test per attempt, including nested `describe` suites. | Must | T | SN-01, SN-02 |
| FR-PRC-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall assign each test a stable test ID that is identical across runs while the test's file, title path and project name do not change. | Must | T | SN-01, SN-03 |
| FR-PRC-03 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall record, per attempt: status, duration, start time, retry number, error messages and stack traces. | Must | T | SN-01, SN-02 |
| FR-PRC-04 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall record, per test per run, the Playwright outcome (`expected`, `unexpected`, `flaky`, `skipped`). | Must | T | SN-01 |
| FR-PRC-05 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall be able to rebuild all processed data from the stored raw reports. | Should | D | SN-05 |

### 4.3 Run history (RUN)

| ID        | Release                                                         | Requirement                                                                                                                                                                 | Priority | Verif. | Traces to    |
| --------- | --------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------ | ------------ |
| FR-RUN-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall display a table of runs showing: received time, branch, commit SHA, CI event, total duration, and counts of expected, unexpected, flaky and skipped tests. | Must     | D      | SN-02, SN-03 |
| FR-RUN-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall allow the runs table to be sorted by any displayed column.                                                                                                 | Must     | T      | SN-03        |
| FR-RUN-03 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall allow the runs table to be filtered by date range, branch and run result (passed / failed).                                                                | Should   | T      | SN-02        |
| FR-RUN-04 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall link each run to its CI run URL when that URL was provided.                                                                                                | Should   | T      | SN-02        |
| FR-RUN-05 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall display, for a selected run, the list of its tests with outcome, duration and retry count.                                                                 | Must     | D      | SN-01, SN-02 |
| FR-RUN-06 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall allow a user to delete a run.                                                                                                                              | Could    | T      | SN-05        |
| FR-RUN-07 |                                                                 | The system shall allow a user to ignore a run                                                                                                                               | Must     | T      | SN-05        |

### 4.4 Test history (TST)

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| FR-TST-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall display the execution history of a single test across runs, ordered by run time. | Must | D | SN-01 |
| FR-TST-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall display the error messages and stack traces of every failed attempt. | Must | D | SN-01, SN-02 |
| FR-TST-03 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall allow test results to be filtered by outcome, spec file, project name and title text. | Should | T | SN-01, SN-02 |
| FR-TST-04 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall display each test's pass rate over a user-selected number of recent runs. | Should | T | SN-03 |
| FR-TST-05 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall allow the displayed table data to be exported as CSV. | Could | T | SN-03 |

### 4.5 Insights and trends (INS)

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| FR-INS-01 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall display, per test, a chart of duration over time. | Must | D | SN-03 |
| FR-INS-02 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall display a chart of run results (expected / unexpected / flaky / skipped) over time. | Must | D | SN-03 |
| FR-INS-03 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall compute a cross-run flakiness score per test and display it as a sortable column. | Should | T, A | SN-01 |
| FR-INS-04 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall group the failed attempts of a run by similar error message. | Should | T, A | SN-01 |
| FR-INS-05 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall flag a test whose duration in a run deviates from its recent history by more than a configurable threshold. | Could | T, A | SN-03 |
| FR-INS-06 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall tell the user when there is not enough history for an insight to be computed, instead of showing a value. | Must | T | SN-01, SN-03 |
| FR-INS-07 | ![V3](https://img.shields.io/badge/V3-8250df?style=flat-square) | The system shall suggest an execution order of tests based on their likelihood of failing for a given set of changed files. | Could | A | SN-02 |

### 4.6 Projects, access and integrations (ADM)

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| FR-ADM-01 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The system shall keep runs from different repositories separate and allow the user to select which repository to view. | Should | T | SN-03, SN-05 |
| FR-ADM-02 | ![V3](https://img.shields.io/badge/V3-8250df?style=flat-square) | The system shall require users to sign in before viewing data. | Should | T | SN-05 |
| FR-ADM-03 | ![V3](https://img.shields.io/badge/V3-8250df?style=flat-square) | The system shall send a notification to a configured chat channel when a run has unexpected failures. | Could | D | SN-01 |
| FR-ADM-04 | ![V3](https://img.shields.io/badge/V3-8250df?style=flat-square) | The system shall display Playwright attachments (screenshots, traces) of failed attempts. | Could | D | SN-01 |

---

## 5. Non-functional requirements

Grouped by the ISO/IEC 25010:2023 quality characteristics. Characteristics with no requirement yet are listed as *none identified* so the gap is visible, not forgotten.

### 5.1 Performance efficiency (PERF)

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| NFR-PERF-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The runs table shall display its first page within **2 s (TBD)** with **10,000 (TBD)** stored runs, on the reference deployment (§6). | Should | T | SN-02 |
| NFR-PERF-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall accept and store a report of up to **10 MB (TBD)** within **5 s (TBD)**. | Should | T | SN-04 |
| NFR-PERF-03 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall reject reports larger than a configurable maximum size (default **10 MB, TBD**). *Rationale: protects the server and respects MongoDB's 16 MB document limit.* | Must | T | SN-05 |

### 5.2 Compatibility (COMP)

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| NFR-COMP-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall accept reports produced by Playwright versions **≥ 1.4x (TBD)**, and the supported range shall be documented. | Must | T, I | SN-04 |
| NFR-COMP-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The ingestion API shall be usable from any CI provider with a standard HTTP client (e.g. `curl`), without a provider-specific SDK. | Must | D | SN-04 |
| NFR-COMP-03 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The web UI shall work on the latest two versions of Chrome, Firefox and Safari (desktop). | Should | T | SN-01 |

### 5.3 Interaction capability, formerly usability (USE)

| ID         | Release                                                         | Requirement                                                                                                                        | Priority | Verif. | Traces to    |
| ---------- | --------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | -------- | ------ | ------------ |
| NFR-USE-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | A new user shall be able to deploy the system locally and ingest a first report by following the README in **≤ 15 minutes (TBD)**. | Should   | D      | SN-05, SN-06 |
| NFR-USE-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | Every ingestion error shown to the user shall state what was wrong and what to do next.                                            | Should   | I      | SN-01, SN-04 |
| NFR-USE-03 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The web UI shall conform to WCAG 2.2 level AA.                                                                                     | Should   | T, I   | SN-01        |

### 5.4 Reliability (REL)

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| NFR-REL-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | A report that fails ingestion shall leave no partially stored data. | Must | T | SN-03 |
| NFR-REL-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | A failure in report processing (FR-PRC) shall not cause the raw report to be lost. | Must | T | SN-05 |
| NFR-REL-03 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The provided reporter (FR-ING-08) shall not change the exit code of the Playwright run when the system is unreachable. | Must | T | SN-04 |
| NFR-REL-04 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The project shall document a backup and restore procedure for the database. | Should | I, D | SN-05 |

### 5.5 Security (SEC)

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| NFR-SEC-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The ingestion API shall reject requests that do not carry a valid ingestion token. | Must | T | SN-05 |
| NFR-SEC-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | Secrets (database connection string, ingestion token) shall be provided through environment variables and shall not be stored in the repository. | Must | I | SN-05 |
| NFR-SEC-03 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | Content from reports (test titles, error messages) shall be escaped before being rendered in the UI. *Rationale: error messages can contain HTML.* | Must | T | SN-01 |
| NFR-SEC-04 | ![V3](https://img.shields.io/badge/V3-8250df?style=flat-square) | The system shall restrict data access by user role. | Could | T | SN-05 |

### 5.6 Maintainability (MNT)

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| NFR-MNT-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | Ingestion, processing and presentation shall be separate modules with documented interfaces, so that an analysis component (V2) can be added without modifying ingestion. | Must | I | SN-05 |
| NFR-MNT-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | Backend line coverage by automated tests shall be **≥ 70 % (TBD)**. | Should | A | SN-06 |
| NFR-MNT-03 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | Every change to the main branch shall pass lint, format and test checks in CI. | Must | I | SN-05, SN-06 |
| NFR-MNT-04 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | Each significant architecture decision shall be recorded as an ADR in `docs/adr/`. | Must | I | SN-06 |
| NFR-MNT-05 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The HTTP API shall be described by an OpenAPI document kept in the repository and consistent with the implementation. | Must | I, T | SN-04, SN-05 |
| NFR-MNT-06 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The system shall retain the per-attempt data required by the V2 insights (FR-PRC-02..04) for every ingested run. *Rationale: history cannot be recreated later.* | Must | I | SN-01, SN-03 |

### 5.7 Flexibility, formerly portability (FLX) — customizability and forkability

| ID | Release | Requirement | Priority | Verif. | Traces to |
|---|---|---|---|---|---|
| NFR-FLX-01 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The whole system, including the database, shall start locally with a single documented command. | Must | D | SN-05 |
| NFR-FLX-02 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | All deployment-specific settings shall be configurable through environment variables, each documented with its default value. | Must | I | SN-05 |
| NFR-FLX-03 | ![V1](https://img.shields.io/badge/V1-2ea44f?style=flat-square) | The source code shall be published under an OSI-approved open-source license. | Must | I | SN-05 |
| NFR-FLX-04 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The project shall provide a customization guide that lists the extension points (e.g. table columns, stable test ID strategy, insights job). | Should | I | SN-05 |
| NFR-FLX-05 | ![V2](https://img.shields.io/badge/V2-1f6feb?style=flat-square) | The insights component shall be optional: disabling it shall not affect ingestion or history browsing. | Must | T | SN-05 |

### 5.8 Functional suitability and safety

*None identified* beyond the functional requirements in §4. Safety (ISO/IEC 25010:2023) is not applicable: the system cannot cause physical harm.

---

## 6. Constraints

Constraints restrict the solution; they are not requirements of the problem. Each one that is a decision should have an ADR.

| ID | Constraint | Status |
|---|---|---|
| CON-01 | The frontend shall use Next.js, shadcn/ui and AG Grid **Community** edition. *Note: some AG Grid features (e.g. row grouping, used by FR-INS-04's grouped view) are Enterprise-only and require a commercial license.* | Decided |
| CON-02 | Data shall be stored in MongoDB. | Decided, ADR pending (alternative: PostgreSQL + JSONB) |
| CON-03 | The backend language is Go, Python or Next.js Route Handlers. | Open — ADR pending |
| CON-04 | The primary input format is Playwright's JSON reporter output. | Decided |
| CON-05 | Reference deployment for performance requirements: **TBD** (e.g. Docker Compose on a 2 vCPU / 4 GB machine). | Open |
| CON-06 | V1 is developed by one person within the university semester. | Fixed |

---

## 7. Out of scope for V1 (non-goals)

- Running tests or orchestrating CI (Pinwheel only consumes reports).
- Report formats other than Playwright JSON (JUnit XML, etc.).
- User accounts and permissions (V3).
- Storing or displaying attachments such as videos and traces (V3).
- Any ML-based insight (V2).

---

## 8. Traceability matrix (template)

Fill this as user stories and tests are written. A requirement with no test is not verified; a story with no requirement is scope creep.

| Requirement | Stakeholder need | User story | Test case(s) | Status |
|---|---|---|---|---|
| FR-ING-01 | SN-04 | US-?? | TC-?? | Proposed |
| FR-ING-02 | SN-01, SN-02 | US-?? | TC-?? | Proposed |
| FR-RUN-01 | SN-02, SN-03 | US-?? | TC-?? | Proposed |
| … | | | | |

---

## 9. Open questions

1. What exactly counts as a **failed run** in FR-RUN-03: any `unexpected` test, or also `flaky`?
2. Is the stable test ID strategy (FR-PRC-02) enough when a test is renamed or moved? Should history follow a rename?
3. Is FR-ING-06 (duplicate detection) right when a CI run is **re-run** (same run ID, new attempt)? Two runs, or one replaced?
4. Should the V1 web UI be public, or does V1 already need protection beyond the ingestion token (NFR-SEC-01)?
5. Confirm or replace every **TBD** value (sizes, times, coverage, Playwright versions, reference deployment).
6. Does SN-06 (university evaluator) belong in the stakeholder list, or should course requirements be tracked separately?
