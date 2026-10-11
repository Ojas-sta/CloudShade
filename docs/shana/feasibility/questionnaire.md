# SHANA — Requirements Questionnaire

**Phase 0 · For conversion into a Google Sheets workbook · 2026-10-10**

---

## Instructions for conversion

Convert each table below into a separate sheet, named for its section heading
(e.g. `ADM — School Administration`). Preserve all columns exactly. Do **not**
merge cells. Do **not** reorder rows. Do **not** interpret, answer, infer, or fill
in any `Answer` — leave every status as `UNANSWERED` until a human responds.

Create one extra sheet, `Reference`, containing the priority and status
definitions from §2 below.

**This document is self-contained.** No external research is required to convert
it. No school-specific answer is supplied anywhere in this file, and none may be
supplied by the converter.

---

## 1. How to use this questionnaire

This questionnaire exists because the SHANA feasibility study found that **the
architecture cannot be finalised without answers from the school**. It is not a
wish list — each question is either **safety-critical**, **architecture-blocking**,
or needed before a specific next step.

**Respondents are separated deliberately.** Questions requiring school staff are in
sections ADM, LAB, NET, ROB, ELV, TCH and FAC. Questions that we can answer
ourselves — by measurement or by testing — are isolated in section TEC and marked
`INTERNAL`. **Do not send the TEC section to the school.**

**If you only have time to answer a few questions**, answer every `P0`. They are
the minimum set needed to decide whether the project is worth continuing.

---

## 2. Definitions

### Priority

| Code | Meaning |
|---|---|
| **P0** | **ARCHITECTURE-BLOCKING.** SHANA cannot be designed until this is answered. Also includes SAFETY-CRITICAL items. |
| **P1** | Required before the named next phase can begin. |
| **P2** | Important for quality or scope; does not block architecture. |
| **P3** | Nice to have. Can be deferred. |

### Status

| Code | Meaning |
|---|---|
| `UNANSWERED` | No response recorded yet. **All questions start here.** |
| `ANSWERED` | A response has been recorded. |
| `BLOCKED` | Cannot be answered until a dependency is resolved. |
| `N/A` | Does not apply to this school. |

### Flags

| Flag | Meaning |
|---|---|
| 🔴 **SAFETY** | Answer relates to the safety of students, staff, or the robot. |
| 🚧 **BLOCKS-ARCH** | The architecture cannot be chosen until this is answered. |
| ⚖️ **APPROVAL** | Requires a written decision or authorisation from a named authority. |

---

## 3. ADM — School Administration

| ID | Category | Pri | Flags | Question | Why it matters | Respondent | Answer format | Evidence requested | Depends on | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| ADM-01 | Budget | P0 | 🚧 | What budget exists for this project, and over what period? Is there a capital budget for hardware and a separate budget for consumables? | Determines whether an expansion dock, access points, spare batteries, and markers are affordable. | Principal / Administrator / PTA | Free text + figure | Written budget confirmation | — | UNANSWERED |
| ADM-02 | Infrastructure | P0 | 🚧 ⚖️ | Which single room can host the SHANA server and act as the primary robot base? The study proposes the AI Lab (CP-C3). Is this approved? | The server must be reachable by teachers and have a wired Ethernet port for the robot. | Principal + AI Lab teacher | Single choice + approval signature | Written room allocation | — | UNANSWERED |
| ADM-03 | Governance | P0 | 🚧 ⚖️ | Who is the named owner of this project, and who holds the robot during periods when SHANA is not operating it? | Unclear ownership blocks insurance, maintenance, and every future request. | Principal | Names + roles | Written assignment | — | UNANSWERED |
| ADM-04 | Installation | P0 | 🚧 ⚖️ | Does the school permit installing new equipment in corridors and classrooms — specifically: Wi-Fi access points, ArUco/AprilTag markers on walls, cameras, or any networked device? | Floor identification and wireless coverage both depend on this. **Without it, multi-floor operation is likely impossible.** | Principal + IT | Yes / No / Conditional + conditions | Written permission | NET-01 | UNANSWERED |
| ADM-05 | Safety | P0 | 🔴 ⚖️ | What is the school's policy on autonomous or semi-autonomous robots operating in occupied corridors with students present? Are there existing rules about robotics demonstrations? | Determines whether any public demonstration is possible at all. | Principal + Safety Officer | Free text + policy reference | Copy of existing policy | — | UNANSWERED |
| ADM-06 | Insurance | P1 | 🔴 ⚖️ | Does the school's insurance cover a robot operating on its premises, and does operating it during school hours create additional requirements? | A 15 kg machine with moving legs in a student area is an insurance question. | Principal | Yes / No / Conditional | Written confirmation from insurer | ADM-05 | UNANSWERED |
| ADM-07 | Acceptance | P1 | 🚧 | What outcome would make this project a success for the school? Who judges it? | Determines which capabilities must work. A navigation demo and a teacher-time-saving demo are different products. | Principal | Free text | — | — | UNANSWERED |
| ADM-08 | Scope | P1 | 🚧 | Are teachers expected to actually use this in daily operation, or is it a demonstration/research project? | Daily use needs reliability the research version does not need, and vice versa. | Principal | Single choice | — | ADM-07 | UNANSWERED |
| ADM-09 | Maintenance | P2 | — | Who performs physical maintenance and battery replacement, and during what hours? | Determines whether a recovery protocol is realistic. | Facilities + Admin | Names + hours | — | — | UNANSWERED |
| ADM-10 | Data | P2 | 🚧 | Does the school have a policy on camera images, student data, or network telemetry being processed by a project system? | Affects whether any camera-based perception can be used at all. | Principal + IT | Free text | Copy of policy if it exists | — | UNANSWERED |

---

## 4. LAB — AI Lab / Makerspace Staff

| ID | Category | Pri | Flags | Question | Why it matters | Respondent | Answer format | Evidence requested | Depends on | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| LAB-01 | Host | P0 | 🚧 | Is a school computer in the AI Lab available to run the SHANA server 24/7, or only during school hours? What are its specifications (CPU, RAM, disk, network ports, OS)? | The server must be a Linux host on the `192.168.123.0/24` subnet with **at least two network interfaces** — one for the robot, one for teachers. | AI Lab teacher / IT | Single choice + full specs | `sudo lshw -short` or equivalent output, screenshot of Network settings | ADM-02 | UNANSWERED |
| LAB-02 | Host | P0 | 🚧 | Does that computer have a **free Ethernet port**, or can a USB-Ethernet adapter be added? | The robot link is **Ethernet-only**. Wi-Fi will not work for the robot. | AI Lab teacher | Yes / No | Photo of network ports | LAB-01 | UNANSWERED |
| LAB-03 | Power | P1 | — | Is there a reliable mains power outlet near the proposed robot base, and is there a UPS available? | A power cut during operation is a foreseeable failure. | AI Lab teacher | Yes / No | Photo | ADM-02 | UNANSWERED |
| LAB-04 | Storage | P1 | — | Where would the robot be physically stored when not in use, and is that room accessible to authorised staff only? | Robot security and battery storage safety. | AI Lab teacher | Free text + room ID | Photo | ADM-03 | UNANSWERED |
| LAB-05 | Supervision | P1 | 🔴 | Which staff member is trained and authorised to operate the robot manually? Who are the trained backups? | The human spotter role is mandatory at every stage. There must be at least two. | AI Lab teacher | Names | Training record if any | ADM-03 | UNANSWERED |
| LAB-06 | Schedule | P2 | — | During which periods and days would testing be permitted, and when would students be present in the affected corridors? | Testing windows drive the whole roadmap. | AI Lab teacher | Table of permitted windows | — | ADM-05 | UNANSWERED |

---

## 5. NET — Network Administrator / IT

| ID | Category | Pri | Flags | Question | Why it matters | Respondent | Answer format | Evidence requested | Depends on | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| NET-01 | Wireless | P0 | 🚧 | Does the school have any school-wide Wi-Fi? If yes, how many access points, which bands (2.4/5 GHz), and is it managed or consumer-grade? | Teacher access to the SHANA dashboard needs Wi-Fi. The robot does **not** — it needs Ethernet. | IT / Network Admin | Free text + AP count | Network topology diagram | — | UNANSWERED |
| NET-02 | Robot network | P0 | 🚧 | Does the school permit a **dedicated Ethernet segment** on `192.168.123.0/24` for the robot, isolated from the smartboard LAN? | The manufacturer specifies this exact subnet. It must not share a broadcast domain with smartboards. | IT / Network Admin | Yes / No / Conditional | Written approval + VLAN/port plan | — | UNANSWERED |
| NET-03 | Isolation | P0 | 🚧 | Are VLANs or client isolation enabled on the existing school network that would prevent device-to-device communication? | If isolation is on, a wireless-bridge or AP approach may silently fail. | IT / Network Admin | Yes / No + which segments | Switch configuration summary | NET-01 | UNANSWERED |
| NET-04 | Devices | P1 | 🚧 | Does the school permit **new access points** to be installed, and who installs and maintains them? | If coverage per floor is needed for teachers and for any future wireless sensors, APs are the enabler. | IT / Network Admin | Yes / No / Conditional | Written approval | ADM-04 | UNANSWERED |
| NET-05 | Coverage | P1 | 🚧 | **Is there any specific room, stairwell, corridor, courtyard or sports hall where Wi-Fi does NOT reach or is unreliable?** Please list them by name and location. | This single answer decides whether LoRa is justified at all. The study's conclusion is that with one AP per floor, LoRa is largely redundant. | IT / Network Admin | List of locations + description | — | NET-01 | UNANSWERED |
| NET-06 | Local server | P1 | 🚧 | Can the school network reach a locally hosted server on the school LAN without internet access, and is there a firewall rule that would block a browser-based tool on the local subnet? | The server must be reachable by teachers, with no public exposure. | IT / Network Admin | Yes / No | Firewall rule excerpt | NET-01 | UNANSWERED |
| NET-07 | Local hosting | P1 | 🚧 | Does a NAS or local application server exist that SHANA could be hosted on? | The brief confirms none. If one exists it changes the hosting decision. | IT / Network Admin | Yes / No + model | — | — | UNANSWERED |
| NET-08 | Wireless bridge | P2 | ⚖️ | Would an authorised **wireless bridge** be permitted as an alternative to running a cable to the robot? | Fallback if a cable cannot be run. Note: the robot link itself should remain wired. | IT / Network Admin | Yes / No | — | NET-02 | UNANSWERED |
| NET-09 | Power | P2 | — | Are there network/PoE outages, scheduled maintenance windows, or load-shedding patterns that would take the network offline during a demo? | Determines how much the LoRa backup channel is worth. | IT / Network Admin | Free text | — | — | UNANSWERED |
| NET-10 | RF policy | P1 | ⚖️ | Does the school have any RF or wireless-device acceptable-use policy that would apply to a LoRa module or ESP32 device operating in the building? | Institutional policy layer on top of Indian regulation. | IT / Principal | Free text | Copy of policy if it exists | ADM-04 | UNANSWERED |

---

## 6. ROB — Robot Owner / Responsible Robotics Supervisor

| ID | Category | Pri | Flags | Question | Why it matters | Respondent | Answer format | Evidence requested | Depends on | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| ROB-01 | Hardware | P0 | 🚧 | **What exactly is the Go2 variant and what options are installed?** Is it EDU? Is the **expansion dock** present (Orin Nano 40 TOPS or Orin NX 100 TOPS)? Is the **depth camera** present? | The expansion dock changes the navigation route decision entirely, and whether vendor SLAM is even possible. | Robot owner | Checklist + photos | Photo of robot body, ports, and dock if present | — | UNANSWERED |
| ROB-02 | Hardware | P0 | 🚧 | **Which battery is fitted — 8,000 mAh (BT2-05) or 15,000 mAh (BT2-06)?** Read the label on the pack. | Mission endurance planning. Manufacturer figures conflict (1–5 h across documents). | Robot owner | Single choice | Photo of battery label | — | UNANSWERED |
| ROB-03 | Hardware | P1 | — | Is a **charging pile / dock** present, and is there a spare battery? | There is **no autonomous return-to-dock** in the SDK. Recovery is human-assisted by design. | Robot owner | Yes / No + count | Photo | ROB-01 | UNANSWERED |
| ROB-04 | Firmware | P0 | 🚧 | What firmware version is installed? Check the app: Device → Service Status. Record all version numbers shown. | Firmware **≥1.1.6** is required for the current motion interface. Below it, `SwitchGait` (the only stair-specific API) still exists. | Robot owner | Version strings + date | Screenshot | — | UNANSWERED |
| ROB-05 | Access | P0 | ⚖️ | **Does the school authorise use of the Unitree secondary development interface?** Is there a signed agreement, purchase record, or warranty condition governing it? | Only EDU-and-above models may be programmed at all. This is a permission question, not just a technical one. | Robot owner / Principal | Written confirmation | Purchase invoice or written authorisation | ADM-03 | UNANSWERED |
| ROB-06 | Access | P1 | 🚧 | Does the school have a **dedicated Ubuntu computer** available for robot development, or must the robot share the AI Lab server? | SDK2 requires Ubuntu 20.04/22.04, a dedicated NIC, and a static IP. It cannot run on the robot itself. | Robot owner | Yes / No + machine spec | — | LAB-01 | UNANSWERED |
| ROB-07 | Documentation | P1 | — | What documentation, manuals, and warranty terms came with the robot? Are there local service contacts? | Needed for maintenance, spares, and support escalation. | Robot owner | File list | Copies of all supplied documents | — | UNANSWERED |
| ROB-08 | Maintenance | P1 | 🔴 | Has the robot been repaired or serviced since purchase? Any history of faults, falls, or battery issues? | Reliability history affects the risk register directly. | Robot owner | Free text | Service records if any | — | UNANSWERED |
| ROB-09 | Support | P2 | — | Is there a local distributor or service partner, and what is the turnaround time for a replacement part? | A 2-month parts wait has been documented for the sibling platform. | Robot owner | Name + contact + lead time | — | — | UNANSWERED |
| ROB-10 | Existing code | P2 | — | Does anyone already have code, notebooks, or configuration for this robot? Has anyone previously connected to it? | Avoids duplicate work and may reveal prior findings. | Robot owner | Free text | Any existing files | — | UNANSWERED |

---

## 7. ELV — Elevator Maintenance Provider / Authorised Lift Service Company

> **Address these questions to the school's authorised service provider, not to
> the manufacturer directly.** Copy Johnson Lifts (or Toshiba Johnson, depending
> the nameplate) only for the two EyeRIS questions, which are product questions.

| ID | Category | Pri | Flags | Question | Why it matters | Respondent | Answer format | Evidence requested | Depends on | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| ELV-01 | Identity | P0 | 🚧 | **Which company is this?** Is it Johnson Lifts Pvt. Ltd., or Toshiba Johnson Elevators (India) Pvt. Ltd.? Please confirm from the nameplate on the car operating panel. | The integration counterparty differs between the two entities. | Service provider | Single choice | Nameplate photo | — | UNANSWERED |
| ELV-02 | Specification | P0 | 🚧 | What is the **model, rated load, car internal dimensions (W × D × H), door clear opening width, and threshold sill gap** for each of the two lifts? | The robot is 700 × 310 mm and 15 kg. These numbers determine whether it can enter and turn around. | Service provider | Dimensions table + photos | Lift drawings, nameplate photos | ELV-01 | UNANSWERED |
| ELV-03 | Specification | P1 | 🚧 | Which **Indian standard** were these lifts built and certified to? (The study believes IS 17900 (2022 series); please confirm.) | Determines which regulatory regime applies to any change. | Service provider | Standard + year | Type approval certificate | ELV-01 | UNANSWERED |
| ELV-04 | Service | P1 | ⚖️ | Is there an active **annual maintenance contract**, and does it permit the vendor to authorise third-party equipment in the car or lobby? | Any integration must preserve the AMC. | Service provider | Yes / No + copy reference | AMC document (redacted) | — | UNANSWERED |
| ELV-05 | Permission | P0 | ⚖️ | **Does the vendor or manufacturer offer any supported interface allowing a third-party device or robot to place a hall call or destination call?** For example Otis Integrated Dispatch, TK MAX Robot Calls, or Schindler BuilT-In. | If yes, there is a sanctioned path. The study found **none published for Johnson** — this confirms it. | Service provider | Yes / No + details | Product documentation | ELV-01 | UNANSWERED |
| ELV-06 | EyeRIS | P0 | ⚖️ | **Is the Johnson "EyeRIS" contactless call-booking system available for retrofit on lifts installed earlier?** Is the QR element on the landing operating panel a passive printed label, or an active electronic component requiring wiring? | **This is the single most valuable answer for the project.** A passive QR with retrofit availability would give a manufacturer-sanctioned, no-modification call path. | Johnson Lifts (copy Service provider) | Yes / No / Unknown + explanation | Product documentation | ELV-01 | UNANSWERED |
| ELV-07 | EyeRIS | P0 | ⚖️ | **Can EyeRIS be provisioned to a non-human client** — for example an unattended Android device on a robot — rather than only to a person's phone? | Determines whether EyeRIS is usable for SHANA at all. | Johnson Lifts | Yes / No / Unknown | Written statement | ELV-06 | UNANSWERED |
| ELV-08 | Policy | P0 | ⚖️ ⚖️ | **What is the vendor's written position on attaching any device to, over, or wiring into the car operating panel or landing operating panel?** | The study's position is that this is prohibited without Electrical Inspector permission. This confirms the boundary. | Service provider | Free text | Written policy statement | — | UNANSWERED |
| ELV-09 | Safety | P1 | 🔴 | Does either lift have **firefighter's service / Phase I fire recall** fitted? | During recall the car returns to a designated level and the lift is removed from service. Affects the escort protocol. | Service provider | Yes / No per lift | — | ELV-03 | UNANSWERED |
| ELV-10 | Escort | P1 | ⚖️ | Would the vendor permit a small autonomous robot to ride as a passenger **with a named human escort operating the panel**, as a supervised trial? | This is the Stage 2 design. It changes nothing on the lift, so it should be permissible — but it must be confirmed. | Service provider | Yes / No / Conditions | Written consent | ELV-04, ELV-08 | UNANSWERED |

---

## 8. TCH — Teachers Who Will Request Deliveries

| ID | Category | Pri | Flags | Question | Why it matters | Respondent | Answer format | Evidence requested | Depends on | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| TCH-01 | Need | P0 | 🚧 | What is the single most annoying task you would want a robot to take off you? | Defines whether the project solves a real problem. | Teachers | Ranked list of 3 | — | — | UNANSWERED |
| TCH-02 | Need | P1 | 🚧 | What would you actually carry between rooms — notebooks, worksheets, lab equipment, exam papers, library books, art materials? Approximate weight and typical container? | Determines the payload requirement and whether the 8 kg limit is sufficient. | Teachers | Item list + weight + container | Sample items | TCH-01 | UNANSWERED |
| TCH-03 | Workflow | P1 | 🚧 | When would you request a delivery, and how would you want to be notified when it arrives? (Dashboard, screen in room, email, intercom, spoken announcement?) | Determines the notification design and whether a per-room screen is needed. | Teachers | Free text + preference ranking | — | TCH-01 | UNANSWERED |
| TCH-04 | Destinations | P1 | 🚧 | List the rooms you would actually want deliveries to, and **how often**. | This becomes the initial allow-list. 4–6 destinations is a realistic target. | Teachers | Room list + frequency | — | TCH-01 | UNANSWERED |
| TCH-05 | Floors | P0 | 🚧 | Which floors and blocks do you regularly need deliveries between? **Specifically: would you need the robot to use stairs or an elevator?** | This is the question that determines whether Stage 2/3 is in scope at all. | Teachers | Floor/block list + yes/no on stairs and lift | — | — | UNANSWERED |
| TCH-06 | Trust | P1 | 🔴 | Would you be willing for a 15 kg robot to operate in your corridor while students are present? Under what conditions would you accept or refuse? | Establishes the real acceptance threshold, which may be stricter than the technical one. | Teachers | Free text | — | ADM-05 | UNANSWERED |
| TCH-07 | Failure | P2 | — | If the robot cannot complete a delivery, what should happen? Should you be told immediately, or at the end of the day? | Failure-handling expectations. | Teachers | Free text | — | TCH-03 | UNANSWERED |

---

## 9. FAC — Facilities and Safety

| ID | Category | Pri | Flags | Question | Why it matters | Respondent | Answer format | Evidence requested | Depends on | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| FAC-01 | Survey | P0 | 🚧 | **Measure the stair riser height (floor-to-floor step) for every staircase on any route the robot might need.** Record each stairwell ID, floor pair, number of steps, riser height in mm, and tread depth in mm. | **This is the single most decisive unknown.** The Go2 limit is 160 mm. A 150 mm riser leaves 10 mm of margin; 170 mm is outside spec. | Facilities | Table: stairwell, floor, riser mm, tread mm | Photos + tape measurement | — | UNANSWERED |
| FAC-02 | Survey | P0 | 🚧 | **Measure corridor and stairwell clear widths**, accounting for handrail projection. Record the narrowest point on any candidate route. | The robot is 310 mm wide but needs swing room for leg placement. Typical school stairwells are 900–1200 mm. | Facilities | Table: location, clear width mm | Photos + tape | FAC-01 | UNANSWERED |
| FAC-03 | Survey | P0 | 🚧 | **Record the floor surface type for every candidate route** — carpet, vinyl, polished terrazzo, ceramic tile, concrete, mats. Note any rugs or doormats. | Unitree's manual **explicitly forbids** low-friction ground and names foot slip as a failure mode. Polished terrazzo is common in Indian schools and is the worst case. | Facilities | Table: location, surface, notes | Photos | — | UNANSWERED |
| FAC-04 | Survey | P1 | 🚧 | Are there **thresholds, ramps, cable runs, floor transitions, or level changes** on any candidate route? Record heights in mm. | 160 mm step limit, but a 20 mm threshold at speed is still a trip risk. | Facilities | Table with heights | Photos + tape | — | UNANSWERED |
| FAC-05 | Survey | P1 | 🚧 | **Map the actual routes between blocks.** Confirm the C↔B transition (believed basement and ground floor only) and **document the B↔A route**, which has not been surveyed. | The brief states this is unsurveyed. Without it, cross-block routes cannot be declared. | Facilities with AI Lab staff | Annotated floor plan | Floor plans or annotated photos | — | UNANSWERED |
| FAC-06 | Survey | P1 | 🚧 | Which rooms are permanently or temporarily obstructed — storage, cleaning equipment, fire extinguishers, furniture in corridors? | Obstruction map for the route planner. | Facilities | Annotated floor plan | Photos | FAC-05 | UNANSWERED |
| FAC-07 | Checkpoint | P1 | 🚧 | Confirm the **AI Lab (CP-C3)** and **Makerspace Lab (CP-A1)** locations, floor numbers, room IDs, and whether each can serve as a robot checkpoint with charging. | These are the proposed base stations. Their positions must be in the map. | Facilities | Room IDs + confirmation | Floor plan | ADM-02 | UNANSWERED |
| FAC-08 | Battery | P1 | 🔴 | What is the school's procedure for **handling and charging lithium battery packs**? Where can charging happen, supervised, on a non-flammable surface? | Manufacturer limits: discharge −20 to 60 °C, charge 0 to 55 °C. Unitree warns exceeding these "may cause the battery to ignite or even explode." | Facilities | Free text + location | — | — | UNANSWERED |
| FAC-09 | Battery | P1 | 🔴 | Is there a designated **battery quarantine / damaged battery** procedure? Who do you call if a pack swells, is damaged, or is dropped? | A safety procedure that must exist before the first pack is charged. | Facilities | Procedure + contact | — | FAC-08 | UNANSWERED |
| FAC-10 | Safety | P0 | 🔴 | **What emergency stop arrangements will be in place?** Who holds the remote, where is the nearest hard power switch, and what is the procedure if a student approaches the robot? | Mandatory at every stage. The manual requires the robot stay under control in the user's view and 2 m from crowds. | Facilities + AI Lab staff | Written procedure | Signed procedure document | ADM-05, LAB-05 | UNANSWERED |
| FAC-11 | Safety | P1 | 🔴 | Where can a **physically secured exclusion zone** be set up for testing, and who enforces it? | Testing must not happen in an occupied corridor. | Facilities | Location + owner | — | ADM-05 | UNANSWERED |
| FAC-12 | Safety | P1 | 🔴 | Is there a first-aid contact and a procedure for a student or staff member being struck or tripping over the robot? | A realistic risk given 12 moving legs. | Facilities + Admin | Procedure + contacts | — | — | UNANSWERED |

---

## 10. TEC — Internal Technical Verification (do NOT send to the school)

> These are answerable by us, with a measuring device in hand or in front of a
> computer. They are listed here so the register is complete and so nobody
> duplicates the work.

| ID | Category | Pri | Flags | Question | Why it matters | Answer format | Evidence requested | Depends on | Status |
|---|---|---|---|---|---|---|---|---|---|
| TEC-01 | Hardware | P0 | 🚧 | Establish a **read-only DDS connection** on a wired Ethernet NIC with static IP `192.168.123.222/24`. Subscribe `rt/lowstate` and `rt/sportmodestate` for 30 minutes. **No actuation.** | Proves the network, the NIC binding, the topic set, and firmware compatibility. Zero risk. | Log file + screenshots | Subscription log, robot IP, NIC name | ROB-01, ROB-04, NET-02 | UNANSWERED |
| TEC-02 | Hardware | P1 | 🚧 | **Measure the actual camera frame rate.** Unitree documentation contradicts itself: `about_Go2` says 15 fps, the FAQ says 30/60 fps. | Determines whether real-time perception is viable. | Measured fps + method | Screen recording or frame counter | TEC-01 | UNANSWERED |
| TEC-03 | Hardware | P1 | — | Confirm the LiDAR point cloud flows on `rt/utlidar/cloud_deskewed` and record its rate and message size. | Sizing the SLAM input and the host CPU load. | Rate + bytes/msg | Subscription log | TEC-01 | UNANSWERED |
| TEC-04 | Hardware | P0 | 🚧 | Run `unitree_mujoco` (BSD-3, CPU-only) with the Go2 model and confirm our ROS 2 nodes run unchanged against it. **The simulator speaks the same DDS channels as the robot.** | Validates the whole software stack at zero risk and zero cost before the robot is switched on. | Build log + demo | Video of a simulated mission | — | UNANSWERED |
| TEC-05 | Battery | P1 | 🔴 | Record `LowState.bms_state` fields over a full charge-discharge cycle: `soc`, `current`, `cell_vol[15]`, `bq_ntc`. Build a coulomb-counting range estimate. | Battery telemetry is on `rt/lowstate`, **not** `SportModeState`. Manufacturer figures conflict (1–5 h). We need our own number. | Measured Wh/km, endurance curve | Log files | TEC-01 | UNANSWERED |
| TEC-06 | Stairs | P0 | 🔴 **SAFETY** | **Only if FAC-01 shows risers ≤160 mm**: run the supervised unloaded stair protocol — flat corridor, single step, full flight, ascent and both descent modes, in that order, with two people present. Stop at the first hold point that fails. | The only way to establish whether stairs are usable at all. Descent is the gating capability. | Test log with pass/fail per stage | Video, riser height, per-attempt record | FAC-01, FAC-02, ADM-05, LAB-05 | UNANSWERED |
| TEC-07 | Gait | P1 | 🚧 | Determine whether stair gaits are reachable from the EDU SDK. `SwitchGait` is absent on firmware ≥1.1.6 but exists in the consumer firmware and in the B2-W client. Read `gait_type` from `rt/sportmodestate` during any stair attempt. | If we cannot command a stair mode, we rely on an emergent property of `ClassicWalk` with no safety envelope. | Documented method + gait_type values | Source references + telemetry | TEC-06 | UNANSWERED |
| TEC-08 | Floor ID | P0 | 🚧 | Test ArUco detection with the Go2 head camera at realistic approach distances and angles under school lighting. Measure detection rate. | Floor identification is the mandatory multi-floor blocker, and the charging pile already uses ArUco — so this tests existing hardware. | Detection rate vs distance/angle | Test images + counts | ROB-01, ADM-04 | UNANSWERED |
| TEC-09 | Charging | P1 | 🚧 | Locate and read the official **"Aruco Recharge Service Interface"** documentation (page exists in the doc nav; body has not rendered). Determine the `rt/aruco_cmd` contract. | There is **no** autonomous return-to-dock in the service list. If an ArUco recharge interface exists, automatic docking becomes possible. | Documented API summary | Source quote + URL | — | UNANSWERED |
| TEC-10 | Regulation | P1 | ⚖️ | Email the **DoT WPC Helpdesk** asking whether a school requires a possession authorisation under the Telecommunications (Radio Equipment Possession Authorisation) Rules, 2026 (G.S.R. 592(E), 8 Jul 2026), for 865–868 MHz equipment. | The rules changed two months before this study. Rule 10(c)/(h) probably exempts a school, but **no DoT clarification was found**. | Written reply from DoT | Email thread | ADM-01 | UNANSWERED |

---

## 11. Summary of `P0` questions — the minimum viable answer set

If only ten answers are available, these ten unblock the architecture:

| # | ID | Question in one line |
|---|---|---|
| 1 | ROB-01 | Which Go2 variant and which options are installed? |
| 2 | ROB-04 | What firmware version is running? |
| 3 | ROB-05 | Is secondary development authorised in writing? |
| 4 | NET-02 | Is a dedicated `192.168.123.0/24` Ethernet segment permitted? |
| 5 | NET-05 | Is there any room where Wi-Fi does not reach? |
| 6 | LAB-01/02 | Is there a Linux host with a spare Ethernet port in the AI Lab? |
| 7 | **FAC-01** | **What are the actual stair riser heights?** |
| 8 | ADM-04 | Are access points and wall markers permitted? |
| 9 | ELV-06/07 | Is EyeRIS retrofittable, passive, and provisionable to a robot? |
| 10 | TCH-05 | Do teachers actually need stairs or elevator transit? |

**Questions 7, 9 and 10 together determine whether Stage 2 and Stage 3 exist at
all.** The other seven determine how Stage 1 is built.

---

## 12. Conversion notes for Gemini

- Create **10 sheets**: `Reference`, plus one per section (ADM, LAB, NET, ROB, ELV,
  TCH, FAC, TEC) — that is `Reference` + 8 section sheets.
- Add an `Answer` column and a `Answered By` column after `Status` on every
  section sheet, both empty.
- Preserve `UNANSWERED` in every `Status` cell. **Do not pre-fill answers.**
- Keep the `🔴 SAFETY`, `🚧 BLOCKS-ARCH`, and `⚖️ APPROVAL` flags as text so they
  can be filtered.
- Add a `Filter` to the `Priority` and `Status` columns on each sheet.
- **Do not delete the TEC section** — but mark it clearly as internal so it is not
  sent to school staff.