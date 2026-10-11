# ADR-011: Simulation before hardware

**Status:** `Accepted`
**Date:** 2026-10-10
**Decides:** In what order navigation, control and state-machine software is
developed
**Reversal condition:** Reversed only if simulation proves *unrepresentative* of
the real platform in a way that invalidates development — for example, if the
simulator's dynamics cannot reproduce the platform's locomotion envelope at all.
Even then, reversal buys hardware time, not hardware *operation*: it does not
authorise robot motion, and it does not pass any go/no-go gate.
**Evidence:** `docs/shana/feasibility/decision_log.md` `D-010` ·
`docs/shana/testing/future_pilot_plan.md` · `docs/integration/TASK_BOARD.md` `SH-B9`

---

## Context

SHANA has no robot access, no permissions, and no passed gates. The usual
instinct — and the one this project has been drifting toward — is to argue that
development cannot begin until hardware is available.

That argument fails. `unitree_mujoco` (BSD-3) simulates the Go2 **on the same
DDS channels as the hardware**, CPU-only, with no GPU requirement. The entire
software stack — navigation, the delivery state machine, the validator, corridor
management, battery policy — can be built, debugged and demonstrated against the
simulator before any permission exists.

---

## Decision

**Software is developed against the simulator first. Hardware is the last step,
not the first.**

| # | Rule |
|---|---|
| 1 | All navigation, control and state-machine work targets the simulator first |
| 2 | The stack must run **identically** against simulation and hardware — same DDS channels, same message contracts |
| 3 | 🔴 **Passing simulation is a precondition for requesting hardware access, never a substitute for it** |
| 4 | A go/no-go gate is passed by evidence. Simulation evidence passes no gate on its own (`AGENTS.md` §L.12) |

---

## Rationale

**Why this is unblocked.** Every other engineering task in this project is
blocked on a person, a permission, or a measurement. This one is blocked on
nothing. It needs no robot, no network, no school, and no student data. It is the
only route to producing a demonstrable artefact under current conditions.

**Why same-channel equivalence matters.** If the simulator used a different
interface from the hardware, every fix would have to be made twice and every
simulation result would be worth little. Sharing the DDS interface means the
software written in simulation is the software that would run.

**Why this is not a substitute for hardware validation.** 🔴 **Simulation proves
logic, not physics.** A simulator cannot tell you the robot will actually stop in
time, that the corridor is actually wide enough, or that the stair risers are
within limits. Every claim that requires physical evidence remains
`DOCUMENTED_NOT_HARDWARE_VERIFIED` no matter how convincing the simulation is.

**Why rule 3 exists.** The predictable failure is a simulation result being cited
as though it justified hardware operation. It does not. `AGENTS.md` §L is explicit
that technical feasibility and institutional authorisation are different gates.

---

## Evidence

| Fact | Class | Source |
|---|---|---|
| `unitree_mujoco` simulates Go2 on the same DDS channels, CPU-only | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | `decision_log.md` `D-010` |
| `unitree_mujoco` is BSD-3 licensed | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | same |
| Simulation-first is the accepted decision | `DESIGN_ONLY` | `decision_log.md` `D-010` |
| 🔴 **No simulation has been built or run** | `VERIFIED` | repository audit 2026-10-11 |
| `SH-B9` exists on the task board | `VERIFIED` | `docs/integration/TASK_BOARD.md` |

---

## Consequences

**Accepted**

- Engineering can begin immediately, with no permission and no school contact.
- Demonstrations become possible **as simulation**, correctly labelled as such.
- Hardware access requests become defensible: *"here is a working system against
  your simulator; we now need the real robot to validate physics."*
- Route-safety and state-machine logic can be tested exhaustively — something no
  amount of hardware time allows.

**Rejected**

- Waiting for hardware before writing any code.
- Developing against a hardware-only or hardware-coupled stack.
- Treating simulation success as gate passage.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Build on hardware when access arrives | Nothing can be built until a permission that may take a year. The project produces nothing meanwhile. |
| Develop against a different simulator | Breaks same-channel equivalence; every fix made twice; simulation results worth little. |
| Skip simulation, prototype on hardware | Requires hardware access *and* consumes the scarce access on debugging rather than validation. |
| Simulation only, never hardware | Rejected. Physics claims need physics. Simulation is the first step, not the last. |

---

## Follow-up tasks

| # | Task | Owner | Status |
|---|---|---|---|
| 1 | 🔴 Build the simulation PoC: simulated Go2, surveyed-format map, obstacle avoidance, human-in-the-loop stop, schema-conformant messages | Agent 1 | ⏳ **`NOT STARTED` — unblocked** |
| 2 | Exhaustive state-transition coverage of the delivery state machine | Agent 1 | ⛔ blocked on 1 |
| 3 | Demonstrate the full delivery journey in simulation, labelled as simulation | Agent 1 | ⛔ blocked on 1 |
| 4 | Record actual commands and output (none exist today) | Agent 1 | ⛔ blocked on 1 |

⚠️ **Task 1 is the highest-value engineering action available in this project and
it is not blocked.** Everything else waits on people.

---

## Related

- [`../../testing/future_pilot_plan.md`](../../testing/future_pilot_plan.md)
- [`../../testing/test_inventory.md`](../../testing/test_inventory.md)
- [`../../handoff/agent_handoff.md`](../../handoff/agent_handoff.md) §6
- [`../../feasibility/decision_log.md`](../../feasibility/decision_log.md) — `D-010`
- `docs/integration/TASK_BOARD.md` — `SH-B9`
- `AGENTS.md` §L.12 — 🔴 *"A gate is never PASSED without evidence attached."*