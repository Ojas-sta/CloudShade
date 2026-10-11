# CloudLight — Goals and Non-Goals

**Phase 0 · 2026-10-11**

Two lists. The first is what CloudLight is trying to achieve. The second is what
it will deliberately **not** do.

⚠️ **A non-goal recorded here is a decision, not a deferral.** Every non-goal
carries a one-line reason and, where one exists, the question or evidence that
would reopen it. A non-goal with no reason is a decision nobody made.

---

## 1. Goals

| # | Goal | What it means concretely | How we would know |
|---|---|---|---|
| **G-01** | **Put trustworthy classroom information on a wall** | Current lesson, subject, room, teacher and times, shown without asking a human | A teacher reads the screen and acts on it correctly |
| **G-02** | **Make a substitution reach every affected room correctly and quickly** | One entry, one named approval, one propagation — with no transcription between decision and screen | The substitute arrives at the right room without a phone call |
| **G-03** | **Make a wrong publication withdrawable** | A wrong name comes off every affected display, recorded, with the correction propagated | A rehearsal demonstrates withdrawal and rollback end to end |
| **G-04** | **Never show stale data as current** | Every displayed value carries its age; a display that cannot reach the host says so visibly | A teacher can tell, at a glance, how fresh the screen is |
| **G-05** | **Keep school data on school premises** | Local-first operation; nothing leaves the network by default | The system runs with the uplink unplugged for a week |
| **G-06** | **Require human authority before anything is published** | One approval gate, human, attributed, timestamped, reversible | Every published record has a named approver and a retained source document |
| **G-07** | **Make every consequential action attributable to a named person** | Individual accounts; a shared code may authorise one action or act as a second factor, never identify a teacher | The audit log answers *"who did this"* for a year |
| **G-08** | **Be operable by one person with other duties** | One process, one database, one host, restore instructions a non-specialist can follow | IT restores service from written instructions without calling anyone |
| **G-09** | **Degrade honestly rather than fail silently** | Cached PUBLIC-class data with a staleness marker; never blank, never the last privileged screen | Every degraded state is a designed screen, not an error page |
| **G-10** | **Never put a motor command within CloudLight's reach** | The boundary is an *absence of capability* — no motor, path, speed, actuator or localisation command exists in any code path | Code review confirms no such representation exists |
| **G-11** | **Keep the robot optional forever** | A stalled, absent or broken SHANA changes nothing a teacher sees | The timetable is identical with SHANA powered off |
| **G-12** | **Make the school's authoritative record the authority** | CloudLight consumes and renders; it does not become a second source of truth | On disagreement, the school's file wins and a human reconciles |
| **G-13** | **Show a student nothing identifying** | No names, marks, grades, attendance or photographs on a student-facing display | Every signed-out screen carries PUBLIC-class content only |
| **G-14** | **Extend to later capabilities without rewriting the spine** | The correct-data path is built first; later modules attach to it | Resource search and the assistant, if ever built, need no change to the ingest path |
| **G-15** | **Refuse what it cannot do safely** | Unknown is a valid answer; a plausible invention is not | A missing value stays missing and is flagged, never guessed |

⚠️ **G-01 through G-09 are the goals a first release could plausibly demonstrate.**
G-10 through G-15 are constraints that shape everything, including the goals that
are not yet written.

---

## 2. Non-goals

⚠️ **Each of these is a decision. None is a deferral in disguise.**

| # | Non-goal | One-line reason | What would reopen it |
|---|---|---|---|
| **N-01** | **No student surveillance** | A system that watches, scores or tracks individual students is a different product with a different legal and ethical status, and no school has asked for one | Nothing in this project. It is out permanently |
| **N-02** | **No per-student tracking of any kind** | No attendance, no behaviour, no history, no per-student record — the no-identification invariant is inherited from SHANA and is not relaxed by this team | Nothing in this project. `constraints.md` `C10` |
| **N-03** | **No motor, path or actuator control of any kind** | A software defect on the path to a motor command is physical motion in an occupied corridor. The boundary is an absence of capability | ⛔ **Nothing.** ADR-014, **not reversible by the CloudLight team** |
| **N-04** | **CloudLight will never assert or influence a robot safety judgement** | Safety ownership never transfers, and CloudLight is not qualified to make that judgement | ⛔ **Nothing.** `constraints.md` `C4` |
| **N-05** | **No cloud AI by default** | Sending school material to a third party is a data-transfer decision the school has not made, and must not be made by us | Written school authorisation for a **named** service — `CL-AI-001`, `CL-SEC-011` |
| **N-06** | **No AI that publishes, approves, grants or commands** | These are irreversible once they reach a wall, a staff record or a robot. A model that can act converts a plausible error into a visible or physical one | ⛔ Nothing at the current confidence level. `D-007` |
| **N-07** | **No multi-school support in the first release** | One school, one academic year, one set of answers. Multi-tenancy before a single tenant is proven is a classic inversion | A second school that wants it — `CL-BUD-018` |
| **N-08** | **No replacement of the school's authoritative records** | Two live copies diverge, and the fastest way to lose staff trust is a display disagreeing with the staff-room board | Written school designation of CloudLight as a system of record — `CL-TIME-001` |
| **N-09** | **No write-back to the school's timetable system** | Two-way sync creates an ownership problem we have no answer to | A written data-ownership agreement — `CL-DATA-019` |
| **N-10** | **No automatic publication, ever** | The cost of a missed publication is a display briefly out of date; the cost of a wrong one is a person's name on a wall | A written, per-class, school decision — never by default. `D-008` |
| **N-11** | **No shared classroom code as identity** | 10⁶ possibilities, brute-forceable in a free period. It identifies nobody, cannot be revoked from a person, and makes the audit trail fictional | ⛔ Nothing as identity. It may authorise one action or act as a second factor — `D-005` |
| **N-12** | **No photograph of a classroom board treated as authoritative** | Visibility is not authority, and a misread is structurally plausible — a `7` for a `1`, a struck-out cell read as valid | Written permission **and** a named human confirming every photograph — `CL-SUB-006`, `CL-SUB-007` |
| **N-13** | **No external service in the path of a lesson** | Core features must work with the uplink unplugged for a week | Nothing, unless the school abandons the local-first premise — `constraints.md` `C1` |
| **N-14** | **No secrets in Git, URLs, logs or browser storage** | A secret in version control must be assumed compromised. There is no safe way to un-commit it | ⛔ Nothing. Zero tolerance — `C5` |
| **N-15** | **No writable classroom display** | A student-facing screen with a write path is an unauthenticated write route to the school's timetable | ⛔ Nothing. The display is read-only and unauthenticated — `C7` |
| **N-16** | **No trusting the school LAN** | It is an internal network shared with student devices. "It's internal" is not a security control | Genuine segmentation written into an acceptable-use policy — `C6` |
| **N-17** | **No microservice architecture** | Microservices solve an organisational scaling problem this project does not have, and impose operational concepts on one person who has a day job | Multiple schools, or a genuinely heavy module needing independent scaling — ADR-013 |
| **N-18** | **No robot firmware delivery path** | Firmware flashing is a separate, higher-risk physical activity. Conflating a web bundle refresh with firmware is a category error with safety consequences | ⛔ Nothing. Separately governed — `../architecture.md` §9 |
| **N-19** | **No packaged desktop client by default** | A browser kiosk takes updates from the server; an installer needs privileges on every classroom machine and fails mid-lesson without a visit | Boards demonstrably lack a usable browser — `D-022` |
| **N-20** | **No promises made to the school** | An unstated scope becomes an unstated commitment; a promised feature later withdrawn costs more credibility than one never offered | Never. This one is permanent |

---

## 3. The line between a goal and a non-goal

⚠️ **The distinction is not "important" versus "unimportant". It is "may we change
this unilaterally?"**

| | Goal | Non-goal |
|---|---|---|
| Who may change it | The project team, within the constraints | The school, or nobody — or ⛔ never |
| Evidence needed to change it | Engineering judgement | A written answer, a measurement, or a school decision |
| Status when unanswered | Built, with recorded assumptions | **Not attempted at all** |

⚠️ **Non-goals `N-01`, `N-02`, `N-03` and `N-04` are permanent and are not ours to
reopen.** The first two are an inherited data-protection invariant; the second two
are a safety boundary. The project team can propose changing a goal. It cannot
propose changing these.

---

## 4. Goals that are deliberately not written yet

⚠️ **These are absent on purpose.** Writing a goal for a capability whose
precondition has not been confirmed to exist would convert an unknown into a
commitment nobody agreed to.

| Absent goal | Would require | Status |
|---|---|---|
| **Teachers can find any resource by describing it** | A storage system that exists (`CL-NAS-001`), a permission model (`CL-NAS-009`), a corpus size (`CL-SEARCH-014`) | ❓ None of the three is confirmed |
| **The assistant answers a teacher's question in plain language** | The retrieval goal above to work first, plus a written AI policy (`CL-AI-001`) and hardware (`CL-AI-009`) | ⛔ The assistant's value *is* retrieval |
| **A teacher asks for an item and the robot brings it** | A working platform without the robot (`Gate 5`), a permitted interface (`CL-SHANA-001`), mutual authentication (`CL-SHANA-015`) | ⛔ A robot integration first would put an unconfirmed dependency under the core product |
| **The school maintains it without us** | A year-two owner (`CL-OPS-022`), restore procedures, written diagnostics | ⚠️ **This is the most important absent goal of all**, and the one most likely to be quietly dropped under time pressure |

⚠️ **The fourth is the dangerous one.** `NFR-MNT-01` — *"the school can continue
without the project team"* — is described in
[`../requirements.md`](../requirements.md) §3.9 as **the requirement most likely to
be quietly dropped**, because it has no interface and no demonstration. It is also
the one that decides whether the project was worth doing.

---

## 5. Why the non-goal list is longer than the goal list

⚠️ **This is not pessimism, and it is not a small project.** It is the shape of a
feasibility phase whose unknowns turned out to be properties of a specific school
rather than problems to be solved.

Nine of the sixteen capabilities in the original brief are deferred. Four are
deferred because **the thing they depend on has not been confirmed to exist** —
shared storage, an identity directory, an always-on permitted host, and AI policy.
⚠️ **Every one of those is a question for a person, not a task for an engineer.**
Building against them would mean inventing a school and presenting the invention
as a design.

> **A one-room system that works is worth more than a school-wide system that does
> not.** Seven of the thirteen worst-case answers in
> [`../open_questions_register.md`](../open_questions_register.md) §6 end in a
> smaller project — which is the correct outcome. One does not: *"the year-two
> owner is nobody"* should end the project rather than shrink it.

---

## 6. How to tell whether this document is being honoured

⚠️ **A goals document drifts quietly.** It drifts by addition — a capability gets
suggested, somebody agrees it sounds good, and a row appears. It is worth stating
what using this document correctly looks like.

| If someone proposes… | The correct response |
|---|---|
| "Can we add student attendance?" | ⛔ `N-01`, `N-02`. Not a scope question. **Permanent**, and not ours to reopen |
| "Can we let the robot take priority instructions?" | ⛔ `N-03`, ADR-014. Priority invites a future priority SHANA must honour over its own safety rules |
| "Can we send the query to an AI service to get better answers?" | ⚖️ A data-transfer decision for the Principal — `CL-AI-001`. **And the answer would not unblock the assistant**, whose value is retrieval |
| "Can we auto-publish so substitutions show up faster?" | ⛔ `N-10`. The cost of a missed publication is a display briefly out of date; the cost of a wrong one is a person's name on a wall |
| "Can we support a second school?" | `N-07`. A second school that asks for it — and it is a *different* product with a different tenancy model |
| "Can we just show the most recent file?" | ⛔ Two conflicting instructions are **an event for a human to resolve**. Last-write-wins makes the file ingested last silently correct |
| "Can we make the timetable staff-only on the display?" | ⛔ `N-01`, `C10`. ⛔ **There is no configuration that makes a shared wall display confidential** — the display is read-only and unauthenticated by design |

⚠️ **Two of these are not "no" but "ask the school".** `N-05` and `N-08` can be
reopened by a written answer. The rest cannot be reopened by anybody at all.

---

## 7. Related documents

| Document | Covers |
|---|---|
| [`product_vision.md`](product_vision.md) | Why this exists, for whom, and what would make it a failure |
| [`current_project_state.md`](current_project_state.md) | What exists today — documentation, and nothing else |
| [`../requirements.md`](../requirements.md) | 186 requirements and the `C1`–`C12` constraint register |
| [`../decision_log.md`](../decision_log.md) | Where a decision to change a non-goal would be recorded |
| [`../first_release_scope.md`](../first_release_scope.md) | Release 0 scope, and the twelve exclusions |
| [`../feasibility/constraints.md`](../feasibility/constraints.md) | The binding walls behind most of these non-goals |
| [`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md) | 576 questions. **0 answered** |