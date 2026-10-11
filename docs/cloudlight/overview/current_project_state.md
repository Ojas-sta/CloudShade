# CloudLight — Current Project State

**Audit date:** 2026-10-11
**Audit method:** direct repository inspection, not recollection.

> ## 🔴 **Read this before any other CloudLight document.**
> This file exists because CloudLight has accumulated a substantial body of
> documentation, and it is easy — for a reader, or an agent — to mistake **volume
> of writing** for **progress**. This document corrects that impression.

---

## 1. The one-paragraph version

CloudLight is a **feasibility study**. It contains 30 markdown documents, a
576-question instrument, and a set of argued design positions. It contains **no
application code, no database, no tests, no deployment, and no contact with any
school infrastructure**. Nothing has been built, nothing has been demonstrated,
and nothing has been approved by anybody at any school. **The project cannot be
run, deployed, demonstrated or judged as a working system.**

⚠️ **The single most important fact about this repository, stated once so it
cannot be mistaken later:** there is **zero CloudLight source code**. No
JavaScript, no TypeScript, no Python, no Go, no Rust. No `package.json`, no
`pyproject.toml`, no `Makefile`, no CI configuration, no test file of any kind.
Everything in `docs/cloudlight/` is research, design and questions.

---

## 2. Verified repository facts

Every row below was established by inspecting the repository, not by recalling
what was intended.

| Fact | Value | How verified |
|---|---|---|
| Implementation source files | **0** | Filesystem scan for `.js`, `.ts`, `.tsx`, `.py`, `.go`, `.rs`, `.java`, `.cs`, `.cpp` |
| Package manifests / build configs | **0** | Scan for `package.json`, `pyproject.toml`, `Makefile`, `CMakeLists.txt`, `Cargo.toml`, `requirements.txt` |
| Test files | **0** | No test directory anywhere in the repository |
| Tests written / executed | **0** / **0** | Nothing runnable exists |
| CI configuration | none | No workflow configuration that builds CloudLight |
| Git commits | **0** | `git log` returns nothing on `main` |
| Git remotes | **0** | `git remote -v` returns nothing |
| CloudLight flat documents | **22** | `docs/cloudlight/*.md` |
| CloudLight subfolder documents | **8** | 4 in `research/`, 4 in `testing/` |
| CloudLight ADRs | **2** | `ADR-013`, `ADR-014` under `docs/architecture/architecture_decisions/` |
| Questionnaire questions | **576** | Unique `CL-*` row keys in the questionnaire |
| Questions answered | **0** | Every `Answer` field empty; every `Status` is `UNANSWERED` |
| Requirements written | **186** | 133 functional, 53 non-functional |
| Requirements agreed by any school | **0** | Stated in [`../requirements.md`](../requirements.md) footer |
| Risks registered | **46** | `R-01`…`R-46` in [`../risk_register.md`](../risk_register.md) |
| Risks retired | **0** | ⚠️ **No risk retires through research** |
| Go/no-go gates | **9** | [`../feasibility/go_no_go_gates.md`](../feasibility/go_no_go_gates.md) |
| Gates passed | **0** | Every gate `PENDING` |
| Permissions requested / granted | **0** / **0** | Nothing has been asked of anybody |
| School hardware touched | **never** | No smartboard, no network, no NAS, no robot |
| AI services contacted | **never** | `C2` prohibits it, and it has not been done |
| School documents read | **none** | No timetable, no substitution sheet, no staff record |

⚠️ **The two directories that are *not* empty of structure** — `data/schemas/`
and `data/templates/` — belong to SHANA, not CloudLight. They are proposed YAML
schemas and markdown templates. ⚠️ **No CloudLight schema exists in either
location**, and no SHANA schema has been parsed by any code.

---

## 3. What exists, honestly assessed

| Artefact | Exists | Real status |
|---|---|---|
| Problem framing | Yes | Coherent, argued, **not validated by any school** |
| Capability list | Yes | 16 capabilities, 9 of them deferred |
| Requirements | Yes | 186 written, **0 agreed**. 45 of them are question-shaped holes, not specifications |
| Architecture proposal | Yes | Design only. **Never implemented, never built, never loaded** |
| Feasibility report | Yes | Conclusions classified by evidence strength |
| Risk register | Yes | 46 risks, **0 retired**, 9 of the top-rated cannot be closed by research |
| Constraints register | Yes | [`../feasibility/constraints.md`](../feasibility/constraints.md) |
| Assumptions register | Yes | [`../feasibility/assumptions.md`](../feasibility/assumptions.md) — 24 assumptions, 23 unverified |
| Go/no-go gates | Yes | 9 gates, **0 passed** |
| Decision log | Yes | 10 decisions, 6 provisional defaults, 6 open, 12 rejected options |
| Open questions register | Yes | 18 architecture-blocking questions of 576 |
| Questionnaire | Yes | **576 questions. 0 answered** |
| First-release scope | Yes | A **scope proposal**, not a plan |
| Next actions | Yes | **Ten questions and approvals. Zero engineering** |
| Testing documentation | Yes | 4 documents. ⚠️ **They describe a strategy that has never been executed** |
| Research documentation | Yes | 4 documents, grounded in vendor documentation |
| Source register | Yes | Real sources, with unsourced judgement labelled separately |
| **Application code** | **No** | — |
| **Database or schema** | **No** | CloudLight's data model is a *proposal in prose* |
| **Tests** | **No** | — |
| **Deployment** | **No** | — |
| **Integration with anything** | **No** | — |

---

## 4. The gap between documentation and reality

⚠️ **This table is the most important thing in this document.** It exists because
the documentation reads as though a system exists, and it does not.

| The documentation reads as… | The reality is… |
|---|---|
| "The dashboard shows the current lesson" | **No dashboard exists.** Not a line of it. The behaviour is *specified* in `FR-DASH-03` |
| "The substitution pipeline quarantines invalid records" | **No pipeline exists.** The tiers and rules are *documented* in `timetable_and_substitutions.md` §4, §5 |
| "Permission is re-verified at open time" | No index, no retrieval, no open |
| "The audit log records actor, timestamp and action" | No audit log exists. Its *requirements* are `FR-ADMIN-03`, `FR-ADMIN-04` |
| "Displays accept only signed releases" | No release mechanism exists. Signing is `FR-OTA-01` |
| "The database is a single relational store" | **No database has been created.** `D-023` and ADR-013 describe an intended one |
| "Acceptance criteria 1–… " | **0 met** |
| "The baseline deployment is one host and N boards" | ❓ **N is unknown.** No host exists; no board has been counted |

⚠️ **Phrasing throughout this documentation set is deliberately conditional and
proposed.** Where a sentence appears to assert that something exists, it should
be **corrected rather than trusted** — including by a later agent working from a
summary rather than from the source.

⚠️ **This is a risk in the documentation, not a failure of it.** A design that is
written in the present tense is more useful to a reviewer than one written in
conditional mood — provided nobody mistakes it for a delivery status. That is
what this file exists to prevent.

---

## 5. The sixteen capabilities and their actual status

The capability list comes from [`../overview.md`](../overview.md) §1. The status
column is this document's own assessment of what exists today.

| # | Capability | Proposed status | **Actual status, 2026-10-11** |
|---|---|---|---|
| 1 | Classroom information displays | In scope | 📄 Specified (`FR-DASH-01`…`15`). **0 lines implemented** |
| 2 | Current and upcoming class information | In scope | 📄 Specified. Blocked on `CL-TIME-005` — the day model is unknown |
| 3 | Class timings and countdowns | In scope | 📄 Specified. Blocked on the same unknown; a countdown needs a period model |
| 4 | Teacher and substitution information | In scope | 📄 Specified. Blocked on `CL-SUB-001` — no approver named |
| 5 | Timetable and substitution-sheet ingestion | In scope | 📄 Designed in detail. **No school timetable has ever been seen** |
| 6 | AI/ML-assisted document conversion | Conditional — assistive only | ⛔ **Deferred.** No model, no permission, no hardware. Out of the first release |
| 7 | Teacher resource browsing | Deferred — gated on storage | ⛔ **Does not exist and may never.** ⛔ `CL-NAS-001` unanswered: a NAS may not exist at all |
| 8 | One-click access to authorised NAS folders | Deferred — same gate | ⛔ Same. Requires a storage system nobody has confirmed exists |
| 9 | Search across authorised resources | Deferred — same gate | ⛔ Same. Also gated on a permission model and an unknown corpus size |
| 10 | AI assistant | Deferred — build last, if ever | ⛔ **Deferred.** The largest unverified dependency in the project |
| 11 | Teacher authentication and authorisation | In scope | 📄 Specified. ⛔ Identity source unknown — `CL-AUTH-001` |
| 12 | Requests to SHANA for approved deliveries | Deferred — build last | ⛔ **Deferred.** No interface exists, no robot contacted, no authentication defined |
| 13 | Over-the-air update mechanism | Deferred — after first pilot | ⛔ **Deferred.** No signing infrastructure, no key custody policy |
| 14 | React-based frontend | Accepted direction | 📄 Accepted direction, constraints unconfirmed. **No frontend exists** |
| 15 | Local server on an authorised school network | In scope | ⛔ **Nothing permitted exists.** `CL-SRV-001` is the project's first gate |
| 16 | Extensible modules for future workflows | Architecture goal | 📄 Fourteen modules *named* in [`../architecture.md`](../architecture.md) §3. **None exists** |

**Summary:** 📄 7 specified but not built · ⛔ 9 deferred or gated on an
unconfirmed precondition · ✅ 0 implemented · 🧪 0 demonstrated.

⚠️ **Nine of sixteen capabilities are gated on a precondition that has not been
confirmed to exist** — an always-on permitted host, a staff identity directory,
shared storage, and AI policy. Four of the other five deferred items are deferred
because they are hard or unproven, not because they depend on something missing.
⚠️ **This is the correct outcome of a feasibility phase**, and it is the reason
the phase was worth doing: the gaps are now named rather than assumed.

---

## 6. What is running

**Nothing.**

| Thing | Status |
|---|---|
| CloudLight host | ❓ Does not exist. `CL-SRV-001` — not even permitted |
| CloudLight service | Does not exist |
| Database | Does not exist. No schema, no migration, no engine chosen |
| Search index | Does not exist |
| Classroom displays configured | **0** |
| NAS accessed | **0** — and whether one exists is unknown |
| Robot operated | **0** |
| AI service contacted | **0** |
| CI pipeline | Does not exist |
| Monitoring or alerting | Does not exist |

---

## 7. What has been verified

⚠️ **Very little, and all of it is about the world rather than about this school.**

| Verified | Class | Against |
|---|---|---|
| `.xlsx` is a ZIP package of XML parts — cell contents addressable without inference | ✅ `VERIFIED` | ISO/IEC 29500; Microsoft documentation |
| A text-layer PDF yields text tokens with coordinates; an image-only PDF does not | ✅ `VERIFIED` | Docling pipeline documentation |
| OCR is the dominant cost stage, by one to two orders of magnitude | ✅ `VERIFIED` | Docling published per-stage figures |
| Edge kiosk mode with Assigned Access or Shell Launcher is a documented Microsoft path | ✅ `VERIFIED` | Microsoft documentation |
| A Windows Service cannot display UI, and so cannot be the kiosk launcher | ✅ `VERIFIED` | Microsoft documentation |
| Assigned Access and Shell Launcher are edition-dependent and cannot be combined | ✅ `VERIFIED` | Microsoft documentation |
| SMS/PSTN is a *restricted* authenticator under NIST SP 800-63B | ✅ `VERIFIED` | NIST SP 800-63B |
| TUF is a published specification, and its threat model names rollback, freeze, fast-forward and mix-and-match | ✅ `VERIFIED` | The Update Framework specification |
| SMBv1 is deprecated by Microsoft and unsupported | ✅ `VERIFIED` | Microsoft documentation |
| An access point is a Layer-2 bridge and cannot host an application | ✅ `VERIFIED` | Architecture of an AP |

⚠️ **Zero school-specific facts have been verified, because none has been
checked.** Every statement about *this* school — its Windows editions, its
network, its storage, its timetable, its staff, its approvals — is a question with
a named owner in
[`../open_questions_register.md`](../open_questions_register.md).

⚠️ **Vendor-documented behaviour is 🟡, not ✅, for anything we would actually
depend on.** A vendor documenting a feature proves the feature exists. It does
not prove it behaves the same way on equipment this school happens to own. The
gap between those two statements is where the pilot lives.

---

## 8. What is blocked, and on whom

| Blocked on | Who | Questions |
|---|---|---|
| ⛔ **Everything** | Principal / Administration | `CL-GEN-001`, `CL-SRV-001`, `CL-OPS-022` |
| ⛔ **Every display** | IT / Network Administrator | `CL-NET-003`, `CL-NET-008` |
| ⛔ **The display design** | IT | `CL-WIN-001`, `CL-WIN-003` |
| ⛔ **Authentication and the whole audit trail** | Administration + IT | `CL-AUTH-001`, `CL-AUTH-002`, `CL-AUTH-003` |
| ⛔ **The core feature's publication path** | Timetable / Administration | `CL-SUB-001`, `CL-TIME-001`, `CL-TIME-005` |
| ⛔ **The entire resource track** | IT + Library | `CL-NAS-001`, `CL-NAS-009`, `CL-SEARCH-014` |
| ⛔ **The assistant** | Principal | `CL-AI-001` |
| ⛔ **Robot integration** | Robot custodian | `CL-SHANA-001`, `CL-SHANA-015` |

⚠️ **None of these is a research task.** Each needs a person, a measurement or a
signature. ⚠️ **Research is finished**: the remaining unknowns are properties of a
specific school, and no further reading will produce them. *"Unknown" is a valid
and valuable answer;* a guess is not.

---

## 9. What would meaningfully change this state

In priority order — and ⚠️ **none of these is a software task**:

| # | Action | Cost | Unblocks |
|---|---|---|---|
| 1 | Ask the Principal whether a persistent local service is permitted at all | ~10 min | ⛔ **Everything.** Gate 1 |
| 2 | Ask IT whether client isolation and firewall rules permit classroom devices to reach a host | ~15 min | ⛔ **Every display.** Gate 2 |
| 3 | Ask administration who approves a substitution and who owns the master timetable | ~15 min | ⛔ **The core feature.** Gate 4 |
| 4 | Ask IT what Windows edition is on the boards and whether kiosk accounts have admin rights | ~10 min | The **display design**. Gate 2 |
| 5 | Ask IT whether a NAS or shared file server exists **at all** | ~10 min | The **resource track**, or closes it permanently |
| 6 | Ask whether a staff identity directory exists and whether SSO against it is permitted | ~10 min | **Authentication.** Gate 3 |
| 7 | Ask the Principal whether any external AI service is permitted, in writing | ~5 min | The **assistant**, and nothing else |
| 8 | Run **one** authorised technical test: `winver` on a single non-critical board | ~15 min on site | Converts claims into measurements |

**Total: roughly 25 minutes of a Principal's time and 40 minutes of IT's time.**
⚠️ **That is the entire remaining cost of learning whether this project is
possible.** Everything after it is only worth spending once these return. See
[`../next_actions.md`](../next_actions.md).

---

## 10. If you are an agent about to act

| Do | Do not |
|---|---|
| Read this file first | ❌ Begin implementation — Gates 1–4 have not returned |
| Treat `UNKNOWN` as a valid answer | ❌ Guess a school fact and build on it |
| Mark a gate `PASSED` only with evidence attached | ❌ Mark a gate `PASSED` because someone replied |
| Cite a `CL-*` ID only after grepping the questionnaire | ❌ **Invent an ID.** Every ID must exist as a row key |
| Correct a sentence that implies something is built | ❌ Trust a document's phrasing as proof of existence |
| Append to [`../decision_log.md`](../decision_log.md) | ❌ Rewrite a decision — history is the value of the log |
| Keep `docs/shana/**` untouched | ❌ Relax a safety boundary CloudLight inherited from SHANA |
| Say "proposed", "designed", "specified" — never "built" | ❌ Describe anything as operational before it is demonstrated |

---

## 11. Status

**Status as at 2026-10-11: Phase 0 — documentation complete, implementation not
started, nothing approved, nothing running, zero code.**

| | |
|---|---|
| Documents | 30 in `docs/cloudlight/` — 22 flat, 8 in subfolders |
| Decisions recorded | 16 (`D-001`–`D-010` decided, `D-021`–`D-026` provisional, `D-101`–`D-106` open) |
| ADRs | 2 — `ADR-013` modular monolith (provisional), `ADR-014` never command the robot (accepted, **not reversible**) |
| Questions asked of the school | **576. 0 answered.** |
| Permissions | **0 requested. 0 granted.** |
| Gates passed | **0 of 9** |
| Assumptions verified | **1 of 24** |
| Risks retired | **0 of 46** |
| Code written | **0 lines** |
| Tests run | **0** |
| Systems deployed | **0** |
| Systems demonstrated | **0** |

⚠️ **Next status change requires a person, not an engineer.** The cheapest thing
that would move this project is a ten-minute conversation with the Principal.

---

## 12. Related documents

| Document | Covers |
|---|---|
| [`../overview.md`](../overview.md) | What CloudLight is; the capability list; the three findings that shape everything |
| [`../feasibility_report.md`](../feasibility_report.md) | Conclusions classified by evidence strength |
| [`../requirements.md`](../requirements.md) | 186 requirements by class; `C1`–`C12`; 16 entities |
| [`../open_questions_register.md`](../open_questions_register.md) | The 18 blocking questions and their conservative defaults |
| [`../risk_register.md`](../risk_register.md) | 46 risks with rating, mitigation and residual risk |
| [`../decision_log.md`](../decision_log.md) | Decisions, provisional defaults, open decisions, rejected options |
| [`../next_actions.md`](../next_actions.md) | The next ten actions — questions and approvals, not engineering |
| [`../feasibility/assumptions.md`](../feasibility/assumptions.md) | 24 load-bearing assumptions, 23 unverified |
| [`../feasibility/constraints.md`](../feasibility/constraints.md) | The binding walls, including the absence of staff |
| [`../feasibility/go_no_go_gates.md`](../feasibility/go_no_go_gates.md) | 9 gates, 0 passed |
| [`../../shana/overview/current_project_state.md`](../../shana/overview/current_project_state.md) | SHANA's equivalent state document |