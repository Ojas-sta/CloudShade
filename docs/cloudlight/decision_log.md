# CloudLight — Decision Log

**Phase 0 · 2026-10-11**

Records decisions already made, provisional defaults, decisions still open, and
options rejected with reasons. This is a living document — **append, do not
rewrite.**

**Decision states:** `DECIDED` · `PROVISIONAL` (a default that applies unless
contradicted) · `OPEN` (cannot be decided yet) · `REJECTED` (considered and
ruled out, with reason)

⚠️ **Every decision below carries a reversal condition. A decision without one
becomes a decision nobody dares revisit.** Where a decision has not yet been
challenged, that is stated rather than hidden.

---

## 1. Decisions already made

### D-001 — CloudLight is the platform; SHANA is a subsystem within it
**State: DECIDED** · **Date:** 2026-10-11

**Reasoning.** SHANA is a robot delivery subsystem. CloudLight is the school-facing
platform it lives inside — timetable, substitution, display, resource, search and
assistant. Modelling them as peers would force every timetable decision to
accommodate a robot, and every robot decision to accommodate a timetable.

**Evidence.** [`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)
§ "Relationship to SHANA"; [`shana_integration.md`](shana_integration.md) §2.

**What would change it.** A school decision to run SHANA as an independent
product with its own timetable. Unlikely, but it would make the peer model
correct.

**Consequence.** SHANA's failure modes never enter CloudLight's critical path,
and CloudLight ships without SHANA if necessary.

---

### D-002 — This phase produces documentation only
**State: DECIDED** · **Date:** 2026-10-11

**Reasoning.** No implementation, no deployment, no configuration, no network
probing, no hardware touched. Every question blocking an architecture decision
requires a person, a measurement or a signature — none of which research can
supply, and several of which would be improper to seek without permission.

**What would change it.** Written authorisation from the school to deploy
anything at all — which does not yet exist.

**Consequence.** Nothing in this repository may be described as implemented,
tested or verified. Where a control is described, it is a plan.

---

### D-003 — CloudLight never controls robot motors
**State: DECIDED** · **Date:** 2026-10-11
⚠️ Inherited from SHANA ADR-001. Not a CloudLight decision — a SHANA safety
constraint that CloudLight inherits wholesale.

**Reasoning.** A low-level command path existing anywhere in the chain, even
"just for debugging" or "just for the exhibition", converts a software defect or
an intrusion into physical motion in an occupied corridor. The boundary is
therefore expressed as an **absence of capability**, not as a policy that could
be waived under pressure.

**What would change it.** Nothing. This is not revisable within CloudLight.

**Consequence.** CloudLight's entire vocabulary toward SHANA is *deliver this
approved item to this approved destination within this window*. It has no
representation of motor commands, gait, velocity, waypoints, localisation,
battery thresholds, obstacle avoidance or emergency stop. See
[`shana_integration.md`](shana_integration.md) §1.

---

### D-004 — CloudLight operates fully when SHANA is offline
**State: DECIDED** · **Date:** 2026-10-11

**Reasoning.** The timetable is a wall sign. A wall sign that depends on a robot
is a wall sign that goes blank when the robot has a flat battery or a blocked
corridor. Robot availability is a *feature*, never a *dependency*.

**Evidence.** `classroom_dashboard.md` §8; `shana_integration.md` §7.1.

**What would change it.** Nothing sensible. A design in which a timetable
render requires robot state would need re-architecting from the display
component outward.

**Consequence.** No CloudLight feature may block, slow or fail because a robot
is unreachable. The robot panel shows a clearly unavailable state, or is hidden.

---

### D-005 — A six-digit code may only authorise one action or act as a second factor
**State: DECIDED** · **Date:** 2026-10-11

**Reasoning.** A million possibilities is not an authentication factor. Under
classroom time pressure, any credential becomes the credential people actually
use — within weeks it becomes the room's password, written on a board, passed
between staff, photographed by a student.

**Evidence.** `authentication_and_authorisation.md` §5, §6, §7;
`CL-CODE-002`, `CL-CODE-015`.

**What would change it.** A decision that some action *is* low-risk enough for a
code alone — made per action by the school, in writing, and never for anything
consequential. Even then, it is authorisation, not authentication.

**Consequence.** Codes are short-lived, single-use, rate-limited, and absent from
logs, URLs, screenshots and configuration. ⛔ No per-classroom shared code.

---

### D-006 — Deterministic parsing precedes AI; AI never invents a missing value
**State: DECIDED** · **Date:** 2026-10-11

**Reasoning.** Most timetable and substitution sources are printed tables with
declared headers. Those are parseable exactly, by a program, every time. Handing
a probabilistic system a problem a deterministic parser solves perfectly is
strictly worse: it introduces failure modes, costs more, and cannot be tested
exhaustively.

**Evidence.** `timetable_and_substitutions.md` §4 (parsing tiers in strict
order); `CL-DEC-012`, `CL-AIML-013`.

**What would change it.** Nothing in the current design. OCR and vision are
reachable only for sources no deterministic tier handles — and even then the
result is always reviewed by a person beside the original image.

**Consequence.** A missing value stays missing and is flagged. It is never
guessed, inferred, or completed from context.

---

### D-007 — No AI may publish timetable changes, approve substitutions, grant permissions, or issue robot commands
**State: DECIDED** · **Date:** 2026-10-11

**Reasoning.** Each of these is irreversible once it reaches a wall screen, a
staff record or a robot. A model that can act on a bad inference converts a
plausible error into a visible or physical one.

**Evidence.** `ai_assistant.md` §1, §10; `CL-AI-007`, `CL-AI-024`, `CL-DEC-014`.

**What would change it.** Nothing at the current confidence level of the
technology. The assistant's capability is deliberately scoped so that it *cannot*
act, rather than instructed not to.

**Consequence.** The assistant prepares a draft. A deterministic backend
validates it. A named human confirms it. The model has no pathway to effect.

---

### D-008 — Human approval gates publication of timetable and substitution data
**State: DECIDED** · **Date:** 2026-10-11

**Reasoning.** A wrong substitution names a real teacher in a real classroom and
is seen by their students. It is visible, embarrassing and hard to withdraw. The
cost of a missed publication is a display briefly out of date; the cost of a
wrong one is a person's standing on a wall.

**Evidence.** `timetable_and_substitutions.md` §6, §10, §11; `CL-SUB-003`,
`CL-SUB-023`, `CL-ADMIN-003`.

**What would change it.** A school decision that publication may be
self-approved for low-risk classes, made in writing and per class, never by
default.

**Consequence.** Three states that never conflate: *proposed*, *approved*,
*published*. ⛔ Never publish unapproved. ⛔ Never auto-select a substitute.

---

### D-009 — The classroom display is read-only, unauthenticated, and fails closed with an honest staleness marker
**State: DECIDED** · **Date:** 2026-10-11

**Reasoning.** A display is a wall sign. In its signed-out state it must show
PUBLIC-class content only — no staff surface, no names, no confidential material
— because whoever stands in front of it is not necessarily a teacher. And when it
cannot reach the server it must **say so**, because a screen showing yesterday's
data with no marker is a confident falsehood that looks exactly like the real
thing.

**Evidence.** `classroom_dashboard.md` §6, §7; `security_and_privacy.md` §2;
`CL-DASH-012`, `CL-DASH-013`, `CL-SEC-001`.

**What would change it.** ⛔ Nothing may relax this. The design is the control —
there is no configuration that makes a shared wall display confidential.

**Consequence.** Every displayed value carries its age. A last-synced timestamp
is always visible. The countdown is hidden entirely when the clock is unsynced. A
degraded state is a designed screen, not an error page.

---

### D-010 — School configuration and data live outside the repository
**State: DECIDED** · **Date:** 2026-10-11

**Reasoning.** A repository is a development artefact that will be cloned,
forked, backed up and — if it is ever public — read by people the school did not
authorise. Timetables, substitution records, staff availability, credentials,
certificates and signing keys must never travel with it.

**Evidence.** `CL-GIT-019`, `CL-GIT-014`, `CL-GIT-015`.

**What would change it.** Nothing. Even a private repository is cloned, forked
and mirrored by tooling, and git history cannot be cleanly rewritten.

**Consequence.** The repository holds **code and documentation only**. Runtime
configuration, secrets and all school data are provisioned out of band.

---

## 2. Provisional defaults

⚠️ **Each of these applies automatically unless contradicted.** They exist so that
an unanswered question cannot block the project — and cannot quietly become an
optimistic assumption.

### D-021 — A modular monolith, not microservices
**State: PROVISIONAL** · **Date:** 2026-10-11

**Sizing argument.** One school. A few dozen displays. A single database. **No
operations staff** — the school has few technical staff, and the most optimistic
answer is one person at short notice. Microservices exist to let teams deploy
independently and scale independently. With one team, one deployment target and
one operator, that benefit is zero, while the costs — network calls that can
fail, partial states that must be reasoned about, a distributed rollback — are
all paid in full. ⚠️ **The failure mode is not "too simple". It is a system that
requires a competent distributed-systems engineer to keep running, in a school
with none.**

**Reversal condition.** Reconsider only if the school operates **multiple
sites**, or the platform is extended to something whose load genuinely requires
independent scaling. Not on preference, and not on fashion.

**Consequence.** One deployable unit. Modules with clear internal boundaries, so
extraction later stays possible without it being designed for now.

---

### D-022 — Browser-first, not a packaged desktop client
**State: PROVISIONAL** · **Date:** 2026-10-11

**Reasoning.** The classroom machine is a Windows board that may be locked down,
may lack local administrator rights, and may be replaced without notice. A
packaged installer needs privileges and survives nothing; a browser needs a
browser and updates itself. It is also the client needing the **least** privilege
— which matters more than convenience on a locked-down estate.

**Reversal condition.** Reconsider if boards have **no supported browser**, if
kiosk locking cannot be achieved in-browser, or if a Windows feature update
breaks rendering irrecoverably.

**Consequence.** The display client is a web application. Desktop packaging is
rejected as the default, not as an impossibility.

---

### D-023 — A single relational database, and no additional datastore until a documented need appears
**State: PROVISIONAL** · **Date:** 2026-10-11

**Reasoning.** Timetable slots, substitutions, teachers, classes, rooms,
permissions and audit records are all relational and all need transactional
integrity together. Splitting them across a document store, a graph store and a
time-series store buys nothing at this scale and costs consistency guarantees
that the correctness requirements actually need.

**Reversal condition.** Add a second datastore only when a **specific, named
requirement** cannot be met relationally — full-text search over a corpus large
enough to matter being the likely first, and even that is not yet established.

**Consequence.** Backup, restore and consistency reasoning apply to one thing.

---

### D-024 — Local inference preferred over external AI
**State: PROVISIONAL** · **Date:** 2026-10-11

**Reasoning.** ⚠️ This is a **default**, not a finding. External AI is not
assumed permitted — permission has not been sought and may be refused outright.
Local inference is the answer that remains compliant if nobody answers.

**Reversal condition.** Explicit written school approval for a named external
service, configured not to retain prompts or completions, together with an answer
to which half may leave the network — the question only, or the retrieved
document text as well.

**Consequence.** If refused and no local machine is fundable, the assistant is
**dropped**, not weakened. The core product does not depend on it.

---

### D-025 — Signed update artefacts, even on a LAN
**State: PROVISIONAL** · **Date:** 2026-10-11

**Reasoning.** The school LAN is assumed untrusted, and on a flat network that is
probably correct: anyone who can write to a share or run code on one board can
replace a release file. If a board installs whatever it is handed, one compromise
reaches every classroom.

**Reversal condition.** Relax only if the school can demonstrate genuine network
segmentation **and** write it into an acceptable-use policy. Note this does not
justify removing signing — it only changes who holds the key.

**Consequence.** A well-formed but unsigned build is refused. Key custody,
rotation and school revocation authority are documented and rehearsed before the
first deployment.

---

### D-026 — Permission re-verification at open time
**State: PROVISIONAL** · **Date:** 2026-10-11

**Reasoning.** Permission is evaluated when the index is built, but a file's
permission changes over a school year. A search-time check alone means a result
advertises a filename, title and path the user may no longer open — the metadata
itself is the leak.

**Reversal condition.** Relax to search-time-only if the storage can guarantee
immutable permissions, which no school filesystem does.

**Consequence.** The permission check happens again at the moment of opening,
and index entries are deleted when the source is removed.

---

## 3. Decisions still open

### D-101 — Where does the backend physically run?
**State: OPEN**

**Blocks:** everything.

**The question that unblocks it.** `CL-SRV-001` and `CL-SRV-002` — is a
persistent always-on service permitted on school premises, and where?

**Default if unanswered:** no deployment. ⚠️ **A cloud-hosted VM is a scope
change, not a fallback** — it breaks the local-first premise and needs explicit
school agreement.

---

### D-102 — Which identity source?
**State: OPEN**

**Blocks:** the authorisation model, the audit trail, session design.

**The question that unblocks it.** `CL-AUTH-001`, `CL-AUTH-002` — does a staff
directory exist, may CloudLight create and own a minimal one, and is SSO
permitted?

**Default if unanswered:** a minimal locally-owned account set maintained from
the school's own staff list. ⚠️ If even that is refused, the consequence must be
stated to the school in writing: shared accounts and **no attributable audit
trail**.

---

### D-103 — Which search engine?
**State: OPEN**

**Blocks:** the resource browser and the assistant's retrieval.

**The question that unblocks it.** `CL-SEARCH-014`, `CL-NAS-003` — corpus size,
and which sharing protocols are enabled.

**Default if unanswered:** nothing is indexed. ⚠️ The corpus size is the deciding
number and it is unknown; a full-text engine is justified by a corpus large
enough to matter, and that has not been shown.

---

### D-104 — Is OCR in scope?
**State: OPEN**

**Blocks:** substitution ingestion from photographs, and indexing of scanned PDFs.

**The question that unblocks it.** `CL-SUB-006`, `CL-SUB-007` — may sheets be
photographed, and is a photograph ever authoritative? Plus `CL-SEARCH-004`,
`CL-SEARCH-005` — how much is scanned, and is derived text permitted?

**Default if unanswered:** ⛔ OCR is out of scope. A photograph is never
authoritative, and manual entry through a fast form is provided instead.

---

### D-105 — Is AI ever in scope?
**State: OPEN**

**Blocks:** the assistant, and nothing else.

**The question that unblocks it.** `CL-AI-001`, `CL-AI-009`, `CL-AI-011` — is
external inference permitted, is there a local GPU-capable machine, and if
external is prohibited, is the feature dropped or funded?

**Default if unanswered:** ⛔ Not in scope. ⚠️ **The core product is unaffected**,
which is exactly why the assistant is built last.

---

### D-106 — What is the SHANA interface?
**State: OPEN**

**Blocks:** robot integration, and only robot integration.

**The question that unblocks it.** `CL-SHANA-001`, `CL-SHANA-015` — which
interface exists and is permitted, and how does the robot verify a request came
from this school's CloudLight?

**Default if unanswered:** ⛔ No integration. Build the platform without SHANA and
add it last, if ever. ⚠️ **Mutual authentication is not negotiable** — without it
there is no integration, however convenient.

---

## 4. Options rejected

### X-01 Microservices
⛔ **Rejected.** One team, one deployment target, one operator, no operations
staff. Distributed calls can fail and partial states must be reasoned about, for
zero benefit at this scale. See D-021.

### X-02 An access point or router hosting the backend
⛔ **Rejected in principle**, before any school-specific consideration. An AP is
a Layer-2 bridge with no application runtime, no persistent storage we may use,
and no firmware update path we control. Port forwarding to a router is not
hosting — it requires a host that already exists. See `network_and_hosting.md`
§2, §3.6.

### X-03 A smartboard acting as the server
⛔ **Rejected** for five independent reasons, any one sufficient: boards sleep,
restart mid-lesson, lock and are switched off; the host would sit in front of a
class; boards are shared, so a persistent server identity is a credentials
problem; nobody is on the hook for patching a classroom PC as a server; and
recovery means a walk to a classroom during a lesson. See
`network_and_hosting.md` §3.5.

### X-04 Packaged desktop clients on every board
⛔ **Rejected as the default.** Installation requires privileges that may not
exist, and a package that fails mid-lesson needs someone to visit the room. See
D-022.

### X-05 External AI inference by default
⛔ **Rejected.** Permission has not been sought and may be refused. Assuming it
would send staff and timetable data to a third party on the strength of an
assumption. See D-024, `ai_assistant.md` §5.

### X-06 Photographing classroom boards as an authoritative source
⛔ **Rejected.** A photograph is an input, never a decision. A misread is
structurally plausible and looks correct — a `7` for a `1`, a struck-out cell
read as valid. See D-008, `timetable_and_substitutions.md` §8.

### X-07 A shared six-digit classroom code as identity
⛔ **Rejected.** See D-005. It is authorisation, never authentication, and never
a shared per-classroom credential.

### X-08 Indexing confidential staff material
⛔ **Rejected.** An index is a copy of restricted content that outlives the
permission check on the file. Index an explicitly approved folder list, never a
share. See `risk_register.md` R-28, `CL-NAS-011`, `CL-NAS-012`.

### X-09 Automatic publication without human approval
⛔ **Rejected.** The cost of a missed publication is a display briefly out of
date; the cost of a wrong one is a teacher's standing on a wall. See D-008.

### X-10 CloudLight controlling the robot
⛔ **Rejected permanently.** Inherited from SHANA ADR-001. Not revisable within
CloudLight. See D-003.

### X-11 Coupling the timetable to robot availability
⛔ **Rejected.** The timetable is a wall sign and must behave like one. See D-004.

### X-12 The AI assistant built first
⛔ **Rejected.** Its value is entirely retrieval; if search cannot find the right
document, a perfect model answers the wrong document fluently. Its safety depends
on permission correctness being already solved. See `ai_assistant.md` §15.

---

## 5. How to change a decision

### 5.1 The rules

| Rule | Detail |
|---|---|
| **Append, never rewrite** | A superseded decision stays in the log with its state changed to `REJECTED` or `SUPERSEDED`, and a pointer to the new one. History is the point |
| **Record the reversal condition at the time of deciding** | Not when it is convenient. A decision written without one cannot later be examined |
| **A decision with no reversal condition is unexamined** | ⚠️ It has not been tested against the condition that would break it. Treat it as provisional regardless of the state written beside it |
| **State changes are events** | Each state change is a dated entry, not an edit of the original |
| **Provisional decisions carry a default** | If a question is unanswered, the default applies automatically — so an unanswered question cannot silently block the project, and cannot silently become an optimistic assumption |
| **Nobody may decide for the school** | ⚠️ **A decision requiring school authority is recorded as `OPEN` until the school answers in writing.** This document has no authority to accept anything on the school's behalf |

### 5.2 What changing a decision looks like

A decision changes by being **superseded**, in this shape:

```
### D-0nn — <title>
**State: DECIDED → SUPERSEDED** · **Date:** <ISO date>
**Superseded by:** D-0mm
**Why:** <the evidence or answer that changed it>
```

⚠️ **Deleting a decision is forbidden.** The value of this log is that someone
in a year's time can see what was believed, on what evidence, and why it
changed. A log without that history is just a list of current opinions.

### 5.3 What this log does not record

| Not recorded here | Where it belongs |
|---|---|
| Questions awaiting answers | [`open_questions_register.md`](open_questions_register.md) |
| Risks and their ratings | [`risk_register.md`](risk_register.md) |
| Detailed design reasoning | The topic document — `network_and_hosting.md`, `security_and_privacy.md`, and the rest |
| Anything requiring school authority | ⚠️ **Nowhere. It must be asked for.** |

---

## 6. Log conventions

| Field | Content |
|---|---|
| Date | ISO format |
| Decision state | `DECIDED` / `PROVISIONAL` / `OPEN` / `REJECTED` |
| Owner | Named role |
| Evidence | Source document, `CL-*` identifier, or measurement |
| Assumptions | What was assumed true when deciding |
| Reversal condition | What would cause this to be revisited |
| Consequences | What this commits us to |

⚠️ **Decisions made in Phase 0 will look different once a measurement exists.**
That is not a failure of the decision — it is what a provisional decision is for.
Write the reversal condition now, while the reasoning is still fresh.