# SHANA — Introduction

## The short version

SHANA is a **proposed** autonomous indoor delivery robot for this school. It
would carry approved items between authorised locations, requested by teachers
through a local dashboard.

**It has not been built.** Nothing in this repository has been connected to,
operated, or demonstrated on a robot, a network, a lift, or a camera.

---

## The problem it addresses

Schools have repetitive internal transport tasks: worksheets to a lab, exam
papers to a staffroom, library materials between buildings. They are low-value,
walk-consuming, and often rushed at changeover.

A robot that could take one of those off a teacher's hands has real value. A
robot that reliably does nothing has none.

## Why this is a serious engineering project

Because the robot is a **quadruped**, not a humanoid, and that shapes everything.

**What it does well:**

- Carries a payload on a stable, low centre of mass
- Navigates corridors with obstacle avoidance
- Steps over thresholds and moderate slopes
- Recovers from minor trips on its own

**What it cannot do:**

- Pick anything up. **There is no arm.** Delivery means carrying on a tray.
- Know which floor of a building it is on. **There is no altitude, pressure, or
  barometer sensor anywhere in its message set.**
- Return to a charging dock autonomously. **No such service exists in the
  documented interface.**
- Operate without a person present. The manufacturer's manual states it is
  *"not a toy and is not intended for use by persons under the age of 14"*, and
  requires it to stay under control in the user's view and at least 2 metres from
  crowds.

## The honest position on vertical movement

The school spans multiple floors across three blocks. Reaching across floors
requires stairs or elevators.

| Route type | Status | Why |
|---|---|---|
| Single floor | **Feasible** | Uses documented, tested robot capabilities |
| Stairs | **Not established** | Documented step limit is 160 mm; manufacturer's manual forbids complex terrain; the dedicated stair gait was removed from the current SDK |
| Elevators | **Not established, probably not permitted** | No sanctioned third-party interface has been identified; physical modification is prohibited by law |

⚠️ **The single most valuable measurement in this project is a tape measure.** If
the school's stair risers exceed 160 mm — the manufacturer's documented limit —
stairs are permanently out of scope, and that is settled in an afternoon.

**If teachers do not need cross-floor delivery, the project becomes single-floor
and substantially more likely to succeed.** That is a good outcome, not a
disappointment.

## The privacy position

SHANA's purpose is to move objects, not to observe people.

**It will not:** recognise faces, identify individuals, use biometrics, track
individuals across time, generate automatic misconduct or attendance reports, or
store video by default.

**If corridor congestion is ever sensed**, it produces anonymous states only —
`CLEAR`, `MODERATE`, `BUSY`, `UNKNOWN` — and nothing else. Low confidence or a
failed sensor produces `UNKNOWN`, **never** `CLEAR`.

⚠️ **A currently recommended alternative is even simpler:** staff report corridor
status manually. No camera, no sensor, no privacy determination, no analytics
maintenance. It may be entirely sufficient for a first pilot, and it should be
preferred if it works.

## The safety position

| Control | Status |
|---|---|
| Named supervisor with unilateral stop authority | Required at every stage. **Not yet established.** |
| Physical exclusion zone, 1.5 m either side | Required. **Not yet established.** |
| Speed capped below walking pace | Required. **Not yet enforced.** |
| Payload limited to approved, safe items | Required. **Not yet agreed.** |
| Emergency stop tested before every run | Required. **Not yet tested.** |
| Fire evacuation procedure agreed | Required. **Not yet agreed.** |

⚠️ **Unsupervised operation is not proposed in any phase.** The manufacturer's
documentation and the nature of the environment both prohibit it.

## Where the project actually is

| | |
|---|---|
| Questions written | **434** |
| Questions answered | **0** |
| Permissions requested | **102** |
| Permissions granted | **0** |
| Go/no-go gates defined | **11** |
| Gates passed | **0** |
| Verifications performed | **0** |
| Assumptions unverified | **30 of 34** |

⚠️ **That is the correct state at the end of a feasibility phase.** The output
of this phase is a precise, evidenced account of what is unknown — not a
working system.

## What can happen without anyone's permission

- Build and validate the entire software stack in simulation. The Go2 simulator
  speaks the same communication channels as the real robot and runs on a CPU.
- Read SDK source code and document every interface used.
- Design the architecture, its failure modes, and its decision records.
- Prepare the questions and the permission requests.

## What happens next

1. **Measure the stairs.** A tape measure and an afternoon.
2. **Ask the lift manufacturer one question.** Whether their existing contactless
   system can be used by a device that is not a person.
3. **Ask teachers which rooms they actually need.** This may shrink the project
   to something achievable.
4. **Read the robot's battery label.** It determines the endurance budget.
5. **Request written development authorisation.** Nothing can proceed without it.

Everything else is documented in detail across `docs/`.

---

*This document describes intentions and constraints. It does not describe a
system that exists.*