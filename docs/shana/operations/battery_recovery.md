# SHANA — Battery Recovery and Human Intervention

**Status: PROPOSED. No recovery has been planned, rehearsed, or performed.**

---

## The verified constraint that shapes everything

⛔ **There is no autonomous return-to-dock capability.**

The documented Go2 service list contains no charging or dock service. A charging
pile exists for X and EDU models, and an *"Aruco Recharge Service Interface"*
documentation page exists — but its body could not be retrieved, and no
autonomous return-to-dock method is documented.

> **Recovery is human-assisted by design.** Architect it. Do not assume otherwise.

---

## Battery thresholds — TBD

⚠️ **No numeric threshold is stated here.** All are **TBD** until measured
(`SH-BAT-006`, `SH-BAT-008`).

| Threshold | Value | Basis |
|---|---|---|
| Abort — stop accepting tasks | **TBD** | Must be measured, not guessed |
| Emergency — hold and request help | **TBD** | |
| Robot's own crouch | ⚠️ **10 minutes** at low battery | ✅ VERIFIED |
| Manual retrieval floor | **TBD** | |

⚠️ **Manufacturer runtime figures span 1–5 hours across four official
documents.** Planning uses ~90 minutes of active mission per charge as an
*engineering estimate*, explicitly not a specification.

---

## The reserve rule

```
BEFORE STARTING A TASK
  1. Read state of charge, and coulomb-counted consumption if available
  2. Can the task be completed on the remaining charge?      → else reject
  3. Can the robot RETURN on the remaining charge after?     → else reject
  4. Is the nearest safe checkpoint reachable and safe?      → else reject

AT THE ABORT THRESHOLD
  5. Stop accepting new tasks
  6. Return to a verified, reachable, safe checkpoint — if the route is available
  7. If return is NOT safely feasible → hold position, request human assistance
```

⚠️ **A low battery must never cause the robot to take an unverified stair route,
enter a lift without approval, or cross a closed or hazardous corridor.**

---

## Checkpoint selection

⚠️ **"Nearest" is not the criterion.** The checkpoint is selected on:

| Criterion | Requirement |
|---|---|
| Reachability | Route is `AVAILABLE` and verified |
| Safety | No hazard, closure, or `UNKNOWN` segment |
| Authority | Approved checkpoint |
| Floor transition | ⛔ No unapproved stairs or lifts |
| Charge on arrival | Must be achievable within reserve |
| Supervision | A person must be available to receive it |

If no checkpoint satisfies all criteria → **hold and request assistance.**

---

## Recovery workflow

```
1. Robot holds position. It does NOT wander.
2. Dashboard shows:
     · last known position
     · ⚠️ position AGE, prominently
     · battery, state, fault, time since last contact
3. Supervisor notified with location and reason
4. Supervisor retrieves the robot by the agreed manual method
5. Robot returned to an approved checkpoint
6. Battery charged or swapped per procedure
7. Fault logged; cause reviewed before the next mission
```

⚠️ **If the robot is completely unreachable**, the search begins from the last
**confirmed checkpoint** — labelled as *possibly stale*. The robot may be
anywhere.

---

## Relocation events

⚠️ **Any manual handling of the robot invalidates its pose estimate.**

| Event | Effect |
|---|---|
| Lifted or carried | `pose_valid = false` |
| Repositioned within a floor | `pose_valid = false` |
| Battery removed for swap | `pose_valid = false` |
| Transported anywhere | `pose_valid = false` |

While `pose_valid` is false:
- ⛔ No new task accepted
- Must re-localise before resuming
- Dashboard shows the state prominently
- **The console must make recording a relocation unmissable**

⚠️ **An unrecorded relocation is the most likely way the robot ends up somewhere
it believes it is not.**

⚠️ **A checkpoint's last-known position is not necessarily the robot's current
position.** This is a stated design principle, not an incidental caution.

---

## Human intervention — the escalation path

| Condition | Response time | Escalation |
|---|---|---|
| Robot waits at a checkpoint | Immediate, visual | Supervisor |
| Robot holds on a route | Immediate | Supervisor |
| Recovery required | Immediate, audible | Supervisor → administrator |
| Robot unreachable | Immediate | Supervisor → facilities → administrator |
| Fault on a motor | Immediate | Supervisor → custodian → **stop operations** |
| Repeated faults | — | ⛔ **All operations stop** pending review |
| Injury | Immediate | First aid → administration → **stop the day** |
| Unauthorised interference | Immediate | Supervisor → administration |
| Loss of permission | Immediate | ⛔ **All operations stop** |

⚠️ **The response time expectation must be agreed with the school.** If no one can
attend within a defined time, stopping is not a safe behaviour — because the robot
sits unattended where it stopped.

⚠️ **This is a genuine design dependency, not an administrative detail.** A robot
that safely stops in an unsupervised corridor for an hour is a different
situation from one attended within two minutes.

---

## Human-in-the-loop requirements

| Requirement | Status |
|---|---|
| Named supervisor on every operation | ⚠️ Not established |
| Unilateral stop authority | ⚠️ Not agreed |
| At least two trained people | ⚠️ Not arranged |
| Defined manual recovery method | ⚠️ Not written |
| Response-time expectation agreed | ⚠️ Not agreed |
| Escalation contacts named | ⚠️ Not named |
| Robot retrieval procedure | ⚠️ Not written |

Questions: `SH-SAFE-005`, `-006`, `-007`, `SH-SAFE-023`, `SH-BAT-013`, `-014`

---

## Charging

⚠️ **Full procedure and limits: `docs/shana/hardware/battery_and_charging.md`.**

Key constraints:
- ⚠️ **Discharge −20 to 60 °C; charge 0 to 55 °C.** Exceeding these *"may cause
  the battery to ignite or even explode."*
- ⚠️ Cool to room temperature before charging
- ⚠️ Non-flammable surface, **supervised**, never unattended overnight in an
  occupied building
- ⚠️ Stop use below 10%; quarantine any swollen or damaged pack
- ⚠️ Swap is a **power-off, lay-it-down** procedure — no documented hot-swap
- ⚠️ **One battery ships in the box.** Spares are a procurement decision

---

## The honest position

> **The battery subsystem is the least verified part of the project, and it
> governs whether missions are possible at all.**

⚠️ **We do not know which battery is fitted. We do not know the real endurance
under load. We have no measured threshold.** Every number currently in use is a
manufacturer figure — several of which contradict each other — or an engineering
estimate.

`SH-GO2-006` — read the battery label — is the cheapest high-value question in
the entire questionnaire.