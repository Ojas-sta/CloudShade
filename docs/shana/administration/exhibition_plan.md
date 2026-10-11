# SHANA — Exhibition Demonstration Plan

**Status: PROPOSAL. No permission has been granted. No demonstration has been
planned, scheduled, or rehearsed.**

---

## 1. Scope of the proposed demonstration

### Included

A **single-floor, fixed-route, supervised delivery demonstration** in a verified,
cleared location.

### Explicitly excluded

| Excluded | Reason |
|---|---|
| ❌ Stairs | Step limit is 160 mm; school risers unknown; manufacturer forbids complex terrain |
| ❌ Elevators | No sanctioned integration; physical modification prohibited by law |
| ❌ Doors of any kind | Not tested, not required |
| ❌ Route exploration | The robot never discovers a new route |
| ❌ More than one floor | The robot cannot identify which floor it is on |
| ❌ Live school items | Demonstration payloads only |
| ❌ Unattended operation at any point | A named supervisor is present throughout |
| ❌ Any unscripted behaviour | The demonstration is a repeatable workflow |

⚠️ **If any of these exclusions cannot be honoured, the demonstration does not
proceed.** They are not negotiable on the day.

---

## 2. Preconditions — all must be satisfied

### 2.1 Documentation and permission

- [ ] Written authorisation to use the robot's development interface
- [ ] Written approval of the demonstration scope
- [ ] **Separate written approval for student presence**
- [ ] Venue confirmed and physically surveyed
- [ ] Insurance confirmed
- [ ] Emergency stop procedure agreed and **tested**
- [ ] Named supervisor, plus at least one trained backup
- [ ] Approved item list and approved destination list

### 2.2 Technical readiness

- [ ] Robot identity, firmware and battery confirmed
- [ ] Robot network connection established and stable
- [ ] Server running and reachable
- [ ] Route mapped, walked by a person, and verified
- [ ] **Ten consecutive successful supervised test runs**
- [ ] Battery reserve established **by measurement**
- [ ] Payload carrier tilt-tested and retention verified
- [ ] Dashboard functional, including cancellation and abort

### 2.3 Operational readiness

- [ ] Abort criteria agreed in writing **before** the day
- [ ] Fallback demonstration prepared and tested
- [ ] Charging plan for before, during, and after
- [ ] Incident and first-aid plan agreed
- [ ] Spectator exclusion arrangement agreed with the organisers
- [ ] Rehearsal completed with at least one full dress run

⚠️ **No precondition may be waived on the day.** If any is unmet, the
demonstration is postponed or replaced by the fallback.

---

## 3. Demonstration design

### 3.1 Narrative

The demonstration should tell a coherent story rather than show a trick:

> *"This robot knows where it is, knows where it is allowed to go, and knows when
> to stop and ask a human for help."*

That is a more interesting and more honest story than *"look, it walks"*.

### 3.2 The sequence

| # | Beat | Purpose | Duration |
|---|---|---|---|
| 1 | Show the dashboard and the approved destination list | Establishes that destinations are constrained, not free-typed | 30 s |
| 2 | Teacher submits a delivery request | Shows the teacher workflow | 20 s |
| 3 | Show the system validating and checking readiness | Shows the safety checks happen *before* movement | 20 s |
| 4 | Show the task queued with a task number and ETA | Shows traceability | 15 s |
| 5 | Robot navigates the pre-verified route | The core capability | 60–120 s |
| 6 | **Obstacle injected at a known point** — robot stops or reroutes, **latency shown on screen** | Shows measured safety behaviour | 45 s |
| 7 | Robot arrives, recipient confirms receipt | Shows completion requires human confirmation | 30 s |
| 8 | Show the delivery record: duration, battery, interventions, **any failure** | Shows honest measurement | 30 s |
| 9 | Show the operator abort control being used and then released | Shows a human can always stop it | 30 s |
| 10 | Q&A; show the failed-attempt log if asked | Shows honesty about limits | open |

**Total robot time: approximately 5–6 minutes.** Keep it short. Most
exhibitions lose an audience faster than they gain one.

### 3.3 The obstacle test

⚠️ This must be a **deliberate, controlled, repeatable** test, not a surprise.

- The obstacle is placed at a **known marked point** on the route.
- The same obstacle, same point, every rehearsal.
- The **response latency is measured and displayed**, not estimated.
- If the robot does not respond, that is a valid result — **report it, do not
  re-run until it passes.**

### 3.4 Demonstrating failure honestly

A prepared failure demonstration is stronger than a lucky success:

1. Submit a request for a room that is **not on the approved list** — show the
   system rejecting it with a clear reason.
2. Or: demonstrate the **operator abort** — stop the robot mid-route with the
   remote, show it stops, then show the task resuming or cancelling cleanly.

⚠️ **If a failure occurs spontaneously, do not hide it and do not re-run.** Say
what happened, explain the design response, and move on. Judges remember
honesty far more than perfection.

---

## 4. Physical arrangements

### 4.1 Space

- Venue **surveyed and confirmed** before the event. An unverified venue
  invalidates all prior route work.
- Route **marked**, cleared of cables, rugs, chairs, and temporary equipment.
- A **2 m minimum clearance** maintained along the route at all times.

### 4.2 Exclusion zone

- **Physically marked** with barriers or tape — not signage alone.
- Enforced by a **dedicated marshal** whose job is keeping people out.
- ⚠️ The exclusion zone is the single most important physical control. Every other
  safety measure is secondary to keeping people out of the robot's space.

### 4.3 Spectators

- Kept outside the exclusion zone for the entire demonstration.
- **No movement of the robot while spectators are inside the zone.**
- Consider one or two runs with the robot stationary while the demonstration is
  explained, so the crowd is not walking around a moving robot.

### 4.4 Robot posture and speed

- **Maximum 0.4 m/s.** Slower than a walking person.
- Standing posture throughout. No stairs, so no special gait.
- Payload: **demonstration items only**, ≤2 kg, in a latched carrier with a
  secondary strap.
- ⛔ **No glass, liquids, chemicals, sharps, hot items, or anything fragile.**

---

## 5. Roles on the day

| Role | Responsibility | Notes |
|---|---|---|
| **School supervisor** | Overall authority to stop; sole operator of the remote | **Must be school staff, not the project team.** At least one trained backup present |
| **Marshal** | Enforces the exclusion zone | Cannot be the operator |
| **Safety officer** | Monitors the audience and crowd behaviour | Escalates immediately |
| **Demonstrator** | Narrates; operates the dashboard | May not control the robot |
| **Technical operator** | Watches telemetry, logs results | May not control the robot |
| **First-aid contact** | Present and known | Named in advance |

⚠️ **Only the school supervisor may stop the robot — but anyone present may
call for it to stop, and that call is always honoured without discussion.**

---

## 6. Abort criteria

**Any one of these, at any time, stops the demonstration:**

- Any person enters the exclusion zone
- The robot deviates from the marked route
- The robot behaves unexpectedly in any way
- A spectator becomes distressed or agitated
- Communication with the robot is lost
- Any abnormal noise from the robot
- Battery falls below the abort threshold
- The payload shifts
- Any person present judges the situation unsafe
- The venue, lighting, or floor conditions differ from the surveyed state

### After an abort

1. Stop the robot using the agreed method.
2. Clear the area.
3. **Do not resume** without an explicit decision from the school supervisor.
4. Record what happened and why.
5. If the cause is not immediately understood, **the demonstration is over** for
   the day.

⚠️ **There is no penalty for aborting.** There is a serious penalty for continuing
when something is wrong.

---

## 7. Contingency

### 7.1 Fallback demonstrations

Prepared in advance and tested:

| Fallback | Description |
|---|---|
| **A — Simulation** | Run the same task sequence in the Go2 simulator on a screen. Demonstrates the full software system with zero physical risk. |
| **B — Recorded** | Show a recording of a successful supervised test run, with its actual measured results. |
| **C — Static** | Demonstrate the dashboard, the validation logic, and the data model with the robot present but stationary. |

⚠️ **The fallback is not a failure.** It demonstrates engineering maturity, and
it is honest. An aborted live demo with no fallback damages the project far more
than a clean fallback.

### 7.2 Network failure at the venue

- The robot link is **Ethernet**, not Wi-Fi. A WLAN failure should not affect the
  robot's operation.
- If the server is unreachable, the robot **holds position** and the
  demonstration moves to fallback A or C.

### 7.3 Power failure

- Robot on battery: it will hold or crouch within 10 minutes at low charge.
- Server on UPS: verify UPS runtime covers the demonstration duration.
- Venue power loss: abort.

---

## 8. Evaluation and acceptance

### 8.1 What is measured

| Metric | Method |
|---|---|
| Task completion rate | Completed ÷ submitted, across all runs |
| Delivery success rate | Correct destination, correct item, undamaged |
| Average and p95 task duration | Wall-clock, submit → receipt confirmation |
| Obstacle-response latency | Video timestamp → stop-command timestamp |
| Human interventions | Count per run |
| Localisation failure rate | Injected and natural occurrences |
| Battery consumed per mission | Measured, not calculated from a datasheet |

### 8.2 Evidence rules

⚠️ **Every number shown to a judge must come from measured data collected under
a written protocol agreed in advance.**

- **No target is presented as a result.**
- **No simulation number is presented as a hardware number.**
- **No datasheet figure is presented as a measurement.**
- **Failures and near-misses are reported alongside successes.**

### 8.3 Acceptance criteria

Before the event, agree in writing:

- The number of consecutive successful runs required *(recommended: 10)*
- What constitutes a success
- What constitutes a failure
- Who judges acceptance, and against what criteria
- That acceptance criteria are **not** waived on the day under time pressure

---

## 9. Incident handling during the event

| Event | Response |
|---|---|
| Robot contacts a person or object | Stop. Clear the area. Check for injury. Record. Do not resume without a decision. |
| Any injury | First aid immediately. Stop. Notify school. Do not resume. |
| Robot damage | Stop. Remove from public area. Record. |
| Fire alarm | Execute the school's evacuation procedure with the robot. Do not attempt to save the robot. |
| Loss of control | Spotter uses the agreed stop method. Clear area. Record. |
| Any near-miss | Record it. Review it. Do not dismiss it. |

⚠️ **Any injury stops the demonstration for the day.** There is no exception.

---

## 10. After the demonstration

- [ ] Publish measured results, **including failures**
- [ ] Provide the school with a plain-language report
- [ ] Conduct a review with the supervisor, marshal, and safety officer
- [ ] Record what should change before any further operation
- [ ] Retrieve the robot, and remove all temporary installations
- [ ] Return any rooms to their prior state
- [ ] Archive the delivery logs and incident records
- [ ] Decide whether the project continues, and on what terms

---

## 11. The honest paragraph to read aloud

If asked "can it work in a school?", the accurate answer is:

> *"In a single, verified corridor on one floor, with a person supervising and a
> clearly marked exclusion zone — yes, and we will show you the measurements
> including the things that went wrong. Across multiple floors, using stairs or
> lifts, or without supervision — not yet, and we will not pretend otherwise."*

That answer is more credible than any demonstration, and it is the one this plan
is built to support.