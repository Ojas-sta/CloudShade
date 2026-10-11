# SHANA — Acceptance Criteria

**Status: PROPOSED. No criteria have been agreed or approved.**

⚠️ **Acceptance criteria must be agreed in writing before testing begins.**
Criteria invented afterwards are not criteria.

---

## Gate acceptance criteria

A gate is passed only when **every** criterion below it is met with evidence.

**A gate is never passed without evidence. Authority to pass a gate rests with the
school, not with the project team.**

---

## Layer 1 — Safety acceptance (non-negotiable)

These are not performance targets. They are prerequisites. Failure at any of
them stops everything.

| # | Criterion | Evidence | Verifiable by |
|---|---|---|---|
| S-01 | Emergency stop **physically tested** and passes | Test record | Gate 8 |
| S-02 | Remote authority tested **before every run**, passes every time | Procedure + log | Every run |
| S-03 | Hard power switch reachable within the agreed time | Timed test | Gate 8 |
| S-04 | Named supervisor present at every operation | Roster | Gate 0 |
| S-05 | **Unilateral stop authority confirmed in writing** | Signed confirmation | Gate 0 |
| S-06 | Exclusion zone physically enforced | Site plan + rehearsal | Gate 8 |
| S-07 | Speed cap enforced in software, unoverridable | Code review + test | Gate 7 |
| S-08 | 🔴 **Zero uncommanded movements** across all tests | Test log | Gate 8 |
| S-09 | 🔴 **Host-kill test passes** — robot stops within the watchdog interval | Test record | Gate 8 |
| S-10 | Payload carrier tilt-tested and retention verified | Test record | Gate 8 |
| S-11 | Auto-recovery disabled during payload missions | Code review | Gate 6 |
| S-12 | Fire/evacuation procedure agreed with the school | Written procedure | Gate 7 |
| S-13 | Battery limits known and supervised charging agreed | Procedure | Gate 6 |
| S-14 | Incident and near-miss reporting process in place | Written procedure | Gate 8 |

⚠️ **S-08 and S-09 may not be waived under any circumstances, for any deadline.**

---

## Layer 2 — Architectural acceptance

| # | Criterion | Evidence |
|---|---|---|
| A-01 | The AI model **cannot** reach the robot | Code inspection + test |
| A-02 | The validator is a pure, deterministic function | Code review + N-run test |
| A-03 | Unknown entities are rejected, never guessed | Test suite |
| A-04 | Every model-derived task requires human confirmation | Test |
| A-05 | 🔴 **The system works with the LLM layer disabled** | Demonstration |
| A-06 | The safety supervisor can veto any command, and cannot be overridden by any role | Test |
| A-07 | Teachers have no motor-control capability | Role review |
| A-08 | No delivery completes without explicit recipient confirmation | Test |
| A-09 | 🔴 **The planner never selects stairs or lifts** | Test |
| A-10 | Closed and `UNKNOWN` segments are never traversed | Test |
| A-11 | A checkpoint's last-known position is never treated as current | Test |
| A-12 | A relocation event invalidates the pose | Test |
| A-13 | Loss of communications produces a safe hold | Test |
| A-14 | Battery threshold derived from measurement, not estimate | Test data |
| A-15 | No gate marked passed without evidence | Audit |

---

## Layer 3 — Operational acceptance

| # | Criterion | Threshold | Status |
|---|---|---|---|
| O-01 | Task completion rate over ≥30 runs | ⚠️ **TBD by school** | Unset |
| O-02 | Delivery success rate | ⚠️ **TBD** | Unset |
| O-03 | **Consecutive successful runs before pilot** | ⚠️ **TBD** — 10 recommended | Unset |
| O-04 | Obstacle response — detects and stops | 100% of injections | Proposed |
| O-05 | Localisation failure recovery | Recovers or aborts cleanly, 100% | Proposed |
| O-06 | Battery reserve — returns on every mission | 100% | Proposed |
| O-07 | Human interventions per run | ⚠️ **TBD by school** | Unset |
| O-08 | Every failure has a recorded reason | 100% | Proposed |
| O-09 | 🔴 **Zero safety incidents** | 0 | **Absolute** |
| O-10 | 🔴 **Zero unrecovered falls** | 0 | **Absolute** |
| O-11 | 🔴 **Zero payload drops** | 0 | **Absolute** |
| O-12 | Every intervention logged | 100% | Proposed |

⚠️ **O-09, O-10 and O-11 are absolute and non-negotiable.** A single occurrence
stops the project pending review.

⚠️ **O-01, O-02, O-03 and O-07 are deliberately unset.** ⚠️ **The school sets
performance thresholds, not the project team.** Setting our own would be
choosing a number we can then be measured against — which is not a standard.

---

## Layer 4 — Exhibition acceptance

⚠️ **Gate 10 cannot be satisfied by passing Layers 1–3.** Student-facing
demonstration requires separate authorisation.

| # | Criterion | Evidence |
|---|---|---|
| E-01 | Layers 1–3 all passed | Gate records |
| E-02 | **Separate written authorisation for student presence** | Signed approval |
| E-03 | Venue surveyed and confirmed safe | Site survey |
| E-04 | Scope and exclusions agreed in writing | Written plan |
| E-05 | ≥10 consecutive successful supervised runs | Test records |
| E-06 | Abort criteria agreed **before** the day | Written criteria |
| E-07 | Fallback demonstration prepared and tested | Test record |
| E-08 | Incident and first-aid plan agreed | Written plan |
| E-09 | Named supervisor and trained backup | Roster |
| E-10 | Teachers informed; objections recorded and honoured | Record |
| E-11 | No stairs, lifts, or live items | Written confirmation |

---

## Evidence standards

| Claim | Requires |
|---|---|
| "It works" | A recorded test with parameters and outcome |
| "It is reliable" | ≥30 runs with a confidence interval |
| "It is fast" | Measured duration, stated conditions |
| "It is safe" | 🔴 **Never asserted. Demonstrated by Gate 8 testing only** |
| "It recognises markers" | Detection rate by distance and angle |
| "It avoids obstacles" | Response latency, measured from video |

⚠️ **Simulation results must never be presented as hardware results.**

⚠️ **Manufacturer figures must never be presented as our measurements.**

⚠️ **A datasheet is not a measurement.**

---

## The honesty rule

> **Every number shown to a judge must come from measured data collected under a
> written protocol agreed in advance.**

| Prohibited | Why |
|---|---|
| Presenting a target as a result | Misrepresents capability |
| Presenting simulation as hardware | Different system |
| Presenting a datasheet as a measurement | Not our data |
| Reporting only successes | Destroys the evidence value |
| Rounding a failure away | Misleading |
| Omitting the denominator | A 100% of 3 is not 100% |

⚠️ **A project that reports honestly is more credible than one that reports
well.** Judges can tell the difference, and the second one is not a project — it
is a claim.

---

## Acceptance authority

| Layer | Who accepts |
|---|---|
| Layer 1 — Safety | School safety officer + administrator |
| Layer 2 — Architecture | School administrator |
| Layer 3 — Operations | School administrator + teachers |
| Layer 4 — Exhibition | School administrator + organiser |

⚠️ **The project team may not accept any layer.**

---

## Current state

| Layer | Criteria | Passed | Failures |
|---|---|---|---|
| 1 — Safety | 14 | **0** | **0** |
| 2 — Architecture | 15 | **0** | **0** |
| 3 — Operational | 12 | **0** | **0** |
| 4 — Exhibition | 11 | **0** | **0** |
| **TOTAL** | **52** | **0** | **0** |

⚠️ **52 criteria. None assessed. Nothing built, nothing tested, nothing
connected.**