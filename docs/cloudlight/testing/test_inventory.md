# CloudLight — Test Inventory

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ THIS INVENTORY LISTS TESTS THAT ARE PLANNED AND DO NOT EXIST
>
> **No test has been executed. There are no results. There is no coverage.**
>
> Every row below is a **plan**. The `Status` column reads `NOT WRITTEN` and the
> `Test result` column reads `NOT RUN` **on every single row, without
> exception**. Those two columns exist so that the emptiness is visible rather
> than implied.
>
> ⛔ If a row ever reads anything else, that row is a claim about a system that
> does not exist and must be treated as an error.

**Read with:** [`test_strategy.md`](test_strategy.md) · Test IDs `CT-*` ·
Requirement IDs `FR-*` from [`../requirements.md`](../requirements.md)

---

## How to read this table

| Column | Meaning |
|---|---|
| **Test ID** | Stable identifier, `CT-nnn`. Assigned in blocks so a block can be read as a unit |
| **Target** | The requirement or behaviour under test, by name and `FR-*` ID |
| **Layer** | Unit · Property · Contract · Integration · E2E · Manual — defined in [`test_strategy.md`](test_strategy.md) §3 |
| **Asserts** | The falsifiable statement. Written so that a violation is unambiguously wrong |
| **Status** | `NOT WRITTEN` — the test has not been authored |
| **Test result** | `NOT RUN` — the test has not been executed |

### Blocks

| Block | Area | Testable today? |
|---|---|---|
| `CT-001…018` | Timetable ingestion and parsing | 🔴 Blocked on `CL-TIME-002`/`003` |
| `CT-019…028` | Substitution approval and state model | 🟡 Mechanism testable; policy blocked |
| `CT-029…034` | Conflict detection | 🟡 **Property-based — highest value available now** |
| `CT-035…040` | Publication, versioning and rollback | 🔴 Blocked on service |
| `CT-041…046` | Staleness and offline states | 🔴 Blocked on durable-cache unknown |
| `CT-047…052` | Dashboard rendering | 🟡 Partly — state catalogue testable |
| `CT-053…059` | Authentication and the six-digit code | 🟡 Rules testable; thresholds blocked |
| `CT-060…065` | Permission-filtered search | 🔴 Blocked on `CL-NAS-001` |
| `CT-066…071` | Update signing and rollback | 🔴 Blocked on signing infrastructure |
| `CT-072…077` | SHANA boundary enforcement | 🟡 Negative tests testable now |

⚠️ **"Testable today" does not mean written.** It means *not blocked by a
missing school answer*. The work still has not been done.

---

## `CT-001…018` — Timetable ingestion and parsing

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-001` | Valid source document accepted (`FR-TIME-11`) | Contract | A structurally valid document produces records and **no** quarantine entry | `NOT WRITTEN` | `NOT RUN` |
| `CT-002` | Structurally invalid document rejected (`FR-TIME-10`) | Contract | An invalid document produces **zero** published records, one quarantine entry, and an operator-visible rejection naming the failing rule | `NOT WRITTEN` | `NOT RUN` |
| `CT-003` | Quarantine retains the rejected record (`FR-TIME-10`) | Contract | A quarantined document remains retrievable and inspectable by an operator. ⛔ It is not a bin, not a delete, not silently discarded | `NOT WRITTEN` | `NOT RUN` |
| `CT-004` | Source document preserved (`FR-TIME-12`) | Unit | Every derived record can be traced to the exact source document and location it came from | `NOT WRITTEN` | `NOT RUN` |
| `CT-005` | Unknown columns tolerated (`FR-TIME-13`) | Contract | An unrecognised column does not fail the import and does not create a record | `NOT WRITTEN` | `NOT RUN` |
| `CT-006` | Partial failure reported, not fatal (`FR-TIME-13`) | Contract | A class absent from the source is reported as absent. ⛔ The whole import does not abort | `NOT WRITTEN` | `NOT RUN` |
| `CT-007` | Missing required field rejected deterministically (`FR-TIME-02`) | Contract | The same invalid input is rejected identically on every run, with the same reason code | `NOT WRITTEN` | `NOT RUN` |
| `CT-008` | Read-only to the school's source (`FR-TIME-01`) | Contract | Ingestion opens the source read-only. ⛔ No write, rename, move or lock is attempted on school storage | `NOT WRITTEN` | `NOT RUN` |
| `CT-009` | CloudLight is never the sole copy (`FR-TIME-14`) | Manual | The school's authoritative record exists independently and is unchanged after ingestion | `NOT WRITTEN` | `NOT RUN` |
| `CT-010` | Period profile is configuration, not a constant (`FR-TIME-02`) | Unit | Two different period profiles — differing period counts, durations and breaks — resolve correctly from the same code path | `NOT WRITTEN` | `NOT RUN` |
| `CT-011` | Non-uniform period durations honoured (`FR-TIME-03`) | Property | For any profile containing a double, half or practical period, remaining time is computed from that period's own duration | `NOT WRITTEN` | `NOT RUN` |
| `CT-012` | Breaks terminate the preceding lesson (`FR-TIME-04`) | Unit | A lesson ending at a break does not continue counting into the next period | `NOT WRITTEN` | `NOT RUN` |
| `CT-013` | Academic calendar boundary handled (`FR-TIME-05`) | Unit | Given a date outside the configured calendar, the system applies the documented rollover/expiry rule rather than serving a prior term | `NOT WRITTEN` | `NOT RUN` |
| `CT-014` | Special day is an alternate timetable (`FR-TIME-06`) | Unit | A special day resolves against its own profile and never as an ad-hoc edit to the master | `NOT WRITTEN` | `NOT RUN` |
| `CT-015` | No machine-extracted timetable publishes without approval (`FR-TIME-10`) | Integration | A timetable derived by extraction never reaches a display without a named human approval step | `NOT WRITTEN` | `NOT RUN` |
| `CT-016` | No AI fills a missing value | Unit | A field absent from the source remains absent. ⛔ No model output, default or inference supplies it | `NOT WRITTEN` | `NOT RUN` |
| `CT-017` | Extraction is characterisation-only | Contract | Recorded as a **characterisation** test: asserts structural properties of extraction output, never that a specific value is correct | `NOT WRITTEN` | `NOT RUN` |
| `CT-018` | Time is injected, never read (`D1`) | Unit | With an injected clock, no code path calls the system clock. A test that would change outcome across a real-time boundary proves this | `NOT WRITTEN` | `NOT RUN` |

---

## `CT-019…028` — Substitution approval and state model

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-019` | Unapproved substitution never displays as a teacher (`FR-SUB-01`) | Unit | For a record in `proposed`, the proposed name appears **nowhere** in rendered output — not in text, tooltip, accessible name, `title`, or DOM | `NOT WRITTEN` | `NOT RUN` |
| `CT-020` | Proposed state shows an explicit pending marker (`FR-SUB-02`) | E2E | A proposed substitution renders a pending state, and `proposed` is visually distinct from `scheduled` | `NOT WRITTEN` | `NOT RUN` |
| `CT-021` | States do not collapse (`FR-SUB-02`) | Property | `scheduled`, `proposed`, `cancelled` and `room-changed` remain four distinct states. ⛔ None is silently mapped to another | `NOT WRITTEN` | `NOT RUN` |
| `CT-022` | No automatic substitute selection (`FR-SUB-03`) | Property | Given no named replacement, no teacher is selected, suggested as confirmed, or displayed in any position | `NOT WRITTEN` | `NOT RUN` |
| `CT-023` | "No teacher" is an explicit outcome (`FR-SUB-04`) | Contract | A substitution with no teacher is stored and displayed as the no-teacher outcome, distinct from both a free period and a cancelled lesson | `NOT WRITTEN` | `NOT RUN` |
| `CT-024` | Missing teacher is never invented (`FR-SUB-03`) | E2E | With the teacher removed from source, the display shows "teacher to be confirmed". ⛔ No fallback to scheduled teacher, department default, previous week, initials or empty field | `NOT WRITTEN` | `NOT RUN` |
| `CT-025` | Approver is recorded (`FR-SUB-11`) | Unit | Every substitution record carries the approving actor and timestamp, and the record is unpublishable without them | `NOT WRITTEN` | `NOT RUN` |
| `CT-026` | Photograph is never authoritative (`FR-SUB-07`) | Integration | An ingested substitution photograph alone never results in a confirmed substitution | `NOT WRITTEN` | `NOT RUN` |
| `CT-027` | Photograph ingestion requires permission (`FR-SUB-08`) | Unit | Ingestion of a photograph is refused absent explicit recorded permission, and the refusal is logged | `NOT WRITTEN` | `NOT RUN` |
| `CT-028` | Mid-lesson effective time honoured (`FR-SUB-09`) | Unit | A substitution effective mid-lesson takes effect at its own effective time, not at the period boundary | `NOT WRITTEN` | `NOT RUN` |

---

## `CT-029…034` — Conflict detection

> ⚠️ **This is the block most worth writing first.** Conflicts are properties of
> the data, not of the school's file format, so they can be tested against
> generated schedules before `CL-TIME-002` is answered. Rules `R-1`…`R-5` are
> defined in [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §5.

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-029` | `R-1` — substitution must match a scheduled lesson | Property | A substitution naming a slot with no scheduled lesson is a **conflict**, never applied | `NOT WRITTEN` | `NOT RUN` |
| `CT-030` | `R-2` — replacement must be free | Property | A substitution to a teacher already allocated in that period is a conflict | `NOT WRITTEN` | `NOT RUN` |
| `CT-031` | `R-3` — no double-claimed slot | Property | Two documents claiming the same slot produces a conflict. ⛔ Never resolved automatically | `NOT WRITTEN` | `NOT RUN` |
| `CT-032` | `R-4` — room occupancy and suitability | Property | An occupied room is a conflict; a subject/room mismatch raises a review flag | `NOT WRITTEN` | `NOT RUN` |
| `CT-033` | `R-5` — late restatement deduplicates (`FR-SUB-13`) | Property | A late sheet restating an already-applied substitution produces no conflict, retains both versions, and creates no second publication | `NOT WRITTEN` | `NOT RUN` |
| `CT-034` | No room double-booking ever publishes (`FR-TIME-09`) | Property | For any generated timetable, no two teachers occupy the same room in the same period in published output. ⛔ Detected and refused, never resolved | `NOT WRITTEN` | `NOT RUN` |

---

## `CT-035…040` — Publication, versioning and rollback

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-035` | Every change attributable to one actor (`FR-TIME-08`) | Unit | Every create, change, correction, withdrawal and rollback carries a single named actor and timestamp | `NOT WRITTEN` | `NOT RUN` |
| `CT-036` | Staging to production is a recorded step (`FR-ADMIN-07`) | Integration | Corrected configuration reaches production only via a recorded promotion. ⛔ No direct edit to live data | `NOT WRITTEN` | `NOT RUN` |
| `CT-037` | Withdrawal propagates to all displays (`FR-SUB-05`) | E2E | After a withdrawal, every display that received the erroneous version reconciles to the prior version | `NOT WRITTEN` | `NOT RUN` |
| `CT-038` | Withdrawal is audited (`FR-SUB-05`) | Unit | The withdrawal appears in the append-only audit trail with actor and timestamp | `NOT WRITTEN` | `NOT RUN` |
| `CT-039` | Rollback does not erase history (`FR-SUB-05`) | Property | After a rollback the superseded version remains reachable and the rollback is itself an immutable record | `NOT WRITTEN` | `NOT RUN` |
| `CT-040` | Correction propagates within tolerance (`FR-SUB-06`) | Integration | Propagation to already-displaying boards completes within the school's stated tolerance. ⚠️ **No tolerance is stated** — `CL-SUB-023` | `NOT WRITTEN` | `NOT RUN` |

---

## `CT-041…046` — Staleness and offline states

> ⛔ **All six are additionally blocked by the durable-cache unknown in
> [`../windows_startup.md`](../windows_startup.md) §7.** The three tiers in §5 of
> that document are not implementable as drawn until durable storage in an
> InPrivate kiosk session is verified on the school's Edge build.

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-041` | Fresh cache shows last-synced (`FR-DASH-06`) | E2E | Cached data within tolerance renders normally **and** displays `Last synced: HH:MM` | `NOT WRITTEN` | `NOT RUN` |
| `CT-042` | Stale cache is visibly marked (`FR-DASH-06`) | E2E | Beyond the threshold, staleness and the age in minutes are both visible on screen | `NOT WRITTEN` | `NOT RUN` |
| `CT-043` | Never a blank screen (`FR-DASH-07`) | Property | For every terminal state — no snapshot, host down, corrupt payload, denied — a distinct, non-empty, diagnosable state renders. ⛔ No black panel, no unbounded spinner | `NOT WRITTEN` | `NOT RUN` |
| `CT-044` | Offline but usable (`FR-DASH-08`) | E2E | With the host stopped, today's lesson continues to render from cache, marked as to age | `NOT WRITTEN` | `NOT RUN` |
| `CT-045` | Holiday is explicit, not a normal day (`FR-DASH-09`) | E2E | An empty or holiday day renders "no lessons today". ⛔ Never a blank and never a normal lesson | `NOT WRITTEN` | `NOT RUN` |
| `CT-046` | Threshold is configurable and visible (`CL-SEC-018`) | Integration | The staleness threshold is a server-side configuration value, and the effective value is visible on the screen itself | `NOT WRITTEN` | `NOT RUN` |

---

## `CT-047…052` — Dashboard rendering

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-047` | Display is read-only (`FR-DASH-01`) | E2E | No interaction from a display changes timetable, substitution or resource state | `NOT WRITTEN` | `NOT RUN` |
| `CT-048` | No login required to view (`FR-DASH-02`, `FR-AUTH-14`) | E2E | A lesson renders with no login prompt. ⛔ No login wall appears as the normal way to view | `NOT WRITTEN` | `NOT RUN` |
| `CT-049` | Bound room, not nominal room (`FR-DASH-03`) | Unit | The lesson shown is the one bound to the display's room, including when the class is nominally elsewhere | `NOT WRITTEN` | `NOT RUN` |
| `CT-050` | Countdown accuracy against reference clock (`FR-DASH-04`) | Unit | Displayed remaining time matches a reference at intervals. ⚠️ **No tolerance stated** — the school sets it | `NOT WRITTEN` | `NOT RUN` |
| `CT-051` | Countdown suppressed when clock untrusted (`FR-DASH-05`) | E2E | With an unsynced or unverified time source, **no** countdown renders; an explicit unreliable marker renders instead | `NOT WRITTEN` | `NOT RUN` |
| `CT-052` | States distinguishable without colour (`FR-DASH-11`) | E2E | Every substitution state is distinguishable in greyscale. ⛔ Colour is never the sole carrier of meaning | `NOT WRITTEN` | `NOT RUN` |
| `CT-053` | No student data on a display (`FR-DASH-10`) | Contract | No student name, mark, grade, attendance or identifier appears in any display payload | `NOT WRITTEN` | `NOT RUN` |
| `CT-054` | State catalogue is bounded (`FR-DASH-15`) | Contract | The number of distinct display states is finite, documented, and exhaustively covered by tests | `NOT WRITTEN` | `NOT RUN` |

---

## `CT-055…062` — Authentication and the six-digit code

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-055` | Individual accounts supported (`FR-AUTH-01`) | Unit | Every action resolves to exactly one named identity. ⛔ No shared account action exists | `NOT WRITTEN` | `NOT RUN` |
| `CT-056` | Code alone never authenticates a person (`FR-AUTH-01`) | E2E | A valid code alone grants an **action authorisation**, never an identity. ⛔ No interface text reads as a signed-in teacher | `NOT WRITTEN` | `NOT RUN` |
| `CT-057` | Never-list is unreachable by code (`FR-AUTH-01`) | Property | For every action in the never-list — permission change, account creation/deletion/disable, MFA settings, confidential folders, break-glass approval, deployment signing — no code sequence authorises it, under any rate-limit, expiry or reuse state | `NOT WRITTEN` | `NOT RUN` |
| `CT-058` | Code is single-use and invalidated on success | Integration | A code is consumed within the same transaction as the authorised action and is dead immediately afterwards | `NOT WRITTEN` | `NOT RUN` |
| `CT-059` | Reissuing does not reset the attempt budget | Property | Rate limiting holds against an attacker who requests a new code after each attempt, across codes, sessions and classrooms | `NOT WRITTEN` | `NOT RUN` |
| `CT-060` | No session survives on an unattended display (`FR-AUTH-02`) | E2E | After idle timeout, absolute timeout, display rebind and sign-out, no valid session remains — invalidated server-side | `NOT WRITTEN` | `NOT RUN` |
| `CT-061` | Idle and absolute timeouts are distinct (`FR-AUTH-03`) | Unit | The two controls are independent; neither can be configured out of existence by setting the other | `NOT WRITTEN` | `NOT RUN` |
| `CT-062` | No token in URL or local storage (`FR-AUTH-05`, `FR-AUTH-06`) | Contract | No session token appears in any URL, `localStorage` or `sessionStorage` | `NOT WRITTEN` | `NOT RUN` |
| `CT-063` | Authorisation re-evaluated per request (`FR-AUTH-07`) | Integration | A permission revoked after sign-in takes effect on the very next request, with no re-login | `NOT WRITTEN` | `NOT RUN` |
| `CT-064` | Lockout locks the action, not the display (`FR-AUTH-08`) | Integration | A lockout contains the pending authorisation; the display continues to show the lesson | `NOT WRITTEN` | `NOT RUN` |
| `CT-065` | No secret in plaintext (`FR-AUTH-12`) | Contract | No password or secret appears in plaintext in storage, logs, URLs or error output | `NOT WRITTEN` | `NOT RUN` |

---

## `CT-066…071` — Permission-filtered search

> ⛔ **The whole block is blocked on `CL-NAS-001`.** A permission test against a
> mock directory proves the mock is correct. See [`test_strategy.md`](test_strategy.md) §2.5.

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-066` | Filter precedes ranking (`FR-SEARCH-02`) | Integration | Entitlement filtering is applied **before** ranking. No ordering, count or aggregate reflects an inaccessible document | `NOT WRITTEN` | `NOT RUN` |
| `CT-067` | Never return unopenable results (`FR-SEARCH-03`) | Integration | Every returned result is openable by that user | `NOT WRITTEN` | `NOT RUN` |
| `CT-068` | Re-check at open time (`FR-RES-04`) | Integration | Permission is re-verified when the document is opened, not only when it was indexed | `NOT WRITTEN` | `NOT RUN` |
| `CT-069` | Storage ACLs are authoritative (`FR-RES-03`) | Integration | CloudLight honours the storage's own ACLs. ⛔ No second, weaker permission model is applied | `NOT WRITTEN` | `NOT RUN` |
| `CT-070` | No cross-boundary aggregate leak (`FR-SEARCH-05`) | Integration | No field name, term count or statistic reveals the existence of a term in an inaccessible document | `NOT WRITTEN` | `NOT RUN` |
| `CT-071` | Read-only service account (`FR-RES-02`) | Integration | CloudLight accesses storage only with a dedicated read-only credential; no staff or admin credential is reused | `NOT WRITTEN` | `NOT RUN` |
| `CT-072` | Index staleness is visible (`FR-SEARCH-04`) | Integration | The age of the index is shown to staff, and "not indexed" is distinguished from "indexed, you may see it" | `NOT WRITTEN` | `NOT RUN` |
| `CT-073` | Metadata-only degradation (`FR-SEARCH-06`) | Integration | When the content index is unavailable, search degrades to metadata rather than failing | `NOT WRITTEN` | `NOT RUN` |
| `CT-074` | No write to school storage (`FR-RES-01`) | Integration | No write, move, rename or delete is ever issued against school storage | `NOT WRITTEN` | `NOT RUN` |

---

## `CT-075…080` — Update signing and rollback

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-075` | Unsigned bundle refused (`FR-OTA-01`) | Integration | A display refuses an unsigned frontend bundle and continues serving the previous version | `NOT WRITTEN` | `NOT RUN` |
| `CT-076` | Wrongly-signed bundle refused (`FR-OTA-01`) | Integration | A bundle signed by an untrusted key is refused. Verification occurs **on the display** | `NOT WRITTEN` | `NOT RUN` |
| `CT-077` | Interrupted update aborts cleanly (`FR-OTA-06`) | Property | Terminating an update at any point leaves a complete previous version, never a partial one | `NOT WRITTEN` | `NOT RUN` |
| `CT-078` | Rollback to last known good (`FR-OTA-02`) | Integration | Rollback restores the previous known-good release on the display | `NOT WRITTEN` | `NOT RUN` |
| `CT-079` | No automatic deployment (`FR-OTA-03`) | Integration | No release reaches any display without an explicit per-deployment approval | `NOT WRITTEN` | `NOT RUN` |
| `CT-080` | Separate approval paths (`FR-OTA-04`) | Integration | No credential or workflow step approves both a software change and a data change | `NOT WRITTEN` | `NOT RUN` |
| `CT-081` | One version behind tolerated (`FR-OTA-05`) | Integration | A display running a cached frontend one version behind the backend continues to render correctly | `NOT WRITTEN` | `NOT RUN` |
| `CT-082` | Per-display release record (`FR-OTA-08`) | Unit | Which release reached which display, when, by whom, and with what outcome is answerable from the record | `NOT WRITTEN` | `NOT RUN` |
| `CT-083` | Platform usable without update infrastructure (`FR-OTA-10`) | Integration | With the update infrastructure unreachable, timetable, substitution and display continue to work | `NOT WRITTEN` | `NOT RUN` |

---

## `CT-084…089` — SHANA boundary enforcement

> ⛔ **`CT-084` is the most important test in this inventory and the one most
> easily written to pass.** `FR-SHANA-01` is a prohibition on a *negative*
> property — it can be satisfied by an absence, and reading the design cannot
> confirm an absence. It can only be confirmed by **attempting to construct a
> motor command and failing**.

| Test ID | Target | Layer | Asserts | Status | Test result |
|---|---|---|---|---|---|
| `CT-084` | ⛔ No motor command path is constructible (`FR-SHANA-01`) | Property | ⛔ **Negative property.** Attempting to construct a message expressing motor, actuator, path, speed or localisation intent **fails to compile or is rejected at the boundary**. Asserted by construction attempt, never by reading the design | `NOT WRITTEN` | `NOT RUN` |
| `CT-085` | Only high-level intent crosses the boundary (`FR-SHANA-02`) | Contract | Every payload field is *what*, *to where* or *by when*. No field carries a motion primitive | `NOT WRITTEN` | `NOT RUN` |
| `CT-086` | CloudLight does not assert safety (`FR-SHANA-03`) | Contract | No CloudLight code path influences, overrides or bypasses SHANA's own safety validation | `NOT WRITTEN` | `NOT RUN` |
| `CT-087` | Full operation when SHANA is unreachable (`FR-SHANA-04`) | Integration | With the robot link down, timetable, substitution and dashboard paths are entirely unaffected | `NOT WRITTEN` | `NOT RUN` |
| `CT-088` | `UNKNOWN` is never rendered as progress (`FR-SHANA-05`, `FR-DASH-14`) | Property | Every status mapping is exhaustively checked; `UNKNOWN` maps only to an unknown state, never to `IN_PROGRESS` or complete | `NOT WRITTEN` | `NOT RUN` |
| `CT-089` | Retry never duplicates a delivery (`FR-SHANA-06`) | Integration | Every request carries a unique idempotency key; a repeated request with the same key yields exactly one delivery | `NOT WRITTEN` | `NOT RUN` |
| `CT-090` | No completion inferred locally (`FR-SHANA-10`) | Integration | A delivery is never displayed complete without a status received from SHANA | `NOT WRITTEN` | `NOT RUN` |
| `CT-091` | No student data in any delivery (`FR-SHANA-12`) | Contract | No student-identifying information appears in any delivery request | `NOT WRITTEN` | `NOT RUN` |
| `CT-092` | Robot link is a distinct network path (`FR-SHANA-08`) | Manual | The robot link shares no broadcast domain with the classroom LAN | `NOT WRITTEN` | `NOT RUN` |

---

## Summary

| | Count |
|---|---|
| Tests planned | **92** |
| Tests written | **0** |
| Tests executed | **0** |
| Coverage | **None exists** |
| Requirements with at least one planned test | 0 of 133 |

| Class | Count | Note |
|---|---|---|
| `CONFIRMED` requirements covered by a planned test | 24 | The invariants; testable once code exists |
| `PROPOSED` requirements covered | 31 | Agreed by nobody |
| `PENDING ANSWER` requirements covered | 18 | Cannot be specified until a `CL-*` answer arrives |
| Requirements with **no** planned test | 60 | Includes every release-capability not in Release 0 |

⚠️ **The "requirements with no planned test" figure is a deliberate omission,
not an oversight.** A test for a capability that is out of scope for the first
release would assert behaviour nobody has agreed to build.

### 92 planned · 0 written · 0 run · 0 results

---

**Related documents:** [`test_strategy.md`](test_strategy.md) ·
[`acceptance_criteria.md`](acceptance_criteria.md) ·
[`known_failures.md`](known_failures.md) ·
[`../requirements.md`](../requirements.md) ·
[`../first_release_scope.md`](../first_release_scope.md) ·
[`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) ·
[`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) ·
[`../resource_search.md`](../resource_search.md) ·
[`../windows_startup.md`](../windows_startup.md) ·
[`../decision_log.md`](../decision_log.md)