# SHANA — Decision Log

**Phase 0 · 2026-10-10**

Records decisions already made, decisions still open, and the evidence each open
decision requires. This is a living document — append, do not rewrite.

**Decision states:** `DECIDED` · `PROVISIONAL` (a default that applies unless
contradicted) · `OPEN` (cannot be decided yet) · `REJECTED` (considered and ruled
out, with reason)

---

## 1. Decisions already made

### D-001 — Platform: Unitree Go2 EDU
**State: DECIDED** · Date: prior to this phase

The school owns a Go2 EDU. This was the conclusion of the preceding platform
selection study, which scored Go2 EDU 63.5/100 against G1 EDU 40.5/100 on
feasibility, autonomy, reliability, safety, and exhibition value.

**Evidence:** `research/` deliverables. Key findings: the G1's only genuine
advantage is manipulation, independently measured at 20–30% pick-and-place success
(arXiv 2610.08119) with 13.5 N measured unbraced end-effector force
(arXiv 2609.25486); the Go2 has documented autonomous navigation interfaces the
G1 lacks; and the Go2's mandatory protective-frame requirement applies to the G1
but not the Go2.

**Consequence:** all Phase 0 work targets the Go2 EDU.

---

### D-002 — This phase produces documentation only
**State: DECIDED**

No robot operated, connected to, or commanded. No packages installed. No network
probed. No hardware configured. No application code written.

**Reason:** the brief mandates research-only execution, and several architecture
decisions cannot be made without school input that only the school can give.

---

### D-003 — The robot-to-server link is wired Ethernet over Cyclone DDS
**State: DECIDED**

**Evidence:** ✅ VERIFIED — Unitree developer FAQ: *"No, Go2 only supports wired
connection for development."* Transport is Eclipse Cyclone DDS, domain 0, explicit
NIC argument. Robot `192.168.123.161`, host `192.168.123.222` on
`192.168.123.0/24`.

**Consequence:** a design putting the robot on Wi-Fi would be non-conformant with
the manufacturer and would fail whenever the WLAN failed. This also means **a
WLAN outage does not stop the robot** — which materially reduces the value of a
LoRa backup for the robot link specifically.

---

### D-004 — The application server runs locally in the AI Lab
**State: PROVISIONAL** — pending ADM-02, LAB-01, LAB-02

**Reasoning:** no confirmed NAS or local application server exists. The AI Lab is
the proposed primary base (CP-C3), staff are co-located, and it is nearest to the
most likely destinations.

**What would change it:** ADM-02 refuses the room, or LAB-01 shows no suitable
machine with a spare Ethernet port.

**Consequence:** no internet dependency. No public exposure. Server binds to the
LAN interface only.

---

### D-005 — LLM sits strictly behind a deterministic validator
**State: DECIDED**

**Reasoning:** brief design principle, and the safety argument is independent of
the model's capability.

**Rules:** the model emits JSON to a pipe and has no network route to the robot;
the validator is a pure function with no I/O; unknown entities are rejected, never
guessed; every model-derived task requires human confirmation; **the demonstration
must work with the LLM layer switched off.**

**Consequence:** the LLM can fail without failing the demonstration.

---

### D-006 — Teachers never receive motor-control access
**State: DECIDED**

**Reasoning:** brief design principle. Teacher role creates requests and confirms
receipts. Robot-admin role cancels and intervenes. **Neither can command motion.**
The safety supervisor is separate and can veto.

---

### D-007 — Delivery completion requires explicit recipient confirmation
**State: DECIDED**

**Reasoning:** brief design principle. Arrival at a destination is **not**
completion. This also removes a class of false-positive reports from the metrics.

---

### D-008 — Loss of communications produces a defined safe behaviour, never a best-effort continuation
**State: DECIDED**

**Behaviour:** no heartbeat → `StopMove()` → hold position → server marks the robot
`ORPHANED` → alarm → no new tasks accepted. The robot never wanders.

---

### D-009 — Route: self-built navigation (Route B) as primary
**State: PROVISIONAL** — pending ROB-01

**Reasoning:** the vendor `slam_operate` service requires EDU + expansion dock +
separately purchased MID-360/XT16 plus a Unitree-issued YSN, and cannot run inside
a ROS graph. The built-in L2 already publishes point clouds on DDS in a
ROS 2 `PointCloud2`-compatible type, so Route B needs **no purchase**.

**What would change it:** the school already owns an expansion dock. Even then,
Route B remains preferable because it avoids the YSN dependency and keeps the
engineering ours.

---

### D-010 — Simulation-first development
**State: DECIDED**

**Reasoning:** `unitree_mujoco` (BSD-3) simulates the Go2 **on the same DDS
channels as the hardware**, CPU-only, no GPU. The entire software stack can be
built, debugged, and demonstrated before the robot is switched on.

**Consequence:** TEC-04 requires no school answer and can begin immediately.
**This is the highest-value action available in Phase 0 and it is not blocked.**

---

### D-011 — Battery telemetry is read from `rt/lowstate`, not `rt/sportmodestate`
**State: DECIDED**

**Evidence:** ✅ VERIFIED — the complete `SportModeState.msg` field list contains
**no battery fields whatsoever**. Battery is on `LowState.bms_state`, which
provides `soc` (%), signed `current` (mA), `cell_vol[15]`, `bq_ntc`, `status`,
`cycle`.

**Consequence:** any design or monitoring tool reading battery from the sport topic
is wrong and must be corrected.

---

### D-012 — Mission abort at 30 % SOC
**State: DECIDED**

**Evidence:** ✅ VERIFIED — *"Low battery warning, will automatically crouch down
within 10 minutes."*

**Reasoning:** a crouch on a staircase is a fall, so on stairs the 10-minute
warning is the entire reaction budget. 20 % is not a safe threshold. 30 % leaves
margin for the return leg plus the countdown.

**Consequence:** planning figure **90 minutes of active mission per charge**.
Manufacturer figures conflict (1–5 h across documents), so our own measurement
(TEC-05) supersedes them once available.

---

### D-013 — `EconomicGait()` is the default delivery gait
**State: DECIDED**

**Evidence:** ✅ VERIFIED — *"The robot's height is higher in this mode, and the
rate of power consumption is slower."*

**Companion rule:** hold **maximum** body height. The manual states that *"lower
body height and more knee bends place a greater burden on the motors and therefore
increase power consumption significantly earlier."* ⛔ Avoid `WalkUpright` and
`CrossStep` — both documented as motor-overheating gaits.

---

### D-014 — Auto-recovery disabled during payload missions
**State: DECIDED**

**Evidence:** ✅ VERIFIED — `RecoveryStand()` *"regardless of whether it is
fallen or not, it will recover to standing."* With 8 kg that is violent.

**Rules:** `AutoRecover` **OFF** with payload; ⛔ never `RecoveryStand()` with
payload; on anomaly use `StopMove()` → `StandDown()` → human recovery; reserve
`Damp()` strictly for imminent human-injury situations, accepting a payload drop.

---

### D-015 — No elevator modification, ever
**State: DECIDED** — and treated as an absolute constraint

**Evidence:** Maharashtra Act 2017 s.10, Delhi/Bombay Rules 6(v) and 6(xv), TN
Act s.15 — all require prior written permission of an Electrical Inspector (Lifts)
for any addition or alteration. *Otis v. Rashmi Handa*, Supreme Court of India,
29 July 2026.

**Reasoning:** even the "non-invasive" academic technique — a servo pressed over
the panel without touching wiring — is a **prohibited addition to the
installation**. Otis, the world's largest elevator company, abandoned button
actuators in favour of sanctioned APIs. The industry moved away from this approach.

---

### D-016 — Autonomous lift operation is not assumed
**State: DECIDED**

**Evidence:** no Johnson public API, SDK, or developer portal found — a
high-confidence negative. WATCH IoT is monitoring-only with no inbound command
path.

**Consequence:** the Stage 2 design is a **human escort protocol**, matching every
comparable commercial operator.

---

### D-017 — LoRa is deferred
**State: PROVISIONAL** — pending NET-05

**Reasoning:** with one working AP per floor, **Wi-Fi beats LoRa for coverage** in
a 4-storey building. A peer-reviewed 16-storey study found LoRa *still lost the
link across floors*. LoRa buys **independence from the WLAN**, not range — and the
robot link is Ethernet, so a WLAN outage does not stop the robot anyway.

**Decision rule:** build Wi-Fi properly first. Add the LoRa heartbeat only if
NET-05 names a specific room with no Wi-Fi coverage. **A ~350-byte heartbeat at
SF9 every 60 s is cheap insurance that will not change the architecture — which is
exactly why it is a good thing to add late.**

---

### D-018 — Four ArUco markers encode floor identity; the same pipeline may enable docking
**State: PROVISIONAL** — pending ADM-04, TEC-08

**Reasoning:** floor identification is **impossible with stock sensors** — verified
negative across every Go2 message field. But the **charging pile mechanism is
already ArUco-based**, so the marker-detection pipeline and camera stack are
proven on this robot. One investment, two capabilities.

**Rule:** ArUco per floor **plus** a **mandatory human confirmation on every floor
transition.** Markers make it work; human confirmation makes the residual risk
acceptable.

---

### D-019 — A manual relocation event invalidates the robot's pose
**State: DECIDED**

**Reasoning:** if a human carries the robot up a flight of stairs, scan-matching
will succeed against **both** floors' maps. The robot will confidently plan
against the wrong floor with **no error raised**. This is the most dangerous
failure mode in the system because it is silent.

**Rule:** any manual handling sets `pose_valid = false`; the robot must
re-localise before accepting any new task; the console must make entering a
relocation event unmissable; `last_pose_age` must be displayed alongside position.

---

### D-020 — The system is deployed in four stages, each independently useful
**State: DECIDED**

Stage 0 Observe → Stage 1 Single Floor → Stage 2 Vertical (human-assisted) →
Stage 3 Multi-Block (contingent).

**Reasoning:** no stage depends on an unverified capability of a later one. Each
stage must re-earn its own permission.

**Consequence:** if Stage 2 cannot be earned, Stages 0–1 still stand. **This is
the structural defence against scope risk R-22.**

---

## 2. Decisions still open

### D-101 — Are stairs in scope at all?
**State: OPEN**

**Blocks:** whether Stages 2/3 have any stair-based fallback.

**Evidence required:**
1. **FAC-01** — measured riser heights, mm, every staircase on every candidate route
2. **TEC-06** — supervised unloaded stair test, all stages including forward descent
3. **TEC-07** — whether a stair gait is reachable from the EDU SDK on the installed
   firmware

**Default if unanswered:** stairs are **out of scope**.

**Note:** the decision may be made for us. If any required riser exceeds 160 mm,
stairs are outside the manufacturer envelope before payload, before supervision,
and before wear — and no test is needed.

---

### D-102 — Is there any sanctioned lift integration path?
**State: OPEN**

**Blocks:** whether Stage 3 can operate without an escort on every single trip.

**Evidence required:** ELV-06 and ELV-07 answered **in writing by Johnson Lifts** —
EyeRIS retrofittability, whether the QR is passive or active, and whether a
non-human client can be provisioned. Plus ELV-05 for any other interface.

**Default if unanswered:** human escort is **permanent**.

**Consequence of the answer:** a passive, retrofittable QR with robot provisioning
would be the **single highest-value answer in the entire register** — a
manufacturer-sanctioned, no-modification call path.

---

### D-103 — What is the robot's installed configuration?
**State: OPEN**

**Blocks:** D-009 (navigation route), and any assumption about sensors.

**Evidence required:** ROB-01 variant, expansion dock present and which Orin, depth
camera present, foot force sensors; ROB-02 which battery; ROB-03 charging pile and
spares; ROB-04 firmware version.

**Default if unanswered:** assume EDU with **no** expansion dock (Route B
mandatory), 8,000 mAh battery, **no** depth camera.

⚠️ **The battery assumption is the most costly one** — it halves the endurance
budget. **Read the label before anything else.**

---

### D-104 — Is the robot link networkable?
**State: OPEN**

**Blocks:** everything technical.

**Evidence required:** NET-02 approval for a dedicated `192.168.123.0/24`
segment, or LAB-02 confirming a spare Ethernet port for a direct cable.

**Fallback if refused:** a direct USB-Ethernet cable from the server to the robot,
requiring **no school network change at all**.

**Cannot be waived:** the robot must never share a broadcast domain with the
smartboard LAN.

---

### D-105 — Where does the server run, and is it always on?
**State: OPEN** — follows D-004

**Evidence required:** ADM-02 room allocation; LAB-01 Linux host with ≥2 NICs,
24/7 or school-hours only; LAB-02 spare Ethernet port.

**Consequence if school-hours-only:** deliveries are constrained to school hours,
and the robot cannot be recovered outside them. This materially changes the
checkpoint design.

---

### D-106 — Which floors and destinations are actually in scope?
**State: OPEN**

**Blocks:** the entire map and route design.

**Evidence required:** TCH-05 (which floors teachers need), TCH-04 (destination
list), FAC-05 (B↔A route survey), FAC-06 (obstruction map), FAC-07 (checkpoint
confirmation).

**This question may shrink the project.** If teachers do not need cross-floor
delivery, Stages 2 and 3 drop out and the project becomes substantially more
likely to succeed.

---

### D-107 — Is any installation permitted at all?
**State: OPEN**

**Blocks:** floor identification (D-018) and any wireless coverage work.

**Evidence required:** ADM-04 for APs and markers; NET-04 for AP installation.

**Default if refused:** single-floor only. **Multi-floor becomes infeasible**
because floor identification has no in-sensor solution. A single-floor system can
still be a good project.

---

### D-108 — Is there a real Wi-Fi coverage gap?
**State: OPEN**

**Blocks:** whether LoRa is justified (D-017).

**Evidence required:** NET-05 — a named list of rooms where Wi-Fi does not reach.

**Default if unanswered:** assume no gap. **Do not build LoRa.**

---

### D-109 — Does a school need a 2026 radio possession authorisation?
**State: OPEN**

**Blocks:** any future radio-based feature.

**Evidence required:** TEC-10 — a written reply from the DoT WPC Helpdesk
regarding the *Telecommunications (Radio Equipment Possession Authorisation)
Rules, 2026* (G.S.R. 592(E), 8 July 2026).

**Reasoning:** Rule 10(c) or 10(h) probably exempts a school, but **no DoT
clarification was found and nearly all secondary sources are stale.** One email,
low cost, high value. Get it before ordering hardware, not after.

---

### D-110 — What does the school consider success?
**State: OPEN**

**Blocks:** which capabilities must work, and therefore what to measure.

**Evidence required:** ADM-07 (what success means), ADM-08 (daily use vs
demonstration), TCH-01 (the single most annoying task).

**Consequence:** a navigation demonstration and a teacher-time-saving system are
different products with different acceptance criteria. Building the wrong one is
a common and expensive failure.

---

## 3. Decisions explicitly rejected

| # | Rejected | Why |
|---|---|---|
| X-01 | Robot on Wi-Fi | Manufacturer: wired-only. SPDP unreliable on Wi-Fi. Would stop the robot on every WLAN failure |
| X-02 | LoRa as primary or control link | Non-deterministic latency, wrong data rate, duty-cycle limits. ⛔ Never a control loop |
| X-03 | LoRa or BLE RSSI for positioning | Signal strength must not drive navigation or floor identity |
| X-04 | UWB positioning | No sub-metre accuracy requirement has been identified. Overkill |
| X-05 | Robot on the smartboard LAN | Manufacturer specifies an isolated `192.168.123.0/24` subnet |
| X-06 | Autonomous elevator operation | No sanctioned interface exists for Johnson lifts |
| X-07 | Actuator on the lift COP/LOP | Prohibited by statute; abandoned by the industry |
| X-08 | Stair descent with payload | Not established; manufacturer forbids complex terrain |
| X-09 | Autonomous exploration | Every route must be mapped and human-verified first |
| X-10 | Cloud-hosted server | Violates the no-internet-dependency and no-public-exposure principles |
| X-11 | Teacher motor-control access | Safety |
| X-12 | Unsupervised operation | Manufacturer: not a toy, not for under-14s |
| X-13 | `unitree_slam` (official repo) | **No licence file** — no legal grant of any right. Closed binaries. Targets H1/B2, not Go2 |
| X-14 | `point_lio_unilidar` | **GPL-2.0** — linking forces release of the whole stack. Also ROS 1 Noetic |
| X-15 | Vendor `slam_operate` as the primary path | Requires a purchase, a YSN, and cannot run inside a ROS graph |
| X-16 | Delivery completion inferred from arrival | Design principle — requires explicit recipient confirmation |

---

## 4. Decision dependencies

```
D-103 (robot config)
  └─► D-009 (navigation route) ──► all navigation work

D-104 (network) ─────────────────► TEC-01 (first connection) ──► everything technical
D-105 (server) ──────────────────┘

D-101 (stairs)  ──► Stage 2/3 scope
D-102 (lift)    ──► Stage 3 scope
D-106 (scope)   ──► the whole product definition
D-107 (install) ──► Stage 2+ (floor ID has no in-sensor solution)
D-108 (coverage) ─► D-017 (LoRa defer or build)
D-109 (RF)      ─► any radio feature

D-110 (success) ──► what to build and what to measure
```

⚠️ **Four of these can be started immediately with no school answer:**
D-010 (simulation, decided), TEC-04 (run the sim), D-003 (already decided), and
preparing the questions and survey sheets for FAC-01, FAC-02, FAC-03.

---

## 5. Log conventions

For future entries:

| Field | Content |
|---|---|
| Date | ISO format |
| Decision state | DECIDED / PROVISIONAL / OPEN / REJECTED |
| Owner | Named role |
| Evidence | Source URL or measurement ID |
| Assumptions | What was assumed true when deciding |
| Reversal condition | What would cause this decision to be revisited |
| Consequences | What this commits us to |

**Provisional decisions carry a default.** If a question is not answered, the
documented default applies automatically. This prevents an unanswered question
from silently blocking the project — and prevents an unanswered question from
silently becoming an assumption.

⚠️ **Record the reversal condition at the time of deciding.** Decisions made in
Phase 0 will look different once a measurement exists. A decision without a
recorded reversal condition becomes a decision nobody dares revisit.