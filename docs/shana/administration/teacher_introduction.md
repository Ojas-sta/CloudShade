# SHANA — Introduction for Teachers and School Administration

**For:** Staff, teachers, and administrators
**Date:** 2026-10-10
**Status:** Proposal. Nothing has been built, connected, or demonstrated.

---

## What SHANA is

SHANA is a proposed **autonomous indoor delivery robot** for this school. It
would carry **approved items only** between **authorised school locations** —
for example, a sealed box of worksheets from a staffroom to a laboratory.

Teachers would request a delivery through a dashboard on their own device. SHANA
would plan the route, navigate the corridors, stop at the destination, and ask
the recipient to confirm they have received it.

## Why feasibility research is happening first

SHANA has **not been built**. Before anything is connected to a robot, we are
establishing three things:

1. **What the robot can actually do** — verified from manufacturer documentation
   and testing, not assumed.
2. **What the school permits** — because owning a robot and permitting it to
   operate autonomously among students are different decisions.
3. **What is genuinely unknown** — and must be measured rather than guessed.

⚠️ **This matters more than it might seem.** The robot is a quadruped. It can
navigate and carry, but it **cannot pick things up** — there is no arm. It is
**more reliable at delivery than at manipulation**, and that shapes everything
proposed here.

## What the first demonstration could include

A **controlled, single-floor delivery demonstration** in a verified, cleared
route:

1. A teacher requests a delivery from a fixed list of approved rooms.
2. The system checks the request, the robot's readiness, and battery level.
3. The robot navigates a short, pre-verified route to the pickup point.
4. It waits for the recipient to place the item and confirm loading.
5. It navigates to the destination and waits for the recipient to confirm receipt.
6. The dashboard records the outcome and the time taken.

Nothing on a staircase. No lift. No doors. No exploration.

## What SHANA will **NOT** do

This is the important section.

| SHANA will not | Why |
|---|---|
| **Identify, recognise, or track students** | No facial recognition. No identity recognition. No biometrics. Anonymous counts only, if any counting is used at all. |
| **Report, accuse, or discipline anyone** | No automatic misconduct reporting. Any incident requires human review and school approval. |
| **Operate without a human present** | A named person supervises every single operation, with the authority to stop it immediately. |
| **Move where it has not been verified** | Every route is walked and approved by a person first. |
| **Use stairs or lifts in the initial pilot** | Neither is established or permitted today. |
| **Carry dangerous, fragile, or unsealed items** | No glass, liquids, chemicals, sharps, hot items, or anything unsealed. |
| **Go online or connect to the public internet** | It runs entirely on a local school network. |
| **Learn or adapt without supervision** | The system does not change its own behaviour. |

## Why student safety and privacy are central

A 15 kg robot with twelve moving legs operating among children deserves more than
caution.

- **Exclusion zone.** A physically marked area the robot stays inside, kept clear
  of people.
- **Distance.** The manufacturer's manual requires the robot to stay under
  control in a person's view and at least 2 metres from crowds.
- **Supervision.** A named adult whose only job is the robot, empowered to stop
  it with no explanation required.
- **Speed.** Capped well below a walking pace.
- **Supervision at the manufacturer.** The manual states the robot is *"not a toy
  and is not intended for use by persons under the age of 14"*. That framing
  governs everything.
- **Privacy.** If corridor congestion is ever sensed, it will produce anonymous
  states — `CLEAR`, `MODERATE`, `BUSY`, `UNKNOWN` — and nothing else. No images
  stored, no person tracked, no count tied to an individual.

⚠️ **If you would not accept this robot in your corridor, say so.** A teacher's
judgement about their own classroom is a harder test than any technical one, and
it will be treated as decisive.

## How delivery requests would work

- A teacher opens the dashboard **on the school network** and selects an origin
  and destination **from an approved list**. Free-typed destinations are not
  accepted, because the robot could not verify them.
- The system either accepts the request with a task number, or rejects it with a
  **specific reason** — for example, "the robot is not ready", "that room is not
  approved for delivery", or "not enough battery to complete and return".
- The teacher sees live status and receives a notification when the delivery
  arrives.
- **A delivery is only complete when a person confirms they received it.**
  Arriving at the door is never treated as completion.

## How teachers could provide feedback

- A short, optional feedback form after each delivery.
- A "something went wrong" button that notifies the team directly.
- An open invitation to say **when it should not be used at all** — teaching
  periods, exams, assemblies, crowded changeovers, or any time it feels wrong.

⚠️ **A system teachers can stop using is a system teachers will trust.** The
feedback loop is not a formality.

## What permissions we are requesting

102 permissions are being sought across ten parties. **None has been granted.**
The full list is in [`permission_checklist.md`](permission_checklist.md).

The most important ones are:

- Written authorisation to use the robot's development interface.
- Approval of a dedicated robot network segment.
- Approval of the server room (proposed: AI Lab).
- Approval of any marker or access point installation.
- A robot operating policy for occupied corridors.
- Approval of a controlled testing plan.
- **Separate** approval for any student-facing demonstration.
- Insurance confirmation.
- The Data Fiduciary's approval, if any data is processed.

## What information we need

| From | What |
|---|---|
| **Robot custodian** | Model, firmware, installed options, battery type |
| **IT** | Network topology, Wi-Fi coverage, firewall and VLAN settings |
| **Facilities** | **Stair dimensions**, corridor widths, floor surfaces, route survey |
| **Teachers** | Which rooms actually need deliveries, how often |
| **Administration** | Budget, room allocation, operating policy |
| **Lift service provider** | Whether any sanctioned integration exists |

The single most decisive unknown is a **tape measure**: if the school's stair
risers exceed 160 mm — the manufacturer's documented limit — stairs are
permanently out of scope, and we can establish that in an afternoon.

## What evidence is needed before a demonstration

- [ ] Written authorisation to use the robot
- [ ] Confirmed model, firmware and battery
- [ ] A working, permitted robot network connection
- [ ] A physically measured and verified route
- [ ] A tested emergency stop and a trained supervisor
- [ ] An approved item list and approved destinations
- [ ] A battery reserve established by measurement, not assumption
- [ ] Ten consecutive successful supervised test runs
- [ ] A written abort procedure
- [ ] **Separate written approval for student presence**

None of these is currently satisfied.

## How the exhibition version will be constrained

- **Single floor.** No stairs, no lifts, no doors.
- **Fixed, pre-verified route.** The robot never explores.
- **Demonstration payloads only**, not live school items.
- **A physically enforced exclusion zone**, and spectators kept outside it.
- **A named supervisor present throughout**, with unilateral stop authority.
- **Pre-agreed abort criteria**, decided before the day rather than during it.
- **A fallback demonstration** that does not depend on live navigation, so an
  event is never jeopardised by a technical failure.
- **Incident and first-aid plan** prepared as for any school event.

## Who supervises it

A named, trained school staff member — **not** the project team — whose only
responsibility during operation is the robot. At least two trained people, with
backup cover.

⚠️ The project team may assist. **The project team may not be the supervisor of
record.**

## How the system can be stopped

1. **Immediately, by anyone:** the physical emergency stop method agreed with the
   school and tested beforehand.
2. **By the supervisor:** the robot's remote controller, tested immediately before
   every run.
3. **By the school:** a dashboard control available to authorised administrators.
4. **By anyone at any time:** if anyone judges the situation unsafe, the run
   stops. **No justification is required and no one may override that judgement.**

There is always a method that does not depend on the software working.

## How incidents and faults are reported

- Every fault, near-miss, and incident is logged and reviewed — including
  near-misses where nothing went wrong.
- A named school contact receives the record.
- Repeated faults pause the system until investigated.
- **Failures are reported alongside successes.** A system that reports only
  successes is not providing evidence.

## How success will be measured

Not by how impressive the robot looks. By:

| Measure | Why |
|---|---|
| Task completion rate | Did it actually deliver? |
| Delivery success rate | Right room, right item, undamaged? |
| Average task duration | Is it faster than walking? |
| Obstacle-response latency | How quickly did it react? |
| Human interventions per run | How much did it need help? |
| Localisation failure rate | How often did it lose its place? |
| Battery consumed per mission | Can we plan reliably? |
| **Failures, with reasons** | What does it not do? |

⚠️ **Every published number must come from measured data collected under a
written protocol.** No target is presented as a result.

## What we are asking of you

1. **Read this.**
2. **Answer the questions that concern you** in the feasibility questionnaire.
   "I don't know" is a perfectly good answer.
3. **Tell us what would make you refuse to use it.** We will design around that.
4. **Decide the go/no-go gates.** Progress must stop if they are not met.

---

## Honest summary

SHANA is a **genuinely useful idea** built on hardware that is well suited to
delivery and **poorly suited to manipulation**. It can carry items reliably. It
cannot pick them up. It will need help at transitions it cannot yet handle
safely. And it operates among children, which sets a very high bar.

The most likely outcome of this phase is a **smaller, single-floor system that
works**, rather than a three-block system that does not. That would be a good
result, not a disappointing one.

---

*Questions about anything here should go to the project contact named in the
feasibility questionnaire. Nothing in this document has been demonstrated.*