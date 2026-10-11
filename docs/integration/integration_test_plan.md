# Integration Test Plan

**Owner:** Agent 3 (Integration Lead)
**Status:** 🔴 **`BLOCKED` — no integration test can be written or run today.**
**Last updated:** 2026-10-11

---

## 🔴 The honest position

| Fact | Value |
|---|---|
| Source files | **0** |
| Tests written | **0** |
| Tests executed | **0** |
| Integration endpoints to test | **0** |
| 🔴 **Test plan status** | **`BLOCKED`** |

🔴 **This document specifies tests that cannot currently be written.** It exists
because the *absence* of a test plan is what makes an integration appear ready,
and because the ordering here — what to test first, and what each test is
entitled to conclude — is a cross-project decision that neither subsystem agent
can make alone.

⚠️ **A strategy that has never been executed is not evidence of testing.**
`docs/shana/testing/` and `docs/cloudlight/testing/` currently describe
strategies only. This does the same and says so.

---

## 1. 🔴 Prerequisite — what must exist before any test is written

| # | Prerequisite | Owner | Status |
|---|---|---|---|
| 1 | `IC-04` delivery state vocabulary agreed | Agent 1 + 2 | 🔴 `CONF-02` open |
| 2 | `IC-02` identifier model agreed | Agent 1 + 2 | 🔴 `CONF-06` open |
| 3 | `IC-06` authentication specified | Agent 1 + 2 | 🔴 not agreed |
| 4 | Location registry defined | Agent 1 | 🔴 none exists |
| 5 | Both subsystems have runnable code | Agent 1 + 2 | ⛔ 0 source files |
| 6 | Error taxonomy agreed | Agent 1 + 2 | 🔴 `IC-10` open |
| 7 | Go/no-go gates permitting implementation | School | 🔴 **0 of 20 passed** |

🔴 **Requirement 7 is not a formality.** SHANA's own go/no-go gates forbid
implementation until they pass. Writing an integration test against code that
must not exist yet would be building the wrong thing confidently.

---

## 2. 🔴 The test that matters most — and cannot be written

### T-00: No state may be understated

> **For every state SHANA can enter, the status CloudLight displays must not
> cause a teacher to take less action than the true state requires.**

**Why this is first.** `CONF-02` demonstrates that a paused robot would display
as **"On the way"**. `AWAITING_RECIPIENT` — which means *"a human must go to the
robot now"* — also collapses into it.

| SHANA state | Required teacher action | Safe display |
|---|---|---|
| `QUEUED` | Wait | "Queued" |
| `VALIDATING` | Wait | "Checking" |
| `READY` | Wait | "Ready" |
| `NAVIGATING_TO_PICKUP` | Wait | "Going to collect" |
| `AWAITING_LOADING_CONFIRMATION` | 🔴 **A person must load it** | "Waiting for loading" |
| `NAVIGATING_TO_DESTINATION` | Wait | "On the way" |
| `AWAITING_RECIPIENT` | 🔴 **Go to the robot now** | "Waiting for you" |
| `COMPLETED` | — | "Delivered" |
| `PAUSED` | 🔴 **Something is wrong; a human is needed** | "Paused — needs help" |
| `CANCELLED` | — | "Cancelled" |
| `FAILED` | Read reason | "Failed: <reason>" |

⚠️ The table above is Agent 3's **proposal**, not an agreed contract. It
illustrates what `ADR-016` would require and what the current CloudLight
vocabulary fails.

**Falsification:** it is defeated the moment any state maps to a string that
understates urgency. 🔴 **Today, two states do.**

---

## 3. Contract tests — once code exists

| ID | Test | Requires | Owner |
|---|---|---|---|
| T-01 | Every SHANA state is displayable by CloudLight | `IC-04` agreed | Agent 2 |
| T-02 | `AWAITING_RECIPIENT` never displays as "On the way" | `IC-04` | Agent 2 |
| T-03 | `PAUSED` never displays as "On the way" | `IC-04` | Agent 2 |
| T-04 | `COMPLETED` requires `recipient_confirmed == true` | schema + code | Agent 1 |
| T-05 | Arrival without confirmation never yields `COMPLETED` | robot-side | Agent 1 |
| T-06 | Unknown destination is **rejected**, never improvised | location registry | Agent 1 |
| T-07 | No free-text destination field exists in any payload | schema | both |
| T-08 | Retrying the same `idempotency_key` creates **no second task** | `IC-02` | Agent 1 + 2 |
| T-09 | 🔴 An **unauthenticated** request is rejected | `IC-06` | Agent 1 |
| T-10 | 🔴 A request from CloudLight cannot express motion | `ADR-014` | Agent 1 + 2 |
| T-11 | 🔴 `system.stop` is honoured with no authorisation check | `IC-11.3` | Agent 1 |
| T-12 | Blocked route → fail closed, never fail open | `IC-11.6` | Agent 1 |
| T-13 | SHANA unreachable → CloudLight dashboard **fully functional** | `IC-08` | Agent 2 |
| T-14 | CloudLight unreachable → SHANA safe, no peer dependency | `IC-11.10` | Agent 1 |
| T-15 | Status is never shown as current without an age | `IC-11.9` | Agent 2 |
| T-16 | Cancellation never displays "Cancelled" pre-confirmation | `IC-09` | Agent 2 |
| T-17 | No AI output reaches any motion command path | `IC-11.1` | Agent 1 + 2 |

⚠️ **T-08 is a physical-safety test, not a data-integrity test.** Two tasks
from one tap is a second robot moving through a corridor. CloudLight's own
documentation classifies it as P0 for that reason.

---

## 4. 🔴 Tests that cannot be automated at all

⚠️ **These require physical, supervised validation by a human.** No amount of
software testing substitutes.

| ID | Validation | Requires |
|---|---|---|
| T-30 | The robot stops when a person is in its path | Hardware + spotter |
| T-31 | Battery reserve triggers a defined safe behaviour | Hardware + measurement |
| T-32 | The robot can complete a defined route repeatedly | Hardware + route survey |
| T-33 | Blocked corridor → human intervention path works | Hardware + supervision |
| T-34 | `system.stop` stops the robot **immediately** | Hardware |
| T-35 | Emergency stop is effective | Hardware + spotter |
| T-36 | Recovery from a failed delivery | Hardware |
| T-37 | The robot does not traverse a wet floor | ⚠️ **Not implemented.** Cannot be tested |
| T-38 | The robot does not attempt stairs | Hardware |
| T-39 | The robot does not enter a lift unaccompanied | Hardware |

🔴 **"Tests pass" is never a statement about safety.** 🔴 **Successful software
development does not authorise robot operation near students.**

---

## 5. 🔴 Tests that must never be written

| ❌ Prohibited | Why |
|---|---|
| Anything exercising a robot without a human spotter | `AGENTS.md` §L.7 |
| Any test on real student data | 🔴 Privacy invariant — none exists |
| Any test against school infrastructure without authorisation | 🔴 `AGENTS.md` §K |
| Any test contacting a CCTV system | 🔴 No authorisation exists |
| Any test contacting a lift system | 🔴 No authorisation exists |
| Any test that exercises motion from CloudLight | 🔴 That path must **not exist** — T-10 proves it does not |

⚠️ **T-10 is best understood as a negative test:** it passes when CloudLight
*cannot* express motion. 🔴 **If it fails, the architecture is broken, not the
test.**

---

## 6. Sequencing

⚠️ **Nothing below step 2 is reachable today.**

| # | Phase | Status |
|---|---|---|
| 1 | Agree `IC-02`, `IC-04`, `IC-06` | 🔴 `BLOCKED` — open conflicts |
| 2 | Write `T-00` as a table-driven assertion | 🔴 `BLOCKED` — needs step 1 |
| 3 | First code exists | ⛔ `BLOCKED` — gates |
| 4 | Contract tests T-01…T-17 | ⛔ `BLOCKED` |
| 5 | 🔴 **Integration test across both subsystems** | ⛔ `BLOCKED` |
| 6 | Physical validation T-30…T-39 | ⛔ `BLOCKED` — hardware + permission |

---

## 7. 🔴 What may be claimed today

| Claim | Status |
|---|---|
| "Integration tests verify…" | 🔴 **FALSE. No integration test exists.** |
| "The architecture has been validated" | 🔴 **FALSE. Design only.** |
| "The contract is agreed" | 🔴 **FALSE. 0 of 11 entries are `AGREED`.** |
| "A test strategy exists" | ✅ **TRUE** — as a document, never executed |
| "0 tests have been run" | ✅ **TRUE** |

⚠️ The second `docs/shana/testing/` document is named `known_failures.md`. ⚠️
**There are no failures, because nothing has run.** Its honest content is the
absence of tests.

---

## 8. Summary

| Metric | Value |
|---|---|
| Tests specified | **27** (17 contract · 9 physical · 1 negative-architecture) |
| 🔴 Tests written | **0** |
| 🔴 Tests run | **0** |
| 🔴 Prerequisites met | **0 of 7** |
| 🔴 Blocked by | Unagreed contract + no code + gates not passed |
| 🔴 Integrable today | **No** |