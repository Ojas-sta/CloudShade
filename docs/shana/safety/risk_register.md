# SHANA — Risk Register

**Phase 0 · 2026-10-10**

Risks to the SHANA project across technical, operational, network, elevator,
battery, regulatory, and student-safety domains. Every risk has a mitigation, an
owner, and a status. Outstanding approvals are tracked explicitly.

**Risk rating:** `L` likelihood × `S` severity, both 1–5.
🔴 High (15–25) · 🟠 Medium (8–14) · 🟡 Low (1–7)

**Status:** `MITIGATION PLANNED` · `OPEN — APPROVAL REQUIRED` ·
`OPEN — ANSWER REQUIRED` · `MONITOR`

---

## 1. Executive risk summary

| # | Risk | Domain | Rating | Status |
|---|---|---|---|---|
| R-01 | Robot cannot clear the school's stair risers | Technical | 🔴 20 | OPEN — ANSWER REQUIRED |
| R-02 | Lift integration is legally impossible | Elevator / Regulatory | 🔴 20 | MITIGATION PLANNED |
| R-03 | Robot localises into the wrong floor's map | Technical | 🔴 20 | MITIGATION PLANNED |
| R-04 | No school Wi-Fi / no Ethernet segment approved | Network | 🔴 20 | OPEN — ANSWER REQUIRED |
| R-05 | Student injury from a 15 kg machine with 12 moving legs | Safety | 🔴 20 | OPEN — APPROVAL REQUIRED |
| R-06 | Secondary development not authorised | Governance | 🔴 20 | OPEN — ANSWER REQUIRED |
| R-07 | Robot disabled mid-mission with no watchdog | Technical | 🔴 16 | MITIGATION PLANNED |
| R-08 | Battery thermal event | Battery | 🔴 16 | OPEN — APPROVAL REQUIRED |
| R-09 | Robot and human trapped in a stalled lift | Elevator / Safety | 🔴 20 | MITIGATION PLANNED |
| R-10 | School rejects the project scope | Programme | 🟠 12 | OPEN — ANSWER REQUIRED |
| R-11 | Stair descent failure | Technical | 🔴 16 | MITIGATION PLANNED |
| R-12 | Wi-Fi coverage gap in basement or sports hall | Network | 🟠 9 | OPEN — ANSWER REQUIRED |
| R-13 | Floor identification hardware refused | Technical | 🔴 15 | OPEN — APPROVAL REQUIRED |
| R-14 | Payload instability or drop | Operational | 🟠 10 | MITIGATION PLANNED |
| R-15 | Regulatory non-compliance with 2026 RF rules | Regulatory | 🟠 9 | OPEN — APPROVAL REQUIRED |
| R-16 | Stair manual handling invalidates pose | Technical | 🟠 12 | MITIGATION PLANNED |
| R-17 | Battery below 10 % stranding the robot | Battery | 🟠 12 | MITIGATION PLANNED |
| R-18 | Camera frame rate insufficient for perception | Technical | 🟡 6 | OPEN — ANSWER REQUIRED |
| R-19 | Vendor `slam_operate` unusable (no dock, no YSN) | Technical | 🟠 8 | MITIGATION PLANNED |
| R-20 | Teachers abandon the system | Adoption | 🟠 8 | OPEN — ANSWER REQUIRED |
| R-21 | Building layout changes invalidate the map | Operational | 🟠 8 | MONITOR |
| R-22 | Project scope grows faster than capability | Programme | 🟠 9 | MITIGATION PLANNED |

---

## 2. Technical risks

### R-01 🔴 20 — Robot cannot clear the school's stair risers
**OPEN — ANSWER REQUIRED** · Owner: Facilities (FAC-01)

The Go2's documented maximum step height is **160 mm**. The manual states it is
*"forbidden to use the robot… where the terrain is complicated… steps higher than
16cm"*, and *"not currently recommended that robot walk stairs higher than 16cm…
otherwise it is likely to trip over the foot."*

Typical Indian school risers are 150–180 mm. A 150 mm riser leaves 10 mm of
tolerance. **A 170 mm riser is outside the manufacturer envelope before payload,
before supervision, and before wear.**

**Mitigation:** measure every riser on every candidate route **before** designing
any cross-floor capability. If any exceed 160 mm, stairs are permanently out of
scope and cross-floor capability is elevator-only. **This is a cheap measurement
that can end the question in an afternoon.**

**Residual risk after mitigation:** accepted as a scope constraint.

---

### R-03 🔴 20 — Robot localises into the wrong floor's map
**MITIGATION PLANNED** · Owner: Engineering

**This is the most dangerous failure mode in the entire system, because it is
silent.** Every message field across the Go2 platform was enumerated: there is
**no altitude, no pressure, no barometer, and no floor field anywhere.** `position[3]`
is dead-reckoned odometry that drifts and resets. `body_height` is the commanded
stance height above ground, not altitude. `HeightMap.origin` is `float32[2]` —
**XY only, no Z datum.**

If a human carries the robot up a flight of stairs, scan-matching will succeed
against **both** floors' maps. The robot will confidently plan against floor 1's
geometry while standing on floor 3. Nothing crashes. Nothing logs an error.

**Mitigation — three independent layers, all mandatory:**

1. **Unique ArUco/AprilTag marker per floor**, marker ID encoding the floor, read
   by the head camera. *Also the same technology the charging pile uses — so this
   tests hardware already proven on the robot.*
2. **Mandatory human confirmation event on every floor transition.**
3. **An explicit `RELOCATION` event** whenever anyone manually handles the robot —
   invalidates the pose, forces re-localisation before any new task is accepted.
   The console must make this unmissable.

⛔ **Never use LoRa or BLE signal strength as the floor-identity source.** A beacon
*ID* is a valid floor signal. Signal *strength* is not a positioning input.

**Residual risk:** low, provided all three layers are implemented. **High** if any
one is skipped.

---

### R-07 🔴 16 — Robot disabled mid-mission with no watchdog
**MITIGATION PLANNED** · Owner: Engineering

The server holds the state machine and the safety supervisor. If the server
crashes or the network drops, **there is no software watchdog** — it died with the
server.

Independent field reports confirm the Go2 has *"powered off completely in the
middle of its movement and collapsed to the floor"*.

**Mitigation:**
- Physical link failure → robot's built-in obstacle avoidance (on by default)
- Software failure → physical remote controller, authority tested before every run
- A **named human spotter at every stage, without exception**
- Safe-state design: the robot holds position; it never wanders
- Pre-armed safe stop: no heartbeat → `StopMove()` then hold

⚠️ **The architecture deliberately keeps the state machine off the robot** so that
if the server dies, the robot stops. That is the correct behaviour — but it means
the human spotter is load-bearing for safety, not merely convenient.

---

### R-11 🔴 16 — Stair descent failure
**MITIGATION PLANNED** · Owner: Engineering

Unitree provides two stair modes precisely because backing down is the fallback,
and **every independent report describes descent as the worse case**: *"can walk
upstairs but has an issue walking downstairs"*; *"loses balance a little bit on
both stairs but is able to recover"*.

**Aggravating factors:** the 160 mm figure is a **no-load** figure and no derated
figure is published; descent has impact loading and lower traction at ground
contact.

**Mitigation:** stairs are **descended unloaded only**, never with payload. The
supervised test ladder advances stage by stage with a hold point at each stage.
**Forward descent is the gating capability — if it fails, the project does not
proceed.** *(`questions.md` TEC-06)*

---

### R-16 🟠 12 — Manual handling invalidates the robot's pose
**MITIGATION PLANNED** · Owner: Engineering

Any human carrying, repositioning, or re-orienting the robot destroys its
odometry. If this is not recorded, the robot believes it is somewhere it is not.

**Mitigation:** the `RELOCATION` event in R-03. The console must display
`pose_valid` prominently and **must require an explicit action before accepting a
task after any manual handling.** Additionally, `last_pose_age` must be shown
alongside position — a 40-minute-old position on a screen next to a physical robot
invites someone to look in the wrong place.

---

### R-18 🟡 6 — Camera frame rate insufficient
**OPEN — ANSWER REQUIRED** · Owner: Engineering (TEC-02)

Unitree's documentation contradicts itself: `about_Go2` states 720p and 1080p at
**15 fps**; the official FAQ states 1080p at **30 fps** and 720p at **60 fps**.
At 15 fps, real-time detection is significantly limited.

**Mitigation:** measure before designing any perception-dependent feature
(TEC-02). Do not design around either figure until measured. Note that the
**ArUco floor-ID requirement may be satisfiable at 15 fps** — markers are large,
static, and high-contrast, unlike general object detection.

---

### R-19 🟠 8 — Vendor `slam_operate` unusable
**MITIGATION PLANNED** · Owner: Engineering

The documented navigation service requires **EDU + expansion dock + separately
purchased MID-360 or XT16 LiDAR**, plus a Unitree-issued **YSN**. It cannot run
inside a ROS graph. The documentation page body has partly failed to render.

**Mitigation:** **Route B (self-built Nav2 on the built-in L2) is already the
recommended primary path** and needs no purchase. Route A is contingency only. If
the dock is not present, this risk is eliminated rather than mitigated.

---

## 3. Elevator risks

### R-02 🔴 20 — Lift integration is legally impossible
**MITIGATION PLANNED** · Owner: Principal + service provider

**No sanctioned third-party or robot call interface exists for Johnson lifts.**
Johnson's WATCH IoT system is monitoring-only, lift → vendor, with no inbound
command path. No Johnson public API, SDK, or developer portal was found — a
high-confidence negative.

Statutory position, verbatim from the **Maharashtra Lifts, Escalators and
Moving Walks Act, 2017, Section 10**:

> *"No additions or alterations… shall be made to any lift… installation **except
> with the prior permission in writing of an Electrical Inspector (Lifts)**."*

**Mitigation — a three-way decision:**
1. **Ask Johnson about EyeRIS** (ELV-06/07). A retrofit-available, **passive** QR
   with robot-client provisioning would be a sanctioned, no-modification path. **Ask
   this first — it is one email.**
2. **If yes:** use it, with a companion device. Written approval still required.
3. **If no:** human escort becomes permanent. This is what every comparable
   operator does.

⛔ **Never attach a servo, actuator, or any device to the COP or LOP.** This is the
academic research trick and it is prohibited here. Otis — the world's largest
elevator company — abandoned button-actuator integration in favour of sanctioned
APIs. The industry moved *away* from this approach.

---

### R-09 🔴 20 — Robot and human trapped in a stalled lift
**MITIGATION PLANNED** · Owner: Facilities + service provider

**This is the *Otis v. Rashmi Handa* scenario exactly** — Supreme Court of India,
29 July 2026. An officer died when MES personnel manually released the brake
during a rescue from a lift stalled between floors.

**The robot's battery is dead in ≤4 hours. A mechanic's rescue will typically take
far longer. The robot must be assumed dead by the time help arrives.**

**Mitigation — the escort protocol is the defence, not a technical control:**
- A **named adult escort rides with the robot on every single trip**. No exceptions.
- Human verifies the car is **empty** before the robot enters. If occupied, abort.
- Human presses both the hall call and the destination.
- Human stays in the car throughout.
- Robot holds position if the human leaves the car — **the robot never attempts to
  exit unattended.**
- Documented abort-and-restrict-to-single-floor if the lift is out of service.
- Phase I fire recall behaviour confirmed in advance (ELV-09).

⚠️ **The robot physically fits easily** — 310 mm vs a 900 mm door, 15 kg vs a
544 kg rated load. **Fit is not the risk. The stall is.**

---

### R-14 🟠 10 — Payload instability or drop
**MITIGATION PLANNED** · Owner: Engineering

No published Go2 load-retention data exists. Community field reports note that
above ~3 kg or off-centre, gait visibly changes. Thresholds and door frames cause
vertical shocks.

**Mitigation:**
- **Payload ≤2 kg initially** — well inside the usable envelope
- Ratcheting cradle **plus** a secondary retention strap. Two independent restraints.
- Centre of mass within the spine hardpoint quadrilateral; measure and mark
- **Tilt-test the rig to 45° before the robot touches it**
- ⛔ **Never carry glass, liquid, chemicals, reagents, sharps, heated apparatus, or
  anything fragile or unsealed**
- `AutoRecover` **OFF** during payload missions; ⛔ never `RecoveryStand()` with
  payload — the documentation says it *"recovers regardless of whether it has
  fallen"*, which is violent with 8 kg

---

## 4. Network risks

### R-04 🔴 20 — No Wi-Fi, no approved Ethernet segment
**OPEN — ANSWER REQUIRED** · Owner: IT (NET-01, NET-02)

The robot link is **Cyclone DDS over wired Ethernet**. Unitree documents
development as wired-only; CycloneDDS SPDP discovery is unreliable over Wi-Fi.

⚠️ **Architectural consequence worth stating plainly: because the robot link is
Ethernet, a WLAN failure does not stop the robot.** A design that put the robot on
Wi-Fi would be non-conformant with the manufacturer and would stop the robot every
time the WLAN failed.

**Mitigation — preferred option first:** a **dedicated USB-Ethernet NIC on the AI
Lab server, directly cabled to the robot.** This requires **no school network
change at all** — the robot is simply a directly-connected Ethernet peer. Ask IT
only if that is impossible, then fall back to a VLAN.

**The robot must never be placed on the smartboard LAN.** It requires the
`192.168.123.0/24` subnet per Unitree documentation.

---

### R-12 🟠 9 — Wi-Fi coverage gap in basement or sports hall
**OPEN — ANSWER REQUIRED** · Owner: IT (NET-05)

The C Block basement contains sports facilities and laboratories. Basements and
sports halls are the classic coverage gap.

**Mitigation:** NET-05 asks IT to **name any room where Wi-Fi does not reach.**
That single answer decides whether LoRa is justified at all.

⚠️ **Research conclusion:** with one working AP per floor, **Wi-Fi beats LoRa for
coverage in a 4-storey building.** A peer-reviewed 16-storey study found LoRa
*still lost the link across floors*. LoRa buys **independence from the WLAN**, not
range. Anyone claiming otherwise is selling something.

---

### R-15 🟠 9 — Regulatory non-compliance with 2026 RF rules
**OPEN — APPROVAL REQUIRED** · Owner: Principal + project lead (TEC-10)

⚠️ **The regime changed two months before this study.** The *Telecommunications
(Radio Equipment Possession Authorisation) Rules, 2026* (G.S.R. 592(E), 8 July
2026; applications opened 06.08.2026) now require authorisation **to possess**
radio equipment under Telecom Act 2023 §3(1)(c). **Almost every secondary source
is stale.**

Specific compliance traps if LoRa is ever built:
- India's licence-exempt band is **865–868 MHz** — the default EU868 channel
  (868.1 MHz) is **outside it**
- Power is limited in **e.r.p., not EIRP** (500 mW e.r.p. = 29.15 dBm EIRP)
- Duty cycle **2.5 % end device / 10 % gateway**
- A **T-Beam at full 32 dBm is illegal** (~1531 mW e.r.p. vs a 500 mW ceiling)

**Mitigation:** **defer LoRa entirely until NET-05 identifies a real coverage
gap.** Then email DoT WPC for written clarification (TEC-10) — one email, low
cost, high value. Getting this wrong on a 2026-vintage rule is expensive to unwind.

---

## 5. Battery risks

### R-08 🔴 16 — Battery thermal event
**OPEN — APPROVAL REQUIRED** · Owner: Facilities (FAC-08, FAC-09)

Manufacturer limits, verbatim: **discharge −20 to 60 °C, charge 0 to 55 °C.**
*"Exceeding these temperature limits during charging or discharging may cause the
battery to ignite or even explode."*

Additional rules: do not charge immediately after running; ideal storage 22–28 °C,
**never above 40 °C**; **stop use below 10 %**; never charge in
electromagnetic/electrostatic environments; scrap any bulging or damaged pack.

**Mitigation:**
- Charge **supervised**, on a **non-flammable surface**, **overnight only in an
  occupied building — never unattended overnight**
- Cool to room temperature before charging
- Store between 20–40 % for long periods
- Written battery quarantine procedure must exist **before the first pack is
  charged** (FAC-09)
- ⛔ No charging below 5 °C or above 40 °C ambient

---

### R-17 🟠 12 — Battery below 10 % stranding the robot
**MITIGATION PLANNED** · Owner: Engineering

✅ Verified behaviour: **at low battery the robot crouches within 10 minutes.** It
does not collapse instantly, and no controlled safe-landing is documented.

Three consequences:
1. **A crouch on a staircase is a fall.** On stairs, the 10-minute warning is the
   entire reaction budget.
2. **20 % is not a safe abort threshold.** Use **30 %**.
3. **Never rely on the robot's own low-battery logic.**

**Mitigation:**
- Mission abort at **30 % SOC**
- Battery read from **`rt/lowstate` → `LowState.bms_state`** — verified to carry
  `soc`, signed `current`, `cell_vol[15]`, `bq_ntc`, `status`
- Coulomb counting from signed current for a defensible reserve estimate
- **Planning figure: 90 minutes of active mission per charge** (manufacturer
  figures conflict — 1–5 h across documents)
- Recovery is **human-assisted by design** — there is no autonomous return-to-dock
- Checkpoint location always known and communicated on abort

---

## 6. Student safety risks

### R-05 🔴 20 — Student injury
**OPEN — APPROVAL REQUIRED** · Owner: Principal + Safety Officer

Unitree's manual, verbatim:
- *"Hand may be pinched at the motion joints, such as at the knee joint."*
- *"Keep it under control in the users' view… keep the robot maintain a safe
  distance of at least 2 meters from obstacles, complex ground, crowds, water."*
- ***"This product is not a toy and is not intended for use by persons under the
  age of 14. Keep out of reach of children and be careful when operating in the
  presence of children."***

⚠️ **This is a device for supervised operation in a school with children under 14.
Unsupervised operation is outside anything the manufacturer describes.**

**Mitigation — non-negotiable:**

1. **Named adult spotter at every stage.** Unilateral stop authority, no
   justification required.
2. **Speed capped at 0.4 m/s** — slower than a walking child.
3. **Physically marked exclusion zone, 1.5 m either side**, enforced at all times.
4. **No testing in occupied corridors.** Out-of-hours, cleared, witnessed.
5. **RC authority tested immediately before every run** — never assumed.
6. **Emergency stop arrangement agreed in writing** (FAC-10) before the first run.
7. **The teacher's acceptance threshold may be stricter than the technical one** —
   ask TCH-06 and honour it.
8. **Battery charging outside student areas.**
9. No food, drink, or personal items on the robot.

⚠️ **Weigh the whole-school benefit before proceeding.** If a teacher's answer to
TCH-06 is "no", the honest response is to redesign — not to persuade.

---

## 7. Governance and programme risks

### R-06 🔴 20 — Secondary development not authorised
**OPEN — ANSWER REQUIRED** · Owner: Principal + robot owner (ROB-05)

✅ VERIFIED: *"Only EDU or above versions can be developed, other versions such as
PRO and AIR versions do not open the secondary development interface."*

But **technical capability and permission are different questions.** The school
may own an EDU and still not have authorised development access.

**Mitigation:** obtain written authorisation **before** any connection. If it
cannot be obtained, the project cannot proceed in its intended form. **Do not
assume permission from ownership.**

---

### R-10 🟠 12 — School rejects the scope
**OPEN — ANSWER REQUIRED** · Owner: Principal (ADM-07, ADM-08)

**This is not a failure.** If the school decides cross-floor delivery is not
needed (TCH-05) or a robot among students is not acceptable (TCH-06), SHANA
becomes a **single-floor system** — smaller, safer, more likely to work.

⚠️ **Five of the twelve worst-case answers in `open_questions_register.md` §5 end
with a smaller project.** That is the correct outcome for a feasibility phase.

---

### R-20 🟠 8 — Teachers abandon the system
**OPEN — ANSWER REQUIRED** · Owner: Teachers (TCH-01, TCH-03)

A robot that is impressive once and unreliable daily is worse than no robot.

**Mitigation:** define success with the school (ADM-07) and with teachers (TCH-01)
**before** building. Measure real reliability, not demo reliability. Make the
dashboard the product, not the robot — the robot is the expensive part, the
dashboard is what teachers touch.

---

### R-22 🟠 9 — Scope grows faster than capability
**MITIGATION PLANNED** · Owner: Project lead

The brief lists a large aspiration: three blocks, four floors, elevators, stairs,
LoRa, ESP32, UWB, BLE, natural language, scheduling, mapping, battery management.

**Mitigation: the staged model in `architecture.md` §2.** Each stage is
independently useful and independently demonstrable. **No stage depends on an
unverified capability of a later one.** If Stage 2 cannot be earned, Stages 0–1
still stand.

---

### R-21 🟠 8 — Building layout changes invalidate the map
**MONITOR** · Owner: Engineering + Facilities

Furniture moves, rooms are reassigned, a wall is added.

**Mitigation:** every destination carries `verified_by` and `verified_pose_date`.
Poses expire and must be re-verified. Facilities notifies the team of layout
changes (FAC-06). `pose_valid` is a first-class field.

---

## 8. Outstanding approvals — consolidated

**None of these may be assumed.**

| # | Approval | From | Status | Blocks |
|---|---|---|---|---|
| A-01 | **Secondary development authorisation** | Principal / robot owner | ⛔ NOT REQUESTED | Everything |
| A-02 | **Dedicated Ethernet segment** `192.168.123.0/24` | IT | ⛔ NOT REQUESTED | TEC-01 |
| A-03 | **Server room allocation** | Principal | ⛔ NOT REQUESTED | All software |
| A-04 | **Access points and wall markers** | Principal + IT | ⛔ NOT REQUESTED | Floor ID |
| A-05 | **Robot operation policy in occupied corridors** | Principal + Safety Officer | ⛔ NOT REQUESTED | All testing |
| A-06 | **Insurance cover** | Insurer | ⛔ NOT REQUESTED | Public demo |
| A-07 | **Emergency stop procedure** | Facilities + Principal | ⛔ NOT REQUESTED | First powered run |
| A-08 | **Battery handling procedure** | Facilities | ⛔ NOT REQUESTED | First charge |
| A-09 | **Lift escort trial permission** | Lift service provider | ⛔ NOT REQUESTED | Stage 2 |
| A-10 | **EyeRIS robot provisioning** | Johnson Lifts | ⛔ NOT REQUESTED | Stage 3 |
| A-11 | **RF possession-authorisation clarification** | DoT WPC | ⛔ NOT REQUESTED | Any LoRa |
| A-12 | **Any lift modification** | Electrical Inspector (Lifts) | ⛔ **NOT PERMITTED** — do not request | — |

---

## 9. Actions that must never be taken

Listed so that under time pressure there is a pre-decided answer.

| # | Action | Why |
|---|---|---|
| 1 | ⛔ Attach anything to the lift COP or LOP | Prohibited without Electrical Inspector written permission. Warranty and AMC void |
| 2 | ⛔ Tap, splice, or connect to the safety circuit | *Otis v. Rashmi Handa* — Supreme Court fatality |
| 3 | ⛔ Bypass, block, or disable door sensors or interlocks | Life-safety devices |
| 4 | ⛔ Short-circuit the car-top switch, overspeed governor, buffers, or ARD | Absolutely prohibited |
| 5 | ⛔ Enter the shaft, pit, or car top | Absolutely prohibited |
| 6 | ⛔ Let the robot ride an elevator **alone** | Battery dead in ≤4 h; mechanical rescue needed |
| 7 | ⛔ Operate unsupervised | Manufacturer: not a toy, not for under-14s, requires control in the user's view |
| 8 | ⛔ Test in an occupied corridor | No exclusion zone, no spotter |
| 9 | ⛔ Power off or emergency-brake in a way that drops the robot | Fall risk |
| 10 | ⛔ Use LoRa or BLE RSSI for positioning | Must not drive navigation or floor identity |
| 11 | ⛔ Let an LLM reach the robot | Design principle; safety |
| 12 | ⛔ Give teachers motor-control access | Design principle; safety |
| 13 | ⛔ Expose the server to any public network | Design principle; privacy and safety |
| 14 | ⛔ Accept a delivery as complete without explicit recipient confirmation | Design principle |

---

## 10. How risks retire

| Risk group | Retires when |
|---|---|
| R-01, R-11 (stairs) | FAC-01 measured **and** TEC-06 supervised test passes — or the scope drops stairs |
| R-02, R-09 (lift) | ELV-06/07 answered in writing **and** ELV-10 escort consent given |
| R-03 (floor ID) | TEC-08 ArUco detection measured **and** relocation handshake implemented |
| R-04, R-12 (network) | NET-02 approved **and** TEC-01 establishes a stable connection |
| R-05 (safety) | ADM-05 policy issued **and** FAC-10 procedure signed **and** spotter trained |
| R-06 (authorisation) | ROB-05 written authorisation in hand |
| R-08 (battery) | FAC-08/09 procedures documented |
| R-15 (regulatory) | DoT written clarification received — **or** LoRa dropped from scope |
| R-22 (scope) | Stage 1 signed off by the school |

⚠️ **No risk in this register retires through further research.** Each requires a
measurement, a decision, or a signature. That is the correct state for Phase 0.