# SHANA — Technical Feasibility Report

**Phase 0 · Documentation only · Research date: 2026-10-10**

This report presents sourced technical findings for the proposed SHANA system on
a school-owned Unitree Go2 EDU. Every important claim carries a source, a
confidence level, and its hardware assumptions.

**No robot was operated, connected to, or commanded. No packages were installed.
No network was probed. No hardware was configured.**

---

## Confidence definitions

| Level | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in official manufacturer documentation or official source code, quoted |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | Manufacturer documents it; not confirmed on the school's specific unit |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | Our engineering judgement; no manufacturer claim |
| ❓ **UNKNOWN** | No evidence found. Neither assumed true nor false |
| ⛔ **NOT PERMITTED / NOT RECOMMENDED** | Prohibited by law, manufacturer terms, or safety judgement |

---

## 1. Executive summary of feasibility

| Component | Feasibility | Confidence |
|---|---|---|
| Single-floor autonomous delivery | **Feasible** | ✅ Interfaces verified; 🧪 real-world reliability unproven |
| Multi-floor via human-assisted lift | **Feasible, with a defined escort protocol** | ✅ Protocol established in commercial practice; 🧡 specific car unverified |
| **Autonomous lift operation** | **Not feasible** — no sanctioned interface exists for Johnson lifts | ⛔ Prohibited except via a written route that does not currently exist |
| **Stairs as a routine transit method** | **Not established** — likely not feasible with payload | ⛔ Manufacturer forbids complex terrain; margin is ~0–10 mm |
| **Floor identification** | **Impossible with stock sensors** | ✅ Verified negative — no altitude/pressure field exists |
| **Autonomous return-to-dock** | **Not available** | ✅ No dock service in the documented service list |
| Battery telemetry sufficient for mission planning | **Yes** | ✅ `BmsState` has `soc` + signed `current` |
| LoRa backup telemetry | **Feasible, but probably unnecessary first** | ✅ Regulation verified; 🧪 value is a judgement call |
| Private Wi-Fi for teachers | **Feasible** if school permits APs | ❓ Blocked on school approval |

**The single largest finding:** the aspiration as stated — autonomous delivery
across three blocks and four floors — depends on **two capabilities that are
either legally prohibited (autonomous lift use) or physically unestablished (stairs
with payload), plus one capability that is impossible with stock sensors (floor
identification).** Each has a documented, feasible workaround. None has a
same-day solution.

---

## 2. Robot platform findings

### 2.1 Development access — VERIFIED

| Claim | Evidence |
|---|---|
| **Only EDU and above can be programmed** | Unitree developer FAQ, verbatim: *"Only EDU or above versions can be developed, other versions such as PRO and AIR versions do not open the secondary development interface"* — `https://support.unitree.com/home/en/developer/faq` |
| Development host: **Ubuntu 20.04 / 22.04 only** | Same FAQ. *"Development on Mac or Windows is currently not supported."* |
| **Wired connection only** | Same FAQ: *"No, Go2 only supports wired connection for development."* |
| Transport: **Cyclone DDS**, domain 0, explicit NIC | `unitree_sdk2` — `ChannelFactory::Instance()->Init(0, argv[1])` |
| Robot IP `192.168.123.161`; host `192.168.123.222` on `192.168.123.0/24` | Unitree Quick Start. *"It is not allowed to set this adapter's IP address to 192.168.123.161"* |

**Confidence: ✅ VERIFIED.** **Hardware assumption:** the school owns an EDU
variant. **If it is a PRO or AIR, the project cannot proceed at all.**

### 2.2 Hardware specification — VERIFIED

Source: `https://www.unitree.com/go2/` and
`https://support.unitree.com/home/en/developer/about_Go2`

| Parameter | EDU value |
|---|---|
| Dimensions (standing) | 70 × 31 × 40 cm |
| Dimensions (crouching) | 76 × 31 × 20 cm |
| Weight (with battery) | ~15 kg |
| DOF | 12 |
| Peak joint torque | ~45 N·m |
| **Payload** | **≈8 kg (max ~12 kg)** |
| **Max climb / drop height** | **~16 cm** |
| Max climb angle | 40° |
| Speed | 0–3.7 m/s (max ~5) |
| **LiDAR** | **4D L2: 360° × 96°, 21,600 samples/s, 0.05 m min detection** |
| Camera | HD wide-angle, F2.2, 120° FOV |
| **Depth camera** | **EDU only** |
| Foot force sensors | EDU only |
| Battery (EDU standard) | **BT2-06, 15,000 mAh / 432 Wh / 2.5 kg** |
| Charge time (EDU) | 1 h 15 (fast, 33.6 V 9 A) |
| Operating temperature | 5–35 °C |
| Charging pile support | X and EDU only |

**Confidence: ✅ VERIFIED as a model specification. 🟡 NOT VERIFIED that the
school's specific unit has the 15,000 mAh pack, depth camera, or charging pile.**
EDU ships **one** battery in the box — a spare is not included.

### 2.3 Locomotion and gait interfaces — VERIFIED

`SportClient` exposes 39–41 methods including `Move(vx, vy, vyaw)`,
`StopMove()`, `Damp()`, `BalanceStand()`, `StandUp()`, `StandDown()`,
`RecoveryStand()`, `Sit()`, `RiseSit()`, `SpeedLevel()`, `ClassicWalk(bool)`,
`EconomicGait()`, `StaticWalk()`, `TrotRun()`, `AutoRecoverSet(bool)`,
`FreeAvoid(bool)`, `SwitchAvoidMode()`.

### 2.4 Floor transitions — the critical findings

#### 2.4.1 Step height margin — 🔴 CRITICAL

| Fact | Value | Label |
|---|---|---|
| Go2 max climb/drop height | **160 mm** | ✅ |
| Manual: *"not currently recommended that robot walk stairs higher than 16cm… otherwise it is likely to trip over the foot"* | 160 mm | ✅ |
| Manual: *"it is **forbidden** to use the robot on the occasions where the terrain is complicated… steps higher than 16cm"* | — | ✅ |

| Typical riser | vs 160 mm limit | Verdict |
|---|---|---|
| 150 mm | +10 mm (6.7 %) | Inside spec — **10 mm of tolerance** |
| 160 mm | **0 mm** | Exactly at limit. **Zero margin** |
| 170 mm | −10 mm | **Exceeds** |
| 180 mm | −20 mm | **Exceeds** |

🧪 **Aggravating factors, our engineering judgement:**
- The 16 cm figure is a **no-load** figure. No Unitree document states a derated
  step height under payload.
- Descent margin is normally **smaller** than ascent in legged robots. No figure
  published.
- Stair surfaces are frequently polished terrazzo or tile. The manual
  **explicitly names slip** as the failure mode and forbids low-friction ground.

#### 2.4.2 The stair gait setter was removed — 🔴 CRITICAL

| Firmware | Dedicated stair API | Evidence |
|---|---|---|
| **< 1.1.6** | ✅ `SwitchGait(3)` = forward climbing, `SwitchGait(4)` = reverse climbing | ✅ VERIFIED — Unitree `sports_services` doc, verbatim: *"3 is forward climbing mode, and 4 is reverse climbing mode"* |
| **≥ 1.1.6 (current)** | ⛔ **`SwitchGait` is ABSENT** from Motion Services Interface V2.0 and from `SportClient` | ✅ VERIFIED — `SwitchGait(int d)` still exists in the B2-W client, so removal from Go2 is deliberate |

**Consequence:** on a current-firmware EDU, stair capability exists only as an
**emergent property of `ClassicWalk`**, which Unitree describes as *"AI-classic
gait, with strong adaptability to complex terrain, supports climbing stairs, gravel,
meadows, and slippery surfaces."* That is a terrain-adaptation claim, **not a
stair mode with its own parameters, step-height setting, or safety envelope.**

🟡 The stair gaits still exist in the *consumer* firmware — the User Manual
documents two remote-control modes: *"Stair Climbing Mode 1: go upstairs forwards
and downstairs backwards"* and *"Mode 2: go downstairs forwards."* Whether these
can be invoked from the EDU SDK is **UNKNOWN** and should be tested directly.

#### 2.4.3 Ascent vs descent — 🧪 documented asymmetrically, verified worse in practice

Unitree provides two modes precisely because backing down is the fallback.
Independent reports (UNVERIFIED, third-party):

| Source | Report |
|---|---|
| DroneBlocks, 2024-10 | On 8-inch (203 mm) steps: *"going forward struggles a little bit more but able to compensate… we go backwards, he'll see that go to loses balance a little bit on both stairs but is able to recover"* |
| Miraenda, 2024-03 | *"Go2 by Unitree can walk upstairs but has an issue walking downstairs"* |
| mybotshop forum, 2025-09 | New owner, post-update: *"I tried out all different walking modes… for all of them the robot just stopped in front of the stairs"* |

⚠️ **There is no report anywhere of a Go2 EDU descending stairs while carrying 8 kg.**

#### 2.4.4 Stair feasibility verdict

> **NOT ESTABLISHED.** No Unitree documentation, no published test data, and no
> independent field report demonstrates a Go2 EDU climbing school stairs while
> carrying an 8 kg payload with a person on the staircase.

**Confidence: ✅ that it is not established.** **Action:** a supervised, unloaded,
single-staircase protocol (`implementation_roadmap` Phase 4 in the prior research;
detailed test ladder available) is the only way to resolve it. **Measure the actual
riser heights first** — if any exceed 160 mm, the question is moot.

### 2.5 Floor identification — ✅ VERIFIED NEGATIVE

This is the finding with the widest architectural consequence.

**Every accessor across every Go2 IDL message was enumerated.** Result:

- ⛔ **No `altitude`. No `pressure`. No `barometer`. No floor field. Zero matches.**
- `IMUState.msg` has exactly **five** fields: `quaternion[4]`, `gyroscope[3]`,
  `accelerometer[3]`, `rpy[3]`, `temperature` (IMU die, not ambient).
- `SportModeState.position[3]` is **dead-reckoned odometry** — drifts, resets on
  reboot. Not absolute.
- `SportModeState.body_height` is the **commanded stance height above ground**,
  relative to the gait. Not altitude.
- `HeightMap.msg` `origin` is **`float32[2]` — XY only, no Z datum.** It is a
  local 2D terrain-height grid, not an absolute reference. **It carries no
  floor-identifying information.**

**Partial mitigation:** the L2 is **not** a 2D lidar. Its **360° × 96°** field of
view is genuinely three-dimensional, so ceiling height, floor-to-ceiling geometry,
and stairwell voids *are* observable in the point cloud. In most Indian school
buildings floor plates are not identical.

> ✅ **VERIFIED: the Go2 EDU has no sensor that can determine which floor of a
> building it is on.** Geometry gives partial, non-authoritative discrimination.
> **An external floor-identity source is mandatory for multi-floor operation.**

### 2.6 The silent wrong-floor failure mode — 🔴 CRITICAL

Scenarios are routinely conflated. They are different problems.

| Term | Meaning |
|---|---|
| **Relocalisation** | Determining pose *within a known map*. A within-floor problem |
| **Floor identification** | Determining *which* map applies. A discrete classification. **Harder and unsolved** |

**The dangerous scenario:**

> A human carries the Go2 up a flight of stairs and sets it down on floor 3.

1. Odometry is destroyed. Not fatal — that is what relocalisation is for.
2. Scan-matching against floor 3 **succeeds**.
3. Scan-matching against floor 1 **also succeeds**, if the corridor is similar —
   which in a school it usually is.
4. **The robot therefore does not know it is on the wrong map.** Nothing crashes,
   nothing logs an error. It confidently plans against floor 1's geometry while
   standing on floor 3.

This is corroborated by independent work on floor-ambiguity in lifts:

> *"it is necessary for the robot to leave the lift before it can detect differences
> in the layout of the various floors, **as the areas immediately adjacent to the
> lift are identical.** Therefore, this measure is only taken as a final check."*
> — McAree et al., Sheffield

**Mitigation (mandatory, three parts):**
1. **Unique ArUco/AprilTag marker per floor**, marker ID encoding the floor. Read
   by the head camera.
2. **A mandatory human confirmation event on every floor transition.**
3. **An explicit `RELOCATION` event** whenever anyone manually handles the robot —
   invalidates the pose, forces re-localisation before any new task is accepted.

⛔ **Do not use LoRa RSSI or BLE RSSI as the floor-identity source.** A beacon ID
is a legitimate floor signal; a *signal strength* is not a positioning input.

### 2.7 Battery, charging and recovery

#### 2.7.1 Telemetry location — 🔴 CORRECTS A COMMON ASSUMPTION

✅ **VERIFIED:** `SportModeState` has **no battery fields at all**. Its full field
list contains `error_code`, `imu_state`, `gait_type`, `foot_force`, `position`,
`velocity`, `body_height`, `range_obstacle` — and nothing else.

**Battery is on `rt/lowstate` → `LowState.bms_state`.** `BmsState.msg`:

```
uint8  version_high / version_low
uint8  status        ← pack fault code
uint8  soc           ← state of charge, percent
int32  current       ← signed, mA  ← enables coulomb counting
uint16 cycle         ← charge cycle count
int8[2]   bq_ntc     ← battery temperatures
int8[2]   mcu_ntc
uint16[15] cell_vol  ← per-cell mV, enables imbalance detection
```

`LowState.msg` also carries `power_v`, `power_a`, `temperature_ntc1/2`.

✅ **This is sufficient for mission planning:** `soc` for the abort threshold,
signed `current` for coulomb counting, `cell_vol[15]` for pack health.

#### 2.7.2 Low battery behaviour — 🔴 ARCHITECTURALLY IMPORTANT

✅ **VERIFIED**, Unitree pilot-lamp table:

> *"Low flash rate in yellow — **Low battery warning, will automatically crouch
> down within 10 minutes**"*

| Consequence | Design implication |
|---|---|
| A **crouch on a staircase is a fall** | Stair testing abort threshold must be ≥40 % |
| 10 minutes is the **entire** reaction budget | Mission abort at **30 %**, not 20 % |
| No documented controlled safe-landing | **Never rely on the robot's own low-battery logic** |

#### 2.7.3 Charging and return-to-dock — ✅ VERIFIED NEGATIVE

| Item | Finding |
|---|---|
| Charging pile support | ✅ X and EDU only |
| **Autonomous return-to-dock** | ⛔ **NOT FOUND.** No charging or dock service in the documented Go2 service list (`sport_mode`/`mcf`, `basic_service`, `chat_go`, `net_switcher`, `obstacles_avoidance`, `ota_box`, `unitree_lidar`, `utrack`, `video_hub`, `webrtc_bridge`, `vui_service`) |
| **An "Aruco Recharge Service Interface" page EXISTS** in the official doc navigation tree | 🟡 Page body would not render through 8 URL variants. **Not read.** |
| Third-party implementation uses topic **`rt/aruco_cmd`** with `StartRecharge(int repeat, double period_sec)` / `StopRecharge(...)` | 🧪 UNVERIFIED — third-party fork, not official |
| Mechanism: the **charging pile presents an ArUco marker**; the robot's onboard camera recognises it and drives onto the pile | 🧪 Corroborated across two sources |

> **This is the key architectural insight:** the same ArUco marker pipeline that
> enables automatic docking **is the same technology needed for floor
> identification** (§2.6). One investment, two capabilities.

⚠️ **But:** the mechanism is marker-based docking driven by a command we publish.
**Something else must decide when to go there and route it there.** That logic is
ours to build.

#### 2.7.4 Recovery behaviours and payload safety

| Method | Official description | Payload safety |
|---|---|---|
| `Damp()` | *"All motor joints stop moving and enter damping state. **This mode has the highest priority and is used for emergency stops***" | ✅ Use for e-stop. Accepts payload drop |
| `RecoveryStand()` | *"**Regardless of whether it is fallen or not, it will recover to standing**"* | ⛔ **UNSAFE with payload** — it will rise even from a heap. Violent with 8 kg |
| `StandDown()` / `Sit()` / `BalanceStand()` / `StopMove()` | Controlled | ✅ Safe |
| `AutoRecoverSet(bool)` | Enable/disable auto-recovery | 🧡 **Unknown under payload.** Recommendation: **leave OFF during payload missions** |

#### 2.7.5 Endurance — ⚠️ MANUFACTURER FIGURES CONFLICT

| Source | Figure |
|---|---|
| `about_Go2` battery table, BT2-06 | **3–5 h** |
| `about_Go2` product matrix, EDU | **"About 2–4h"** |
| `EconomicGait()` doc | *"up to 4 hours on a full charge"* |
| **Go2 User Manual** | *"about **1-2 hours** with no load static standing and upright walking alternately"* |

✅ **VERIFIED: the manufacturer's own figures span 1–5 hours depending on document
and load condition.** There is no single authoritative number.

**🧪 SHANA planning figure: 90 minutes of active mission per charge**, derated for
payload (manual explicitly lists *"running with a load"* as an endurance reducer),
compute load, camera, and the school reality of stops and waits.

✅ **Power reducers, verified from the manual:** *"walking at a faster speed… standing
with the robot legs bent, running with a load, walking at a lower body height, and
terrain with appropriate undulations and slopes."*

→ **Use `EconomicGait()`. Hold maximum body height. Avoid `WalkUpright` and
`CrossStep`** (both documented as motor-overheating gaits).

### 2.8 Safety documentation — ✅ VERIFIED

Verbatim from the Go2 User Manual:

> *"Because the levels of proficiency of the operator are different. To safety
> reasons, it is not currently recommended that robot walk stairs higher than 16cm,
> otherwise it is likely to **trip over the foot** due to improper operation."*

> *"It is **forbidden** to use the robot on the occasions where the terrain is
> complicated, the ground is wet, there are sundries on the ground, the terrain is
> undulating (steps higher than 16cm, etc.), and the slope is large (more than 40°)
> and there are sharp objects on the ground or in the periphery."*

> *"Hand may be **pinched at the motion joints**, such as at the knee joint."*

> *"Please keep it under control in the users' view… and keep the robot maintain a
> **safe distance of at least 2 meters** from obstacles, complex ground, crowds,
> water, and other objects."*

> *"**This product is not a toy and is not intended for use by persons under the
> age of 14.** Keep out of reach of children and be careful when operating in the
> presence of children."*

**Implication for SHANA:** this is a supervised device in a school with children
under 14. **Unsupervised operation is outside anything the manufacturer
describes.**

---

## 3. Communications findings

### 3.1 Robot link — ✅ VERIFIED, and it is not Wi-Fi

| Property | Value |
|---|---|
| Protocol | Eclipse Cyclone DDS — the **only** verified path |
| Physical | **Wired Ethernet, mandatory** |
| Network | Dedicated `192.168.123.0/24`, separate from the smartboard LAN |
| Host | Ubuntu 20.04/22.04, dedicated NIC, static IP |

⚠️ **Architectural consequence:** the robot↔server link is **Ethernet**. A WLAN
failure does **not** stop the robot. This is the single most important argument
against designing the robot link over Wi-Fi — and against treating LoRa as a
solution to a Wi-Fi problem that does not exist.

**Preferred physical design: a dedicated USB-Ethernet NIC on the AI Lab server,
directly cabled to the robot.** This requires **no school network change** — the
robot is simply a directly-connected Ethernet peer.

### 3.2 India RF regulation — ⚠️ WHERE INTERNATIONAL SOURCES WILL BE WRONG

Full detail in `appendix_india_rf_research.md`. The errors that matter:

| # | Finding | Impact |
|---|---|---|
| 1 | Licence-exempt band is **865–868 MHz, not 868 MHz**. Default EU868 channel (868.1 MHz) is **outside it** | An unmodified EU868 config is **non-compliant** |
| 2 | India cites **EN 300 220**, not EN 303 708 | A CE/RED report is built on the wrong standard |
| 3 | Power limited in **e.r.p. (half-wave dipole), not EIRP**. 500 mW e.r.p. = **29.15 dBm EIRP** | Quoting 500 mW as EIRP overshoots by 2.15 dB |
| 4 | Duty cycle: **2.5 % end device, 10 % gateway, 1 % at 25 mW** | At SF12 a 350-byte report fits only every ~2 min |
| 5 | **T-Beam at full 32 dBm is illegal** — ~1531 mW e.r.p. vs a 500 mW ceiling | Cap TX to ~24–27 dBm + implement Adaptive Power Control |
| 6 | **Telecommunications (Radio Equipment Possession Authorisation) Rules, 2026**, G.S.R. 592(E), 8 Jul 2026; applications opened 06.08.2026. Telecom Act 2023 §3(1)(c) now requires authorisation *to possess* radio equipment | ⚠️ **Nearly all secondary sources are stale.** Needs written DoT WPC clarification |
| 7 | **ETA is now 100 % self-declaration**, ₹10,000, lifetime, since 09.09.2024 | Low cost for project quantity |
| 8 | Popular claim that India caps 2.4 GHz Wi-Fi at 20 dBm EIRP is **wrong** for the 2005 rules (30 dBm TX / 36 dBm EIRP) | Genuine ambiguity between instruments; do not rely on blogs |

### 3.3 LoRa — honest assessment

| Question | Answer | Confidence |
|---|---|---|
| Better coverage than Wi-Fi? | **No.** A 16-storey study found LoRa *still lost the link across floors* | ✅ VERIFIED |
| What does it buy? | **Independence from the WLAN** — survives AP reboot, PoE failure, cable cut | ✅ |
| A control path? | ⛔ **Never.** RTT hundreds of ms to seconds, non-deterministic, no QoS | ✅ |
| A navigation channel? | ⛔ **Never.** | ✅ |
| A positioning source? | ⛔ **Never.** RSSI must not drive positioning | ✅ |

**Decision rule from the research:**

> If the robot is docked/mains-powered and the school has ≥1 working AP per floor,
> **a local Wi-Fi mesh makes LoRa largely redundant.** LoRa earns its keep only if
> the robot runs on battery for hours, **or** there is a named room (stairwell,
> sports hall, courtyard) where Wi-Fi does not reach.

**Recommendation:** build the Ethernet + Wi-Fi path first. Add the LoRa heartbeat
later, only if `NET-05` identifies a coverage gap. It is cheap insurance that will
not change the architecture — which is exactly why it is a good thing to add late.

### 3.4 ESP32

✅ ESP32 **adds nothing to the robot link** — the robot has no ESP32-accessible
interface. Its only role is as a **standalone telemetry beacon on the robot**,
bridging LoRa → gateway → server.

🧪 Honest caveat from measured data: **Wi-Fi association costs ~3.9 s** (wake +
DHCP + TLS + MQTT) versus ESP-NOW at 564 ms. For a battery node this is a 6.9×
battery-life penalty. On a **docked robot, this is irrelevant** — further
evidence that LoRa's value here is failure independence, not battery life.

---

## 4. Elevator findings

### 4.1 Manufacturer identity — ✅ VERIFIED, with a correction

Two distinct entities exist:

| Entity | Detail |
|---|---|
| **Johnson Lifts Pvt. Ltd.** | Chennai, est. 1963, ~80 branches, 205+ service centres — `https://www.johnsonliftsltd.com/` |
| **Toshiba Johnson Elevators (India) Pvt. Ltd.** | JV — Toshiba 51% / Johnson 49% |

⚠️ **ACTION: determine which nameplate is on the installed COP/LOP.** The
integration counterparty differs.

⚠️ **Correcting a common error:** the destination-control system called "SIRIO" is
**not** a Johnson product — it is an Italian hydraulic home lift by Extrema. The
Toshiba JV's DCS is called **FLOORNAVI**.

⚠️ **Correcting a standards error:** the governing Indian standard family is
**IS 17900 (2022 series)**, which Johnson itself cites. *IS 14447* is *"Refractories
for use in foundry industry"* and *IS 6914* is *"Cast billets ingots…"* (withdrawn).
Neither is a lift standard. **Confirm with the service provider which standard the
specific installed lifts were built to.**

### 4.2 What Johnson actually offers — ✅ VERIFIED, and it is the key find

**EyeRIS — contactless call-booking system.** Official Johnson site, verbatim:

> *"Once the Johnson EyeRIS app is downloaded, it connects the smartphone to the
> lift system using QR code technology and the user operates the lift by just
> tapping the buttons on the phone's screen. The user will only need to indicate
> the 'Present Floor' and the 'Destination Floor' on the app and once done, the
> lift arrives to take the user to the destination floor."*

> *"Just place the mobile in front of the QR code available in the Landing
> Operating Panels… First select 'present floor' and then press 'destination floor'
> … Please wait for the Lift to reach present floor."*
> — `https://www.johnsonliftsltd.com/eyeris/`

**Why this matters:** Johnson already ships a **QR-code-tagged landing panel that
a phone can read**, and a phone can issue both hall and destination calls. A
companion Android device is a first-class, **manufacturer-sanctioned** client —
**no panel attachment, no servo, no wiring.**

⚠️ **Three unknowns, must be asked in writing:**
1. Retrofitting available for lifts installed earlier?
2. Is the QR a **passive printed label** or an **active electronic component**?
   (If active, fitting it is a modification requiring approval.)
3. Can a **non-human / robot-provisioned client** be issued?

**Also verified:** Johnson's **WATCH** IoT system is **monitoring-only** —
lift → Johnson's data centre → technicians. **No inbound command path** for a
third party was found. ⛔ **No Johnson public API, developer portal, SDK, or
robot-integration product exists** (high-confidence negative).

### 4.3 What is prohibited — ✅ VERIFIED with statutory citation

**Maharashtra Lifts, Escalators and Moving Walks Act, 2017, Section 10**, verbatim:

> *"No additions or alterations, which cause any change in the characteristics of
> lift or escalator or moving walk… shall be made to any lift or escalator or
> moving walk installation **except with the prior permission in writing of an
> Electrical Inspector (Lifts)**."*

**Delhi / Bombay Lift Rules, Rule 6**, still cited as binding by the Supreme Court
in 2026:

> *"The owner shall not carry out any additions or alternations to an installed
> Lift…"* · *"No person shall willfully interfere with any mechanism of the Lift…"*

Comparable provisions exist in the **Tamil Nadu Lifts Act 1997** (s.15) and the
**Karnataka (2012) and Haryana (2008) Acts**.

### 4.4 Liability — ✅ VERIFIED, Supreme Court of India, 29 July 2026

***M/s. Otis Elevator Co. (India) Ltd. v. Rashmi Handa***, Civil Appeal No. 4081
of 2014. An officer died when MES personnel manually released the mechanical
brake during a rescue from a lift stalled between floors.

Held: *"A party that undertakes the comprehensive maintenance of a machine, which
is in the nature of a vehicle, **owes a heightened duty of care towards its
users**."* Liability apportioned 70% Otis / 25% MES / **5% the occupier**.

**Implications:**
1. **Unauthorised third-party intervention does not exonerate anyone.**
2. **The school, as occupier, was held liable — for failure of oversight.**
3. **A robot tampering with the COP would be far more egregious than a human rescue
   reflex** — it is planned, repeatable, and not an emergency.
4. Manufacturer "stickers prohibiting interference" were held **insufficient** as a
   safety measure alone.

### 4.5 Physical fit — ✅ VERIFIED (fit is not the problem)

| Dimension | Robot | Lift car | Clearance |
|---|---|---|---|
| Width | 310 mm | 900 mm door | ~3× |
| Length | 700 mm standing / 760 crouched | ~1400 mm | ~2× |
| Mass | 15 kg | 544 kg rated | 2.8 % |
| Step height | 160 mm | ±10 mm levelling | Not a barrier |

🧡 **The genuine physical problem:** a quadruped has four independently actuated
legs and **must turn around inside the car** to exit facing the door. Real
manoeuvre-planning problem, same as wheeled robots have.

❓ **NOT FOUND — measure on site:** actual threshold sill gap, actual door clear
opening, actual car dimensions. Get them with a tape from the lift's own drawings.

### 4.6 The defining risk — lift stalls with the robot inside

> **This is the *Otis v. Rashmi Handa* scenario exactly.** The robot's battery is
> dead in ≤4 h; a mechanic's rescue will typically take far longer. **The robot must
> be assumed dead by the time help arrives.**

The **defence** is therefore not technical — it is the human-escort protocol:
a named adult rides with the robot, every trip, no exceptions.

### 4.7 Industry precedent — unanimous

| Company | Behaviour |
|---|---|
| **Relay Robotics** | Autonomous, **requires** service company + OEM involvement, written proposal. *"This process can take up to 2 months."* **Johnson not in supported list.** Relay2 explicitly **eliminated** the mechanical button interface in favour of cloud-based integration |
| **Otis / TK / Schindler** | Sanctioned APIs. **Otis:** *"Our technology, obviously, only works with Otis equipment"* |
| **Starship** | Sidewalk-only. Does **not** use passenger elevators. *"if you could, please press the button"*; robots *"wait away from the push button… so they're never in the way"* |

**Conclusion: no cross-vendor robot-elevator standard exists. The industry moved
*away from* button actuators *toward* sanctioned APIs.** Starship's human-asks-a-human
model is the closest analogue to SHANA's situation.

---

## 5. Mapping and positioning findings

| Approach | Finding | Label |
|---|---|---|
| Nav2 `map_server` + `LoadMap` + AMCL `first_map_only: false` | Mature, standard, per-floor map switching | ✅ |
| SLAM Toolbox | `mapping` / `localization` / lifelong modes; pose-graph serialization; KD-Tree relocalisation | ✅ |
| "Multi-floor" in SLAM Toolbox | ⚠️ The phrase is **not used** in the ROS 2 README. It is one pose-graph per floor + an **external floor selector** — the same dependency | ✅ |
| `VoxelMapCompressed_` | Exists in the SDK but is a **local, robot-relative** map for `slam_operate`, **not** a persistent multi-floor building model | ✅ |
| **Quadruped multi-floor navigation** | ❓ **NOT FOUND.** No dedicated body of work. You are applying wheeled-robot patterns to a legged platform | ✅ negative |
| ArUco floor markers | ⛔ Recommended. Also the **same technology** the charging pile uses | 🧪 |
| BLE beacon per floor | 🧪 Published accuracy 0.9877–1.0, ~5 s delay, beacon ID = floor number | 🧪 |
| UWB | ❓ **No sub-metre accuracy requirement has been identified.** Overkill | ❓ |
| LoRa RSSI for positioning | ⛔ **Must not be used for positioning** | ✅ |

---

## 6. Unresolved issues

| # | Issue | Why it cannot be resolved by research |
|---|---|---|
| **N1** | **Actual stair riser heights** | Only a tape measure |
| **N2** | **Actual Wi-Fi coverage per floor/block** | Only a site survey |
| **N3** | **Installed Go2 options** (dock, depth cam, 15,000 mAh?) | Only physical inspection |
| **N4** | **Actual lift car/door/threshold dimensions** | Only physical inspection |
| **N5** | **Which Johnson entity and which lift standard** | Only the nameplate and AMC contract |
| **N6** | **EyeRIS retrofittability and robot-client provisioning** | Only Johnson in writing |
| **N7** | **Whether a school needs possession authorisation** under the 2026 Rules | Only DoT WPC in writing |
| **N8** | **Quartic behaviour of `ClassicWalk` on stairs** | Only a supervised test |
| **N9** | **Whether teachers will actually use the system** | Only asking them |
| **N10** | **Whether the school will permit any installation** | Only asking them |

---

## 7. Recommendations

1. **Stage the deployment.** Single floor first. Each stage must re-earn its own
   permission.
2. **Measure before designing.** Riser heights, Wi-Fi coverage, and installed
   hardware options — all are cheap and all are currently unknown.
3. **Do not plan on stairs.** At 150–170 mm risers the margin is 0–10 mm and the
   manufacturer forbids the terrain. If stairs are needed for reach, escalate to
   the school now rather than discovering it at Phase 4.
4. **Treat the lift as human-operated.** Ask Johnson the EyeRIS questions — it is
   the cheapest possible win and the only sanctioned path. If the answer is no, the
   escort protocol is permanent, which is what every comparable operator does.
5. **Add a floor-identity source before any multi-floor work.** ArUco per floor,
   plus mandatory human confirmation.
6. **Keep the robot on Ethernet.** It is the only manufacturer-supported path, and
   it insulates the robot from WLAN failures.
7. **Defer LoRa.** Build Wi-Fi properly. Add the heartbeat only for a named
   coverage gap.
8. **Keep the LLM strictly behind the validator**, and make the demonstration work
   with the LLM switched off.
9. **Make relocation events unmissable in the console.** This is the most likely
   way the robot ends up somewhere it believes it is not.
10. **Plan for human recovery.** There is no autonomous dock. Design for it rather
    than discovering it.

---

## 8. What would change these conclusions

| Finding | Effect |
|---|---|
| Stair risers ≤150 mm **and** a supervised unloaded test passes cleanly, including forward descent | Stairs become a possible fallback transit — still supervised |
| **Risers >170 mm anywhere on any required route** | Stairs are permanently out. Elevation is elevator-only, therefore escort-only |
| **Johnson confirms EyeRIS retrofit + robot provisioning + passive QR** | Stage 3 gains a sanctioned autonomous-ish path |
| **School permits and funds APs on every floor** | Wi-Fi mesh becomes viable; LoRa likely unnecessary |
| **No room lacks Wi-Fi** | LoRa has no justification. Do not build it |
| **School refuses all installation** | Floor ID and coverage both become blockers. Multi-floor is likely impossible |
| **School has no budget for the expansion dock** | Navigation route B (self-built) is mandatory — which was already the recommendation |

---

## 9. Sources

See `source_register.md` for the full list with URLs, types, dates accessed, and claims
supported. The detailed India RF research, including the gazette citations and the
multi-floor LoRa link-budget study, is in
`appendix_india_rf_research.md`.

---

## 10. Method and limitations

- **No robot was operated, connected to, commanded, or even physically inspected.**
- **No packages were installed. No network was probed. No hardware was configured.**
- `support.unitree.com` is WAF-blocked to direct fetch (HTTP 567); content was
  retrieved through a text-render proxy. URLs cited are origin URLs.
- `api.github.com` rate-limits unauthenticated requests; file contents came from
  `raw.githubusercontent.com`.
- **All findings are static analysis and documentation review. No runtime behaviour
  was tested.**
- Where this document says ❓ UNKNOWN, that is a genuine gap, not an omission. The
  questionnaire in `questions.md` is designed to close every one of them.