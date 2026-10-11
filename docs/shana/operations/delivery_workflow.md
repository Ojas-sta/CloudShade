# SHANA — Delivery Workflow

**Status: PROPOSED. Not implemented. No delivery has been requested or made.**

---

## The end-to-end journey

```
TEACHER                      SYSTEM                        ROBOT
───────                      ──────                        ─────

Select origin ──────────────▶ Validate
Select destination ─────────▶  ├─ schema valid?
Select item ─────────────────▶  ├─ in allow-list?
Press Submit ───────────────▶  ├─ robot READY?
                             │  ├─ battery sufficient?
                             │  └─ route available?
                             │
                             ├─ REJECT ──▶ reason shown ──▶ (nothing happens)
                             │
                             └─ ACCEPT ──▶ task number, ETA
                                                  │
                                        Readiness re-check
                                                  │
                                    Navigate to pickup ─────────────▶ moves
                                                  │◀──────────────── telemetry
                                    Hold, await loading
                             Human places item, confirms
                                                  │
                            Navigate to destination ─────────────────▶ moves
                                                  │◀──────────────── telemetry
                                    Hold at destination
                             Human confirms receipt
                             Log, notify, return ─────────────────────▶ returns
                             COMPLETE
```

---

## Step by step

### 1. Request

The teacher selects from **dropdown lists only**:

| Field | Source | Free text? |
|---|---|---|
| Origin | Verified checkpoint or approved pickup point | ⛔ **No** |
| Destination | Verified allow-list | ⛔ **No** |
| Item | Approved item registry | ⛔ **No** |

⚠️ **Free text destinations are never accepted.** The robot cannot verify a room
name typed by hand, and a mistyped destination is a physical-safety failure.

### 2. Validation

The validator checks fifteen conditions and either accepts or rejects with a
**specific reason**. Full list: `docs/shana/architecture/task_state_machine.md`.

⚠️ **Rejection reasons are always specific.** "Request failed" is a bug.

Examples of acceptable messages:
- *"Robins not ready — currently in maintenance"*
- *"Not enough battery to deliver and return safely"*
- *"Lab 2B is not approved for autonomous delivery"*
- *"No available route — the east corridor is closed"*
- *"Cannot finish before the 13:15 lunch cutoff"*

### 3. Acceptance

The teacher receives a task number, an estimated completion time, and a status.
The task enters the queue.

### 4. Readiness re-check

⚠️ **Conditions are re-checked at the moment of execution**, not only at request.
The battery may have fallen; a corridor may have closed.

If a condition now fails, the task is rejected with the same specific reasons.

### 5. Pickup navigation

The robot navigates the pre-verified route. The route planner:
- Excludes any segment not `AVAILABLE`
- Excludes any segment not authorised for robot access
- ⚠️ Excludes stairs and lifts regardless of graph edges
- Refuses if no path exists

### 6. Loading

The robot **holds position**. A person places the item and confirms loading.

⚠️ **The robot does not verify the load.** It cannot see it, and it cannot pick
it up. The human confirms.

### 7. Destination navigation

As step 5. The robot holds on arrival.

### 8. Receipt

⚠️ **A delivery is not complete until a person confirms they have received it.**

| If no recipient | Action |
|---|---|
| Initially | Robot holds; reminder sent |
| After a timeout | Reminder escalates to supervisor |
| After a longer timeout | Task `FAILED` — reason *"No recipient confirmation"* |

⚠️ **Arrival is never completion.** If the robot arrives and the item is stolen,
the system must not report success.

### 9. Completion

Logged with: task number, origin, destination, item, duration, battery delta,
interventions, obstacles encountered, any failure.

Both teacher and recipient are notified.

---

## Task states

`QUEUED` → `VALIDATING` → `READY` → `NAVIGATING_TO_PICKUP` →
`AWAITING_LOADING_CONFIRMATION` → `NAVIGATING_TO_DESTINATION` →
`AWAITING_RECIPIENT` → `COMPLETED`

With `PAUSED`, `CANCELLED`, `FAILED` reachable from any state.

Full specification: `docs/shana/architecture/task_state_machine.md`.

---

## Roles

| Role | May | May not |
|---|---|---|
| **Teacher** | Create request; cancel own; confirm receipt; give feedback | Command motion; bypass validation; add destinations |
| **Room admin** | All teacher actions; manage block destinations | Command motion |
| **Robot custodian** | All above; physical handling; record relocation events | Override safety supervisor |
| **Supervisor** | Pause; cancel; abort; **stop at any time** | — |
| **System admin** | Manage users, allow-lists, schedules | Override safety supervisor |

⚠️ **No role has motor control.** Motion commands originate only from the state
machine, and only after validation and safety approval.

---

## Controls

| Control | Behaviour |
|---|---|
| **Duplicate prevention** | Same origin, destination, item, and recipient within a short window → rejected |
| **Rate limiting** | Per-teacher and per-system limits to protect the queue and the robot |
| **Destination conflict** | One active delivery per destination |
| **Cancellation** | Any stage, by teacher, admin, or custodian |
| **Pause** | Holds the task; resumable if conditions still hold |
| **Duplicate confirmation** | A second receipt confirmation is recorded but ignored |

---

## Notifications

| Event | To | Method |
|---|---|---|
| Accepted | Requesting teacher | Dashboard |
| Rejected with reason | Requesting teacher | Dashboard |
| Task started | Teacher, supervisor | Dashboard |
| Waiting for loading | Supervisor | Dashboard + audible |
| Arrived | Recipient | Dashboard + audible |
| Receipt confirmed | Teacher, recipient | Dashboard |
| Delayed | Teacher | Dashboard |
| Failed with reason | Teacher, supervisor | Dashboard + audible |
| Recovery needed | Supervisor | Dashboard + audible + escalation |

⚠️ **Notifications always carry a reason.** A notification without a reason forces
the recipient to investigate, which is worse than no notification.

---

## Data recorded per delivery

| Field | Sensitivity |
|---|---|
| Task ID, timestamps, durations | Low |
| Origin and destination IDs | ⚠️ **Medium — reveals layout** |
| Item ID and weight | Low |
| Battery start and end | Low |
| Human interventions | Low |
| Obstacles encountered | Low |
| Failure reason if any | Low |
| ⚠️ **Never** | Student data, identity, or behaviour |

---

## Questions

| ID | Question | Priority |
|---|---|---|
| `SH-DEL-001` | The single most valuable task to delegate? | **P0** |
| `SH-DEL-002` | **Absolute item prohibitions** | **P0** |
| `SH-DEL-003` | Maximum payload weight and container size | **P0** |
| `SH-DEL-006` | How will teachers submit requests? | P1 |
| `SH-DEL-009` | **Confirm completion never inferred from arrival** | **P0** |
| `SH-DEL-010` | Who may confirm receipt? | P1 |
| `SH-DEL-015` | Who may cancel, and when? | P1 |
| `SH-DEL-017` | How many teachers, how often? | P1 |
| `SH-DEL-020` | **Will teachers accept the robot in their corridor?** | **P0** |
| `SH-DEL-022` | **Confirm teachers have no motor control** | **P0** |

---

⚠️ **No delivery has been requested, validated, scheduled, or executed. This
document describes a design that does not exist.**