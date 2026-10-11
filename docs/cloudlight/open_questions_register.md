# CloudLight — Open Questions Register

**Phase 0 · 2026-10-11**

The questions that must be answered before the architecture can be chosen, with
the party who can answer each one.

---

## 1. Purpose

⚠️ **The authoritative question list is
[`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)
— 576 questions, every one `UNANSWERED`. This register is a filtered view of
it, not a substitute for it.**

The filter is narrow and deliberate. Only questions that **block an
architecture decision** appear here. A question is included only if at least one
of these is true:

- a different answer would produce a **different architecture**, not a different
  configuration value;
- the answer determines whether a capability **exists at all**, rather than how
  it behaves;
- no safe default applies, so silence would be mistaken for permission;
- the answer is a **signature**, not a fact.

⚠️ **Nothing in this register is a research task.** Every entry needs a **person**,
a **measurement**, or a **written approval**. Research has been done as far as it
can be — the remaining unknowns are properties of a specific school, and no
amount of further reading will produce them. **"Unknown" is a valid and valuable
answer**; a guess is not.

Each blocking question carries a **conservative default** — the assumption that
applies automatically if nobody answers. Defaults are recorded so that an
unanswered question cannot silently become an optimistic assumption, and cannot
silently block the project either.

---

## 2. The blocking set

These eighteen determine whether the architecture can be chosen at all.

| # | ID | Question | Blocks what | Who can answer | If unanswered — conservative default |
|---|---|---|---|---|---|
| **OQ-1** | `CL-GEN-001` | Who holds the **authority to approve or refuse** a system storing student, staff and timetable data on school infrastructure? | ⚠️ **Everything.** No lawful basis, no project | Principal / Administration | ⛔ **Project cannot start.** Nothing may be stored |
| **OQ-2** | `CL-SRV-001` | Is the school willing and permitted to host a **persistent always-on service** on its own premises? | ⚠️ **The entire platform.** No host, no platform | Principal / Administration | ⛔ **No deployment.** Design remains hypothetical |
| **OQ-3** | `CL-NET-003` | Is the **server subnet permitted to accept connections** from the classroom VLAN or wireless segment? | ⚠️ **Every display.** Boards that cannot reach the server show nothing | School IT | Assume **no cross-subnet access.** A proxy or same-subnet host is required |
| **OQ-4** | `CL-NET-008` | Is **client isolation / AP isolation** enabled on the wireless network? | Whether wireless boards can reach a local server at all | School IT | ⚠️ Assume **isolation is ON.** Require a wired path for the pilot |
| **OQ-5** | `CL-NAS-001` | Does a **NAS or shared file server exist at all**, or do teachers keep materials on laptops, pen drives and personal cloud drives? | ⚠️ **Resource search and the entire assistant.** Nothing to index | School IT + Library | Assume **no shared storage.** ⛔ Drop search and the assistant from scope |
| **OQ-6** | `CL-NAS-009` | What **permission model** is in use on the shares — per-user, per-group, or inherited folder by folder? | ⚠️ **The permission model.** Without per-user permissions, search cannot be safe | School IT + Library | Assume **folder-inherited only.** ⛔ No per-teacher filtering is possible |
| **OQ-7** | `CL-NAS-011` | Is **confidential staff material** — appraisal, salary, medical, disciplinary — stored on the same shares as teaching resources? | ⚖️ **Whether indexing is permissible at all** | Principal | ⛔ Assume **yes, and exclude everything unapproved.** Index nothing by default |
| **OQ-8** | `CL-AUTH-001` | Does the school maintain a **staff identity directory** — one list of teachers, each with a stable identifier? | ⚖️ **Every authorisation decision and the entire audit trail** | Administration | Assume **no directory.** ⚠️ Consequence: shared accounts and **no attributable audit trail** |
| **OQ-9** | `CL-AUTH-004` | Must every action be **attributable to one named teacher**, or is a shared classroom account acceptable? | Whether per-user accounts are required at all | Principal / Teachers | Assume **individual accounts for everything consequential** |
| **OQ-10** | `CL-WIN-001` | What exact **Windows edition and build** is installed on the classroom display computers? | ⚠️ **Whether the kiosk design is feasible at all** | School IT | Assume **Home or an unconfirmed edition.** ⛔ Browser-first, no Assigned Access |
| **OQ-11** | `CL-WIN-003` | Does the account used on a classroom display have **local administrator rights**? | Whether the display can self-update and self-repair | School IT | Assume **no elevation.** ⛔ Signed configuration data only; no self-install |
| **OQ-12** | `CL-TIME-001` | Which single system or document is the **authoritative source** of the master timetable, and who last edited it? | The **write path**, the integration surface and data ownership | Timetable department | Assume **paper or a spreadsheet held by one person.** Manual entry required |
| **OQ-13** | `CL-TIME-005` | How is a teaching day structured — **how many periods**, of what length, separate morning and afternoon blocks? | The **data model**. A wrong model rejects correct data | Timetable department | Assume **a regular grid of equal periods.** ⚠️ Design must still tolerate irregularity |
| **OQ-14** | `CL-SUB-001` | Who currently **decides a substitution** — class teacher, timetable clerk, Vice Principal, subject head? | The **approval gate**. A design with the wrong approver cannot publish | Administration | Assume **a single named timetable official.** Require one named approver |
| **OQ-15** | `CL-SUB-007` | Is a **photograph of the board treated as authoritative**, or must a person always confirm what was photographed? | ⚖️ **Whether OCR/photograph ingestion is in scope at all** | Administration | ⛔ Assume **a person must confirm.** A photograph is never authoritative |
| **OQ-16** | `CL-AI-001` | Does policy permit **any school file or query to be sent to an external AI service**? | ⚖️ **The assistant**, and whether local inference must be funded | Principal | ⛔ Assume **prohibited.** Local inference only, or the assistant is dropped |
| **OQ-17** | `CL-DASH-012` | What should a display show when it **cannot reach the network** — last known data marked stale, or blank? | The **offline contract**, caching scope and staleness thresholds | Administration + Teachers | Assume **last known data, clearly marked stale with a timestamp.** Never blank |
| **OQ-18** | `CL-OPS-022` | Once the team finishes, **who in the school owns this system** — a person, a department, or nobody? | ⚖️ **Whether the project should proceed at all** | Principal | ⚠️ Assume **nobody.** ⛔ A project with no year-two owner should not be built |

⚠️ **OQ-1, OQ-2 and OQ-18 are gates, not questions.** If any of the three comes
back "no", the correct response is to stop or to redesign — not to proceed with
the answer assumed.

---

## 3. Sequencing — which answers unlock which work

```
WEEK 1  ──► OQ-1  named approver          ← gates EVERYTHING. Ask first.
           ├─► OQ-2  always-on host        ← gates all software
           └─► OQ-18 year-two owner        ← gates whether to build at all

WEEK 2  ──► OQ-3  server reachability     ← gates every display
           ├─► OQ-4  client isolation      ← same afternoon, same person
           ├─► OQ-10 Windows edition       ← gates the kiosk design
           └─► OQ-11 admin rights         ← gates the update model

WEEK 3  ──► OQ-5  does storage exist?     ← gates search AND the assistant
           ├─► OQ-6  permission model      ← gates the permission model
           └─► OQ-7  confidential material ← gates whether to index at all
           (If OQ-5 = NO, OQ-6 and OQ-7 are moot and the search track closes.)

WEEK 3  ──► OQ-8  staff directory         ← gates authentication
           ├─► OQ-9  attributable?         ← gates the account model
           └─► OQ-14 substitution approver ← gates publication

WEEK 4  ──► OQ-12 authoritative timetable ← gates the data model
           ├─► OQ-13 day structure         ← same conversation
           └─► OQ-15 photograph authority ← gates OCR scope

WEEK 4  ──► OQ-16 external AI permitted?  ← gates the assistant, and nothing else
           └─► OQ-17 offline behaviour     ← gates the display contract

ALWAYS   ──► Everything above is answered BY PEOPLE. No parallel research
           task exists. ⛔ Do not begin implementation on a guess.
```

⚠️ **The critical path is OQ-1 → OQ-2 → OQ-3 → OQ-5.** Everything else can be
reordered. If that chain is answered, a coherent Phase 0 is possible. If OQ-5
returns "no shared storage exists", the search and assistant tracks close and
the project becomes a timetable and substitution system — **which is still a
complete and useful product.**

---

## 4. Grouped by respondent

Each respondent is asked only for what they alone can answer, and for it first.

### Principal / Administration — ask first

| ID | One-line | Why them |
|---|---|---|
| `CL-GEN-001` | Who can approve or refuse a system storing school data? | ⚖️ Gates everything |
| `CL-SRV-001` | Is a persistent always-on service permitted on school premises? | ⚖️ The platform's home |
| `CL-OPS-022` | Who owns this system in year two? | ⚠️ Decides whether to build |
| `CL-GEN-004` | Under what policy may identifiable staff availability be stored? | Substitution feature |
| `CL-GEN-006` | Which three measurable outcomes define success? | Acceptance gate |
| `CL-DEC-004` | What would require a return to a simpler design? | Scope control |
| `CL-DEC-005` | What would make you say **stop** rather than reduce? | Kill criteria |

### School IT / Network Administrator — ask first

| ID | One-line | Why them |
|---|---|---|
| `CL-NET-003` | May the server subnet accept connections from the classroom VLAN? | ⚠️ Every display |
| `CL-NET-008` | Is client / AP isolation enabled? | ⚠️ Silent killer |
| `CL-WIN-001` | Exact Windows edition and build on the boards | ⚠️ Kiosk feasibility |
| `CL-WIN-003` | Does the display account have local administrator rights? | ⚠️ Update model |
| `CL-NAS-001` | Does a NAS or shared file server exist **at all**? | ⚠️ Search + assistant |
| `CL-NET-001` | Does the network segment classroom devices from other traffic? | ⚠️ Written acceptance may be needed |

### Timetable / Administration department — ask first

| ID | One-line | Why them |
|---|---|---|
| `CL-TIME-001` | What is the authoritative source, and who last edited it? | ⚠️ Write path |
| `CL-TIME-005` | How many periods, of what length, morning and afternoon blocks? | ⚠️ Data model |
| `CL-SUB-001` | Who decides a substitution? | ⚠️ Approval gate |
| `CL-SUB-009` | How early does the final substitution list exist, and how often does it change after? | Refresh design |
| `CL-DASH-002` | What content may appear on a student-facing display? | Signed-out state |
| `CL-TIME-024` | Is publishing the timetable to classroom screens already approved practice? | ⚖️ Existing permission |

### Teachers — ask first

| ID | One-line | Why them |
|---|---|---|
| `CL-GEN-011` | Would you accept a substitution board that is **occasionally** out of date? | ⚠️ Sets the whole accuracy target |
| `CL-DASH-004` | Ranked by usefulness, what should a screen show **first**? | Screen design |
| `CL-SEARCH-001` | What would you actually type into a resource search box? | Search design |
| `CL-AUTH-012` | Would you accept an extra sign-in step on every classroom login? | Feasibility of identity |
| `CL-UX-020` | How do we stop a child triggering an action mid-lesson? | Safety of interaction |

### Library / Resources — ask first

| ID | One-line | Why them |
|---|---|---|
| `CL-NAS-010` | Are folders structured by teacher, class, subject and year? | Index structure |
| `CL-RES-001` | What kinds of teaching resource does the school actually hold? | Metadata model |
| `CL-RES-006` | Which metadata fields are mandatory before publication? | Publication rules |
| `CL-SEARCH-004` | How large a share of material is scanned PDFs or photographs? | OCR scope |
| `CL-SEARCH-014` | Roughly how many files, and how many gigabytes, are on the NAS? | ⚠️ Index sizing |

### Facilities

| ID | One-line | Why them |
|---|---|---|
| `CL-SRV-002` | Which physical locations are candidates for a server? | ⚠️ Hosting |
| `CL-SRV-003` | Is there a UPS or backup power at any candidate location? | Power resilience |
| `CL-SRV-011` | Who would physically restart the server after an unplanned shutdown? | ⚠️ Recovery |
| `CL-DASH-020` | If a display fails or shows an error, who notices and how quickly? | ⚠️ Monitoring |
| `CL-OPS-005` | After a power cut or reboot, who restarts classroom equipment, and how long does it take? | Recovery time |

### Robot custodian (SHANA)

| ID | One-line | Why them |
|---|---|---|
| `CL-SHANA-001` | Which SHANA interface exists and is permitted — SDK, network API, or manual only? | ⚠️ Integration exists at all |
| `CL-SHANA-015` | How does the robot verify a request came from this school's CloudLight? | ⚖️ Mutual authentication |
| `CL-SHANA-002` | Is it this school's requirement that CloudLight may issue only high-level requests? | ⚖️ Boundary sign-off |
| `CL-SHANA-018` | If a delivery request arrives twice, what should the robot do with the duplicate? | Idempotency contract |

### Internal — ours, no school input needed

| ID | One-line |
|---|---|
| `CL-GIT-001` | One repository or two for CloudLight and SHANA? |
| `CL-GIT-008` | What test coverage is expected before merge, especially timetable logic? |
| `CL-GIT-014` | What is our secrets policy, so they never enter the repository? |
| `CL-AIML-013` | What happens when extraction is not confident — block, flag, or accept anyway? |
| `CL-AI-006` | What happens when a retrieved document contains instructions addressed to the assistant? |
| `CL-OTA-004` | Must a board refuse a well-formed but unsigned release? |

⚠️ **These six are the only questions in this register answerable without the
school** — and all six are already decided in
[`decision_log.md`](decision_log.md). They are listed here so that answering
them again is not mistaken for progress.

---

## 5. Questions research cannot answer

Recorded explicitly so nobody wastes time looking for them.

| Question | Why research cannot answer it |
|---|---|
| Can these specific smartboards reach the server? | Needs a packet test from an actual board on an actual subnet |
| What Windows edition is installed? | Needs a photograph of `winver` on one board |
| Does client isolation block device-to-device traffic? | Needs the AP configuration, read by someone who administers it |
| Does a NAS exist at all? | Needs someone who has been in the server room |
| What permission model do the shares use? | Needs a share listing and an ACL export |
| Is confidential staff material on the same shares? | ⚠️ **Needs the Principal.** It is a policy question disguised as a technical one |
| Who approves a substitution? | ⚠️ **Needs the Principal.** A technically perfect pipeline with the wrong approver cannot publish |
| Is photographing the substitution board permitted? | ⚠️ **Needs written permission.** A convenient technical answer with no permission is a policy violation |
| May any file or query leave the network for an external AI service? | ⚠️ **Needs the Principal in writing.** Not inferable from any technical fact |
| How many files are on the NAS? | Needs a directory listing |
| How much does this school have in the way of budget? | Needs the Principal |
| Will teachers actually use it? | Needs teachers |
| Does the school's insurance or policy cover a long-lived local service? | Needs the insurer and the Principal |
| Who restarts the server when it crashes? | Needs a person to agree to it |
| Is a demonstration permitted in school hours, and where? | Needs the Principal |
| Who owns this in year two? | ⚠️ **The single most important answer in this register.** Needs a decision |

⚠️ **Six of these are not technical questions at all.** They are permission and
governance questions wearing a technical costume, and no measurement will ever
answer them.

---

## 6. What happens if the worst-case answers come back

| Scenario | Consequence | Response |
|---|---|---|
| **No named approver** (`CL-GEN-001`) | ⛔ Nothing may lawfully be stored | Stop. Do not build. Escalate to the Principal. This is not recoverable by design work |
| **No always-on host permitted** (`CL-SRV-001`) | ⚠️ The platform has no home | Ask what a hosted VM would mean — note it breaks local-first and is a **scope change**. Otherwise: document the platform and stop before building |
| **Boards cannot reach the server** (`CL-NET-003`) | Every display is blank | Request the firewall rule. Failing that, place the host in the classroom subnet, or use a proxy. Ask IT, do not work around the firewall |
| **Client isolation is on** (`CL-NET-008`) | Wireless boards cannot reach any local host | Require wired network for pilot rooms. Ask IT whether a separate SSID is possible |
| **No shared storage exists** (`CL-NAS-001`) | ⚠️ **Resource search and the assistant have nothing to index** | ⛔ **Drop both from scope.** Build timetable and substitutions, which are the core product and stand alone. ✅ **A smaller but complete system** |
| **Permission model is folder-only** (`CL-NAS-009`) | No per-teacher filtering is possible | ⛔ Do not index. Search over documents a user cannot selectively access is not safe to ship |
| **Confidential material is co-located** (`CL-NAS-011`) | Indexing becomes a data-protection risk | ⚠️ Obtain an explicit approved folder list before anything is indexed. If none can be given, do not index |
| **No staff identity directory** (`CL-AUTH-001`) | ⚠️ **No attributable audit trail** | State this to the school in writing. Either a minimal directory is created, or shared accounts are accepted with the audit consequence stated. ⛔ Do not imply an audit trail that does not exist |
| **External AI prohibited** (`CL-AI-001`) | ⚠️ The assistant needs local inference, or nothing | Ask whether a local machine can be funded. If not, ⛔ drop the assistant. ✅ **The core product is unaffected** — this is why it is built last |
| **No budget** (`CL-BUD-001`) | No server, no storage, no replacement boards | Run on a machine the school already owns. Reduce to timetable and substitutions. ✅ **Still useful** |
| **No pilot permitted** (`CL-EXPO-013`) | No route to real teacher feedback | ⚠️ Do not build user-facing features against imagined needs. Narrow to what can be demonstrated and evaluated without a pilot |
| **Timetable is paper only** (`CL-TIME-002`) | Every entry is keyed manually | Provide a fast manual entry form from day one. This is not a failure — it is the honest first version |
| **The year-two owner is nobody** (`CL-OPS-022`) | ⚠️ Nobody will maintain it | ⛔ **Do not build.** This is the one worst case that should stop the project rather than shrink it |

⚠️ **Seven of these thirteen worst cases end with a smaller project.** That is
the correct outcome of a feasibility phase. A timetable and substitution display
that works, on a machine the school already owns, is worth more than a full
platform that nothing maintains.

⚠️ **One does not.** *"The year-two owner is nobody"* should end the project.
Everything else is a scope conversation.

---

## 7. Explicitly NOT open questions

Settled by the analysis already in this repository, recorded so they are not
re-litigated.

| Question | Answer | Basis |
|---|---|---|
| Can an access point host the backend? | **No.** It is a Layer-2 bridge with no application runtime, persistent storage we may use, or update path we control | [`network_and_hosting.md`](network_and_hosting.md) §2, §3.6 |
| Can a router host it via port forwarding? | **No.** Forwarding to a host that does not exist is not hosting | `network_and_hosting.md` §2.3 |
| Can a smartboard act as the server? | **No.** Boards sleep, restart mid-lesson, lock, and are switched off by staff | `network_and_hosting.md` §3.5 |
| Can a six-digit code authenticate a person? | **No.** A million possibilities is not an authentication factor. It may authorise one action or act as a second factor | [`authentication_and_authorisation.md`](authentication_and_authorisation.md) §5 |
| Are microservices the right default here? | **No.** One school, a few dozen displays, no operations staff. A modular monolith | [`decision_log.md`](decision_log.md) D-021 |
| Must the AI assistant be browser-first? | **Yes** — a browser client is the only client needing the least privilege on a locked-down board | `authentication_and_authorisation.md` §8 |
| Can CloudLight command the robot? | **Never.** No representation of motor, path, speed or actuator exists anywhere in the design | [`shana_integration.md`](shana_integration.md) §1 |
| Must CloudLight work when SHANA is offline? | **Yes.** The timetable is a wall sign and must behave like one | `shana_integration.md` §7.1, `classroom_dashboard.md` §8 |
| Is external AI assumed permitted? | **No.** It is not permitted until written approval says otherwise | [`ai_assistant.md`](ai_assistant.md) §5 |
| Should AI be tried before deterministic parsing? | **No.** Parsing tiers run in strict order; AI last, and it never invents a missing value | [`timetable_and_substitutions.md`](timetable_and_substitutions.md) §4 |
| Is a photograph of a classroom board authoritative? | **No.** A person must always confirm what was photographed | `timetable_and_substitutions.md` §8 |
| Should the assistant be built first? | **No.** It is built **last**, after permission-correct search works with no model present | `ai_assistant.md` §15 |
| Must the display fail silently when stale? | **No.** It fails **closed**, with a visible staleness marker. Silent staleness is the failure mode | `classroom_dashboard.md` §6 |
| Is school configuration kept in the repository? | **No.** Configuration and school data live outside it | `decision_log.md` D-017 |
| Can a stale screen be replaced by nothing at all? | **No.** Last known data, clearly marked, always beats a blank screen in front of a class | `classroom_dashboard.md` §6 |

⚠️ **Fifteen settled questions, recorded so they are not re-asked.** Several of
them feel obvious only because they have already been thought through. They will
feel non-obvious again in three months unless they are written down here.

---

## 8. How this register shrinks

⚠️ **The questionnaire is meant to shrink.** It exists to collapse from 576
questions to a short list of decisions, and then to zero as each answer is
recorded.

| When | What happens |
|---|---|
| Phase 0 (now) | 576 questions, all `UNANSWERED`. This register names the 18 that block architecture |
| First answers land | Blocking set collapses. Decisions move from `OPEN` to `DECIDED` or `PROVISIONAL` in [`decision_log.md`](decision_log.md) |
| Design begins | Most P1 questions become implementation detail. The P3 backlog stays for later |
| Pilot begins | Questions change character — from "does this exist" to "is this good enough" |

**No risk in [`risk_register.md`](risk_register.md) retires through research.**
This register is the list of what a person must do instead.

---

## 9. Conventions

| Field | Content |
|---|---|
| ID | `CL-*` identifier from the questionnaire. Never invented |
| Blocks | What becomes impossible to design without it |
| Who can answer | Named role, never "the school" |
| Default | The conservative assumption that applies if unanswered. ⚠️ **A default must always be safe, never optimistic** |
| Priority | Taken from the questionnaire. `P0` = blocks the activity entirely; `P1` = architecture blocker; `P2` = pilot requirement; `P3` = later |

⚠️ **A blocking question without a recorded default is a question that will
eventually be answered by accident.** Someone will assume an answer, build on it,
and the assumption will be discovered in front of a class.