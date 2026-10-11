# CloudLight — Go/No-Go Gates

**Phase 0 · Documentation only · 2026-10-11**

> ## ⛔ **NO GATE HAS BEEN PASSED.**
> **0 of 9 gates passed. 0 permissions requested. 0 permissions granted.**
> **A gate is never marked passed without evidence attached to it.**

> ## ⚠️ **There is no CloudLight software.** No host exists, no smartboard has been
> configured, no network has been probed, no storage has been reached, no
> timetable has been read and no robot has been operated. Gates 1–8 describe work
> that **could** be done once permission exists. None of it has begun.

These gates prevent implementation from proceeding prematurely. A gate may be
marked **`PASSED`**, **`BLOCKED`**, **`NOT APPLICABLE`** or **`PENDING`** — and
nothing else.

⚠️ **A gate is never marked `PASSED` on the strength of a conversation.**
`ANSWERED` means a person replied. `VERIFIED` means evidence exists and is
attached. `PASSED` means a named authority accepted that evidence. The three are
different, and conflating them is how a project discovers, in front of a class,
that it was built on an assumption.

Style follows the SHANA gate document at
[`../../shana/feasibility/go_no_go_gates.md`](../../shana/feasibility/go_no_go_gates.md).
⚠️ **No criterion below has been assessed by anyone.** Every gate is in its
initial state.

---

## Status values

| Status | Meaning |
|---|---|
| `PASSED` | All criteria met, evidence attached, declared by a named authority |
| `BLOCKED` | Cannot proceed; the stated blocker must be resolved first |
| `NOT APPLICABLE` | Deliberately excluded from scope, with a recorded reason |
| `PENDING` | Not yet assessed — ⚠️ **the current state of every gate in this document** |

---

## Gate authority

⚠️ **A gate may only be declared by a named school authority, not by the project
team.** The project team may assemble the evidence. It may not accept it.

| Role | Authority over |
|---|---|
| Principal or Administration | Gates 0, 1, 5, 6, 7, 8 |
| IT / Network Administrator | Gates 1 (hosting), 2 (network), 3 (identity, technical half) |
| Timetable / Administration department | Gates 4, 5 (approval path) |
| Facilities | Gate 1 (physical location, power) |
| Robot custodian (SHANA) | Gate 8 |
| Library / Resources | Gate 6 |

⚠️ **Authority to declare `NOT APPLICABLE` is equivalent to authority to declare
`PASSED`.** A gate cannot be made to disappear to avoid the work it represents.
Gate 6 and Gate 7 in particular are expected to resolve as `NOT APPLICABLE`, and
that is a legitimate, complete way to pass them.

---

## GATE 0 — Research complete

**Blocks everything except itself.**

The purpose of this gate is to establish that further reading is pointless, so
that the next conversation is with a person rather than with a document.

| # | Criterion | Evidence required |
|---|---|---|
| 0.1 | Every capability in the brief is documented, argued for and classified by evidence strength | [`../feasibility_report.md`](../feasibility_report.md) complete |
| 0.2 | Every capability is traced to requirements and to gating questionnaire categories | [`../requirements.md`](../requirements.md) §7 — 16 of 16 |
| 0.3 | Every blocking question has a named respondent and a conservative default | [`../open_questions_register.md`](../open_questions_register.md) §2 |
| 0.4 | Every constraint is recorded with its source and its non-negotiability | [`constraints.md`](constraints.md) |
| 0.5 | Every load-bearing assumption is recorded with a cheap test | [`assumptions.md`](assumptions.md) |
| 0.6 | Every source is classified; unsourced judgement is labelled as such | [`../sources.md`](../sources.md) §4 |
| 0.7 | Every rejected option is recorded with its reason | [`../decision_log.md`](../decision_log.md) §4 — `X-01`…`X-12` |
| 0.8 | Confirmation that **no research question remains that research can answer** | [`../open_questions_register.md`](../open_questions_register.md) §5 |

⚠️ **Criterion 0.8 is the substantive one.** Everything remaining requires a
person, a measurement or a signature. Six of the open items are not technical
questions at all — they are permission and governance questions in a technical
costume, and no amount of reading will answer them.

**Status: `PENDING`**
**Blocker:** internal review of this documentation set by anyone other than its
authors has not taken place.

⚠️ **Passing Gate 0 does not mean the project is possible.** It means the project
knows precisely what it does not know.

---

## GATE 1 — Permission to host

**The gate that ends the project rather than changing its shape.**

| # | Criterion | Evidence required |
|---|---|---|
| 1.1 | A **named authority** is identified who can approve or refuse a system storing school data on school infrastructure | Written record — `CL-GEN-001` |
| 1.2 | **Written authorisation** for the project in principle | Signed authorisation |
| 1.3 | A persistent **always-on service is permitted** on school premises, during lessons | Written approval — `CL-SRV-001` |
| 1.4 | A specific **physical machine** is approved, with a location | Written record — `CL-SRV-002` |
| 1.5 | **Power resilience** at that location: a UPS, or an accepted risk | Written confirmation — `CL-SRV-003` |
| 1.6 | A **named person will physically restart** the host after an unplanned shutdown | Written record — `CL-SRV-011` |
| 1.7 | A **year-two owner** is named — a person or a department | Written decision — `CL-OPS-022` |
| 1.8 | Confirmation that **school data will not leave school infrastructure** by default | Written confirmation — `CL-NET-015` |
| 1.9 | Any **insurance or policy** covering a long-lived local service is identified | Written confirmation |

⚠️ **Criterion 1.7 is the one most often skipped, and it is the only criterion
whose absence should stop the project rather than shrink it.** Everything else in
this document produces a smaller product when unanswered. *"Nobody will own this"*
produces a screen that quietly stops updating in year two — worse than not
building it, because it destroys trust in the one thing the school relies on.

⚠️ **Criterion 1.2 is not satisfied by the questionnaire being filled in.** The
questionnaire records *answers*. This criterion records a *decision*.

**Status: `PENDING`**
**Blocker:** nothing has been requested from the Principal or from IT.

---

## GATE 2 — Network reachability proven

**The gate with the most silent failure modes.**

| # | Criterion | Evidence required |
|---|---|---|
| 2.1 | **Client / AP isolation is confirmed OFF** on the wireless segment boards use | Written confirmation from whoever administers the APs — `CL-NET-008` |
| 2.2 | The **firewall permits** the display segment to reach the host, on the application port only | Firewall rule review — `CL-NET-003`, `CL-NET-014` |
| 2.3 | A **local DNS name** and a **stable address** exist for the host | Record — `CL-NET-006`, `CL-NET-012` |
| 2.4 | A **TLS certificate the displays trust** can exist without internet access | Certificate or written confirmation of an internal CA — `CL-SEC-024` |
| 2.5 | **One display is proven to reach the host end to end** — DNS, address, firewall, TLS, application response | A recorded test on a machine IT has named as non-critical — `CL-WIN-007` |
| 2.6 | Displays are identified as **wired or wireless**, per room | Written inventory — `CL-NET-005` |
| 2.7 | The **administration surface is unreachable** from the display segment | Configuration confirmation — `CL-SEC-023` |

⚠️ **Criterion 2.1 fails silently and invisibly.** Boards associate, show full
signal strength, and never load a page. There is no error to read. This is the
single most likely quiet failure in the entire project, and it is why criterion
2.1 exists as a written confirmation rather than a test we run ourselves.

⚠️ **Criterion 2.5 is the criterion that proves the chain, and criterion 2.5 is
the only one that can be satisfied by measurement rather than by reading somebody
else's configuration.** A working ping proves none of the rest — name resolution,
firewall, TLS and application response each fail independently.

⚠️ **We may not probe the network ourselves** (`C13`, `C24`). Every criterion here
is answered by a person, or by a test run *with* a person, on a machine they have
named as non-critical.

**Status: `PENDING`**
**Blocker:** nothing has been requested from IT.

---

## GATE 3 — Identity decided

| # | Criterion | Evidence required |
|---|---|---|
| 3.1 | Determined whether a **staff identity directory exists** — one list of teachers with stable identifiers | Written answer — `CL-AUTH-001` |
| 3.2 | Determined **what that directory is** — AD, LDAP, Google Workspace, M365, or none | Written answer — `CL-AUTH-002` |
| 3.3 | **Written permission** to authenticate against it, if one exists | Written approval — `CL-AUTH-003` |
| 3.4 | Determined whether **every action must be attributable to one named teacher**, or whether a shared classroom account is acceptable | Written decision — `CL-AUTH-004` |
| 3.5 | If no directory exists: an **authoritative staff list is named as a signed artefact** — produced, dated and accepted by a person | Written record — `CL-ADMIN-005` |
| 3.6 | **Offboarding** is assigned to a named person, with a stated process and window | Written process — `CL-AUTH-007` |
| 3.7 | A **recovery path** exists that does not require the Principal for a routine reset | Written procedure — `CL-AUTH-014` |
| 3.8 | Written acknowledgement that a **six-digit code may not authenticate a person** | Written acknowledgement — `CL-CODE-002`, `CL-CODE-015` |
| 3.9 | **Display sessions** — whether a board may hold a signed-in teacher session while unattended — decided | Written decision — `CL-AUTH-020`, `CL-SEC-001` |

⚠️ **This gate may legitimately close as `NOT APPLICABLE` for its SSO half.** A
school with no directory is a normal outcome, not a failure. What may not happen is
the *other* half: if there is no directory, criterion 3.5–3.6 become **more**
important, because CloudLight has become the system of record for teacher identity
and therefore owns both provisioning and offboarding.

⚠️ **Criterion 3.4 is where the brief's proposal is most likely to be re-litigated.**
A shared classroom PIN is ⛔ rejected as identity (see
[`identity_provider.md`](../integrations/identity_provider.md)). It may authorise
one action or act as a second factor. It may never stand in for a named teacher.

**Status: `PENDING`**
**Blocker:** identity source unknown.

---

## GATE 4 — First release built

⚠️ **The first gate that describes engineering work, and the first that cannot
begin until Gates 1–3 return.**

| # | Criterion | Evidence required |
|---|---|---|
| 4.1 | A **named approver** exists for a substitution | Written record — `CL-SUB-001` |
| 4.2 | The **master timetable source** is identified, and the person who last edited it is known | Written record — `CL-TIME-001` |
| 4.3 | The **teaching day structure** is documented — periods, durations, blocks, breaks | Written answer — `CL-TIME-005`, `CL-TIME-006`, `CL-TIME-007` |
| 4.4 | The **academic calendar** is available — holidays, term breaks, special days | Written answer — `CL-TIME-013`, `CL-TIME-015` |
| 4.5 | **Display deployment** is permitted on classroom machines | Written permission — `CL-WIN-003` |
| 4.6 | A correct-data path is demonstrated **end to end**: ingest → deterministic validation → named human approval → versioned publication → read-only display → rollback | Test record |
| 4.7 | A **wrong publication can be withdrawn** across all displays, and the withdrawal is rehearsed | Test record |
| 4.8 | **Fail-closed offline behaviour** is demonstrated: host unreachable, display shows cached data with a visible staleness marker, never blank | Test record |
| 4.9 | **A backup has been restored** onto a clean machine, successfully | Restore record — `NFR-REC-03` |
| 4.10 | **Health checks** exist and reach a named person | Configuration + named recipient — `NFR-AVL-05` |
| 4.11 | An **audit log** records actor, timestamp, action and object | Configuration + sample record |
| 4.12 | No **unapproved substitution** has ever been displayed as a confirmed teacher | Test record |

⚠️ **Criterion 4.6 is the whole product.** The screen is not the deliverable; the
correct-data path is. Every later capability attaches to that path, and building it
first is the only ordering in which later work is cheap.

⚠️ **Criterion 4.9 is the criterion that is always skipped and never forgiven.** A
backup that has never been restored is a file, not a plan.

⚠️ **Criterion 4.3 cannot be engineered around.** A wrong day model does not
degrade — it *rejects correct data*, which is indistinguishable from being wrong.

**Status: `PENDING`**
**Blocker:** no implementation exists. Gates 1–3 have not returned.

---

## GATE 5 — Pilot approved

| # | Criterion | Evidence required |
|---|---|---|
| 5.1 | Gates 0–4 all passed | Gate records |
| 5.2 | **Separate written authorisation** for a pilot in this school | Signed authorisation — `CL-EXPO-013` |
| 5.3 | **Pilot scope** written and approved — rooms, dates, hours, who may see it | Written scope |
| 5.4 | **Success criteria** defined *before* data collection, by the school | Written definitions |
| 5.5 | A **manual fallback** is agreed — what happens when the backend is down | Written procedure — `CL-SEC-022` |
| 5.6 | **Stop authority** at every stage is unambiguous, and the stop authority is told they hold it | Written confirmation |
| 5.7 | **Stop criteria** agreed — the conditions under which the pilot is halted | Written criteria |
| 5.8 | Teachers affected have been **informed, and consent or objection recorded** | Record |
| 5.9 | **Acceptance criteria confirmed not to be waivable** under time pressure | Written confirmation |

⚠️ **Criterion 5.9 matters more than it appears.** Every deadline becomes a reason
to lower a standard, and the standard that gets lowered is always the one that
protects the school.

⚠️ **A demonstration with mock data is not a pilot** and cannot satisfy this gate.
It is a separate, labelled activity (`CL-EXPO-002`).

**Status: `PENDING`**
**Blocker:** Gate 4 has not passed. Nothing exists to pilot.

---

## GATE 6 — Resource features

⚠️ **This gate is expected to resolve as `NOT APPLICABLE`, and that is a valid,
complete outcome.** A school with no shared storage is common. The timetable and
substitution product stands alone without it.

| # | Criterion | Evidence required |
|---|---|---|
| 6.1 | A **NAS or shared file server exists at all** | Written confirmation — `CL-NAS-001` |
| 6.2 | **Confidential staff material is not co-located** with teaching resources, or an explicit approved folder list is given | Written confirmation — `CL-NAS-011`, `CL-NAS-008` |
| 6.3 | The **permission model** is documented — per-user, per-group, or folder-inherited | ACL export — `CL-NAS-009` |
| 6.4 | A **dedicated read-only service account** is created for CloudLight | Account record — `CL-NAS-007` |
| 6.5 | The **corpus size is measured** — file count and gigabytes | A directory listing and a total — `CL-SEARCH-014` |
| 6.6 | **Permission filtering is tested in both directions** — an over-broad principal and a missing principal | Test record |
| 6.7 | **Permission is re-verified at open time** | Test record — `D-026` |
| 6.8 | Index entries are **deleted when the source is deleted** | Test record — `CL-DATA-020` |

⚠️ **Criterion 6.1 is a gate within a gate.** If it returns *"no"*, criteria
6.2–6.8 are **moot** and the resource track closes permanently. Do not ask about
indexing permission unless storage has been confirmed to exist.

⚠️ **Criterion 6.3 can close the feature by itself.** If permissions are
folder-inherited only, per-teacher filtering is not achievable safely, and search
over material a user cannot selectively access is not shippable.

⚠️ **Criterion 6.5 is the number that decides the search architecture**, and it is
the cheapest measurement in the entire project.

**Status: `PENDING`**
**Blocker:** unknown whether shared storage exists at all.

---

## GATE 7 — AI assistant

⚠️ **This gate is expected to resolve as `NOT APPLICABLE`, and the core product is
completely unaffected.** That is precisely why the assistant is built last, if ever.

| # | Criterion | Evidence required |
|---|---|---|
| 7.1 | **Gate 6 passed** — there is a corpus to retrieve from | Gate record |
| 7.2 | **Retrieval quality is proven** without any model present: a teacher can find the right document by keyword and by metadata | Evidence from a real corpus, a real query set |
| 7.3 | Written school decision on whether **external inference is permitted** — for a named service, or ⛔ not at all | Written decision — `CL-AI-001` |
| 7.4 | If external is ⛔ prohibited: **a local machine is available**, or the feature is dropped rather than weakened | Written confirmation — `CL-AI-009` |
| 7.5 | The **data-protection boundary is stated**: what may leave the network, and what may not | Written policy — `CL-SEC-011` |
| 7.6 | **Prompt-injection defences are designed as segregation, not filtering** — retrieved content is data, never instruction | Design review — `CL-AI-006` |
| 7.7 | The assistant is **propose-only**: it may draft; deterministic code validates; a human approves | Design review — `D-007` |
| 7.8 | **Conversation retention** is stated by the school | Written policy — `CL-SEC-025` |

⚠️ **Criterion 7.2 is the gate most likely to be skipped, and it is the one that
decides whether the feature is worth building at all.** An assistant's entire
value is retrieval. If search cannot find the right document, a perfect model
answers the wrong document fluently and confidently — which is worse than no
assistant, because the fluency is indistinguishable from correctness.

⚠️ **Criterion 7.4 records a legitimate outcome: the assistant is dropped.** The
core product does not depend on it. Weakening it — external inference without
permission — is not an option.

**Status: `PENDING`**
**Blocker:** Gate 6 has not passed. No permission has been sought.

---

## GATE 8 — SHANA integration

**⚠️ Built last, and only after the platform is demonstrated working without it.**

| # | Criterion | Evidence required |
|---|---|---|
| 8.1 | Gates 1–5 passed — a **working platform exists without SHANA** | Gate records |
| 8.2 | **Separate written authorisation** for a robot-facing interface | Signed authorisation — `CL-SHANA-001` |
| 8.3 | The **interface is known** — SDK, network API, or manual only, confirmed by the custodian and vendor | Written confirmation — `CL-SHANA-001` |
| 8.4 | **Mutual authentication** is defined and implemented | Design review + test — `CL-SHANA-015` |
| 8.5 | **Idempotency** is defined: a retry must not create a second physical delivery | Test record — `CL-SHANA-018` |
| 8.6 | The robot link is a **separate network path**, sharing no broadcast domain with the classroom LAN | Design review — `FR-SHANA-08` |
| 8.7 | Confirmed that **no representation of motor, path, speed or actuator exists** anywhere in the CloudLight codebase | Code review against ADR-014 |
| 8.8 | Confirmed that **CloudLight operates fully** when SHANA is unreachable | Test record — `D-004` |

⚠️ **Criterion 8.4 is not negotiable.** Without mutual authentication on a robot
link, any host on the school network can move the robot. There is no integration
however convenient; there is only no integration.

⚠️ **Criterion 8.7 is expressed as an absence of capability, not a policy.** A
low-level command path that exists "only for debugging" or "only for the
exhibition" converts a software defect into physical motion. CloudLight is not
qualified to make a safety judgement and must have no way to express one.

⚠️ **Criterion 8.6 exists because a shared broadcast domain produces a fault that
looks like a network fault and is very hard to diagnose.**

**Status: `PENDING`**
**Blocker:** no platform exists, and no interface has been confirmed.

---

## Gate dependency chain

```
GATE 0 ──► everything
   │
   ▼
GATE 1 ──► GATE 2 ──┬──► GATE 4 ──► GATE 5
   │                │      ▲
   │                └──► GATE 3 ┘
   ▼                     ▲
GATE 6 ─────────────────┘
   │
   ▼
GATE 7 (may be NOT APPLICABLE)

GATE 8 (independent path; requires Gates 1–5 passed)
```

⚠️ **Gate 8 does not sit below Gate 7.** It sits below Gate 5 and nowhere else. A
robot integration does not require an assistant, and requiring one would put an
unconfirmed dependency under a safety-adjacent feature.

⚠️ **Gates 6 and 7 can close as `NOT APPLICABLE` at any time**, and doing so
unblocks the project immediately. That is the recommended path when a school has
no shared storage.

---

## Current gate summary

| Gate | Subject | Status | Principal blocker |
|---|---|---|---|
| 0 | Research complete | `PENDING` | No independent review of this documentation |
| 1 | Permission to host | `PENDING` | Nothing requested from the Principal |
| 2 | Network reachability proven | `PENDING` | Nothing requested from IT |
| 3 | Identity decided | `PENDING` | Identity source unknown |
| 4 | First release built | `PENDING` | No implementation exists |
| 5 | Pilot approved | `PENDING` | Gate 4 has not passed |
| 6 | Resource features | `PENDING` | Unknown whether shared storage exists |
| 7 | AI assistant | `PENDING` | Gate 6 has not passed; no permission sought |
| 8 | SHANA integration | `PENDING` | No platform exists; interface unconfirmed |

**0 of 9 gates passed. 0 permissions requested. 0 permissions granted.**

---

## A note on pressure

Every gate in this document exists because there will come a moment when someone
asks for a gate to be skipped — a deadline, an exhibition, a demonstration
opportunity, a teacher who wants it on the board by Monday.

⚠️ **The correct response to that moment is to delay the activity, not to lower
the standard.** A delayed demonstration is a minor inconvenience. A gate skipped
on the way to a demonstration is an incident waiting for an audience.

⚠️ **The two gates that will be hardest to hold are Gate 1 and Gate 4** — one
because it is a request rather than an engineering task, and one because by then
there will be something to show. Neither gets easier under schedule pressure.