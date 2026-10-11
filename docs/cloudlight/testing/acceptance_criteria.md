# CloudLight — Acceptance Criteria

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ NOTHING IN THIS DOCUMENT HAS BEEN DEMONSTRATED
>
> **CloudLight does not exist.** There is no code, no service, no display, no
> host and no test suite. Every criterion below is a **condition that would have
> to hold** for a release to be accepted — not a condition that holds.
>
> **Every "Evidence required" entry names evidence that does not exist yet.**
> ⛔ No cell in this document records a passing result, because none has been
> produced.
>
> Vocabulary follows [`../requirements.md`](../requirements.md) §1.1:
> **`CONFIRMED`** · **`PROPOSED`** · **`PENDING ANSWER`**.

**This document has two parts and they are not interchangeable:**

| Part | What it accepts | Audience |
|---|---|---|
| **Part A** | The **first release** — the correct-data spine described in [`../first_release_scope.md`](../first_release_scope.md) | The school, in a real pilot |
| **Part B** | The **exhibition** — an honest demonstration of a project that has not been built | A visitor, at a display |

> ⚠️ **A demonstration is not a deployment.** Part B accepts *evidence quality*.
> Part A accepts *system behaviour*. A project can satisfy Part B entirely and
> satisfy none of Part A — and that is the normal and expected state today.

---

# Part A — Acceptance criteria for the FIRST RELEASE

## A.1 What acceptance means here

> **Release 0 is complete when every criterion in this part is
> *demonstrated*, not *asserted*, and every exclusion in
> [`../first_release_scope.md`](../first_release_scope.md) §2.2 is still standing.**

⚠️ *"We can roll back"* is a claim about a design. *"The rollback ran at 09:14
and every display reconciled"* is a claim about a system. Only the second kind
of statement is evidence, and **no statement of that kind exists yet.**

### A.1.1 Class definitions used in Part A

| Class | Meaning here | What it takes to promote |
|---|---|---|
| **`CONFIRMED`** | Follows from a constraint already accepted in this repository. True whether or not anyone answers a question | Nothing to state it; it can still be **disproved** by a fact we do not have |
| **`PROPOSED`** | The intended behaviour. Sensible, argued for, **agreed by nobody** | Written agreement from the named party who would be affected |
| **`PENDING ANSWER`** | Cannot be specified until a named party answers. Written as a hole, not a specification | The answer to the named `CL-*` question |

⚠️ **The most important class is the third.** `PENDING ANSWER` criteria are
*holes*, and writing them as specifications would be a guess with normative
grammar ([`../requirements.md`](../requirements.md) §1.2).

---

## A.2 Correctness invariants — the release has not shipped without these

> ⚠️ **A2.1–A2.9 are not performance targets.** A release that misses any of them
> has not shipped, **regardless of how well it performs**. A fast system showing
> a wrong name is worse than a slow one showing an explicit error.

| # | Criterion | How measured | Class | Evidence required |
|---|---|---|---|---|
| **A2.1** | **An unapproved substitution is never displayed as a teacher** | Attempt to publish an unapproved record. The display must show an explicit pending state. ⛔ The proposed name appears **nowhere** — not in text, tooltip, accessible name, `title`, or DOM | `CONFIRMED` | Automated test `CT-019`; captured render of the pending state with the name string absent from the full DOM dump |
| **A2.2** | **No teacher is ever invented** | Remove a teacher from the source. The display must show "teacher to be confirmed". ⛔ No fallback to the scheduled teacher, no department default, no previous week's value, no initials, no empty field | `CONFIRMED` | Automated test `CT-024`; three distinct render captures showing the explicit state |
| **A2.3** | **A quarantined record never publishes** | Feed a structurally invalid file. It must fail deterministic validation, be quarantined, retain the rejected document, and produce an operator-visible rejection naming the rule that rejected it | `CONFIRMED` | Automated tests `CT-002`, `CT-003`; the rejection record and the quarantined artifact |
| **A2.4** | **A bad publication is rolled back across all displays** | Publish an error, then withdraw it. Every display that received it must reconcile to the prior version, and the withdrawal must appear in the audit trail | `CONFIRMED` | Automated test `CT-037`; ⚠️ **a timestamped rollback rehearsal log** — *"we can roll back"* is not evidence |
| **A2.5** | **Stale data is always marked** | Stop the host. Content may remain, but it must carry a visible staleness marker **and** its age. ⛔ Never a frozen clock, never an unmarked timer | `CONFIRMED` | Automated tests `CT-041`, `CT-042`; a capture of each of the three offline tiers |
| **A2.6** | **A blank screen never appears where data is absent** | Start with an empty snapshot. Every terminal state must render a distinct, non-empty, diagnosable state. ⛔ Never a black panel, never an unbounded spinner, never a browser error page | `CONFIRMED` | Automated test `CT-043`, run across every state in the catalogue (`CT-054`) |
| **A2.7** | **A code alone never authenticates a person** | Present a six-digit code only. ⛔ It must not authenticate a named teacher. It may authorise one action, or act as a second factor — never either alone. The interface wording must not read as a signed-in teacher | `CONFIRMED` | Automated tests `CT-056`, `CT-057`; a wording review of every screen reachable after code entry |
| **A2.8** | **Every authentication and publication event is auditable** | Attempt log-in (success and failure), publish, correct, withdraw, roll back. Each must be attributable to one named actor with a timestamp, append-only | `CONFIRMED` | Automated test `CT-035`; ⛔ **if no staff identity directory exists, the absence must be stated to the school in writing — never implied as a trail that exists** |
| **A2.9** | **The administration surface is unreachable from a classroom display** | No credential, endpoint, route or session originating from a classroom segment reaches an administrative function | `CONFIRMED` | Automated test `CT-060`; a network-segmentation record from school IT |

---

## A.3 Publication and approval

| # | Criterion | How measured | Class | Evidence required |
|---|---|---|---|---|
| **A3.1** | **Every published record is attributable to a named approver** | Inspect a published record. The approver, the source document and the extracted values must all be retained | `PROPOSED` | Automated test `CT-025`; ⛔ blocked by `CL-SUB-020`, *who approves a substitution* |
| **A3.2** | **Separation between proposing and approving** | Determine whether one person may both propose and approve. Configure and test per the school's answer | `PENDING ANSWER` | ⚠️ **Cannot be specified.** `CL-SUB-002`, `CL-ADMIN-05`. The mechanism can be built; the policy is the school's |
| **A3.3** | **A conflict is surfaced, never silently resolved** | Submit two instructions of equal authority that conflict. The system must escalate to a human. ⛔ It must not pick one, and must not resolve by timestamp | `CONFIRMED` | Automated tests `CT-031`, `CT-034`; a conflict record showing both inputs retained |
| **A3.4** | **The source document is preserved alongside every derived record** | For any published record, the originating source document and location are retrievable | `PROPOSED` | Automated test `CT-004` |
| **A3.5** | **CloudLight is read-only to the school's authoritative timetable** | Attempt to write back. The attempt must be refused | `CONFIRMED` | Automated test `CT-008`; ⚠️ **`CL-TIME-001` must be answered before writing back is even a question** |

---

## A.4 Timing and figures — ⚠️ EVERY NUMBER IS A PROPOSAL

> ### ⚠️ **No school has stated a single one of these figures.**
>
> No refresh rate, latency budget, session timeout, staleness threshold, device
> count, restore time or rollback window has been stated by anyone at this
> school ([`../requirements.md`](../requirements.md) §1.3). ⛔ **The figures in
> this section are ours, not the school's.** They are written as ranges rather
> than precise numbers precisely because a precise number would misrepresent the
> confidence we do not have — and they are marked `PROPOSED` or
> `PENDING ANSWER` accordingly.
>
> ⚠️ **None of them may be quoted to the school as an agreed target.**

| # | Criterion | How measured | Class | Evidence required |
|---|---|---|---|---|
| **A4.1** | **The correct lesson appears within an agreed window of period start** | Compare display against a reference clock at a period boundary | `PROPOSED` | ⚠️ **No number is proposed here.** `CL-REACT-011`, `CL-SUB-009` — the school has not stated a tolerance. The window must be written down by the school before this criterion is measurable |
| **A4.2** | **The countdown is accurate against a reference clock** | Compare displayed remaining time to a reference at intervals. Drift must be **detected**; on drift the countdown is suppressed or visibly marked unreliable | `PROPOSED` | Automated test `CT-050`, `CT-051`. ⚠️ **No drift tolerance stated** — the school sets it |
| **A4.3** | **A rollback completes within an acceptable window** | Measure propagation to every display | `PENDING ANSWER` | ⚠️ **The acceptable window is UNKNOWN and is a school decision.** `CL-SUB-023`, `CL-OTA-007`. ⛔ **No number is proposed.** See [`test_strategy.md`](test_strategy.md) §2.9 |
| **A4.4** | **Data is marked stale beyond a school-set threshold** | Age of displayed data compared against the configured threshold | `PENDING ANSWER` | ⚠️ `CL-SEC-018` — *how old may data be before it is misleading rather than late?* Deliberately left blank in the questionnaire |
| **A4.5** | **Sessions time out on both idle and absolute controls** | Configure both; verify they are independent | `PROPOSED` | ⚠️ **No durations proposed.** `CL-AUTH-016`. Automated test `CT-061` proves independence, not values |

⚠️ **A4.3 and A4.4 have no proposed numbers at all**, and that is deliberate. The
staleness threshold has a **safety cost** — a stale substitution displayed for
40 minutes could send a class to an empty room — so defaulting it would be
choosing a safety parameter on the school's behalf.

---

## A.5 Data protection and the no-identification invariant

| # | Criterion | How measured | Class | Evidence required |
|---|---|---|---|---|
| **A5.1** | **No student-identifying information reaches any display** | Inspect every display payload for name, mark, grade, attendance or identifier | `CONFIRMED` | Automated test `CT-053`; a payload-level assertion across all display endpoints |
| **A5.2** | **No student-identifying information enters substitution records** | Inspect the substitution data model | `CONFIRMED` | Schema constraint plus a test (`CT-053` shared) |
| **A5.3** | **Storing identifiable staff availability is permitted** | ⛔ **Not a technical criterion.** A legal precondition | `PENDING ANSWER` | ⚠️ **`CL-GEN-004` must be answered by the Principal.** ⛔ Until it is, the substitution dashboard cannot ship at all |
| **A5.4** | **Publishing the timetable to classroom screens is approved practice** | ⛔ **Not a technical criterion.** An administrative precondition | `PENDING ANSWER` | ⚠️ **`CL-TIME-024`, `CL-DASH-002`.** ⛔ Without it, the display half cannot proceed as designed |

---

## A.6 Operational and recoverability

| # | Criterion | How measured | Class | Evidence required |
|---|---|---|---|---|
| **A6.1** | **One person can operate the whole thing** | Observe the single IT generalist running a normal day of operations without help | `PROPOSED` | ⚠️ **A supervised human check.** `P6` in [`../first_release_scope.md`](../first_release_scope.md) §3. Cannot be automated |
| **A6.2** | **A documented manual fallback exists for each administrative workflow** | For each workflow, the paper equivalent is written down and has been used at least once | `PROPOSED` | The written fallback, plus a record of it being exercised |
| **A6.3** | **Backups exist and have been restored from** | Restore the backup into a clean environment | `PROPOSED` | ⚠️ **A backup that has never been restored is not a backup.** Test `CT-008` context; a restore rehearsal record |
| **A6.4** | **The display boots unattended to the dashboard** | Power-cycle a real smartboard and observe | `PENDING ANSWER` | ⚠️ **Requires the school to permit the test on their hardware.** `CL-WIN-001` is `P0`, `UNANSWERED`. ⛔ The display design may be impossible on their Windows edition |
| **A6.5** | **The whiteboard still works after lockdown** | Exercise every smartboard function after configuring the kiosk | `PENDING ANSWER` | ⚠️ [`../windows_startup.md`](../windows_startup.md) §8 calls this **the risk most likely to end the project** — *politically, not technically*. Requires school IT and the board vendor |

---

## A.7 What acceptance is contingent on

> ⛔ **None of Part A can be evaluated until these gates are answered.** They are
> reproduced from [`../first_release_scope.md`](../first_release_scope.md) §7
> because an acceptance criterion evaluated against unconfirmed premises is not
> an acceptance criterion.

| Gate | `CL-*` question | If unanswered |
|---|---|---|
| **G1** | May a persistent local service run on school premises? `CL-SRV-001`, `CL-GEN-002` | ⛔ **No release.** The entire local-first premise is unconfirmed |
| **G2** | Who holds authority to approve storing staff and timetable data? `CL-GEN-001` | ⛔ **No lawful basis.** Nothing may be stored |
| **G3** | Can classroom devices reach the host? `CL-NET-003`, `CL-NET-008` | ⛔ **Every display blank** — ⚠️ *silently*, if client isolation is on |
| **G4** | Who approves a substitution, and who owns the master timetable? `CL-SUB-001`, `CL-TIME-001` | ⛔ **The core feature has no publication path.** A perfect pipeline with the wrong approver cannot publish |
| **G5** | Is there a machine, a location, power and a person who can restart it? `CL-SRV-002`, `CL-SRV-003`, `CL-SRV-007` | ⚠️ No deployment. The platform stays hypothetical |
| **G6** | Will the school run a pilot at all? `CL-EXPO-013` | ⚠️ No route to real feedback. Narrow to what can be demonstrated without one |
| **G7** | **Who owns this in year two?** `CL-OPS-022` | ⛔ **A project with no year-two owner should not be built.** The one worst case that ends the project rather than shrinking it |

⚠️ **G1–G4 are the gates that end the project.** G1–G3 have no workaround. G4 has
no default that is safe to assume. **G7 is not a technical question at all**, and
it is the one most easily deferred.

---

## A.8 Honest position on Part A

> ⚠️ **Zero criteria in Part A can currently be evaluated.** Not because the
> evidence is weak, but because **there is nothing to produce it about.**
>
> The realistic path is not "meet these criteria". It is:
>
> 1. **G1–G4 answered in writing by the named owners.**
> 2. Build the spine.
> 3. Then, and only then, demonstrate these criteria on real hardware.
>
> ⛔ **Any claim of current compliance with Part A is false.** No test has been
> run; see [`test_inventory.md`](test_inventory.md).

---

---

# Part B — Acceptance criteria for the EXHIBITION

## B.1 What this part accepts

> **Part B does not accept functionality. It accepts the *quality of the
> evidence presented at a demonstration*.**

⚠️ CloudLight has no software. A demonstration therefore shows **proposals**,
not **working features**. Part B exists so that demonstrating a project which
does not exist is still done honestly — and so that an exhibition crowd is not
mistaken for a pilot audience
([`../first_release_scope.md`](../first_release_scope.md) §8.3).

> ### ⚠️ **An exhibition crowd is not a representative test environment.**
> Showing a system to people who have gathered to watch it produces applause,
> not evidence — and **applause reads exactly like success.**
> `CL-EXPO-003` — *should the demonstration be the acceptance event, or a
> showcase with acceptance reserved for a pilot?* — is `P0` and **UNANSWERED**.

---

## B.2 Evidence-quality criteria

| # | Criterion | How measured | Class | Evidence required |
|---|---|---|---|---|
| **B2.1** | **Every demo label distinguishes *proposed* from *built*** | Walk the demonstration and read every label, slide and caption. Any element implying a working feature that does not exist is a failure | `CONFIRMED` | A transcript of the demonstration narration checked against the inventory of 0 written tests |
| **B2.2** | **Mock data is visibly labelled as mock** | Every displayed name, class and lesson must be visibly synthetic. A real teacher's name must never appear in mock data | `CONFIRMED` | Each mock record carries a visible provenance marker; ⚠️ note that `FR-DASH-10` already prohibits student data and using a **real staff name** would breach the same spirit |
| **B2.3** | **No claim of school approval** | No artefact may state or imply that the school has approved, adopted, agreed or deployed anything | `CONFIRMED` | ⚠️ **576 of 576 questionnaire questions are `UNANSWERED`.** Every one of those may be quoted as unanswered |
| **B2.4** | **Every limitation is shown, not hidden** | The demonstration must include at least: no shared storage confirmed, no identity directory, no approved host, no pilot agreed, no year-two owner | `CONFIRMED` | The five gating failures `G1`–`G5` presented, with their consequences stated |
| **B2.5** | **The demonstration is not presented as a deployment** | No artefact may use deployment language — *installed*, *running in*, *live at* — for anything that is not running | `CONFIRMED` | ⚠️ [`../first_release_scope.md`](../first_release_scope.md) §1.2: a demonstration with mock data is a **separate, labelled activity** (`CL-EXPO-002`) |
| **B2.6** | **The number of written and executed tests is stated accurately** | The demonstration states plainly that **zero tests have been written and zero have been executed** | `CONFIRMED` | ⚠️ **Any other figure is false.** There is no coverage number |
| **B2.7** | **Proposals are marked as proposals** | Any numeric target shown is labelled as a proposal requiring school confirmation | `CONFIRMED` | ⚠️ **No number in [`../requirements.md`](../requirements.md) §3 has been agreed by anyone.** Showing one unmarked misrepresents the confidence we do not have |
| **B2.8** | **Refused and excluded capabilities are named** | The demonstration names what the product will **not** do, with reasons | `CONFIRMED` | The twelve exclusions in [`../first_release_scope.md`](../first_release_scope.md) §2.2, each with the question that would unblock it |
| **B2.9** | **Process failures are disclosed** | Known failures of the documentation and research process are available to anyone who asks | `CONFIRMED` | [`known_failures.md`](known_failures.md), including uncorrected items |
| **B2.10** | **Questions are shown as questions** | Any requirement blocked on a `CL-*` answer is presented as a question, never as a resolved specification | `CONFIRMED` | ⚠️ [`../requirements.md`](../requirements.md) §1.2 — a specification written before the answers is a guess with normative grammar |

---

## B.3 The distinction, stated plainly

> ⚠️ **Exhibition acceptance is not pilot acceptance, and satisfying Part B does
> not advance Part A by one line.**

| | Part A — first release | Part B — exhibition |
|---|---|---|
| **What is accepted** | System behaviour | Evidence quality |
| **Where the evidence comes from** | A real school, real teachers, real classrooms | A demonstration with mock data |
| **Audience** | The school | A visitor |
| **Status today** | ⛔ **0 of 9 invariants demonstrated** | 🟡 **Satisfiable now**, because it judges honesty rather than function |
| **What passing means** | The system works | ⛔ **Nothing works yet, and the demonstration says so** |

⚠️ **A demonstration must never be presented as proof of a working
deployment.** The two differ in kind: one is evidence produced by a running
system under real conditions, the other is a presentation about a design.

⚠️ [`../first_release_scope.md`](../first_release_scope.md) §8.3 records the
specific failure mode: *a pilot is mistaken for the acceptance event*, producing
**confident conclusions drawn from an unrepresentative audience, and a go/no-go
decision made on them.**

---

## B.4 Part B is achievable today — and that is the point

> 🟡 **Part B can be satisfied now. Part A cannot.**
>
> This asymmetry is not a defect in the criteria. It is the honest position:
> the project has done real work on **design, risk and scope**, and has produced
> **no software**. A demonstration may show that work. It may not hide its
> absence.
>
> ⚠️ **If a demonstration cannot state all of the above honestly, it should not
> be given.**

---

**Related documents:** [`test_strategy.md`](test_strategy.md) ·
[`test_inventory.md`](test_inventory.md) ·
[`known_failures.md`](known_failures.md) ·
[`../first_release_scope.md`](../first_release_scope.md) ·
[`../requirements.md`](../requirements.md) ·
[`../feasibility_report.md`](../feasibility_report.md) ·
[`../windows_startup.md`](../windows_startup.md) ·
[`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) ·
[`../decision_log.md`](../decision_log.md) ·
[`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md)