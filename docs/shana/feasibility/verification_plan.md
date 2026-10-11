# SHANA — Verification Plan

**How each unresolved question will actually be resolved, and what must be
approved first.**

This document exists because **no risk in this project retires through further
research**. Every item requires a measurement, a written approval, or a
supervised test. The plan says what each one is, who must authorise it, and what
it produces.

⚠️ **No test described here has been performed, scheduled, or authorised.**

---

## Verification types

| Type | What it means | Who authorises |
|---|---|---|
| **M — Measurement** | A physical measurement with a tool | Facilities, robot custodian |
| **R — Research** | Reading documentation or source code | Nobody — we can do this |
| **A — Answer** | A written response from a party | The party themselves |
| **T — Test** | A supervised physical or system test | School, plus insurer where relevant |

⚠️ **Only `R` can be completed without anyone's permission.** Everything else
needs a request issued by the repository owner.

---

## Priority 1 — No permission required, can start immediately

| ID | Verification | Type | Resolves | Produces |
|---|---|---|---|---|
| V-01 | Read SDK2 and ROS 2 source; document every interface used | R | `SH-GO2-010` to `-022` | An interface inventory with citations |
| V-02 | Confirm battery telemetry fields in the message definitions | R | `SH-GO2-019`, `SH-BAT-001` | Source citation |
| V-03 | Enumerate every Go2 message field; confirm absence of altitude/pressure | R | `SH-MAP-010` | Documented negative finding *(already done — see `assumptions.md` A-09)* |
| V-04 | Read the "Aruco Recharge Service Interface" documentation | R | `SH-BAT-022` | API summary or a documented failure to retrieve |
| V-05 | Build a link budget and site-survey plan for radio telemetry | R | `SH-RAD-012` | Survey plan with pass criteria |
| V-06 | Compare mmWave presence sensing against CCTV analytics | R | `SH-CCTV-024`, `SH-DEC-011` | Options analysis |
| V-07 | Draft architecture decisions with reversal conditions | R | `SH-DEC-007` to `-015` | Decision records |
| V-08 | **Run `unitree_mujoco` and put the intended ROS 2 stack on it** | R | `SH-GO2-016`, `SH-DEC-007` | **A working stack, validated with zero hardware and zero cost** |

⚠️ **V-08 is the highest-value item in this plan.** It requires no school
answer, no hardware, and no permission, and it de-risks the entire software
architecture. It should start first.

---

## Priority 2 — Requires only a phone call or a written request

| ID | Verification | Type | Approver | Resolves |
|---|---|---|---|---|
| V-09 | **Ask Johnson Lifts whether EyeRIS is retrofittable, whether its QR is passive or active, and whether it can be provisioned to a non-human client** | A | None — a request | `SH-LIFT-014`, `-015`, `-016` |
| V-10 | **Ask the DoT WPC whether a school needs a radio possession authorisation** | A | None — a request | `SH-RAD-018` |
| V-11 | Photograph the robot, its nameplate and its battery label | M | Robot custodian | `SH-GO2-001` to `-006` |
| V-12 | Obtain the school timetable in a usable format | A | Administration | `SH-TIME-001` to `-003` |
| V-13 | Ask teachers which rooms they actually need deliveries to | A | Teachers | `SH-DEL-001` to `-004`, `SH-TIME-005` |

⚠️ **V-09 and V-13 together may eliminate most of the project's difficulty.** If
the answer to V-13 is "only rooms on this floor", then stairs and lifts leave the
scope entirely and the project becomes achievable.

---

## Priority 3 — Requires a measurement we are permitted to take

| ID | Verification | Type | Approver | Produces |
|---|---|---|---|---|
| V-14 | **Measure every stair riser, tread and clear width on candidate routes** | M | Facilities | **The number that decides whether stairs are in scope** |
| V-15 | Measure corridor widths, floor surfaces, thresholds, ramps | M | Facilities | Route survey data |
| V-16 | Survey and annotate the B↔A block connection | M | Facilities + admin | The missing route |
| V-17 | Measure lift car dimensions, door clear opening, sill gap | M | Facilities + service provider | Physical feasibility |
| V-18 | Walk and record a Wi-Fi coverage survey | M | IT | Dead-zone map |

⚠️ **V-14 is the single highest-value measurement in the project.** A tape
measure and an afternoon could determine that vertical transit is permanently
out of scope.

⚠️ **V-14 through V-18 require access to the building.** That access must be
arranged with the school. This plan does not assume it.

---

## Priority 4 — Requires written permission before anything can be attempted

| ID | Verification | Type | Approver | Depends on |
|---|---|---|---|---|
| V-19 | **Read-only DDS connection to the robot** — subscribe to state topics, no actuation | T | School + robot custodian | Gate 0, Gate 1 |
| V-20 | Measure the real camera frame rate at the stationary robot | T | Robot custodian | V-19 |
| V-21 | Confirm the LiDAR point cloud flows; record rate and size | T | Robot custodian | V-19 |
| V-22 | **Battery endurance characterisation** over a charge–discharge cycle | T | Robot custodian + facilities | V-19, Gate 6 |
| V-23 | Carrier tilt-test at 45° | T | None — bench test, no robot | Gate 0 |
| V-24 | Supervised low-speed velocity test in a cleared area | T | School + insurer | V-19, Gate 8 |
| V-25 | Emergency stop and remote-authority test | T | School | Gate 8 |
| V-26 | Supervised stair test, unloaded, single step first | T | School + insurer + manufacturer | V-14, Gate 4 |
| V-27 | Marker detection testing with the robot camera | T | Robot custodian | V-19, marker approval |

⚠️ **V-19 is the first action involving the physical robot, and it actuates
nothing.** It exists to prove the network and the toolchain. It cannot be
attempted before Gate 0 and Gate 1 are passed.

---

## Priority 5 — Requires manufacturer or service provider confirmation

| ID | Verification | Type | Approver | Note |
|---|---|---|---|---|
| V-28 | **Manufacturer written position on lift integration** | A | Johnson Lifts via service provider | Includes the button-actuator question, where the expected answer is no |
| V-29 | Manufacturer written position on the proposed stair use | A | Unitree | ⛔ Do not assume permitted |
| V-30 | Service provider site assessment, if vertical transit is pursued | T | Service provider | Requires Gate 4 |
| V-31 | Unitree written confirmation of EDU restrictions and external device connection | A | Unitree | ⚠️ Requires issue by the repository owner |

⚠️ **V-28 includes asking whether an external button actuator is permitted. We
expect the answer to be no, and we record it as such.** The question is asked so
that the position is documented rather than assumed.

---

## Verification dependencies

```
V-08 (simulation) ──► validates software stack ──┐
                                                     │
V-14 (risers) ──► decides stairs ──┐               │
                                    │               ▼
V-09 (EyeRIS) ──► decides lifts ───┼──► scope decision ──► architecture
                                    │                            │
V-13 (teacher needs) ───────────────┘                            │
                                                                 ▼
V-11, V-19, V-22 ──► measurements ──► thresholds ──► Gate 6 ──► Gate 8 ──► Gate 9
```

⚠️ **V-08, V-14 and V-09 should start in parallel and immediately.** They have no
dependencies on each other, and between them they determine the scope of
everything else.

---

## What cannot be verified, and what to do instead

| Cannot be verified | Why | Do instead |
|---|---|---|
| Whether the robot *will* work in a school corridor | Requires a live demonstration | Plan the first demonstration as a *measurement*, not a claim |
| Whether teachers *will* use it | Requires real behaviour | Ask them; design for a small number of committed users |
| Whether a lift service provider *will* permit integration | Requires their decision | Default to human escort and design for it |
| Whether congestion routing *improves* anything | Requires comparative measurement | Measure it during the pilot and publish the result, whatever it is |
| Whether the project will *succeed* | Not knowable in advance | Define success criteria in advance so the question becomes answerable |

⚠️ **These are not failures of the plan. They are the boundaries of what any
feasibility study can establish, and stating them plainly is more useful than
projecting false confidence.**

---

## Current verification status

| Type | Items | Complete | Requires permission |
|---|---|---|---|
| R — Research | 8 | 0 (some findings already recorded) | 0 |
| A — Answer | 5 | 0 | 0 (requests not yet sent) |
| M — Measurement | 5 | 0 | Building access |
| T — Test | 9 | 0 | School, insurer, manufacturer |
| **Total** | **27** | **0** | |

⚠️ **Zero verifications have been performed.** The first four research items
are the only ones that can begin without anyone's permission, and they should.