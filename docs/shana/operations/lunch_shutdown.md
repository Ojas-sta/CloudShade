# SHANA — Lunch and Out-of-Hours Shutdown

**Status: PROPOSED. No schedule has been obtained. No shutdown workflow exists.**

---

## Why the lunch shutdown exists

At lunch, corridors fill with students moving between areas. The robot's safety
case assumes reduced traffic and an enforced exclusion zone. **Lunch is neither.**

The shutdown is therefore not an optimisation — it is a **standing safety
restriction**: the robot does not operate in a high-traffic period it was not
designed or tested for.

---

## The workflow

```
1.  Calculate the cutoff from the APPROVED schedule
        cutoff = lunch_start − required_return_lead_time
        ⚠️ Lead time is TBD — it must be measured, not estimated

2.  At cutoff − task_acceptance_window:
        STOP accepting new tasks
        ⚠️ Reject with a specific reason and suggest the next available window

3.  Complete or abort the active task per approved policy
        ⚠️ Never rush a task because a cutoff is approaching

4.  Return to an approved checkpoint on a VERIFIED, AVAILABLE route
        ⚠️ If no safe route exists → hold and request human assistance

5.  Confirm arrival and a safe state
        · Position verified
        · Battery at or above threshold
        · No fault flag

6.  Disable new deliveries
        ⚠️ Requires an AUTHORISED PERSON to resume — never automatic

7.  Per school policy: power down, or remain standing, or be secured
        ⚠️ TBD — see below
```

---

## Cutoff policy questions

| ID | Question | Status |
|---|---|---|
| `SH-TIME-001` | Official timetable, and who maintains it? | ❓ Unknown |
| `SH-TIME-002` | Does it change on examination days or special events? | ❓ Unknown |
| `SH-TIME-003` | Is it available in machine-readable form? | ❓ Unknown |
| `SH-TIME-005` | **Exact lunch start and end times, including staggering** | ❓ Unknown |
| `SH-TIME-006` | **Required return lead time** | ⚠️ **TBD — must be measured** |
| `SH-TIME-007` | Should a delivery be rejected when it cannot finish before cutoff? | ❓ Open |
| `SH-TIME-008` | What happens to a delivery in progress at cutoff? | ❓ Open |
| `SH-TIME-009` | What if the robot cannot return before cutoff? | ❓ Open |
| `SH-TIME-013` | Is there a manual disable control, and who operates it? | ❓ Open |
| `SH-TIME-014` | **Who may resume operations?** | ❓ Open |

---

## The security question — do not assume

⚠️ **A robot at a checkpoint is not automatically secure.**

| Question | Status |
|---|---|
| Is the checkpoint area access-controlled? | ❓ `SH-MAP-024` |
| How is student access prevented? | ❓ `SH-TIME-012` |
| Must the robot be powered down? | ❓ `SH-TIME-011` |
| Must it be attended? | ❓ `SH-TIME-011` |
| Must it be physically secured? | ❓ `SH-MAP-024` |

⚠️ **The answer changes the design.** A checkpoint in an open corridor is not a
checkpoint; it is a robot standing where students gather.

⚠️ **Powering down has a cost:** the robot cannot be teleoperated out, and
recovery requires physical handling. ⚠️ **Leaving it powered has a different
cost:** student interference risk. **This trade-off must be decided by the
school**, not assumed by the project team.

---

## Other high-traffic periods

⚠️ **Lunch is not the only one.** The same logic applies to:

| Period | Concern | Status |
|---|---|---|
| Class changeover | Peak corridor movement | ❓ `SH-TIME-001` |
| Morning assembly | Concentrated students | ❓ |
| Examination periods | Unauthorised access risk | ❓ `SH-TIME-015` |
| Break times | Peak movement | ❓ |
| Fire drills | ⚠️ Evacuation | Procedure required |
| Events and visitors | Unfamiliar people | ❓ |

⚠️ **Where the timetable is unavailable or stale, the safe default is that
operations do not run.** `SH-TIME-016`.

---

## Battery interaction

⚠️ **Shutdown and charging must not conflict with the battery reserve.**

- The robot should reach the checkpoint **with charge remaining**, not on empty.
- ⚠️ If the return route is blocked at cutoff → hold, request assistance.
- ⚠️ **A low battery must never trigger an unverified route** — not a stair, not
  a lift, not a closed corridor.
- Charging after shutdown follows `docs/shana/hardware/battery_and_charging.md`;
  ⚠️ attended, non-flammable surface, within temperature limits.

---

## Data failure behaviour

| Failure | Required behaviour |
|---|---|
| Timetable unavailable | ⚠️ **Do not run operations.** Fail safe, not permissive |
| Timetable stale | Treat cutoff as unknown → no new tasks |
| Timetable incorrect | ⚠️ **This is a safety failure, not a scheduling one.** Must be reported |
| System clock wrong | ⚠️ **Treated as a data failure** — a wrong cutoff is a wrong cutoff |

⚠️ **A wrong timetable is worse than a missing one.** Missing data stops the
system; wrong data could release the robot into a crowded corridor.

---

## Resumption

```
1.  An AUTHORISED PERSON resumes operations
2.  System re-checks every precondition
        · Battery
        · Robot state
        · Route availability
        · Corridor states
        · Supervisor present
3.  If all clear → operations resume
4.  If any unclear → remains suspended, with a stated reason
```

⚠️ **Resumption is never automatic.** A scheduled time is not an authorisation.

---

## The honest position

⚠️ **The lunch shutdown is one of the cheapest safety controls in the entire
system.** It costs nothing but availability, it eliminates the highest-traffic
period, and it requires no sensor, no model, and no engineering.

⚠️ **If the schedule cannot be obtained or trusted, the correct decision is that
SHANA does not operate at all**, rather than that it operates on a guess.