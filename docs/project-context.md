# Pinwheel — Project Context

> A stable, high-level overview of Pinwheel: what it is, why it exists, the ideas it is built on, and where to find details.
> It deliberately avoids implementation details, versions, numbers and status, which change often and live in the documents listed in [§9](#9-where-to-find-what).

## 1. What Pinwheel is

**Pinwheel** is an open-source web application that **receives Playwright test reports, stores them, and lets users analyze test execution history**.

Playwright's built-in reports describe one execution at a time. Pinwheel connects many executions so people can answer questions that only history can answer: *Is this failure new? Does this test fail randomly? Has the suite been getting slower?*

The name is a tribute to the Pinwheel boss from *Dark Souls*, a nod to Playwright's mask logo.

## 2. The problem

Teams that run Playwright in CI produce a report on every run, but each report is isolated. To understand a failure in context, someone has to open several reports one by one and compare them by hand. As a result:

- recurring and intermittent failures are hard to tell apart from new ones;
- it is slow to find which tests failed on a given branch or commit, and why;
- there is no simple view of suite health over time (pass rate, duration, flakiness).

## 3. Who it is for

| Stakeholder | What they want from Pinwheel |
|---|---|
| QA engineer | Triage failures quickly: new, recurring or intermittent. |
| Developer | See what failed on their branch or commit, and the error. |
| Tech lead | Follow the health of the suite over time. |
| CI pipeline | Send reports automatically, without breaking the pipeline if Pinwheel is unavailable. |
| Forker / maintainer | Deploy, configure and extend Pinwheel without editing core code. |
| University evaluator | Traceable engineering artifacts, from requirements to design to tests. |

## 4. Goals and non-goals

**Goals**

- Make Playwright execution history easy to browse at the **run** level and the **test** level.
- Keep ingestion simple and CI-agnostic: any pipeline that can send an HTTP request can use it.
- Be **easy to customize and fork**. This is a first-class quality goal, not an afterthought.
- Serve as a well-documented software engineering project, where the planning and design artifacts matter as much as the code.

**Non-goals**

- Running tests or orchestrating CI. Pinwheel only consumes reports.
- Replacing Playwright's own HTML report for inspecting a single run in depth.
- Supporting every test framework. Playwright's JSON reporter output is the input format.

The exact scope of each release is defined in `requirements.md`.

## 5. Product direction

Pinwheel grows in three broad stages. Each stage builds on the data collected by the previous one.

| Stage | Theme | In short |
|---|---|---|
| V1 | **History** | Ingest reports (upload or API), store them, browse run and test history in a table. |
| V2 | **Insights** | Charts and trends, cross-run flakiness, failure grouping, duration anomalies, an automatic reporter, multiple projects. |
| V3 | **Integrations and scale** | CI and chat integrations, user accounts, multi-team use. |

Because V2 insights depend on history that cannot be recreated later, V1 already stores the raw data those features will need.

## 6. Core concepts

These terms are used consistently across code, docs and UI.

| Term | Meaning |
|---|---|
| **Report** | The JSON produced by Playwright's `json` reporter for one execution of a suite. |
| **Raw report** | A report stored exactly as received, without transformation. |
| **Run metadata** | Context that Playwright does not include, supplied alongside the report: repository, commit, branch, CI event, CI run ID and attempt, CI run URL. |
| **Run** | One ingested report plus its run metadata. |
| **Test** | One spec executed in one Playwright project (for example, one browser). |
| **Attempt** | One execution of a test within a run. Retries create additional attempts. |
| **Outcome** | Playwright's overall verdict for a test in a run: `expected`, `unexpected`, `flaky` or `skipped`. |
| **Stable test ID** | An identifier that matches the same test across runs, so its history can be followed. |
| **Flaky (within a run)** | Playwright's own label: failed, then passed on retry, in the same run. |
| **Flaky (across runs)** | Pinwheel's concern: a test whose result changes between runs without a code change explaining it. |

The data hierarchy of a report is:

```
run → file suite → (nested describe suites) → spec → test (per project) → attempt
```

## 7. Guiding principles

These principles guide design decisions. A change that goes against one of them should be discussed and recorded in an ADR.

1. **Raw data is the source of truth.** Reports are stored unmodified. Everything else (tables, statistics, insights) is derived from them and can be rebuilt if the data model changes.
2. **Separate ingestion, processing and presentation.** Each has a clear interface, so new components (such as an insights job) can be added, replaced or disabled without touching the others.
3. **Customizability by configuration.** Deployment settings come from environment variables, extension points are documented, and a fork should not need to edit core code for common changes.
4. **CI-friendly and failure-tolerant ingestion.** Sending a report must be simple, and Pinwheel being unavailable must never make a test pipeline fail.
5. **Never lose history.** A failed ingestion leaves no partial data, and a processing error never loses the raw report.
6. **Design before code.** Requirements before features, API contract before endpoints, data model designed around the queries the UI needs.
7. **Decisions are written down.** Every significant architectural decision has an ADR explaining the context, options and consequences.
8. **Dogfooding.** Pinwheel's own test reports are a natural data source for Pinwheel.
9. **Built with AI assistance, owned by people.** AI tools help with the work, but every decision and every line of code must be understood and explainable by the person committing it.

## 8. Development approach

- **Lightweight process:** work is tracked on a task board and delivered in small increments.
- **Incremental delivery:** start from a walking skeleton (ingest → store → list) and grow one user story at a time, using its acceptance criteria as the definition of done.
- **Traceability:** stakeholder needs → requirements → user stories → tests. A requirement without a test is not verified; a story without a requirement is scope creep.
- **Quality gates:** every change to the main branch passes linting, formatting and automated tests in CI.

## 9. Where to find what

This document is intentionally general. Details live in the documents below, and those are the ones to update as the project evolves.

| Document | Answers |
|---|---|
| `README.md` | How to run, configure and use Pinwheel. |
| `docs/vision.md` | The product vision in one page. |
| `docs/requirements.md` | Functional and non-functional requirements, priorities and release scope. |
| `docs/adr/` | Architecture decisions: technology choices, data storage, ingestion, and why. |
| `docs/v2-possible-features.md` | Candidate insight features and the data they need. |
| API contract (OpenAPI) | The HTTP API, kept consistent with the implementation. |
| Architecture and data model docs | Diagrams and how data is structured and queried. |
| `CLAUDE.md` | Conventions for AI-assisted development in this repo. |
| [the-internet-playwright](https://github.com/b1zz1/the-internet-playwright) | Pinwheel's first real data source: a sample Playwright suite whose CI stores its reports. |

## 10. Keeping this document current

Update this file only when something **fundamental** changes: the problem, the audience, the goals, a core concept or a guiding principle. If a change is about *how* something is built (a library, an endpoint, a field, a number), it belongs in an ADR, the requirements or the technical docs instead.
