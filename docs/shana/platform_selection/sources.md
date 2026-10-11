# Sources

Every source consulted, with title, URL, type, the claims it supports, the date
accessed, and its limitations.

**All sources accessed 2026-10-10** unless noted.

---

## Method and its limits

**Tools used:** direct page fetch, `curl` against the GitHub REST API and
`raw.githubusercontent.com`, and WebSearch.

**Limitations — read before relying on any claim below:**

1. **`support.unitree.com` is WAF-blocked to direct fetch.** HTTP 567 bot
   challenge from `TencentEdgeOne`. Content was retrieved through a text-render
   proxy. URLs cited are the **origin** URLs; content is the origin's rendered
   page. Treat as high-confidence but proxied.
2. **`api.github.com` rate-limits unauthenticated requests** (60/hour). Directory
   listings were obtained via the authenticated `gh` CLI and via `codeload`
   tarballs. File contents came from `raw.githubusercontent.com`, which is
   unaffected.
3. **No code was built and no repository was cloned.** All findings are static
   analysis of READMEs, manifests, CMake files, and source files. **Runtime
   behaviour is UNVERIFIED for every claim in this study.**
4. **No robot was inspected, powered, or connected to.** All hardware facts come
   from published documentation.
5. **Contradictions are recorded in §5, not silently resolved.**

**Evidence labels:** VERIFIED (primary source read directly) · INFERRED (follows
logically) · UNVERIFIED (claimed but unconfirmed) · NOT FOUND (searched, not
located) · CONTRADICTED (sources disagree, documented below).

---

## 1. Official manufacturer sources

### 1.1 Product pages

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 1 | Unitree G1 product page | `https://www.unitree.com/g1/` | Official marketing + spec table | G1 vs G1 EDU spec table (34 rows × 3 cells, **no merged cells**). **`Secondary Development: /` for standard G1** — the gating fact. `High Computing Power Module: /`. `Single Hand DOF: /`. Arm load 2 vs 3 kg. Knee torque 90 vs 120 N·m. 9000 mAh, ~2 h. Depth camera + 3D LiDAR listed for **both**. Price $13.5K / contact sales. Warranty 8 vs 18 months. | **Marketing page.** The merged-cell question was resolved by structural HTML parsing, not by prose. Unitree publishes no SKU-level bill of materials. **The support site carries the identical rows, meaning the docs were copied from marketing, not independently verified.** |
| 2 | Unitree Go2 product page | `https://www.unitree.com/go2/` | Official marketing + spec table | Go2 AIR/PRO/X/EDU matrix (38 rows × 4 cells, **no merges**). Payload ≈7/8/8/8 kg. Speed 2.5/3.5/3.7/3.7 m/s. Slope 30°/40°. **Secondary development ○/○/◕/●**. Depth camera EDU-only. Battery 8000/8000/8000/**15000** mAh. Warranty 6/12/12/12 months. Prices $1600/$2800/$4500/**contact sales**. "3D LiDAR Mapping … with a dedicated App". ISS 2.0 "over 30m". | Marketing. **Camera fps contradicted internally** (see §5). Long-battery runtime quoted three different ways. |
| 3 | Official Open Source index | `https://www.unitree.com/opensource/` | Official index | Entry point; **page body is JS-rendered and returned no content** on direct fetch. Repository data was obtained from the GitHub API instead. | Not usable as a source directly. |

### 1.2 Developer documentation

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 4 | **Unitree Developer FAQ** | `https://support.unitree.com/home/en/developer/faq` | Official docs | **THE GATING FACT, verified verbatim:** *"Only EDU or above versions can be developed, other versions such as PRO and AIR versions do not open the secondary development interface, EDU version can be equipped with 40tops or 100tops docking station for development."* Also: Ubuntu 20.04 required; Mac/Windows not supported; Go2 wired-only; **camera 1080p@30 / 720p@60** (contradicting source 2). | Proxied retrieval. |
| 5 | About Go2 | `https://support.unitree.com/home/en/developer/about%20Go2` | Official docs | The `●` / `◕` / `○` legend. Lithium battery specifications (BT2-05 8000 mAh / 236.8 Wh / 2 kg / 3.5 A / 2 h; BT2-06 15000 mAh / 432 Wh / 2.5 kg / 9 A / 1 h). L1 and L2 LiDAR specs. Camera F2.2 / 120° / **15 fps**. Civilian-robot disclaimer. | Proxied. Camera fps conflicts with source 4. |
| 6 | SLAM and Navigation service | `https://support.unitree.com/home/en/developer/SLAM%20and%20Navigation_service` | Official docs | **The Go2 navigation capability.** `slam_operate` v`1.0.0.1`; API IDs 1801/1802/1804/1102/1201/1202/1901. *"only supports the EDU robot dogs with expansion dock and the official Unitree-purchased lidar (MID-360/XT16)"*. Speed 0.2–1.0 m/s. ≥0.8 m width; ≤0.5 m obstacles; ≥20 cm height; ≤10 m range. Dock IP `.18`, LiDAR IP `.20`. Topics. YSN requirement. Mid-360 extrinsics `(0.1870, 0, 0.0803)`, 13° Y rotation. | Proxied. **Interaction with a ROS 2 graph is undocumented.** Patch log shows updates as recent as 2026-04. |
| 7 | LiDAR service | `https://support.unitree.com/home/en/developer/LiDAR%20Service` | Official docs | Go2 LiDAR publishes to DDS at power-on: `rt/utlidar/cloud`, `cloud_deskewed`, `height_map_array`, `range_info`, `switch`. Type `PointCloud2_`. ⚠️ *"If the operation control is turned off … the height map data cannot be obtained."* | Proxied. |
| 8 | Obstacles Avoid Client | `https://support.unitree.com/home/en/developer/ObstaclesAvoidClient` | Official docs | `obstacles_avoid` v`1.0.0.2`. `Move(x,y,yaw)` x ±1.5 m/s, y ±1.0 m/s, yaw ±1.57 rad/s. `MoveToIncrementPosition` ±2.0 m. `MoveToAbsolutePosition` world frame. `UseRemoteCommandFromApi` required first. | Proxied. Ordering requirement is implied, not stated as a precondition. |
| 9 | Motion Services Interface V2.0 | `https://support.unitree.com/home/en/developer/Motion_Services_Interface_V2.0` | Official docs | Go2 gait semantics and cautions. *"These interfaces apply to Go2 Edu models with software version ≥ V1.1.6."* `TrotRun` "somewhat risky". `StaticWalk` "does not have the ability to handle complex terrain". `ClassicWalk` "supports climbing stairs, gravel, meadows, and slippery surfaces". `WalkUpright`/`CrossStep` overheating caution. `EconomicGait` "up to 4 hours". | Proxied. |
| 10 | AI motion service | `https://support.unitree.com/home/en/developer/AI_motion_service` | Official docs | `FreeAvoid(2048)`: *"when stationary, it can dodge objects in front."* `SwitchAvoidMode(2058)`: *"It is generally not recommended to use this mode."* | Proxied. **The uncommanded-dodge hazard is documented by the manufacturer.** |
| 11 | Payload | `https://support.unitree.com/home/en/developer/Payload` | Official docs | *"The Go2 Pro and Go2 Air versions do not support any of the extended electrical interfaces, including the SBUS port, DC 28.8V power output, and the Ethernet (RJ45) interface. These interfaces are exclusive to the Go2 EDU version."* Expansion dock M3×50 mounting. `AutoRecoverSet` payload warning. | Proxied. **Back-load mounting dimensions are published as PNG only — UNVERIFIED as text.** Contradicts the spec table on X's RC (see §5). |
| 12 | Architecture Description | `https://support.unitree.com/home/en/developer/Architecture%20Description` | Official docs | *"the EDU version can call the interface through DDS or ROS2."* GStreamer for image transmission. | Proxied. |
| 13 | Multimedia Services | `https://support.unitree.com/home/en/developer/Multimedia_Services` | Official docs | GStreamer UDP H.264 pipeline: `udpsrc address=230.1.1.1 port=… ! application/x-rtp,media=video,encoding-name=H264 ! rtph264depay ! … ! appsink drop=1`. | Proxied. |
| 14 | G1 sport services interface | `https://support.unitree.com/home/en/G1_developer/sport_services_interface` | Official docs | **`SetSpeedMode` caps: 0 = 1.0 m/s, 1 = 2.0, 2 = 2.7, 3 = 3.0 m/s.** FSM table. `SwitchToInternalCtrl` standing-posture warning. *"The 29dof device ai_sport was updated to version 802 after version 8.6.x.x."* `rt/sportmodestate` requires firmware ≥ 1.5.1. | Proxied. |
| 15 | G1 Vui / audio client | `https://support.unitree.com/home/en/G1_developer/VuiClient_Service` | Official docs | Offline ASR and TTS on the G1. `speaker_id 0` = Chinese, `1` = English. **"Mixed Chinese and English modes are not supported."** PCM 16 kHz. `rt/audio_msg`. Version gates. | Proxied. |
| 16 | G1 obtain-SDK page | `https://support.unitree.com/home/en/G1_developer/get_sdk` | Official docs | G1-EDU sub-configurations: Standard 23 DOF, Advanced 29, Ultimate A (Dex3-1 no tactile), B (tactile), C (Inspire DFQ), D (Inspire FTP). | Proxied. Contains typos ("Andvanced"), indicating low editorial quality. |
| 17 | G1-D developer page | `https://support.unitree.com/home/en/G1D_Developer/about` | Official docs | G1-D chassis autonomy: laser SLAM 500×500 m, ±5 cm positioning, laser-tag docking, HTTP REST API, RoboStudio. **Chassis HDMI and USB "not currently open."** LIVOX-MID360 LiDAR named. Chassis 557×525 mm, 1.5 m/s, 10° incline. | Proxied. **Internally contradictory on weight and column speed (see §5). HTTP API contract unpublished.** |
| 18 | Unitree Camera SDK page | `https://support.unitree.com/home/en/G1_developer/VuiClient_Service` and camera docs | Official docs | G1 head-camera topics including `rt/utlidar/cloud_livox_mid360`. | Proxied. **The G1 depth camera model is NOT named in any official source.** |

### 1.3 Unitree-sourced third-party material

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 19 | G1 User Manual (EN) | `https://www.docs.mybotshop.de/downloads/G1_User_Manual.pdf` — mirror `https://sonnyrobotics.com/cdn/shop/files/Unitree_G1_FCC_Manual.pdf` | Vendor manual (reseller-hosted) | ⚠️ **Mandatory protective frame and protective rope** (three verbatim warnings). ≥2 m safe distance. "Uncontrolled to the crowd". Under-18 restriction. Battery temperature limits (discharge −20–60 °C, charge 0–55 °C, "may cause the battery to ignite or even explode"). Not waterproof / not dustproof. 0–40 °C. Emergency stop `L1+A`. | ⚠️ **CONTAINS AN EMBEDDED QUADRUPED SAFETY SECTION** (5–35 °C, 25° slopes, 10 cm steps, 3 m, "no visual perception system"). **This is Go2/B2 content in a G1 document and must not be attributed to the G1.** Emergency-stop combo is internally inconsistent (`L1+A` vs `L2+B`). Reseller-hosted, not a vendor-controlled URL. |
| 20 | G1 product warranty terms | Referenced by `unitree.com/g1` | Vendor terms | Repeats the 2 m safe-distance requirement. | Not separately retrieved. |

---

## 2. Official repositories

All under `github.com/unitreerobotics`. Licences from `LICENSE` files where marked
**[L-fetched]**; others from the GitHub API `license.spdx_id` field.

### 2.1 Core SDKs

| # | Repository | URL | Licence | Claims supported | Limitations |
|---|---|---|---|---|---|
| 21 | `unitree_sdk2` ★1395 | `https://github.com/unitreerobotics/unitree_sdk2` | **BSD-3-Clause [L-fetched]** | CycloneDDS transport. `ChannelFactory::Init(0, argv[1])`. 15 example dirs, 11 robot families. `SportClient` 41 methods + 40 API IDs. `LocoClient` full method set + 12 API IDs. `G1ArmActionClient` 16-entry action map. `VideoClient::GetImageSample`. `UtrackClient` 3 methods. `RoboticArmClient`-absent. `g1/agv/` 3 files, 2 methods. Dex3 example with 7 motors + joint-limit arrays. `g1d_height_control.cpp` PD loop (kP=20, kD=0.3, kMaxVz=0.8, 500 ms timeout). Closed `libunitree_sdk2.a` (27.6 MB x86_64 / 29.1 MB aarch64). `dt=0.005` (200 Hz). | **The README is 40 lines of build instructions with zero API documentation** — "no locomotion algorithm work required" is nowhere stated, only inferred. `ChannelFactory` implementation is in the closed `.a`. |
| 22 | `unitree_sdk2_python` ★820 | `https://github.com/unitreerobotics/unitree_sdk2_python` | **BSD-3-Clause [L-fetched]** | `setup.py` declares `name='unitree_sdk2py'`. README dependencies `cyclonedds==0.10.2`. CycloneDDS build instructions. `obstacles_avoid` Python examples. `camera_opencv.py`. **Python is a strict subset of C++** (missing `StandUp`, `Squat`, arm custom actions). C++/Python disagree on `right kiss` ID and the JSON payload key. | ⚠️ **The Python README is substantially out of date** — it references example paths that no longer exist and claims parity that does not hold. |
| 23 | `unitree_sdk2_rs` ★5 | `https://github.com/unitreerobotics/unitree_sdk2_rs` | BSD-3-Clause (Cargo.toml); crates.io metadata **null** | crates.io `unitree_sdk2_rs` v0.3.0, created 2026-07-31, **8 versions in 5 days**, 183 total downloads. Build-time GitHub clone of SDK2. | **Experimental.** Inherits the closed-binary limitation and adds a heavy build-time network dependency. |
| 24 | `unitree_dds_wrapper` ★34 | `https://github.com/unitreerobotics/unitree_dds_wrapper` | BSD-3-Clause **[L-fetched]** | Legacy wrapper; contains only `ArmString_` and `SetServoAngle_` (the old G1-EDU 7-DOF `arm_sdk`). Confirms `unitree_arm_interface.h` / `ArmAction` / control-mode enums are **legacy-only**. | **Stale — last push 2024-10-16.** Superseded by SDK2. |

### 2.2 ROS 2, simulation, models

| # | Repository | URL | Licence | Claims supported | Limitations |
|---|---|---|---|---|---|
| 25 | `unitree_ros2` ★858 | `https://github.com/unitreerobotics/unitree_ros2` (default branch **`master`**) | **BSD-3-Clause [L-fetched]** | **4 packages only**: `unitree_go`, `unitree_hg`, `unitree_api`, `unitree_ros2_example`. 23 build targets. **No nav2/slam_toolbox/amcl/tf2/nav_msgs/ros2_control dependency.** `unitree_ros2_joy` / `unitree_go2_slam` / `unitree_lidar_slam` / `unitree_go2_camera` **do not exist** (verified against the complete 221-file `master` tree and the 119-file `v0.1.0` tag tree). G1 joint indices. `g1_dex3_example` pub/sub topics. Foxy/Humble only. `record_bag` commented out of CMakeLists. `g1_dual_arm_example` conditional on yaml-cpp. | Last push 2026-07-02 — ~3 months behind SDK2. README robot list is stale. |
| 26 | `unitree_mujoco` ★1234 | `https://github.com/unitreerobotics/unitree_mujoco` (note: **`readme.md`**, lowercase) | **BSD-3-Clause [L-fetched]** | *"Users can easily integrate the control programs developed with Unitree_sdk2, unitree_ros2, and unitree_sdk2_python into this simulator, enabling a seamless transition from simulation to physical development."* Models: a2, as2, b2, b2w, g1, go2, go2w, h1, h1_2, h2, r1. **CPU physics.** *"Current version only supports low-level development."* | **`README.md` returns 404** — the file is lowercase. No high-level locomotion controller, so the G1 needs a policy. |
| 27 | `unitree_model` ★156 | `https://github.com/unitreerobotics/unitree_model` | BSD-3 | URDF models. | Not inspected in detail. |
| 28 | `unitree_rl_lab` ★1395 | `https://github.com/unitreerobotics/unitree_rl_lab` | Apache-2.0 | Go2, H1, G1-29dof in IsaacLab. **GPU required.** | Isaac Sim licence **NOT VERIFIED** (see §4). |
| 29 | `unitree_rl_mjlab` ★676 | `https://github.com/unitreerobotics/unitree_rl_mjlab` | Apache-2.0 | Go2, G1, R1, H2 on MuJoCo. No/light GPU. | — |
| 30 | `unitree_sim_isaaclab` ★558 | `https://github.com/unitreerobotics/unitree_sim_isaaclab` | Apache-2.0 **[L-fetched]** | IsaacLab wrapper. | NVIDIA Omniverse licence applies separately and was **not verified**. |
| 31 | `unitree_rl_gym` ★3600 | `https://github.com/unitreerobotics/unitree_rl_gym` | BSD-3 | Go2, H1, G1 in Isaac Gym. | **Isaac Gym is EOL**; untouched since 2025-07-25. |
| 32 | `unitree_rl_gym` / `unitree_pybullet` | `https://github.com/unitreerobotics/unitree_pybullet` | **None** | — | **No LICENSE file** (verified by 404 on all variants). Dead since 2020-10-15. |

### 2.3 Perception, hands, teleoperation

| # | Repository | URL | Licence | Claims supported | Limitations |
|---|---|---|---|---|---|
| 33 | `unilidar_sdk` ★142 | `https://github.com/unitreerobotics/unilidar_sdk` | BSD-3 | **L1** standalone LiDAR SDK. | Stale since 2024-02. Standalone workflow (192.168.1.2 / ttyACM0) — **not documented as reaching the Go2's internal L2.** |
| 34 | `unilidar_sdk2` ★106 | `https://github.com/unitreerobotics/unilidar_sdk2` | BSD-3 | **L2** standalone LiDAR SDK; `setLidarWorkMode`; publishes `unilidar/cloud`, `unilidar/imu`. | **FOV contradiction:** README reports 180°/192°, docs say 96°. Whether it reaches the Go2's internal L2 is **UNVERIFIED**. |
| 35 | `UnitreecameraSDK` ★122 | `https://github.com/unitreerobotics/UnitreecameraSDK` | MPL-2.0 | *"UnitreeCameraSDK 1.1.0 is a cross-platform library for unitree stereo cameras."* | **Go1 only.** Not for G1 or Go2. |
| 36 | `avp_teleoperate` ★1696 | `https://github.com/unitreerobotics/avp_teleoperate` | NOASSERTION | XR teleoperation: **G1 23 DoF ✅ complete, G1 29 DoF ✅**, H1, H2, R1. Apple Vision Pro, PICO 4 Ultra, Quest 3. Demo shows G1 29DoF + Dex3-1. | **Licence NOASSERTION — verify before use.** Only teleoperation; no autonomy. |
| 37 | `unitree_lerobot` ★775 | `https://github.com/unitreerobotics/unitree_lerobot` | NOASSERTION | LeRobot fork; supports `pi05` and `groot`; v0.2 adds BrainCo and Inspire hands; v0.1 real-world G1 + Dex1 + Dex3. | **Licence NOASSERTION.** A data/training/eval harness, **not** a teleoperation stack. |
| 38 | `dex1_1_service` ★20 | `https://github.com/unitreerobotics/dex1_1_service` | BSD-3 | G1 + Dex1-1 hand bridge. C++ only. | Consumes `unitree_go` (`MotorCmds_`) messages **on a G1**. |
| 39 | `brainco_hand_service` ★18 | `https://github.com/unitreerobotics/brainco_hand_service` | **None** | BrainCo Revo2 on G1; topics `rt/brainco/{side}/{cmd,state}`. | **No licence.** C++ only. |
| 40 | `linker_hand_service` ★3 | `https://github.com/unitreerobotics/linker_hand_service` | **None** | Linkerbot O6 on G1; Modbus RTU; topics `rt/linker/{side}/{cmd,state}`. | **No licence.** C++ only. |

### 2.4 Repositories to avoid

| # | Repository | URL | Licence | Reason for exclusion |
|---|---|---|---|---|
| 41 | `unitree_slam` ★2 | `https://github.com/unitreerobotics/unitree_slam` | **None — NO LICENSE FILE** | Verified: 796-file tree contains zero LICENSE files. **No legal grant of any right.** Ships closed prebuilt binaries. CMake targets `demo_h1 demo_b2 demo_xt16 demo_mid360` — **H1 and B2 only, NOT Go2, NOT G1**. Created 2026-07-21, 2★. **Looks like official navigation and is not usable.** |
| 42 | `point_lio_unilidar` ★532 | `https://github.com/unitreerobotics/point_lio_unilidar` | **GPL-2.0** | **Strong copyleft** — linking forces release of the entire combined work. **And ROS 1 Noetic** (catkin), incompatible with ROS 2 Humble/Jazzy. Last push 2025-06-05. |
| 43 | `unitree_ros_to_real` ★174 | `https://github.com/unitreerobotics/unitree_ros_to_real` | **None** | 404 on LICENSE, LICENSE.txt, LICENSE.md, COPYING. ROS 1 Melodic. Dead since 2023-06. |
| 44 | `unitree_ros2_to_real` ★89 | `https://github.com/unitreerobotics/unitree_ros2_to_real` | **MPL-2.0 [L-fetched]** | File-level copyleft. **Go1-only, ROS 2 Eloquent (2020), last push 2023-01-13.** Unusable. |
| 45 | `unitree_guide` ★637 | `https://github.com/unitreerobotics/unitree_guide` | BSD-3 | **Not SDK2.** ROS 1 Melodic + Gazebo quadruped controller. No current robot support. Last push 2024-07-22. |
| 46 | `unifolm-vla`, `unifolm-wla`, `unifolm-world-model-action` | `https://github.com/unitreerobotics/unifolm-vla` etc. | None / NOASSERTION | **No technical documentation, model card, licence, or API reference published.** UNVERIFIED. |

---

## 3. Peer-reviewed and preprints

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 47 | **AutodidactWAM** (arXiv 2610.08119) | `https://arxiv.org/abs/2610.08119` | Preprint | **THE decisive manipulation measurement.** Real Unitree G1, closed-loop physical trials, BrainCo hands. Native foundation-model action: ~17% pre-grasp, ~10% grasp, **~7% pick-and-place**. Recovered-action gate: ~75/47/**~42%**. Best hybrid (DPO+SFT+DTW) on held-out object: **20–30% full-task**. | **Preprint, not peer-reviewed.** BrainCo hands, **not** the Dex3-1. Cumulative-gate figures are not full-task. |
| 48 | **AgenticTactileVLA** (arXiv 2610.04391) | `https://arxiv.org/abs/2610.04391` | Preprint | Strongest positive result. Real G1 + BrainCo Revo2. 5 objects **held out from VLA training**. Base VLA **61.3%**; unconditional close-to-stall **72.0%**; + contact-guided supervisor **84.0%**. | Preprint. **A fixed VLA on 5 known object types plus a runtime supervisor — not generalist manipulation, and not the Dex3-1.** |
| 49 | **"Brace Yourself"** (arXiv 2609.25486) | `https://arxiv.org/abs/2609.25486` | Preprint | **Measured on a real G1: 13.5 N maximum sustained end-effector force unbraced; 60 N with environmental bracing.** The single most decision-relevant hardware number. | Preprint. One measurement, one bracing condition. |
| 50 | **HumanoidToolBench** (arXiv 2610.02089) | `https://arxiv.org/abs/2610.02089` | Preprint | GR00T N1.7 on real G1: *"reduced selection accuracy on unseen tools and continued task execution under unrelated instructions"* — a documented instruction-following failure. | Preprint. |
| 51 | Workhorse (arXiv 2610.09117) | `https://arxiv.org/abs/2610.09117` | Preprint | Real G1 sorts boxes, catches a thrown box, topples a suitcase. 77% **simulation** success; 64% under 40 N·s pushes. | **No real-robot success rate published.** |
| 52 | Blind Dexterity (arXiv 2608.29487) | `https://arxiv.org/abs/2608.29487` | Preprint | Real G1, proprioception only, no cameras: lifts a suitcase by its handle, traps a ball, mounts a skateboard. | **No grasp rate published.** |
| 53 | BiGym 2.0 (arXiv 2610.07594) | `https://arxiv.org/abs/2610.07594` | Preprint | **Simulation only.** 20 G1 household tasks, 60 VR demos/task. π0.5 stays low on pick-box; cross-workspace stacking **remains open**. | **SIMULATION — must not be cited as hardware capability.** Shows the task is unsolved even in sim. |
| 54 | Toward Humanoid Robots in Construction (arXiv 2610.00718) | `https://arxiv.org/abs/2610.00718` | Preprint | Real G1, XR teleoperation: **100% tool transport, 80% surface painting**, with teleop substantially slower than manual. | Preprint. **Teleoperation, not autonomy.** |
| 55 | **AgiBot World / GO-1** (arXiv 2503.06669) | `https://arxiv.org/abs/2503.06669` | Preprint | ⚠️ **NAME COLLISION WARNING.** The ">60% success rate on complex tasks" figure is on **AgiBot's own robots** (100 units), **NOT the Unitree G1**. AgiBot G1 is a different robot from a different company. | **Frequently miscited as Unitree evidence. Do not use it for Unitree.** |
| 56 | RoboMIND (arXiv 2412.13877) | `https://arxiv.org/abs/2412.13877` | Preprint | 107k demos, 479 tasks. Embodiments: **Franka Panda, UR5e, AgileX dual-arm, and an unnamed humanoid. NO Unitree G1.** | — |
| 57 | RoboMIND 2.0 (arXiv 2512.24653) | `https://arxiv.org/abs/2512.24653` | Preprint | 310k+ dual-arm trajectories, 6 embodiments, 739 tasks. **NO Unitree G1.** | — |
| 58 | Whole-Body UMI (arXiv 2609.22829) | `https://arxiv.org/abs/2609.22829` | Preprint | *"collecting whole-body demonstrations for humanoid manipulation mostly relies on teleoperation, which is costly and hard to scale up."* | Preprint. |
| 59 | UMI (arXiv 2402.10329) | `https://arxiv.org/abs/2402.10329` | Preprint | Low-cost handheld data collection (~$100). Used to *feed* humanoid data. | Handheld tool only; not a robot-arm interface. |
| 60 | EquivDP3 (2609.36575), Fiatlux (2609.38216), FetchMan (2608.17027), HumanoidVLM (2601.14874), Hierarchical VLP (2506.22827) | arXiv | Preprints | Simulation-only or research-system results for G1. FetchMan: *"cloning synthetic demonstrations results in a low performance ceiling."* | **All simulation-only or lacking real-hardware rates. Not cited as capability.** |

---

## 4. Independent and community sources

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 61 | **GR00T-WholeBodyControl issue #121** | `https://github.com/NVlabs/GR00T-WholeBodyControl/issues/121` | GitHub issue | **Open since 2026-04-27.** Maintainer `swumh`: *"the G1 alert an overheating warning after a few minutes, even when it is just standing idle. After that, Sonic shuts down automatically. As a result, the robot has fallen multiple times."* NVIDIA `rylinjames` on Inspire hands: *"zero-shot on Inspire is likely to cause joint saturation and potential balance instability."* Follow-up 2026-09-21: *"no finger use for now, but it's still not perfect."* | **Single-reporter; a second user reports no such issue with their own RL policies.** May be policy-specific. **Unresolved in NVIDIA's own tracker.** |
| 62 | **GR00T-WholeBodyControl README** | `https://github.com/NVlabs/GR00T-WholeBodyControl` | Official NVIDIA | *"Decoupled WBC: the decoupled controller (RL for lower body, and **IK for upper body**) used in GR00T N1.5 and N1.6."* **GR00T's G1 arm behaviour is IK following a reference, not learned manipulation.** | Highly relevant to correcting a widespread misreport. |
| 63 | **Munro & Associates teardown** (via MotorTrend) | `https://www.motortrend.com/news/first-humanoid-robot-teardown-analysis` | Independent teardown, via news | Base G1 *"was very buggy and needed several software updates before it would do anything. Then they pushed and shoved it until it fell down, whereupon it proved unable to stand back up."* Non-strain-wave joints with backlash; *"many aluminum parts appear milled from billet and many plastic parts appear 3D printed"*; exposed greased ankle Cardan joint. Actuators partly ported from **Go2 Pro**. | **Via news report of a teardown.** Secondary engineering analysis via humanoid.guide and roboterminal. A single unit. |
| 64 | Auckland University of AUT G1 lab | `https://www.youtube.com/watch?v=5OVOnFw3rkA` | University lab video | G1 described as *"stumbling heavily… This instability makes the robot unusable."* Pelvis IMU replaced, still unstable. Unitree engineer phoned back to the factory. Hip motors needed replacement. | One lab. YouTube. No measured fall rate. |
| 65 | $100k research buyer abandons G1 | `https://mike.gold/notes/x-bookmarks/robotics/we-built-our-own-unitree-g1-after-issues` | Independent blog | *"a knee issue that required a 2-month wait for a replacement part. Two months for one part stops your entire development cycle."* *"frequent hardware failures, particularly in its arms."* | Single report, first-person. |
| 66 | **Go2 teardown** | `https://www.simplexitypd.com/blog/unitree-go2-quadruped-robot-dog-teardown/` | Independent teardown | *"emphasis on robustness and long-term durability… leg modules designed for rapid field replacement… prioritised vibration resistance, shock absorption, and overall impact survivability."* | Single teardown. No measured reliability figure. |
| 67 | Go2 field operator report | `https://travis-mendoza.com/portfolio/unitree-go2.html` | Independent operator | *"On start up, I saw the Go2 fail to stand and balance properly… it flailed its legs around wildly. This happened three times on the first day."* *"run into a door… while it was supposed to be in autonomous navigation and obstacle avoidance mode."* *"powered off completely in the middle of its movement."* ⚠️ *"The Go2 Pro and Air models do not come with the SDK enabled nor do they have pre-drilled mounting holes."* | **Single operator.** Anecdotal, but corroborates the official EDU-only gating. |
| 68 | Go2 payload tray field data | `https://robohub.app/addons/quadruped-payload-tray-molle` | Community field data | Four M5 spine hardpoints. *"Up to 3kg centred on the spine: no measurable gait change."* *"Above 3kg or off-center:… slows to 80%… balance starts looking drunk."* | **One vendor's marketing page.** Not peer-reviewed. Used only for the order of magnitude. |
| 69 | **Alias Robotics audit** (arXiv 2509.14096) | `https://arxiv.org/html/2509.14096v1` | Security audit preprint | All G1 units share **one hardcoded AES key**; a BLE exploit enables root-level takeover. Robot runs **EOL ROS 2 Foxy** with outdated CycloneDDS. | Preprint. |
| 70 | IEEE Spectrum — UniPwn | `https://spectrum.ieee.org/unitree-robot-exploit` | News | *"takeover of fleets of Unitree robots."* | News report of source 69. |
| 71 | Help Net Security | `https://www.helpnetsecurity.com/2025/10/16/unitree-g1-humanoid-robot-vulnerability/` | News | G1 transmits camera/audio/map telemetry to a **Chinese IP (43.175.228.x) every five minutes without user consent**. | News report. ⚠️ **This is a governance issue for the school and was NOT scored** in the decision matrix. |
| 72 | theresarobotforthat — G1 review | `https://theresarobotforthat.com/blog/unitree-g1-review-2026/` | Independent blog | At 1,320 mm *"it can't reach standard workbenches."* 9,000 mAh yields *"about 90 minutes of active walking."* ETH Zurich runs G1 units to transport samples between wet-lab rooms using RTAB-Map corridor navigation **and no arm at all.** Dex3-1 grip ≈30 N. UnifoLM-X1-0 factory demo shown only at 2× speed. | Blog. Corroborated where possible. |
| 73 | **Robots Rated** | `https://robotsrated.com/robots/unitree-g1/` | Aggregator | ***"No independent reviews found. No independent, finished hands-on review of this robot yet."*** | **Explicit statement of an evidence gap — valuable.** |
| 74 | **RoboPulse** | `https://robopulse.net/reviews/unitree-g1-review/` | Aggregator | ***"RoboPulse has not conducted hands-on testing — all analysis is drawn from publicly available data."*** | As above. |
| 75 | **RoboTesters** | `https://robottesters.com/robots/unitree-g1/` | Aggregator | *"Score derived from manufacturer data."* | As above. |
| 76 | IEEE Spectrum — Unitree G1 | `https://spectrum.ieee.org/unitree-g1` | News/interview | Verified by direct fetch: a **3-paragraph 2024 ICRA interview**. **No testing.** | Recorded to prevent it being cited as a review. |
| 77 | r/robotics discussions | `https://www.reddit.com/r/robotics/comments/1t74nli/` and related | Forum | Bipeds characterised as *"inherently unsafe"*; fast get-up behaviour described as *"a safety nightmare."* | Forum opinion. Low evidentiary weight; used only as corroboration of the documented safety picture. |
| 78 | r/unitree — spare parts | `https://www.reddit.com/r/unitree/comments/1j3fpwy/` | Forum | *"There is no spare parts. They dont provide it. They required you to send it for repair."* | Anecdotal. Consistent with source 65. |

### 4.1 Community repositories — all community, none official

Searches run: `unitree+go2+ros2` (150 hits), `unitree+g1+ros2` (60), `unitree+nav2` (56), all sorted by stars. **`org:unitreerobotics` + "ros2" returns only `unitree_ros2` and `unitree_ros2_to_real` — zero official community-facing navigation stacks exist.**

| # | Repository | ★ | Licence | Maintained | Notes |
|---|---|---|---|---|---|
| 79 | `abizovnuralem/go2_ros2_sdk` | 1057 | BSD-2 | ✅ 2026-07 | WebRTC + CycloneDDS. ⚠️ Joint states now **1 Hz** on fw 1.1.7. |
| 80 | `Zhefan-Xu/isaac-go2-ros2` | 594 | none | ⚠️ 2025-09 | GPU required. |
| 81 | `botbotrobotics/BotBrain` | 374 | MIT | ✅ 2026-06 | Web UI, teleop, nav, mapping. |
| 82 | `hucebot/g1pilot` | 289 | BSD-3 | ✅ 2026-06 | Best G1 ROS 2 package. Joint + Cartesian control. |
| 83 | `h-naderi/unitree-go2-slam-nav2` | 189 | **none** | ❌ 19 mo | Canonical reference — **unlicensed, unusable.** |
| 84 | `andy-zhuo-02/go2_ros2_toolbox` | 181 | MIT | ✅ 2025-11 | SLAM + nav toolbox. |
| 85 | `khaledgabr77/unitree_go2_ros2` | 176 | none | ✅ 2026-09 | ROS 2 **Jazzy** via CHAMP. |
| 86 | `Adyansh04/grove-g1` | 71 | **BSD-3 [L-fetched]** | ✅ 2026-10-09 | **The reference G1 architecture** — ros2_control 50 Hz on `rt/lowcmd`, MoveIt + octomap collision checking, slam_toolbox + AMCL + Nav2, FAST-LIO2, BehaviorTree.CPP, simulation-first. Honest limitation: *"GR00T N1.7 does not grasp yet."* |
| 87 | `shaoxiang/awesome-unitree-robots` | 57 | none | — | Curated index. |

### 4.2 Dataset sources

| # | Source | URL | Claims | Limitations |
|---|---|---|---|---|
| 88 | **Unitree HuggingFace org** | `https://huggingface.co/unitreerobotics` | **~180 G1 manipulation datasets.** Families: `G1_Dex3_*`, `G1_Brainco_*`, `G1_WBT_Inspire_*`, `G1_Dex1_*`, `G1_WBT_*`, plus `R1_Dex1_*`, `Z1_Dual_Dex1_*`. Scale example: `G1_Dex3_ToastedBread_Dataset` — 418 episodes, 352,022 frames, 30 Hz, 640×480, wrist + head binocular cameras, LeRobot v2.0. | ⚠️ **Dataset cards report episode COUNTS, not SUCCESS RATES.** Instructions note objects must be placed *"within the fully extended reach"* at ~45°, and that *"Data collection is not completed in a single session."* Download counts are low (hundreds to ~7k); likes 0–12 — **not broadly validated by third parties.** |
| 89 | G1 pick/grasp dataset family enumeration | Derived from source 88 | Pick: apple, bottle, charger, gum, snack, tissue, doll, drink, toothpaste. Grasp: square, Oreo, Rubik's cube. **NO flat-sheet, notebook, book, magazine, or thin-panel task anywhere.** | **Inference from absence.** Strong negative signal, but not proof that grasping a notebook is impossible. |

---

## 5. Documented contradictions

Reported, not resolved. Each needs a call to Unitree or a physical check.

| # | Item | Source A | Source B | Handling |
|---|---|---|---|---|
| C1 | Go2 L2 vertical FOV | `unitree.com/go2` + `about_Go2`: 360°×96° | `unilidar_sdk2` README: 180° / 192° | Likely ±96° half-angle vs full FOV, but **no document states this.** UNVERIFIED. |
| C2 | Go2 camera frame rate | `about_Go2`: 720p/1080p **15 fps** | FAQ: 1080p **30 fps** / 720p **60 fps** | **Must be measured in Phase 2.** Design assumes the worse figure. |
| C3 | Go2 long-battery runtime | Spec table: 2–4 h | Battery table: 3–5 h; `EconomicGait` doc: "up to 4 hours" | **Plan on ~2 h** — the most conservative official figure. |
| C4 | Go2 LiDAR dimensionality | Website: "4D" | `about_Go2` spec row: "3D" (same page's LIDAR section says 4D) | Cosmetic. L2 is documented as 4D elsewhere. |
| C5 | ISS 2.0 naming | Website: "Intelligent Side-follow System" | `about_Go2`: "Intelligent Companion" | Cosmetic. |
| C6 | Go2 X remote controller | Spec table: manual controller **●** | `Payload`/`about_Go2`: SBUS interface is **EDU-exclusive** | X likely has a proprietary RC. **Ask Unitree.** |
| C7 | G1 EDU compute module | `unitree.com/g1`: "Multiple brands… Orin" | Dev doc: "NVIDIA Jetson Orin" | Dev doc is stale. |
| C8 | **G1-D column speed** | Marketing + docs: ~60 mm/s | `g1_agv_client.hpp`: `vz × 76.5` mm/s | **28% gap.** Source vs doc. Do not size cycle times until reconciled. |
| C9 | **G1-D total weight** | Marketing: ~90 kg (both configs) | Dev doc: ~50 kg / ~80 kg — **and self-contradicts** (chassis alone listed at 63 kg) | **Do not rely on either.** Require a per-configuration weight on any quote. |
| C10 | G1 `right kiss` action ID | C++: `{"right kiss", 12}` | Python: `"right kiss": 13` | **One is wrong.** C++ also maps `left kiss` to 12. Resolve empirically. |
| C11 | G1 arm-action JSON key | C++: `{"action_id":N}` | Python: `{"data":N}` | Same API ID 7106, different payload. **One is wrong.** |
| C12 | G1 `StandUp2Squat()` | Python sends fsm 706 | Python `Squat2StandUp()` **also** sends 706 | One is mislabelled. |
| C13 | **G1 emergency-stop RC combo** | Manual: `L1 + A` | Manual: `L2 + B` | **Verify on hardware before relying on it.** |
| C14 | Dex3-1 state topic | `g1_dex3_example.cpp`: `rt/lf/dex3/left/state` | File-scope initializer + prose docs: `rt/dex3/left/state` | In-repo bug. Use `rt/dex3/{side}/state`. |
| C15 | **G1 user manual contamination** | G1 manual §environment: 0–40 °C, not waterproof | **Same PDF** contains 5–35 °C, 25° slopes, 10 cm steps, 3 m distance, *"does not have a visual perception system"* | **The second passage is quadruped (Go2/B2) content in a G1 document.** Those figures circulate online as G1 specs. **They are not.** |
| C16 | Go2 SLAM version gate | `Motion_Services_Interface_V2.0`: ≥ 1.1.6, EDU only | `ObstaclesAvoidClient`, `LiDAR_service`: no version or model gate stated | Whether these also require ≥1.1.6 + EDU is **UNVERIFIED**. |
| C17 | Python SDK completeness | README: *"maintains consistency with unitree_sdk2"* | Source: `StandUp`, `Squat`, arm custom actions, `StopCustomAction` all **absent in Python** | **The README's parity claim is false.** |
| C18 | SDK2 README scope | No feature or robot list | 11 families with examples | Not a contradiction — but worth noting that **all real documentation is off-repo.** |
| C19 | `unitree_ros2` robot support | README: "Supports Go2, B2, and H1" | Tree covers Go2, B2W, G1, H1, H1-2, H2 | Stale README. |
| C20 | IsaacLab licence | `unitree_sim_isaaclab` LICENSE: Apache-2.0 **[L-fetched]** | Wraps **NVIDIA Omniverse**, not OSI-approved | ⚠️ **NVIDIA's licence text was NOT retrieved. UNVERIFIED.** Treat as a blocker for anything redistributed. |

---

## 6. NOT FOUND — searched for and not located

Absence of evidence, not evidence of absence. Each is carried as a risk.

| # | Not found | Searched |
|---|---|---|
| N1 | **Any published grasp success rate for the Unitree Dex3-1 hand** | arXiv, repositories, datasets |
| N2 | **Any published fall-rate statistic for either robot** | All sources |
| N3 | Any independent third-party payload or grasp test of the G1 arms | All sources |
| N4 | Any rigorous independent battery runtime test for either robot | All sources |
| N5 | **Any official object-detection or person-following demo for the Go2** | All 56 official repos, SDK doc tree |
| N6 | Go2 minimum turning radius; threshold/step-over heights; carpet-vs-tile guidance | Official docs, spec tables |
| N7 | Go2 payload centre-of-mass envelope, mounting-offset tolerance, quantified stability penalty | Official docs, spec tables |
| N8 | G1 walking vx/vy/vyaw absolute limits; stand height in metres; swing height in metres | SDK2, SDK docs |
| N9 | G1 battery charging time; IP rating; legged max incline | Official docs, manual |
| N10 | **G1 depth camera model** — in any official source | All official pages |
| N11 | Name of G1 FSM mimic motion `550507` | Docs — must be enumerated at runtime |
| N12 | Any statement by Unitree on Go2 leg-injury risk or human-proximity safety | Official docs |
| N13 | What Go2 X's `◕` "Basic support" concretely permits | Official docs — **ask in writing** |
| N14 | Whether the Go2's internal L2 is reachable by `unilidar_sdk2` | SDK README, official docs |
| N15 | G1-D HTTP REST API contract | Official docs — chassis ports marked "not currently open" |
| N16 | Definition of G1-D's "Physical Collision Sensor" | All official pages |
| N17 | Usable Go2 back-plate mounting dimensions (published as PNG only) | Official docs |
| N18 | Any published reliable-delivery success rate for a Go2 in an indoor school | All sources |
| N19 | Any evidence that the G1 is *more* deployable than the Go2 in real environments | Deliberately searched; **the available evidence points the other way** |
| N20 | NVIDIA Omniverse / Isaac Sim licence text | Not retrieved — see C20 |

---

## 7. Deliberately excluded sources

| Source | Reason |
|---|---|
| Robots Rated, RoboPulse, RoboTesters "reviews" | **Each explicitly states it did not test the hardware.** Recorded in §4 for that reason, not as review evidence. |
| Reseller listings (usrobotstore, robostore, roboticscenter, scan.co.uk, abrarobotics) | Used only for indicative G1-D street prices. Naming is inconsistent (Standard/Ultimate vs Standard/Flagship vs A–E). No vendor-published G1-D price exists. |
| General AI-robotics marketing articles | Do not meet the evidence bar for a technical claim. |
| Any source describing "AgiBot G1" as "Unitree G1" | **Different robot, different company.** See source 55. |

---

## 8. Claims with the weakest evidence

Stated plainly, because a research report that hides its weak points is not useful.

| Claim | Evidence strength | Why | How to resolve |
|---|---|---|---|
| **Go2 reliability 7/10** | 🟡 **Moderate** | Teardown endorsement + Asian Games deployment + industrial product line, against operator anecdotes of falls and spontaneous shutdowns. No controlled testing. | Run the Phase 7/8 endurance and 30-run tests. |
| **G1 reliability 2/10** | 🟡 **Moderate** | Five independent sources agree, which is strong. But no published fall rate exists. | Cannot be resolved by research. Only by observation. |
| **G1 manipulation 6/10** | 🔴 **Low-moderate** | Rests on results obtained with **different hands** (BrainCo, not Dex3-1) and a single force measurement. | Bench-test with the actual hand. |
| **Go2 navigation 8/10** | 🟡 **Moderate** | The API is documented and verified. But its real-world reliability in a school corridor is unproven, and the vendor path requires hardware we may not obtain. | Phase 4. |
| **All runtime behaviour** | 🔴 **Static analysis only** | No repository was cloned, built, or run. No robot was touched. | Phase 2 onward. |
| **Camera frame rate** | 🔴 **Contradicted** | 15 vs 30/60 fps across official sources. | Measure in Phase 2. |
| **Battery runtime** | 🔴 **Contradicted** | Three different official figures. | Measure in Phase 8. |
