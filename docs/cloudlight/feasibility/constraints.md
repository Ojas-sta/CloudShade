# CloudLight — Constraints

**Phase 0 · Documentation only · 2026-10-11**

The walls inside which every CloudLight requirement must be written.

> **⚠️ A constraint is not a requirement to be negotiated.** It follows from a
> decision already accepted in this repository, from a documented platform limit, or
> from a safety boundary. Constraints are the walls; requirements are the rooms
> built between them.

⚠️ **These are distinct from assumptions.** [`assumptions.md`](assumptions.md)
records what we are *hoping* is true, so that we can find out. This document
records what is *already binding* regardless of what anybody answers. An
unanswered assumption may turn out to be false and the design changes. A
constraint does not change because someone is busy.

The numbered list below restates and extends §4 of
[`../requirements.md`](../requirements.md), which remains the authoritative
register of `C1`–`C12`. This document adds the constraints that emerged while
writing the feasibility work, and adds §3 — the constraints created by the
**absence of staff**, which is not a fact about the school but a fact about what
can realistically be supported.

---

## 1. The binding constraints

| ID | Constraint | Source of the constraint | Non-negotiable? | What it forbids |
|---|---|---|---|---|
| **C1** | **Local-only operation by default.** Core features do not require the public internet | `requirements.md` C1; `network_and_hosting.md` §7.2; `D-010` | ✅ Yes — unless the school rewrites the premise | Any design where a lesson depends on an external service. Also forbids "we will use a cloud database" as an opening move |
| **C2** | **No external AI service without written authorisation** — for that named service, from a named person | `requirements.md` C2; `CL-SEC-011`, `CL-GEN-004`; `D-024` | ✅ Yes | "We will try it and see". Asking the permission question *by sending school content to the service* is prohibited — that act is the thing being asked about |
| **C3** | **CloudLight never controls robot motors.** No motor, path, speed, actuator, waypoint or localisation command may cross the boundary | ADR-014; `requirements.md` C3; `D-003` | ✅ Absolute — **not revisable by the CloudLight team** | Any CloudLight-side safety reasoning about the robot. The boundary is an *absence of capability*, so no debug channel, no maintenance channel, no exhibition shortcut |
| **C4** | **SHANA validates robot safety independently.** Safety ownership never transfers | ADR-014; `requirements.md` C4; `CL-SHANA-003` | ✅ Yes | CloudLight asserting, pre-empting, inferring or influencing a safety judgement. Also forbids offering any fallback that could degrade a SHANA safety function |
| **C5** | **No secret in Git, in a URL, in a log, or in browser local storage** | `requirements.md` C5; `CL-SEC-006`, `CL-GIT-014`; `D-010` | ✅ Yes — zero tolerance | Convenience. A secret that reaches version control must be **assumed compromised** and rotated, not edited out |
| **C6** | **A school LAN is not trusted.** It is an internal network shared with student devices | `requirements.md` C6; `network_and_hosting.md` §6.1 | ✅ Yes | Relying on "it's internal" as a security control. Consequence: releases are signed, administration surfaces are separated, and authorisation is enforced server-side regardless of network location |
| **C7** | **The classroom display is read-only and unauthenticated** | `requirements.md` C7; `D-009`; `FR-DASH-01`, `FR-DASH-02` | ✅ Yes | Any write path from a display. Any session held open on a board. Any configuration that makes a shared wall display confidential |
| **C8** | **Publication of timetable and substitution data requires human approval** by a named person | `requirements.md` C8; `D-008`; `CL-SUB-003` | ✅ Yes | Fully automatic publication. ⚠️ Note this constrains *publication*, not *proposal* — machine drafts remain permitted, and must be visibly marked as unconfirmed |
| **C9** | **Software updates and data updates use separate approval paths** | `requirements.md` C9; `FR-OTA-04`; `CL-OTA-019`, `CL-OTA-001` | ✅ Yes | A single "deploy everything" pipeline. A data correction is urgent and routine; a software change is neither. One path forces one of them to be wrong |
| **C10** | **No student-identifying data on a student-facing display or in a delivery request** | `requirements.md` C10; `../safety/student_privacy.md` (SHANA); `CL-SEC-010`, `CL-DASH-003` | ✅ Yes | Individualised dashboards. Attendance on a wall. Period. This constraint is inherited wholesale from SHANA's no-identification invariant and is **not relaxed by the CloudLight team** |
| **C11** | **No AI output may reach the robot**, by any path — including tool use or function calling | ADR-001 (inherited via ADR-014); `FR-AI-04` | ✅ Yes | Any assistant capability that could construct a robot-bound message. The constraint is expressed as an inability, not an instruction |
| **C12** | **The classroom smartboard is a client, never the server** | `requirements.md` C12; `deployment_options.md` §2.5; `X-03` | ✅ Yes | Peer-hosted and board-hosted designs — which also fail on client isolation |
| **C13** | **Scanning, probing or port-scanning the school network is prohibited** | `requirements.md` C11; `../infrastructure/school_network.md` (SHANA) | ✅ Yes | Resolving any network unknown by ourselves. IT checks what needs checking, and we ask. **This constraint is what makes `CL-NET-003` and `CL-NET-008` questions rather than tests** |
| **C14** | **Deterministic parsing precedes AI; AI never invents a missing value** | `D-006`; `timetable_and_substitutions.md` §4 | ✅ Yes | Handing a probabilistic system a problem a parser solves exactly. A missing value stays missing and is flagged — it is never guessed, inferred or completed from context |
| **C15** | **School configuration and school data live outside the repository** | `D-010`; `CL-GIT-019` | ✅ Yes | Timetables, substitution records, staff availability, credentials, certificates and signing keys travelling with the source. A repository gets cloned, forked and mirrored, and git history cannot be cleanly rewritten |
| **C16** | **A six-digit code may authorise one action or act as a second factor. It may never authenticate a person** | `D-005`; `CL-CODE-002`, `CL-CODE-015` | ✅ Yes | A shared per-classroom credential. A flow designed to *look* like authentication — a code prompt, a green tick, a "signed in as…" message — because that is how an audit trail becomes fictional while still appearing to exist |
| **C17** | **The display fails closed.** When it cannot reach the host it must say so, visibly and with a timestamp | `D-009`; `FR-DASH-06`, `FR-DASH-07`; `CL-DASH-012`, `CL-DASH-013` | ✅ Yes | Silent staleness. It is indistinguishable from "nothing is scheduled" and is a confident falsehood that looks exactly like the real thing. A blank screen is also forbidden — last-known data, clearly marked, always beats it |
| **C18** | **An outage may never make anything more accessible** | `security_and_privacy.md` T19 | ✅ Yes | Degrading to an "open" mode when the server is unreachable. ⚠️ This is the subtle one: a system that fails *open* has silently converted an authenticated system into an unauthenticated one, and a board stuck on a cached privileged view is a disclosure with a long tail |

---

## 2. Constraints that are about *what this project is*

These are not implementation limits. They define what CloudLight is by
prohibiting what it is not.

| ID | Constraint | Source | What it forbids |
|---|---|---|---|
| **C19** | **CloudLight is a consumer and renderer of the school's authoritative record**, not a second authoritative copy | `FR-TIME-01`, `FR-TIME-14`; `D-010` | Two live copies, which diverge. Also forbids writing back to the school's authoritative timetable system — that is an ownership question we have no answer to (`CL-TIME-001`, `CL-DATA-019`) |
| **C20** | **Every displayed value carries its age** | `D-009`; `NFR-OFF-03` | A display whose freshness cannot be judged by the person in the room |
| **C21** | **No capability is described as operational before it is demonstrated** | `../overview.md` §7; ground rule 1 | Documentation phrased as delivery status. This constraint is self-applied and exists because 30 documents about a system with no code is exactly the condition that produces that phrasing |
| **C22** | **A gate is never marked verified without evidence** | `../overview.md` §7; [`go_no_go_gates.md`](go_no_go_gates.md) | "Verified" meaning "a conversation happened". ⚠️ `ANSWERED` means a person replied; `VERIFIED` means evidence exists |
| **C23** | **The project team may not decide for the school** | `decision_log.md` §5.1 | Accepting any answer on the school's behalf. A decision requiring school authority stays `OPEN` until the school answers in writing |
| **C24** | **No scanning of, or unauthorised contact with, school infrastructure** — including the robot | C13; `../infrastructure/school_network.md` | Testing a real board, probing the network, or contacting a vendor on the school's behalf. Testing happens only on a machine IT has named as non-critical, with permission (`CL-WIN-007`) |

---

## 3. ⚠️ Constraints imposed by the absence of staff

> **This is a real operational constraint and it shapes the architecture more than
> any technical limit in §1.** It is not a fact about the school that we have
> confirmed; it is a fact about what a project of this size can reasonably assume
> anyone will maintain.

The most optimistic staffing assumption is **one IT generalist with other duties**.
No operations team. No on-call rota. No redundancy budget. No second person who
could read the same code.

⚠️ **This assumption is load-bearing for every architecture decision in this
repository**, and it is stated plainly rather than hidden because it is the kind of
thing that looks like modesty and is actually a constraint that forecloses options.

### 3.1 What one person cannot be asked to do

| Constraint | What it rules out | Where it bites |
|---|---|---|
| **C25 — One person, not a team** | Any architecture whose *normal state* requires coordinated human action across services | ADR-013 (modular monolith); `architecture.md` §2.2 |
| **C26 — No on-call** | Any design whose recovery requires a decision at 03:00 | Fail-closed offline behaviour (`C17`); rehearsed restore rather than failover |
| **C27 — No distributed-systems expertise** | Microservices, service discovery, distributed tracing, coordinated rollouts, inter-service version compatibility. Each is a concept that person must also learn and operate, forever, on top of their job | `architecture.md` §2.2; `X-01` |
| **C28 — No spare capacity** | Anything requiring an operator to be present during a lesson | Release staging on a nominated board for a school day, not a full fleet rollout |
| **C29 — One machine, one room, one circuit** | High-availability architecture of any kind | Accepted as a **deliberate single point of failure** (`R-01`), bought back with speed of recovery: off-host encrypted backups, a rehearsed restore, and displays that degrade honestly without the server |
| **C30 — Documentation is the only handover** | Tribal knowledge, and "the person who built it understands it" | `NFR-MNT-02`: common failures diagnosable from written instructions by non-technical staff, **tested, not assumed** |
| **C31 — Restore, not failover** | Clustering, replication, automatic failover | `deployment_options.md` §5 Stage 3: a redundant host, or a **proven** restore |
| **C32 — Supportability is a security property** | Undocumented admin paths, hidden toggles, debug affordances | If a control exists only as knowledge in one person's head, it is not a control. It is a secret with extra steps — and `C5` forbids secrets |

### 3.2 What this rules out that most systems assume

| Normally assumed | Not assumed here | Why |
|---|---|---|
| Someone is on call | ❌ | `C26` |
| The configuration is understood by a second person | ❌ | `C30` |
| A failed deploy can be debugged under pressure | ❌ | `C25` |
| A backup exists and someone has restored it | 🟡 Assume **nothing until a restore has been rehearsed** | A backup that has never been restored is a file, not a plan (`NFR-REC-03`) |
| Operating the system is part of someone's job | ❌ | `CL-OPS-022` — the year-two owner is the single most important unanswered question in the register |

⚠️ **The consequence that matters:** `CL-OPS-022` returning *"nobody"* should
**stop the project**, not shrink it. Everything else in
[`../open_questions_register.md`](../open_questions_register.md) §6 ends in a
smaller project, which is a correct and useful outcome. A system with no year-two
owner ends in a screen that quietly stops updating — which is worse than not
building it, because it destroys trust in the one thing the school relies on.

---

## 4. What these constraints buy

Constraints are not only prohibitions. Each one removes a class of failure that
would otherwise need to be engineered against later.

| Constraint | Failure it makes impossible |
|---|---|
| `C3` · `C4` · `C11` | A software defect becoming physical motion in an occupied corridor |
| `C8` | A wrong teacher named on a classroom wall |
| `C13` · `C24` | A project committing a serious act against infrastructure nobody administers |
| `C14` | A plausible invented value passing as a fact |
| `C17` · `C18` | An outage, a stale screen, or a cached privileged view becoming a silent disclosure |
| `C19` | Two diverging timetables, and losing staff trust the fastest way available |
| `C25`–`C32` | Building a system that requires a competence the school does not have, permanently |

---

## 5. How a constraint is changed

| Rule | Detail |
|---|---|
| **A constraint is not relaxed by a decision** | It is superseded by a **new, dated entry** in [`../decision_log.md`](../decision_log.md), with the evidence and the authority that permitted it |
| **`C3`, `C4` and `C11` are not ours to change** | They are inherited from SHANA's safety case. A change is a SHANA decision and is not reversible by the CloudLight team under any circumstances |
| **A constraint never becomes an assumption** | If a constraint turns out to be inconvenient, the honest response is to design within it. Converting a wall into a hope is how a safety boundary is lost |
| **Every constraint carries a question or a source** | A constraint with neither is a preference wearing a wall's clothing, and should be demoted to a proposal in [`../requirements.md`](../requirements.md) |

⚠️ **C21 and C22 are self-applied, and they are the two most likely to erode.**
They constrain how *this documentation* is written, not what the software does —
which means they can be violated by a well-meaning sentence rather than by a bug,
and nobody will notice in a test.

---

## 6. Related documents

| Document | Covers |
|---|---|
| [`../requirements.md`](../requirements.md) | §4 is the authoritative `C1`–`C12` register; §5 the data entities |
| [`assumptions.md`](assumptions.md) | What we are hoping is true, so that we can find out |
| [`go_no_go_gates.md`](go_no_go_gates.md) | The checkpoints that convert assumptions into evidence |
| [`../security_and_privacy.md`](../security_and_privacy.md) | Threat model, data classification, `T19`, logging, accepted risks |
| [`../shana_integration.md`](../shana_integration.md) | The CloudLight/SHANA ownership boundary in detail |
| [`../architecture.md`](../architecture.md) | How these constraints shape the proposed architecture |
| [`../decision_log.md`](../decision_log.md) | Where a constraint change would be recorded |
| [`../../shana/safety/student_privacy.md`](../../shana/safety/student_privacy.md) | SHANA's no-identification invariant, inherited by `C10` |