# SHANA — Delivery State Machine

**Status: PROPOSED. Not implemented. No state has been entered.**

---

## Design principle

> **A delivery is a deterministic sequence of states with a defined abort path
> from every one of them.**

There is no branching on model output. There is no partial execution. A task
either completes through a defined path or is recorded as failed with a reason.

---

## States

```
                      ┌──────────┐
                      │  QUEUED  │
                      └────┬─────┘
                           │ validator: accept
                           ▼
                     ┌──────────┐
              ┌─────►│VALIDATING│
              │      └────┬─────┘
              │           │ validator: reject
              │           ▼
              │      ┌──────────┐
              │      │ CANCELLED│◄────────────────────┐
              │      └──────────┘                     │
              │                                        │
              │ validator: accept                      │
              ▼                                        │
        ┌──────────┐                                   │
        │   READY  │  all preconditions pass            │
        └────┬─────┘                                   │
             │                                         │
             ├──────────────┐                          │
             ▼              │                          │
    ┌────────────────┐      │                          │
    │NAVIGATING_TO_  │      │ robot not ready           │
    │    PICKUP      │      │ battery insufficient      │
    └───────┬────────┘      │ corridor unavailable      │
            │               │ segment closed            │
            │               └───────────────────────────┘
            ▼
   ┌─────────────────────┐
   │AWAITING_LOADING_   │  robot holds. Payload secured.
   │    CONFIRMATION    │  Battery still monitored.
   └───────┬─────────────┘
           │ human confirms loaded
           ▼
   ┌─────────────────────┐
   │NAVIGATING_TO_      │
   │    DESTINATION     │
   └───────┬─────────────┘
           │
           ▼
   ┌─────────────────────┐
   │ AWAITING_RECIPIENT │  robot holds at destination
   └───────┬─────────────┘
           │ explicit recipient confirmation
           ▼
     ┌───────────┐
     │ COMPLETED │
     └───────────┘

   Any state ──► PAUSED ──► resume to prior state, or CANCELLED
   Any state ──► FAILED   (with a recorded reason)
```

---

## State definitions

| State | Meaning | Robot does | Who can exit |
|---|---|---|---|
| `QUEUED` | Accepted, waiting for a slot | Idle at checkpoint | Scheduler |
| `VALIDATING` | Under deterministic checks | Idle at checkpoint | Validator only |
| `READY` | All preconditions pass | Idle at checkpoint | Executor |
| `NAVIGATING_TO_PICKUP` | Moving to origin | **Moving** | Executor |
| `AWAITING_LOADING_CONFIRMATION` | Waiting for a person to load | Holding position | Human only |
| `NAVIGATING_TO_DESTINATION` | Moving to destination | **Moving** | Executor |
| `AWAITING_RECIPIENT` | Arrived, waiting for confirmation | Holding position | **Human only** |
| `COMPLETED` | Recipient confirmed receipt | Idle or returning | — terminal |
| `PAUSED` | Halted by an operator | Holding position | Operator |
| `CANCELLED` | Withdrawn before completion | Idle or returning | — terminal |
| `FAILED` | Could not proceed | Holding position | — terminal |

⚠️ **Only two states advance without a human action, and neither reaches a
destination.** Every delivery requires at least two human confirmations — loading
and receipt.

---

## Invariants

These hold in every state. They are testable.

| # | Invariant | Rationale |
|---|---|---|
| I-01 | The robot **only moves** in the two `NAVIGATING_*` states | Minimises moving states |
| I-02 | `COMPLETED` requires explicit human confirmation | Never inferred from arrival |
| I-03 | A task enters `READY` only after every precondition passes | No premature motion |
| I-04 | The validator alone may move `QUEUED` → `VALIDATING` → `READY` or `CANCELLED` | Deterministic gate |
| I-05 | No state transition is caused by model output | AI cannot act |
| I-06 | Every state has a defined abort path | No dead ends |
| I-07 | Every transition is logged with timestamp and reason | Full auditability |
| I-08 | The safety supervisor may abort any state at any time | Independent veto |
| I-09 | In any non-moving state the robot **holds position** | No wandering |
| I-10 | Battery and connectivity are monitored in every state | No blind states |
| I-11 | `FAILED` requires a recorded reason | Learn from failures |
| I-12 | A destination is only ever selected from the verified allow-list | No unverified goals |

---

## Entry conditions for `READY`

**All must be true.** Any failure sends the task to `CANCELLED` with a specific
reason.

| # | Condition | Failure reason shown to the teacher |
|---|---|---|
| E-01 | Robot model, firmware and health state acceptable | "Robot is not ready" |
| E-02 | Robot state is `IDLE`, not mid-task | "Robot is busy" |
| E-03 | Battery above the reserve threshold | "Not enough battery to complete and return" |
| E-04 | Return route feasible and verified | "Return route is not available" |
| E-05 | Origin reachable via `AVAILABLE` segments | "No available route to pickup" |
| E-06 | Destination reachable via `AVAILABLE` segments | "No available route to destination" |
| E-07 | Destination in the verified allow-list | "Destination is not approved" |
| E-08 | Origin in the verified allow-list | "Pickup point is not approved" |
| E-09 | Item in the approved list and within weight limit | "Item is not permitted" or "Item too heavy" |
| E-10 | Within operating window | "Outside operating hours" |
| E-11 | Enough time to complete before the next cutoff | "Cannot complete before the shutdown cutoff" |
| E-12 | No corridor in the route is `HAZARD` or `CLOSED` | "Route includes a closed segment" |
| E-13 | Human supervisor present and in position | "No supervisor on station" |
| E-14 | Destination not already being served by another task | "Destination is already being served" |
| E-15 | Confirmation obtained if the task was model-derived | "Awaiting human confirmation" |

⚠️ **E-15 is where the AI boundary is enforced.** A task derived from natural
language cannot enter `READY` without a human confirming the structured
interpretation.

---

## Abort conditions

Any of these moves the task to `FAILED` with a recorded reason:

| Trigger | Source |
|---|---|
| Loss of communication with the server | Robot or server |
| Loss of localisation | Navigation |
| Map mismatch or floor identity unknown | Floor identification |
| Battery below the recovery threshold | Battery guard |
| Corridor segment becomes unavailable mid-route | Corridor state |
| Sensor failure on the safety-relevant path | Sensor monitor |
| Robot fault flag set | Telemetry |
| Safety supervisor veto | Safety supervisor |
| Physical emergency stop | Human |
| Deadline exceeded | Scheduler |
| Payload dropped or shifted | Cargo monitor |

⚠️ **Abort means stop and hold.** The robot does not improvise. If it can safely
return to a checkpoint on a verified available route, it may; otherwise it holds
and requests human assistance.

---

## Pause and cancel semantics

| Action | Who | Effect |
|---|---|---|
| Pause | Authorised operator | Robot holds. Task state preserved. |
| Resume | Authorised operator | Returns to the paused state **if** all conditions still hold |
| Cancel | Teacher, admin, or custodian | Task ends. Robot returns if it can do so safely. |

⚠️ **Cancel does not override a physical emergency stop.** Once the physical stop
is engaged, only a human can release it.

⚠️ **A task may not be cancelled to `COMPLETED` under any circumstances.**

---

## Worked example — the full path

**Request:** *"Take the physics notebooks to the physics lab."*

```
1.  Teacher selects PHYSICS_NOTES and PHYSICS_LAB from the allow-list
2.  → QUEUED            Logged, timestamped, task number issued
3.  → VALIDATING        E-01..E-15 checked
     ├─ E-03 battery 34% ≥ 30% threshold        ✓ pass
     ├─ E-06 route C3 corridor state AVAILABLE  ✓ pass
     ├─ E-07 destination verified               ✓ pass
     └─ all others pass
4.  → READY             Supervisor confirmed present
5.  → NAVIGATING_TO_PICKUP   Robot moves along verified route
     └─ mid-route: corridor marked BUSY by staff
        → alternate segment checked → CLOSED
        → no available route
        → ABORT → FAILED  reason: "Route became unavailable"
        → Robot holds → supervisor notified → teacher notified
6.  Alternative: staff clears the corridor before the task starts
        → E-06 re-checked → AVAILABLE
        → continues
7.  → AWAITING_LOADING_CONFIRMATION   Human places item, confirms
8.  → NAVIGATING_TO_DESTINATION      Robot moves
9.  → AWAITING_RECIPIENT             Robot holds at door
     └─ 10 min pass, no recipient
        → reminder sent
        └─ no response within limit
        → FAILED  reason: "No recipient confirmation"
        ⚠️ NOT marked complete. Arrival is not completion.
10. → Human confirms receipt
     → COMPLETED   Duration, battery delta, interventions, failures logged
```

---

## 🔴 State → published message mapping

**This machine is the single source of truth for delivery status.** The mapping
below is authoritative and versioned with the machine. It is published in
[`cloudlight_integration.md`](cloudlight_integration.md) §3 and mirrored in
`docs/integration/interface_contracts.md`.

⚠️ **Publishes every state. No state is internal-only.** `PAUSED` is published
because a paused robot is a physical fact a teacher must be able to see.

| State | Published message | Terminal | Human action required |
|---|---|---|---|
| `QUEUED` | `status.assigned` | — | None |
| `VALIDATING` | `status.assigned` (unchanged) | — | None |
| `READY` | `status.assigned` (unchanged) | — | None |
| `NAVIGATING_TO_PICKUP` | `status.en_route` | — | None — **robot is moving** |
| `AWAITING_LOADING_CONFIRMATION` | `status.awaiting_loading` | — | 🔴 **Yes — a person must load** |
| `NAVIGATING_TO_DESTINATION` | `status.en_route` | — | None — **robot is moving** |
| `AWAITING_RECIPIENT` | `status.awaiting_recipient` | — | 🔴 **Yes — go to the robot now** |
| `PAUSED` | `status.paused` | — | ⚠️ **Yes — operator action** |
| `COMPLETED` | `status.delivered` | ✅ | None — already done |
| `CANCELLED` | `status.cancelled` | ✅ | None |
| `FAILED` | `status.error` | ✅ | None |

### Messages that are not state transitions

| Message | When | Note |
|---|---|---|
| `status.ready` | Robot idle, no task | Subsystem availability, **not** a delivery state |
| `status.blocked` | Corridor blocked, robot holding | Overlay on the current state; never replaces it |
| `status.estop` | Emergency stop engaged | 🔴 **Out-of-band.** Halts the current task → `FAILED` or `PAUSED` |

### 🔴 Rules governing this mapping

| # | Rule | Rationale |
|---|---|---|
| M-1 | 🔴 **`AWAITING_RECIPIENT` and `NAVIGATING_TO_DESTINATION` must never collapse into one display value** | One means *"wait"*; the other means *"go to the robot now"*. Merging tells a teacher the opposite of the truth |
| M-2 | 🔴 **`PAUSED` must appear in any consumer's display model** | A paused robot shown as "on the way" sends a person to an empty corridor |
| M-3 | 🔴 **`status.estop` is out-of-band and takes precedence over every state** | A stop is not a delivery status |
| M-4 | **`status.ready` is never a delivery state** | It describes the robot, not the task |
| M-5 | **Validation rejection routes to `CANCELLED`, not a `REJECTED` state** | See below |
| M-6 | Every transition is published with a timestamp and the **state it came from** | No status is ever presented without its age |

### ⚠️ On `REJECTED`

SHANA has no `REJECTED` state. A validator rejection is a `CANCELLED` transition
carrying `failure_reason`, e.g. `"Destination is not approved"` (E-07).

⚠️ **A consumer that needs to distinguish *rejected* from *withdrawn* must read
`failure_reason`, not invent a state SHANA cannot emit.**

### Known defect

⚠️ **`status.awaiting_loading` and `status.awaiting_recipient` do not yet appear in
`cloudlight_integration.md` §3**, which lists only eight messages. This table is
the correction. Until the interface contract is agreed
([`CONF-02` / `CONF-08`](../../integration/unresolved_conflicts.md)), the two
messages above are **`Proposed` additions**.

---

A test suite should demonstrate:

| Property | Test |
|---|---|
| `COMPLETED` unreachable without human input | Attempt to force `COMPLETED`; must fail |
| Robot never moves outside `NAVIGATING_*` | Assert on every logged transition |
| Validator is deterministic | Same input, N repetitions, identical output |
| Every state has a reachable abort path | Exhaustive state-transition coverage |
| Model output cannot advance a task | Inject model output as the transition source; must be rejected |
| Battery check precedes `READY` | Submit at low battery; must be rejected |
| Closed segment blocks the route | Close the only segment; must be rejected |
| Human can abort from any state | Exercise every state |
| Every transition is logged | Audit-log completeness check |

⚠️ **These are the acceptance tests in `docs/shana/testing/acceptance_criteria.md`. They
are not implemented and cannot be run until the gates are passed.**