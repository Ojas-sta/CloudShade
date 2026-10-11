# Proposed System Architecture

For **both** platforms. Every interface is labelled:

| Label           | Meaning                                                           |
| --------------- | ----------------------------------------------------------------- |
| ✅ **VERIFIED**  | Exists in official Unitree source or documentation. Source cited. |
| 🟡 **INFERRED** | Follows logically from verified facts; not itself stated.         |
| 🔵 **PROPOSED** | **Our design. Does not exist.** We would build it.                |
| ⚠️ **UNKNOWN**  | Referenced in official docs but the contract is not published.    |

**No SDK function, ROS topic, or API endpoint in this document is invented.**
Anything we could not verify is marked ⚠️ or 🔵, never presented as fact.

---

## 1. The governing constraint

Both robots are controlled over **Eclipse Cyclone DDS**, and the robot's
motion controller is a **closed on-robot binary** reached by RPC. This means:

1. There is **no ROS-level abstraction of the robot's own services** in
   `unitree_ros2` — it defines message *types* and read/write nodes, but not
   service clients for `sport`, `obstacles_avoid`, or `slam_operate`.
2. Our architecture must therefore either (a) use `unitree_sdk2` /
   `unitree_sdk2_python` directly for actuation and `unitree_ros2` for state, or
   (b) wrap `unitree_sdk2` in ROS 2 nodes we write.

**We recommend (b) for the Go2**: thin ROS 2 wrapper nodes so that the whole
navigation stack sees a normal ROS 2 world. **(a) is acceptable for the G1**, where
most control is raw DDS anyway.

---

## 2. Go2 EDU architecture — recommended

### 2.1 Component map

```
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 1 — USER INTERFACE                        🔵 ours             │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────┐  │
│  │ Web Dashboard    │  │ Voice Terminal   │  │ REST / WebSocket   │  │
│  │ (dropdowns,      │  │ (optional)       │  │ API                │  │
│  │  confirmations)  │  │                  │  │                    │  │
│  └────────┬─────────┘  └────────┬─────────┘  └─────────┬──────────┘  │
└───────────┼─────────────────────┼──────────────────────┼─────────────┘
            └─────────────────────┴──────────────────────┘
                              │  WebSocket (🔵 ours)
┌─────────────────────────────▼────────────────────────────────────────┐
│  LAYER 2 — REQUEST & TASK MANAGER                🔵 ours             │
│  ┌────────────────┐ ┌─────────────────────┐ ┌──────────────────────┐ │
│  │ Task Queue     │ │ Scheduler           │ │ Notification Router  │ │
│  │ (SQLite-backed)│ │ (deadlines, battery)│ │ (dashboard, speaker) │ │
│  └────────┬───────┘ └──────────┬──────────┘ └──────────────────────┘ │
│           └─────────────┬─────┘                                       │
│  ┌───────────────────────▼─────────────────────────────────────────┐ │
│  │  TASK VALIDATOR  (deterministic — no LLM reaches this boundary) │ │
│  │  destination ∈ approved registry ?                               │ │
│  │  item ∈ approved item list AND weight within limit ?             │ │
│  │  origin→destination pair permitted ?                             │ │
│  │  deadline feasible given battery + queue ?                      │ │
│  │  robot state == READY ?                                          │ │
│  └───────────────────────────┬─────────────────────────────────────┘ │
└──────────────────────────────┼───────────────────────────────────────┘
                               │  ValidatedTask (typed object)
┌──────────────────────────────▼───────────────────────────────────────┐
│  LAYER 3 — TASK EXECUTOR (deterministic state machine)   🔵 ours   │
│  SELECT_ROUTE → CHECK_BATTERY → NAVIGATE → CONFIRM_ARRIVAL           │
→ UNLOAD → NOTIFY → LOG → RETURN                                       │
│                                                                      │
│  ⛔ Interlock: aborts to SAFE_STOP on any failed precondition        │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────────┐
│  LAYER 4 — SAFETY SUPERVISOR                      🔵 ours (critical) │
│  • Watchdog: no heartbeat → StopMove() + Damp() within N ms          │
│  • Battery floor: abort below threshold                              │
│  • Heartbeat: requires liveness before every motion command           │
│  • Zone enforcement: refuses commands outside the permitted corridor │
│  • Speed cap: clamps all velocity requests                           │
│  • Operator e-stop: hard override from RC and dashboard              │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────────┐
│  LAYER 5 — ROBOT CONTROL ADAPTER                    🔵 ours (wrapper)│
│  Wraps unitree_sdk2 / unitree_ros2. Publishes a clean internal API.  │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ SUBLAYER 5a — LOCALISATION & NAVIGATION        🔵 ours / ✅    │  │
│  │                                                                │  │
│  │  ROUTE B (recommended, no extra hardware):                     │  │
│  │    ✅ rt/utlidar/cloud_deskewed  (PointCloud2_ on DDS)         │  │
│  │    ✅ rt/sportmodestate         (odometry, foot forces)        │  │
│  │    🔵 slam_toolbox / FAST-LIO2 → map                            │  │
│  │    🔵 AMCL → localisation                                       │  │
│  │    🔵 Nav2 (planner + controller) → cmd_vel                     │  │
│  │                                                                │  │
│  │  ROUTE A (fallback, needs dock + MID-360/XT16):                │  │
│  │    ✅ slam_operate service, API 1102 pose navigation           │  │
│  │    ✅ rt/unitree/slam_mapping/odom, /points                    │  │
│  │    ✅ rt/slam_info (status)                                    │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ SUBLAYER 5b — ACTUATION                           ✅ VERIFIED    │  │
│  │  ✅ SportClient.Move(vx, vy, vyaw)   ← velocity locomotion     │  │
│  │  ✅ ObstaclesAvoidClient.Move(x, y, yaw)   ← through avoidance │  │
│  │  ✅ ObstaclesAvoidClient.MoveToAbsolutePosition(x,y,yaw)       │  │
│  │  ✅ SportClient.StopMove(), Damp(), BalanceStand()             │  │
│  │  ✅ SportClient.SpeedLevel(int)                                 │  │
│  │  ⚠️  ObstaclesAvoidClient.UseRemoteCommandFromApi(true)         │  │
│  │      — must be true before Move* works. Ordering is            │  │
│  │      undocumented; VERIFY EMPIRICALLY in Phase 2.              │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │ SUBLAYER 5c — STATE MONITORING                      ✅ VERIFIED  │  │
│  │  ✅ rt/sportmodestate — position, velocity, foot forces, BMS   │  │
│  │  ✅ rt/lowstate       — IMU, motor states, temperatures        │  │
│  │  ✅ SportClient.AutoRecoverGet(bool&)                          │  │
│  └────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────┬───────────────────────────────────────┘
                               │  DDS (✅ verified transport)
┌──────────────────────────────▼───────────────────────────────────────┐
│  UNITREE GO2 EDU — closed on-robot binaries                          │
│  sport_mode / mcf  ·  obstacles_avoidance  ·  unitree_lidar  ·  OTA  │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  PERCEPTION — independent of navigation                🔵 ours / ✅   │
│  ✅ rt/utlidar/height_map_array    ✅ rt/utlidar/range_info           │
│  ✅ GStreamer UDP H.264 → hd camera          (⚠️ fps contradicted)  │
│  ✅ rt/utlidar/cloud, cloud_deskewed                                │
│  🔵 AprilTag detection  🔵 object detection  🔵 QR decoding            │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  STATE & TELEMETRY                                       🔵 ours     │
│  SQLite: tasks, state transitions, metrics, event log                │
│  Rolling telemetry ring buffer → time-series store                  │
│  Concept 7 dashboard: live charts, fault log, predictive maintenance│
└──────────────────────────────────────────────────────────────────────┘
```

### 2.2 Onboard vs external

| Component | Runs on robot? | Rationale |
|---|---|---|
| Motion controller, gait, obstacle avoidance | **Onboard (closed)** | We cannot reimplement it |
| LiDAR driver, point cloud publication | **Onboard (closed)** | Published on DDS automatically |
| Sport mode, obstacles_avoid services | **Onboard (closed)** | RPC endpoints |
| **Navigation stack** (slam_toolbox, AMCL, Nav2) | **External host** | Point cloud is on DDS; no need to be onboard. Moving it onboard is possible on the 100 TOPS dock but adds risk |
| **Perception / CV** | **External host, or onboard on Orin NX** | 15–60 fps at 720p with a detector likely needs the NX. **Benchmark in Phase 2.** |
| Task manager, validator, executor | **External host** | Needs durability, testability, and auditability. Keep the state machine off the robot. |
| Dashboard, LLM, ASR/TTS | **External host** | Off-the-shelf CPU workloads; a GPU host is likely for the LLM |
| **Safety supervisor** | **External host, with RC as the hard backstop** | A software supervisor can fail; the physical remote controller cannot |

⚠️ **Important constraint:** the safety supervisor lives on the external host, so
**if the host crashes, the software watchdog dies.** The robot's own built-in
obstacle avoidance (on by default, `ObstaclesAvoidClient`) and the physical RC are
the backstops. This is why the spotter is non-negotiable and why the robot's
velocity authority must always be reachable from the RC.

### 2.3 Layer-by-layer interface inventory

| # | Component | Interface | Status |
|---|---|---|---|
| 1 | User interface | WebSocket to task manager | 🔵 ours |
| 2 | Request/task manager | In-process, SQLite | 🔵 ours |
| 3 | Task validation | In-process, pure function, no I/O | 🔵 ours |
| 4 | Robot state monitoring | `rt/sportmodestate`, `rt/lowstate` | ✅ VERIFIED |
| 5a | Localisation & mapping | `rt/utlidar/cloud_deskewed`, `rt/sportmodestate` | ✅ input / 🔵 SLAM |
| 5a' | (Route A) | `slam_operate` APIs 1801/1802/1804/1102/1201/1202/1901 | ✅ VERIFIED |
| 5b | Locomotion | `SportClient.Move(vx, vy, vyaw)` | ✅ VERIFIED |
| 5b' | Locomotion w/ avoidance | `ObstaclesAvoidClient.Move(x,y,yaw)`, `MoveToAbsolutePosition` | ✅ VERIFIED |
| 5b'' | Stop / safe state | `StopMove()`, `Damp()`, `BalanceStand()` | ✅ VERIFIED |
| 6a | Perception — geometry | `rt/utlidar/height_map_array`, `range_info` | ✅ VERIFIED |
| 6b | Perception — vision | GStreamer UDP H.264 → OpenCV | ✅ interface / ⚠️ fps |
| 6c | Perception — detection | AprilTag / detector | 🔵 ours |
| 7 | Task execution | Deterministic state machine over 5a/5b | 🔵 ours |
| 8 | Safety supervision | RC + software watchdog + speed clamp | 🔵 ours + ✅ RC |
| 9 | Notifications | Dashboard WebSocket + speaker | 🔵 ours |
| 10 | Logging & diagnostics | SQLite + ring buffer + event log | 🔵 ours |

### 2.4 Transport choice, justified

**Recommendation: DDS for robot-side, HTTP/WebSocket for application-side.**

| Segment | Protocol | Why |
|---|---|---|
| Host ↔ robot | **Cyclone DDS** over **Ethernet** | ✅ VERIFIED and the *only* verified path. Wi-Fi is documented as unreliable for SPDP discovery. |
| Host internal | **ROS 2** (Humble) | Nav2, slam_toolbox, and AMCL require it. `unitree_ros2` is BSD-3 and ROS 2 Humble is officially supported. |
| Task manager ↔ dashboard | **WebSocket** 🔵 | Standard, works through the school's existing Wi-Fi, easy to build a UI against. **The brief suggested WebSockets; we confirm it is the right choice here.** |
| Task manager ↔ LLM | HTTPS 🔵 | Only if the school permits an external API. |
| Persistence | **SQLite** 🔵 | Correct for this data rate and volume. The brief suggested it; confirmed. |

⚠️ **Do not** attempt DDS over Wi-Fi for the robot link. It is documented as
unreliable and will produce intermittent discovery failures that are very hard to
diagnose.

### 2.5 Network configuration

Verified from Unitree documentation:

```
Robot onboard      192.168.123.161     ← do NOT assign this to the host
Development host   192.168.123.222     (or .99)
Subnet             192.168.123.0/24
NIC                passed explicitly to the SDK, e.g. ./go2_low_level enp3s0
DDS domain         0
```

⚠️ Known failure mode, documented by Unitree: "192.168.123.161/24 and
192.168.1.10/24 on the same VLAN … you can discover each other, but unicast
communication fails." Keep the robot link on an isolated subnet.

---

## 3. The AI / deterministic-control boundary

This is the architectural requirement in the brief, and it is the single most
important design decision in the system.

```
╔══════════════════════════════════════════════════════════════════════╗
║  UNTRUSTED LAYER — may not reach the robot                          ║
║                                                                      ║
║  Natural language or image input                                     ║
║         ↓                                                            ║
║  LLM  →  {action, item, destination, deadline, confidence}           ║
║         ↓                                                            ║
║  JSON Schema validation                                              ║
║         ↓                                                            ║
║  Entity resolution against a CLOSED registry                        ║
║         ↓                                                            ║
║  ⚡ HALT: HARD HUMAN CONFIRMATION GATE ⚡                            ║
╚═════════════════════════════╤════════════════════════════════════════╝
                              │
╔═════════════════════════════▼════════════════════════════════════════╗
║  TRUSTED LAYER — complete, auditable, deterministic                  ║
║                                                                      ║
║  Validator:                                                          ║
║    · Schema valid?                          → else REJECT           ║
║    · destination ∈ approved registry?         → else REJECT           ║
║    · item ∈ approved list, weight ≤ limit?     → else REJECT           ║
║    · origin→destination pair permitted?       → else REJECT           ║
║    · deadline feasible (battery, queue)?      → else REJECT           ║
║    · robot == READY (state, battery, faults)? → else REJECT           ║
║                                                                      ║
║  Executor: fixed state machine, no branching on model output         ║
║  Safety supervisor: independent of both, can veto at any step        ║
╚══════════════════════════════════════════════════════════════════════╝
                              │
╔═════════════════════════════▼════════════════════════════════════════╗
║  ACTUATION — verified interfaces only                               ║
║  SportClient.Move · ObstaclesAvoidClient.Move · StopMove · Damp      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### Rules, stated as testable properties

1. **The LLM process has no DDS handle and no network route to the robot.** This
   should be verifiable by inspecting the process's open sockets — and that is a
   good thing to demonstrate.
2. **The LLM has no tool access.** It emits JSON to a pipe, nothing more.
3. **Every LLM-derived task requires human confirmation.** No exceptions, no
   "low-risk auto-execute" path.
4. **The validator is a pure function with no network access.** Same input, same
   output, every time.
5. **Unknown entities are rejected, never guessed.** If the LLM emits
   `"destination": "the chemistry lab upstairs"`, the resolver fails to match and
   the task is rejected. There is no fuzzy fallback.
6. **The safety supervisor can veto any command**, including one produced by a
   fully validated task.
7. **The exhibition demo must work with the LLM layer disabled.** A kill switch
   leaves the manual dashboard fully functional. This guarantees the demonstration
   cannot fail because of an external API.

### Prompt injection

User text is treated as **data, never as instructions**. Mitigations:

- Structured output only; the model cannot emit commands
- The registry is closed and lives in code, not in the prompt
- No tool-calling, no code execution, no retrieval of executable content
- A fixed system prompt with no user-controllable instructions
- An adversarial test suite of ≥100 crafted inputs, with a **target of zero
  unauthorised actions** — reported as a measured pass rate

---

## 4. G1 EDU architecture — for comparison

The G1 architecture is **materially different**, not a variant.

```
┌──────────────────────────────────────────────────────────────────┐
│  LAYERS 1–4 (UI, task manager, validator, executor)              │
│  IDENTICAL to the Go2 design. Deliberately platform-independent.  │
└───────────────────────────┬──────────────────────────────────────┘
                            │
┌───────────────────────────▼──────────────────────────────────────┐
│  SAFETY SUPERVISOR — STRICTLY STRONGER ON G1                     │
│  🔵 • Mandatory frame + rope confirmation at every session start │
│  🔵 • Motion whitelist: standing, walking at ≤0.2 m/s only      │
│  🔵 • Continuous attitude watch → immediate Damp() on anomaly    │
│  🔵 • Hard interlock: no manipulation while walking              │
│       (Unitree: "not recommended to perform overly intense      │
│        motions with the dexterous hand installed, such as        │
│        running or balance tests")                                │
│  🔵 • E-stop does NOT use emergency brake unless frame present  │
└───────────────────────────┬──────────────────────────────────────┘
                            │
┌───────────────────────────▼──────────────────────────────────────┐
│  G1 CONTROL ADAPTER                                                │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ LOCOMOTION                                      ✅ VERIFIED │  │
│  │  ✅ LocoClient.Move(vx, vy, vyaw[, continuous])             │  │
│  │  ✅ LocoClient.SetSpeedMode(0..3)  — 1.0/2.0/2.7/3.0 m/s   │  │
│  │  ✅ LocoClient.SetFsmId(n)   — 0,1,2,3,4,500,501,702,706   │  │
│  │  ✅ LocoClient.Damp()=1 · StopMove() · BalanceStand()       │  │
│  │  ✅ LocoClient.StandUp()=4 · Squat()=2 · Sit()=3            │  │
│  │  ⚠️ NO SLAM / NAV / AVOIDANCE SERVICE PUBLISHED FOR G1      │  │
│  │  🔨 Navigation must be built entirely (slam_toolbox + AMCL  │  │
│  │     + Nav2), and requires a working locomotion policy.      │  │
│  └────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ ARM — HIGH LEVEL (reliable, but opaque)          ✅ VERIFIED │  │
│  │  ✅ G1ArmActionClient.ExecuteAction(id)                     │  │
│  │  ✅ 16 named actions: wave hand, clap, high five, hug, ...  │  │
│  │  ✅ GetActionList(std::string&)                             │  │
│  │  ⚠️ Unitree: "These actions may cause the robot to fall"    │  │
│  │  ❌ No ArmAction struct, no joint space, no IK              │  │
│  └────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ ARM — JOINT LEVEL (powerful, hazardous)        ✅ VERIFIED  │  │
│  │  ✅ publish unitree_hg/msg/LowCmd on rt/arm_sdk @ 50 Hz      │  │
│  │  ✅ subscribe rt/lowstate                                   │  │
│  │  ✅ subscribe rt/secondary_imu (IMUState_)                   │  │
│  │  ⚠️ Joint indices hand-transcribed (G1Arm5/G1Arm7 enums)     │  │
│  │  ⚠️ CRC32 poly 0x04c11db7 MANDATORY                        │  │
│  │  ⚠️ MotionSwitcherClient MUST release internal controller   │  │
│  │  ⚠️ Official example requires a HELD GAMEPAD BUTTON interlock│  │
│  │  Gaps: kp=60, kd=1.5 (waist gains ×4)                       │  │
│  └────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ DEX3-1 HANDS                                  ✅ VERIFIED  │  │
│  │  ✅ publish HandCmd_ → rt/dex3/{left,right}/cmd             │  │
│  │  ✅ subscribe HandState_ ← rt/dex3/{left,right}/state       │  │
│  │  ✅ 7 motors, 9 pressure sensors (33 tactile elements)       │  │
│  │  ✅ mode byte: id|0x0F, status<<4, timeout<<7                │  │
│  │  🔴 NO published grasp success rate for this hand           │  │
│  └────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ AUDIO                                     ✅ VERIFIED       │  │
│  │  ✅ service "voice": TTS(1001), ASR(1002), PLAY, VOLUME, LED │  │
│  │  ✅ subscribe rt/audio_msg (String_) for ASR results        │  │
│  │  ⚠️ speaker_id 0=Chinese, 1=English. No mixed mode.         │  │
│  │  ✅ G1-only advantage: offline ASR + TTS on the robot       │  │
│  └────────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ PERCEPTION                                    ⚠️ WEAK      │  │
│  │  ⚠️ Depth camera + 3D LiDAR listed; camera MODEL not found  │  │
│  │  ❌ No video client, no point-cloud client in SDK2           │  │
│  │  🔵 Any camera/point-cloud access must be built or sourced   │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### Where the architectures diverge

| Concern | Go2 | G1 |
|---|---|---|
| Navigation | Vendor service or self-built on published LiDAR | Self-built only; nothing published |
| Locomotion control | Clean velocity API | Opaque FSM integers; lower-level PD needs CRC + gamepad interlock |
| Manipulation | None | 16 opaque actions, or raw DDS joint control |
| Audio | `vui`: volume + brightness only | `voice`: **offline ASR + TTS** ✅ |
| Perception interface | Rich — DDS point clouds, GStreamer video | Weak — no client published |
| Safety interlocks | Standard | Frame + rope + motion whitelist + attitude watch |
| Fall recovery | `RecoveryStand()`, auto-recovery gaits | **Cannot self-recover on base model** |
| Simulation | CPU-only, high-level usable | Low-level only; needs locomotion policy |

---

## 5. Verified interface quick reference

Everything in this section is quoted from official Unitree source or documentation.

### Go2 actuation

| Interface | Signature | Notes |
|---|---|---|
| `SportClient` | `int32_t Move(float vx, float vy, float vyaw)` | API 1008. Body-frame velocity. |
| `SportClient` | `StopMove()` / `Damp()` / `BalanceStand()` | Safe states |
| `SportClient` | `StandUp()` / `StandDown()` / `RecoveryStand()` | |
| `SportClient` | `SpeedLevel(int level)` | Range **NOT DOCUMENTED** |
| `SportClient` | `AutoRecoverSet(bool)` / `AutoRecoverGet(bool&)` | **Disable when carrying payload** |
| `ObstaclesAvoidClient` | `Move(x, y, yaw)` | x ±1.5, y ±1.0, yaw ±1.57 |
| `ObstaclesAvoidClient` | `MoveToIncrementPosition(x,y,yaw)` | ±2.0 m per axis |
| `ObstaclesAvoidClient` | `MoveToAbsolutePosition(x,y,yaw)` | World frame |
| `ObstaclesAvoidClient` | `UseRemoteCommandFromApi(bool)` | Must be true before `Move*` |
| `ObstaclesAvoidClient` | `SwitchSet` / `SwitchGet` | Avoidance on by default |
| `VideoClient` | `GetImageSample(std::vector<uint8_t>&)` | Single JPEG, service `videohub` |
| `UtrackClient` | `SwitchSet` / `SwitchGet` / `IsTracking` | ISS 2.0 — switch only |

⚠️ **No Go2 speed or velocity limits are documented for `SportClient.Move()`.**
Plan to discover the usable envelope empirically in Phase 3, with the frame
present, starting at 0.2 m/s.

### Go2 topics

| Topic | Type | Content |
|---|---|---|
| `rt/sportmodestate` | `unitree_go/SportModeState_` | Position, velocity, foot forces, BMS |
| `rt/lowstate` | `unitree_go/LowState_` | IMU, motor states, temperatures |
| `rt/utlidar/cloud` | `PointCloud2_` | Raw L2 point cloud |
| `rt/utlidar/cloud_deskewed` | `PointCloud2_` | Motion-deskewed |
| `rt/utlidar/height_map_array` | `HeightMap_` | 2.5D height map |
| `rt/utlidar/range_info` | — | Front/left/right distances |
| `rt/utlidar/switch` | `String_` | `"ON"` / `"OFF"` |
| `rt/lowcmd` | `unitree_go/LowCmd_` | **Low-level only** — requires CRC |
| `/wirelesscontroller` | `unitree_go/WirelessController_` | RC input |

⚠️ Firmware **≥ 1.1.6** required for the V2.0 motion interface.
⚠️ Low-level control requires **`mcf` disabled** first.

### Go2 SLAM service (Route A)

| API ID | Function |
|---|---|
| 1801 / 1802 | start / end mapping |
| 1804 | init pose |
| 1102 | pose navigation |
| 1201 / 1202 | pause / resume |
| 1901 | close slam |

⚠️ Requires EDU + expansion dock + MID-360 or XT16. Requires a Unitree-issued
**YSN**. Cannot run inside an ROS/ROS2-initialised process.

### G1 actuation

| Interface | Signature | Notes |
|---|---|---|
| `LocoClient` | `Move(vx, vy, vyaw[, continuous])` | |
| `LocoClient` | `SetSpeedMode(0..3)` | 1.0 / 2.0 / 2.7 / 3.0 m/s |
| `LocoClient` | `SetFsmId(n)` | 0,1,2,3,4,500,501,702,706,801/802, 550+NNN |
| `LocoClient` | `Damp()` / `StopMove()` / `BalanceStand()` | |
| `LocoClient` | `StandUp()` / `Squat()` / `Sit()` / `ZeroTorque()` | **Python lacks StandUp/Squat** |
| `G1ArmActionClient` | `ExecuteAction(int32_t id)` | Service `arm`, 16 named actions |
| `G1ArmActionClient` | `GetActionList(std::string&)` | Enumerate on-robot actions |
| raw DDS | `LowCmd_` → `rt/arm_sdk` | 50 Hz, CRC32, gamepad interlock |
| raw DDS | `HandCmd_` → `rt/dex3/{side}/cmd` | 7 motors |
| raw DDS | `HandState_` ← `rt/dex3/{side}/state` | 9 pressure sensors |
| `G1AudioClient` | TTS / ASR / play / volume / LED | Service `voice` |

⚠️ Firmware **≥ 1.5.1** for `rt/sportmodestate` on the G1.
⚠️ VUI service ≥ 2.0.3.8 for G1 audio.

---

## 6. What is NOT available — build list

| Layer | Status |
|---|---|
| SLAM | 🔨 Build it (`slam_toolbox` / FAST-LIO2) **or** use `slam_operate` (Route A) |
| Localisation | 🔨 `AMCL` |
| Global + local planning | 🔨 `Nav2` |
| Costmaps | 🔨 `Nav2` |
| Velocity command bridge | 🔨 We write a ROS 2 node wrapping `SportClient.Move` / `ObstaclesAvoidClient.Move` |
| TF tree | 🔨 `robot_state_publisher` / `ros2_control` |
| Camera streaming node | 🔨 GStreamer → OpenCV bridge |
| Object / fiducial detection | 🔨 Everything |
| Task manager, queue, validator, executor | 🔨 Everything |
| Dashboard, notifications | 🔨 Everything |
| Telemetry, metrics, logging | 🔨 Everything |
| Safety supervisor | 🔨 Everything — **and it is the most important thing we write** |
| G1 manipulation (IK, grasp, verify, retry) | 🔨 Everything, on an unreliable base |
| `unitree_ros2` official navigation | ❌ Does not exist |
| `unitree_ros2_joy`, `unitree_go2_slam`, `unitree_lidar_slam`, `unitree_go2_camera` | ❌ **Do not exist in any official repo** |

---

## 7. Failure-mode architecture

Every transition in the executor must have a defined abort path.

| Failure | Detection | Response |
|---|---|---|
| Host process dies | Robot: no heartbeat | Watchdog (if host alive) or operator RC: `StopMove()` + `Damp()` |
| Network partition | DDS timeout | Same |
| Localisation lost | Nav2: no transform | Stop. Re-localise. If it fails within N s, abort task, return to last known good pose or dock |
| LiDAR degraded | Missing / stale `rt/utlidar/cloud_deskewed` | Abort navigation. Do not drive blind |
| Battery below floor | `rt/sportmodestate` | Abort task, return to dock |
| Obstacle blocking route | Nav2 local planner + `rt/utlidar/range_info` | Reroute within limits; after N attempts, abort |
| Robot fault | `rt/lowstate`, `AutoRecoverGet` | Stop, notify operator, task fails cleanly |
| Validator rejects | Deterministic | Reject with a specific reason. **Never execute** |
| LLM unavailable / returns garbage | Timeout or schema failure | **Fall back to manual entry. Never block the demonstration** |
| Wi-Fi to dashboard lost | WebSocket close | Robot completes its current task autonomously; notifications queue; no new tasks accepted |
| Uncommanded motion | `rt/sportmodestate` velocity ≠ commanded | Stop, alert operator |

**Design rule: every failure is a clean stop, never a partial action.** A task
either completes or is recorded as failed with a reason.
