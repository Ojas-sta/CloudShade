# SHANA — Proposed System Architecture

**Phase 0 · Documentation only · 2026-10-10**

This document describes the proposed end-to-end system, separates what is known
from what is unknown, defines component responsibilities, and compares
alternative architectures. It does **not** commit to a final architecture —
several decisions are blocked on answers from the school (see
`open_questions_register.md`).

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in official manufacturer documentation or source code, quoted with URL |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | Manufacturer documents it; we have not confirmed it on the school's actual unit |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No manufacturer claim; our engineering judgement |
| ❓ **UNKNOWN** | No evidence found. Not assumed either way |
| ⛔ **NOT PERMITTED / NOT RECOMMENDED** | Prohibited by law, by manufacturer terms, or by our own safety judgement |

---

## 1. Headline: the aspiration is not the first deliverable

The stated aspiration — teachers request deliveries across three blocks and four
floor levels, with SHANA handling scheduling, navigation, progress reporting and
confirmation — is **not achievable in a first phase**, and several parts of it are
not achievable at all without approvals we do not yet have.

Four findings drive the architecture:

| # | Finding | Consequence |
|---|---|---|
| **1** | **Stair climbing is not established.** The Go2's documented limit is a 16 cm step. Indian school risers are typically 150–180 mm. A 160 mm riser consumes the entire margin. Unitree's own manual **forbids** operation on "complex terrain." | Multi-floor reachability is **blocked** pending measurement and testing |
| **2** | **The stair gait setter was removed from the SDK.** `SwitchGait(3\|4)` — the only documented stair-specific API — existed on firmware **< 1.1.6** and is **absent** from the current Motion Services Interface V2.0. | Even if a staircase is passable, **we would be relying on an emergent property of `ClassicWalk`**, not a controllable stair mode |
| **3** | **The robot cannot know which floor it is on.** Every Go2 message field was enumerated: there is **no altitude, no pressure, no barometer** anywhere on the platform. | Multi-floor operation **requires added infrastructure** — a unique visual or radio marker per floor |
| **4** | **There is no autonomous return-to-dock.** No charging or dock service appears in the documented Go2 service list. | Recovery is **human-assisted by design**, which is acceptable, but must be architected rather than assumed |

**Therefore the architecture is staged.** Each stage is independently useful and
independently demonstrable. No stage depends on an unverified capability of a
later one.

---

## 2. Staged deployment model

```
STAGE 0 ── OBSERVE          Single floor · no actuation · no autonomy
   │                       Proves: network, telemetry, power, school approval
   │                       Value: safety case + baseline data
   ▼
STAGE 1 ── SINGLE FLOOR     One floor · human-docked · supervisor nearby
   │                       Proves: navigation, delivery loop, teacher workflow
   │                       Value: a complete, repeatable demonstration
   ▼
STAGE 2 ── VERTICAL         Multi-floor · human-assisted transit only
   │                       Proves: floor identification, recovery protocol
   │                       Value: cross-floor delivery with an escort
   ▼
STAGE 3 ── MULTI-BLOCK      Cross-block · CONTINGENT
                           Requires: stairs pass + lift permission + site survey
```

**Stage 1 is the first thing that could be called a delivery system. Stages 2–3
are extensions that must each re-earn permission.**

⚠️ **Design principle honoured throughout: a checkpoint's last-known position is
not the robot's current position.** See §7.4.

---

## 3. Known facts

### 3.1 Hardware (partially known)

| Fact | Label | Source |
|---|---|---|
| School owns a **Unitree Go2 EDU** | ✅ | Brief, confirmed |
| Go2 EDU: 12 DOF, ~15 kg, 70×31×40 cm standing | ✅ | `support.unitree.com/home/en/developer/about_Go2` |
| Rated payload ≈8 kg (max ~12 kg) | ✅ | `unitree.com/go2` |
| Max step/climb height **16 cm** | ✅ | `about_Go2`; corroborated by User Manual |
| Max slope 40° | ✅ | `about_Go2` |
| Built-in **4D LiDAR L2**, 360°×96°, 21,600 samples/s, 0.05 m min | ✅ | `about_Go2` |
| LiDAR publishes on DDS: `rt/utlidar/cloud`, `cloud_deskewed`, `height_map_array`, `range_info` | ✅ | Unitree `LiDAR_service` |
| Secondary development is **EDU-only** | ✅ | Unitree developer FAQ, verbatim |
| Go2 EDU gets **15,000 mAh** battery, 1 h15 fast charge | ✅ | `about_Go2` battery table |
| EDU has **depth camera**, **foot force sensors**, **charging pile support**, **high-compute module option** | ✅ | `unitree.com/go2` matrix |

### 3.2 Software interfaces (known and verified)

| Fact | Label | Source |
|---|---|---|
| Transport: **Eclipse Cyclone DDS**; domain 0; explicit NIC argument | ✅ | `unitree_sdk2` source |
| **Wired Ethernet required.** Ubuntu 20.04/22.04 host. | ✅ | Unitree FAQ |
| Robot IP `192.168.123.161`; host `192.168.123.222` on `192.168.123.0/24` | ✅ | Unitree Quick Start |
| `SportClient` — 39–41 methods, `Move(vx,vy,vyaw)` locomotion | ✅ | `sport_client.hpp` |
| `ObstaclesAvoidClient` — `Move(x,y,yaw)`, `MoveToAbsolutePosition` | ✅ | Unitree `ObstaclesAvoidClient` doc |
| `slam_operate` service exists — **EDU + expansion dock + separately purchased MID-360/XT16 only** | ✅ | `SLAM and Navigation_service`, verbatim |
| **Battery telemetry is on `rt/lowstate` → `LowState.bms_state`, NOT on `SportModeState`** | ✅ | `SportModeState.msg` / `BmsState.msg` field lists |
| `BmsState` provides `soc` (%), `current` (signed mA), `cell_vol[15]`, `bq_ntc`, `status`, `cycle` | ✅ | `BmsState.msg` |
| `gait_type` field reports `3=climb stair`, `4=forwardDownStair` | ✅ | `High_motion_control`; `SportModeState_.hpp` |
| Low battery → **"will automatically crouch down within 10 minutes"** | ✅ | `about_Go2` pilot-lamp table |
| Manual: **"it is forbidden to use the robot… where the terrain is complicated… steps higher than 16 cm"** | ✅ | Go2 User Manual §9 |
| Manual: **"not intended for use by persons under the age of 14"** | ✅ | Go2 User Manual |
| Manual: keep **2 m safe distance** from obstacles, crowds, water | ✅ | Go2 User Manual |
| Manual: *"Hand may be pinched at the motion joints, such as at the knee joint"* | ✅ | Go2 User Manual |
| Manual: *"not currently recommended that robot walk stairs higher than 16cm"* | ✅ | Go2 User Manual |

### 3.3 School environment (from brief — ⚠️ not yet verified on site)

⚠️ **None of the following has been surveyed.** Every item is a brief input, not a
measured fact.

- Three blocks A/B/C; 3–4 levels; two Johnson elevators (C and B Block)
- C-to-B indoor transition available **only via basement and ground floor**
- B-to-A routes **unsurveyed**
- Wired LAN for smartboards exists; **no confirmed school-wide Wi-Fi**
- No confirmed NAS or local application server
- AI Lab (CP-C3, C Block 3rd floor) proposed primary base; Makerspace Lab (CP-A1, A Block 1st floor) secondary
- Stair riser heights, corridor widths, floor surfaces — **unknown**

---

## 4. Unknowns that block architecture

| # | Unknown | Blocks | Who can answer |
|---|---|---|---|
| U1 | **Which Go2 options are installed?** Expansion dock? Depth camera? 15,000 mAh or 8,000 mAh battery? | Navigation route choice, battery budget, sensing | ROB-01 |
| U2 | **Is school Wi-Fi available to the robot, and can it be extended?** | **Everything.** DDS is Ethernet-only | NET-01…05 |
| U3 | **Actual stair riser heights and floor surfaces** | Stage 2/3 reachability | FAC-01 |
| U4 | **Is autonomous or even supervised lift use permitted?** | Stage 3 | ELV-01…04 |
| U5 | **Does the school permit installing APs / markers / electronics?** | Floor ID, coverage | ADM-04, NET-06 |
| U6 | **Is there a room where Wi-Fi does not reach?** | Whether LoRa is justified at all | NET-05 |
| U7 | **Actual elevator car and door dimensions** | Robot-elevator protocol | ELV-02 |
| U8 | **Budget and installation permissions** | Everything | ADM-01…03 |
| U9 | **Whether teachers will actually use the system** | Value proposition | TCH-01…04 |

---

## 5. Component architecture

### 5.1 System diagram

```
┌───────────────────────────────────────────────────────────────────────┐
│  TEACHER FACES                                                       │
│  Browser (any device on school network) — no installation             │
│  • Submit delivery request (destination from allow-list)              │
│  • See live robot status                                              │
│  • Confirm receipt                                                    │
└───────────────────────────────┬───────────────────────────────────────┘
                                │ HTTPS (local, no internet)
┌───────────────────────────────▼───────────────────────────────────────┐
│  LOCAL APPLICATION SERVER  (AI Lab or Makerspace Lab)                 │
│  🔵 All components ours. Python + FastAPI + SQLite + WebSocket.       │
│                                                                       │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐  │
│  │ Auth & Roles │ │ Room Directory│ │ Task Manager │ │ Delivery FSM │  │
│  │ teacher /    │ │ stable IDs,   │ │ queue,       │ │ deterministic│  │
│  │ admin /      │ │ allow-list,   │ │ scheduling,  │ │ state machine│  │
│  │ robot-admin  │ │ floor + block │ │ deadlines    │ │              │  │
│  └──────────────┘ └──────────────┘ └──────┬───────┘ └──────┬───────┘  │
│                                          └────────┬───────┘          │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐  │
│  │ Task         │ │ Safety       │ │ Event Log &  │ │ Network &    │  │
│  │ Validator    │ │ Supervisor   │ │ Metrics      │ │ Robot Health │  │
│  │ (pure fn)    │ │ (independent)│ │ (SQLite)     │ │ (watchdogs)  │  │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘  │
└───────┬──────────────────────────────────────────┬────────────────────┘
        │ ROS 2 (Humble) over LAN                  │ DDS over ETHERNET
        │                                          │ (dedicated NIC)
┌───────▼──────────────────────┐      ┌────────────▼───────────────────┐
│ NAVIGATION & PERCEPTION     │      │  GO2 EDU                        │
│ (runs on server host)       │      │  • sport / mcf service (closed) │
│ • slam_toolbox / FAST-LIO2 │      │  • obstacles_avoidance          │
│ • AMCL (per-floor map)      │      │  • unitree_lidar (L2 driver)    │
│ • Nav2 (planner + control)  │      │  • LiDAR → rt/utlidar/* on DDS  │
│ • Velocity bridge → SDK2    │      │  • battery → rt/lowstate        │
│ • Floor-ID classifier 🔨    │      │  • gait_type → rt/sportmodestate│
│ • ArUco/AprilTag reader 🔨  │      └─────────────────────────────────┘
└───────┬──────────────────────┘
        │ 🔵 optional, late-stage, contingency only
┌───────▼──────────────────────┐
│ LoRa BACKUP HEARTBEAT       │   ← NOT the primary link. NOT control.
│ robot ESP32/SX1262 → gateway│      See §9.
│ ~35 bytes / 60 s            │
└──────────────────────────────┘
```

### 5.2 Component responsibilities

| # | Component | Responsibility | Runs on | Depends on |
|---|---|---|---|---|
| 1 | User interface | Request submission, live status, receipt confirmation | Browser | Server |
| 2 | Request & task manager | Queue, scheduling, deadline tracking, notification | Server | DB |
| 3 | Task validator | **Pure function.** Rejects anything not allow-listed | Server | Directory |
| 4 | Robot state monitor | Subscribes `rt/lowstate`, `rt/sportmodestate`; health & fault | Server | DDS |
| 5 | Localisation & navigation | Per-floor maps, AMCL, Nav2, velocity bridge | Server host | LiDAR, odom |
| 6 | Floor identification 🔨 | **The key unsolved block.** Classifies which floor the robot is on | Server host | Marker or beacon |
| 7 | Perception | Arrival confirmation (ArUco), obstacle assist | Server host | Camera |
| 8 | Task execution | Deterministic FSM: no model in the loop | Server host | 2,3,5 |
| 9 | Safety supervisor | Independent veto; comms-loss abort; battery floor | Server host | 4 |
| 10 | Logging & diagnostics | Event log, metrics, alerting, replay | Server (SQLite) | All |
| 11 | Notifications | Dashboard, speaker, optional teacher alert | Server | 2 |
| 12 | LoRa heartbeat 🔵 | **Contingency telemetry only.** Never a control path | ESP32 on robot | Independent of Wi-Fi |

### 5.3 What runs on the robot vs the server

| On the robot (closed, on-board) | On the local server (ours) |
|---|---|
| Gait generation, balance, obstacle avoidance | **Everything else** |
| LiDAR driver + point-cloud publication | Navigation, perception, task logic |
| Sport/motion service | Dashboard, auth, DB |
| Battery/BMS telemetry read-out | Safety supervision, logging |

**Rationale:** keep the state machine, validator, and safety supervisor on the
server. They must be testable, inspectable, and durable. The robot must not hold
authoritative task state — if the server dies, the robot stops, and that is the
correct failure behaviour.

⚠️ **Consequence:** if the server dies, there is no software watchdog. The
backstops are the robot's built-in obstacle avoidance (on by default) and the
physical remote controller. **The human spotter is therefore not optional at any
stage.**

---

## 6. Alternative architectures considered

### 6.1 Navigation stack

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| **A. Vendor `slam_operate`** | Documented API; no SLAM work | **Requires expansion dock + separately purchased MID-360/XT16.** Needs a Unitree-issued **YSN**. Cannot run inside an ROS graph. Page body partly unverifiable | **Contingency.** Only if dock already present |
| **B. Self-built Nav2 on the built-in L2** 🏆 | **No purchase.** Built-in LiDAR already on DDS in a ROS 2-compatible type. Full control, no YSN. Demonstrates our engineering. Hardware-independent | We build it. Requires DDS→ROS bridge work | **Recommended primary** |
| C. Sim-only, no hardware nav | Zero risk | Not a delivery system | Rejected |

**B is recommended** because the built-in L2 point cloud *is* published on DDS
(`rt/utlidar/cloud_deskewed`, `PointCloud2_`) and `unitree_mujoco` simulates the
Go2 on **the same DDS channels, CPU-only** — so the entire stack can be built and
tested before the robot is switched on.

### 6.2 Application server location

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| **AI Lab (CP-C3)** 🏆 | Proposed primary base; staff co-located; near the robot | — | **Recommended** |
| Makerspace Lab (CP-A1) | Secondary base | Different block; farther from most destinations | Secondary / DR |
| NAS | Already maintained | **No confirmed NAS exists.** NAS ≠ application server | Not available |
| Cloud / public | Managed | Violates design principle: no public exposure, no internet dependency | **Rejected** |

**Recommendation:** AI Lab machine as primary; Makerspace Lab as a documented
secondary with a cold standby copy of the DB. **No internet dependency.** The
server must reach the teacher browsers and the robot, both on the school LAN.

### 6.3 Vertical transit strategy

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| **Human-assisted, no lift integration** 🏆 | **Legal, safe, zero approvals.** Matches every commercial precedent | Needs a named escort per trip | **Stage 2 design** |
| Johnson **EyeRIS** QR app on the robot's companion phone | Manufacturer-sanctioned; **no panel modification** | Retrofit availability unknown; non-human client provisioning unknown | **Ask Johnson first** — cheapest possible win |
| Sanctioned OEM API (Otis OID / TK MAX / Schindler) | The industry direction of travel | **Not available on Johnson.** Requires OEM + service company + Electrical Inspector permission | Not applicable |
| Actuator on COP/LOP button | Academic papers do this | ⛔ **Prohibited.** Maharashtra Act s.10; Delhi Rules 6(v); TN Act s.15. Otis abandoned this approach | **NEVER** |
| Any safety-circuit or interlock modification | — | ⛔ **NEVER.** Supreme Court of India, *Otis v. Rashmi Handa*, 29 Jul 2026 | **NEVER** |

### 6.4 Floor identification 🔨 — the unsolved block

| Option | Robustness | Cost | Notes |
|---|---|---|---|
| **ArUco/AprilTag per floor, unique ID encodes floor** 🏆 | **High** — absolute, works in low light | ₹0–2k | **Recommended.** Also proves the marker pipeline already works on this robot — the charging pile uses ArUco |
| BLE beacon per floor, ID = floor | Medium–high | ₹1–3k/floor | Good backup; RSSI-thresholded. Published accuracy 0.9877–1.0, ~5 s delay |
| Barometric altimeter on the host | Low alone | ₹500 | **Not available from the Go2 itself.** Corroboration only |
| UWB | Medium | ₹5–15k | Overkill. No sub-metre accuracy requirement identified |
| LoRa RSSI for floor ID | — | — | ⛔ **Must not be used for positioning.** A stale last-known position is a fallback fact, never a navigation input |
| **Unique per-floor geometry (paint patch, ceiling fixture)** | Low–medium | ₹0 | Free, fragile, degrades on repainting |
| **Human confirmation at every floor change** | High as a *check* | ₹0 | **Mandatory regardless of which sensor is chosen** |

**Recommendation:** ArUco per floor **plus** a mandatory human confirmation on
every floor change. Markers make it work; human confirmation makes the residual
risk acceptable.

---

## 7. Key design decisions

### 7.1 The AI / deterministic boundary

```
UNTRUSTED ──▶ [Teacher request text] ──▶ [parse to structured fields]
                                              │
                                              ▼
                              ┌───────────────────────────┐
                              │  TASK VALIDATOR (pure fn) │
                              │  destination ∈ allow-list │
                              │  item ∈ allow-list        │
                              │  weight ≤ limit           │
                              │  origin→dest permitted    │
                              │  robot READY              │
                              │  battery ≥ floor          │
                              │  floor transition legal?  │
                              └───────────┬───────────────┘
                                          ▼
                              [ DETERMINISTIC FSM ] ──▶ [ Safety Supervisor veto ]
                                                          │
                                                          ▼
                                                    [ SportClient / ObstaclesAvoidClient ]
```

**Rules, stated as testable properties:**

1. **No model may reach the robot.** Not in the FSM, not in the safety layer, not
   as a fallback.
2. **The validator is a pure function.** No network, no clock, no I/O. Same input,
   same output.
3. **Teachers never receive motor-control access.** Teacher role can create
   requests and confirm receipts. Robot-admin role can cancel and intervene.
   Neither can command motion.
4. **Unknown destinations are rejected, never guessed.**
5. **Every delivery requires explicit recipient confirmation.** Arrival at a
   destination is not completion. Completion is a human action.
6. **Loss of communications produces a defined safe behaviour** (§7.3), never a
   best-effort continuation.

### 7.2 Movement authorisation

| Actor | May create request | May cancel | May command motion | May intervene physically |
|---|---|---|---|---|
| Teacher | ✅ | Their own | ⛔ | ⛔ |
| Room admin | ✅ | Their block | ⛔ | ⛔ |
| Robot admin | ✅ | All | ⛔ (via FSM only) | ✅ |
| Safety supervisor | ⛔ | ✅ (veto) | ✅ (stop only) | ⛔ |
| Human spotter | ⛔ | ✅ | ✅ (**manual RC only**) | ✅ |

### 7.3 Loss-of-communications behaviour

| Condition | Robot behaviour | Server behaviour |
|---|---|---|
| Server unreachable > N s | Robot executes a **pre-armed** safe stop: `StopMove()` then holds position | Mark robot `ORPHANED`, alarm, accept no new tasks |
| Server unreachable, robot stationary at checkpoint | **Holds indefinitely.** No autonomous wandering | as above |
| Wi-Fi/WLAN loss (server↔robot) | Same as "server unreachable" — but **DDS itself is Ethernet**, so Wi-Fi loss does not affect robot↔server if wired | Alarm |
| Battery below floor | Abort mission, hold position, notify | Alarm, request human recovery |
| Localisation lost | Stop. Re-localise. If it fails within timeout → abort task, request human | Task marked failed with reason |

⚠️ **Critical design note:** the robot↔server link is **DDS over Ethernet**, not
Wi-Fi. So a WLAN failure does not disable the robot. **A network design that
relies on Wi-Fi for the robot link would make the robot stop every time the WLAN
fails** — this is precisely the failure LoRa cannot fix and is why §9 rejects
LoRa as a primary path.

### 7.4 The checkpoint problem

⚠️ **Design principle from the brief: a checkpoint's last-known position is not
necessarily the robot's current position.**

Scenarios that break the naive model:

| Scenario | What breaks | Required handling |
|---|---|---|
| A human carries the robot up stairs | Odometry destroyed; pose wrong | Explicit `RELOCATION` event; invalidate pose; force re-localisation on arrival |
| A human moves the robot within a floor | Same | Same |
| Robot is manually repositioned for charging | Same | Charging station is modelled as a checkpoint **with** an enforced relocation handshake |
| Battery removed for swap | Odometry resets | Same |

**Design rule:** the system stores `last_confirmed_pose` **and** a
`pose_valid` flag. Any manual handling event sets `pose_valid = false` and the
robot must re-localise before accepting any new task. **The console must make
entering a relocation event obvious and unmissable** — an unrecorded relocation
is the single most likely way the robot ends up somewhere it believes it is not.

### 7.5 Battery and mission limits

Derived from verified telemetry and manufacturer behaviour:

| Rule | Value | Basis |
|---|---|---|
| Mission abort threshold | **30 % SOC** | 10-minute crouch warning (verified) + return margin |
| Battery telemetry source | `rt/lowstate` → `LowState.bms_state` | **Verified** — not `SportModeState`, which has no battery fields |
| Planning endurance | **~90 minutes active mission per charge** | Engineering estimate; manufacturer figures span 1–5 h depending on document and load |
| Default gait | `EconomicGait()` | Lowest documented power draw |
| Recovery policy with payload | `AutoRecover` **OFF**; no `RecoveryStand()` with payload | `RecoveryStand` "recovers regardless of whether it has fallen" — violent with 8 kg |
| Range estimation | Coulomb counting from signed `current` + measured Wh/km | `BmsState.current` is signed mA — sufficient |
| Recharge | **Human-assisted.** No autonomous return-to-dock | No dock service in the documented service list |

---

## 8. Robot-to-server communications design

### 8.1 Primary link

| Property | Value |
|---|---|
| Protocol | **Eclipse Cyclone DDS** (only verified path) |
| Physical | **Wired Ethernet** — USB-Ethernet adapter or onboard NIC |
| Network | Dedicated `192.168.123.0/24`; robot `192.168.123.161`; server `192.168.123.222` |
| Isolation | Robot link on its **own subnet**, separate from the smartboard LAN |
| Host requirement | Ubuntu 20.04/22.04, dedicated NIC, static IP |
| Failure mode | Cable/port/NIC failure → watchdog → safe stop |

⚠️ **Wi-Fi is not an option for the robot link.** Unitree documents development as
wired-only, and CycloneDDS SPDP discovery is unreliable over Wi-Fi. A design that
puts the robot on Wi-Fi would be non-conformant with the manufacturer and
operationally fragile.

### 8.2 What this means for the school network

The robot link needs a **dedicated wired segment**. Options, in preference order:

1. **Dedicated USB-Ethernet NIC on the AI Lab server**, connected to the robot's
   port with a direct cable. **Zero school network involvement. Recommended.**
2. Dedicated VLAN + switch port if the school's network allows.
3. Wireless bridge (portable, no cabling) — requires IT approval.

Option 1 is strongly preferred because it **involves no school network change at
all** — the robot is simply a directly-connected Ethernet peer.

### 8.3 Teacher-to-server link

Ordinary school LAN browser access to `https://<server-ip>` on a local address.
**No public exposure.** No port forwarding. No internet dependency. Server binds
to the LAN interface only.

---

## 9. LoRa and ESP32 — what they are and are not

### 9.1 The honest verdict

| Question | Answer |
|---|---|
| Is LoRa worth adding? | **Probably not, as a first step.** Build the Wi-Fi/Ethernet path well first. |
| Does LoRa provide better coverage than Wi-Fi? | **No.** A peer-reviewed 16-storey study found LoRa *still lost the link across floors*. Wi-Fi with one AP per floor beats it. |
| What does LoRa actually buy? | **Independence from the WLAN.** It survives AP reboot, PoE failure, cable cut, or config error. |
| Is it a control path? | ⛔ **Never.** Round trip is hundreds of ms to seconds, non-deterministic, no QoS. |
| Is it a navigation channel? | ⛔ **Never.** |
| Is it a positioning source? | ⛔ **Never.** RSSI must not be used for indoor positioning. |

### 9.2 Indian regulatory constraints — the things that differ from international sources

⚠️ **These are the errors a generic AI or blog will make:**

| # | Finding | Impact |
|---|---|---|
| 1 | India's licence-exempt band is **865–868 MHz, not 868 MHz**. The default EU868 channel (868.1 MHz) is **outside it**. | An unmodified EU868 config is **non-compliant** |
| 2 | India cites **EN 300 220**, not the modern EN 303 708 | A CE/RED test report is built on the wrong standard |
| 3 | Power limited in **e.r.p., not EIRP**. 500 mW e.r.p. = 29.15 dBm EIRP | Quoting 500 mW as EIRP overshoots by 2.15 dB |
| 4 | **Duty cycle: 2.5 % end device, 10 % gateway, 1 % at 25 mW** | At SF12, one 350-byte report every ~2 min max |
| 5 | A **LilyGO T-Beam at full 32 dBm is illegal** (~1531 mW e.r.p. vs 500 mW ceiling) | Must cap TX to ~24–27 dBm + implement APC |
| 6 | **Telecommunications (Radio Equipment Possession Authorisation) Rules, 2026** — G.S.R. 592(E), 8 Jul 2026; applications opened 06.08.2026. Telecom Act 2023 §3(1)(c) now requires authorisation *to possess* radio equipment | **Nearly all secondary sources are stale.** Needs a written DoT WPC clarification |
| 7 | **ETA is now 100 % self-declaration**, ₹10,000, lifetime, since 09.09.2024 | Lowers cost of importing project quantity |

### 9.3 What a heartbeat actually carries

~35 bytes suffices, and it is genuinely useful:

| Field | Bytes |
|---|---|
| Device ID, sequence | 8 |
| Battery % + mV | 2 |
| Charging state | 1 |
| Robot state enum (IDLE/MOVING/CHARGING/FAULT/E-STOP) | 1 |
| Fault code | 2 |
| Position X, Y | 8 |
| **Pose confidence — dead-reckoned vs last-known** | 1 |
| Zone/task ID | 2 |
| CRC | 2 |

⚠️ **The `pose confidence` field is mandatory.** It exists precisely to stop a
stale last-known position being mistaken for the robot's current position.

### 9.4 What LoRa definitively cannot carry

| Not possible | Why |
|---|---|
| Camera stills | ESP32-CAM JPEG ≈ 20–60 kB → 20–60 s airtime per frame; duty cycle >100 % |
| LiDAR | Even a decimated occupancy grid is kilobytes |
| Video | Not a closeable question |
| Motor control / closed loop | Hundreds of ms to seconds, non-deterministic, no QoS |
| Data needed in <1 s | SF7 minimum airtime still ~38 ms; SF9–SF12 typical |

### 9.5 Security requirements if LoRa is ever built

Message integrity (HMAC or authenticated encryption), monotonic sequence numbers
to reject replays, per-device keys, rate limiting, and — critically — **the
gateway must never be able to command the robot**. The LoRa path is
**telemetry-only, inbound to the server, never outbound to the robot.**

---

## 10. Elevator architecture

### 10.1 What is legally permitted without any approval

1. Robot rides as an ordinary passenger while **a human operates the lift**.
2. Robot travels freely on any single floor.
3. **Passive** observation of lift approach, dwell, and door timing from the lobby.
4. Measuring the car's physical geometry with a tape.
5. Piloting the robot on stairs within a single floor.

### 10.2 The Stage 2 human-escort protocol

```
1. Robot is escorted by a NAMED adult to the lift lobby
2. Robot waits in a marked box, clear of the door swing
3. Human presses the hall call
4. Human verifies the car is EMPTY — if occupied, ABORT (do not share)
5. Robot enters, centres, rotates to face the door
6. Human presses the destination button
7. Human STAYS IN THE CAR and rides with the robot
8. At destination, human confirms the car is level and stationary
9. Human exits last and confirms the robot is clear
```

**Failure modes of this protocol, which must be in the risk register:**

| # | Failure | Consequence |
|---|---|---|
| 1 | **Lift stalls between floors with the robot inside** | The exact *Otis v. Rashmi Handa* scenario. Robot battery dead in ≤4 h. Mechanical rescue needed. **The defining risk.** |
| 2 | **Fire alarm → Phase I recall** | Car recalled, doors open, lift removed from service. Robot may block a landing door |
| 3 | Supervisor distracted / walks away | **The dominant real-world failure.** |
| 4 | Student crowd | Occupancy management needed |
| 5 | Wrong floor pressed | Caught by the human present — a real benefit of the assisted protocol |
| 6 | Lift out of service | Documented abort-and-restrict-to-single-floor needed |

### 10.3 The EyeRIS question — cheapest possible win

**Johnson Lifts already ships "EyeRIS," a contactless call-booking system** where
a phone scans a QR code on the landing operating panel and issues both the hall
call and the destination call. This is a **manufacturer-sanctioned, no-panel-modification**
integration path.

⚠️ **Three things are unknown and must be asked in writing:**

1. Is EyeRIS retrofittable to lifts installed earlier?
2. Is the QR element a **passive printed label** or an **active electronic
   component**? (If active, fitting it is a modification requiring approval.)
3. Can a **non-human / robot-provisioned client** use it?

**If the answer is yes to (1) and (3) and passive on (2), this becomes the Stage 3
path** — with no modification and no servo. **If the answer is no, human escort is
the permanent design**, which is what every comparable operator has converged on.

---

## 11. Mapping architecture

### 11.1 Per-floor maps, not one map

Nav2's `map_server` serves one map at a time and exposes `LoadMap` to swap at
runtime; AMCL must be told to accept map updates (`first_map_only: false`).
SLAM Toolbox supports `mapping`, `localization`, and lifelong modes with
pose-graph serialization.

**Design:** one serialized map per floor. A `SelectMap → LoadMap → re-seed
/initialpose` sequence on floor change, with explicit
`map_switch_succeeded` / `map_switch_failed` status reporting. On a failed
re-localisation the task is a **terminal failure**, not a silent retry.

⚠️ **The navigation stack cannot infer floor identity.** It needs an external
floor-identity signal (§6.4). This is the single most important architectural
dependency and it is not solved by any stock Go2 capability.

### 11.2 Destination directory

Manually created, verified by a person, and stable:

```
destination_id     : CP-C3-AILAB        (immutable key, never reused)
display_name       : "AI Lab"
block              : C
floor              : 3
floor_map_id       : c_block_f3
position           : verified pose on c_block_f3
approach_pose      : where the robot should stop
arrival_fiducial   : ARUCO_0042  (required)
delivery_window    : permitted hours
allowed_origins    : which checkpoints may deliver here
floor_transition   : NONE | ESCORTED_LIFT | STAIRS
status             : ACTIVE | BLOCKED | NEEDS_SURVEY
verified_by        : <name, date>
verified_pose_date : <date>          ← pose expires, must be re-verified
```

**Every destination must have `verified_by` and `verified_pose_date`.** A map
without human verification is not usable, and a pose that was verified six months
ago is not trustworthy after a building change.

### 11.3 Route and restriction representation

Routes are **declared, not inferred**. Each declares: permitted origin, permitted
destination, the floor map it uses, whether it crosses a floor transition, and
whether it requires escort.

⛔ **Unverified routes are not executable.** The FSM refuses any task whose route
is not `VERIFIED`.

---

## 12. Human checkpoint architecture

| Checkpoint | ID | Block / floor | Role |
|---|---|---|---|
| AI Lab | `CP-C3` | C / 3 | **Primary base.** Server host, robot storage, charging |
| Makerspace Lab | `CP-A1` | A / 1 | Secondary base, DR server, alternative checkpoint |

### 12.1 Recovery procedure

| Condition | Action |
|---|---|
| Battery low mid-mission | Abort → hold → notify → human retrieves |
| Fault flag set | Stop → `StandDown()` → human retrieves |
| Lost, no comms | Physical search begins at **last known checkpoint** — recorded as `last_confirmed_pose` with its timestamp, **explicitly labelled as possibly stale** |
| Robot unreachable entirely | Escalate to facilities; do not assume position |

⚠️ **The console must display `last_pose_age` prominently.** A position 40 minutes
old on a screen next to a physical robot is dangerous — it invites someone to look
in the wrong place.

---

## 13. What this architecture deliberately does NOT include

| Excluded | Why |
|---|---|
| Autonomous elevator operation | No sanctioned API exists for Johnson lifts; all direct approaches are prohibited |
| Any lift hardware modification | Prohibited by law and by manufacturer terms |
| Autonomous stair descent with payload | Not established; Unitree forbids complex terrain; all independent reports show descent is worse |
| Multi-floor operation without floor ID | **Impossible.** The robot would localise confidently into the wrong floor's map with no error |
| LoRa as a control or navigation channel | Non-deterministic, wrong latency class, wrong data rate |
| UWB positioning | No sub-metre accuracy requirement has been identified |
| LoRa or BLE for indoor positioning | Must not be used for positioning |
| Public network exposure | Design principle; safety and privacy |
| Teacher motor-control access | Design principle; safety |
| Autonomous exploration | Every route must be mapped and verified by a person first |
| Unsupervised operation | Unitree: *"This product is not a toy and is not intended for use by persons under the age of 14"* |

---

## 14. Architecture decision status

| Decision | Status | Blocked by |
|---|---|---|
| Platform: Go2 EDU | ✅ **Decided** | — |
| Navigation stack: self-built Nav2 (B) | 🟡 **Recommended**, pending Phase 2 verification | U1 |
| Robot link: dedicated Ethernet segment | 🟡 **Recommended**, pending school approval | U2, ADM-04 |
| Server location: AI Lab | 🟡 **Recommended**, pending approval | ADM-02 |
| Floor ID: ArUco per floor | 🟡 **Recommended**, pending approval | U5 |
| Vertical transit: human escort | 🟡 **Recommended for Stage 2** | U3, U4 |
| LoRa: defer | 🟡 **Recommended to defer** | U6 |
| Stairs in scope? | ❓ **OPEN** | U3 |
| Elevator integration? | ❓ **OPEN** | U4, ELV answers |
| Which destinations are in scope? | ❓ **OPEN** | U9, TCH answers |

See `decision_log.md` for the full record.