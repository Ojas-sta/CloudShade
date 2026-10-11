# CloudLight — Implementation Inventory

**Date:** 2026-10-11 · **Phase 0 · Research and design only**

> **🔴 THE HEADLINE FINDING, STATED ONCE:**
> **There is no CloudLight source code in this repository. Not one line.**
> Every CloudLight component below is `DESIGN_ONLY`, `RESEARCH_ONLY`, or `BLOCKED`.
> **Nothing is built, nothing runs, nothing is deployed, and no test has ever been executed.**

This document exists so that a future agent cannot mistake a well-written design
for a working product.

---

## 1. Verification method

Every claim in this inventory was established by inspecting the repository, not
by assumption. The commands used and their results:

| Check | Method | Result |
|---|---|---|
| Application source files | `find` for `*.js *.jsx *.ts *.tsx *.py *.go *.rs *.java *.cs *.rb *.php *.c *.cpp *.html *.css` outside `.git` | **0 files** |
| Package manifests | `find` for `package.json`, `package-lock.json`, `pyproject.toml`, `requirements*.txt`, `Cargo.toml`, `go.mod` | **0 files** |
| Build configuration | `find` for `Makefile`, `Dockerfile`, `docker-compose*.yml`, `tsconfig*.json` | **0 files** |
| Test files | `find` for `*.test.*`, `*.spec.*`, `test_*.py`, `*_test.go` | **0 files** |
| CI configuration | inspection of `.github/` | Issue and PR templates only. **No workflows, no CI** |
| Version control | `git log`, `git remote`, `git status` | **0 commits, 0 remotes.** Everything is untracked |
| Runtime artefacts | inspection for `node_modules/`, `venv/`, `build/`, `dist/` | **None present** |

### Complete list of every non-markdown file in the repository

| Path | Owner | What it is |
|---|---|---|
| `.editorconfig` | shared | Formatting rules |
| `.gitignore` | shared | Ignore rules |
| `.github/ISSUE_TEMPLATE/config.yml` | shared | Issue template config |
| `.obsidian/*.json` | SHANA | Obsidian vault configuration |
| `data/schemas/corridor_state.schema.yaml` | SHANA | **Schema, documentation only** — not implemented |
| `data/schemas/delivery.schema.yaml` | SHANA | **Schema, documentation only** — not implemented |
| `data/schemas/destination.schema.yaml` | SHANA | **Schema, documentation only** — not implemented |
| `data/schemas/route_graph.schema.yaml` | SHANA | **Schema, documentation only** — not implemented |
| `start_opencode_server.sh` | SHANA | Local tooling script, **gitignored**, contains a hardcoded plaintext password that its owner should rotate |

Everything else in the repository is Markdown.

⚠️ **The four YAML schemas are the closest thing to an interface contract that
exists anywhere in this project — and SHANA's own header comment on each reads
`# PROPOSED — not implemented. Documentation only.`** They are design artefacts,
not code.

---

## 2. Status labels used

| Label | Meaning |
|---|---|
| `IMPLEMENTED_AND_TESTED` | Built **and** a test was actually executed and passed |
| `IMPLEMENTED_NOT_TESTED` | Built, but no test was executed |
| `PARTIALLY_IMPLEMENTED` | Some of it built |
| `DESIGN_ONLY` | Fully specified on paper; nothing built |
| `RESEARCH_ONLY` | Investigated; no design committed |
| `BLOCKED` | Cannot proceed until a named party answers |
| `UNKNOWN` | Nobody knows; including us |
| `DEPRECATED` | Superseded |

⚠️ **The brief's instruction is explicit and correct: do not claim a feature works
because a placeholder exists.** There are no placeholders. There are no
components in any implemented state.

---

## 3. Component inventory

### 3.1 Frontend

| Component | Purpose | Status | Files | Entrypoint | Runtime reqs | Tests | Result |
|---|---|---|---|---|---|---|---|
| React application | Classroom display, dashboard, admin | **`DESIGN_ONLY`** | Design in `../architecture.md`, `../requirements.md` | **none** | **none defined** | **none** | **NOT RUN** |
| Display rendering / layouts | Show timetable on a panel | **`DESIGN_ONLY`** | `../classroom_dashboard.md` | **none** | — | — | **NOT RUN** |
| Teacher-facing UI | Browse, search, request | **`DESIGN_ONLY`** | `../requirements.md` | **none** | — | — | **NOT RUN** |
| Admin UI | Configure, approve, publish | **`DESIGN_ONLY`** | `../requirements.md` | **none** | — | — | **NOT RUN** |
| Any frontend framework choice | React was *proposed*, never selected as a build target | **`RESEARCH_ONLY`** | `../architecture.md` §7 | — | — | — | — |

⚠️ **There is no `package.json`.** No dependency has been installed. No component
library, router, state manager or build tool has been chosen. React is a
*proposition under evaluation*, documented in `../../questionnaires/Additional_CloudLight_Questionnaire.md`
category `CL-REACT`, not a decision recorded in the ADR log.

### 3.2 Backend and services

| Component | Purpose | Status | Files | Tests | Result |
|---|---|---|---|---|---|
| Backend application | Serve the API and the frontend | **`DESIGN_ONLY`** | `../architecture.md` | **none** | **NOT RUN** |
| Timetable service | Own timetable data and versions | **`DESIGN_ONLY`** | `../timetable_and_substitutions.md` | — | — |
| Substitution service | Own substitution records and approval state | **`DESIGN_ONLY`** | `../timetable_and_substitutions.md` | — | — |
| Resource browser | Present authorised storage | **`DESIGN_ONLY`** | `../resource_browser.md` | — | — |
| Search service | Permission-filtered retrieval | **`DESIGN_ONLY`** | `../resource_search.md` | — | — |
| AI assistant | Grounded answers and proposals | **`DESIGN_ONLY`** | `../ai_assistant.md` | — | — |
| Document ingestion | Parse timetables and sheets | **`DESIGN_ONLY`** | `../timetable_and_substitutions.md` | — | — |
| Authentication service | Identity, sessions, one-action codes | **`DESIGN_ONLY`** | `../authentication_and_authorisation.md` | — | — |
| Administration service | Config, users, roles | **`DESIGN_ONLY`** | `../requirements.md` | — | — |
| Update management | Verify, stage, deploy, roll back | **`DESIGN_ONLY`** | `../ota_updates.md` | — | — |
| Logging / monitoring / audit | Observability and tamper-evident audit | **`DESIGN_ONLY`** | `../security_and_privacy.md` | — | — |

**There is no API. No endpoint. No route. No schema migration. No background job.**

### 3.3 Data and storage

| Component | Purpose | Status | Detail |
|---|---|---|---|
| Database | Timetable, substitutions, users, audit | **`DESIGN_ONLY`** | Engine not chosen. `CL-DATA-002` **UNANSWERED** |
| Data model | Entities and relationships | **`DESIGN_ONLY`** | `architecture/data_model.md`. Provisional — the real period/day structure is **UNKNOWN** |
| Migrations | Schema evolution | **`DESIGN_ONLY`** | None written. None needed yet |
| Search index | Metadata and text index | **`DESIGN_ONLY` / `BLOCKED`** | Blocked on whether shared storage exists (`CL-NAS-001`) |
| Backup / restore | Recoverability | **`DESIGN_ONLY`** | No backup has been configured because no system exists |
| Live timetable data | Real school timetable | **`UNKNOWN`** | **No school timetable has ever been seen by this project** |
| Live resource data | Real teaching resources | **`UNKNOWN`** | **No NAS has ever been accessed by this project** |

⚠️ **No database contains any school data, because no database exists.** Any
timetable or resource content appearing in a future demonstration would be
synthetic.

### 3.4 Integrations

| Integration | Purpose | Status | Evidence of connection |
|---|---|---|---|
| School LAN | Reach displays from the server | **`UNKNOWN` / `BLOCKED`** | **None. No network was probed.** `CL-NET-003` |
| NAS / shared storage | Resource browsing and search | **`UNKNOWN` / `BLOCKED`** | **None. Never accessed.** `CL-NAS-001` |
| Identity provider | Teacher authentication | **`UNKNOWN` / `BLOCKED`** | **None. None chosen, none contacted.** `CL-AUTH-002` |
| External AI service | — | ⛔ **NOT_RECOMMENDED_WITHOUT_APPROVAL** | **None. Prohibited by default.** `CL-AI-001` |
| Local AI inference | Assistant, if ever permitted | **`RESEARCH_ONLY`** | **None. No model downloaded or run.** |
| Update source | Signed releases | **`DESIGN_ONLY`** | None. No key, no repository |
| **SHANA interface** | Delivery requests and status | **`DESIGN_ONLY`** | **None. See §4.** |
| Smartboards | Display clients | **`UNKNOWN`** | **None. No smartboard has been touched.** |

### 3.5 Client / device side

| Component | Purpose | Status | Detail |
|---|---|---|---|
| Kiosk configuration | Auto-start the display fullscreen | **`DESIGN_ONLY`** | `../windows_startup.md` |
| Offline snapshot cache | Keep the display useful without network | **`DESIGN_ONLY`** | ⚠️ **Highest-risk unknown**: whether a durable cache installs inside a browser kiosk InPrivate session is unverified |
| Client update mechanism | Deliver frontend changes | **`DESIGN_ONLY`** | `../ota_updates.md` |
| Any installed client on a smartboard | — | **Does not exist** | No software has been installed on any school device |

---

## 4. SHANA integration status

**Status: `DESIGN_ONLY` · not implemented · not tested · no agreement reached.**

### 4.1 What exists

| Artefact | Owner | State |
|---|---|---|
| CloudLight's proposal | CloudLight | `../shana_integration.md` |
| SHANA's proposal | SHANA | `docs/shana/architecture/cloudlight_integration.md` (read-only) |
| Four YAML schemas | SHANA | `data/schemas/*.yaml` — headers read `PROPOSED — not implemented` |

### 4.2 🔴 The two interfaces were drafted in isolation and they differ

This is a **material finding**. Neither side has seen the other's contract, and
they are not equivalent.

| Aspect | CloudLight's proposal | SHANA's proposal | Divergence |
|---|---|---|---|
| Identifier | `request_id` + explicit `idempotency_key` | `delivery_id` only | 🔴 **SHANA's schema has no idempotency field.** A retried request cannot be deduplicated as designed |
| Negative outcome | Distinct `REJECTED` (policy decline) vs `FAILED` (operational) | Only `FAILED` and `CANCELLED`; `status.error` | 🟠 No way to distinguish "SHANA declined on policy" from "something broke" |
| Inbound commands | Request only, plus a cancellation *request* | `task.assign`, `task.cancel`, `task.pause`, `task.resume`, `system.stop`, `system.pause` | 🟠 SHANA anticipates pause/resume/stop messages CloudLight's design says it must never send |
| Authentication | 🔴 Mutual authentication required (P0) | **Not mentioned anywhere** | 🔴 **No authentication requirement is stated on SHANA's side** |
| Waiting for recipient | Explicit `AWAITING_CONFIRMATION` state | `AWAITING_RECIPIENT` in the schema | ✅ Consistent |
| Completion | Requires recipient confirmation | `COMPLETED requires recipient_confirmed == true` | ✅ Consistent — both honour the invariant |
| Destination | `destination_id`, no free text | `no free-text destination field exists` by design | ✅ Consistent |
| Requester identity | Reference, not credentials | `requested_by` = "teacher role id" | ✅ Consistent |

⚠️ **The two gaps that matter most are the missing idempotency field and the
missing authentication requirement.** Both are safety-relevant: the first permits
a duplicate physical task, the second permits any host on the network to request
one. **Neither can be closed by CloudLight alone.** They require a joint session
between both owners — see `../handoff/unresolved_dependencies.md`.

### 4.3 A hard blocker on SHANA's side

🔴 **`destination.schema.yaml` sets `status: NEEDS_SURVEY` as the default**, and
validation requires `verified_by` and `verified_pose_date` before any
destination may become `ACTIVE`.

> **This means SHANA currently has, by construction, zero usable delivery
> destinations.** No destination can be `ACTIVE` until it has been physically
> surveyed and its approach pose verified.

Consequence: **no live robot delivery can be demonstrated at all** until a site
survey produces at least one verified destination. This is SHANA-owned work and
is entirely outside CloudLight's control.

### 4.4 Dependencies CloudLight has on SHANA

| # | Dependency | Currently | If never satisfied |
|---|---|---|---|
| 1 | A delivery-request interface exists | `UNKNOWN` | CloudLight ships without the feature |
| 2 | Schemas are agreed between both owners | **Not agreed** | No integration |
| 3 | Mutual authentication is defined | **Not defined** | 🔴 Unsafe if built |
| 4 | Idempotency is expressible | **Not in SHANA's schema** | 🔴 Duplicate dispatch possible |
| 5 | At least one verified destination exists | **Zero exist** | No delivery can occur |
| 6 | A documented policy for behaviour when CloudLight is down | Referenced by SHANA, not agreed with us | SHANA decides; CloudLight adapts |
| 7 | Robot network reachable from CloudLight's segment | `UNKNOWN` | Integration deferred |

⚠️ **Every one of these is BLOCKED or UNKNOWN. CloudLight's own guidance, recorded
in `../shana_integration.md` §12, is to build CloudLight without the SHANA
integration and add it last.** That guidance is unchanged.

---

## 5. Data provenance

| Data type | Provenance | Real or synthetic |
|---|---|---|
| Timetable content | **None has ever been received** | N/A — no data exists |
| Substitution records | **None has ever been received** | N/A |
| Teaching resources | **No NAS has been accessed** | N/A |
| Example queries in documentation | Written by the project team for illustration | **Synthetic** |
| Example schemas in documentation | Illustrative field sketches | **Synthetic** |
| SHANA's four YAML schemas | Design artefacts authored by SHANA | **Synthetic / illustrative** |

⚠️ **There is no real school data anywhere in this repository.** Not one field.
This is intentional and is the strongest privacy position available: nothing
about students or staff has been collected, because nothing has been built.

---

## 6. Tests actually executed

| Metric | Value |
|---|---|
| Test files in the repository | **0** |
| Test framework configured | **None** |
| Test commands run | **None** |
| Tests passed | **0** |
| Tests failed | **0** |
| Coverage measured | **Not measurable — nothing to cover** |
| CI runs | **0 — no CI configuration exists** |

⚠️ **No test result may be reported anywhere.** `../testing/test_inventory.md`
lists **planned** tests, every one marked `NOT WRITTEN` / `NOT RUN`. That
inventory is a plan, not a record.

---

## 7. Honest assessment

**What is genuinely strong:**

- The **question set** — 576 questions across 26 categories, every response
  field blank, no ID duplicated, no collision with the SHANA namespace.
- The **safety and permission reasoning** — the six-digit-code finding, the
  scheduled-vs-confirmed rendering contract, the fail-closed design, and the
  absent-motor-command boundary are all argued carefully and consistently.
- The **decision record** — decisions carry reversal conditions; rejected options
  carry reasons.
- The **honesty discipline** — unknowns are labelled as unknowns rather than
  smoothed over.

**What does not exist at all:**

- **Any code.**
- **Any running system.**
- **Any tested claim about school infrastructure.**
- **Any agreement with SHANA.**
- **Any contact with the school.**
- **Any approved deployment.**

⚠️ **A future agent must not let the volume and quality of the documentation imply
that the system is further along than it is.** Twenty-two substantial documents
and a well-structured questionnaire are a *research artefact*, not a product.
The distance between this repository and a working classroom dashboard is
measured in permissions, not in prose.

---

## 8. What would change this inventory

| To become | Requires |
|---|---|
| `IMPLEMENTED_NOT_TESTED` | Passing the Gate 1–3 go/no-go gates, then writing code |
| `IMPLEMENTED_AND_TESTED` | The above, plus an executed test suite with recorded results |
| `BLOCKED → DESIGN_ONLY` | Answering `CL-NET-003`, `CL-SRV-001`, `CL-NET-008` |

Full gates: `../feasibility/go_no_go_gates.md`.
First actions: `../next_actions.md`.

---

## Related

- [`../handoff/agent_handoff.md`](agent_handoff.md) — where to start
- [`../handoff/exhibition_evidence_inventory.md`](exhibition_evidence_inventory.md) — what may be shown
- [`../handoff/unresolved_dependencies.md`](unresolved_dependencies.md) — what is blocked and on whom
- [`../overview/current_project_state.md`](../overview/current_project_state.md) — factual state
- [`../testing/known_failures.md`](../testing/known_failures.md) — failures of this process itself