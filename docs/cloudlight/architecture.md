# CloudLight — Proposed System Architecture

**Phase 0 · Documentation only · 2026-10-11**

This document describes the proposed architecture, separates what is known from
what is assumed, defines module responsibilities, and compares alternatives. It
does **not** commit to a final architecture — several decisions are blocked on
answers only the school can give (see
[`open_questions_register.md`](open_questions_register.md)).

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in official vendor, standards or regulatory documentation, quoted with a source |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | A vendor documents it; we have not confirmed it on this school's equipment |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No vendor claim; our engineering judgement |
| ❓ **UNKNOWN** | No evidence found. Not assumed either way |
| ⛔ **NOT RECOMMENDED / NOT PERMITTED** | Rejected by law, by vendor terms, or by our own judgement |

---

## 1. Headline: the architecture is an argument about scale

Every major architectural choice in CloudLight follows from one fact:

> **This is one school. A few dozen displays. One host. Very few technical staff.
> No operations team. No on-call rota. No redundancy budget.**

Almost every "best practice" for a large distributed system is a **liability** at
this scale. This section states that plainly, because the default pull in modern
software engineering is toward the opposite.

---

## 2. Microservices vs a modular monolith

The brief asks whether a modular monolith would be simpler and safer than
microservices for a deployment of this size.

### 2.1 The comparison

| Concern | Microservices | Modular monolith | Verdict here |
|---|---|---|---|
| Deployment | Each service independently deployed; N pipelines | One artefact; one pipeline | **Monolith** |
| Network | Every call may cross a process boundary | In-process call | **Monolith** |
| Observability | Distributed tracing required to answer "why is this slow?" | A stack trace | **Monolith** |
| Failure modes | Partial failure, retries, timeouts, circuit breakers everywhere | A crash is a restart | **Monolith** |
| Local dev | Run the whole platform, or mock 6 services | Run one process | **Monolith** |
| Onboarding | Understand N codebases | Understand one | **Monolith** |
| Data consistency | Eventual consistency across services by default | Transactions | **Monolith** |
| Scaling | Scale the hot service | Scale the whole app | Monolith adequate |
| Blast radius of a bug | Can affect several services unpredictably | Bounded by module boundaries | **Monolith** |

### 2.2 Sizing it honestly

🧪 **The scaling argument for microservices does not apply.** The peak load is a
few dozen wall displays polling on a schedule, plus perhaps a few dozen teacher
devices. That is a load a single modest machine handles with room to spare. There
is no hot path to scale independently.

⛔ **The operations argument actively favours the monolith.** Microservices
require someone to run, patch, monitor and debug a distributed system. The school
will have **one** IT generalist with other duties. Every operational concept a
microservice architecture requires — service discovery, distributed tracing,
per-service alerting, coordinated rollouts, inter-service version compatibility —
is a thing that person must also learn and operate, forever, on top of their job.

### 2.3 The correct form of the argument

⚠️ **A modular monolith is not "a monolith".** It is a monolith *with enforced
module boundaries*. The boundaries exist so that a later split is possible and so
that modules cannot quietly reach into each other's data. They are enforced
**in code and by test**, not by process topology.

The honest statement is:

> **Microservices solve an organisational scaling problem. This project does not
> have an organisational scaling problem. Adopting microservices here would be
> importing a large amount of risk and cost to solve a problem that does not
> exist.**

**Reversal condition:** if CloudLight ever serves multiple schools, or if a
genuinely heavy module (for example on-device model inference, or a high-volume
indexing job) needs independent scaling, the modules are already the seams.

Recorded in [`decision_log.md`](decision_log.md).

---

## 3. Modules

The brief lists fourteen conceptual components. These are **modules, not
deployments.** One process, one database, one deployable — with the boundaries
below held in code.

| # | Module | Responsibility | May NOT |
|---|---|---|---|
| A | **Classroom Display** | Render the read-only dashboard; hold an offline snapshot; report its own status | Mutate timetable data |
| B | **Timetable Service** | Own timetable data, period models, academic calendars, versions | Invent a teacher; auto-publish |
| C | **Substitution Service** | Own substitution records and their approval state | Publish an unapproved record |
| D | **Resource Browser** | Present authorised storage as browsable folders; open files at source | Grant access the storage layer denies |
| E | **Search Service** | Index and retrieve authorised resources, permission-filtered | Return a document the user cannot open |
| F | **AI Assistant** | Retrieve, ground and answer; **propose** actions | Publish, approve, grant, or command anything |
| G | **Authentication** | Identity, sessions, MFA, one-action codes | Authenticate a person by a shared code |
| H | **Administration** | Configuration, users, roles, approvals | Bypass its own audit |
| I | **Update Management** | Verify, stage, deploy, roll back signed artefacts | Deploy unsigned artefacts |
| J | **API and Backend** | The single HTTP surface; orchestration; the deterministic validator | Contain business rules of its own |
| K | **Data and Integration** | Database, storage access, index, migrations, backups | Reach outside its data-access boundary |
| L | **SHANA Adapter** | Translate an authorised request into a SHANA request; translate status back | **Send a motor, path or actuator command** |
| M | **Monitoring, Logging, Audit** | Health, structured logs, append-only audit | Hold mutable audit state |
| N | **Deployment and Config** | Host provisioning, service supervision, configuration, secrets | Contain business logic |

### 3.1 The rule that makes modules real

⚠️ **Modules may not import each other's internals.** Each exposes an interface;
cross-module access goes through that interface or through the database with an
explicitly owned schema. A test asserts this.

Without that rule, a "modular monolith" silently becomes a ball of mud in the
first month, and the future split becomes impossible.

---

## 4. Data flow

```
   SOURCES                    PIPELINE                      CONSUMERS
   ────────                   ────────                      ─────────

  timetable file ──┐
  (xlsx/csv)       │
                    ├──► ingest ──► normalise ──► validate ──► ┌──► publish
  substitution      │              (deterministic)   │          │    to specific
  sheet ────────────┤                                              │    displays
                    │                    ┌─────────────┴──────┐   │
  scanned /         │                    │  FAIL              │   │
  photographed ─────┘                    ▼                    ▼   │
  document       AI-assisted       quarantine           human    │
  (conditional)   extraction       (never published)    review   │
                                        │                    │   │
                                        └─── corrections ───┘   │
                                                                    ▼
                                                          ┌───────────────┐
                                                          │ classroom     │
                                                          │ display       │
                                                          │ + last-synced │
                                                          │   timestamp   │
                                                          └───────────────┘

  NAS / shares ──► indexer ──► permission-aware index ──► search ──► authorised
                  (read-only      (query-time filter)              results only

  teacher query ──► assistant ──► retrieval ──► grounded answer / refusal
                       │
                       └──► PROPOSAL ──► deterministic backend validates ──► human confirms
                                                                              │
                                                                              └──► SHANA request
```

⚠️ **Two invariant paths:**

1. **Publication always passes through human approval.** No automated path reaches
   a display, regardless of input quality or model confidence.
2. **Search and the assistant are permission-filtered before retrieval**, and
   re-verified at open time.

---

## 5. Failure behaviour

Design for the failure, not just the happy path. The school's failure modes are
mundane: a power cut, a cable, a power update, someone pressing the wrong button.

| Failure | CloudLight behaviour | Classroom impact |
|---|---|---|
| Smartboard loses network | Show cached snapshot **with visible staleness marker** | Degraded, honest, useful |
| Smartboard loses network AND has no snapshot | Show an explicit "no timetable data" state | **Never a blank or frozen screen** |
| Backend down | Displays continue from cache; actions queue or fail clearly | Timetable still correct-as-of-last-sync |
| NAS down | Timetable unaffected; resource features degrade with a clear message | Core product unaffected |
| Database corrupt | Restore from backup; displays keep the last published snapshot | Known-good data on screen |
| Disk full | Alert before exhaustion; refuse new writes cleanly | No silent corruption |
| Certificate expired | Alert ahead of expiry | Planned, not an outage |
| SHANA offline | **Entirely unaffected.** Robot panel shows unavailable-with-age | **Zero impact on classroom** |
| **SHANA task rejected** | Shown as a normal outcome with a reason | Teacher sees why |
| **Bad data published** | Roll back to the previous version across all displays | Bounded blast radius, rehearsed |
| Clock drift on a display | Detect against server; suppress or mark the countdown | Honest rather than confidently wrong |
| Update fails on one board | Auto re-pin to last known-good | One board recovers itself |
| **Unauthorized person at an unattended board** | No secrets on screen; no persistent privileged session; actions require authentication | Cannot act as a teacher |

⚠️ **The SHANA rows are the most important.** They exist because the brief makes
"CloudLight must operate if SHANA is offline" a hard requirement, and because a
robot problem should never be able to blank a classroom screen.

---

## 6. Why one database and no extra datastores

🧪 **Proposed:** one relational database. No cache tier, no message broker, no
search cluster, no vector database, in the first release.

Reasoning, stated as sizing logic rather than fashion:

| Component | Added when | Not now because |
|---|---|---|
| Cache (Redis) | Measured read latency exceeds target | A few dozen displays against one relational database is not a cache problem |
| Message broker | A workflow genuinely needs decoupling | Publication is a synchronous, human-gated action |
| Search engine (OpenSearch/Elastic) | Corpus size and query load exceed what the database can serve | Corpus size is **unknown**; see below |
| Vector database | Semantic search is approved and in scope | Semantic search is deferred |
| Separate cache/queue/log stack | — | Each adds a process someone must run |

⚠️ **The honest caveat:** the search and AI decisions are genuinely size-dependent
and the size is unknown. `CL-SEARCH-014` (file count and gigabytes) must be
answered before the search engine is chosen. Recommending a specific engine now
would be guessing at a number nobody has measured.

**Reversal condition:** any component in the table above becomes justified by a
*measured* number, recorded with the measurement.

---

## 7. The frontend

🧪 **React is suitable** for the classroom dashboard, resource browsing, search,
and administration, subject to constraints the school has not yet confirmed:

| Use | Assessment | Constraint |
|---|---|---|
| Classroom dashboard | **Suitable** | Large type, no hover, high contrast, glanceable |
| Touchscreen interfaces | **Suitable** | Minimum touch target sizes; confirm panel digitisation |
| Resource browsing | **Suitable** | Rendering must not proxy file content |
| Search | **Suitable** | Latency budget unknown (`CL-SEARCH-013`) |
| Administration | **Suitable** | Not on the classroom display |
| Responsive layouts | **Suitable** | Panel inventory unknown; test the worst case |
| Accessibility | **Achievable** | Must be designed in, not retrofitted |

⚠️ **Browser-first, not a desktop client** — see [`windows_startup.md`](windows_startup.md)
for the comparison and the reversal condition. The dominant reason is that a
browser kiosk gets updates from the server; an installed client needs an installer
and an update mechanism on every classroom machine.

---

## 8. Security architecture

| Boundary | Control |
|---|---|
| Display ↔ server | HTTPS; short-lived session cookies; `httpOnly`, `Secure`, `SameSite=Strict` |
| Server ↔ NAS | SMB 3.x; **dedicated read-only service account**; no teacher credentials |
| Server ↔ search index | Query-time permission filtering; **re-verification at open time** |
| Server ↔ external AI | ⛔ **Prohibited by default.** Requires written school authorisation |
| Server ↔ update source | Signed artefacts; client verifies before accepting |
| Server ↔ SHANA | Mutually authenticated; idempotency keys; **no motor-command path** |
| Admin ↔ server | Separated segment; strongest authentication; append-only audit |
| Secrets | Never in Git, URLs, logs, browser storage or config files |

Full treatment in [`security_and_privacy.md`](security_and_privacy.md).

---

## 9. Update architecture

⚠️ **"OTA" is eight different things**, not one. Frontend bundle, backend, kiosk
configuration, timetable data, search index, AI model, SHANA integration, and — kept
strictly separate — **robot firmware, which CloudLight must never be the delivery
path for.**

Each has a different owner, blast radius and rollback story. Treating them as one
"update button" is how a routine data correction ends up requiring a software
release approval. See [`ota_updates.md`](ota_updates.md).

---

## 10. What is deliberately NOT in this architecture

| Rejected | Why |
|---|---|
| Microservices | Organisational scaling problem this project does not have |
| An access point or router hosting the backend | It is a Layer-2 bridge, not a host |
| A smartboard acting as the server | Re-imaging a classroom board would take down every classroom |
| Redis / broker / search cluster / vector DB | No measured need yet |
| An external AI API by default | Would send school data to a third party without authorisation |
| A packaged desktop client on every board | A browser kiosk updates itself from the server |
| Photograph-of-board as an authoritative source | An image is an input to a proposal, not a record |
| A shared six-digit code as identity | Identifies nobody; guessable; capturable from a shared screen |
| Automatic publication without human approval | The one thing that must never be automated |
| SHANA integration in the first release | Deferred deliberately; the first release must work without it |

---

## 11. Open architectural decisions

| # | Decision | Blocked on | Conservative default if unanswered |
|---|---|---|---|
| 1 | Where the backend physically runs | `CL-SRV-001`, `CL-SRV-002`, `CL-SRV-003` | No pilot is possible |
| 2 | Which identity source | `CL-AUTH-001`, `CL-AUTH-002`, `CL-AUTH-003` | CloudLight owns accounts and offboarding |
| 3 | Search engine | `CL-SEARCH-014`, `CL-SEARCH-001` | Database full-text search only |
| 4 | Whether OCR is ever in scope | `CL-AIML-009`, `CL-SUB-006`, `CL-SEC-005` | No OCR; no board photographs |
| 5 | Whether AI is ever in scope | `CL-AI-001`, `CL-DEC-010`, `CL-AI-009` | No assistant feature |
| 6 | Whether shared storage exists at all | `CL-NAS-001` | No resource features |
| 7 | The SHANA interface | `CL-SHANA-001`, `CL-SHANA-015` | No integration; CloudLight ships without it |
| 8 | Whether automatic updates are permitted | `CL-OTA-012`, `CL-WIN-003` | Manual, logged, human-triggered deploys |

⚠️ **Every one of these has a conservative default, and every default produces a
smaller but still useful system.** That is the correct outcome of a feasibility
phase — a one-room system that works is worth more than a school-wide system that
does not.

---

## 12. Related documents

| Document | Covers |
|---|---|
| [`overview.md`](overview.md) | What CloudLight is; relationship to SHANA |
| [`requirements.md`](requirements.md) | Requirements by class |
| [`feasibility_report.md`](feasibility_report.md) | Conclusions classified by evidence strength |
| [`decision_log.md`](decision_log.md) | Decisions, provisional defaults, reversal conditions |
| [`risk_register.md`](risk_register.md) | Risks by domain |
| [`network_and_hosting.md`](network_and_hosting.md) | Hosting and reachability |
| [`windows_startup.md`](windows_startup.md) | Kiosk, autostart, recovery |
| [`shana_integration.md`](shana_integration.md) | The integration boundary |
| `../../questionnaires/Additional_CloudLight_Questionnaire.md` | 576 questions across 26 categories |