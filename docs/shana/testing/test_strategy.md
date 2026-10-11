# SHANA — Test Strategy

**Status: PROPOSED. No test has been run, scheduled, or approved.**

⚠️ **Nothing can be tested on hardware until Gates 0, 1 and 8 are passed.** The
simulation tier can begin immediately.

---

## The five tiers

| Tier | What | Needs permission? | Can start now? |
|---|---|---|---|
| **T0** | Static analysis and code review | No | ✅ Yes |
| **T1** | Simulation (`unitree_mujoco`) | No | ✅ **Yes — start here** |
| **T2** | Bench tests — no robot movement | Some | ⚠️ After Gate 0 |
| **T3** | Supervised robot tests | Yes — Gate 8 | ❌ |
| **T4** | Pilot measurement | Yes — Gates 0–9 | ❌ |

⚠️ **Tiers 0 and 1 require nobody's permission and validate the entire software
architecture.** They should start immediately, in parallel with the permission
process.

---

## Tier 0 — Static analysis

| Test | Verifies |
|---|---|
| Model output cannot reach the robot | The AI boundary holds structurally |
| Validator is pure | No I/O, no network, no clock |
| No motion command outside the two navigating states | State machine invariant |
| Speed clamp cannot be bypassed | No interface overrides the cap |
| Validator is deterministic | Same input, N runs, identical output |
| No credentials in the repository | Hygiene |
| No dependency is GPL or unlicensed | Licence compliance |
| No `NOT APPLICABLE` gate without a recorded reason | Governance |

---

## Tier 1 — Simulation

⚠️ **`unitree_mujoco` simulates the Go2 on the same DDS channels as the hardware,
CPU-only, no GPU required.**

### Tests

| Test | Verifies |
|---|---|
| Full task loop in simulation | End-to-end behaviour without hardware |
| Rejection of every invalid input | Validator completeness |
| Fault injection: server loss, sensor loss, battery low, corridor closure | Every abort path |
| Safety supervisor veto | Veto cannot be bypassed |
| Route planning with all closures | Never selects an unavailable edge |
| 🔴 **Route planning never selects stairs or lifts** | Graph edges are not permissions |
| Floor identity failure → `UNKNOWN` | Never guesses a floor |
| Relocation event invalidates pose | No stale navigation |
| State-machine exhaustive coverage | Every state reachable and abortable |
| 20-task synthetic campaign | Metrics collection works |

### Why this tier matters most

⚠️ **It validates the entire software architecture at zero risk and zero cost**,
before a motor turns. Every bug found here is a bug found without a robot in
jeopardy.

---

## Tier 2 — Bench tests (no robot movement)

| Test | Verifies | Gate |
|---|---|---|
| Read-only DDS connection, 30 min | Network, NIC binding, topics | 1 |
| Camera frame rate measurement | Resolves the 15 vs 30/60 fps contradiction | 1 |
| LiDAR point rate and size | SLAM input sizing | 1 |
| Battery telemetry field enumeration | Confirms `bms_state` | 1 |
| **Carrier tilt test at 45°** | Retention before robot contact | 0 |
| Dashboard functional test | Interface correctness | — |
| Failure-mode logging test | Diagnostics work | — |

---

## Tier 3 — Supervised robot tests

**All require Gate 8.** All require a named supervisor. All require the abort
procedure.

### Sequence — advance only on a clean pass

| # | Test | Pass criterion | Hold point |
|---|---|---|---|
| 1 | Stand / damp / balance transitions | No unexpected motion | H1 |
| 2 | Emergency stop | Stops within the stated time | **H2 — hard gate** |
| 3 | Remote authority takeover | Stops on remote input, every time | **H3 — hard gate** |
| 4 | Velocity at 0.1 m/s, tethered | Responds as commanded | |
| 5 | 10 m route, 10 consecutive runs | All complete | H4 |
| 6 | Obstacle injection at a known point | Stops or reroutes; latency measured | |
| 7 | Localisation loss injected | Detects, stops, recovers or aborts | |
| 8 | Battery consumption over a route | Reserve calculated from measurement | H5 |
| 9 | **Full task loop, 20 runs** | Completion rate recorded | H6 |
| 10 | Uncommanded motion check | **Zero** uncommanded movements | **H7 — hard gate** |
| 11 | Host-kill test | Robot stops within the watchdog interval | **H8 — hard gate** |
| 12 | Endurance, 2 h continuous | No thermal or mechanical fault | |

⚠️ **Hard gates H2, H3, H7, H8 may not be waived.** A failure at any of them stops
the project.

---

## Tier 4 — Pilot measurement

⚠️ **The protocol is written and agreed BEFORE data is collected.** Writing it
afterwards invites the accusation that the protocol was fitted to the results.

### Metrics

| Metric | Method | Trials |
|---|---|---|
| Task completion rate | Completed ÷ submitted | ≥30 |
| Delivery success rate | Correct destination, item, undamaged | ≥30 |
| Average, p95 task duration | Wall-clock | ≥30 |
| Navigation error | Fiducial pose vs robot pose at arrival | ≥30 |
| Obstacle-response latency | Video timestamp → stop-command timestamp | ≥20 |
| Localisation failure rate | Injected and natural | ≥30 |
| Human interventions | Count per run | ≥30 |
| Perception accuracy | Per angle and distance | ≥50 |
| Battery consumed per mission | Measured, not calculated | ≥5 full routes |
| **Failures with reasons** | Failure taxonomy | All |

### Statistics

- **Wilson confidence intervals** for proportions, not bare percentages
- 20/20 and 200/200 mean very different things
- Report the denominator, always

⚠️ **Report failures with the same prominence as successes.**

---

## Non-negotiable test rules

| # | Rule |
|---|---|
| 1 | ⚠️ **No test on hardware before Gates 0, 1 and 8 are passed** |
| 2 | ⚠️ **No test with students present before Gate 10** |
| 3 | ⚠️ **No stair or lift test** without every `ADR-003` condition met |
| 4 | **A named supervisor is present at every test** |
| 5 | **The abort procedure is known to everyone present before starting** |
| 6 | **Abort on any doubt, with no penalty** |
| 7 | **A test result is evidence only for the conditions under which it was run** |
| 8 | ⚠️ **A test pass on one staircase does not validate unsupervised operation** |

---

## What testing cannot establish

| Cannot | Why |
|---|---|
| That the robot is safe among students | Only a demonstrated, authorised pilot can suggest it |
| That teachers will use it | Behaviour, not engineering |
| That the lift provider will permit integration | Their decision |
| That the robot will work in every corridor | Every corridor must be surveyed |
| That performance will hold after a firmware update | Must be retested |

⚠️ **These boundaries should be stated to judges and to ourselves.** A test
programme that claims more than it can establish is worse than none.

---

## Test environment requirements

| Requirement | Reason |
|---|---|
| Out of student hours, unless Gate 10 | Student presence changes the risk profile |
| Cleared, physically secured space | Prevents uncontrolled interaction |
| Video recording from two angles | Enables latency measurement and incident review |
| Logging and telemetry active | Enables post-test analysis |
| Spare battery charged | Test interruption recovery |
| First-aid contact present | Required |
| Named supervisor with the remote | Load-bearing |

---

## Documentation

| From each test | Record |
|---|---|
| Preconditions met | Checklist |
| Parameters | Speed, payload, battery, riser height, lighting |
| Outcome | Pass / fail / abort |
| Timing | Start, end, duration |
| Interventions | Count and type |
| Anomalies | Full detail |
| Video | Retained per policy |
| Decision | Proceed, adjust, or stop |

⚠️ **An undocumented test is a test that did not happen.**