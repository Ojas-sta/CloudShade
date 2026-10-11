# SHANA — Sources

All sources consulted for the Phase 0 feasibility study. **All accessed
2026-10-10** unless noted.

**Source types:** `OFFICIAL-MFR` (manufacturer documentation) · `OFFICIAL-GOV`
(government/regulator) · `OFFICIAL-CODE` (official source code) · `STD`
(standard) · `LEGAL` (judgment or statute) · `PEER-REVIEWED` ·
`INDEPENDENT` (third-party test, teardown, operator report) · `COMMUNITY`
(forum, blog) · `AGGREGATOR`

---

## Method and its limits

1. **`support.unitree.com` is WAF-blocked to direct fetch** — HTTP 567 bot
   challenge from `TencentEdgeOne`. Content retrieved through a text-render proxy.
   **URLs cited are the origin URLs; content is the origin's rendered page.**
2. **`api.github.com` rate-limits unauthenticated requests** (60/hour). Directory
   listings were obtained via authenticated `gh` CLI and `codeload` tarballs; file
   contents from `raw.githubusercontent.com`.
3. **No robot was operated, connected to, or commanded. No packages installed. No
   network probed. No hardware configured.** All findings are documentation
   review and static source analysis.
4. **Contradictions are recorded in §6, not silently resolved.**
5. **NOT FOUND means searched and not located.** It is not evidence of absence.

---

## 1. Unitree — manufacturer documentation

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 1 | Unitree Go2 product page | `https://www.unitree.com/go2/` | OFFICIAL-MFR | Payload ≈7/8/8/8 kg by variant; speed 2.5/3.5/3.7/3.7 m/s; max climb 15/16/16/16 cm; slope 30°/40°; **secondary development ○/○/◕/●**; depth camera EDU-only; foot force sensors EDU-only; battery 8000×3 / **15000** mAh; charger 3.5 A ×3 / **9 A**; warranty 6/12/12/12 months; prices $1600/$2800/$4500/contact-sales | Marketing. Camera fps and battery runtime contradicted elsewhere (§6 C2, C3) |
| 2 | Go2 developer — About Go2 | `https://support.unitree.com/home/en/developer/about%20Go2` | OFFICIAL-MFR | Dimensions 70×31×40 cm / crouch 76×31×20 cm; ~15 kg; 45 N·m; **max step 16 cm**; slope ±40°; **4D LiDAR L2, 360°×96°, 21,600 samples/s, 0.05 m min, 905 nm**; camera F2.2, 120° FOV, **15 fps**; **"Low battery warning, will automatically crouch down within 10 minutes"**; battery table BT2-05 8000 mAh/236.8 Wh/2 kg/3.5 A/2 h and BT2-06 **15000 mAh/432 Wh/2.5 kg/9 A/1 h 15**; charging pile X+EDU only; EDU 5–35 °C | Proxied. Fps conflicts with source 6 |
| 3 | **Developer FAQ** | `https://support.unitree.com/home/en/developer/faq` | OFFICIAL-MFR | **THE GATING FACT, verbatim:** *"Only EDU or above versions can be developed, other versions such as PRO and AIR versions do not open the secondary development interface."* **Ubuntu 20.04 required**; Mac/Windows not supported; **"Go2 only supports wired connection for development"**; camera **1080p 30 fps / 720p 60 fps** | Proxied. Fps conflicts with source 2 |
| 4 | Quick Start | `https://support.unitree.com/home/en/developer/Quick_start` | OFFICIAL-MFR | Robot `192.168.123.161`; host `192.168.123.222`; *"It is not allowed to set this adapter's IP address to 192.168.123.161"*; NIC passed as argv | Proxied |
| 5 | LiDAR Service | `https://support.unitree.com/home/en/developer/LiDAR%20Service` | OFFICIAL-MFR | DDS topics `rt/utlidar/cloud`, `cloud_deskewed`, `height_map_array`, `range_info`, `switch`; type `PointCloud2_`; **"If the operation control is turned off… the height map data cannot be obtained"** | Proxied |
| 6 | SLAM and Navigation Service | `https://support.unitree.com/home/en/developer/SLAM%20and%20Navigation_service` | OFFICIAL-MFR | **"only supports the EDU robot dogs with expansion dock and the official Unitree-purchased lidar (MID-360/XT16)"**; `slam_operate` v1.0.0.1; APIs 1801/1802/1804/1102/1201/1202/1901; 0.2–1.0 m/s; ≥0.8 m width; ≤0.5 m obstacles; ≥20 cm; ≤10 m; YSN required; Mid-360 extrinsics `(0.1870, 0, 0.0803)`, 13° Y rotation | Proxied. Patch log shows changes as recent as 2026-04 |
| 7 | Obstacles Avoid Client | `https://support.unitree.com/home/en/developer/ObstaclesAvoidClient` | OFFICIAL-MFR | `obstacles_avoid` v1.0.0.2; `Move(x,y,yaw)` x ±1.5, y ±1.0, yaw ±1.57; `MoveToIncrementPosition` ±2.0 m; `MoveToAbsolutePosition` world frame; `UseRemoteCommandFromApi` | Proxied |
| 8 | **Motion Services Interface V2.0** | `https://support.unitree.com/home/en/developer/Motion_Services_Interface_V2.0` | OFFICIAL-MFR | **33 documented functions, `SwitchGait` ABSENT**; *"These interfaces apply to Go2 Edu models with software version ≥ V1.1.6"*; **`ClassicWalk`** *"supports climbing stairs, gravel, meadows, and slippery surfaces"*; `EconomicGait` *"up to 4 hours"*; `TrotRun` "somewhat risky"; `StaticWalk` "does not have the ability to handle complex terrain"; `WalkUpright`/`CrossStep` overheating caution; **`Damp()`** *"highest priority and is used for emergency stops"*; **`RecoveryStand()`** *"Regardless of whether it is fallen or not, it will recover to standing"* | Proxied. **The absence of `SwitchGait` is the finding that matters** |
| 9 | High motion control / sports_services | `https://support.unitree.com/home/en/developer/High_motion_control`, `.../sports_services` | OFFICIAL-MFR | `gaitType()` enum **0 idle, 1 trot, 2 run, 3 climb stair, 4 forwardDownStair, 9 adjust**; **`SwitchGait(int d)`** for firmware **< V1.1.6**, verbatim: *"3 is forward climbing mode, and 4 is reverse climbing mode"* | Proxied. **The only documented stair-specific API, and it is version-gated** |
| 10 | AI motion service | `https://support.unitree.com/home/en/developer/AI_motion_service` | OFFICIAL-MFR | `FreeAvoid(2048)`: *"when stationary, it can dodge objects in front"* | Proxied. **The uncommanded-dodge hazard** |
| 11 | Payload | `https://support.unitree.com/home/en/developer/Payload` | OFFICIAL-MFR | *"The Go2 Pro and Go2 Air versions do not support any of the extended electrical interfaces… These interfaces are exclusive to the Go2 EDU version"*; M3×50 dock mounting; `AutoRecoverSet` payload warning | Proxied. Back-load dimensions are PNG-only — UNVERIFIED as text |
| 12 | Robot State Client | `https://support.unitree.com/home/en/developer/RobotStateClient` | OFFICIAL-MFR | **The full service list — used for the return-to-dock negative:** `sport_mode`/`mcf`, `basic_service`, `chat_go`, `net_switcher`, `obstacles_avoidance`, `ota_box`, `unitree_lidar`, `utrack`, `video_hub`, `webrtc_bridge`, `vui_service`. **No charging or dock service.** | Proxied |
| 13 | Application Development index | `https://support.unitree.com/home/en/developer/Motion_Services_Interface_V2.0` (nav tree) | OFFICIAL-MFR | **An "Aruco Recharge Service Interface" page EXISTS** in the documentation navigation tree | ⚠️ **Body would not render through 8 URL variants. NOT READ.** See `questions.md` TEC-09 |
| 14 | Architecture Description / Multimedia | `https://support.unitree.com/home/en/developer/Architecture%20Description`, `.../Multimedia_Services` | OFFICIAL-MFR | GStreamer UDP H.264 pipeline for video; *"the EDU version can call the interface through DDS or ROS2"* | Proxied |

---

## 2. Unitree — source code

| # | Repository / file | URL | Type | Claims supported |
|---|---|---|---|---|
| 15 | `unitree_sdk2` — Go2 SportClient | `https://raw.githubusercontent.com/unitreerobotics/unitree_sdk2/main/include/unitree/robot/go2/sport/sport_client.hpp` | OFFICIAL-CODE | 39 methods enumerated. **No `SwitchGait`, no stair method, no charging method, no dock method** |
| 16 | `SportModeState.msg` | `https://raw.githubusercontent.com/unitreerobotics/unitree_ros2/master/cyclonedds_ws/src/unitree/unitree_go/msg/SportModeState.msg` | OFFICIAL-CODE | **Full field list — contains NO battery fields.** `gait_type` is field index 5 of `SportModeState_` |
| 17 | `BmsState.msg` | `.../unitree_go/msg/BmsState.msg` | OFFICIAL-CODE | `soc` (uint8 %), `current` (int32 signed mA), `cell_vol[15]`, `bq_ntc`, `mcu_ntc`, `status`, `cycle`. **The battery source** |
| 18 | `LowState.msg` | `.../unitree_go/msg/LowState.msg` | OFFICIAL-CODE | `power_v`, `power_a`, `temperature_ntc1/2`, `bms_state` |
| 19 | `IMUState.msg` | `.../unitree_go/msg/IMUState.msg` | OFFICIAL-CODE | **Exactly five fields: `quaternion[4]`, `gyroscope[3]`, `accelerometer[3]`, `rpy[3]`, `temperature`. No altitude. No pressure.** The decisive negative |
| 20 | `HeightMap.msg` | `.../unitree_go/msg/HeightMap.msg` | OFFICIAL-CODE | **`origin` is `float32[2]` — XY only, no Z datum.** A local 2D terrain grid, not an absolute reference |
| 21 | `SportModeState_.hpp` | `https://raw.githubusercontent.com/unitreerobotics/unitree_sdk2/main/include/unitree/idl/go2/SportModeState_.hpp` | OFFICIAL-CODE | `gait_type` at index 5 |
| 22 | B2-W SportClient | `https://raw.githubusercontent.com/unitreerobotics/unitree_sdk2/main/include/unitree/robot/b2w/sport/sport_client.hpp` | OFFICIAL-CODE | **`SwitchGait(int d)` still exists here** — proving removal from Go2 is deliberate, not a header omission |
| 23 | `unitree_ros2` package.xml | `https://raw.githubusercontent.com/unitreerobotics/unitree_ros2/master/cyclonedds_ws/src/unitree/unitree_go/package.xml` | OFFICIAL-CODE | **Dependencies: `unitree_go`, `unitree_hg`, `unitree_api`, `rclcpp`, `std_msgs`, `rosbag2_cpp`. NO nav2, slam_toolbox, amcl, tf2, nav_msgs, or ros2_control.** The official ROS 2 stack is a DDS bridge, not a navigation system |
| 24 | `unitree_mujoco` | `https://github.com/unitreerobotics/unitree_mujoco` | OFFICIAL-CODE (BSD-3) | *"based on Unitree sdk2 and mujoco… enabling a seamless transition from simulation to physical development"*; models include **go2**; **CPU physics**. ⚠️ Note the readme is `readme.md`, lowercase — `README.md` 404s |
| 25 | `unitree_slam` | `https://github.com/unitreerobotics/unitree_slam` | OFFICIAL-CODE | **No LICENSE file** (796-file tree verified). CMake targets `demo_h1 demo_b2 demo_xt16 demo_mid360` — **H1/B2 only, NOT Go2**. ⛔ **EXCLUDED — no legal grant of any right** |
| 26 | `point_lio_unilidar` | `https://github.com/unitreerobotics/point_lio_unilidar` | OFFICIAL-CODE (GPL-2.0) | ROS 1 Noetic (catkin). ⛔ **EXCLUDED — GPL-2.0 strong copyleft + ROS 1** |

---

## 3. Unitree — user manual

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 27 | **Go2 User Manual V1.0** | `https://physical-computing-lab.net/wp-content/uploads/Go2-User-Manual.pdf` | OFFICIAL-MFR (third-party mirror) | **Battery insert procedure** — side insertion, *"click"* latch, robot must lie on a flat surface; **§5** *"not currently recommended that robot walk stairs higher than 16cm… likely to trip over the foot"*; **§9** *"it is forbidden to use the robot on the occasions where the terrain is complicated… steps higher than 16cm"*; **pinch hazard** *"Hand may be pinched at the motion joints"*; **2 m safe distance**; **"not a toy and is not intended for use by persons under the age of 14"**; **stair climbing modes 1 and 2** on the remote; endurance **1–2 h** unloaded alternating standing/walking; power reducers incl. *"running with a load"* and *"walking at a lower body height"* | ⚠️ **Third-party mirror of an official document.** Contains a **quadruped-vs-other-product contamination risk** analogous to the G1 manual issue found in the prior phase — the 14-year age limit differs from the G1 manual's 18, consistent with a different document |

---

## 4. Elevator — manufacturer and legal

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 28 | Johnson Lifts — official site | `https://www.johnsonliftsltd.com/` | OFFICIAL-MFR | Company identity, Chennai, est. 1963, 80+ branches, 205+ service centres; **"Johnson Lifts compliant with IS17900"**; product lines Nextra, Scenic, Sukranti, Eazy Ride Plus CS6 | — |
| 29 | About Johnson Lifts | `https://www.johnsonliftsltd.com/about-johnson-lifts/` | OFFICIAL-MFR | *"Established in 1963 and headquartered at Chennai"*; 2 lift factories Chennai + 1 Nagpur | — |
| 30 | **EyeRIS contactless call booking** | `https://www.johnsonliftsltd.com/eyeris/`, `.../eyeris-contactless-call-booking-system/`, `.../eyeris-launch/` | OFFICIAL-MFR | **THE KEY FINDING.** Verbatim: *"connects the smartphone to the lift system using QR code technology and the user operates the lift by just tapping the buttons on the phone's screen"*; *"Just place the mobile in front of the QR code available in the Landing Operating Panels… select 'present floor' and then press 'destination floor'"* | ⚠️ **A 2020 COVID-era product.** Retrofit availability, QR active-vs-passive, and non-human provisioning are **NOT FOUND** — see §5 |
| 31 | WATCH IoT | `https://www.johnsonliftsltd.com/watch-iot/`; `https://timesofindia.indiatimes.com/johnson-lifts-launches-an-iot-based-technological-feature-in-its-lifts-named-watch/articleshow/94354506.cms`; `https://elevatorworld.com/article/advanced-intelligent-software-introduced/` | OFFICIAL-MFR + news | **Monitoring only.** Lift → Johnson data centre → technicians. **No inbound third-party command path found** | Negative finding, high confidence |
| 32 | Nextra product page | `https://www.johnsonliftsltd.com/nextra/` | OFFICIAL-MFR | VVVF; *"Automatic Rescue Device – ARD"*; *"BMS/RMS Compatible"*; 2 m/s; up to 1768 kg | ⚠️ **"BMS/RMS Compatible" is a building-management compatibility claim, NOT an open control API** |
| 33 | Toshiba Johnson Elevators (India) | `https://toshibaelevators.in/destination-control-system-floornavi.aspx` | OFFICIAL-MFR | Separate JV entity (Toshiba 51% / Johnson 49%); DCS named **FLOORNAVI** | ⚠️ **Corrects a common error: "SIRIO" is not a Johnson product** — it is an Extrema (Italy) hydraulic home lift |
| 34 | **Maharashtra Lifts, Escalators and Moving Walks Act, 2017, s.10** | `https://maharashtrahousingandbuildinglaws.com/the-maharashtra-lifts-escalators-and-moving-walks-act-2017/` | LEGAL | **Verbatim:** *"No additions or alterations… shall be made to any lift… installation except with the prior permission in writing of an Electrical Inspector (Lifts)."* Penalties to ₹50,000 | State-specific. ⚠️ **The school's State Act must be identified — the Tamil Nadu, Karnataka and Haryana equivalents are cited in `appendix_india_rf_research.md`'s companion research and cited in the risk register** |
| 35 | Delhi / Bombay Lift Rules, Rule 6 | `https://sarthac.gov.in/view-act-rule-file?file_id=3116` | LEGAL | *"The owner shall not carry out any additions or alternations to an installed Lift"*; *"No person shall willfully interfere with any mechanism"* | Cited as binding by the Supreme Court in 2026 |
| 36 | Tamil Nadu Lifts Act 1997, s.13/15 | `http://indiankanoon.org/doc/119476286/` | LEGAL | *"the lift installation shall not be interfered in any way"*; penalties | Relevant if the school is in Tamil Nadu — ⚠️ **UNKNOWN** |
| 37 | **Otis Elevator Co. (India) v. Rashmi Handa** | `https://api.sci.gov.in/supremecourt/2014/6284/6284_2014_5_1501_72672_Judgement_29-Jul-2026.pdf`; report `https://www.advocatekhoj.com/library/judgments/index.php?go=2026%2Fjuly%2Fm-s-otis-elevator-company-india-ltd-vs-rashmi-handa-94` | LEGAL | **Supreme Court of India, 29 July 2026.** Verbatim: *"A party that undertakes the comprehensive maintenance of a machine, which is in the nature of a vehicle, owes a heightened duty of care towards its users."* Tampering with the Brake Release Key in violation of Bombay Lift Rules Rule 6(xiii). Liability 70/25/5 — **the occupier was held 5% liable for failure of oversight** | ⚠️ **The most consequential single source in this study.** Establishes that unauthorised third-party intervention does not exonerate, and that the school inherits exposure |
| 38 | IS 17900 series (BIS) | `https://www.services.bis.gov.in/php/BIS_2.0/bisconnect/knowyourstandards/Indian_standards/isdetails_mnd/28090`; `https://www.bis.gov.in/wp-content/uploads/2026/02/SOM_January_2026.pdf` | STD | **The governing Indian lift standard family (2022 series).** Part 1:2022 safety rules (= ISO 8100-1:2019 modified); Part 4:2025 control devices, buttons, signals; Part 6:2022 maintenance; Part 11:2025 safety norms for existing lifts. January 2026 BIS bulletin: **900 mm minimum door clear opening**, **±10 mm levelling accuracy** | ⚠️ **Full text is paywalled — the index and bulletin were read, not the standard.** ⚠️ **Corrects an error: IS 14447 is foundry refractories and IS 6914 is structural steel billets (withdrawn). Neither is a lift standard** |
| 39 | Otis Integrated Dispatch | `https://www.otis.com/en/us/products-services/products/otis-integrated-dispatch`; datasheet `https://www.otis.com/documents/256045/257664901/OID_Robot-Prodivers_Datasheet_InDesign_WHQ_English_Final.pdf` | OFFICIAL-MFR | *"cloud-based or on-premise API solution for integrating service robots with Otis elevators"*; Place Hall Call / Car Call / Destination Call; *"Our field engineers and mechanics will install a small amount of hardware in the building and coordinate with local safety inspectors to ensure compliance"* | Vendor-locked. **Not applicable to a Johnson lift** |
| 40 | TK MAX Robot Calls API | `https://www.tkelevator.com/global-en/products/digital-solutions/api/` | OFFICIAL-MFR | *"Enable autonomous robot elevator usage"*; `developer.ea.tkelevator.com` | Vendor-locked |
| 41 | Schindler BuilT-In Robot Service Connection | `https://www.jardineschindler.com/en/elevators/built-in/robot-service-connection.html` | OFFICIAL-MFR | Manufacturer-offered robot service connection | Vendor-locked |
| 42 | The Robot Report — Otis | `https://www.therobotreport.com/otis-shares-secrets-controlling-elevators-robots/` | INDEPENDENT | *"Our technology, obviously, only works with Otis equipment"*; Otis encourages robots to *react* to fire-service mode, never to invoke it | — |

---

## 5. Elevator — precedent and practice

| # | Title | URL | Type | Claims supported |
|---|---|---|---|---|
| 43 | Relay Robotics elevator integration | `https://relayrobotics.com/`; `https://hotelguru.com/articles/elevator-integration-for-hotel-delivery-robots/`; `https://www.therobotreport.com/relay2-delivery-robot-offers-2x-payload-new-elevator-integration/` | INDEPENDENT | **Service company + OEM involvement, written proposal, no lift modification.** *"This process can take up to 2 months."* Johnson **not** in the supported list. **Relay2 explicitly eliminated the mechanical button interface** in favour of cloud-based integration. **The clearest precedent, and it points away from actuators** |
| 44 | Starship Technologies | `https://www.starship.xyz/faq/`; `https://medium.com/starshiptechnologies/robots-in-the-wild-4ba48aeec1ee` | INDEPENDENT | **Sidewalk-only. Does not use passenger elevators.** *"if you could, please press the button"*; robots *"wait away from the push button… so they're never in the way"* |
| 45 | Bitriá et al., *Sensors* 23(13):6089 | `https://www.mdpi.com/1424-8220/23/13/6089` | PEER-REVIEWED | Academic non-invasive approach — *"attached over the original internal elevator button panel using servomotos to press the original buttons"*. Verbatim: *"most IoT devices used to monitor and remotely control elevators require the intrusive manipulation of both their wiring and mechanical structure"* | ⚠️ **Even this "non-invasive" academic technique is a prohibited addition to the installation under Maharashtra s.10 / Delhi Rule 6(v)**. Academics get away with it by owning the building; a school does not have that position |
| 46 | RobInCare, *Frontiers in Robotics and AI* 2025 | `https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2025.1546894/full` | PEER-REVIEWED | Kinova arm presses buttons. *"41 out of 44 elevator rides"*; *"the door only stays open for 6 s"*; *"task completion still fails in 14% of cases"*; *"ultimately there will always be situations that cannot be anticipated and require human intervention"* | Wheeled platform, not a quadruped |
| 47 | McAree et al. (Sheffield) | `https://exa.ai/library/publication/bz73qw2gzqj` | PEER-REVIEWED | *"it is necessary for the robot to leave the lift before it can detect differences in the layout of the various floors, **as the areas immediately adjacent to the lift are identical**"*; robot *"will need to request assistance from other lift occupants"* | **Directly corroborates the wrong-floor risk** |
| 48 | Firefighter's service references | `https://omfpoa.com/wp-content/uploads/2023/06/FF-Elevators.pdf`; `https://standards.globalspec.com/std/14346907/din-en-81-72`; `https://www.iso.org/standard/74750.html`; `https://www.iso.org/standard/73072.html` | STD | ASME A17.1 Phase I/II; EN 81-72:2020; ISO/TR 8101-10:2024; ISO 8100-20:2018 | ⚠️ **Whether these specific lifts have Phase I recall is NOT FOUND** — depends on the school's fire strategy and State rules. See ELV-09 |
| 49 | NBC 2016 Part 8 §5A | `https://www.bis.gov.in/wp-content/uploads/2022/08/Booklet-Guide-for-Using-NBC-2016.pdf` | STD | Lift planning/installation per Lift Acts and Rules; typical school-lift car ≥1100 × 1400 mm with 900 mm door for disabled access | ⚠️ **The 1100 × 1400 figure is a disabled-access minimum, not a typical school lift.** Actual car dimensions are **NOT FOUND** — measure on site (ELV-02) |

---

## 6. Indian radio regulation

> Full detail, with gazette citations and computed link budgets, is in
> **`appendix_india_rf_research.md`**. Summarised here because it is a gating
> input for any radio-based feature.

| # | Title | URL | Type | Claims supported |
|---|---|---|---|---|
| 50 | **NFAP-2025** (National Frequency Allocation Plan, in force 30 Dec 2025) | `https://www.dot.gov.in/static/uploads/2026/02/b110cdc386d3a4e41c8483d7ffd7c410.pdf` | OFFICIAL-GOV | India's band allocation; **865–868 MHz ISM**; Annexure 1 exemption table |
| 51 | **G.S.R. 853(E), 10 Dec 2021** — SRD rules | via `eservices.dot.gov.in` and NFAP-2025 | OFFICIAL-GOV | Verbatim duty cycles: **2.5 % end device, 10 % network access point, 1 % at 25 mW**; **e.r.p.** defined relative to a half-wave dipole; 500 mW e.r.p. = **29.15 dBm EIRP**; cites **EN 300 220**, not EN 303 708 |
| 52 | **Telecommunications (Radio Equipment Possession Authorisation) Rules, 2026**, G.S.R. 592(E), 8 Jul 2026 | `https://eservices.dot.gov.in/sites/default/files/circular-notifications/telecommunications-radio-equipment-possession-authorisation-rules-2026.pdf` | OFFICIAL-GOV | **The regime changed two months before this study.** Verbatim rule 10 exemptions (a)/(c)/(g)/(h). Applications opened 06.08.2026 |
| 53 | DoT — WPC & SACFA | `https://www.dot.gov.in/spectrum-management/2457?page=1` | OFFICIAL-GOV | **WPC Wing is the National Radio Regulatory Authority.** Verbatim: *"created in 1952… issues licenses to establish, maintain and operate wireless stations"* |
| 54 | DoT Telecom eServices / Acts & Rules | `https://eservices.dot.gov.in/act-and-rules`; `https://eservices.dot.gov.in/` | OFFICIAL-GOV | The single most useful official index. **The WPC Helpdesk for TEC-10 clarification** |
| 55 | ETA — self-declaration since 09.09.2024 | via `eservices.dot.gov.in` | OFFICIAL-GOV | **ETA is now 100 % self-declaration, ₹10,000, lifetime validity** |
| 56 | ⚠️ **Correction** — the `wpc.gov.in` domain | — | — | **Several blogs cite `wpc.gov.in`. That domain is NXDOMAIN — it does not exist.** The correct host is `wpc.dot.gov.in` |

---

## 7. LoRa hardware and radio practice

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 57 | Heltec WiFi LoRa 32 V3 / V2 datasheets | Heltec / vendor documentation | OFFICIAL-MFR | V3 HF spans **863–928 MHz**; datasheet lists "IN868" as **865–867 MHz**. Measured current: LoRa TX 10 dBm = 50 mA, 20 dBm = 130 mA; **Wi-Fi AP mode = 135 mA** | ⚠️ **Conflict between the board's 863–928 span and its own IN868 = 865–867, which is narrower than the gazette's 865–868.** Verify on the bench |
| 58 | LoRaWAN airtime computation | standard LoRa datasheets / SF tables | PEER-REVIEWED-adjacent | 350 B @ SF9/BW125 ≈ 1.2 s airtime → 1 report/min ≈ 2 % duty cycle. **350 B @ SF12 ≈ 16 s** → one report every ~2 min max | ⚠️ **The 2.5 % cap is breached at SF12 with acknowledged downlink** |
| 59 | WiFi and LoRa energy comparison | `https://hal.science/hal-04083109v1/document` | PEER-REVIEWED | *"the power consumption for both WiFi and LoRa transmissions is practically the same"* | ⚠️ **Used SX1278 — a 433/470 MHz part that cannot legally operate at 865–868 MHz in India.** Directional only |
| 60 | ESP32 ultra-long battery life with ESP-NOW | `https://thingpulse.com/esp32-ultra-long-battery-life-with-espnow/` | INDEPENDENT | Wi-Fi association **3.877 s** vs ESP-NOW **564 ms**; 6.9 months → 3.3 years. *"WiFi and ESP-NOW cannot run the same time"*; 10 encrypted peers, 250 B/frame | Blog. ⚠️ **Irrelevant on a docked, mains-powered robot** — which is further evidence LoRa's value here is failure independence, not battery life |
| 61 | Multi-floor LoRa propagation studies | See `appendix_india_rf_research.md` §B.11–B.12 | PEER-REVIEWED | **5–6 storey reinforced-concrete studies; measured floor attenuation 5.5–21.7 dB per floor — a 4× spread.** A 16-storey study found LoRa *still lost the link across floors* with near-zero PDR in the far zone | ⚠️ **No measurement in a school, and none in India. NOT FOUND.** This is why no single range figure is quoted |
| 62 | Indoor floor-ambiguity and beacon hints | `https://www.indoor-mapping-wayfinding.com/indoor-positioning-fingerprinting/floor-level-detection/resolving-floor-ambiguity-with-beacon-hints/`; `https://exa.ai/library/publication/0lqnfd4fpf7` | PEER-REVIEWED | *"Buildings with repeating floor plates produce fingerprints that a radio estimator cannot separate… The fix is not a better estimator — it is one piece of evidence that only exists on one floor."* BLE beacon floor detection: 0.9877–1.0 accuracy, ~5 s delay. Design: tx_power −12 to −4 dBm, mount away from stair voids, `floor_rssi` ≈ −85 dBm | **Corroborates R-03. Beacon ID is valid; signal strength alone is not** |
| 63 | ArUco multi-marker localisation | `https://arxiv.org/html/2509.17345v3` | PEER-REVIEWED | *"if more than one marker (>4 pairs) is used… With 5+ markers, reliable localization… is possible"* | Ceiling-mounted monocular camera case |

---

## 8. Navigation framework sources

| # | Title | URL | Type | Claims supported |
|---|---|---|---|---|
| 64 | Nav2 Map Server | `https://docs.nav2.org/configuration/packages/configuring-map-server.html` | OFFICIAL-CODE | One map at a time on `/map`; `nav2_msgs/srv/LoadMap` for runtime swap; AMCL `first_map_only: false` for multi-map |
| 65 | Multi-floor map switching implementation | `https://github.com/shimz-robotics/mapoi` — `docs/architecture.md`, `mapoi_nav2_bridge_map_switch.cpp` | COMMUNITY | A real per-floor implementation: `SelectMap` → `LoadMap` → re-seed `/initialpose`, with `map_switch_succeeded`/`map_switch_failed` status and **cancelling the current goal on switch**. Treats failed re-localisation as terminal |
| 66 | SLAM Toolbox ROS 2 README | `https://raw.githubusercontent.com/SteveMacenski/slam_toolbox/ros2/README.md` | OFFICIAL-CODE | `mapping`, `localization`, lifelong modes; pose-graph serialization; KD-Tree reinitialisation; "Start By Dock"; merging multiple pose-graphs. ⚠️ **The word "multi-map" is not used** |
| 67 | Go2 stair-climbing community reports | `https://www.youtube.com/watch?v=lnXTzD2sj-U` (DroneBlocks, 2024-10); `https://www.youtube.com/watch?v=613JCha77GY` (Miraenda, 2024-03); `https://forum.mybotshop.de/t/unitree-go2-no-more-stair-climbing-mode-after-update/1459` (2025-09); `https://github.com/NMadhub/go2-blind-stairs-newton` | INDEPENDENT / COMMUNITY | Ascent vs descent asymmetry; **8-inch (203 mm) steps = 27 % above spec, visible struggle**; *"stair climbing is a perception problem"*; post-update remote removed stair mode |
| 68 | Quadruped multi-floor navigation | `https://www.reddit.com/r/unitree/comments/1v62qjv/` (2026-07) | COMMUNITY | *"i find its unable the stairs in most public multi-floor datasets"* | ⚠️ **Verdict: NOT FOUND — no dedicated body of work on quadruped multi-floor navigation.** Applying wheeled-robot patterns to a legged platform is our own engineering |

---

## 9. Documented contradictions

Reported, not resolved.

| # | Item | Source A | Source B | Handling |
|---|---|---|---|---|
| C1 | **Go2 camera frame rate** | `about_Go2`: **15 fps** | Developer FAQ: **1080p 30 / 720p 60** | **Must be measured (TEC-02).** Design assumes the worse figure |
| C2 | **Go2 battery runtime** | Product matrix: **2–4 h**; battery table BT2-06: **3–5 h**; `EconomicGait`: **up to 4 h** | **User Manual: 1–2 h** | **Manufacturer figures span 1–5 h.** Plan on ~90 min and measure (TEC-05) |
| C3 | **Stair API availability** | `sports_services` documents `SwitchGait` with stair modes | **Absent from Motion Services Interface V2.0 and from Go2 `SportClient`** (present in B2-W) | Removal is deliberate. On current firmware, stairs are an **emergent property of `ClassicWalk`** |
| C4 | **Heltec band** | Board spans 863–928 MHz | Datasheet "IN868" = 865–867 MHz | Gazette says 865–868. **Verify on the bench** |
| C5 | **LoRa airtime vs duty cycle** | 2.5 % end-device cap | SF12 350 B ≈ 16 s per report | At SF12, one report every ~2 min **max**. Use SF9 |
| C6 | **Go2 X remote controller** | Spec table: manual controller ● | SBUS is EDU-exclusive | Likely a proprietary RC. Not material to SHANA |
| C7 | **Go2 manual age limit** | Go2 manual: **under 14** | G1 manual (prior phase): **under 18** | Different products, different documents. **Neither is a substitute for school policy** |
| C8 | **Lift standard number** | Common secondary sources cite IS 14447 / IS 6914 | **IS 14447 = foundry refractories. IS 6914 = structural steel billets, withdrawn. Correct family is IS 17900** | **Corrected.** Confirm with the service provider (ELV-03) |
| C9 | **Lift DCS product name** | "SIRIO" commonly attributed to Johnson | **SIRIO is an Extrema (Italy) hydraulic home lift.** Toshiba JV's DCS is FLOORNAVI | **Corrected** |
| C10 | **G1 manual contamination** *(from the prior phase)* | G1 manual contains a quadruped section (5–35 °C, 25° slopes, 3 m) | That is Go2/B2 content | Recorded in the prior phase. **Same class of error is possible in the Go2 manual — hence source 27's warning** |

---

## 10. NOT FOUND — searched for and not located

| # | Not found | Searched |
|---|---|---|
| N1 | **Johnson public API, developer portal, SDK, or robot-integration product** | Johnson site, product pages, documentation |
| N2 | **Whether EyeRIS is retrofittable to lifts installed earlier** | Johnson site, product pages |
| N3 | **Whether the EyeRIS QR on the LOP is passive or active** | Johnson site. ⚠️ **Determines whether fitting it is a modification** |
| N4 | **Whether EyeRIS can be provisioned to a non-human/robot client** | Johnson site |
| N5 | **Model, controller, or rated load of the school's specific lifts** | Johnson, Toshiba Johnson. Only on-site inspection resolves it |
| N6 | **Actual car internal dimensions, door clear opening, threshold sill gap** | NBC, BIS. **Measure with a tape** (ELV-02) |
| N7 | **Whether the school's lifts have Phase I fire recall fitted** | Depends on fire strategy and State rules |
| N8 | **Any published quadruped stability data inside a moving elevator car** | Academic search. All located work uses wheeled platforms |
| N9 | **Whether any school in India requires possession authorisation under the 2026 Rules** | DoT, legal commentary. ⚠️ **TEC-10 — ask in writing** |
| N10 | **Any published measurement of Go2 stair performance in a school** | All sources. **Nobody has tested the case we care about** |
| N11 | **Any published Go2 load-retention data** | All sources |
| N12 | **Any published Go2 range-under-payload figure** | All sources |
| N13 | **The full text of IS 17900** | BIS — paywalled. Index and bulletin read |
| N14 | **The "Aruco Recharge Service Interface" documentation body** | 8 URL variants. **TEC-09** |
| N15 | **Any published battery endurance test for the Go2 EDU** | All sources. **Measure it ourselves (TEC-05)** |
| N16 | **Nuro and OTTO Motors elevator behaviour** | Deliberately not researched. Not decision-relevant |
| N17 | **The school's State lift Act** | ⚠️ **UNKNOWN.** Maharashtra, TN, Karnataka and Haryana equivalents are all cited |
| N18 | **A room in this school with no Wi-Fi coverage** | Cannot be determined without a site survey (NET-05) |

---

## 11. Claims with the weakest evidence

| Claim | Strength | Why | How to resolve |
|---|---|---|---|
| Stair descent reliability | 🔴 **Weak** | Three independent third-party reports, none controlled, none with payload | TEC-06 supervised test |
| Actual riser height | 🔴 **None** | No measurement exists | FAC-01 tape measure |
| EyeRIS suitability | 🔴 **None** | Product exists; retrofit, QR type, and robot provisioning all unknown | ELV-06/07 in writing |
| Multi-floor behaviour | 🔴 **None** | No quadruped precedent exists at all | TEC-04 + TEC-08 + design |
| Camera frame rate | 🔴 **Contradicted** | 15 vs 30/60 fps across official sources | TEC-02 measurement |
| Battery endurance | 🔴 **Contradicted** | 1–5 h across four official documents | TEC-05 measurement |
| Practical payload ~3 kg | 🟡 **Moderate** | One community vendor's field report, not a controlled study | TEC-05 + payload testing |
| LoRa indoor coverage | 🟡 **Moderate** | Peer-reviewed studies exist, but floor attenuation varies 4× and none are in an Indian school | NET-05 site survey + survey per `appendix_india_rf_research.md` §B.12 |
| Go2 stair capability in firmware | 🟡 **Moderate** | Consumer remote has two modes; SDK ≥1.1.6 exposes none | TEC-07 |

---

## 12. Companion documents

| Document | Contents |
|---|---|
| `appendix_india_rf_research.md` | Full India RF regulation research — gazette citations, duty-cycle computations, ESP32 board comparison, multi-floor LoRa link-budget study, site-survey checklist with pass criteria |
| `../research/` (repository root) | The prior platform-selection study: G1 vs Go2 comparison, SDK research, safety analysis, implementation roadmap, sources |