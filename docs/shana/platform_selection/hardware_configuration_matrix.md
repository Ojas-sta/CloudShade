# Hardware Configuration Matrix

Every variant that can be substantiated from primary sources. **Configuration-
dependent features are flagged in bold.** Anything we could not resolve is marked
UNVERIFIED rather than guessed.

All data collected 2026-10-10.

---

## 0. Read this first: the model number decides whether the project is possible

Before any other specification matters:

| Platform | Model | Can you write custom software for it? |
|---|---|---|
| G1 | **Standard** ($13,500) | ❌ **NO** — `Secondary Development: /` |
| G1 | **EDU** (quote only) | ✅ YES |
| Go2 | **AIR** ($1,600) | ❌ **NO** — secondary development `○`; no RJ45, no DC-out, no SBUS |
| Go2 | **PRO** ($2,800) | ❌ **NO** — same |
| Go2 | **X** ($4,500) | ⚠️ `◕` "basic support" — **undefined by Unitree; confirm in writing** |
| Go2 | **EDU** (quote only) | ✅ YES — full SDK + ROS 2 + RJ45 + 28.8 V DC-out + SBUS |

**Verify the G1 table structure.** The G1 spec table at `unitree.com/g1` is a list
of `<li>` elements each containing exactly three `<div class="valPart">` children
(label, G1, G1 EDU). All 34 rows have exactly three cells; there are **no merged
cells**. The `/` symbol is Unitree's own notation for "not supplied" — the same
notation used for `Single Hand Degrees of Freedom || /` and
`High Computing Power Module || /`.

**Verify the Go2 table structure.** 38 rows × 4 cells, no merges. The `●` / `◕` /
`○` glyph legend (`●` = Yes/Support, `◕` = Basic support, `○` = No) exists in the
`about_Go2` doc page but not on the website. Earlier suspicion that
"Basic Computing Power" or "Secondary development" were merged cells across
columns is **disproven** — `○` is a real value meaning "no".

**Verify the Go2 gating from the primary source.** Unitree developer FAQ, verbatim:

> "**Only EDU or above versions can be developed, other versions such as PRO and
> AIR versions do not open the secondary development interface**, EDU version can
> be equipped with 40tops or 100tops docking station for development."

---

## 1. G1 variants

### 1.1 G1 vs G1 EDU — official table, verbatim

| Parameter | G1 | G1 EDU |
|---|---|---|
| Height (stand) | 1320 × 450 × 200 mm | 1320 × 450 × 200 mm |
| Height (folded) | 690 × 450 × 300 mm | 690 × 450 × 300 mm |
| Weight (with battery) | About 35 kg | About 35 kg + |
| **Total DOF** | **23** | **23–43** |
| Single leg DOF | 6 | 6 |
| Waist DOF | 1 | 1 + (opt. 2 more) |
| Single arm DOF | 5 | 5 |
| **Single hand DOF** | **`/`** | **7 (Dex3-1) + 2 (opt. wrist)** |
| Joint output bearing | Crossed roller, industrial grade | Same |
| Joint motor | Low-inertia high-speed inner-rotor PMSM | Same |
| **Max knee torque** | **90 N·m** | **120 N·m** |
| **Arm max load** | **≈2 kg** | **≈3 kg** |
| Calf + thigh length | 0.6 m | 0.6 m |
| Arm span | ≈0.45 m | ≈0.45 m |
| Waist joint range | Z ±155° | Z ±155°, X ±30°, Y ±30° |
| Knee joint range | 0–165° | 0–165° |
| Hip joint range | P ±154°, R −30…+170°, Y ±158° | Same |
| Wrist joint range | P ±92.5°, Y ±92.5° | Same |
| Electrical routing | Full joint hollow | Same |
| Joint encoder | Dual | Dual |
| Cooling | Local air | Local air |
| Power supply | 13S Li-ion | 13S Li-ion |
| Basic compute | 8-core high-performance CPU | 8-core high-performance CPU |
| **Sensing** | **Depth camera + 3D LiDAR** | **Depth camera + 3D LiDAR** |
| **4-microphone array** | **YES** | **YES** |
| **5 W speaker** | **YES** | **YES** |
| WiFi 6 / BT 5.2 | YES | YES |
| **High computing module** | **`/`** | **Multiple vendors (Orin etc.)** |
| Smart battery (quick release) | 9000 mAh | 9000 mAh |
| Charger | 54 V 5 A | 54 V 5 A |
| Manual controller | YES | YES |
| Battery life | About 2 h | About 2 h |
| OTA | YES | YES |
| **Secondary development** | **`/`** | **YES** |
| **Warranty** | **8 months** | **18 months** |
| **Price (ex tax/shipping)** | **US $13,500** | **Contact sales** |

### 1.2 Five differences that actually matter

Everything else is cosmetic. These five decide outcomes:

1. **Secondary development: `/` vs YES.** Not a feature difference — an access
   difference. The $13,500 unit cannot run the project.
2. **Hands: `/` vs Dex3-1.** The standard G1 has no hands at all. Any manipulation
   concept requires the EDU.
3. **High computing module: `/` vs optional Orin.** The standard G1 has no GPU-class
   module. Anything beyond the 8-core CPU must run externally.
4. **Warranty: 8 vs 18 months.** For a project timeline this is material — a
   failure in month 5 is a total loss on the standard, a repair on the EDU.
5. **Knee torque 90 vs 120 N·m and arm load 2 vs 3 kg.** 50% more, but see
   `robot_comparison.md` §C — mass is not the binding constraint on the G1;
   force and stability are.

### 1.3 G1-EDU sub-configurations

VERIFIED from `support.unitree.com/home/en/G1_developer/get_sdk`:

| SKU | Configuration |
|---|---|
| G1-EDU Standard | 23 DOF |
| G1-EDU Advanced | 29 DOF |
| **G1-EDU Ultimate A** | 29 DOF + 2× Dex3-1 (**no tactile**) |
| **G1-EDU Ultimate B** | 29 DOF + 2× Dex3-1 (**tactile included**) |
| **G1-EDU Ultimate C** | 29 DOF + 2× **Inspire DFQ** hands |
| **G1-EDU Ultimate D** | 29 DOF + 2× **Inspire FTP** hands |

⚠️ **If manipulation is ever attempted, the hand choice is decision-critical.**
Ultimate B (Dex3-1 with tactile) and Ultimate C/D (Inspire, via `rt/hand_sdk`
weight-blended injection) are different manipulation systems with different
services, different topics, and different — both unmeasured — grasp performance.
No published grasp rate exists for any of them.

### 1.4 G1-D — a different robot entirely, and worth knowing about

**G1-D is not a legged robot.** It is the G1 upper body (7 DOF per arm) on a
telescoping column, on either a fixed base or a motorized chassis. It has no legs
and cannot walk. VERIFIED: `example/g1/g1d/g1d_arm_example.cpp` defines a 29-motor
layout with indices 0–11 (all hip/knee/ankle) annotated `// NOTE INVALID for g1d`.

⚠️ **G1-D is not in `unitree_ros2`** — only in `unitree_sdk2`. If the school owns
one, the ROS 2 route is different.

| Aspect | Finding |
|---|---|
| SDK surface | **Exactly two calls**: `Move(vx, vy, vyaw)` (vx ∈ [−1.5, 1.5] m/s, **vy ignored**), `HeightAdjust(vz)` (vz ∈ [−1.0, 1.0] → ±76.5 mm/s) |
| Odometry / pose / waypoints | ❌ **None in the SDK** |
| Height control | Requires a **hand-written PD loop** against a loosely-typed `Point32_` telemetry topic — the column has no position-command API |
| Vendor autonomy | A separate chassis platform documents laser SLAM (500×500 m), obstacle avoidance, multi-target navigation, ±5 cm positioning, laser-tag docking, auto return-to-dock, and an **HTTP REST API** (Python/Java/Node) with RoboStudio tooling |
| ⚠️ But | The same doc marks the chassis **HDMI and USB Type-C as "built into chassis, not currently open"** |
| Price | **Not published by Unitree.** Third-party street prices **$38,800–$51,900 USD** — roughly 3–5× the base G1 |

**Assessment:** G1-D is the most *natural* hardware for a delivery robot — wheeled,
stable, no fall risk, tall column, big payload. But its autonomy is a vendor-locked
product whose low-level data ports are documented as not open, at 3–5× the price,
with no published contract for the HTTP API. If the school owns a G1-D this
conclusion changes materially — **verify this first.**

---

## 2. Go2 variants

All four share: 70 × 31 × 40 cm standing (76 × 31 × 20 cm crouched), ~15 kg,
aluminium + high-strength engineering plastic, 28–33.6 V, ~3000 W peak, 12 DOF,
~45 N·m knee torque, intra-joint knee circuit, joint heat-pipe cooler, 4D LiDAR,
HD wide-angle camera, basic actions, OTA, RTT 2.0, graphical programming, front
lamp, WiFi 6, Bluetooth.

| Parameter | AIR | PRO | X | EDU |
|---|---|---|---|---|
| **Payload** | ≈7 kg (max ~10) | ≈8 kg (max ~10) | ≈8 kg (max ~12) | **≈8 kg (max ~12)** |
| **Speed** | 0–2.5 m/s | 0–3.5 m/s | 0–3.7 (max ~5) | 0–3.7 (max ~5) |
| Max climb/drop | 15 cm | 16 cm | 16 cm | 16 cm |
| Max slope | 30° | 40° | 40° | 40° |
| **Basic compute** | **`○` none** | 8-core CPU | 8-core CPU | 8-core CPU |
| Wireless vector (UWB) | ○ | ● | ● | ● |
| HD wide-angle camera | ● | ● | ● | ● |
| **Foot force sensor** | ○ | ○ | ○ | **●** |
| Auto-scaling strap | ○ | ● | ○ | ○ |
| 4G + GPS | ○ | ● (CN/GB) | ● | ● |
| Voice function | ○ | ● | ● | ● |
| ISS 2.0 | ○ | ● | ● | ● |
| Intelligent detection + avoidance | ● | ● | ● | ● |
| Charging pile | ○ | ○ | ● | ● |
| **Secondary development** | **○** | **○** | **◕ basic** | **● full** |
| Manual controller | optional | optional | ● | ● |
| **High compute module** | ○ | ○ | ○ | Optional Orin **40–100 TOPS** |
| **Depth camera** | ○ | ○ | ○ | **●** |
| Smart battery | 8000 mAh | 8000 mAh | 8000 mAh | **15000 mAh** |
| Battery life | ~1–2 h | ~1–2 h | ~1–2 h | ~2–4 h |
| Charger | 33.6 V 3.5 A | 33.6 V 3.5 A | 33.6 V 3.5 A | **33.6 V 9 A fast** |
| **Warranty** | **6 months** | 12 months | 12 months | 12 months |
| **Price (ex tax/freight)** | **$1,600** | **$2,800** | **$4,500** | **Contact your sales expert** |

### 2.1 Configuration-dependent features that change the recommendation

| Feature | Depends on | Consequence if wrong |
|---|---|---|
| **Programmability** | Must be EDU | Project is impossible |
| **Depth camera** | EDU only | CV pipeline must work on 720p RGB alone — possible but weaker |
| **Foot force sensors** | EDU only | Foot-force walking unavailable on X |
| **Battery** | EDU gets 15,000 mAh | On an 8,000 mAh pack, plan for **1–2 h**, not 2–4 h |
| **High compute module** | EDU only | Without it, no onboard CV; everything runs on the external host |
| **Orin variant** | 40 TOPS (Nano 8 GB) vs 100 TOPS (NX 16 GB) | Determines which detection models can run onboard |
| **Warranty** | AIR = 6 months | A 6-month warranty on a multi-month project is a real risk |

### 2.2 Battery detail

| | BT2-05 (standard) | BT2-06 (long endurance) |
|---|---|---|
| Capacity | 8000 mAh / **236.8 Wh** | 15000 mAh / **432 Wh** |
| Rated voltage | 29.6 V | 28.8 V |
| Weight | 2 kg | 2.5 kg |
| Charge current | 3.5 A | **9 A** |
| **Charge time** | **2 h** | **1 h** |
| Runtime (battery table) | 1–2 h | **3–5 h** |

⚠️ **Three-way official contradiction on long-battery runtime:** spec table says
"about 2–4 h"; the battery table says "3–5 h"; the `EconomicGait()` doc says "up to
4 hours". **Plan on ~2 h** — the most conservative official figure — until measured.

⚠️ `EconomicGait()` also states: "the robot's height is higher in this mode, and
the rate of power consumption is slower." So endurance costs ride height, which
interacts with the 16 cm step-climb limit. Do not use the endurance gait if the
route has thresholds.

⚠️ The 2.5 kg long battery is **17% of the robot's total mass**. Mass distribution
matters for payload capacity.

---

## 3. Sensor detail

### Go2 built-in 4D LiDAR (L2)

| Parameter | Value |
|---|---|
| Technology | Unitree 4D (3D position + 1D grayscale) |
| Ranging rate | **21,600 samples/s** |
| FOV | **360° × 96°** |
| Min detection | **0.05 m** |
| Size | 75 × 75 × 65 mm |
| Supply | 12 V DC, 905 nm |
| Rings (SDK output) | **18** |
| **Accessible via** | ✅ **DDS topics** (below) |

⚠️ **FOV contradiction:** website and docs say 360°×96°; `unilidar_sdk2` README
reports 180°/192°. 192 = 2 × 96, suggesting ±96° half-angle vs full FOV, but
**no document states this.** UNVERIFIED — flag to Unitree.

**The point cloud IS on DDS** (VERIFIED, official `LiDAR_service` doc):

| Topic | Content |
|---|---|
| `rt/utlidar/cloud` | Raw L2 point cloud (frame `utlidar_lidar`) |
| `rt/utlidar/cloud_deskewed` | Motion-deskewed (frame `odom`) |
| `rt/utlidar/height_map_array` | 2.5D height map, world/robot-centred |
| `rt/utlidar/range_info` | Simplified front/left/right obstacle distances |
| `rt/utlidar/switch` | Publish `"ON"`/`"OFF"` to stop/start rotation |

Message type is `PointCloud2_`, ROS 2 `sensor_msgs/msg/PointCloud2`-compatible.
⚠️ But `unitree_sdk2` ships **no** L2 client and contains **zero** `utlidar`
references — you subscribe with a generic `ChannelSubscriber`. That is
straightforward and also makes the data directly usable by ROS 2.

⚠️ **Critical dependency:** "the height map function requires the use of the
`sportmodestate` topic… **If the operation control is turned off, such as when
debugging your own operation control program, the height map data cannot be
obtained.**"

### Go2 camera

| Parameter | Value |
|---|---|
| Sensor | F2.2, FOV 120° |
| Resolution | 720p and 1080p dual stream |
| **Frame rate** | ⚠️ **CONTRADICTED**: `about_Go2` says **15 fps**; FAQ says 1080p@30 / 720p@60 |
| SDK interface | `VideoClient::GetImageSample(std::vector<uint8_t>&)` — **single JPEG snapshot**, service `"videohub"`, API 1001 |
| Real streaming | **GStreamer UDP H.264**: `udpsrc address=230.1.1.1 port=… ! application/x-rtp,media=video,encoding-name=H264 ! rtph264depay ! … ! appsink drop=1` |
| Depth camera | EDU only. **Model NOT FOUND** |

### G1 sensors

| Sensor | Status |
|---|---|
| Depth camera | Listed for both G1 and G1 EDU (verified: no merged cells). **Model NOT FOUND anywhere.** |
| 3D LiDAR | Listed. **LIVOX-MID360 explicitly named for G1-D**; the G1 SLAM doc references `rt/utlidar/cloud_livox_mid360`, strongly implying Mid360 on the legged G1 too |
| 4-mic array | YES (G1 and G1 EDU) |
| 5 W speaker | YES (G1 and G1 EDU) |
| Head display | ❌ **NOT FOUND** — no HMD on G1, G1 EDU, G1-D, or G1+ |
| Audio service | `voice` — offline ASR + TTS + playback + volume + RGB LED. **"Mixed Chinese and English modes are not supported."** |
| Tactile (hands) | Dex3-1: 9 pressure-sensor arrays per hand (33 elements); Dex5-1: 94 per hand |

⚠️ **Do not conflate G1 with G1+.** `unitree.com/G1pl` documents a G1+ with
6 microphones, 2× 10 W speakers, **binocular + chest wide-angle cameras** (not
"depth camera + 3D LiDAR"), head-integrated tactile sensors, continuous external
power, 25–45 DOF, 1340 mm, arm load 3–4 kg. Different product.

---

## 4. Dexterous hands (G1 family only)

| Hand | DOF | Fits G1? | Tactile | Force control | Interface | Notes |
|---|---|---|---|---|---|---|
| **Dex3-1** | 7 (thumb 3, index 2, middle 2) | ✅ | **33 sensors** | ✅ force-pos hybrid | USB2.0 / RS485 24 V | The G1 spec table's "force control three-fingered hand" |
| **Dex5-1 / 5-1P** | 20 (16 active + 4) | ⚠️ NOT FOUND explicitly | **94/hand** | ✅ | USB2.0 | G1 fit not stated |
| **Dex5-S / S Pro** | 22 | ⚠️ NOT FOUND explicitly | not confirmed | ✅ direct-drive backdrivable | GbE/USB/RS485 | from **$6,350** |
| **Dex2/5** | 10 (2 actuated, tendon) | ✅ "Compatible with G1/R1" | none | gear-tendon | RS485 | 4 fingers share 1 DOF |
| **Dex1-1** | gripper, 90 mm stroke | ✅ (also H2, R1) | Advanced has 1080p camera | ✅ 5–120 N | RS485 6 Mbps | Via `dex1_1_service` |

**Dex3-1 control** (VERIFIED): 7 motors, motor order
`0 thumb_0, 1 thumb_1, 2 thumb_2, 3 middle_0, 4 middle_1, 5 index_0, 6 index_1`.
Topics `rt/dex3/{left,right}/cmd` and `rt/dex3/{left,right}/state`. Mode byte
packed `id & 0x0F | (status & 0x07) << 4 | (timeout & 0x01) << 7`.
Gains: rotate `kp=0.5 kd=0.1`, grip `kp=1.5 kd=0.1`, stop `kp=0 kd=0`.

Third-party hand services (all **C++ only**, all separate repos, all install as
systemd services):

| Service | Hand | Topics |
|---|---|---|
| `brainco_hand_service` | BrainCo Revo2 (6 DOF) | `rt/brainco/{left\|right}/{cmd\|state}` |
| `linker_hand_service` | Linkerbot O6 (6 DOF, Modbus) | `rt/linker/{left\|right}/{cmd\|state}` |
| `dex1_1_service` | Dex1-1 | — |

⚠️ All three consume `unitree_go` (`MotorCmds_`) message types — **Go2 messages —
on a G1 robot.** Cross-family message reuse. And all three are C++ with no Python
binding.

---

## 5. Contradictions and unresolved items

Do not resolve these by guessing. Each needs a call to Unitree or a physical check.

| # | Item | Source A | Source B | Status |
|---|---|---|---|---|
| C1 | **Go2 L2 vertical FOV** | website/docs 96° | `unilidar_sdk2` 180°/192° | UNVERIFIED |
| C2 | **Go2 camera fps** | `about_Go2` 15 fps | FAQ 30/60 fps | UNVERIFIED |
| C3 | **Go2 long-battery runtime** | spec 2–4 h | battery table 3–5 h; gait doc 4 h | Plan on 2 h |
| C4 | **Go2 LiDAR dimensionality** | website "4D" | `about_Go2` spec row "3D" (same page) | Cosmetic |
| C5 | **G1 EDU compute module** | "Multiple brands... Orin" | dev doc "NVIDIA Jetson Orin" | Dev doc stale |
| C6 | **G1-D column speed** | docs ~60 mm/s | `g1_agv_client.hpp` 76.5 mm/s | **28% gap** |
| C7 | **G1-D total weight** | marketing ~90 kg (both configs) | dev doc ~50 kg / ~80 kg — and self-contradicts (chassis alone = 63 kg) | Do not rely |
| C8 | **G1 "right kiss" action ID** | C++ = 12 | Python = 13 | One is wrong |
| C9 | **G1 arm action JSON key** | C++ `{"action_id":N}` | Python `{"data":N}` | One is wrong |
| C10 | **G1 emergency-stop RC combo** | manual `L1 + A` | manual `L2 + B` | Verify on hardware |
| C11 | **Go2 X RC** | spec table: manual controller included | SBUS is EDU-exclusive | X likely has a proprietary RC |
| C12 | **G1-D chassis sensors** | "Physical Collision Sensor ×2" | "2 ultrasonic (0.5 m)" | "Physical Collision Sensor" undefined |
| C13 | **G1-EDU EDU cell merges** | Standard/Ultimate | Flagship | Naming differs by source |
| C14 | **G1 user manual contamination** | Manual contains a **quadruped section** (5–35 °C, 25° slopes, 10 cm steps, 3 m distance) | That is **Go2/B2, not G1** | Do not attribute to G1 |

### Unresolved — requires a call to Unitree

1. What does Go2 X's `◕` **"Basic support"** concretely permit? No official
   definition exists.
2. Does Go2 X have RJ45 / DC-out / SBUS? The `Payload` doc says these are
   EDU-exclusive.
3. Is the Go2's internal L2 reachable by `unilidar_sdk2` from an external host?
   The README documents only a standalone-unit workflow (192.168.1.2 / ttyACM0).
4. Minimum turning radius, threshold heights, carpet-vs-tile guidance — **NOT
   FOUND** in any official source.
5. Payload centre-of-mass envelope / mounting-offset tolerance — **NOT FOUND**.
   The mounting map is published as a **PNG only**; dimensions are not in text.
6. G1 walking vx/vy/vyaw absolute limits, stand height, swing height — **NOT
   FOUND**. Only the `SetSpeedMode` caps (1.0 / 2.0 / 2.7 / 3.0 m/s) are published.
7. G1 charging time, IP rating, legged max incline — **NOT FOUND**.
8. Any official object-detection or person-following demo — **NOT FOUND**.
9. The name of FSM mimic motion `550507` — not published; enumerate at runtime.

---

## 6. Physical inspection checklist — do this first

This is what only a person standing in front of the robot can determine. Run it
before any procurement or design decision.

### Identity
- [ ] Photograph the model badge / nameplate. Record exact model string.
- [ ] **G1**: is it "G1" or "G1 EDU"? — the single most important fact for the G1.
- [ ] **Go2**: is it AIR, PRO, X, or EDU? Only EDU is confirmed programmable.
- [ ] Record serial number.
- [ ] Record firmware version (via the app's Device → Service Status screen).

### Compute
- [ ] Is a high-computing module present? Which model (Orin Nano 40 TOPS /
      Orin NX 100 TOPS / none)?
- [ ] Is the Go2 **expansion dock** present? Which type?
- [ ] Confirm: is there a second compute unit, or only the base 8-core CPU?

### Sensors
- [ ] G1: are Dex3-1 hands fitted? Which generation? Is the **tactile** array
      fitted (Ultimate A = no, Ultimate B = yes)? Or Inspire DFQ / FTP?
- [ ] Go2: is the **depth camera** present (EDU-only feature)? Where is it mounted?
- [ ] Go2: is the 4D LiDAR physically unobstructed? Any third-party sensor
      already installed on the spine?
- [ ] Go2: are the **foot force sensors** present (EDU-only)?

### Power
- [ ] Read the battery label. **8000 mAh or 15000 mAh?**
- [ ] Is the matching charger present? 33.6 V 3.5 A or 9 A?

### Mechanical
- [ ] Go2: photograph the spine. Locate the four M5 hardpoints and measure the
      usable flat mounting area in mm (Unitree publishes this as a PNG only).
- [ ] Go2: is any carrier, tray, or payload structure already fitted?
- [ ] G1: is a protective frame and protective rope present? **Required by
      Unitree for any operation.**

### Environment (survey the space, not the robot)
- [ ] Measure the demonstration corridor: width, length, floor surface
      (tile / carpet / vinyl), threshold heights, ramp gradients.
- [ ] Identify the network point and whether a **wired Ethernet** drop can reach
      the robot. Both robots require wired Ethernet for development.
- [ ] Confirm whether the school permits purchasing a MID-360 or XT16 LiDAR, and
      whether a docking station can be bought.
- [ ] **Ask the school's IT/网络 policy team whether a robot with known telemetry
      behaviour may join the network.** The G1 has documented fleet-wide
      cryptographic and telemetry issues; some institutions will refuse.

### Record, do not assume
For each item, record: **present / absent / unknown**, with a photograph. Where a
check cannot be completed, write "unknown" and carry it as a risk. Do not fill gaps
with the value from another variant.
