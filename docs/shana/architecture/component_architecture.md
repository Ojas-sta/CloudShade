# SHANA — Component Architecture

**Status: PROPOSED. Nothing described here has been built.**

---

## The architectural rule that shapes everything

> **The AI layer may interpret. The validator decides. Only a deterministic state
> machine may execute. A human may always stop.**

Everything below follows from that rule.

---

## Layers

### Layer 1 — User interface 🔵 ours

| Component | Responsibility |
|---|---|
| Teacher dashboard | Submit a request from an allow-list; view status; confirm receipt |
| Admin console | Manage destinations, items, corridors, users, schedules |
| Safety console | Stop, pause, cancel, view faults |
| Notifications | Arrival, failure, fault, recovery requests |

**Constraints:** no motor control from any teacher-facing screen. Destinations
selected from an allow-list, never typed free text.

### Layer 2 — Request and task management 🔵 ours

| Component | Responsibility |
|---|---|
| Request intake | Receive, normalise, reject malformed input |
| Task validator | **Pure function.** Check every rule. Refuse anything unrecognised |
| Task queue | Order accepted tasks; respect deadlines and battery reserve |
| Scheduler | Decide what runs next and when |
| Notification router | Dispatch events to the right recipients |

⚠️ **The validator is the security boundary.** It has no network access, no
clock dependency, and no ability to reach the robot. Same input, same output,
every time.

### Layer 3 — Execution 🔵 ours

| Component | Responsibility |
|---|---|
| Delivery state machine | The only thing that may advance a task |
| Route planner | Choose a path over the verified route graph |
| Corridor state store | Track availability, occupancy, hazards, closures |
| Checkpoint manager | Track checkpoints, recovery feasibility, battery reserve |

**Constraints:** no branching on model output. Every transition has a defined
abort path. Every failure is a clean stop, never a partial action.

### Layer 4 — Safety supervision 🔵 ours

| Component | Responsibility |
|---|---|
| Command validator | Final check before any motion command leaves the process |
| Speed limiter | Clamp every velocity to the approved maximum |
| Heartbeat manager | Prove liveness; trigger safe stop on absence |
| Battery guard | Refuse tasks and trigger recovery below threshold |
| Zone guard | Refuse commands outside the approved area |
| Kill switch | Stop everything, from dashboard, remote, or physical method |

⚠️ **Independent of the execution layer.** A bug in the state machine must not be
able to bypass the safety supervisor. Both must fail to *stop* only if the
physical supervisor also fails.

### Layer 5 — Robot interface 🔵 ours wrapping ✅ Unitree

| Component | Responsibility | Unitree interface |
|---|---|---|
| Motion adapter | Velocity and gait commands | `SportClient.Move(vx,vy,vyaw)`, `StopMove()`, `Damp()`, `StandUp()`, `StandDown()` ✅ |
| Avoidance adapter | Commands that route through obstacle avoidance | `ObstaclesAvoidClient.Move`, `MoveToAbsolutePosition` ✅ |
| State monitor | Battery, position, faults, gait | `rt/lowstate` → `LowState.bms_state` ✅ · `rt/sportmodestate` ✅ |
| Perception adapter | LiDAR, height map, obstacle distances | `rt/utlidar/cloud_deskewed`, `height_map_array`, `range_info` ✅ |
| Marker reader | Floor identity, checkpoint confirmation | Robot camera + ArUco 🔵 |

⚠️ **The adapter is the only place Unitree-specific code may appear.** Everything
above it is platform-independent.

### Layer 6 — Environment inputs 🔵 ours consuming external data

| Source | Provides | Owner |
|---|---|---|
| Corridor state | Closure, hazard, occupancy | Facilities, staff, sensors |
| Schedule | Timetable, lunch cutoff, operating windows | Administration |
| Item registry | Approved items, weights, containers | Administration |
| Destination registry | Approved locations, approach poses, checkpoints | Administration + Facilities |

⚠️ **Every environment input is treated as untrusted and potentially stale.**
Staleness has a defined expiry, and expired input is treated as `UNKNOWN`.

---

## The trust boundary in detail

```
┌──────────────────────────────────────────────────────────────┐
│  UNTRUSTED                                                   │
│                                                              │
│   Teacher input        External sensors      Model output     │
│        │                     │                    │          │
│        └─────────────┬───────┴────────────┬───────┘          │
│                      ▼                    ▼                   │
│              ┌───────────────────────────────────┐           │
│              │  TASK VALIDATOR  (pure function) │           │
│              │                                   │           │
│              │  · schema valid?                  │           │
│              │  · destination ∈ allow-list?      │           │
│              │  · item ∈ allow-list, weight ok?   │           │
│              │  · corridor segment available?    │           │
│              │  · battery ≥ reserve?             │           │
│              │  · robot state READY?             │           │
│              │  · human confirmation present?    │           │
│              │                                   │           │
│              │  NO NETWORK ACCESS. NO I/O.       │           │
│              │  REJECT → explain → never execute │           │
│              └────────────────┬──────────────────┘           │
│                               │ validated task               │
└───────────────────────────────┼──────────────────────────────┘
                                ▼
┌──────────────────────────────────────────────────────────────┐
│  TRUSTED                                                     │
│                                                              │
│   State machine ──► Route planner ──► Safety supervisor ──┐   │
│                                             (veto)        │   │
│                                                          ▼   │
│                                              Robot adapter   │
└──────────────────────────────────────────────────────────────┘
```

### The five testable properties

1. **The model cannot reach the robot.** No network route, no tool access,
   verifiable by inspection.
2. **The validator is pure.** No I/O, no clock, no network. Deterministic.
3. **Unknown entities are rejected, never guessed.**
4. **Every task requires human confirmation** — particularly model-derived ones.
5. **The safety supervisor can veto anything**, including a fully valid task.

⚠️ **These are testable claims. They should be demonstrated, not asserted** —
see `docs/shana/testing/acceptance_criteria.md`.

---

## What runs where

| On the robot (closed, on-board) | On the local server (ours) |
|---|---|
| Gait generation, balance | Everything else |
| Obstacle avoidance | Task management and validation |
| LiDAR driver and point-cloud publication | Navigation and perception |
| Motion services | Safety supervision |
| Battery telemetry read-out | Dashboard, authentication, storage |
| | Logging, metrics, notifications |

**Rationale:** the state machine, validator, and safety supervisor live on the
server so they are testable, inspectable, and durable. If the server dies, the
robot stops — which is the correct behaviour.

⚠️ **Consequence: there is no software watchdog if the server dies.** The
backstops are the robot's built-in obstacle avoidance and the physical remote
controller. This is why the human supervisor is not optional.

---

## Failure isolation

| Failure | Containment | Residual risk |
|---|---|---|
| Model returns nonsense | Schema validation rejects | None |
| Validator bug | Reviewed, deterministic, testable | Execution blocked — safe direction |
| State-machine bug | Safety supervisor vetoes | Execution blocked — safe direction |
| Server crash | Robot holds; physical supervisor | ⚠️ No software watchdog |
| Network loss | Robot holds; marks orphaned | Requires human retrieval |
| Sensor failure | Treat as `UNKNOWN`, never clear | Conservative, may over-refuse |
| Corridor closure | Route replans or stops | Conservative, may over-refuse |
| Battery low | Refuse tasks; recovery route | Recovery may be impossible |
| Stale data | Expiry → `UNKNOWN` | Conservative |
| Human error | Spotters, training, authority | ⚠️ Inherent — mitigated not eliminated |

⚠️ **Every automatic failure mode degrades toward "stop and ask a human."** That
is the design principle. The failure mode we cannot design away is human error,
which is why the human has authority to stop.