# SDK and Software Research

Verified inventory of every officially documented interface, with direct links.
**This document is the boundary between what exists and what we would have to
build.** Anything not listed here with a source is either UNVERIFIED or does not
exist.

Collected 2026-10-10.

---

## 1. Official repositories

56 repositories under `github.com/unitreerobotics`. The decision-relevant ones:

| Repository | ★ | Licence | Language | Covers | Last push |
|---|---|---|---|---|---|
| [`unitree_sdk2`](https://github.com/unitreerobotics/unitree_sdk2) | 1395 | **BSD-3** | C++ | **11 robot families** | 2026-09-21 |
| [`unitree_sdk2_python`](https://github.com/unitreerobotics/unitree_sdk2_python) | 820 | **BSD-3** | Python | 12 families | 2026-09-21 |
| [`unitree_ros2`](https://github.com/unitreerobotics/unitree_ros2) | 858 | **BSD-3** | C++ | Go2, B2, G1, H1, H2 | 2026-07-02 |
| [`unitree_mujoco`](https://github.com/unitreerobotics/unitree_mujoco) | 1234 | **BSD-3** | Python | 11 families, **CPU-only** | 2026-09-07 |
| [`unitree_model`](https://github.com/unitreerobotics/unitree_model) | 156 | BSD-3 | URDF | models | — |
| [`unitree_rl_lab`](https://github.com/unitreerobotics/unitree_rl_lab) | 1395 | Apache-2.0 | Python | Go2, H1, G1 (IsaacLab, **GPU**) | 2026-05-25 |
| [`unitree_rl_mjlab`](https://github.com/unitreerobotics/unitree_rl_mjlab) | 676 | Apache-2.0 | Python | Go2, G1, R1, H2 (MuJoCo) | 2026-04-13 |
| [`unitree_dds_wrapper`](https://github.com/unitreerobotics/unitree_dds_wrapper) | 34 | BSD-3 | C++ | **Legacy** — superseded | 2024-10-16 |
| [`unilidar_sdk`](https://github.com/unitreerobotics/unilidar_sdk) | 142 | BSD-3 | C++/ROS | **L1** standalone LiDAR | 2024-02-18 |
| [`unilidar_sdk2`](https://github.com/unitreerobotics/unilidar_sdk2) | 106 | BSD-3 | C++/ROS | **L2** standalone LiDAR | 2025-03-10 |
| [`UnitreecameraSDK`](https://github.com/unitreerobotics/UnitreecameraSDK) | 122 | MPL-2.0 | C++ | **Go1 stereo cameras only** | 2026-08-28 |
| [`avp_teleoperate`](https://github.com/unitreerobotics/avp_teleoperate) | 1696 | NOASSERTION | Python | **XR teleop: G1 23 & 29 DOF, H1, H2, R1** | 2026-10-10 |
| [`unitree_lerobot`](https://github.com/unitreerobotics/unitree_lerobot) | 775 | NOASSERTION | Python | LeRobot fork + G1 data tooling | 2026-09-10 |
| [`dex1_1_service`](https://github.com/unitreerobotics/dex1_1_service) | 20 | BSD-3 | C++ | G1 + Dex1-1 hand | 2026-09-23 |
| [`brainco_hand_service`](https://github.com/unitreerobotics/brainco_hand_service) | 18 | none | C++ | G1 + BrainCo Revo2 | 2026-09-08 |
| [`linker_hand_service`](https://github.com/unitreerobotics/linker_hand_service) | 3 | none | C++ | G1 + Linkerbot O6 | 2026-10-01 |

### Avoid — documented reasons

| Repository | Reason |
|---|---|
| `unitree_slam` | **No licence file** (confirmed: 796-file tree, zero LICENSE). Ships closed binaries. Targets **H1 and B2 only** — not Go2, not G1. 2★. Created 2026-07-21. |
| `point_lio_unilidar` | **GPL-2.0** + **ROS 1 Noetic** (catkin). Strong copyleft — linking forces you to release your entire stack. |
| `unitree_ros_to_real` | **No licence.** ROS 1 Melodic. Dead since 2023-06. |
| `unitree_pybullet` | **No licence.** Dead since 2020. |
| `unitree_rl_gym` | Isaac Gym — **deprecated by NVIDIA** in favour of Isaac Lab. Untouched since 2025-07. |
| `unitree_sdk2_rs` | 8 versions in 5 days on first publish. 183 total downloads. Build-time GitHub clone. Experimental. |
| `unitree_guide` | Not SDK2 at all — ROS 1 Melodic + Gazebo, no current robot support. |
| `unitree_ros` | ROS 1. Superseded by `unitree_ros2`. |
| `unifolm-vla`, `unifolm-wla`, `unifolm-world-model-action`, `unifolm-wla` | Foundation-model repos. **No technical documentation, no model card, no licence, no API reference published.** UNVERIFIED. |
| `Unitree-Motor-Assistant`, `unitree-motor-debugging-assistant`, `digital_servo` | Motor-level, not platform-level. Not relevant. |

---

## 2. Transport layer

**Eclipse Cyclone DDS.** VERIFIED three ways: vendored `thirdparty/lib/*/libddsc.so`
and `libddscxx.so`; `licenses/eclipse-cyclonedds/` in-tree; and `cyclonedds==0.10.2`
pinned in the Python `install_requires`.

```cpp
unitree::robot::ChannelFactory::Instance()->Init(0, argv[1]);
//                                                             ^ NIC name
```

| Property | Value |
|---|---|
| DDS domain | 0 |
| NIC | Explicit string argument, every example takes it as `argv[1]` |
| Discovery | Standard CycloneDDS SPDP, `239.255.0.1:7400` |
| Robot IP | `192.168.123.161` — **do not assign to the host** |
| Host IP | `192.168.123.222` (or `.99`), `192.168.123.0/24` |
| Network | **Wired Ethernet required.** Wi-Fi unreliable for SPDP |

⚠️ **`239.127.0.10` is NOT an SDK2 address.** It does not appear in any of the five
SDK repositories. It belongs to Unitree's separate WebRTC/Robot-WebSocket path
(`unitree_ros2_websocket`, `unitree_lerobot`). Do not conflate them.

⚠️ **QoS is opaque.** `include/unitree/common/dds/dds_qos.hpp` exists, but
`CreateSendChannel(name)` and `CreateRecvChannel(name, cb, queuelen=0)` expose no
QoS parameter, and no example ever constructs one. Actual QoS is fixed inside the
closed `.a`. The only override is the standard `CYCLONEDDS_URI` environment
variable.

⚠️ **The SDK implementation is closed.** `lib/x86_64/libunitree_sdk2.a` is 27.6 MB;
`lib/aarch64/libunitree_sdk2.a` is 29.1 MB. There is **no `src/` directory**. You
may ship it under BSD-3; you may not audit or patch it.

---

## 3. Robot coverage in SDK2

15 example directories: `a2, as2, b2, b2w, g1, go2, go2w, h1, h2, helloworld,
jsonize, r1, state_machine, wireless_controller`.

**Both G1 and Go2 are first-class in the same repo, same commit.**

### G1 examples
```
example/g1/audio/       example/g1/g1d/       example/g1/low_level/
example/g1/dex3/        example/g1/high_level/
```
Headers: `include/unitree/robot/g1/{agv, arm, audio, common, loco}/`

### Go2 examples
```
example/go2/go2_sport_client.cpp      example/go2/go2_robot_state_client.cpp
example/go2/go2_low_level.cpp         example/go2/go2_stand_example.cpp
example/go2/go2_trajectory_follow.cpp (COMMENTED OUT of CMakeLists — never built)
example/go2/go2_video_client.cpp      example/go2/go2_vui_client.cpp
```
Headers: `include/unitree/robot/go2/{config, obstacles_avoid, public, robot_state,
sport, utrack, video, vui}/` — **8 header dirs, richer than G1's 5.**

### Absent from SDK2
`go1`, `aliengo` — NOT FOUND. `z1` is a separate `z1_sdk`.

---

## 4. Go2 interfaces — verbatim

### 4.1 `SportClient` — 41 methods

Source: `include/unitree/robot/go2/sport/sport_client.hpp`. API version `1.0.0.1`.

```cpp
int32_t Damp();            int32_t BalanceStand();   int32_t StopMove();
int32_t StandUp();         int32_t StandDown();      int32_t RecoveryStand();
int32_t Euler(float roll, float pitch, float yaw);
int32_t Move(float vx, float vy, float vyaw);          // ← locomotion
int32_t Sit();             int32_t RiseSit();        int32_t SpeedLevel(int level);
int32_t Hello();           int32_t Stretch();       int32_t SwitchJoystick(bool);
int32_t Content();         int32_t Heart();         int32_t Pose(bool);
int32_t Scrape();          int32_t FrontFlip();     int32_t FrontJump();
int32_t FrontPounce();     int32_t Dance1();        int32_t Dance2();
int32_t LeftFlip();        int32_t BackFlip();      int32_t HandStand(bool);
int32_t FreeWalk();        int32_t FreeBound(bool); int32_t FreeJump(bool);
int32_t FreeAvoid(bool);   int32_t ClassicWalk(bool); int32_t WalkUpright(bool);
int32_t CrossStep(bool);   int32_t AutoRecoverSet(bool); int32_t AutoRecoverGet(bool&);
int32_t StaticWalk();      int32_t TrotRun();       int32_t EconomicGait();
int32_t SwitchAvoidMode();
```

**API IDs** (40, from `sport_api.hpp`): 1001 DAMP, 1002 BALANCESTAND, 1003
STOPMOVE, 1004 STANDUP, 1005 STANDDOWN, 1006 RECOVERYSTAND, 1007 EULER, 1008 MOVE,
1009 SIT, 1010 RISESIT, 1015 SPEEDLEVEL, 1016 HELLO, 1017 STRETCH, 1020 CONTENT,
1022 DANCE1, 1023 DANCE2, 1027 SWITCHJOYSTICK, 1028 POSE, 1029 SCRAPE, 1030
FRONTFLIP, 1031 FRONTJUMP, 1032 FRONTPOUNCE, 1036 HEART, 1061 STATICWALK, 1062
TROTRUN, 1063 ECONOMICGAIT, 2041 LEFTFLIP, 2043 BACKFLIP, 2044 HANDSTAND, 2045
FREEWALK, 2046 FREEBOUND, 2047 FREEJUMP, 2048 FREEAVOID, 2049 CLASSICWALK, 2050
WALKUPRIGHT, 2051 CROSSSTEP, 2054 AUTORECOVERY_SET, 2055 AUTORECOVERY_GET, 2058
SWITCHAVOIDMODE.

⚠️ **NOT DOCUMENTED anywhere:** max velocity for `Move()`, the `Euler()` ranges,
the `SpeedLevel()` scale. The only numeric range in the entire SDK2 example tree
is a *commented-out* `BodyHeight(0.0); // relative height [-0.18~0.03]` — which is a
**Go1 legacy value**, and `BodyHeight` **does not exist** on the current Go2
`SportClient`. Do not use it as spec.

⚠️ `MoveForward` / `MoveBackward` / `MoveLeft` / `MoveRight` / `Rotate` /
`BodyHeight` / `SwitchGait` **do not exist** on Go2 in SDK2. The first five are
legacy `unitree_legged_sdk`; the last two exist on B2/A2/AS2 but not Go2.

### 4.2 `ObstaclesAvoidClient` — the under-advertised gem

Service `obstacles_avoid`, version `1.0.0.2`. Header
`include/unitree/robot/go2/obstacles_avoid/obstacles_avoid_client.hpp`.
Python examples ship in `unitree_sdk2_python/example/obstacles_avoid/`.

| Method | Limits |
|---|---|
| `SwitchSet(bool)` / `SwitchGet(bool&)` | Enable / query avoidance |
| `UseRemoteCommandFromApi(bool)` | **Must be `true` before `Move*` works** |
| `Move(x, y, yaw)` | x ∈ [−1.5, 1.5] m/s · y ∈ [−1.0, 1.0] m/s · yaw ∈ [−1.57, 1.57] rad/s |
| `MoveToIncrementPosition(x, y, yaw)` | ±2.0 m per axis, yaw ±1.57 rad |
| `MoveToAbsolutePosition(x, y, yaw)` | **World-frame target pose** |

⚠️ No C++ example is shipped for this — headers exist but `example/go2/CMakeLists.txt`
has no target. Python has runnable examples.

### 4.3 `VideoClient`
Service `"videohub"`, v`1.0.0.0`, API 1001 `GETIMAGESAMPLE`.
`int32_t GetImageSample(std::vector<uint8_t>&)` — returns **one compressed JPEG
frame**. Not a stream. The example writes a timestamped `.jpg` and `sleep(3)`.
Real streaming requires **GStreamer UDP H.264**.

### 4.4 `UTrackClient` — ISS 2.0
Service `"uwbswitch"`, v`1.0.0.1`, APIs 1001 SET / 1002 GET / 1003 IS_TRACKING.
Three methods: `SwitchSet(bool)`, `SwitchGet(bool&)`, `IsTracking(bool&)`.
**No target-pose API. No velocity API. No setpoint API.** ISS 2.0 is a switchable
consumer mode, not an SDK.

### 4.5 `RobotStateClient`
`ServiceList` confirms these internal services exist: `sport_mode` (<v1.1.6) /
`mcf` (≥v1.1.6), `basic_service`, `chat_go`, `net_switcher`,
`obstacles_avoidance`, `ota_box`, `unitree_lidar`, `utrack`, `video_hub`,
`webrtc_bridge`, `vui_service`.

⚠️ **Low-level control requires disabling `mcf`** (app: Device → Service Status, or
`MotionSwitcherClient`), or the robot receives two command streams.

### 4.6 Go2 SLAM / navigation service — **EDU + dock + external LiDAR only**

Service `slam_operate`, version `1.0.0.1`. Official doc states verbatim:

> "**The SLAM navigation service interface only supports the EDU robot dogs with
> expansion dock and the official Unitree-purchased lidar (MID-360/XT16) versions
> !!!**"

| API ID | Function |
|---|---|
| 1801 | start mapping |
| 1802 | end mapping |
| 1804 | init pose |
| 1102 | pose navigation |
| 1201 | pause |
| 1202 | resume |
| 1901 | close slam |

| Property | Value |
|---|---|
| Deploy location | Expansion dock PC, default `192.168.123.18` |
| LiDAR IP | `192.168.123.20` |
| Navigation speed | Go2 0.2–1.0 m/s (Go2_W 0.2–1.5) |
| Obstacle mode | `1` = stop, `0` = bypass; navigable width ≥ 0.8 m; obstacles ≤ 0.5 m wide |
| Range limit | Target ≤ 10 m from current position, straight line |
| Obstacle height | ≥ 20 cm (LiDAR FOV limit) |
| Point clouds | `rt/unitree/slam_mapping/points`, `rt/unitree/slam_relocation/points` |
| Odometry | `rt/unitree/slam_mapping/odom`, `rt/unitree/slam_relocation/odom` |
| Status | `rt/slam_info`, `rt/slam_key_info` |
| Control rate | 500 Hz low-level, 500 Hz high-level |
| Firmware floor | **≥ V1.1.6** |

⚠️ **Operational gotchas:** requires CycloneDDS and **cannot run inside an
ROS/ROS2-initialised process**; requires a **YSN per LiDAR/robot combination** from
Unitree; Mid-360 extrinsics in the Go2 IMU frame are `(0.1870, 0, 0.0803)` with a
**13° Y-axis rotation**; config lives at
`/unitree/module/unitree_slam/config/slam_interfaces_server_config/param.yaml`.
The update log shows YSN verification added 2025-09-02 and Mid-360 adaptation
2026-04-01 — this service was still being patched within six months.

⚠️ **The built-in L2 is NOT the navigation LiDAR.** That is a separate purchase.

### 4.7 Go2 LiDAR topics — no client needed

VERIFIED from the official `LiDAR_service` doc. The onboard service publishes to
DDS automatically at power-on:

| Topic | Content |
|---|---|
| `rt/utlidar/cloud` | Raw L2 point cloud, frame `utlidar_lidar` |
| `rt/utlidar/cloud_deskewed` | Motion-deskewed, frame `odom` |
| `rt/utlidar/height_map_array` | 2.5D height map |
| `rt/utlidar/range_info` | Front/left/right obstacle distances |
| `rt/utlidar/switch` | `"ON"` / `"OFF"` |

Type: `PointCloud2_` (ROS 2 `sensor_msgs/msg/PointCloud2`-compatible).
⚠️ `unitree_sdk2` contains **zero** `utlidar` references and ships no L2 client —
subscribe with a generic `ChannelSubscriber`.
⚠️ **Height map dies if you take over motion control** (see §7).

---

## 5. G1 interfaces — verbatim

### 5.1 `LocoClient`

API version `1.0.0.0`. DDS service name is `"sport"` despite the `loco` namespace.

```cpp
// low-level
GetFsmId(int&); GetFsmMode(int&); GetBalanceMode(int&);
GetSwingHeight(float&); GetStandHeight(float&); GetPhase(std::vector<float>&);
SetFsmId(int); SetBalanceMode(int); SetSwingHeight(float); SetStandHeight(float);
SetVelocity(float vx, float vy, float omega, float duration = 1.f);
SetTaskId(int); SetSpeedMode(int);
SwitchToUserCtrl(); SwitchToInternalCtrl(InternalFsmMode);

// high-level (marked as such in source)
Damp(); Start(); Squat(); Sit(); StandUp(); ZeroTorque();
StopMove(); HighStand(); LowStand();
Move(float vx, float vy, float vyaw, bool continous_move);
Move(float vx, float vy, float vyaw);
BalanceStand(); ContinuousGait(bool); SwitchMoveMode(bool);
WaveHand(bool turn_flag = false); ShakeHand(int stage = -1);
SetSpeedMode(int); GetMimicMotion(std::string&);
```

**API IDs**: 7001 GET_FSM_ID, 7002 GET_FSM_MODE, 7003 GET_BALANCE_MODE, 7004
GET_SWING_HEIGHT, 7005 GET_STAND_HEIGHT, 7006 GET_PHASE (deprecated), 7100
LOCO_FSM_API, 7101 SET_FSM_ID, 7102 SET_BALANCE_MODE, 7103 SET_SWING_HEIGHT, 7104
SET_STAND_HEIGHT, 7105 SET_VELOCITY, 7106 SET_ARM_TASK, 7107 SET_SPEED_MODE,
7110 SWITCH_TO_USER_CTRL, 7111 SWITCH_TO_INTERNAL_CTRL.

**Documented `SetSpeedMode` caps** (VERIFIED, official doc):
**mode 0 = 1.0 m/s · mode 1 = 2.0 m/s · mode 2 = 2.7 m/s · mode 3 = 3.0 m/s.**
This is the only numeric locomotion limit published for the G1.

**FSM table** (VERIFIED, doc + source): 0 zero-torque, 1 damping, 2 squat,
3 sit, 4 lock-stand, 500 walk, 501 walk+3-DOF waist, 702 lie-down/stand-up,
706 squat/stand, 801 run (**→ 802** on firmware after 8.6.x.x), 550+NNN mimic.

⚠️ Safety note from the official doc on `SwitchToInternalCtrl`:
*"make sure that both the first and last actions of your motion control are in a
standard standing posture! Otherwise, the robot may lose control."*

⚠️ **Python/C++ divergence:** Python's `LocoClient` is **missing** `StandUp()` and
`Squat()`. It also mislabels — both `StandUp2Squat()` and `Squat2StandUp()` send FSM
706. The Python README's parity claim is false.

### 5.2 `G1ArmActionClient` — 16 animations, not an interface

Service `"arm"`, version `1.0.0.14`.
APIs: 7106 EXECUTE_ACTION, 7107 GET_ACTION_LIST, 7108 EXECUTE_CUSTOM_ACTION,
7113 STOP_CUSTOM_ACTION.

```cpp
int32_t ExecuteAction(int32_t action_id);
int32_t ExecuteAction(const std::string &action_name);   // custom teach actions
int32_t StopCustomAction();
int32_t GetActionList(std::string &data);
```

**The complete action map:**
```cpp
{"release arm", 99}, {"two-hand kiss", 11}, {"left kiss", 12}, {"right kiss", 12},
{"hands up", 15}, {"clap", 17}, {"high five", 18}, {"hug", 19}, {"heart", 20},
{"right heart", 21}, {"reject", 22}, {"right hand up", 23}, {"x-ray", 24},
{"face wave", 25}, {"high wave", 26}, {"shake hand", 27}
```
Note gaps at 13, 14, 16 — IDs that exist on the robot but are not in this map.

State topic `rt/arm/action/state`:
```json
{ "holding": false, "id": 99, "name": "release_arm" }
```
`holding` retains the last keyframe for up to **20 seconds**.

⚠️ Unitree's warning, verbatim: *"Some actions will not be displayed on the APP,
but can be executed by the program. **These actions may cause the robot to fall**,
so please execute them with caution."*

⚠️ **Python is missing 2 of 4 APIs** — `EXECUTE_CUSTOM_ACTION` (7108) and
`STOP_CUSTOM_ACTION` (7113) are absent. Python users cannot run or stop teach-arm
actions.

⚠️ **`unitree_arm_interface.h` does not exist** in any current official repo
(checked `unitree_sdk2`, `unitree_sdk2_python`, `unitree_ros2`, `unitree_sdk2_rs`,
`unitree_dds_wrapper`). Neither does an `ArmAction` struct nor a
`JointControl/ArmControl/GripperControl` enum. Those are legacy `arm_sdk`
vocabulary.

### 5.3 G1 joint-level arm control (raw DDS)

Topic `rt/arm_sdk`, type `unitree_hg/msg/LowCmd`. State on `rt/lowstate`.
Joint indices from `unitree_ros2/example/src/include/g1/g1.hpp`:

```cpp
enum class G1Arm7JointIndex : int {
  ... WAIST_YAW=12, WAIST_ROLL=13, WAIST_PITCH=14,
  LEFT_SHOULDER_PITCH=15, LEFT_SHOULDER_ROLL=16, LEFT_SHOULDER_YAW=17,
  LEFT_ELBOW=18, LEFT_WRIST_ROLL=19, LEFT_WRIST_PITCH=20, LEFT_WRIST_YAW=21,
  RIGHT_SHOULDER_PITCH=22, ... RIGHT_WRIST_YAW=28, NOT_USED_JOINT=29..34
};
```
`G1Arm5JointIndex` is identical but with `*_ELBOW_ROLL=19` and a hole at 20–21.

**Mandatory conditions:** 50 Hz loop, `kp=60, kd=1.5` (waist gains ×4), **CRC32**
(poly `0x04c11db7`) on every `LowCmd`, `motion_switcher` releasing the internal
controller, and — in Unitree's own example — **a physical gamepad A-button press
to arm and disarm** (a hardware interlock).

⚠️ The G1-D arm example additionally `#include`s
`unitree/robot/b2/motion_switcher/motion_switcher_client.hpp` — G1-D reuses the
**B2** motion switcher.

### 5.4 Dex3-1 hand

Topics `rt/dex3/{left,right}/cmd` (`HandCmd_`) and
`rt/dex3/{left,right}/state` (`HandState_`).
7 motors: `0 thumb_0, 1 thumb_1, 2 thumb_2, 3 middle_0, 4 middle_1, 5 index_0,
6 index_1`. 9 pressure sensors.
Mode byte: `mode |= (id & 0x0F); mode |= (status & 0x07) << 4; mode |= (timeout & 0x01) << 7;`
with `RIS_Mode_t{ uint8_t id:4; uint8_t status:3; uint8_t timeout:1; }`.

Joint limits (the only numeric ranges in SDK2):
```cpp
maxLimits_left  = { 1.05,  1.05 , 1.75,  0,     0,     0,     0    };
minLimits_left  = {-1.05, -0.724,  0,   -1.57, -1.75, -1.57, -1.75 };
maxLimits_right = { 1.05,  0.742,  0,    1.57,  1.75,  1.57,  1.75 };
minLimits_right = {-1.05, -1.05, -1.75,  0,     0,     0,     0    };
```

⚠️ **In-repo bug:** `g1_dex3_example.cpp` declares
`sub_namespace = "rt/dex3/left/state"` at file scope then overrides it with
`"rt/lf/dex3/left/state"` inside `main()`. The prose docs use `rt/dex3/.../state`.
Treat the `rt/lf/` variant as wrong.

### 5.5 `rt/hand_sdk` — third-party hands on G1

Publishes into `ai_sport` with a blend weight:
`Motor_real = weight × Hand_SDK + (1 − weight) × G1_Internal_Cmd`
with `weight ∈ [0,1]` encoded as `weight × 100` (uint8) in `cmds[0].mode`.
Used for Inspire DFQ/FTP hands (G1-EDU Ultimate C and D). **4 motors** in this
configuration — distinct from Dex3-1's 7.

### 5.6 G1 audio

Service `"voice"`, v`1.0.0.1`:
1001 TTS · 1002 ASR · 1003 START_PLAY · 1004 STOP_PLAY · 1005 GET_VOLUME ·
1006 SET_VOLUME · 1010 SET_RGB_LED.

**ASR**: offline, local, non-streaming; includes azimuth, emotion, speaker role.
Subscribe `rt/audio_msg`.
**TTS**: local offline. `speaker_id 0` = Chinese, `speaker_id 1` = English.
⚠️ **"Mixed Chinese and English modes are not supported."** This is a hard
constraint on language coverage.
Version gate: `Vui_Service ≥ 2.0.3.8`, `Vui Module ≥ 2.0.0.3`.
Streaming playback: `PlayStream(app_name, stream_id, pcm)` — PCM 16 kHz mono 16-bit.

### 5.7 G1-D AGV client — two methods total

Service `"agv"`, v`1.0.0.1`. APIs 1001 MOVE, 1002 HEIGHT_ADJUST. Errors 9101/9102/9103.

```cpp
int32_t Move(float vx, float vy, float vyaw);
//   vx ∈ [-1.5, 1.5] m/s; vy IGNORED ("not supported for AGV, ignored");
//   vyaw ∈ [-0.6, 0.6] rad/s
int32_t HeightAdjust(float vz);   // vz ∈ [-1.0, 1.0] → vz * 76.5 mm/s
```

**No odometry, no pose, no waypoints, no state feedback in the client API.**
Height control requires a hand-written PD loop against `rt/hispeed_state`
(`geometry_msgs/msg/dds_/Point32_`, height read from `.y()`), 50 ms period,
`kP=20.0 kD=0.3 kMaxVz=0.8`, **500 ms data-timeout watchdog** calling
`HeightAdjust(0.0F)`.

---

## 6. ROS 2

`unitreerobotics/unitree_ros2` — BSD-3-Clause, default branch **`master`** (not
`main`).

### What exists: exactly four packages

1. `unitree_go` — Go2/B2 messages: `LowCmd`, `LowState`, `SportModeState`,
   `IMUState`, `MotorCmd`, `WirelessController`, `AudioData`,
   `Go2FrontVideoData`, `LidarState`, `HeightMap`, `UwbState`
2. `unitree_hg` — G1/H1/H1-2/H2 messages: `LowCmd`, `LowState`, `HandCmd`,
   `HandState`, `PressSensorState`, `MainBoardState`
3. `unitree_api` — `Request`/`Response` transport
4. `unitree_ros2_example` — 23 example nodes

`example/src/package.xml` depends on: `unitree_go`, `unitree_hg`, `unitree_api`,
`rclcpp`, `std_msgs`, `rosbag2_cpp`, plus `Eigen3` and optional `yaml-cpp`.

### What does not exist

**No `nav_msgs`. No `tf2`. No `ros2_control`. No Nav2. No `slam_toolbox`. No AMCL.
No MoveIt. No costmap. No planner. No localisation.** VERIFIED against the
complete non-truncated 221-file `master` tree and the 119-file `v0.1.0` tag tree.

Widely-cited packages that **do not exist** in any official repository:
`unitree_ros2_joy`, `unitree_go2_slam`, `unitree_lidar_slam`,
`unitree_go2_camera`. Any tutorial referencing them is wrong.

`unitree_go` *defines* `Go2FrontVideoData` and `LidarState` message types — but
**message definitions are not pipelines.** There is no decode node, no driver, no
TF publisher. Your `/utlidar/cloud` in RViz comes from the robot's own onboard
process.

### Supported ROS 2 distributions: Foxy and Humble only

Confirmed by CI (`.github/workflows/build-foxy.yml`, `build-humble.yml`) and
`.devcontainer/Dockerfile-humble`, which is stronger evidence than the README.
**No Iron, no Jazzy.** Jazzy is the current LTS (May 2024–May 2029); Humble reaches
EOL May 2027. Expect port friction on Jazzy.

### Example nodes (23 build targets)

**Go2**: `go2_sport_client`, `go2_stand_example`, `go2_robot_state_client`,
`low_level_ctrl` (with `motor_crc.cpp` CRC).
⚠️ `go2_trajectory_follow` is **commented out of CMakeLists** — never built.

**G1** (the richest set): `g1_low_level_example`,
`g1_arm_sdk_dds_example` (compile-time `G1ARM5` = 13 joints / `G1ARM7` = 17),
`g1_arm_action_example`, `g1_dex3_example` (L/R independent nodes, pub
`/dex3/{side}/cmd`, sub `/lf/dex3/{side}/state` QoS 1, 7 motors + 9 pressure
sensors, state machine `INIT → ROTATE → GRIP → STOP → PRINT`),
`g1_dual_arm_example` ⚠️ **silently skipped unless yaml-cpp ≥ 0.6**,
`g1_loco_client_example`, `g1_ankle_swing_example`, `g1_audio_client_example`
(hand-rolled WAV parser, **playback only** — no capture, no ASR, no wake word).

**All**: `read_low_state`, `read_low_state_hg`, `read_motion_state`,
`read_wireless_controller`.
⚠️ `record_bag` is **commented out** of CMakeLists — must be re-enabled manually.

**B2/B2W, H1-2, H2**: `b2_sport_client`, `b2_stand_example`, `b2w_sport_client`,
`b2w_stand_example`, `low_level_ctrl_hg`, `h2_loco_client`,
`h2_ankle_swing_example`.

### The honest verdict

> A student **must build the navigation, perception, and manipulation-planning
> layers themselves.** Official Unitree support is roughly the bottom 10% of the
> stack: a DDS bridge, message definitions, state readers, a sport-mode client,
> and per-joint PD writes. Everything above that is community or our own work.

Realistic build list:
1. ✅ `unitree_ros2` for the DDS interface (BSD-3) — genuinely good, gets you out
   of the DDS weeds, which is the hardest part
2. ✅ `unitree_mujoco` for CPU-only simulation (BSD-3)
3. 🔨 `slam_toolbox` + `AMCL` + `Nav2` (from upstream ROS)
4. 🔨 FAST-LIO2 or a ROS 2 Point-LIO port — **avoid** `point_lio_unilidar`
5. 🔨 `ros2_control` hardware interface + `robot_state_publisher`/TF
6. 🔨 BehaviorTree.CPP or a deterministic state machine
7. 🔨 Perception (detection, fiducials, language grounding)

Precedent: `Adyansh04/grove-g1` (71★, BSD-3, updated 2026-10-09) demonstrates all
of these against a real G1, simulation-first — built by **one person**. That is
the honest scope.

---

## 7. Firmware and version gates

| Gate | Requirement | Consequence if unmet |
|---|---|---|
| Go2 motion interface V2.0 | **≥ V1.1.6** | Interfaces unavailable |
| Go2 `mcf` service | ≥ v1.1.6 | Sport-mode service name changes (`sport_mode` → `mcf`) |
| G1 `rt/sportmodestate` | **≥ 1.5.1** | `fsm_mode` and `task_id` unavailable |
| G1 VUI (audio) | `Vui_Service ≥ 2.0.3.8`, `Vui Module ≥ 2.0.0.3` | ASR/TTS unavailable |
| G1 FSM 801 → 802 | Firmware after 8.6.x.x | Hard-coded 801 breaks |
| Go2 SLAM `slam_operate` | YSN per LiDAR/robot combination | Service refuses to run |
| Go2 low-level control | `mcf` must be **disabled** | Robot receives two command streams |

**Pin firmware versions per project. Do not let OTA update mid-development.**
Both platforms support OTA; disable it or gate it.

---

## 8. Simulation

`unitree_mujoco` (BSD-3, 1,234★) — **the student answer.**

> "`unitree_mujoco` is a simulator developed based on `Unitree sdk2` and `mujoco`.
> Users can easily integrate the control programs developed with `Unitree_sdk2`,
> `unitree_ros2`, and `unitree_sdk2_python` into this simulator, enabling a
> **seamless transition from simulation to physical development**."

Models: `a2, as2, b2, b2w, g1, go2, go2w, h1, h1_2, h2, r1`. **CPU physics — no GPU
required.**

⚠️ Two limits:
- **"Current version only supports low-level development"** — `LowCmd`, `LowState`,
  `SportModeState`, `IMUState`
- For the **G1** there is no high-level walking controller in simulation; you must
  supply a locomotion policy. For the **Go2**, `SportModeState` is retained in
  simulation even though it is not readable on real hardware with the motion
  service disabled.

Other simulators:

| Repo | ★ | Licence | Robots | GPU |
|---|---|---|---|---|
| `unitree_rl_lab` | 1395 | Apache-2.0 | Go2, H1, G1-29dof (IsaacLab 2.3) | **Required** |
| `unitree_rl_mjlab` | 676 | Apache-2.0 | Go2, G1, R1, H2 (MuJoCo) | No / light |
| `unitree_sim_isaaclab` | 558 | Apache-2.0 | IsaacLab wrapper | **Required** |
| `unitree_rl_gym` | 3600 | BSD-3 | Go2, H1, G1 (Isaac **Gym**) | Required — **but Isaac Gym is EOL** |

⚠️ **Isaac Sim ships under the NVIDIA Omniverse License Agreement, which is not
OSI-approved** — free for research/education, with redistribution and commercial-use
restrictions. This was **not verified** in this study (NVIDIA's licence text was not
retrieved). Verify before shipping anything containing Isaac Sim assets.
`unitree_mujoco` (BSD-3) and `unitree_rl_mjlab` (Apache-2.0) avoid this entirely.

---

## 9. Python installation — the trap

**Distribution name ≠ import name.**

```bash
pip install unitree-sdk2          # ✅ CORRECT
pip install unitree_sdk2py        # ❌ 404 on PyPI
```

| Project | PyPI status |
|---|---|
| `unitree-sdk2` | ✅ 200 — the real package |
| `unitree_sdk2` | ✅ same (normalised) |
| `unitree_sdk2py` | ❌ **404** |
| `unitree-ros2` | ❌ 404 |

Metadata: version **1.0.1**, published **2025-09-09**, single release, wheel
`py3-none-any`.

⚠️ **PyPI is ~13 months behind the GitHub repo** (pushed 2026-09-21). **Install
from git**:
```bash
git clone https://github.com/unitreerobotics/unitree_sdk2_python.git
cd unitree_sdk2_python && pip3 install -e .
```

⚠️ **You must build CycloneDDS from source first**, per the README's own FAQ:
```bash
git clone https://github.com/eclipse-cyclonedds/cyclonedds -b releases/0.10.x
cd cyclonedds && mkdir build install && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=../install
cmake --build . --target install
export CYCLONEDDS_HOME=~/cyclonedds/install
```
⚠️ The hard pin `cyclonedds==0.10.2` is a 2023-era release. ⚠️ **Windows and macOS
have no documented path.**

---

## 10. Licences — summary for a school project

| Repo | Licence | Usable? |
|---|---|---|
| `unitree_sdk2` / `_python` / `_rs` / `unitree_ros2` | **BSD-3-Clause** | ✅ Cleanly |
| `unitree_mujoco` | **BSD-3-Clause** | ✅ |
| `unitree_rl_lab` / `unitree_rl_mjlab` | Apache-2.0 | ✅ |
| `grove-g1`, `go2_ros2_sdk`, `BotBrain`, `go2_ros2_toolbox`, `g1pilot` | BSD-3 / BSD-2 / MIT | ✅ |
| `unitree_ros2_to_real` | MPL-2.0 (file-level copyleft) | ⚠️ Go1-only, ROS 2 **Eloquent**, dead since 2023-01 |
| `point_lio_unilidar` | **GPL-2.0** | 🔴 **Strong copyleft** — linking forces you to release your whole stack. Plus ROS 1. |
| `unitree_slam`, `unitree_ros_to_real`, `unitree_pybullet` | **No licence** | 🔴 **No legal grant of any right.** Do not copy, modify, or redistribute. |

**Bottom line:** an entirely permissive stack is buildable —
`unitree_ros2` + `unitree_sdk2(_python)` + `unitree_mujoco` + `unitree_rl_mjlab` +
Nav2 + slam_toolbox + FAST-LIO2 + BehaviorTree.CPP.

**The licensing risk is not the licence text. It is the closed
`libunitree_sdk2.a`** — you can ship it but you cannot audit it, which matters if
this ever needs institutional IT or safety review.

---

## 11. Community projects — all community, none official

| Project | ★ | Licence | Robots | Notes |
|---|---|---|---|---|
| `abizovnuralem/go2_ros2_sdk` | 1057 | BSD-2 | Go2 | WebRTC + CycloneDDS; joint states now **1 Hz** on fw 1.1.7; being refactored |
| `botbotrobotics/BotBrain` | 374 | MIT | Legged | Web UI, teleop, nav, mapping, monitoring |
| `hucebot/g1pilot` | 289 | BSD-3 | G1 | Best G1 ROS 2 package; dual control; **joint AND Cartesian**; URDF + RViz |
| `h-naderi/unitree-go2-slam-nav2` | 189 | **none** | Go2 | Canonical reference — **unlicensed, unusable** |
| `andy-zhuo-02/go2_ros2_toolbox` | 181 | MIT | Go2 | SLAM + nav toolbox |
| `khaledgabr77/unitree_go2_ros2` | 176 | none | Go2 | Complete **ROS 2 Jazzy** integration via CHAMP |
| `Adyansh04/grove-g1` | 71 | BSD-3 | G1 | **The reference architecture** — ros2_control 50 Hz on `rt/lowcmd`, MoveIt + octomap collision checking, slam_toolbox + AMCL + Nav2, FAST-LIO2 odometry, BehaviorTree.CPP. Honest limitation: *"GR00T N1.7 does not grasp yet; that needs demonstrations recorded on this robot."* |
| `shaoxiang/awesome-unitree-robots` | 57 | none | — | Curated index |

---

## 12. Interface maturity summary

| Capability | Go2 | G1 |
|---|---|---|
| High-level locomotion | ✅ 41 typed methods, clean velocity API | ⚠️ ~17 methods, opaque FSM integers |
| Autonomous navigation | ✅ `slam_operate` documented (EDU only) | ❌ None published |
| Programmable obstacle avoidance | ✅ `ObstaclesAvoidClient`, documented limits | ❌ None published |
| LiDAR point cloud | ✅ DDS topics, ROS 2-compatible type | ❌ No client |
| Camera | ✅ Snapshot + GStreamer streaming | ❌ No video client |
| Depth camera | ✅ EDU only, model unspecified | ⚠️ Listed, model unspecified |
| Arm — high level | ❌ No arm | ⚠️ 16 opaque animations |
| Arm — joint level | ❌ | ⚠️ Raw DDS + CRC + gamepad interlock + manual indices |
| Hand control | ❌ | ✅ Dex3-1 (7 DOF + 9 pressure sensors), 4 third-party hands |
| Audio TTS/ASR | ⚠️ `vui` = volume + brightness only | ✅ `voice` = offline ASR + TTS |
| Simulation parity | ✅ CPU-only, same DDS | ⚠️ Low-level only; needs locomotion policy |
| Python SDK completeness | ✅ Fuller | ❌ Subset; README out of date |
| ROS 2 example depth | ⚠️ 4 nodes | ✅ 8 nodes |

**The G1 has the deeper ROS 2 example set. The Go2 has the deeper, better
documented, and more complete SDK surface.** For this project the second matters
more.
