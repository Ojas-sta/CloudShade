# SHANA — Go/No-Go Gates

**These gates prevent implementation from proceeding prematurely.**

A gate may be marked **PASSED**, **BLOCKED**, or **NOT APPLICABLE**. A gate is
**never** marked passed without evidence. This rule is absolute.

---

## Status values

| Status | Meaning |
|---|---|
| `PASSED` | All criteria met, with evidence attached |
| `BLOCKED` | Cannot proceed; the stated blocker must be resolved first |
| `NOT APPLICABLE` | Deliberately excluded from scope, with a recorded reason |
| `PENDING` | Not yet assessed *(the current state of every gate)* |

⚠️ **Every gate is currently `PENDING`. Nothing has been passed. No permission
has been granted.**

---

## Gate authority

⚠️ **A gate may only be declared by a named school authority, not by the project
team.**

| Role | Authority |
|---|---|
| School Principal or delegate | Gates 0, 7, 9, 10 |
| IT / Network Administrator | Gates 2, 3 (network components) |
| Robot custodian | Gates 1, 6 |
| Facilities / Safety Officer | Gates 4, 7, 8 |
| Data Fiduciary | Gates 3, 7 |
| Lift service provider | Gate 4 (lift components) |
| Exhibition organiser | Gates 9, 10 |

⚠️ **Authority to declare `NOT APPLICABLE` is equivalent to authority to declare
`PASSED`.** A gate cannot be made to disappear to avoid the work it represents.

---

## GATE 0 — School authorisation and project ownership

**Blocks everything.**

| # | Criterion | Evidence required |
|---|---|---|
| 0.1 | Project authorised in writing | Signed authorisation |
| 0.2 | Named project owner | Written record |
| 0.3 | Named robot custodian | Written record |
| 0.4 | Ownership of robot, server and data recorded | Ownership record |
| 0.5 | **Secondary development access authorised in writing** | Signed authorisation |
| 0.6 | Repository ownership decided | Written decision |
| 0.7 | Gate-passing authority assigned to named people | Written decision |
| 0.8 | Explicit **stop criteria** for the project agreed | Written criteria |

**Status: `PENDING`** — *Blocker: none of the above has been requested.*
**Questions:** `SH-GO2-008`, `SH-DEC-016` to `-019`, `SH-BUD-017`, `SH-GIT-001`

---

## GATE 1 — Go2 EDU model, SDK and supported interfaces confirmed

| # | Criterion | Evidence required |
|---|---|---|
| 1.1 | Exact model confirmed as EDU | Nameplate photograph |
| 1.2 | Firmware version recorded, including motion service version | Screenshot |
| 1.3 | Installed options recorded — dock, depth camera, foot sensors | Photographs |
| 1.4 | **Battery type identified** | Label photograph |
| 1.5 | Approved SDK version documented | Written approval |
| 1.6 | A Linux development host identified and approved | Written confirmation |
| 1.7 | **A read-only connection established successfully** | Subscription log, ≥30 min |
| 1.8 | All required topics confirmed present and stable | Log |
| 1.9 | Approved licence recorded for every SDK component | Licence file |

⚠️ **Criterion 1.4 is the most consequential and the most often skipped.** The
battery determines the endurance budget and it differs between variants.

**Status: `PENDING`** — *Blocker: model, firmware and battery unknown.*
**Questions:** `SH-GO2-001` to `-010`, `SH-BAT-001`

---

## GATE 2 — Network and local server feasibility confirmed

| # | Criterion | Evidence required |
|---|---|---|
| 2.1 | Dedicated robot network segment permitted | Written approval |
| 2.2 | Static IP assigned; robot subnet confirmed | Assignment record |
| 2.3 | **UDP multicast permitted** on that subnet | Configuration confirmation |
| 2.4 | Server host approved and specified | Specification |
| 2.5 | Server reachable by teacher devices on the LAN | Firewall review |
| 2.6 | **Server not exposed to any public network** | Written confirmation |
| 2.7 | Core operation confirmed to require no internet | Design confirmation |
| 2.8 | Credential storage policy agreed | Written policy |
| 2.9 | Stable connection demonstrated over ≥2 hours | Connection log |

⚠️ **Criterion 2.3 is a common failure.** Multicast discovery is required by the
robot's protocol and is frequently blocked by default. Discovering this on
demonstration day would stop the project.

**Status: `PENDING`** — *Blocker: nothing has been requested from IT.*
**Questions:** `SH-NET-001` to `-024`, `SH-SRV-001` to `-018`

---

## GATE 3 — CCTV access and privacy permissions confirmed, or the feature excluded

⚠️ **This gate may legitimately be closed by exclusion.** If corridor occupancy
is supplied manually during the pilot, no CCTV access is needed at all — and
that is currently the recommended path.

| # | Criterion | Evidence required |
|---|---|---|
| 3.1 | Data Fiduciary identified | Written record |
| 3.2 | Data Fiduciary has authorised the project in principle | Written approval |
| 3.3 | **Determination made on whether any exemption covers corridor-routing analytics** | Written opinion from a qualified adviser |
| 3.4 | Privacy impact assessment completed, if required | Assessment |
| 3.5 | CCTV analytics access authorised | Written authorisation |
| 3.6 | Design confirmed to compute **anonymous counts only** | Design confirmation |
| 3.7 | Design confirmed to perform **no facial or biometric identification** | Design confirmation |
| 3.8 | Design confirmed to **discard frames**, storing no raw video | Design confirmation |
| 3.9 | Design confirmed never to decode audio | Design confirmation |
| 3.10 | Design confirmed never to produce automated misconduct or attendance output | Design confirmation |
| 3.11 | Retention policy for derived data agreed | Written policy |
| 3.12 | Low-confidence and feed-failure behaviour defined as `UNKNOWN`, never `CLEAR` | Design confirmation |
| 3.13 | **CCTV confirmed not to be the sole obstacle-avoidance system** | Design confirmation |

**Status: `PENDING`** — *Blocker: no privacy determination has been made.*
**Questions:** `SH-CCTV-001` to `-024`, `SH-PRIV-001` to `-022`

---

## GATE 4 — Elevator and stair transition feasibility confirmed or excluded

⚠️ **The expected outcome of this gate is EXCLUSION.** The default and
recommended position is that neither stairs nor lifts are in scope for the
initial pilot. Exclusion is a valid, complete way to pass this gate.

### If exclusion is chosen

| # | Criterion | Evidence required |
|---|---|---|
| 4.1 | Stair riser heights measured | Measurements |
| 4.2 | **A single-floor route to every required destination confirmed** | Route survey |
| 4.3 | Stairs formally excluded from scope | Decision record |
| 4.4 | Lifts formally excluded from scope | Decision record |
| 4.5 | No required route depends on a vertical transition | Route verification |

### If inclusion is sought (requires ALL of the following)

| # | Criterion | Evidence required |
|---|---|---|
| 4.6 | Riser heights confirmed **≤160 mm** on every candidate route | Measurements |
| 4.7 | **Manufacturer written confirmation** that the proposed use is permitted | Written response |
| 4.8 | School written approval for any stair test | Written approval |
| 4.9 | Insurer written approval | Written confirmation |
| 4.10 | Supervised unloaded stair test completed, **including forward descent** | Test record |
| 4.11 | Lift identity, dimensions and standard confirmed | Documentation |
| 4.12 | **Manufacturer written confirmation of a sanctioned interface** | Written response |
| 4.13 | Electrical Inspector permission obtained, if required by State rules | Written permission |
| 4.14 | Maintenance contract compatibility confirmed | Written confirmation |
| 4.15 | Insurance, warranty and certification impact confirmed | Written confirmation |
| 4.16 | Approved fault and emergency workflow for robot-in-lift agreed | Written procedure |
| 4.17 | **Human escort protocol agreed and staffed** | Written agreement |

⚠️ **Criteria 4.12–4.15 are legally constrained.** Any physical modification is
prohibited without Electrical Inspector permission. Do not plan around an
assumption that permission will be granted.

**Status: `PENDING`** — *Blocker: nothing has been measured or requested.*
**Questions:** `SH-LIFT-001` to `-026`, `SH-STAIR-001` to `-020`

---

## GATE 5 — Mapping, routes and checkpoints documented and verified

| # | Criterion | Evidence required |
|---|---|---|
| 5.1 | Route graph created from a physical survey | Annotated floor plan |
| 5.2 | Every segment authorised for robot access | Authorised segment list |
| 5.3 | Fire and evacuation routes excluded | Fire plan confirmation |
| 5.4 | Corridor widths and floor surfaces surveyed | Survey table |
| 5.5 | Permanent and recurring obstructions recorded | Survey |
| 5.6 | Checkpoints identified, authorised, secure, with power | Confirmation |
| 5.7 | **Every destination physically walked and approved by a named person** | Verification records |
| 5.8 | Every destination carries `verified_by` and `verified_date` | Records |
| 5.9 | **Floor identification mechanism defined**, or the project is single-floor | Design confirmation |
| 5.10 | Floor-transition behaviour defined as human-confirmed | Design confirmation |
| 5.11 | Marker installation approved and recorded | Approval + inventory |
| 5.12 | **Relocation event mechanism defined** — manual handling invalidates pose | Design confirmation |
| 5.13 | Unknown-area behaviour defined: stop, never explore | Design confirmation |
| 5.14 | Localisation-loss behaviour defined and safe | Design confirmation |

⚠️ **Criterion 5.9 cannot be satisfied with stock sensors.** The Go2 has no
altitude, pressure, or barometer. Single-floor operation avoids the problem
entirely; multi-floor operation requires added infrastructure.

**Status: `PENDING`** — *Blocker: no survey has been carried out.*
**Questions:** `SH-MAP-001` to `-024`, `SH-CONG-005` to `-010`

---

## GATE 6 — Battery and safe-stop behaviour understood

| # | Criterion | Evidence required |
|---|---|---|
| 6.1 | Battery telemetry source confirmed | Source citation |
| 6.2 | State-of-charge accuracy measured | Calibration record |
| 6.3 | **Battery endurance measured under representative load** | Test data |
| 6.4 | Reserve threshold established **by measurement** | Threshold record |
| 6.5 | Task-completion feasibility check defined | Design |
| 6.6 | Return-feasibility calculation defined and conservative | Design |
| 6.7 | Checkpoint selection defined on reachability and safety, not distance | Design |
| 6.8 | **Low battery may not trigger an unverified stair, lift, or closed-corridor route** | Design confirmation |
| 6.9 | Battery-telemetry-loss behaviour defined | Design |
| 6.10 | Official charging procedure obtained | Manufacturer documentation |
| 6.11 | Battery handling and quarantine procedures approved | Written procedures |
| 6.12 | **Safe-stop behaviour defined and physically tested** | Test record |
| 6.13 | Auto-recovery and `RecoveryStand()` confirmed disabled with payload | Design confirmation |
| 6.14 | Confirm no autonomous return-to-dock is assumed | Design confirmation |

⚠️ **Criterion 6.4 must come from measurement.** Manufacturer figures span 1–5
hours across four official documents. **Do not invent a threshold.**

**Status: `PENDING`** — *Blocker: nothing has been measured.*
**Questions:** `SH-BAT-001` to `-022`

---

## GATE 7 — Hazard and crowd-management design reviewed

| # | Criterion | Evidence required |
|---|---|---|
| 7.1 | Named authority to close a corridor or declare a hazard | Authority list |
| 7.2 | Staff closure workflow agreed and testable | Process |
| 7.3 | **No automatic resumption** after a closure is cleared | Design confirmation |
| 7.4 | Hazard state machine defined, including `UNKNOWN` and `SENSOR_FAILURE` | Design |
| 7.5 | Confirmed that camera-derived hazard detection is **advisory only** | Design confirmation |
| 7.6 | Congestion thresholds defined experimentally | Threshold record |
| 7.7 | Alternate routes verified, or busy routes treated as hard blocks | Route survey |
| 7.8 | Oscillation prevention designed | Design |
| 7.9 | Stale occupancy data handled conservatively | Design |
| 7.10 | No-route-available behaviour resolves to safe-stop and human request | Design |
| 7.11 | Fire alarm and evacuation behaviour agreed | Procedure |
| 7.12 | Occupancy data confirmed useless for individual identification or assessment | Design confirmation |
| 7.13 | Operating speed cap agreed and enforced in software | Policy + design |

**Status: `PENDING`** — *Blocker: no authority has been named.*
**Questions:** `SH-ENV-001` to `-020`, `SH-CONG-001` to `-020`, `SH-BLOCK-001` to `-020`

---

## GATE 8 — Controlled testing plan approved

| # | Criterion | Evidence required |
|---|---|---|
| 8.1 | Controlled testing plan written and approved | Written plan |
| 8.2 | Test location is physically secured and excluded from normal use | Site confirmation |
| 8.3 | Testing scheduled outside student hours | Schedule |
| 8.4 | **Named supervisor present at every test** | Roster |
| 8.5 | Supervisor training completed and evidenced | Training record |
| 8.6 | Emergency stop tested and evidenced | Test record |
| 8.7 | Remote control authority tested before every run | Procedure + record |
| 8.8 | Abort criteria agreed and known to all present | Written criteria |
| 8.9 | Incident and near-miss reporting process in place | Written procedure |
| 8.10 | Payload tilt test completed before first robot contact | Test record |
| 8.11 | Insurance confirmed to cover testing | Written confirmation |
| 8.12 | Rollback: how testing is stopped and the robot secured | Procedure |

**Status: `PENDING`** — *Blocker: no testing has been proposed or approved.*
**Questions:** `SH-SAFE-001` to `-026`, `SH-STAIR-016`, `SH-STAIR-017`, `SH-ENV-019`

---

## GATE 9 — Initial pilot acceptance criteria approved

| # | Criterion | Evidence required |
|---|---|---|
| 9.1 | Pilot scope written and approved — destinations, routes, hours | Written scope |
| 9.2 | **Approved item list agreed, with absolute prohibitions** | Written policy |
| 9.3 | Approved destination list agreed | Written list |
| 9.4 | Measurement protocol agreed **before** data collection | Written protocol |
| 9.5 | Success, failure and intervention definitions agreed | Written definitions |
| 9.6 | Required number of consecutive successful runs agreed | Agreed criterion |
| 9.7 | **Acceptance criteria confirmed not to be waived under time pressure** | Written confirmation |
| 9.8 | Lunch and out-of-hours shutdown procedure agreed | Written procedure |
| 9.9 | Battery charging and attendance arrangement agreed | Written procedure |
| 9.10 | Human intervention and escalation process agreed | Written procedure |
| 9.11 | Data retention and audit agreed | Written policy |
| 9.12 | Stop authority at every stage unambiguous | Written confirmation |

⚠️ **Criterion 9.7 matters more than it appears.** Without it, every deadline
becomes a reason to lower the standard.

**Status: `PENDING`** — *Blocker: pilot scope not defined.*
**Questions:** `SH-DEC-001` to `-006`, `SH-DEL-001` to `-022`, `SH-TIME-001` to `-020`

---

## GATE 10 — Student-facing demonstration separately authorised

⚠️ **Gate 10 cannot be satisfied by passing Gates 0–9.** It is a separate
authorisation. Student presence changes the risk profile fundamentally.

| # | Criterion | Evidence required |
|---|---|---|
| 10.1 | Gates 0–9 all passed | Gate records |
| 10.2 | **Separate written authorisation for student presence** | Signed authorisation |
| 10.3 | Venue surveyed and confirmed safe | Site survey |
| 10.4 | Demonstration plan approved, with exclusions agreed | Written plan |
| 10.5 | Exclusion zone arrangement agreed with organisers | Written plan |
| 10.6 | Named event supervisor, plus trained backup | Roster |
| 10.7 | Abort criteria agreed **before** the day | Written criteria |
| 10.8 | Incident and first-aid plan agreed | Written plan |
| 10.9 | Fallback demonstration prepared and tested | Test record |
| 10.10 | Teachers informed, and **consent or objection recorded** | Record |
| 10.11 | **Ten consecutive successful supervised test runs completed** | Test records |
| 10.12 | Confirmation that no stairs, lifts, or live items are involved | Written confirmation |

⚠️ **Criterion 10.10 is frequently forgotten.** Teachers affected by a
demonstration in their space should have been asked, and a refusal should be
honoured.

**Status: `PENDING`** — *Blocker: Gates 0–9 not passed.*
**Questions:** `SH-EXPO-001` to `-020`, `SH-DEL-020`, `SH-SAFE-026`

---

## Gate dependency chain

```
GATE 0 ──► everything
   │
   ▼
GATE 1 ──► GATE 2 ──┬──► GATE 5 ──┬──► GATE 9 ──► GATE 10
   │                │             │
   │                └──► GATE 3 ──┘
   ▼
GATE 6 ──────────────► GATE 5
   │
   ▼
GATE 4 (independent; may be excluded)

GATE 7 ──────────────► GATE 8 ───► GATE 9
```

⚠️ **Gate 4 can complete as `NOT APPLICABLE` at any time.** Excluding stairs and
lifts unblocks the project immediately and is the recommended path.

---

## Current gate summary

| Gate | Subject | Status | Principal blocker |
|---|---|---|---|
| 0 | School authorisation | `PENDING` | Nothing requested |
| 1 | Robot model, SDK, interfaces | `PENDING` | Robot identity unknown |
| 2 | Network and server | `PENDING` | Nothing requested from IT |
| 3 | CCTV and privacy | `PENDING` | No privacy determination |
| 4 | Elevator and stairs | `PENDING` | Nothing measured or requested |
| 5 | Mapping and routes | `PENDING` | No survey carried out |
| 6 | Battery and safe stop | `PENDING` | Nothing measured |
| 7 | Hazard and crowd design | `PENDING` | No authority named |
| 8 | Controlled testing plan | `PENDING` | No plan proposed |
| 9 | Pilot acceptance criteria | `PENDING` | Scope undefined |
| 10 | Student-facing demonstration | `PENDING` | Gates 0–9 not passed |

**0 of 11 gates passed. 102 of 102 permissions requested: 0 granted.**

---

## A note on pressure

Every gate in this document exists because there will come a moment when
someone asks for a gate to be skipped — a deadline, an exhibition, a
demonstration opportunity.

⚠️ **The correct response to that moment is to delay the activity, not to lower
the standard.** A delayed demonstration is a minor inconvenience. A gate skipped
on the way to a demonstration is an incident waiting for an audience.