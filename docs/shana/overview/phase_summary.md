# SHANA — Phase 0 Summary

**Feasibility and requirements discovery · 2026-10-10**

> **Nothing has been built, connected to, commanded, or demonstrated.** This
> document summarises what a documentation-only phase established.

---

## 1. What is known

| # | Fact | Class |
|---|---|---|
| 1 | Only **EDU-and-above** Go2 variants may be programmed. Verbatim: *"Only EDU or above versions can be developed, other versions such as PRO and AIR versions do not open the secondary development interface."* | `VERIFIED` |
| 2 | Development is **wired Ethernet only**, Ubuntu 20.04/22.04, on `192.168.123.0/24` | `VERIFIED` |
| 3 | The Go2 EDU is a quadruped: **12 DOF, ~15 kg, ~8 kg rated payload, 160 mm max step height** | `VERIFIED` |
| 4 | **Battery telemetry is on `rt/lowstate` → `LowState.bms_state`** — `SportModeState` has **no battery fields at all** | `VERIFIED` |
| 5 | **At low battery the robot crouches within 10 minutes.** ⚠️ *A crouch on a staircase is a fall.* | `VERIFIED` |
| 6 | 🔴 **The robot has no altitude, pressure, or barometer sensor anywhere.** It cannot determine which floor it is on | `VERIFIED (negative)` |
| 7 | 🔴 **There is no autonomous return-to-dock.** No charging service appears in the documented service list | `VERIFIED (negative)` |
| 8 | 🔴 **The stair gait setter `SwitchGait` was removed** from the current SDK (firmware ≥1.1.6) | `VERIFIED` |
| 9 | **No sanctioned third-party lift interface exists for Johnson lifts.** WATCH IoT is monitoring-only | `VERIFIED (negative)` |
| 10 | **Johnson EyeRIS** — a QR-based contactless call system — is the only sanctioned lift path found. Retrofit, QR type, and robot provisioning all unknown | `VERIFIED` / `NOT FOUND` |
| 11 | ⚠️ **Lift modification is prohibited by statute** without Electrical Inspector written permission | `VERIFIED` |
| 12 | **The school, as occupier, was held 5% liable** in a lift fatality — *Otis v. Rashmi Handa*, Supreme Court of India, 29 July 2026 | `VERIFIED` |
| 13 | Manufacturer requires **≥2 m** from crowds and states the robot is **not for under-14s** | `VERIFIED` |
| 14 | **No CCTV analytics permission exists.** Neither the DPDP Act 2023 nor the DPDP Rules 2025 mentions CCTV, surveillance, biometrics or facial recognition in their text; operative sections from ~May 2027 | `VERIFIED (negative)` |
| 15 | **CBSE 4.7.10 already mandates audio-visual CCTV in corridors** (21 Jul 2025) — the stream carries audio | `VERIFIED` |

---

## 2. What is unknown

| # | Unknown | Why it blocks | Answered by |
|---|---|---|---|
| 1 | **Written authorisation for development access** | ⛔ **Everything** | Administration |
| 2 | Exact robot model, firmware, installed options | All technical work | Robot custodian |
| 3 | 🔴 **Which battery is fitted — 8,000 or 15,000 mAh** | Halves or doubles the endurance budget | Read the label |
| 4 | Whether a robot network segment is permitted | The robot link | IT |
| 5 | 🔴 **Actual stair riser heights** | May eliminate stairs permanently | Tape measure |
| 6 | Whether a single-floor route exists to needed rooms | May eliminate vertical transit entirely | Facilities + teachers |
| 7 | Whether EyeRIS is retrofittable, passive, robot-provisionable | The only sanctioned lift path | Johnson Lifts |
| 8 | Whether APs and wall markers are permitted | Floor identification | Administration |
| 9 | Whether corridor congestion is needed at all | Removes the entire privacy question | Team decision |
| 10 | Who has authority to declare a gate passed | Governance | Administration |

---

## 3. What is unsafe to assume

| Assumption | Why it is unsafe |
|---|---|
| ⛔ "The robot can identify which floor it is on" | **Verified negative.** Silent wrong-floor navigation |
| ⛔ "The robot can handle stairs" | 160 mm limit vs unknown risers; stair gait removed from SDK; descent is worse |
| ⛔ "The robot can use lifts" | No sanctioned interface; modification prohibited by law |
| ⛔ "A maintenance app means remote control" | Maintenance software is not a third-party API |
| ⛔ "Cameras can tell us if a floor is safe" | RGB and depth cannot determine friction |
| ⛔ "A busy corridor means an alternate exists" | Not established. Without one, it is a hard block |
| ⛔ "Radio telemetry is more reliable than Wi-Fi indoors" | Peer-reviewed evidence points the other way |
| ⛔ "Battery telemetry comes with the motion topic" | It does not |
| ⛔ "The robot can return to its dock" | No such service exists |
| ⛔ "Owning an EDU means development is permitted" | Capability is not permission |
| ⛔ "Standard G1/Go2 documentation applies to the EDU" | Options differ; verify by inspection |
| ⛔ "Our manual contains Go2 safety figures" | The G1 manual contains a **quadruped section** — cross-contamination risk |

---

## 4. What requires permission

**102 permissions. 0 granted.** Full list:
`docs/shana/administration/permission_checklist.md`

The five that block the most:

| Permission | From |
|---|---|
| **Secondary development authorisation** | Administration |
| **Dedicated network segment** | IT |
| **Server room allocation** | Administration |
| **Stair and lift measurement access** | Facilities + service provider |
| **Separate student-facing demonstration authorisation** | Administration |

---

## 5. What requires a vendor or technician

| Question | From |
|---|---|
| Is EyeRIS retrofittable; is its QR passive or active; can it be provisioned to a robot | Johnson Lifts |
| Does any sanctioned third-party lift interface exist | Lift service provider |
| Is a physical button actuator permitted | Lift service provider — **expected answer: no** |
| Is the proposed stair use permitted | Unitree |
| Does a school need a 2026 radio possession authorisation | DoT WPC |

---

## 6. What can be investigated in parallel, without permission

| # | Activity |
|---|---|
| 1 | 🔴 **Run the Go2 simulator.** Same DDS channels, CPU-only. Validates the entire software stack at zero risk |
| 2 | Read SDK source; document every interface used |
| 3 | Read the "Aruco Recharge Service Interface" documentation |
| 4 | Build a radio link-budget plan and decide whether LoRa is justified |
| 5 | Compare mmWave presence sensing against CCTV analytics |
| 6 | Draft architecture decisions with reversal conditions |

⚠️ **Item 1 is the highest-value action available and requires nobody's
permission. It should start immediately, in parallel with the permission
process.**

---

## 7. The minimum viable and safe first demonstration

**One floor. One fixed, verified route. Supervised. Demonstration payloads.**

```
1. Teacher submits a request from an allow-list
2. System validates; rejects with a specific reason if any check fails
3. Robot readiness and battery confirmed
4. Robot navigates the pre-verified route
5. Robot holds; a person loads and confirms
6. Robot navigates to the destination
7. Robot holds; a person confirms receipt
8. Dashboard records duration, battery, interventions, and any failure
```

**Explicitly excluded:** stairs · lifts · doors · exploration · multiple floors ·
live school items · unsupervised operation · any free-typed destination.

---

## 8. What must be proven before any student-facing operation

| # | Requirement | Gate |
|---|---|---|
| 1 | Written authorisation and named owners | Gate 0 |
| 2 | Confirmed hardware and a stable read-only connection | Gate 1 |
| 3 | Permitted, isolated, tested network | Gate 2 |
| 4 | Privacy determination | Gate 3 |
| 5 | Vertical transit excluded or proven feasible | Gate 4 |
| 6 | Verified maps, routes, checkpoints | Gate 5 |
| 7 | Measured battery behaviour and tested safe stop | Gate 6 |
| 8 | Reviewed hazard and crowd design | Gate 7 |
| 9 | Approved testing plan; 10 consecutive successful runs | Gates 8, 9 |
| 10 | 🔴 **Separate written authorisation for student presence** | Gate 10 |

---

## 9. Features to postpone or exclude

| Feature | Decision | Reason |
|---|---|---|
| Stairs | ⛔ **Exclude** | Not established; 160 mm limit; stair gait removed from SDK |
| Lifts | ⛔ **Exclude** | No sanctioned interface; modification prohibited |
| Multi-floor | ⛔ **Exclude** | No floor-identification sensor |
| CCTV analytics | ⛔ **Defer** | Privacy determination required; a manual report may suffice |
| mmWave occupancy | ⛔ **Defer** | Try manual first |
| Radio / LoRa | ⛔ **Defer** | No demonstrated problem |
| Object manipulation | ⛔ **Never** | No arm |
| Doors | ⛔ **Exclude** | Not tested, not required |
| Natural-language requests | ⛔ **Defer** | Manual workflow first |
| Outdoors | ⛔ **Never** | Not weatherproof |

---

## 10. The next 10 actions

| # | Action | Owner | Blocks |
|---|---|---|---|
| 1 | 🔴 **Measure every stair riser** — a tape measure and an afternoon may eliminate stairs permanently | Facilities | Stage 2/3 |
| 2 | 🔴 **Email Johnson Lifts** — three questions about EyeRIS. One email, longest lead time, potentially the highest-value answer in the project | Repository owner → service provider | Stage 3 |
| 3 | 🔴 **Ask teachers which rooms they actually need** — this may shrink the project to a single floor, which would be a good outcome | Teachers → administration | Scope |
| 4 | 🔴 **Photograph the robot and read the battery label** | Robot custodian | Endurance planning |
| 5 | **Request written development authorisation** | Administration | **Everything** |
| 6 | **Run the Go2 simulator and put the intended stack on it** — requires nobody's permission | Team, now | Software validation |
| 7 | **Request the network segment and server room** | IT, administration | All technical work |
| 8 | **Survey corridors, floor surfaces, widths, and obstructions** | Facilities | Route graph |
| 9 | **Request the privacy and Data Fiduciary determination** | Administration | Occupancy features |
| 10 | **Agree the safety case, emergency stop, and spotter arrangements** | Administration + safety officer | Gate 8 |

---

## 11. Project state

| | |
|---|---|
| Questions written | **434** |
| Questions answered | **0** |
| Permissions requested | **102** |
| Permissions granted | **0** |
| Gates defined | **11** |
| Gates passed | **0** |
| Verifications performed | **0** |
| Assumptions unverified | **30 of 34** |
| Documentation files | **80** |

---

## 12. The honest closing position

SHANA is a **genuinely useful idea on hardware that is well suited to delivery
and poorly suited to manipulation.**

The robot can carry items reliably. It cannot pick them up. It will need help at
transitions it cannot yet handle safely. It operates among children, which sets a
very high bar.

⚠️ **The most likely outcome of the next phase is a smaller, single-floor system
that works** — rather than a three-block system that does not. That would be a
successful outcome, not a disappointing one.

⚠️ **Nothing in this repository is operational. Every capability described is
proposed. The project's next step is not building — it is answering ten
questions that only the school, its staff, and one lift manufacturer can
answer.**