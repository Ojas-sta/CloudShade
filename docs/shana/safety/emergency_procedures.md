# SHANA — Emergency Procedures

**Status: DRAFT. No procedure here has been agreed, tested, or approved.**

⚠️ **A procedure that has not been physically tested is not a procedure. It is a
draft.** Every procedure below requires school approval and a supervised test
before it is relied upon.

---

## Principle: five ways to stop, one way to start

| # | Method | Who | Depends on software? |
|---|---|---|---|
| 1 | **Physical emergency stop** — agreed method | Anyone | ⛔ No |
| 2 | **Remote control** | Supervisor | ⛔ No |
| 3 | **Hard power switch** | Supervisor / facilities | ⛔ No |
| 4 | **Dashboard stop** | Administrator | ✅ Yes |
| 5 | **Calling for a stop** | **Anyone present** | ⛔ No |

⚠️ **Only one method starts the robot, and it is a supervised manual action.**
That asymmetry is intentional.

---

## E1 — Immediate stop

### Trigger

Any of:

- Any person enters the exclusion zone
- The robot deviates from the marked route
- The robot behaves unexpectedly in any way
- A spectator becomes distressed or agitated
- Communication with the robot is lost
- Any abnormal noise from the robot
- Battery falls below the abort threshold
- Payload shifts
- **Anyone judges the situation unsafe**

### Action

```
1. Stop the robot using the fastest available method
2. Keep people back from the robot
3. Assess for injury; provide first aid if needed
4. Do NOT restart
5. Do NOT resume
6. Record what happened
7. Await an explicit decision from the school supervisor
```

⚠️ **There is no penalty for stopping.** ⚠️ **Any person may call for a stop, and
that call is honoured without discussion and without justification.**

---

## E2 — After a stop

```
1. Robot holds position or is placed in a safe state
2. Supervisor determines whether the cause is understood
   ├─ Cause understood AND resolved → supervisor may authorise resumption
   └─ Cause unknown or unresolved  → activity ENDS for the day
3. Record the event as incident, fault, or near-miss
4. Review before any further operation
5. If the cause involves hardware → do not operate until inspected
```

⚠️ **Repeated faults pause the system** until investigated.

---

## E3 — Fire or evacuation

```
1. Execute the SCHOOL'S evacuation procedure immediately
2. The robot follows the building procedure, not SHANA's
3. If safe, move the robot clear of the route ONLY if a person is already doing so
4. ⚠️ DO NOT attempt to save, recover, or retrieve the robot
5. Account for all people first
6. Do not resume until the school has stood down the alarm
   and confirmed the building is safe
7. Record the event
```

⚠️ **A robot is never worth a person's safety, and the robot itself is never
retrieved during an evacuation.**

### Fire recall in lifts

⚠️ If the robot is inside a lift when a fire alarm triggers, Phase I recall may
return the car to a designated level with doors open and remove the lift from
service. **The robot must not obstruct a landing door.** This is one reason
lift use is excluded from the pilot.

---

## E4 — Power failure

| Event | Consequence | Response |
|---|---|---|
| **Server power lost** | Robot has no watchdog; it holds position | Supervisor retrieves; battery monitor becomes critical |
| **Mains lost, robot on battery** | Robot holds or crouches within 10 min at low charge | Retrieve before the crouch |
| **Full site power loss** | Emergency lighting only | Abort. Robot is not retrieved in the dark unless immediate life-safety risk |

⚠️ **The robot holds position when the server is lost — it does not wander.** But
it has no software watchdog, which is why the supervisor is mandatory.

---

## E5 — Loss of communication

```
Server side: mark robot ORPHANED, raise alarm, accept no new tasks
Robot side:  receive no further commands, HOLD POSITION

Recovery:
1. Supervisor notified with last known position and position age
2. Supervisor attempts to restore communication
3. If restored → verify state before any further command
4. If not restored → retrieve physically by the agreed method
5. ⚠️ If the robot has been moved by anyone → RELOCATION EVENT must be recorded
```

⚠️ **`last_pose_age` must be displayed prominently.** A 40-minute-old position
next to a physical robot invites someone to look in the wrong place.

---

## E6 — Battery depletion

```
Below abort threshold:
1. Stop accepting new tasks
2. Return to a verified, reachable, safe checkpoint — IF the route is available
3. If return is NOT safely feasible → hold position, request human assistance

⚠️ NEVER: take an unverified stair route
⚠️ NEVER: enter a lift without approval
⚠️ NEVER: cross a closed or hazardous corridor
⚠️ NEVER: rely on the robot's own 10-minute crouch warning
```

⚠️ **At low battery the robot crouches within 10 minutes.** If that happens in a
corridor it is safe. **If it happens on a staircase it is a fall.** This is why
stair operation is excluded from the pilot.

---

## E7 — Student interference or tampering

```
1. Stop the robot
2. Do not attempt to continue while students are interacting with it
3. Retrieve or secure the robot
4. Record the event
5. ⚠️ If a student has modified, mounted on, or damaged the robot
   → STOP ALL OPERATION until it has been inspected
6. Notify administration
```

⚠️ **A modified robot is an unverified robot.** It is not operated until it has
been returned to a known state and inspected.

---

## E8 — Robot immobilisation

If the robot cannot move and cannot be recovered by the normal method:

```
1. Do not attempt a risky manual recovery
2. Clear the area
3. Escalate to facilities and the custodian
4. If the robot is blocking an emergency route → clear it immediately,
   accepting some risk to the robot
5. Record the event
```

⚠️ **Clearing a fire route takes precedence over protecting the robot.**

---

## E9 — Injury

```
1. Stop everything
2. Provide first aid
3. Notify school administration immediately
4. Do not resume operation
5. Preserve any video and logs
6. Record a full incident report
7. ⚠️ Any injury STOPS the activity for the day
```

⚠️ **No exception. No resumption pending formal review.**

---

## E10 — Loss of authorisation

⚠️ **If any permission is withdrawn — network, premises, robot, insurance — all
operation stops immediately.**

```
1. Stop the robot
2. Retrieve it if practical
3. Remove any installations
4. Confirm in writing what was withdrawn and when
5. Do not resume until re-authorised
```

⚠️ **This includes revocation of school Wi-Fi access, insurance lapse, or the
robot custodian leaving the organisation.**

---

## Pre-event checklist

**Before any operation — every time, no exceptions:**

- [ ] Exclusion zone marked and clear
- [ ] Route physically surveyed and clear of cables, rugs, chairs
- [ ] Spotter present, briefed, RC in hand
- [ ] **RC authority tested** — robot responds, then stops
- [ ] Emergency stop method confirmed and accessible
- [ ] Battery above threshold
- [ ] Payload secured — cradle AND strap
- [ ] Firmware matches the pinned version
- [ ] Dashboard reachable; logging active
- [ ] Abort criteria known to everyone present
- [ ] *(G1 only)* frame and rope fitted, damping confirmed
- [ ] Camera recording if required

---

## Questions

| ID | Question | Priority |
|---|---|---|
| `SH-SAFE-001` | What is the approved emergency-stop arrangement? | **P0** |
| `SH-SAFE-002` | **Has it been physically tested?** | **P0** |
| `SH-SAFE-004` | Which documented method applies — `Damp()`, controller, or hard power? | P1 |
| `SH-SAFE-005` | Named spotter and their exact duties | **P0** |
| `SH-SAFE-006` | **Unilateral stop authority confirmed** | **P0** |
| `SH-SAFE-008` | Exclusion zone definition and enforcement | **P0** |
| `SH-SAFE-010` | Speed cap enforced in software | P1 |
| `SH-SAFE-025` | Incident and near-miss reporting process | P1 |
| `SH-BLOCK-016` | Fire or evacuation behaviour | **P0** |

---

⚠️ **Nothing in this document has been agreed, tested, or approved. During
initial development, no procedure may be relied upon without a physical test
first.**