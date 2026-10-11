# CloudLight — Recommended First Release Scope

**Phase 0 · Documentation only · 2026-10-11**

> **⚠️ NOTHING IN THIS DOCUMENT DESCRIBES A BUILT SYSTEM.**
> This is a *scope proposal*. No host exists, no smartboard has been configured,
> no timetable has been seen, no school has approved anything. Every `CL-*` ID
> below is `UNANSWERED` in
> [`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md).
>
> This document proposes what a **Release 0** would contain **if** the gates in
> §7 are answered favourably. It is not a commitment, a plan, or a schedule.

⚠️ The scope conclusion it argues for is the same one reached in
[`feasibility_report.md`](feasibility_report.md) §21 and
[`requirements.md`](requirements.md) §6. It is restated here as a *release
boundary* so that a reader can hold the boundary against it.

---

## 1. The principle

> **The first release must be useful on its own, must depend on no unanswered
> blocking question, and must establish the correct-data path that everything
> later depends on.**

Three tests follow from that, and every entry in §2 is decided by them.

| Test | Meaning | What it rules out |
|---|---|---|
| **Useful alone** | A school gets something it would keep using even if nothing after Release 0 ever ships | Anything that only makes sense next to a later feature |
| **No blocked dependency** | Every precondition is confirmed to exist, or does not need to exist | Anything gated on shared storage, an identity directory, AI policy, or a robot interface |
| **Establishes the spine** | Later releases are added to it; none of them require replacing it | Anything that would have to be thrown away in Release 1 |

⚠️ **The third test is the one usually skipped.** The valuable output of a first
release is not the screen — it is the **correct-data path**: ingest →
deterministic validation → named human approval → versioned publication →
read-only display, with rollback. Every later capability attaches to that path.
Building it first and deferring everything else is not a reduced ambition; it is
the only ordering in which later work is cheap.

### 1.1 The aspiration is not achievable in a first phase

> **The original brief describes a system with an AI assistant, semantic search
> over school storage, OCR of photographed boards, resource browsing, robot task
> submission, mobile clients and self-updating agents. Those are not a first
> release. They are the product, and the product is not reachable from here.**

**This is a normal outcome, not a failure.** A feasibility phase exists to find
out which capabilities have preconditions that exist. Four of the deferred items
are deferred **not because they are hard, but because the thing they depend on has
not been confirmed to exist at all** — shared storage, an identity directory, an
always-on permitted host, and AI policy
([`overview.md`](overview.md) §1, [`feasibility_report.md`](feasibility_report.md)
§22).

⚠️ **A first release that pretended to contain the assistant would be worse than
a smaller one**, because it would require inventing the retrieval it depends on,
the permissions it must respect, and the approval it must respect them under —
none of which are confirmed. The honest first release is narrow, complete, and
ends.

### 1.2 What Release 0 is not

| It is not | Because |
|---|---|
| A smaller version of the full vision | It is a **spine**, not a subset. Later features attach to it |
| A demo | Nothing in it may be faked. A demonstration with mock data is a separate, labelled activity (`CL-EXPO-002`) |
| A pilot | A pilot is an acceptance event with a real school (`CL-EXPO-013`); Release 0 is what a pilot would be built from |
| Something that depends on Release 1 | Every item in §2 stands alone or not at all |

---

## 2. Release 0 — the definition of "in scope"

### 2.1 In scope

| Capability | In? | Why |
|---|---|---|
| **Classroom dashboard** — current lesson, subject, times, countdown | ✅ **IN** | The reason the product exists. Needs nothing unconfirmed. See [`classroom_dashboard.md`](classroom_dashboard.md) |
| **Scheduled teacher** shown for the current lesson | ✅ **IN** | The baseline the school already publishes by hand. No new authority required |
| **Confirmed substitute** shown, and **only** when approved | ✅ **IN** | The whole value of the product. ⛔ An unapproved substitution never appears as a teacher — [`decision_log.md`](decision_log.md) D-008 |
| **Visible last-synchronised timestamp** | ✅ **IN** | The difference between information and decoration. A screen frozen since 08:00 looks exactly like a correct one ([`classroom_dashboard.md`](classroom_dashboard.md) §6) |
| **Deterministic timetable ingestion** from spreadsheet or CSV | ✅ **IN** | `.xlsx` and `.csv` have stable machine-readable structure. A library reads them exactly; a model must not ([`feasibility_report.md`](feasibility_report.md) §9) |
| **Substitution pipeline with a named human approver** | ✅ **IN** | The approval gate is the correctness property, not an accessory to it |
| **Versioned publication with rollback** | ✅ **IN** | A wrong name on a wall must be withdrawable. Append-only versions, pointer moved, never edited in place ([`timetable_and_substitutions.md`](timetable_and_substitutions.md) §7) |
| **Individual teacher accounts** | ✅ **IN** | A shared code identifies nobody. Anything consequential needs attribution ([`overview.md`](overview.md) §4.2) |
| **A separated administration surface** | ✅ **IN** | ⛔ The administration surface must never be reachable from a classroom display |
| **Fail-closed offline behaviour with a staleness marker** | ✅ **IN** | Schools have power cuts. The boot race is real and is not fixed by a browser flag ([`windows_startup.md`](windows_startup.md) §4) |
| **Backups and health checks** | ✅ **IN** | No interface, no demo, and the requirements most likely to be skipped under time pressure ([`requirements.md`](requirements.md) §7.1) |
| **Audit logging** | ✅ **IN** | *"Who published this, and when?"* must be answerable for a year, not remembered |

### 2.2 Out of scope — and the question that would unblock each

⚠️ **Every exclusion names the question that would unblock it**, so that none of
them becomes a silent omission. An exclusion with no unblocking question is a
decision someone forgot to make.

| Capability | Why out | What would unblock it |
|---|---|---|
| **AI assistant** | The largest unverified dependency in the project. Its entire value depends on permission-aware retrieval, which is itself deferred. Largest data-protection surface in the product | `CL-AI-001`, `CL-DEC-010`, `CL-AI-009`, `CL-AI-023`, `CL-BUD-006`, `CL-SEC-011` |
| **Semantic / vector search** | Requires a model, hardware and a corpus whose **size is unknown**. Filename and metadata search cover most real teacher queries at a fraction of the cost | `CL-SEARCH-014`, `CL-SEARCH-001`, `CL-AI-009`, `CL-SRV-015` |
| **OCR of photographed boards** | Vendor accuracy figures do not transfer to this school's handwriting, lighting and board types. Permission to photograph is also unconfirmed | `CL-AIML-009`, `CL-SUB-006`, `CL-SUB-007`, `CL-AIML-010`, `CL-UX-016` |
| **AI-assisted extraction** | The highest-risk, lowest-value component in the pipeline. It is gated on an approval workflow that does not exist yet, and a model must never fill a missing value | `CL-AIML-013`, `CL-SUB-007`, `CL-AIML-009`, `CL-SUB-001` |
| **Resource browsing and search** | ⛔ **Shared storage is not confirmed to exist.** A school may have no NAS at all; teachers may use laptops, pen drives and personal cloud drives | `CL-NAS-001`, `CL-NAS-002`, `CL-NAS-004` |
| **SHANA task submission** | Robot-side authentication is an unresolved `P0` and the interface is a proposal checked against no vendor SDK. Adding it first would put an unconfirmed dependency under the core product | `CL-SHANA-001`, `CL-SHANA-015`, `CL-SHANA-008` |
| **OTA self-update** | There is nothing to update yet. It also requires release-signing infrastructure, a key custody policy and staged rollout — none exists | `CL-OTA-005`, `CL-OTA-010`, `CL-OTA-012`, `CL-OTA-009` |
| **Packaged desktop clients** | A browser kiosk takes updates from the server; an installed client needs an installer and an update path on every classroom machine | `CL-DEC-009` — only if a browser is *demonstrably* unacceptable |
| **Mobile apps** | Multiplies client maintenance by an unknown device count for no Release 0 classroom benefit | `CL-REACT-017`, `CL-AUTH-018`, `CL-BUD-004` |
| **Multi-school support** | One school, one academic year, one set of answers. Multi-tenancy before a single tenant is proven is a classic inversion | `CL-BUD-018`, and a second school that asks for it |
| **Real-time push** | Periodic refresh is adequate until a latency requirement is **measured**. `CL-REACT-011` asks the school for the number we lack | `CL-REACT-011`, `CL-SUB-009` |
| **Writing back to the school's authoritative timetable** | Two-way sync creates an ownership problem with no answer. CloudLight is a consumer and renderer until the school says otherwise | `CL-TIME-001`, `CL-DATA-019` |

⚠️ **Twelve exclusions, and not one of them is excluded because it is
technically hard.** Most are excluded because a precondition has not been
confirmed to exist. That is the correct outcome of a feasibility phase, and it is
recorded rather than quietly absorbed.

---

## 3. What the first release proves

> **Release 0 is not a small product. It is the set of experiments that decides
> whether the large product is possible.**

Each claim below is architectural. If Release 0 fails, it does not shrink the
product — it invalidates a load-bearing assumption that is cheaper to find now
than in Release 3.

| # | Claim proven | Why it is the thing to prove first |
|---|---|---|
| **P1** | **A display can reach a host.** | ⚠️ The single most important unknown in the project. Client isolation fails *silently* — boards associate, show full signal, and simply never load (`CL-NET-008`). No amount of software work fixes a network that was never opened |
| **P2** | **Data can be approved and published safely.** | The publication path is the product. If a wrong name reaches a wall and cannot be withdrawn, nothing later matters |
| **P3** | **The dashboard stays correct and honest when degraded.** | A wall display is trusted by everyone in the room and verified by nobody. Wrong-and-unmarked is worse than blank-and-explicit |
| **P4** | **Authentication works on a shared, unattended device.** | Classrooms are unattended and frequently shared. ⛔ A persistent teacher session on an open panel means whoever is in the room inherits teacher authority (`CL-AUTH-020`) |
| **P5** | **A human-approved pipeline prevents bad data reaching a screen.** | The single property the entire design exists to guarantee. If it does not hold under a deliberately corrupted input, the architecture is wrong, not the parser |
| **P6** | **One person can operate the whole thing.** | The school has one IT generalist with other duties. An architecture needing an operations team is an architecture this school cannot run ([`architecture.md`](architecture.md) §2.2) |

⚠️ **P1, P2 and P5 are the ones that end the project rather than change its
shape.** P3, P4 and P6 determine whether the release is pleasant, survivable and
sustainable. In that order.

---

## 4. Minimum hardware and configuration

> ### ⚠️ **PROVISIONAL — every figure below requires confirmation.**
>
> **No hardware has been seen, requested or measured.** This is sizing *logic*, not
> a specification. It deliberately does **not** state CPU or RAM figures as
> requirements, because presenting a precise number we cannot justify would
> misrepresent the confidence we do not have
> ([`requirements.md`](requirements.md) §1.3).

### 4.1 The smallest configuration that could carry Release 0

| Element | Requirement | Status |
|---|---|---|
| **One always-on host** | An authorised school machine permitted to run a persistent service for the whole working day | ⛔ **Not confirmed to be permitted** — `CL-SRV-001`, `CL-GEN-002` |
| **Power at that host** | Mains, plus backup power if outages occur during the school day | ❓ UNKNOWN — `CL-SRV-003`, `CL-SRV-012`, `CL-NET-019` |
| **One network port** at the host, on a segment classroom devices can reach | A stable address, ideally a managed DNS name | ❓ UNKNOWN — `CL-NET-003`, `CL-NET-006`, `CL-NET-012` |
| **One or two classroom smartboards** | Rooms permitted to have a display | ❓ UNKNOWN — `CL-DASH-001`, `CL-WIN-001` |
| **A verified reachability path** | Board → resolves a name → reaches the host → passes the firewall → trusts the certificate → receives a response | ❓ **All five links UNKNOWN.** ⚠️ A working ping proves none of the rest ([`network_and_hosting.md`](network_and_hosting.md) §5.2) |
| **A backup copy** | A second copy, not a second folder on the same disk | ❓ UNKNOWN — `CL-DATA-008`, `CL-SRV-016` |
| **Someone who can restart it** | During school hours | ❓ UNKNOWN — `CL-SRV-011`, `CL-OPS-006` |

### 4.2 The sizing logic, stated so it can be argued with

| Question the hardware must answer | What drives the answer | Why it is not yet a number |
|---|---|---|
| How much compute does serving N displays need? | N is **unknown** | `CL-WIN-020` has not been asked. A handful of polling displays is not a compute problem |
| How much storage? | The database is small; a derived index is the larger consumer | Corpus size is **unknown** — `CL-SEARCH-014`, `CL-SRV-013` |
| Does anything need a GPU? | Only local model inference does, and it is out of scope | `CL-AI-009` is UNKNOWN. ⚠️ **Do not size the host around inference** — size it for the core product ([`network_and_hosting.md`](network_and_hosting.md) §4.2) |
| SSD or spinning disk? | Rebuilding an index on a spinning disk is a *visible stall in a classroom* | Provenance for a filesystem choice, not a measured requirement |
| How many network interfaces? | One works. Two allow a separate management path | Whether SHANA ever needs one is unresolved — `CL-SHANA-001` |

⚠️ **The honest statement is directional:** Release 0 is a small workload — one
service, one relational database, a few polling displays — so the host should be
an ordinary office machine the school already owns, not a purchase. Confirm
against [`network_and_hosting.md`](network_and_hosting.md) §4 and
[`requirements.md`](requirements.md) §3 once `CL-SRV-002` and `CL-SRV-007` are
answered.

### 4.3 Client form

| Choice | Verdict | Basis |
|---|---|---|
| **Managed kiosk browser** | ✅ Proposed | Takes updates from the server; no installer on every classroom machine ([`windows_startup.md`](windows_startup.md) §2(b)) |
| **Packaged desktop client** | ❌ Not proposed | Needs an installer and an update path on every board — [`decision_log.md`](decision_log.md) X-04 |

⚠️ **Both collapse if the boards run an edition without Assigned Access or Shell
Launcher.** `CL-WIN-001` is `P0` and UNANSWERED. The fallback is a browser opened
by hand daily, which is not an acceptable experience and would change this
release's shape.

---

## 5. Acceptance criteria

> **Class** uses the vocabulary already established in
> [`requirements.md`](requirements.md) §1.1:
> **`CONFIRMED`** — follows from a constraint already decided in this repository,
> and is true whether or not anyone answers a question.
> **`PROPOSED`** — the intended behaviour, agreed by nobody.

| # | Criterion | How it is measured | Class |
|---|---|---|---|
| 1 | **No unapproved substitution is ever displayed as a teacher** | Attempt to publish an unapproved record. Display must show an explicit "pending confirmation" state. ⛔ The proposed name appears nowhere on screen in any position or treatment | `CONFIRMED` |
| 2 | **No invented teacher name is ever displayed** | Remove a teacher from the source. Display must show "Teacher to be confirmed". ⛔ No silent fallback to the scheduled teacher, no department default, no previous week's value, no empty field | `CONFIRMED` |
| 3 | **The last-synced timestamp is always visible** | Every rendered state carries it, including stale, no-data and pending | `CONFIRMED` |
| 4 | **A deliberately corrupted timetable is rejected and quarantined, never published** | Feed a structurally invalid file. It must fail deterministic validation, be quarantined, and produce an operator-visible rejection naming the reason | `CONFIRMED` |
| 5 | **A bad publication is rolled back across all displays** | Publish an error, then withdraw it. Every display that received it must reconcile to the prior version, and the withdrawal must appear in the audit trail. ⚠️ Until rehearsed, this is a claim about a design, not a system | `CONFIRMED` |
| 6 | **Displays show a marked stale state when the server is stopped** | Stop the host. Content may remain, but it must carry a visible, unambiguous staleness marker and its age. ⛔ Never a frozen clock, never an unmarked timer | `CONFIRMED` |
| 7 | **Displays show an explicit no-data state when no snapshot exists** | Start with an empty snapshot. A bare black panel must never be readable as "no lesson" | `CONFIRMED` |
| 8 | **A teacher cannot authenticate with a shared code alone** | Present a six-digit code only. ⛔ It must not authenticate a named teacher. It may authorise one action or act as a second factor — never either on its own | `CONFIRMED` |
| 9 | **Every authentication and publication event is in the audit log** | Attempt log-in (success and failure), publish, correct, withdraw, roll back. Each must be attributable to a named actor with a timestamp, append-only | `CONFIRMED` |
| 10 | **The correct lesson appears within N seconds of period start** | Compare display against the reference clock at a period boundary. 🧪 **`N` is a proposal requiring confirmation** — no school has stated a tolerance. Stated as a range rather than a number because a precise figure would misrepresent the confidence we do not have | `PROPOSED` |
| 11 | **The countdown is accurate against a reference clock** | Compare displayed remaining time to a reference at intervals. Clock drift must be **detected**; on drift the countdown is suppressed or visibly marked unreliable | `PROPOSED` |
| 12 | **A rollback completes within an acceptable window** | Measure propagation to every display. ⚠️ ❓ The acceptable window is **UNKNOWN** and is a school decision, not ours — `CL-SUB-023`, `CL-OTA-007`. No number is proposed here | `PENDING ANSWER` |
| 13 | **Every published record is attributable to a named approver** | Inspect the record. The approver, the source document and the extracted values must be retained | `PROPOSED` |

⚠️ **Criteria 1–9 are not performance targets.** They are correctness
invariants. A release that misses any of them has not shipped, regardless of how
well it performs. Criteria 10–13 carry figures nobody has agreed, and are written
as questions rather than specifications until they are answered.

⚠️ **No criterion here has been tested.** Every one is a proposal for a test that
does not yet exist.

---

## 6. Explicit non-goals for this release

Restated so nobody measures Release 0 against the full vision.

| This release is **not**… | Why it might look like it should be |
|---|---|
| **A smaller CloudLight** | It is a spine. Later capabilities attach to it; none of them is a reduced version of it |
| **A demonstration** | Nothing in it is simulated. A demonstration is a separate, explicitly labelled activity (`CL-EXPO-002`) |
| **The pilot's acceptance** | ⚠️ A pilot is an acceptance event with real teachers in real classrooms (`CL-EXPO-013`). Release 0 is what a pilot would be built *from* |
| **A search engine** | There is nothing to search. ⛔ Shared storage is unconfirmed (`CL-NAS-001`) |
| **Intelligent or automated** | Deterministic parsing and a named human decide everything. ⛔ No AI publishes, approves or invents |
| **A teacher-facing application** | The display is read-only and unauthenticated by design ([`decision_log.md`](decision_log.md) D-009) |
| **A multi-school platform** | One school. Multi-tenancy is a later problem, not this one |
| **A robot interface** | ⛔ CloudLight must operate fully when SHANA is offline ([`shana_integration.md`](shana_integration.md) §7.1) |
| **Self-maintaining** | No unattended update path. Every deploy is a logged human action until proven otherwise (`CL-OTA-012`) |
| **A finished product** | It is the first of several releases, or it may be the only one |

---

## 7. What must be answered before it can start

> ⚠️ **No item in §2 may begin implementation until its gating question is
> answered.** Cross-referenced against
> [`open_questions_register.md`](open_questions_register.md) §2.

| # | Gating question | Owner | Consequence if unanswered |
|---|---|---|---|
| 1 | May a persistent local service run on school premises? `CL-SRV-001`, `CL-GEN-002` | Principal / Administration | ⛔ **No release.** The entire local-first premise is unconfirmed |
| 2 | Who holds authority to approve storing staff and timetable data? `CL-GEN-001` | Principal | ⛔ **No lawful basis.** Nothing may be stored |
| 3 | Can classroom devices reach that host — subnet, isolation, firewall? `CL-NET-003`, `CL-NET-008`, `CL-NET-014` | School IT | ⛔ **Every display blank**, and ⚠️ silently if isolation is on |
| 4 | Who approves a substitution, and who owns the master timetable? `CL-SUB-001`, `CL-TIME-001` | Timetable department | ⛔ **The core feature has no publication path.** A perfect pipeline with the wrong approver cannot publish |
| 5 | What Windows edition is on the boards, and may a kiosk account have admin rights? `CL-WIN-001`, `CL-WIN-003` | School IT | ⛔ **The display design may be impossible.** Fallback is a browser opened by hand |
| 6 | Is a staff identity directory available, and may SSO be used? `CL-AUTH-001`, `CL-AUTH-002`, `CL-AUTH-003` | Administration / IT | ⚠️ **No attributable audit trail.** State this to the school in writing; ⛔ never imply a trail that does not exist |
| 7 | Is storing identifiable staff availability permitted? `CL-GEN-004` | Principal | ⛔ The substitution dashboard cannot ship |
| 8 | Is publishing the timetable to classroom screens approved practice? `CL-TIME-024`, `CL-DASH-002` | Administration | ⛔ The display half cannot proceed as designed |
| 9 | What should a display show when it cannot reach the network? `CL-DASH-012` | Administration + Teachers | Assume **last known data, clearly marked stale with a timestamp. Never blank** |
| 10 | Is there a machine, a location, power and a person who can restart it? `CL-SRV-002`, `CL-SRV-003`, `CL-SRV-007`, `CL-SRV-011` | Facilities / IT | ⚠️ No deployment. The platform stays hypothetical |
| 11 | Will the school run a pilot at all? `CL-EXPO-013` | Administration | ⚠️ No route to real feedback. Narrow to what can be demonstrated without one |

⚠️ **Items 1, 2, 3 and 4 are the gates.** Items 1–3 have no workaround at all;
item 4 has no default that is safe to assume. The remaining items have recorded
conservative defaults in [`open_questions_register.md`](open_questions_register.md)
§2, and those defaults apply if nobody answers.

⚠️ **One gate is not a question at all.** `CL-OPS-022` — who owns this in year
two — is the answer that decides whether the project should proceed. ⛔ A project
with no year-two owner should not be built. It is the one worst case that ends
the project rather than shrinking it.

---

## 8. Risks specific to a first release

Risks that exist *because* this is a first release, over and above those already
in [`risk_register.md`](risk_register.md) §2.8.

### 8.1 The pilot fails and becomes permanent

> **A failed pilot, with no failure path agreed in advance, does not get
> discussed. It gets absorbed.** The screen stays on the wall, nobody maintains
> it, and the school quietly returns to paper while the project still appears to
> be running.

| | |
|---|---|
| **Question** | `CL-EXPO-017` — *what happens if the pilot fails, who makes that call?* — `P0`, `NEEDS_SCHOOL_PERM`, **UNANSWERED** |
| **Owner** | Administration, agreed **before** the pilot starts |
| **Mitigation** | Agree pass/fail criteria and the failure path in writing first. Keep the pilot small enough that abandoning it costs little |
| **Register** | R-41 🟠 12 |

### 8.2 Teachers ignore it

| | |
|---|---|
| **Risk** | The system is accurate, available and correct, and teachers keep using paper — because it added a step rather than removing one |
| **Question** | `CL-EXPO-016` — *what would count as pilot success, agreed in advance?* |
| **Related** | `CL-GEN-011` — *would teachers accept a board that is occasionally out of date?* ⚠️ **That answer sets the entire accuracy target**, and it is the number the staleness threshold depends on |
| **Owner** | Teachers, asked directly |
| **Register** | R-42 🟠 8 |

### 8.3 The pilot is mistaken for the acceptance event

> ⚠️ **An exhibition crowd is not a representative test environment.** Showing a
> system to people who have gathered to watch it produces applause, not evidence
> — and applause reads exactly like success.

| | |
|---|---|
| **Question** | `CL-EXPO-003` — *should the demonstration be the acceptance event, or a showcase with acceptance reserved for a pilot?* — `P0`, **UNANSWERED** |
| **Related** | `CL-EXPO-015` — *should pilot rooms include mixed subject areas, or a single friendly early adopter?* A single-room pilot proves feasibility and tells us almost nothing about scale |
| **Consequence** | Confident conclusions drawn from an unrepresentative audience, and a go/no-go decision made on them |

### 8.4 Scope grows faster than capability

| | |
|---|---|
| **Risk** | Each deferred capability is individually reasonable; together they exceed what can be delivered to a school standard. **An unfinished system is worse than a finished smaller one** |
| **Question** | `CL-GEN-021`, `CL-DEC-017` — a signed must-have list and an explicit deferrable list, recorded **before** building |
| **Owner** | Someone with authority to say no |
| **Register** | R-39 🟠 12 |

---

## 9. Exit criteria and the next step

> **Release 0 is complete when every criterion in §5 is demonstrated, not
> asserted, and every exclusion in §2 is still standing.**

| Gate to proceed | What must be true |
|---|---|
| **All of §5 demonstrated** | Each criterion tested on real hardware, with evidence recorded |
| **§5 criteria 1–9 hold** | These are invariants. A miss is not a tuning problem |
| **§7 items 1–4 answered** | In writing, by the named owner, not inferred from silence |
| **Every §2 exclusion still justified** | ⚠️ If a school answer has unblocked one, it is **not** automatically added — it is a scope decision, recorded in [`decision_log.md`](decision_log.md) |
| **Rollback rehearsed, not intended** | ⚠️ *"We can roll back"* is a claim about a design. *"The rollback ran at 09:14 and every display reconciled"* is a claim about a system |
| **A named year-two owner exists** | `CL-OPS-022`. ⛔ Otherwise stop rather than continue |
| **Nothing depends on a later release** | The spine stands alone |

### 9.1 The next step — Release 1

> **Release 1 adds resource features, and only if shared storage is confirmed to
> exist.**

| Condition | Consequence |
|---|---|
| `CL-NAS-001` = **yes** | Release 1 is reachable: authorised resource browsing first, then permission-aware search. Requires `CL-NAS-002` (what it is), `CL-NAS-007` (dedicated read-only service account), `CL-NAS-008` (approved folder list) |
| `CL-NAS-001` = **no** | ⛔ Resource features and the assistant are **dropped, permanently**. ✅ Build what exists instead — more classrooms on the same spine |

⚠️ **"No shared storage" is not a failure.** It produces a timetable and
substitution display that works, on a machine the school already owns. ✅ **A
smaller but complete system is worth more than a larger one that does not exist**
([`open_questions_register.md`](open_questions_register.md) §6).

⚠️ **Search is not a Release 1 feature even when storage exists.** It requires
permission filtering at query time **and** re-verification at open time, over a
permission model that is itself unconfirmed (`CL-NAS-009`, `CL-NAS-011`). Browse
first. Search only after permissions are understood.

### 9.2 What Release 0 is worth even if nothing follows

**A correct, honest, human-approved timetable on one or two classroom displays,
on a machine the school already owns, that a teacher can trust because it never
shows a name nobody approved.**

⚠️ That is a small sentence. It is also the entire difference between a system
that is acted upon and one that is looked past.

---

**Related documents:** [`overview.md`](overview.md) ·
[`feasibility_report.md`](feasibility_report.md) · [`architecture.md`](architecture.md) ·
[`requirements.md`](requirements.md) · [`classroom_dashboard.md`](classroom_dashboard.md) ·
[`timetable_and_substitutions.md`](timetable_and_substitutions.md) ·
[`network_and_hosting.md`](network_and_hosting.md) ·
[`windows_startup.md`](windows_startup.md) ·
[`deployment_options.md`](deployment_options.md) ·
[`authentication_and_authorisation.md`](authentication_and_authorisation.md) ·
[`open_questions_register.md`](open_questions_register.md) ·
[`risk_register.md`](risk_register.md) · [`decision_log.md`](decision_log.md) ·
[`next_actions.md`](next_actions.md) ·
[`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)