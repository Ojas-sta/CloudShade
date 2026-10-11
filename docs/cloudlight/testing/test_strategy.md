# CloudLight — Test Strategy

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ THERE IS NO TEST SUITE IN THIS REPOSITORY, AND NONE HAS EVER BEEN RUN
>
> **CloudLight has no source code.** There is no application, no service, no
> frontend, no parser, no database schema and no configuration. There is no
> `package.json`, no `pyproject.toml`, no `go.mod`, no `Cargo.toml`, no
> `Makefile`, no `Dockerfile`.
>
> **There is no test framework.** No test runner, no assertion library, no
> fixtures, no mocks, no CI pipeline that executes anything.
>
> **There is no coverage figure** — not zero, because zero implies something
> was measured. There is nothing to measure.
>
> ⛔ **Every test discussed in this document is a plan for a test that does not
> exist.** No result recorded anywhere in this repository is real. If you find
> a test result, a coverage percentage or a passing assertion quoted as fact
> anywhere in these documents, it is wrong.

**This document is a strategy, not a report.** It describes how CloudLight
*should* be tested once it exists — and why large parts of it cannot be written
yet.

---

## 1. Current state

### 1.1 What exists

| Question | Answer |
|---|---|
| Production code? | **No.** Verified by search for `.js`, `.ts`, `.tsx`, `.py`, `.go`, `.rs`, `.java`, `.cs` |
| Build or dependency manifest? | **No.** No `package.json`, `tsconfig.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Makefile`, `Dockerfile` |
| Test files? | **No.** None, of any framework, in any language |
| Test runner or CI? | **No.** The repository has an issue template; it has no workflow that runs anything |
| Tests executed? | **Never.** There is nothing to execute |
| Coverage number? | **None exists.** No instrumented code has ever been run |
| Requirements with a test? | **Zero**, against 133 `FR-*` requirements in [`../requirements.md`](../requirements.md) |

The only non-markdown files in the repository are `.editorconfig`,
`.gitignore`, `.github/ISSUE_TEMPLATE/config.yml`, four SHANA-authored YAML
schemas under `data/schemas/`, `.obsidian/` vault configuration, photographs,
and one gitignored local tooling script. **None of it is CloudLight.**

### 1.2 Why a strategy anyway

A feasibility phase that only asks questions produces an unanswered question
list. This document exists so that the *first* tests written are the
safety-relevant ones rather than the easy ones — and so that the blockers are
named before anyone writes a test against an unconfirmed design.

⚠️ **Read §7 before treating this as a work plan.**

### 1.3 Vocabulary

Terms follow [`../requirements.md`](../requirements.md) §1.1 and are used
unchanged across these four documents: **`CONFIRMED`** (follows from a
constraint already accepted here; true whether or not anyone answers a
question), **`PROPOSED`** (the intended behaviour, agreed by nobody), and
**`PENDING ANSWER`** (cannot be specified until a named party answers a `CL-*`
question — written as a hole, not a specification).

---

## 2. Testability risks

> ⚠️ **These are not implementation difficulties. Most are consequences of not
> yet knowing what the school actually has.** They are the reason a large test
> suite cannot be written in advance.

### 2.1 No deterministic fixtures exist

The tests in [`test_inventory.md`](test_inventory.md) need input data. There is
none. Not a sample, not an anonymised extract, not a synthetic file written to
look like the school's.

**Consequence:** every data-shaped test is blocked on receiving one real
document. A parser written and tested against a file the author invented
proves only that the parser handles the author's imagination.

⚠️ This is why §2.2 is the single most important entry in this section.

### 2.2 ⛔ The timetable format is UNKNOWN — no representative fixture can be written

`CL-TIME-002` and `CL-TIME-003` are `P0` and `UNANSWERED`. We do not know
whether this school uses a spreadsheet, a printed PDF, a portal, a commercial
timetable package, or four of these at once.

The period model is worse: `CL-TIME-005` and `CL-TIME-006` are both `P0` and
`UNANSWERED`. A period may be defined by wall-clock time, by bell, by teacher,
by room, or by a rotating weekly matrix. [`../requirements.md`](../requirements.md)
§1.2 is explicit that **a specification written before those answers is a guess
with normative grammar, and considerably harder to notice than a guess in a
comment.** The same applies to its tests.

| What cannot be written yet | Unblocked by |
|---|---|
| A representative timetable fixture | `CL-TIME-002`, `CL-TIME-003` |
| A period-resolution test | `CL-TIME-005`, `CL-TIME-006` |
| A calendar / term-boundary test | `CL-TIME-013`, `CL-TIME-014`, `CL-TIME-015` |
| Any test whose expected output depends on the school's day shape | `CL-TIME-008` |

⚠️ **Writing these tests against a guessed format would produce a green suite
over a fiction.** That is worse than no suite, because it reads as evidence.

### 2.3 Extraction is non-deterministic

OCR and AI-assisted extraction produce different output on different runs, on
different lighting, on different handwriting
([`../feasibility_report.md`](../feasibility_report.md) §10). Two runs over
identical input may disagree.

**Consequence:** any test asserting an exact extracted value will be flaky.
Determinism must come from the *pipeline*, not the model — see §4. Extraction
is **out of scope for Release 0** ([`../first_release_scope.md`](../first_release_scope.md)
§2.2); it is recorded here because a suite written now must not *assume*
determinism anywhere.

### 2.4 Degraded and offline states need fault injection

Three of the highest-value tests — stale marking, no-data, rollback
reconciliation — require the backend to be **stopped, slow or lying**. That
needs a way to make it fail.

There is no test double, no fault-injection harness, and no environment to
break. Building one is itself engineering work that does not yet exist.

⚠️ The offline path is the worst case because of §2.6: the *display side* of it
is also unproven.

### 2.5 ⛔ Permission filtering depends on an integration that does not exist

`FR-SEARCH-02` (filter by entitlement **before** ranking), `FR-SEARCH-03`
(never return something the user cannot open), `FR-RES-04` (re-check at query
time *and* at open time) are `CONFIRMED` invariants. They are also untestable
today, because:

- ⛔ `CL-NAS-001` — **does a shared file store exist at all?** `P0`, `UNANSWERED`
- ⛔ `CL-NAS-002` — what is it? `PENDING ANSWER`
- ⛔ `CL-NAS-007` — is a dedicated read-only service account permitted?
- ⛔ `CL-NAS-011` — is there a permission model to model at all?

**A permission test against a mock directory proves the mock is correct.** The
only meaningful proof requires the school's real store, which does not exist.
This is why the inventory marks every `CT-3xx` search test as blocked rather
than merely unwritten.

### 2.6 ⛔ The display-side offline path is itself unproven

[`../windows_startup.md`](../windows_startup.md) §7 records this as **the
highest-risk technical unknown in that document**: both Edge kiosk lockdown
experiences run an InPrivate session, and whether a service worker can install
a durable offline cache — or whether `localStorage` survives a kiosk restart —
is version-dependent behaviour **on the specific Edge build this school uses,
which we have not tested and cannot test today.**

⚠️ If neither works, the offline design is not implementable as drawn and the
architecture must change. A test suite that assumes durable caching tests a
design that may not survive contact with the board.

### 2.7 The school LAN and client isolation cannot be probed

`CL-NET-003`, `CL-NET-008`, `CL-NET-014` are unanswered. Probing school
infrastructure is **prohibited** by the constraint already accepted in
[`../requirements.md`](../requirements.md) §4 and §1.4, and restated in
[`../network_and_hosting.md`](../network_and_hosting.md). So the tests that
would matter most for deployment — *can a board reach the host?* — cannot be
written, cannot be run, and must not be attempted.

⚠️ This is a deliberate constraint, not an oversight. It is why
[`../first_release_scope.md`](../first_release_scope.md) §3 lists **P1 — a
display can reach a host** as the single most important unknown: client
isolation fails *silently*. Boards associate, show full signal, and never load.

### 2.8 The approver is unnamed

The approval gate is the correctness property of the whole design
([`../decision_log.md`](../decision_log.md) D-008). But `CL-SUB-001` — *who
owns the master timetable* — and `CL-SUB-003` are `P0`, `UNANSWERED`, and
`CL-ADMIN-05` (may one person both propose and approve?) is `PENDING ANSWER`.

**Consequence:** the approval workflow can be tested as a *mechanism* but not
validated against a *policy*. Tests cannot fail for the reason that would
actually matter.

### 2.9 No numbers exist to test against

Per [`../requirements.md`](../requirements.md) §1.3, **every numeric target in
this project is a proposal requiring confirmation.** No school has stated a
refresh rate, latency budget, session timeout, staleness threshold or rollback
window. `CL-SEC-018` — *how old may data be before it is misleading rather than
late?* — is deliberately left blank.

⛔ Any performance test written today would assert a figure we invented. A test
that passes because the threshold is our own guess is not a test of the
system; it is a test of the guess.

---

## 3. Test layers

⚠️ **"Can it be written now?"** is the honest column. Most of this table reads
"no", and that is the finding.

| Layer | What it covers | Can it be written now? | Blocked by |
|---|---|---|---|
| **Unit** | Pure functions: time arithmetic, period resolution, conflict predicates, name normalisation, state-machine transitions | 🟡 **Partly** — only for logic independent of the school's day shape. Period resolution is blocked | `CL-TIME-005`, `CL-TIME-006`, no code |
| **Property-based** | Invariants that must hold for *any* timetable: no two lessons in one room at once, every published record traceable to an approver, no substitution without a matching scheduled lesson, append-only audit | 🟡 **Partly** — properties can be written as statements now, and this is the **highest-value layer available today**, because the properties do not require knowing the format. 🧪 Needs a generator over synthetic schedules, which needs a period model | Period model (`CL-TIME-005`/`006`); generator code |
| **Schema / contract** | Every ingested document validates against a declared schema before any row is written; unknown fields, wrong types and missing required fields are rejected | 🟡 **Partly** — the *validation harness* can be designed; the schema cannot be finalised | `CL-TIME-002`, `CL-TIME-003`; no schema authored |
| **Integration** | Service against a **fake NAS** and a **fake SHANA**, covering mounts, permission responses, timeout, corruption, robot-unreachable | 🔴 **No** — no service exists to test | No code; `CL-NAS-001`/`002`/`007`; `CL-SHANA-001` |
| **End-to-end** | A browser renders the display against a live host, across period boundaries and degraded states | 🔴 **No** — nothing to render | No frontend; ⛔ §2.7 blocks the real-network variant |
| **Manual / observational** | Real smartboard boot, real classroom viewing distance, a real substitution sheet filled in by a real coordinator | 🔴 **No** — no hardware access | `CL-WIN-001` (`P0`, `UNANSWERED`), no school permission |

### 3.1 A note on the ordering

> ⚠️ **Property-based tests are listed above unit tests deliberately.**

When the format is unknown, the useful question is not *"does the parser return
the right rows?"* — nobody knows what the right rows are. The useful question
is *"does the system **never** do these things?"* A double-booked room, a
substitution with no scheduled lesson, a published record with no approver, an
audit log that shrank. Those are falsifiable without knowing the format, and
they are the properties that matter.

[`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §5
already defines five conflict rules (R-1 … R-5). Each is a property. Each is
testable against generated input. None requires the school's file.

---

## 4. Determinism rules

> ### ⛔ These are rules, not preferences. A test suite that violates them is worse than no suite, because it produces green runs that mean nothing.

| # | Rule | Why |
|---|---|---|
| **D1** | **No test may read the wall clock.** Time is **injected** — a `Clock` passed in, never `now()` called inside | A test that reads the real clock passes at 09:00 and fails at 09:01. More importantly, it cannot test a period boundary without waiting for one |
| **D2** | **No test may depend on the school calendar.** Holidays, term dates and the academic year are **fixtures**, not imports | `CL-TIME-013`/`014`/`015` are unanswered. A test bound to real dates fails in August |
| **D3** | **No test may touch the network.** No live NAS, no live SHANA, no live school host | ⛔ Probing school infrastructure is prohibited. Integration tests use fakes, and are labelled as such |
| **D4** | **No test may depend on an AI model.** | See D5 |
| **D5** | **Any AI-dependent test is a *characterisation* test, not a correctness test.** It records what the system did on given input. It never asserts the answer is *right* | Extraction is non-deterministic (§2.3). A correctness assertion against a model is a flaky test that gets deleted when it annoys someone |
| **D6** | **Failures must be injected, not waited for.** Stopping the host must be a deliberate act in a controlled test, never an ambient condition | §2.4 |
| **D7** | **Every test states which `FR-*` it serves.** A test serving no requirement is either redundant or unmotivated | 133 requirements, no coverage — the link is the only thing that makes the suite navigable |
| **D8** | **⛔ A test that passes for an unstated reason is a defect.** If it would still pass if the feature were removed, it does not test the feature | The commonest false-confidence failure in suites written ahead of implementation |
| **D9** | **Every assertion must be one a human would agree is wrong if violated.** "Response is 200 OK" is not a safety assertion | The invariants in §5 are the specification that matters |

⚠️ **D5 is the one that gets broken first.** Under deadline, a test asserting
extracted text equals a fixture string looks like coverage. It is not. It is a
snapshot of one run.

---

## 5. Safety-relevant invariants that must be tested before any pilot

> ⚠️ **These are the reason the product is safe or is not.** They are drawn from
> [`../first_release_scope.md`](../first_release_scope.md) §5, the three
> invariants in [`../classroom_dashboard.md`](../classroom_dashboard.md), and
> `FR-*` requirements marked `CONFIRMED` in [`../requirements.md`](../requirements.md).

Each is written as a falsifiable statement, because *"be careful"* is not
testable and *"never"* is.

### 5.1 Display correctness

| # | Invariant | Testable statement | Serves |
|---|---|---|---|
| **S1** | **An unapproved substitution is never displayed as a teacher** | For any substitution in state `proposed`, the string of the proposed teacher's name appears **nowhere** in the rendered output — not in the main region, not in a tooltip, not in an accessible name, not in a `title` attribute, not in the DOM | `FR-SUB-01`, `FR-SUB-02`, [`../decision_log.md`](../decision_log.md) D-008 |
| **S2** | **A missing teacher is never invented** | With a teacher removed from source, the display shows an explicit "teacher to be confirmed" state. ⛔ No fallback to the scheduled teacher, no department default, no previous week's value, no empty field, no initials | `FR-SUB-03`, `FR-SUB-04` |
| **S3** | **A quarantined record never publishes** | A document failing deterministic validation produces no published output on any display, is retained and inspectable, and yields an operator-visible rejection naming the rule that rejected it | `FR-TIME-10`, [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §8 |
| **S4** | **A blank screen never appears where data is absent** | For every terminal state — no snapshot, host down, corrupt payload, permission denied — the display renders a distinct, non-empty, diagnosable state. ⛔ Never a black panel, never an unbounded spinner, never a browser error page | `FR-DASH-07`, [`../windows_startup.md`](../windows_startup.md) §5 tier (iii) |
| **S5** | **Stale data is always marked** | Whenever data is served from cache, the age of that data is visible on screen. When age exceeds the configured threshold, staleness is visible, not merely implied | `FR-DASH-06`, `CL-SEC-018` |
| **S6** | **No number is shown the system does not currently believe** | With an unsynced or unverified clock, no countdown is displayed; an explicit unreliable marker is shown instead | `FR-DASH-05` |
| **S7** | **Scheduled and confirmed are visually distinguishable without colour** | Every substitution state is distinguishable in greyscale. ⛔ Colour is never the sole carrier of meaning | `FR-DASH-11` |
| **S8** | **No student-identifying information ever reaches a display** | No name, mark, grade, attendance or identifier appears in any display payload | `FR-DASH-10`, `FR-SUB-12` |

### 5.2 Access control

| # | Invariant | Testable statement | Serves |
|---|---|---|---|
| **S9** | **Search never returns a document the user cannot open** | For every result returned, the same user is entitled to open that document. Checked at **query time and again at open time** | `FR-SEARCH-03`, `FR-RES-04` |
| **S10** | **Entitlement filtering precedes ranking** | No property of an inaccessible document influences ordering, counts, or any aggregate, in any direction | `FR-SEARCH-02`, `FR-SEARCH-05` |
| **S11** | **A code alone never authenticates a person** | Presenting a valid six-digit code alone yields an *action authorisation*, never an identity. The UI wording must not read as a signed-in teacher | [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) §5.2, `FR-AUTH-01` |
| **S12** | **A code can never change a permission or an account** | For every never-list action, no sequence of code entries — including under lockout, expiry and reuse conditions — grants the action | [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) §5.1 |
| **S13** | **No teacher session survives on an unattended display** | After idle timeout, after absolute timeout, after the display is rebound, and after sign-out, no valid session remains. Invalidated **server-side**, not by deleting a token client-side | `FR-AUTH-02`, `FR-AUTH-03`, `FR-AUTH-04` |
| **S14** | **Authorisation is re-evaluated on every request** | A permission revoked after sign-in takes effect on the next request with no re-login | `FR-AUTH-07` |
| **S15** | **Administration is unreachable from a display** | No credential, endpoint, route or session from a classroom segment reaches an administrative function | `FR-ADMIN-02`, `FR-DASH-01` |
| **S16** | **Auditing cannot be disabled, by anyone** | ⛔ No configuration, role or debug path exists that stops audit records being written, including for an administrator | `FR-ADMIN-06`, `FR-ADMIN-03` |
| **S17** | **A six-digit code cannot be brute-forced by reissuing** | Rate limiting holds against an attacker who requests a fresh code after every attempt, across codes, sessions and classrooms | [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) §6.2 |

### 5.3 Publication and rollback

| # | Invariant | Testable statement | Serves |
|---|---|---|---|
| **S18** | **Every published record is attributable to one named actor** | Every version, publication, correction, withdrawal and rollback carries an actor and a timestamp, retained for the retention period | `FR-TIME-08`, `FR-SUB-11`, `FR-ADMIN-03` |
| **S19** | **A withdrawn publication is withdrawn everywhere** | After a withdrawal, every display that received the erroneous version reconciles to the prior version, and the withdrawal appears in the audit trail | `FR-SUB-05`, `FR-SUB-06` |
| **S20** | **Rollback never erases the rollback** | A rollback is itself an immutable record; the version it superseded remains reachable | [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §7 |
| **S21** | **An interrupted update never leaves a partial release** | Killing an update at any point leaves the display on a complete previous version, never a half-applied one | `FR-OTA-06` |
| **S22** | **An unsigned or wrongly-signed bundle is refused** | Signature verification occurs **on the display**, at the point of consumption | `FR-OTA-01` |
| **S23** | **Software and data use separate approval paths** | No action approving a software change also approves a data change | `FR-OTA-04` |
| **S24** | **A conflict is surfaced, never silently resolved** | Two instructions of equal authority conflicting produces a conflict state visible to a human. ⛔ The system never picks one | `FR-SUB-13`, `FR-TIME-09` |
| **S25** | **No automated deployment without per-deployment approval** | Until the school says otherwise, no release reaches any display without an explicit approval step | `FR-OTA-03` |

### 5.4 Boundary with SHANA

| # | Invariant | Testable statement | Serves |
|---|---|---|---|
| **S26** | **⛔ No motor command path is reachable from CloudLight** | Statically and dynamically: no function, route, payload field or serialisation in the CloudLight codebase can express motor, actuator, path, speed or localisation intent. This is a **negative** property and must be tested by attempting construction, not by reading the design | `FR-SHANA-01`, `FR-SHANA-02`, [`../decision_log.md`](../decision_log.md) X-10 |
| **S27** | **CloudLight operates fully when SHANA is unreachable** | With the robot link down, the timetable, substitution and dashboard paths are unaffected | `FR-SHANA-04` |
| **S28** | **`UNKNOWN` is never rendered as `IN_PROGRESS` or complete** | Every robot status mapping is checked; `UNKNOWN` maps only to an unknown state | `FR-SHANA-05`, `FR-DASH-14` |
| **S29** | **A retry never creates a second delivery** | Every delivery request carries a unique idempotency key; a repeated request with the same key produces exactly one delivery | `FR-SHANA-06` |
| **S30** | **A delivery is never shown complete without a status from SHANA** | No local inference of completion | `FR-SHANA-10` |

⚠️ **S26 deserves particular attention.** It is the one invariant in this
document that cannot be verified by reading a specification, because it is a
property of an absence. It can only be tested by *trying to build one* and
failing. See [`known_failures.md`](known_failures.md).

---

## 6. What cannot be automated

⚠️ **Some things are only provable by a supervised human with the real thing in
front of them.** Listing these is not admitting defeat; it is the difference
between evidence and its appearance.

| Check | Why no test can substitute | Owner |
|---|---|---|
| **A real smartboard boots to the dashboard unattended** | Requires the board, the kiosk edition, and permission. `CL-WIN-001` is `P0`, `UNANSWERED` — the display design may be impossible | School IT, supervised |
| **Durable offline caching inside an InPrivate kiosk session** | ⛔ **Version-dependent on the school's exact Edge build.** Cannot be simulated with a different browser | School IT, on one board |
| **Legibility at real classroom viewing distance** | A 32-character subject name and a 2-character one are both "correct" and both unreadable from row 8 | Teacher, standing where a pupil stands |
| **A real substitution sheet, filled in by a real coordinator, in the real handwriting** | The only real test of the ingestion path. ⛔ A synthetic fixture proves the parser handles the author | Timetable department |
| **Legibility under classroom lighting and glare** | Screenshots do not model reflection | Teacher |
| **That a teacher trusts it enough to use it** | `CL-GEN-011` — *would teachers accept a board that is occasionally out of date?* ⛔ That answer sets the entire accuracy target | Administration + teachers |
| **That the coordinator can operate it under time pressure** | `P6` in [`../first_release_scope.md`](../first_release_scope.md) §3 — one IT generalist with other duties | The IT generalist, observed |
| **Colour-blind legibility of substitution states** | Colour vision is not a unit-testable property | Teacher |
| **Whether the whiteboard still works after lockdown** | ⚠️ [`../windows_startup.md`](../windows_startup.md) §8 calls this the risk most likely to end the project — *politically*, not technically | School IT, with the vendor |

⚠️ **Every one of these is a school-permissioned human check.** None can be
performed from this repository, and none should be simulated and reported as
done.

---

## 7. Honest position

> ### ⛔ **No test suite is justified before the `P0` gates pass.**

The reasoning is not that testing is expensive. It is that **tests written
against an unconfirmed design verify the design's assumptions rather than its
behaviour**, and the assumptions here are unusually load-bearing:

1. **The period model is unknown.** `CL-TIME-005`/`006` — a timetable test suite
   written now tests an invented day.
2. **The approver is unnamed.** `CL-SUB-001`/`003` — the approval gate is the
   core correctness property, and we do not know who operates it.
3. **The storage may not exist.** `CL-NAS-001` — half the `FR-*` requirements
   have no system to test.
4. **The display may be impossible to deploy.** `CL-WIN-001` — the recommended
   client may not be available.
5. **The host may not be permitted.** `CL-SRV-001` — ⛔ no local-first premise
   at all.

### 7.1 What to do instead, now

| Priority | Action | Why |
|---|---|---|
| **1** | **Write the invariants as executable specifications** — S1…S30 above, as executable assertions against stub types | They need no real format, no real data and no school. They encode intent before it is diluted |
| **2** | **Property-based tests over generated schedules** | Highest-value layer available today (§3.1) |
| **3** | **A deterministic test harness skeleton** — injected clock, injected data source, seeded randomness | This is infrastructure, not a guess about behaviour, and it survives every design answer |
| **4** | **Schema validation scaffolding**, once a real document exists | Blocked by `CL-TIME-002`/`003` |
| **5** | **Everything in [`test_inventory.md`](test_inventory.md)** | Blocked |

### 7.2 The honest summary

> ⚠️ **Testing is downstream of confirmation, not upstream of it.** In this
> project the scarce resource is not engineering effort — it is *answers from
> named people at one school*. Every hour spent writing tests against a design
> whose format, approver, storage, display and host are all unconfirmed is an
> hour that may need to be spent again.
>
> ✅ The tests worth writing now are those that hold **whatever the answers turn
> out to be** — the negative properties, the invariants, the never-lists. Those
> are [`test_inventory.md`](test_inventory.md)'s `CT-029…034` block, and they
> are the only tests genuinely ready.

---

**Related documents:** [`test_inventory.md`](test_inventory.md) ·
[`acceptance_criteria.md`](acceptance_criteria.md) ·
[`known_failures.md`](known_failures.md) · [`../requirements.md`](../requirements.md) ·
[`../first_release_scope.md`](../first_release_scope.md) ·
[`../feasibility_report.md`](../feasibility_report.md) ·
[`../architecture.md`](../architecture.md) ·
[`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) ·
[`../classroom_dashboard.md`](../classroom_dashboard.md) ·
[`../windows_startup.md`](../windows_startup.md) ·
[`../network_and_hosting.md`](../network_and_hosting.md) ·
[`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) ·
[`../resource_search.md`](../resource_search.md) ·
[`../decision_log.md`](../decision_log.md) ·
[`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md)