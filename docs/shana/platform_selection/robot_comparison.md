# Robot Comparison — G1 vs Go2

Engineering comparison across all criteria in the Phase 0 brief. Assumption
throughout: **both units are EDU models**, because nothing else is developable
(see `executive_summary.md` §2). Scores are in `weighted_decision_matrix.md`.

All claims carry evidence labels defined in `README.md`.

---

## Summary table

| Dimension | Unitree G1 EDU | Unitree Go2 EDU | Edge |
|---|---|---|---|
| Secondary development | **YES** | **YES** | Tie — but both gated |
| Onboard compute | 8-core CPU + optional Orin (multiple vendors) | 8-core CPU + optional Orin 40–100 TOPS | Tie |
| Locomotion | Bipedal walking, `LocoClient`, max 3.0 m/s | Quadrupedal, `SportClient` (41 methods) | — |
| **Autonomous navigation API** | **None published** | **`slam_operate`, documented API IDs** | **Go2** |
| **Programmable obstacle avoidance** | None published | `ObstaclesAvoidClient`, documented limits | **Go2** |
| LiDAR point cloud on DDS | Not exposed via SDK | `rt/utlidar/cloud`, `cloud_deskewed`, `height_map_array`, `range_info` | **Go2** |
| Rated payload | ≈3 kg **per arm** | ≈8 kg (max ~12 kg) on the body | **Go2** |
| Measured usable force | **13.5 N** unbraced (≈1.38 kgf) | n/a (no arm) | G1 only |
| Manipulation hardware | 2× 5-DOF arms + optional Dex3-1 (7 DOF, tactile) | **None — no arm on any variant** | **G1** |
| Published pick-and-place success | 20–30% full-task, best method, BrainCo hands | **0% — impossible** | G1 |
| Battery | 9,000 mAh, ~2 h spec | 15,000 mAh (EDU), ~2–4 h spec | **Go2** |
| Manufacturer safety requirement | **Protective frame + rope mandatory** | Civilian disclaimer only | **Go2** |
| Certification | None | None | Tie |
| Simulation | `unitree_mujoco` low-level only; needs locomotion policy | `unitree_mujoco` CPU-only | **Go2** |
| High-level API maturity | LocoClient routes through opaque `SetFsmId(N)` | Clean velocity `Move(vx,vy,vyaw)` + 41 typed methods | **Go2** |
| Python SDK parity | **Subset** — missing `StandUp`, `Squat`, arm custom actions; README out of date | Fuller; README broadly accurate | **Go2** |
| Known open reliability defect | Idle overheating → shutdown → falls (NVIDIA #121, open since Apr 2026) | No equivalent defect found | **Go2** |
| Price | Quote only (EDU) | Quote only (EDU) | Tie |
| Exhibition impact | 9/10 | 7/10 | **G1** |

**One-line verdict:** the G1 is the more impressive machine; the Go2 is the more
deployable one. The project brief describes the second.

---

## A. Mobility

### Locomotion interface

**Go2** — `SportClient` exposes 41 methods
(`include/unitree/robot/go2/sport/sport_client.hpp`, BSD-3, VERIFIED). Locomotion
is purely velocity-based:

```cpp
int32_t Move(float vx, float vy, float vyaw);
```

Synthesise forward, back, strafe, and rotate by choosing those three numbers.
Also available: `Damp()`, `BalanceStand()`, `StopMove()`, `StandUp()`,
`StandDown()`, `RecoveryStand()`, `Euler(roll,pitch,yaw)`, `SpeedLevel(int)`,
`Sit()`, `RiseSit()`, and ~20 stunt/gait methods including `TrotRun()`,
`StaticWalk()`, `EconomicGait()`, `ClassicWalk(bool)`, `WalkUpright(bool)`,
`CrossStep(bool)`, `FreeWalk()`, `FreeAvoid(bool)`, `AutoRecoverSet(bool)`.

⚠️ `MoveForward` / `MoveBackward` / `MoveLeft` / `MoveRight` / `Rotate` **do not
exist** on the Go2 in SDK2. Those are from the legacy `unitree_legged_sdk`. Any
tutorial using them is describing an older SDK.

**G1** — `LocoClient` exposes ~17 high-level methods
(`include/unitree/robot/g1/loco/g1_loco_client.hpp`). Most route through an opaque
FSM integer:

```cpp
int32_t Damp()      { return SetFsmId(1); }
int32_t Squat()     { return SetFsmId(2); }
int32_t Sit()       { return SetFsmId(3); }
int32_t StandUp()   { return SetFsmId(4); }
int32_t Start()     { return SetFsmId(500); }
```

Documented FSM table (VERIFIED, `support.unitree.com` + source): 0 zero-torque,
1 damping, 2 squat, 3 sit, 4 lock-stand, 500 walk, 501 walk with 3-DOF waist, 702
lie-down/stand-up, 706 squat/stand, 801 run (→ **802** on firmware after 8.6.x.x),
550+NNN mimic motions (e.g. 550507, name not published).

⚠️ The Python `LocoClient` is a strict subset of the C++ one — `StandUp()` and
`Squat()` are **absent in Python**. The Python README claims parity with C++; that
claim is false (VERIFIED by source comparison).

### Corridor behaviour

| Requirement | Go2 | G1 |
|---|---|---|
| Corridor travel | Low, stable, self-correcting | Requires a trained/tuned locomotion policy |
| Turning in narrow spaces | ✅ Tight turning; ISS 2.0 can follow a person at 1.5 m/s | ⚠️ Wide turning circle; bipeds need more space to stabilise |
| Around people | ⚠️ No vendor safety case. `FreeAvoid` can produce an **uncommanded lateral dodge when stationary** | ❌ Unitree mandates **≥2 m safe distance** from crowds |
| Indoor flooring | ✅ Rated 40° slope, 16 cm step/climb; documented gaits for stairs, gravel, slippery surfaces | ✅ Walks on flat indoor surfaces; **stairs not supported** ("avoid attempting to climb stairs") |
| Door thresholds | ✅ 16 cm step height comfortably exceeds typical thresholds | ⚠️ Threshold crossing destabilises a biped |
| Ramps and stairs | ✅ Stairs via `ClassicWalk`/gait 3; documented stair mode | ❌ Stairs not supported. Max incline **NOT FOUND** for legged G1 |
| Localisation and mapping | ✅ `slam_operate` (EDU) or self-built on built-in LiDAR | ❌ Nothing published |
| Route execution | ✅ Pose navigation, world-frame targets, waypoints | ❌ Must be built from scratch |
| Localisation-failure recovery | ✅ `start_relocation`, `recover_nav`, `pause_nav`/`resume_nav` | ⚠️ Only `RecoveryStand()` for a physical fall |
| Safe stopping | ✅ `StopMove()` + `Damp()`; built-in obstacle avoidance is **on by default** | ✅ `StopMove()` = zero velocity; but stopping a biped still leaves it standing |

### The G1 walking-safety documentation, verbatim

Unitree's G1 User Manual:

> "When using the robot, keep it within your field of vision and a safe distance of
> **at least 2 meters** from obstacles, complex terrain, crowds, water and other
> objects."

> ⚠️ "**Do not use the emergency braking function when the robot is not protected
> by the protective frame and protective rope provided by Unitree** in the software
> manual. Otherwise, it will cause the robot to fall to the ground and cause
> serious damage!!!"

> ⚠️ "In developer mode, the user must ensure that the robot protects the robot by
> the **protection frame and the protection rope**."

> "When users use their own programs, there is a lot of uncertainty, the robot may
> do some unexpected actions (**Uncontrolled to the crowd**, rushing to the water,
> high places Fall, leg movements, tumbling)."

> "This product is not a toy and is not intended for use by persons under the age
> of 18."

This is not a robot that can be driven among students without a frame, a rope, and
a spotter. It is a robot for a fenced-off test area.

---

## B. Carrying capacity

### Rated vs usable — the important distinction

| | Manufacturer max | Practical operating figure | Source of the practical figure |
|---|---|---|---|
| **Go2 EDU** | ≈8 kg, max ~12 kg | **~3 kg centred on the spine** | Community field data: "up to 3 kg centred on the spine: no measurable gait change… above 3 kg or off-centre: slows to 80% and balance starts looking drunk" |
| **G1 EDU** | ≈3 kg per arm | **1.38 kgf sustained, unbraced** | arXiv 2609.25486, measured on real hardware |

Unitree's own footnote on the G1 table is honest about this:
*"The maximum load of the arm varies greatly under different arm extension
postures."*

**Unitree publishes no centre-of-mass envelope, no mounting-offset tolerance, and
no quantified stability penalty for the Go2** (NOT FOUND). The one official signal
connecting payload to behaviour is `AutoRecoverSet`:

> "When the device has a payload, it is recommended to **disable** this setting. If
> the device falls and flips violently, it may damage the payload."

### Carrier attachment

**Go2.** The spine provides four M5 hardpoints. The documented (but image-only)
mounting map is a `back load installation hole map, unit: mm` figure on
`support.unitree.com/home/en/developer/Payload` — **dimensions are published as
PNGs, not text, so usable dimensions are UNVERIFIED.** The EDU expansion dock
mounts with **M3×50 screws** to the back plate.

A practical carrier is a 3 mm 6061-aluminium plate on 6 mm standoffs, with a lip
or tray. This is straightforward and low-risk. Budget for 3 kg of payload
including the carrier's own mass.

**G1.** No documented mounting interface exists for a torso tray. Carrying means
holding the load in the hands while walking — which is precisely the configuration
in which the published reliability failures occur.

### Stability while carrying

This is the sharpest structural difference.

- A Go2 carrying 3 kg has that mass **between its feet**, close to the ground, on a
  statically stable base. The quadruped does not need to balance; it needs to not
  trip.
- A G1 carrying 0.5 kg has that mass **at the end of a 0.45 m arm span**, on a
  12-DOF biped, whose actuators are already near their thermal budget when
  stationary (NVIDIA #121). This is a materially harder control problem, and it is
  the exact scenario in which the documented overheating → shutdown → fall occurs.

### Does payload affect battery and movement?

- **Go2:** `EconomicGait()` documentation states "the robot's height is higher in
  this mode, and the rate of power consumption is slower." So low-power operation
  is available, at the cost of ride height — which interacts with the 16 cm step
  limit. Payload → battery effect is **NOT FOUND** as a quantified statement.
- **G1:** the User Manual states battery life "will depend on the robot's load,
  ambient temperature, and other factors," listing "carrying a load" among the
  activities that reduce it. Independent measurement: 9,000 mAh yields
  **~90 minutes of active walking**, less with continuous manipulation.

---

## C. Object manipulation

**Go2 has none. No variant of the Go2 — AIR, PRO, X, or EDU — has an arm.** VERIFIED
from the Go2 product page (12 knee joints, no manipulator listed) and from
Unitree's arm pages, which pair the Z1 with **Aliengo/B1** and the D2 with
**A2/As2** — never with the Go2. A Go2 with an arm is a third-party build with no
vendor support, warranty, kinematic model, or safety certification.

**G1 arm hardware** (VERIFIED, `unitree.com/g1`):

- 5 DOF per arm; arm span ~0.45 m
- Max load ~2 kg (G1) / ~3 kg (G1 EDU), posture-dependent
- Single-hand DOF: `/` on standard G1; 7 DOF (Dex3-1) + 2 optional wrist on EDU
- G1-EDU Ultimate A: 29 DOF + 2× Dex3-1 (no tactile); B: with tactile; C: Inspire DFQ; D: Inspire FTP

**The high-level arm API is not an interface.** `G1ArmActionClient` exposes four
methods and a 16-entry map of pre-recorded animations:

```cpp
{"release arm", 99}, {"two-hand kiss", 11}, {"left kiss", 12}, {"right kiss", 12},
{"hands up", 15}, {"clap", 17}, {"high five", 18}, {"hug", 19}, {"heart", 20},
{"right heart", 21}, {"reject", 22}, {"right hand up", 23}, {"x-ray", 24},
{"face wave", 25}, {"high wave", 26}, {"shake hand", 27}
```

There is **no** `ArmAction` joint-space struct and **no**
`JointControl/ArmControl/GripperControl` enum in any current official repository
(NOT FOUND — these belong to a removed legacy `unitree_arm_interface.h`).

⚠️ Unitree's own warning on that API: *"Some actions will not be displayed on the
APP, but can be executed by the program. **These actions may cause the robot to
fall**, so please execute them with caution."*

⚠️ A C++/Python contradiction exists in the official sources: C++ maps both
`"left kiss"` and `"right kiss"` to ID **12**; Python maps `right kiss` to **13**.
One is wrong. Resolve empirically. The C++ and Python clients also use different
JSON keys for the same API ID (`{"action_id":N}` vs `{"data":N}`).

**Real joint-level arm control** requires raw DDS on `rt/arm_sdk`
(`unitree_hg/msg/LowCmd`) with hand-transcribed joint indices, mandatory CRC32,
disabling the internal motion controller via `motion_switcher`, and — in
Unitree's own official example — **holding a physical gamepad button to arm and
disarm**. Gains are `kp=60, kd=1.5` at 50 Hz.

**Dex3-1 hands** (VERIFIED): 7 motors, 9 pressure sensors (33 tactile sensing
elements), topics `rt/dex3/{left,right}/cmd` and `rt/dex3/{left,right}/state`,
mode byte packed as `id & 0x0F | (status & 0x07) << 4 | (timeout & 0x01) << 7`.
⚠️ The official example writes the *state* topic as `rt/lf/dex3/left/state` —
an apparent in-repo typo; the prose docs use `rt/dex3/left/state`.

Unitree's constraint on the hand, verbatim:

> "During development, **it is not recommended to perform overly intense motions
> with the dexterous hand installed, such as running or balance tests**."

That couples hand work to locomotion work in a way that directly conflicts with
the project's delivery requirements.

### Does the G1 reliably pick up a notebook?

| Sub-claim | Verdict |
|---|---|
| The arm can physically hold a 0.3–0.5 kg notebook | **TRUE** — 13.5 N ≈ 1.38 kgf |
| It can reach a surface where the notebook lies | **CONDITIONAL** — 1,320 mm tall; cannot reach standard workbenches |
| Dex3-1 (3 finger, 7 motor) can grip a 1–3 mm flat sheet | **UNPROVEN, negatively signalled** — zero flat-sheet tasks in Unitree's ~180-dataset corpus |
| Full pick-and-place on a real object | **DEMONSTRATED at 20–30%** |
| Reliably (≥90%) | **FALSE** — best published full-task figure is 30% |
| Autonomously out of the box | **FALSE** — every result required per-task data collection and training |
| Reliably over an extended session | **FALSE/CONTRADICTED** — NVIDIA #121, unresolved since Apr 2026 |
| Safely around people | **FALSE** — no ISO 10218 / ISO/TS 15066; frame + rope mandatory |

**Honest one-liner:** somebody has made a G1 pick up an object, on video, in a
research lab, at roughly a 20–30% success rate per attempt, after months of
teleoperated data collection and policy training. It has never been demonstrated
to do so reliably, repeatedly, autonomously, or safely. Treat it as a demo, not a
capability.

---

## D. Perception

| Capability | Go2 EDU | G1 EDU |
|---|---|---|
| RGB camera | ✅ HD wide-angle, F2.2, 120° FOV (all variants) | Depth camera + 3D LiDAR (listed for both G1 and G1 EDU) |
| Depth camera | ✅ EDU only (`●`) | "Depth Camera" listed — **model NOT FOUND anywhere** |
| LiDAR | ✅ Built-in 4D L2: 360°×96°, 21,600 samples/s, 0.05 m min detection, 18 rings | 3D LiDAR listed; **Mid360 strongly implied** (named explicitly for G1-D and in the G1 SLAM doc topic `rt/utlidar/cloud_livox_mid360`) |
| **LiDAR data accessible** | ✅ **`rt/utlidar/cloud`, `cloud_deskewed`, `height_map_array`, `range_info`, `switch`** on DDS, `sensor_msgs/PointCloud2`-compatible | ❌ No client published in SDK2 |
| Obstacle detection | ✅ Built-in, **on by default**; `ObstaclesAvoidClient` programmable | Not published |
| Person detection | ❌ No official demo exists (NOT FOUND) | ❌ No official demo exists |
| Object recognition | ❌ Must be built | ❌ Must be built |
| QR / marker recognition | ❌ Must be built (camera exists) | ❌ Must be built |
| Destination recognition | ❌ Must be built | ❌ Must be built |
| Camera stream interface | `VideoClient::GetImageSample` = **single JPEG snapshot**, caller-paced; real streaming needs **GStreamer UDP H.264** | No video client in SDK2 at all |
| Audio | `vui` service: volume + brightness only | `voice` service: **TTS + ASR (offline) + playback + volume + RGB LED** |
| Depth for manipulation | EDU depth camera (model unspecified) | "Depth camera" (model unspecified) |

⚠️ **G1 audio is the one place the G1 is functionally richer.** G1 exposes offline
ASR and TTS on DDS (service `"voice"`, API IDs 1001–1010). Documented limitation:
**"Mixed Chinese and English modes are not supported"** — speaker 0 is Chinese,
speaker 1 is English. G1 hardware: 4-microphone array, 5 W speaker. (G1+ has
6 mics and 2×10 W, but G1+ is a different product — do not conflate.)

⚠️ **Go2 camera frame rate is contradicted by Unitree's own docs:** `about_Go2`
says 720p/1080p at **15 fps**; the FAQ says 1080p at 30 fps or 720p at 60 fps.
Do not rely on either without bench-testing. 720p at 15 fps is adequate for
AprilTag/QR detection and modest object classification, but not for anything
requiring fast tracking.

⚠️ **Taking over motion control kills the height map.** Official doc: "the height map
function requires the use of the `sportmodestate` topic… **If the operation
control is turned off, such as when debugging your own operation control program,
the height map data cannot be obtained.**"

---

## E. Software development

### Verified constraints (identical for both robots)

| Constraint | Value | Label |
|---|---|---|
| Transport | **Eclipse Cyclone DDS** (`libddsc.so` vendored; `cyclonedds==0.10.2` pinned in Python) | VERIFIED |
| DDS domain | 0, with an explicit NIC argument | VERIFIED |
| Discovery | Standard CycloneDDS SPDP, `239.255.0.1:7400` | VERIFIED |
| Robot IP | `192.168.123.161` — **do not assign to the host** | VERIFIED |
| Host IP | `192.168.123.222` (or `.99`), on the `123` subnet | VERIFIED |
| Network | **Wired Ethernet required.** Wi-Fi unreliable for SPDP discovery | VERIFIED |
| Host OS | **Ubuntu 20.04 / 22.04 only.** "Development on Mac or Windows is currently not supported" | VERIFIED |
| Where the SDK runs | An **external** Linux host. "nor is development on Go2's internal computer" | VERIFIED |
| Python install | `pip install unitree-sdk2` — **NOT** `unitree_sdk2py` (that name 404s on PyPI) | VERIFIED |
| Python build step | Must compile CycloneDDS from source first; `CYCLONEDDS_HOME` must be set | VERIFIED |
| Python freshness | PyPI release is **1.0.1 from 2025-09-09**, ~13 months behind the GitHub repo. Install from git | VERIFIED |
| Control rate | Go2: 500 Hz low-level and high-level. G1 examples: 200 Hz (`dt=0.005`) | VERIFIED |
| Critical prerequisite | Low-level control **requires disabling `mcf`** first, or the robot receives two command streams | VERIFIED |
| Licence | BSD-3-Clause across `unitree_sdk2`, `unitree_sdk2_python`, `unitree_ros2`, `unitree_mujoco` | VERIFIED |

⚠️ The SDK's implementation is a **closed ~28 MB prebuilt static library**
(`libunitree_sdk2.a`, both arches). You may ship it under BSD-3; you may not audit
or patch it.

### What exists officially — and what doesn't

**`unitree_ros2` provides exactly four packages**: `unitree_go` (Go2/B2 messages),
`unitree_hg` (G1/H1/H2 messages incl. `HandCmd`/`HandState`), `unitree_api`
(request/response transport), `unitree_ros2_example` (~23 nodes).

Its `package.xml` depends on `unitree_go`, `unitree_hg`, `unitree_api`, `rclcpp`,
`std_msgs`, `rosbag2_cpp`. **It does not depend on `nav_msgs`, `tf2`, `ros2_control`,
`nav2`, `slam_toolbox`, or `amcl`.** It is a DDS message bridge plus teleoperation.
It is not a navigation stack.

Widely-cited packages **`unitree_ros2_joy`, `unitree_go2_slam`,
`unitree_lidar_slam`, `unitree_go2_camera` do not exist** in any official
repository (confirmed against the complete non-truncated `master` tree and the
`v0.1.0` tag tree). Any tutorial referencing them is wrong or vendor-private.

**Official ROS 2 support is Foxy and Humble only** — confirmed by CI workflows,
not just the README. Jazzy (the current LTS) is unsupported. Expect friction.

**G1 ROS 2 example set is the richest** — `g1_low_level_example`,
`g1_arm_sdk_dds_example`, `g1_arm_action_example`, `g1_dex3_example`,
`g1_dual_arm_example` (⚠️ silently skipped unless yaml-cpp ≥ 0.6),
`g1_loco_client_example`, `g1_ankle_swing_example`, `g1_audio_client_example`.

**Go2 ROS 2 example set is small** — `go2_sport_client`, `go2_stand_example`,
`go2_robot_state_client`, `low_level_ctrl`. `go2_trajectory_follow` is commented out
of `CMakeLists.txt` and never built. But `unitree_sdk2_python` ships runnable
`obstacles_avoid` examples that C++ does not.

### Simulation — the decisive developer-experience factor

`unitree_mujoco` (BSD-3, 1,234★) simulates **a2, as2, b2, b2w, g1, go2, go2w, h1,
h1_2, h2, r1** and states:

> "`unitree_mujoco` is a simulator developed based on `Unitree sdk2` and `mujoco`.
> Users can easily integrate the control programs developed with `Unitree_sdk2`,
> `unitree_ros2`, and `unitree_sdk2_python` into this simulator, enabling a
> **seamless transition from simulation to physical development**."

It runs **on CPU** — no GPU required. That is decisive for a school with no GPU
budget.

⚠️ Two limits: it supports "**only low-level development**" (`LowCmd`, `LowState`,
`SportModeState`, `IMUState`), and for the **G1** you must supply a locomotion
policy — there is no high-level walking controller in the simulator. For the Go2,
`SportModeState` is retained in simulation even though it is not readable on real
hardware with the motion service disabled.

**This is the single largest practical difference for a student team.** The Go2's
entire software stack can be built, debugged, and demonstrated in simulation
before the robot is ever switched on. The G1's cannot.

### Community stacks — all community, none official

| Project | ★ | Licence | Robots | Maintained |
|---|---|---|---|---|
| `abizovnuralem/go2_ros2_sdk` | 1057 | BSD-2 | Go2 | ✅ (joint states now 1 Hz on fw 1.1.7) |
| `hucebot/g1pilot` | 289 | BSD-3 | G1 | ✅ Dual control + Cartesian + URDF |
| `botbotrobotics/BotBrain` | 374 | MIT | Legged | ✅ Web UI, teleop, nav |
| `andy-zhuo-02/go2_ros2_toolbox` | 181 | MIT | Go2 | ✅ |
| `Adyansh04/grove-g1` | 71 | BSD-3 | G1 | ✅ ROS 2 Jazzy, sim-first, SLAM+AMCL+Nav2+MoveIt+BT |
| `h-naderi/unitree-go2-slam-nav2` | 189 | **none** | Go2 | ❌ stale 19 months — **unlicensed, unusable** |

`grove-g1` is notable: it is a complete G1 delivery stack — ros2_control, MoveIt,
slam_toolbox, AMCL, Nav2, BehaviorTree — built by one person against a real G1,
simulation-first. It is also candid that a GR00T N1.7 policy "does not grasp yet;
that needs demonstrations recorded on this robot." It is proof that a competent
individual *can* build the G1 stack, and simultaneously proof of the scope.

### Avoid

| Repo | Reason |
|---|---|
| `unitree_slam` | **No licence file** (no legal grant of any right), ships closed binaries, targets **H1 and B2 only**, 2★, created 2026-07-21. Looks like official navigation; is not usable. |
| `point_lio_unilidar` | **GPL-2.0** (strong copyleft — linking forces you to release your whole stack) **and ROS 1 Noetic** (catkin). |
| `unitree_ros_to_real` | No licence, ROS 1 Melodic, dead since 2023-06 |
| `unitree_pybullet` | No licence, dead since 2020 |
| `unitree_rl_gym` | Isaac Gym — deprecated by NVIDIA in favour of Isaac Lab |
| `unitree_sdk2_rs` | 8 versions in 5 days on first publish, 183 total downloads, build-time GitHub clone. Treat as experimental. |

---

## F. Reliability

| Concern | Go2 | G1 |
|---|---|---|
| Startup / initialisation | ✅ Generally clean; occasional reports of failing to stand properly on first power-up | ❌ Teardown lab: "needed several software updates before it would do anything" |
| Battery monitoring | ✅ `rt/sportmodestate` includes BMS state | ✅ Same |
| Communication loss | DDS timeout handling; Python daemon reader threads | Same |
| Navigation failure | ✅ `recover_nav`, `start_relocation` | ⚠️ Only `RecoveryStand()` for a physical fall |
| Perception failure | Graceful degradation — avoid the dependency | Same |
| Emergency stop | `StopMove()` + `Damp()`; physical RC | `Damp()` = FSM 1; **manual is internally inconsistent on the RC combo (`L1+A` vs `L2+B`) — verify on hardware** |
| Manual takeover | ✅ RC with velocity authority (`UseRemoteCommandFromApi` toggles) | ✅ RC |
| Task cancellation | Application-level; must be built | Same |
| Safe recovery | ✅ Auto-recovery gaits, `RecoveryStand()`, `AutoRecoverSet` | ⚠️ **Cannot self-recover from a fall on the base model** |
| Repeatability | ✅ Consistent gait; documented failure modes are software-level | ❌ Multiple independent reliability reports |

### The reliability evidence, in full

**Against the G1** (all VERIFIED):

1. **Munro & Associates teardown**, reported by MotorTrend, 2026-06-26 — a
   professional engineering firm bought a base G1: *"out of the box it was very
   buggy and needed several software updates before it would do anything. Then
   they pushed and shoved it until it fell down, whereupon it proved unable to
   stand back up."* Their engineering findings: joints are **not** strain-wave
   (two-stage planetary, ~15:1, cheap and backdrivable at the cost of backlash);
   *"many aluminum parts appear milled from billet and many plastic parts appear
   3D printed"*; an **exposed, greased Cardan joint at the ankle**. Panel verdict:
   *"definitely not ready for prime time."* Actuators were partly ported from the
   **Go2 Pro** — i.e. Unitree's most field-proven actuator line is the one in the
   Go2.
2. **Auckland University of AUT**, 2026-09 — a university lab's G1 described as
   *"stumbling heavily… This instability makes the robot unusable."* Pelvis IMU
   replaced (Unitree-suggested), still unstable. **Unitree's own engineer, present
   in the lab, called back to the factory in China for guidance.** Hip motors
   needed replacement.
3. **NVIDIA `GR00T-WholeBodyControl` issue #121**, open since 2026-04-27: a
   maintainer reports G1 + Dex3 overheating "after a few minutes, even when it is
   just standing idle", then auto-shutdown, then **repeated falls**. A second user
   reports no such issue with their own RL policies, so it may be policy-specific —
   but it is **unresolved in NVIDIA's own tracker**, and a fifth-month follow-up
   (2026-09-21) reads: *"we were able to fine-tune... for stability (**no finger
   use for now**), but it's still not perfect."*
4. **A $100k research buyer abandoned the platform** (2026-02): *"a knee issue that
   required a 2-month wait for a replacement part. Two months for one part stops
   your entire development cycle."* Plus *"frequent hardware failures, particularly
   **in its arms**."* Outcome: built their own humanoid.
5. **No fall-rate statistic exists anywhere.** Not from Unitree, not from academia,
   not from reviewers. (NOT FOUND)

**For the Go2:**

1. **Teardown endorsement** (simplexitypd.com): *"The design reflects a clear
   emphasis on robustness and long-term durability... leg modules designed for rapid
   field replacement... fastener retention and meticulous cable management suggests
   that UniTree prioritized vibration resistance, shock absorption, and overall
   impact survivability."*
2. **Deployed in anger**: Hangzhou Asian Games javelin/discus transport on the field.
3. **Unitree's own quadruped inspection product line** exists; no comparable G1
   product ships.
4. **But** — an independent operator reports: *"It has fallen or run into things —
   like when using the companion controller and it freaks out for no obvious
   reason"*; *"On start up, I saw the Go2 fail to stand and balance properly.
   Instead of righting itself, it flailed its legs around wildly. This happened
   three times on the first day"*; *"the Go2 has, without warning, powered off
   completely in the middle of its movement and collapsed to the floor"*; and
   *"I have seen it run into a door on its way out of a small office while it was
   supposed to be in autonomous navigation and obstacle avoidance mode."*
5. **No published report of a Go2 tipping over under payload.** (NOT FOUND)
6. **No rigorous independent runtime test.** (NOT FOUND)

**Honest read:** the Go2 is more robust than the G1, but it is not a reliable
unattended robot either. Its failure modes skew software/controller rather than
mechanical — which is a problem we can actually engineer against. The G1's skew
mechanical and thermal — which we cannot.

---

## G. Safety

Full treatment in `safety_and_failure_analysis.md`. Summary:

| Risk | Go2 EDU | G1 EDU |
|---|---|---|
| Crush / impact injury | Low — 15 kg, low CoM, no rigid upper body | **High** — 35 kg falling biped; Unitree mandates frame + rope |
| Trip / entanglement | **Real** — 12 moving legs at knee height; pinching hazard | Low |
| Falling objects | Main risk — payload on a tray | Main risk — objects held overhead-ish |
| Balance failure | n/a (quadruped, self-righting) | **Primary hazard** |
| Uncommanded motion | `FreeAvoid` may produce a lateral dodge when stationary | Custom-code risk: *"Uncontrolled to the crowd"* (Unitree's words) |
| Operating speed | Cap for demonstration: **0.4–0.6 m/s** | Cap: **0.2–0.3 m/s**, and only with frame + rope |
| Vendor safety case | **None.** Civilian disclaimer only | **None**, plus mandatory frame/rope procedure |
| Certification | None | None — no ISO 10218, no ISO/TS 15066 |
| Environment limits | 5–35 °C | 0–40 °C; **not waterproof, not dustproof** |
| Battery | Charge 0–55 °C, discharge −20–60 °C | Same class |
| Min safe distance | Not specified by Unitree | **≥2 m**, mandated |

**Neither robot has a manufacturer safety case. Both require a physical exclusion
zone, a human spotter, and a documented abort procedure. That is non-negotiable
and is the same for both platforms.**

**Recommendation for the G1 if it is ever selected:** it cannot be demonstrated
safely in a normal school corridor. It would need a fenced test pen, a protective
frame, a spotter whose only job is the robot, and a scripted-only motion policy.
This is a materially different event from a Go2 demonstration, and it is the
strongest practical argument against the G1.

---

## H. What would change this comparison

| If this is discovered | Effect |
|---|---|
| School's Go2 is AIR / PRO / **or X** | Project is blocked on hardware. X's `◕` must be confirmed in writing by Unitree. |
| School's G1 is the **standard** model | G1 option closed entirely — `Secondary Development: /` |
| School already owns a Go2 EDU **expansion dock + MID-360/XT16** | Go2 gains the vendor navigation path at zero marginal cost; margin widens |
| School's network policy blocks devices with known telemetry | G1 excluded on IT grounds regardless of merit |
| School's brief turns out to be "demonstrate humanoid manipulation" | Re-run `weighted_decision_matrix.md` §5 with manipulation-weighted criteria; the G1 wins |
| Go2 EDU battery turns out to be the 8,000 mAh pack, not 15,000 mAh | Lose the endurance margin; plan around 1–2 h instead of 2–4 h |
| Demonstration space has no flat, wide, obstacle-free corridor | Both robots are impaired; Go2's advantage narrows but does not reverse (Go2 tolerates thresholds and slopes the G1 does not) |
