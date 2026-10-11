# SHANA — Human Intervention

**Status: PROPOSED. No human has been trained, briefed, or assigned.**

---

## The principle

> **Every capability SHANA has can be withdrawn by a human in seconds. Nothing
> SHANA can do requires the software to be working.**

⚠️ **This is the load-bearing safety property of the entire project.** It is also
the property most likely to be eroded by convenience — "the dashboard is easier",
"just let it finish", "the schedule is tight."

---

## Roles

| Role | Responsibility | May stop? | May move? |
|---|---|---|---|
| **Supervisor (spotter)** | Sole responsibility for the robot during operation | ✅ **Unilaterally** | ✅ Manual remote |
| Backup supervisor | Cover when the supervisor is unavailable | ✅ | ✅ |
| Room administrator | Manage destinations for their block | ✅ | ⛔ |
| Robot custodian | Physical handling, relocation recording | ✅ | ✅ Manual |
| Facilities | Corridor declarations | ✅ | ⛔ |
| Safety officer | Policy, incident review | ✅ | ⛔ |
| Data Fiduciary | Approve data processing | ✅ | ⛔ |
| **Teacher** | Request, confirm, cancel | ✅ | ⛔ |
| Safety supervisor (software) | Veto commands | ✅ **Automatic** | ⛔ |
| **The robot** | Nothing | ⛔ | ⛔ |

⚠️ **No teacher-facing role has motor control. No role has motor control through
software. The only way to move the robot manually is the physical remote, held
by a supervisor.**

---

## The supervisor

### Required

- Named, in writing
- Trained, with a training record
- Present for **every** operation
- Given the remote controller
- Able to reach the hard power switch
- **Empowered to stop with no justification and no override**

### Duties

```
BEFORE
  · Confirm the exclusion zone is marked and clear
  · Confirm the route is clear
  · Test remote control authority  ← every run
  · Confirm battery is above threshold
  · Confirm payload is secured
  · Confirm abort criteria are known to everyone present

DURING
  · Maintain visual contact with the robot at all times
  · Keep people outside the exclusion zone
  · Listen for abnormal noise
  · Watch telemetry
  · STOP on any doubt

AFTER
  · Confirm the robot is at a checkpoint in a safe state
  · Record interventions
  · Report faults and near-misses
```

⚠️ **The supervisor's job is not to watch the robot's task. It is to keep people
away from the robot and to notice anything wrong.**

---

## Unilateral stop authority

> **Any person may call for the robot to stop. That call is always honoured,
> without discussion, and without requiring justification.**

⚠️ This is written explicitly because under social pressure — a demonstration
in front of judges, a teacher wanting to see the end of a delivery — people will
hesitate. The rule exists to remove that hesitation.

⚠️ **No one may overrule a stop call.** Not a teacher, not an administrator, not
the project team.

---

## Five ways to stop

| # | Method | Depends on software | Intended use |
|---|---|---|---|
| 1 | **Physical emergency stop** — agreed method | ⛔ No | Immediate threat |
| 2 | **Remote controller** | ⛔ No | Normal supervision |
| 3 | **Hard power switch** | ⛔ No | Last resort |
| 4 | **Dashboard stop** | ✅ Yes | Planned stop |
| 5 | **Calling for a stop** | ⛔ No | Anyone, any time |

⚠️ **Methods 1, 2, 3, and 5 work even if the server, the dashboard, and the robot's
software are all failed.** Only method 4 depends on the system working.

---

## Manual intervention

| Situation | Action |
|---|---|
| Robot stopped on route | Assess; recover physically; **record relocation if moved** |
| Robot at wrong location | Do not attempt to correct remotely; retrieve and re-localise |
| Robot unresponsive | Do not attempt risky recovery; clear the area; escalate |
| Robot blocking a fire route | ⚠️ **Clear it immediately, accepting some risk to the robot** |
| Robot damaged | Stop all operations until inspected |
| Robot modified or tampered with | Stop all operations until returned to a known state |

⚠️ **The robot is never worth a person's safety.** Not in a demonstration, not in
an emergency, not ever.

---

## Response-time requirement

⚠️ **A robot that safely stops but sits unattended is a different situation from
one that is attended within two minutes.**

| Question | Why it matters |
|---|---|
| How quickly can a supervisor attend a stopped robot? | Determines whether stopping is safe |
| What is the maximum acceptable unattended time? | Sets the system's tolerance for faults |
| Who attends at night, at weekends, during holidays? | ⚠️ Robot may be stopped then too |

Questions: `SH-SAFE-019`, `SH-TIME-018`

⚠️ **If the response time is unacceptable, stopping is not a safe fallback, and
the operating model must change.** This is a real design dependency, not an
administrative detail.

---

## Manual override of automated decisions

| Situation | Who may override | Constraints |
|---|---|---|
| Manual corridor closure | Named authority | Logged |
| Manual corridor reopening | Named authority | ⚠️ Not automatic |
| Cancelling a task | Teacher, admin, custodian | Logged |
| Pausing the system | Supervisor, admin | Logged |
| Manual mission override | Supervisor | ⚠️ Only via direct remote, not by editing state |
| **Overriding the safety supervisor** | ⛔ **No one** | — |
| **Overriding a stop call** | ⛔ **No one** | — |

⚠️ **The software safety supervisor can be stopped, but it cannot be overridden
by any human role.** That is the point of it being independent.

---

## Manual relocation — the most under-recorded event

⚠️ **Moving the robot by hand invalidates its pose estimate. This must be
recorded every time.**

```
Why it matters:
  If the robot is carried up a flight of stairs and placed on floor 3,
  scan matching succeeds against BOTH floor 3 and floor 1.
  The robot does not know it is on the wrong map.
  It plans confidently against the wrong floor, with no error raised.

Who records it:
  · The custodian
  · The supervisor
  · Anyone who physically handled the robot

When:
  · Before the robot may accept a new task
```

⚠️ **This is the most likely way the robot ends up somewhere it believes it is
not.** Every manual relocation must be entered in the console, and the console
must make it difficult to forget.

---

## Training

| Trainee | Trained on | Evidence required |
|---|---|---|
| Supervisor | Remote control, emergency stop, abort criteria, relocation recording, response to fault | Training record + practical test |
| Backup supervisor | As above | Same |
| Robot custodian | Battery handling, quarantine, physical handling, relocation recording | Training record |
| Facilities | Corridor declaration and reopening | Briefing record |
| Teachers | Requesting, confirming, cancelling, reporting a problem | Written guide |

⚠️ **"Trained" means demonstrated, not attended.**

Questions: `SH-SAFE-007`, `SH-SAFE-023`, `LAB-P5`

---

## The honest position

⚠️ **The human is the safety system. The software is a convenience.**

That is an unusual thing for a robotics project to state plainly, and it is
true here: the robot carries approved items in trays, never picks anything up,
never travels where a person has not walked, and stops whenever anything is
unclear. Its value is that it can carry a box down a corridor reliably while
someone else holds the remote and watches.

⚠️ **If the school cannot staff a supervisor reliably, SHANA should not operate.**
That is a governance decision, and it is the right one.