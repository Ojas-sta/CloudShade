# SHANA — Open Questions Register

**Phase 0 · 2026-10-10**

Every question that must be answered before architecture selection can be
finalised, with the named party who can answer it. Full question text is in
`questions.md`.

**Nothing in this register is a research task.** Each one requires a person, a
measurement, or a written approval. Research has already been done as far as it
can be.

---

## 1. The blocking set

These nine determine whether the architecture can be chosen at all.

| # | ID | Question | Blocks | Who can answer | If unanswered |
|---|---|---|---|---|---|
| **OQ-1** | **ROB-01** | Which Go2 variant and which options are installed? (EDU? expansion dock? depth camera?) | **Navigation route A vs B.** The dock decides whether vendor `slam_operate` is even an option | Robot owner | Assume **Route B** (self-built Nav2) — which was already the recommendation, and is the no-purchase path |
| **OQ-2** | **ROB-04** | Firmware version? | Whether `SwitchGait` (the only stair-specific API, removed on ≥1.1.6) is reachable at all | Robot owner | Assume ≥1.1.6 → **no stair gait setter available** |
| **OQ-3** | **ROB-05** | Is secondary development authorised **in writing**? | **The entire project.** Only EDU-and-above may be programmed | Robot owner + Principal | Project cannot start |
| **OQ-4** | **NET-02** | Is a dedicated `192.168.123.0/24` Ethernet segment permitted? | **The robot link.** DDS is Ethernet-only; Wi-Fi will not work | IT / Network Admin | Fall back to a **direct cable from the AI Lab server's USB-Ethernet NIC to the robot** — no school network change at all |
| **OQ-5** | **FAC-01** | **Actual stair riser heights?** | **Stage 2/3 reachability.** Go2 limit is 160 mm | Facilities | **Assume stairs are out of scope.** This is the conservative default |
| **OQ-6** | **ELV-06, ELV-07** | Is Johnson EyeRIS retrofittable, is the QR passive, and can it be provisioned to a robot client? | Whether any non-escort lift path exists | Johnson Lifts (via service provider) | Assume **human escort is permanent**, which is the industry norm |
| **OQ-7** | **TCH-05** | Do teachers actually need stairs or elevator transit between floors? | **Whether Stage 2 and Stage 3 exist** | Teachers | Assume **single-floor only**. Build Stage 1 well |
| **OQ-8** | **ADM-04** | Are access points and wall-mounted markers permitted? | **Floor identification.** Without markers, multi-floor is likely impossible | Principal + IT | Assume **single floor only.** Multi-floor becomes infeasible |
| **OQ-9** | **LAB-01, LAB-02** | Is there a Linux host with a spare Ethernet port in the AI Lab? | Where the server runs | AI Lab teacher | Fall back to a dedicated machine, which requires ADM-01 budget |

---

## 2. Sequencing — which answers unlock which work

```
WEEK 1 ──► OQ-3 (authorisation)      ← gates EVERYTHING. Ask first.
         └► OQ-9 (server host)        ← gates TEC-01
         └► OQ-1, OQ-2 (robot inspect) ← gates TEC-01

WEEK 2 ──► OQ-4 (Ethernet segment)   ← gates TEC-01
         └► OQ-5 (measure risers)    ← gates TEC-06 and the whole Stage 2/3 question
         └► OQ-8 (installation)      ← gates floor-ID work
         └► OQ-7 (teacher needs)     ← gates SCOPE, not architecture

WEEK 3 ──► OQ-6 (Johnson)            ← long lead time; start the email immediately

ONGOING ──► TEC-01 read-only DDS connection   ← first technical action once OQ-3/4/9 land
```

⚠️ **OQ-6 has the longest lead time and the lowest cost.** It is a single email to
Johnson Lifts. **Send it in week 1, in parallel with everything else.** If the
answer is yes, it changes Stage 3 permanently.

---

## 3. Grouped by who can answer

### Principal / Administration — 5 questions

| ID | One-line |
|---|---|
| ADM-01 | Budget, and over what period? |
| ADM-02 | Which room hosts the server and base? |
| ADM-03 | Who owns the robot and the project? |
| ADM-04 | Are APs and markers permitted? ⚖️ |
| ADM-05 | What is the policy on robots in occupied corridors? 🔴 |
| ADM-06 | Does insurance cover it? ⚖️ |
| ADM-07 | What does "success" mean for the school? |
| ADM-08 | Daily use or demonstration? |
| ADM-10 | Data/camera policy? |

### IT / Network Administrator — 6 questions

| ID | One-line |
|---|---|
| NET-01 | Is there school-wide Wi-Fi? How many APs? |
| NET-02 | Is a dedicated `192.168.123.0/24` segment permitted? |
| NET-03 | Are VLANs / client isolation preventing device-to-device? |
| NET-04 | Are new APs permitted, and who maintains them? |
| NET-05 | **Is there any room where Wi-Fi does not reach?** |
| NET-06 | Can a local server be reached without internet? |
| NET-07 | Does a NAS or local app server exist? |
| NET-10 | Is there an RF acceptable-use policy? |

### Facilities — 5 questions, one of them decisive

| ID | One-line |
|---|---|
| **FAC-01** | **Measure every stair riser height in mm.** 🔴🚧 |
| FAC-02 | Corridor and stairwell clear widths |
| FAC-03 | Floor surface types (Unitree forbids low friction) |
| FAC-04 | Thresholds, ramps, cable runs |
| FAC-05 | **Map the actual B↔A route** (unsurveyed) |
| FAC-07 | Confirm CP-C3 and CP-A1 |
| FAC-08/09 | Battery handling and quarantine procedure 🔴 |
| FAC-10 | Emergency stop arrangement 🔴 |
| FAC-11 | Where is a physically secured exclusion zone? 🔴 |

### Robot owner — 3 blocking questions

| ID | One-line |
|---|---|
| ROB-01 | Variant + installed options 🚧 |
| ROB-02 | Which battery? |
| ROB-03 | Charging pile and spare battery present? |
| ROB-04 | Firmware version 🚧 |
| ROB-05 | **Is secondary development authorised?** 🚧 |
| ROB-08 | Repair/fault history 🔴 |

### Johnson Lifts (via service provider) — 3 questions, highest value

| ID | One-line |
|---|---|
| ELV-06 | **EyeRIS retrofittable? QR passive or active?** ⚖️ |
| ELV-07 | **Can EyeRIS be provisioned to a robot client?** ⚖️ |
| ELV-05 | Any sanctioned third-party/robot call interface at all? |

### Teachers — 2 questions

| ID | One-line |
|---|---|
| TCH-05 | **Do you need stairs or elevator transit?** 🚧 |
| TCH-06 | Would you accept a 15 kg robot near students? 🔴 |
| TCH-02 | What would you actually carry, and how heavy? |

### Internal — ours to answer, no school input needed

| ID | One-line |
|---|---|
| TEC-01 | Establish read-only DDS connection |
| TEC-04 | Run the whole stack in `unitree_mujoco` — zero hardware risk |
| TEC-02 | Measure the real camera frame rate (docs contradict) |
| TEC-05 | Build our own battery endurance curve from `bms_state` |
| TEC-08 | Test ArUco detection with the head camera |
| TEC-09 | Find the "Aruco Recharge Service Interface" documentation |
| TEC-10 | Email DoT WPC about the 2026 possession-authorisation rules |

---

## 4. Questions research cannot answer

Explicitly recorded so nobody wastes time looking for them.

| Question | Why research cannot answer it |
|---|---|
| Can the Go2 climb *this* school's stairs? | Needs a tape measure and a supervised test |
| Do these specific lifts have EyeRIS? | Needs the nameplate and a phone call to Johnson |
| Is that QR code passive or active? | Needs physical inspection |
| Does Wi-Fi reach the sports hall in the basement? | Needs a site survey |
| Is there a budget? | Needs the Principal |
| Will teachers use it? | Needs teachers |
| Does the school's insurance cover it? | Needs the insurer |
| Does a school need a 2026 radio possession authorisation? | Needs DoT in writing |
| Is the school's stair nosing polished terrazzo? | Needs eyes on the floor |
| What is the actual car depth of that lift? | Needs a tape measure |

---

## 5. What happens if the worst-case answers come back

| Scenario | Consequence | Response |
|---|---|---|
| Standard G1 → *not applicable* (Go2 EDU confirmed) | — | — |
| Go2 is **not** an EDU | Project impossible | Stop. Escalate. Only EDU may be programmed |
| Secondary development **not** authorised | Project impossible | Stop. Escalate to Principal |
| No Ethernet segment permitted **and** no spare NIC | Robot link blocked | Direct cable from server NIC to robot; requires ADM-04 |
| **All risers >170 mm** | Stairs permanently out | Elevator + escort only. Reassess whether cross-floor is worth it at all |
| Stair risers 150–170 mm **and** TEC-06 fails | Stairs out | As above |
| **No installation permitted at all** | No markers → no floor ID → single floor only | **Redesign as a single-floor system.** This can still be a good project |
| **Johnson says no to everything** | Human escort is permanent | Accept it. Every comparable operator does this |
| Teachers say **cross-floor is not needed** | Stage 2/3 drop out | **Excellent outcome — the project gets simpler and more likely to work** |
| No budget for a dock | Route B mandatory | Already the recommendation |
| **No room lacks Wi-Fi** | LoRa unjustified | Do not build it |
| Battery is 8,000 mAh | Endurance ~1–2 h | Shorter missions; checkpoint per floor becomes essential |

⚠️ **Note the pattern: five of the twelve worst cases end with a smaller project.**
That is the correct outcome for a feasibility phase. A one-floor system that
works is worth more than a three-block system that does not.

---

## 6. Explicitly NOT open questions

Settled by research. Recorded so they are not re-litigated.

| Question | Answer | Basis |
|---|---|---|
| Which robot? | **Go2 EDU** | Given, and confirmed as the only viable option of the two platforms |
| Robot-to-server protocol? | **Cyclone DDS over wired Ethernet** | Only manufacturer-supported path |
| Will the robot work on Wi-Fi? | **No** | Unitree: wired-only for development; SPDP unreliable on Wi-Fi |
| Is there an official navigation stack? | **No** | `unitree_ros2` has no nav2/slam_toolbox/amcl dependency at all |
| Can the robot know which floor it is on? | **No, not with stock sensors** | Every message field enumerated — no altitude, no pressure, no barometer |
| Is autonomous lift operation possible? | **Not without prohibited modification** | No Johnson API exists; all direct routes prohibited by statute |
| Is autonomous return-to-dock available? | **No** | No dock service in the documented service list |
| Can the robot deliver without manipulation? | **Yes** | It carries on a tray; nothing is picked up |
| Where is battery telemetry? | **`rt/lowstate` → `LowState.bms_state`** | `SportModeState` has no battery fields |
| Should LoRa be the primary link? | **No — never** | Non-deterministic, wrong latency class, wrong data rate |
| Can LoRa carry camera or lidar? | **No** | 20–60 kB per frame = 20–60 s airtime; duty cycle >100 % |
| Will teachers get motor control? | **No** | Design principle; safety |

---

## 7. Recommended action sequence

**This week:**

1. Send the **Johnson EyeRIS email** (ELV-06, ELV-07). One email, longest lead
   time, potentially the highest-value answer in the register.
2. Ask the Principal for **ADM-03 and ADM-05** — ownership and safety policy.
   Without these, nothing else is worth asking for.
3. Ask the **robot owner** for **ROB-01, ROB-02, ROB-04** — photograph the robot
   and its battery label. Fifteen minutes of work.
4. Ask **Facilities to measure the stair risers** (FAC-01). A tape measure and
   an afternoon. **This may end the multi-floor question immediately.**

**Next week:**

5. Ask **IT** for **NET-01, NET-02, NET-05**.
6. Ask **Teachers** for **TCH-05** — the floors they actually need.
7. Email **DoT WPC** (TEC-10) if LoRa remains under consideration.

**Once OQ-3 and OQ-4 land:**

8. Establish the read-only DDS connection (TEC-01).
9. Run the full stack in simulation (TEC-04) — **this can happen in parallel and
   requires no school answer at all.**