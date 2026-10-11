# SHANA — Feasibility & Requirements Questionnaire

**Project:** SHANA — School Autonomous Notification & Assist
**Platform:** Unitree Go2 EDU (school-owned)
**Phase:** Feasibility & Requirements Discovery — **no implementation**
**Version:** 1.0 · **Date:** 2026-10-10

---

## START HERE

### What this document is

A structured questionnaire to establish what SHANA **can** do, what it **may** do,
and what **remains unknown**. It exists because the feasibility study found that
the architecture cannot be finalised without answers from the school, its IT
department, its facilities team, its robot custodian, and its lift service
provider.

### What this document is NOT

- ❌ It is **not** an implementation plan.
- ❌ It contains **no answers**. Every `Answer` and `Evidence / Source /
  Attachment` field is deliberately empty and every `Status` is `UNANSWERED`.
- ❌ It contains **no production code**.
- ❌ No school-specific answer has been supplied or inferred anywhere in this file.

### How to fill it in

1. Work through **P0** questions first. They are safety, permission, or
   fundamental-feasibility questions that block everything else.
2. Answer only what you can answer with evidence. **"Unknown" is a valid and
   valuable answer** — it is far better than a guess.
3. Where a question says `Blocks: YES`, leave it unanswered rather than guessing.
4. Put supporting documents, photos, screenshots or links in the
   `Evidence / Source / Attachment` column.
5. Set `Status` to `VERIFIED` **only** when an attached piece of evidence supports
   the answer. `ANSWERED` means a person replied; `VERIFIED` means evidence
   exists.
6. Use `Follow-up Required` to record what must happen next.

### Priority definitions

| Priority | Meaning |
|---|---|
| **P0** | **SAFETY / PERMISSION / FUNDAMENTAL FEASIBILITY.** Must be answered before the relevant activity can proceed at all. |
| **P1** | **ARCHITECTURE BLOCKER.** Determines a major architecture choice. |
| **P2** | **PILOT REQUIREMENT.** Must be answered before the relevant controlled pilot. |
| **P3** | **OPTIMISATION / FUTURE ENHANCEMENT.** Improves convenience, reporting or performance. Does not block a first safe prototype. |

### Status values

`UNANSWERED` · `IN PROGRESS` · `ANSWERED` · `VERIFIED` · `BLOCKED` · `NOT APPLICABLE`

### Blocks Implementation values

`YES` · `NO` · `CONDITIONAL`

### Evidence classification (used in the `Evidence` column)

| Code | Meaning |
|---|---|
| `VERIFIED` | Authoritative documentation or direct authorised evidence |
| `DOC_NOT_HW_VERIFIED` | Documented by manufacturer/vendor, not tested on our equipment |
| `PLAUSIBLE_TEST` | Technically plausible, feasibility not established |
| `UNKNOWN` | Insufficient information |
| `NEEDS_SCHOOL_PERM` | Requires explicit school approval |
| `NEEDS_VENDOR` | Requires confirmation from manufacturer or authorised provider |
| `NOT_PERMITTED` | Unsafe/unsupported without authorisation and approved procedure |

### Respondent prefixes

`ADM` school administration · `IT` network/IT · `ROB` robot owner/custodian ·
`LAB` AI Lab / Makerspace staff · `TCH` teachers · `FAC` facilities & housekeeping ·
`CCTV` CCTV owner & privacy authority · `LIFT` lift manufacturer/service provider ·
`VENDOR` Unitree / equipment vendor · `EXPO` exhibition organisers · `INT` internal (SHANA team)

### Category index

| # | Prefix | Category | Questions |
|---|---|---|---|
| A | `SH-GO2` | Go2 EDU hardware, software, SDK and interfaces | 24 |
| B | `SH-NET` | School network, Wi-Fi, wired LAN and connectivity | 24 |
| C | `SH-SRV` | Local server, dashboard, authentication and security | 20 |
| D | `SH-RAD` | LoRa, ESP32, gateways, antennas, long-range telemetry | 20 |
| E | `SH-CCTV` | CCTV, computer vision, occupancy estimation and privacy | 24 |
| F | `SH-CONG` | Corridor congestion and dynamic route selection | 20 |
| G | `SH-ENV` | Wet floors, rain, lighting and environmental hazards | 20 |
| H | `SH-BLOCK` | Obstacles, construction, maintenance, blocked corridors | 20 |
| I | `SH-LIFT` | Elevators, lift control, authorised interfaces, service support | 26 |
| J | `SH-STAIR` | Stairs, ramps and vertical navigation | 20 |
| K | `SH-MAP` | Maps, named routes, localisation, checkpoints, ArUco markers | 24 |
| L | `SH-BAT` | Battery monitoring, charging, recovery, emergency return | 22 |
| M | `SH-TIME` | School schedules, lunch breaks and operating windows | 20 |
| N | `SH-DEL` | Delivery workflow, teachers, recipients, item restrictions | 22 |
| O | `SH-SAFE` | Safety, emergency stops, failure handling, human intervention | 26 |
| P | `SH-PRIV` | Permissions, school policies, student privacy, data retention | 22 |
| Q | `SH-EXPO` | Exhibition demonstration, evaluation and acceptance testing | 20 |
| R | `SH-BUD` | Budget, installation, maintenance, long-term ownership | 20 |
| S | `SH-GIT` | Git repository, documentation, configuration, deployment | 20 |
| T | `SH-DEC` | Architecture decisions and project go/no-go criteria | 20 |
| | | **TOTAL** | **434** |

---

# A. Go2 EDU hardware, software, SDK and interfaces — `SH-GO2`

> **Context.** Unitree documents secondary development for **EDU-and-above models
> only**: *"Only EDU or above versions can be developed, other versions such as
> PRO and AIR versions do not open the secondary development interface."*
> Generic Go2 documentation does not automatically apply to the EDU variant.

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-GO2-001 | Identity | P0 | Cannot design anything without knowing the actual unit | What is the exact Go2 model designation and hardware revision of the school robot? | The EDU variant differs from PRO/X in compute, sensors, battery and secondary-development support. | ROB | Photo of nameplate, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-GO2-002 | Identity | P0 | Determines whether development is possible at all | What firmware versions are installed (motion service, VUI, lidar, OTA)? | Firmware ≥ V1.1.6 changes the motion interface and removes `SwitchGait`. | ROB | Screenshot of app Device → Service Status | SH-GO2-001 | YES | | | UNANSWERED | | |
| SH-GO2-003 | Identity | P1 | Decides navigation architecture | Is the expansion compute dock present? If so, which (Orin Nano 40 TOPS / Orin NX 100 TOPS)? | The vendor SLAM service requires the dock plus a separately purchased LiDAR. | ROB | Photo, `NEEDS_SCHOOL_PERM` | SH-GO2-001 | YES | | | UNANSWERED | | |
| SH-GO2-004 | Identity | P1 | Determines available sensors | Is the EDU depth camera fitted? | Depth is needed for stair-edge and low-obstacle perception. | ROB | Photo | SH-GO2-001 | NO | | | UNANSWERED | | |
| SH-GO2-005 | Identity | P2 | Affects walking control quality | Are foot force sensors fitted? | EDU-only feature; enables better gait feedback. | ROB | Photo | SH-GO2-001 | NO | | | UNANSWERED | | |
| SH-GO2-006 | Identity | P1 | Halves or doubles the endurance budget | Which battery is fitted — 8000 mAh (BT2-05) or 15000 mAh (BT2-06)? Read the label. | Manufacturer runtime figures conflict (1–5 h across documents). | ROB | Photo of battery label | SH-GO2-001 | YES | | | UNANSWERED | | |
| SH-GO2-007 | Identity | P2 | Recovery depends on it | Is a charging pile/dock present? How many spare batteries? | There is no autonomous return-to-dock in the SDK; recovery is human-assisted. | ROB | Photo | SH-GO2-001 | NO | | | UNANSWERED | | |
| SH-GO2-008 | Permissions | P0 | Technical capability is not permission | Does the school hold **written authorisation** to use the Unitree secondary development interface? | Purchasing an EDU does not automatically grant development access. | ADM / ROB | Written authorisation, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-GO2-009 | Permissions | P1 | Affects who may connect | Who is the named custodian authorised to connect a computer to the robot? | Controls physical and network access to the device. | ADM | Named custodian | SH-GO2-008 | NO | | | UNANSWERED | | |
| SH-GO2-010 | SDK | P0 | Determines whether development can start | Which Unitree SDK version is approved for use, and is it already downloaded by the school? | Using an unapproved SDK version may breach warranty or support terms. | ROB | Document reference | SH-GO2-008 | YES | | | UNANSWERED | | |
| SH-GO2-011 | SDK | P1 | Determines build environment | Is there a school-approved Linux host for robot development (Ubuntu 20.04/22.04)? | Unitree: *"Development on Mac or Windows is currently not supported."* | ROB / IT | Host specification | SH-NET-001 | YES | | | UNANSWERED | | |
| SH-GO2-012 | Interfaces | P1 | Primary motion interface | Which high-level motion methods does the school intend to use, and has any been tested? | `SportClient.Move(vx,vy,vyaw)` is the primary locomotion call. | INT | Test log | SH-GO2-010 | NO | | | UNANSWERED | | |
| SH-GO2-013 | Interfaces | P1 | Determines navigation feasibility | Does the school wish to use Unitree's `slam_operate` service, or build its own navigation on the built-in LiDAR? | `slam_operate` requires EDU + dock + separately purchased MID-360/XT16 + a Unitree YSN. | INT | Written decision | SH-GO2-003 | YES | | | UNANSWERED | | |
| SH-GO2-014 | Interfaces | P1 | Data source for mapping | Has anyone confirmed the built-in LiDAR publishes point clouds on DDS? | `rt/utlidar/cloud_deskewed` is `PointCloud2_`-compatible. | INT | Subscription log | SH-GO2-010 | NO | | | UNANSWERED | | |
| SH-GO2-015 | Interfaces | P2 | Affects perception design | What camera frame rate has actually been measured? | Unitree documentation contradicts itself: 15 fps vs 30/60 fps. | INT | Measurement | SH-GO2-010 | NO | | | UNANSWERED | | |
| SH-GO2-016 | Mapping | P1 | Core navigation requirement | Does Unitree document map persistence and reloading for the Go2? | Without persistence, the map must be rebuilt every session. | INT | Source citation | SH-GO2-010 | NO | | | UNANSWERED | | |
| SH-GO2-017 | Mapping | P1 | Core navigation requirement | Is goal-based navigation (send goal, receive result) exposed through an official interface? | Without it, position tracking must be built externally. | INT | Source citation | SH-GO2-010 | NO | | | UNANSWERED | | |
| SH-GO2-018 | Sensors | P2 | Affects obstacle strategy | What are the documented environmental constraints of the depth camera (range, FOV, minimum distance)? | Determines whether low obstacles and stair edges are detectable. | INT | Source citation | SH-GO2-004 | NO | | | UNANSWERED | | |
| SH-GO2-019 | Sensors | P1 | Battery monitoring requirement | Through which official interface is battery state of charge read? | Battery is on `rt/lowstate` → `LowState.bms_state`, **not** `SportModeState`. | INT | Source citation | SH-GO2-010 | NO | | | UNANSWERED | | |
| SH-GO2-020 | Safety | P0 | Safety requirement | What does Unitree document about emergency stop and safe-stop behaviour? | Determines SHANA's safe-state design. | INT / VENDOR | Manual citation | — | YES | | | UNANSWERED | | |
| SH-GO2-021 | Safety | P1 | Recovery planning | What recovery behaviours does the SDK expose, and which are safe with payload? | `RecoveryStand()` rises *"regardless of whether it has fallen"* — unsafe with 8 kg. | INT | Source citation | SH-GO2-010 | NO | | | UNANSWERED | | |
| SH-GO2-022 | Payload | P1 | Delivery design | What payload mounting points and limits are documented for the EDU? | Rated ≈8 kg but no centre-of-mass envelope is published. | INT | Source citation | SH-GO2-001 | YES | | | UNANSWERED | | |
| SH-GO2-023 | Restrictions | P1 | Scope limitation | What manufacturer restrictions apply to the EDU in a school with children? | Manual: *"not a toy and is not intended for use by persons under the age of 14."* | INT / VENDOR | Manual citation | — | YES | | | UNANSWERED | | |
| SH-GO2-024 | Support | P2 | Long-term viability | What warranty and support arrangements exist, and who is the local service partner? | Repair turnaround materially affects project continuity. | ROB / VENDOR | Contract reference | — | NO | | | UNANSWERED | | |

---

# B. School network, Wi-Fi, wired LAN and connectivity — `SH-NET`

> **Context.** The robot-to-server link is **wired Ethernet over Cyclone DDS**.
> Unitree documents development as wired-only. A WLAN failure therefore does not
> stop the robot — but only if the link is genuinely wired.

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-NET-001 | Wired LAN | P0 | The robot link depends on it | What is the existing wired LAN topology? Which switch, VLANs, and what does it serve? | The robot requires an isolated `192.168.123.0/24` subnet, not the smartboard LAN. | IT | Topology diagram, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-NET-002 | Wired LAN | P0 | Physical prerequisite | Are free Ethernet ports available, or can a USB-Ethernet adapter be added to the server host? | DDS discovery is unreliable without a dedicated wired NIC. | IT / LAB | Port count | SH-NET-001 | YES | | | UNANSWERED | | |
| SH-NET-003 | Wired LAN | P0 | Permission required | Does the school permit a dedicated robot network segment separate from the smartboard LAN? | Manufacturer specifies an isolated subnet; sharing a broadcast domain is unsupported. | IT | Written approval, `NEEDS_SCHOOL_PERM` | SH-NET-001 | YES | | | UNANSWERED | | |
| SH-NET-004 | Wired LAN | P1 | Addressability | Can a static IP be assigned to the server on the robot subnet? | Unitree requires `192.168.123.222`; the robot is `192.168.123.161`. | IT | Assignment record | SH-NET-003 | YES | | | UNANSWERED | | |
| SH-NET-005 | Wired LAN | P0 | Security control | Are VLANs or client isolation enabled that would prevent device-to-device traffic? | Isolation silently breaks any wireless or bridged approach. | IT | Switch config excerpt | SH-NET-001 | YES | | | UNANSWERED | | |
| SH-NET-006 | Firewall | P1 | Determines whether the server is reachable | Do firewall rules permit the server to accept teacher-device connections on the LAN? | The dashboard must be reachable by teachers. | IT | Rule excerpt | SH-NET-001 | YES | | | UNANSWERED | | |
| SH-NET-007 | Firewall | P1 | Blocks the LiDAR path if wrong | Do firewall rules permit UDP multicast on the robot subnet? | CycloneDDS SPDP discovery depends on multicast. | IT | Rule excerpt | SH-NET-001 | YES | | | UNANSWERED | | |
| SH-NET-008 | Wi-Fi coverage | P1 | Determines teacher access | Does the school have Wi-Fi, how many access points, and which floors/blocks does it cover? | Teachers need to reach the dashboard. The robot does not. | IT | AP inventory + floor plan | — | YES | | | UNANSWERED | | |
| SH-NET-009 | Wi-Fi coverage | P2 | Determines LoRa justification | **Is there any specific room, stairwell, corridor or hall where Wi-Fi does not reach?** | This single answer decides whether LoRa is justified at all. | IT | Named list of locations | SH-NET-008 | NO | | | UNANSWERED | | |
| SH-NET-010 | Wi-Fi policy | P1 | Permission required | May robot-associated devices join the existing school Wi-Fi? | If permitted, simplifies telemetry backup. | IT | Written policy | SH-NET-008 | NO | | | UNANSWERED | | |
| SH-NET-011 | Wi-Fi policy | P1 | Permission required | Is a **dedicated private Wi-Fi network** permitted for SHANA? | Preferred over joining the main WLAN for isolation. | IT | Written approval, `NEEDS_SCHOOL_PERM` | SH-NET-008 | NO | | | UNANSWERED | | |
| SH-NET-012 | Wi-Fi policy | P2 | Deployment option | Are additional access points permitted, and who installs and maintains them? | Needed if per-floor coverage gaps exist. | IT | Approval + owner | SH-NET-011 | NO | | | UNANSWERED | | |
| SH-NET-013 | Wi-Fi policy | P2 | Deployment option | Is wireless backhaul (mesh or bridge) permitted? | Fallback where cabling is impossible. | IT | Written approval | SH-NET-011 | NO | | | UNANSWERED | | |
| SH-NET-014 | Power | P2 | AP feasibility | Can access points be powered through approved PoE infrastructure? | Determines install cost. | IT / FAC | PoE inventory | SH-NET-012 | NO | | | UNANSWERED | | |
| SH-NET-015 | Local-only network | P1 | Design principle | Is a local-only network with no internet dependency permitted? | Core operation must not require external connectivity. | IT | Written approval | SH-NET-001 | YES | | | UNANSWERED | | |
| SH-NET-016 | Naming | P2 | Convenience | Can a local DNS name or fixed IP be authorised for the dashboard? | Teachers need a memorable address. | IT | DNS record approval | SH-NET-004 | NO | | | UNANSWERED | | |
| SH-NET-017 | Security | P1 | Mandatory | What authentication and encryption standard applies to school network credentials? | Determines credential provisioning for any wireless component. | IT | Policy reference | SH-NET-011 | NO | | | UNANSWERED | | |
| SH-NET-018 | Security | P0 | Mandatory | Are there school rules prohibiting storage of credentials in Git or documentation? | Applies to Wi-Fi keys and device credentials. | IT | Written rule | — | YES | | | UNANSWERED | | |
| SH-NET-019 | Coverage survey | P2 | Validates assumptions | What coverage survey method and standard would the school require before approval? | RSSI/PDR measurement at specific points. | IT | Survey method | SH-NET-012 | NO | | | UNANSWERED | | |
| SH-NET-020 | Coverage survey | P2 | Known weak areas | Are stairwells and elevators known Wi-Fi dead zones? | Relevant to telemetry continuity during vertical transit. | IT | Survey data | SH-NET-008 | NO | | | UNANSWERED | | |
| SH-NET-021 | Roaming | P3 | Affects telemetry continuity | How does the school WLAN handle roaming between access points? | Affects latency spikes during a mission. | IT | Controller config | SH-NET-008 | NO | | | UNANSWERED | | |
| SH-NET-022 | Resilience | P1 | Safety-relevant | Are there scheduled network outages, maintenance windows, or load-shedding patterns? | Determines whether missions can be planned around outages. | IT | Outage schedule | SH-NET-001 | NO | | | UNANSWERED | | |
| SH-NET-023 | Segmentation | P1 | Architecture requirement | Is a physically or logically separate robot network required or preferred? | Isolating the robot subnet protects the school LAN. | IT | Decision record | SH-NET-003 | YES | | | UNANSWERED | | |
| SH-NET-024 | Exhibition | P2 | Event-specific | For the exhibition, is a temporary isolated network preferable to the production network? | Reduces risk on a time-limited deployment. | IT / EXPO | Decision record | SH-NET-023 | NO | | | UNANSWERED | | |

---

# C. Local server, dashboard, authentication and security — `SH-SRV`

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-SRV-001 | Hosting | P0 | No server, no system | Which physical location will host the SHANA server, and is it approved? | AI Lab (CP-C3) is proposed; Makerspace Lab (CP-A1) is the alternate. | ADM | Room allocation, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-SRV-002 | Hosting | P0 | Minimum technical requirement | What CPU, RAM, disk and network ports does the available host have? | Must run Linux with at least two NICs. | LAB / IT | `lshw` output | SH-SRV-001 | YES | | | UNANSWERED | | |
| SH-SRV-003 | Hosting | P1 | Determines availability | Will the host run continuously, or only during school hours? | Recovery and mission windows depend on availability. | ADM | Written confirmation | SH-SRV-001 | YES | | | UNANSWERED | | |
| SH-SRV-004 | Hosting | P1 | Resilience | Is a UPS available at the chosen location? | A power cut mid-mission leaves the robot without a watchdog. | FAC | Confirmation | SH-SRV-001 | NO | | | UNANSWERED | | |
| SH-SRV-005 | Hosting | P2 | Deployment option | Does a NAS or existing local application server exist that could host SHANA? | Would reduce new hardware. | IT | Inventory | SH-SRV-001 | NO | | | UNANSWERED | | |
| SH-SRV-006 | Dashboard | P1 | Teacher-facing deliverable | What devices will teachers use to access the dashboard (desktop, tablet, phone)? | Determines minimum supported screen size and touch targets. | TCH | Survey | — | NO | | | UNANSWERED | | |
| SH-SRV-007 | Dashboard | P2 | Usability | Are teachers willing to use a web interface, or is a simpler mechanism preferred? | Adoption risk. | TCH | Survey | SH-SRV-006 | NO | | | UNANSWERED | | |
| SH-SRV-008 | Dashboard | P2 | Language | What languages must the dashboard support? | Indian school context may require regional language support. | ADM / TCH | Requirement | — | NO | | | UNANSWERED | | |
| SH-SRV-009 | Authentication | P0 | Security requirement | Who may hold a teacher account, and how are accounts provisioned? | Determines the identity model. | ADM | Account policy | SH-SRV-001 | YES | | | UNANSWERED | | |
| SH-SRV-010 | Authentication | P0 | Security requirement | Is school SSO available, or must SHANA run standalone accounts? | SSO integration is substantially more work. | IT | Confirmation | SH-SRV-009 | YES | | | UNANSWERED | | |
| SH-SRV-011 | Authentication | P0 | Safety requirement | Confirm that teacher roles must **not** include motor-control functionality. | Design principle; requires explicit acknowledgement. | ADM | Written confirmation | SH-SRV-009 | YES | | | UNANSWERED | | |
| SH-SRV-012 | Authentication | P1 | Access control | What roles are required beyond teacher and administrator? | E.g. custodian, facilities, viewer. | ADM | Role list | SH-SRV-009 | NO | | | UNANSWERED | | |
| SH-SRV-013 | Security | P1 | Credential hygiene | Where will credentials be stored, and how will they be rotated? | Must not be committed to Git. | INT / IT | Policy | SH-SRV-009 | NO | | | UNANSWERED | | |
| SH-SRV-014 | Security | P1 | Exposure control | Confirm the server will not be exposed to any public network. | Design principle. | ADM / IT | Written confirmation | SH-SRV-001 | YES | | | UNANSWERED | | |
| SH-SRV-015 | Security | P2 | Data protection | Is TLS required on the local dashboard? | Depends on school policy. | IT | Policy | SH-SRV-014 | NO | | | UNANSWERED | | |
| SH-SRV-016 | Storage | P1 | Audit requirement | What delivery and audit data must be retained, and for how long? | Determines database design and storage sizing. | ADM | Policy | — | YES | | | UNANSWERED | | |
| SH-SRV-017 | Storage | P1 | Privacy requirement | Must any data capable of identifying a person be excluded from logs? | Directly relevant to occupancy work. | ADM | Policy | SH-CCTV-020 | YES | | | UNANSWERED | | |
| SH-SRV-018 | Maintenance | P2 | Sustainability | Who will maintain the server, and what backup arrangement applies? | Long-term ownership. | ADM | Named owner | SH-SRV-001 | NO | | | UNANSWERED | | |
| SH-SRV-019 | Monitoring | P2 | Operations | What monitoring or alerting does the school expect for the server itself? | Disk, service health. | IT | Requirement | SH-SRV-001 | NO | | | UNANSWERED | | |
| SH-SRV-020 | Data model | P2 | Design input | Are there existing school systems SHANA must integrate with (library, timetable, directory)? | Integration scope. | IT | Inventory | — | NO | | | UNANSWERED | | |

---

# D. LoRa, ESP32, gateways, antennas and long-range telemetry — `SH-RAD`

> **Context and warning.** This category exists to establish whether radio hardware
> is **needed at all**. SHANA's robot link is wired Ethernet, so a WLAN outage
> does not stop the robot. Peer-reviewed evidence indicates Wi-Fi with one AP per
> floor outperforms LoRa for coverage in a multi-storey building. **Add radio
> hardware only to solve a demonstrated problem.**

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-RAD-001 | Justification | P0 | Do not build unjustified hardware | **Is there a demonstrated communication failure that radio hardware would solve?** | If no, this entire category is dropped. | INT / IT | Written justification | SH-NET-009 | YES | | | UNANSWERED | | |
| SH-RAD-002 | Justification | P1 | Avoids solving a non-problem | Would SHANA lose any safety capability if the WLAN failed? | The robot link is Ethernet; likely no. | INT | Written analysis | SH-NET-003 | NO | | | UNANSWERED | | |
| SH-RAD-003 | Justification | P2 | Clarifies scope | What specific information would a low-bandwidth channel carry? | Realistic payload is ~35 bytes per report. | INT | Data dictionary | SH-RAD-001 | NO | | | UNANSWERED | | |
| SH-RAD-004 | Data rate | P1 | Constrains design | What reporting interval is required for that data? | Duty cycle limits constrain the answer. | INT | Requirement | SH-RAD-003 | NO | | | UNANSWERED | | |
| SH-RAD-005 | Data rate | P2 | Correctness | Confirm that no video, LiDAR, or motor-control data is proposed over this link. | These are impossible or unsafe on LoRa. | INT | Design confirmation | SH-RAD-003 | NO | | | UNANSWERED | | |
| SH-RAD-006 | Hardware | P2 | If justified | What radio module and frequency variant would be used? | India's band is 865–868 MHz; EU868 defaults are non-compliant. | INT | Datasheet | SH-RAD-001 | NO | | | UNANSWERED | | |
| SH-RAD-007 | Hardware | P2 | If justified | Would the node be an ESP32 with an integrated LoRa radio? | Determines power and firmware effort. | INT | Part number | SH-RAD-006 | NO | | | UNANSWERED | | |
| SH-RAD-008 | Power | P2 | Mounting feasibility | What power supply and mounting would the gateway need? | Gateway must be mains-powered and accessible. | INT / FAC | Plan | SH-RAD-001 | NO | | | UNANSWERED | | |
| SH-RAD-009 | Power | P2 | Battery impact | What is the measured current draw of the chosen node in transmit and sleep? | Datasheet figures vary widely. | INT | Measurement | SH-RAD-007 | NO | | | UNANSWERED | | |
| SH-RAD-010 | Antennas | P2 | Determines coverage | Where would the gateway antenna be mounted, and what clearance exists? | Reinforced concrete and floor slabs dominate loss. | FAC | Location plan | SH-RAD-008 | NO | | | UNANSWERED | | |
| SH-RAD-011 | Antennas | P2 | Regulatory compliance | What antenna gain is used, and is TX power capped to remain within e.r.p. limits? | 500 mW e.r.p. = 29.15 dBm EIRP; many boards exceed this. | INT | Calculation | SH-RAD-006 | NO | | | UNANSWERED | | |
| SH-RAD-012 | Coverage | P1 | Determines viability | What measured floor attenuation exists in this building? | Published studies vary 5.5–21.7 dB per floor — a 4× spread. | INT | Site survey | SH-RAD-001 | NO | | | UNANSWERED | | |
| SH-RAD-013 | Coverage | P2 | Deployment detail | Are there reinforced-concrete floors, lift shafts, or metal obstructions? | Floor plate construction dominates penetration. | FAC | Building survey | SH-RAD-012 | NO | | | UNANSWERED | | |
| SH-RAD-014 | Interference | P3 | Quality of service | What other RF systems exist in the building? | 865–868 MHz is well separated from Wi-Fi. | IT | RF inventory | SH-RAD-001 | NO | | | UNANSWERED | | |
| SH-RAD-015 | Security | P1 | Mandatory if built | How would messages be authenticated and replay-protected? | Unauthenticated telemetry is a spoofing vector. | INT | Design | SH-RAD-001 | NO | | | UNANSWERED | | |
| SH-RAD-016 | Security | P0 | Mandatory if built | Confirm the radio path can **never** command the robot. | Telemetry must be inbound-only. | INT | Design confirmation | SH-RAD-015 | YES | | | UNANSWERED | | |
| SH-RAD-017 | Failure behaviour | P1 | Safety design | What does the system do when telemetry is lost? | Must degrade to safe behaviour, not to autonomous action. | INT | Design | SH-RAD-001 | NO | | | UNANSWERED | | |
| SH-RAD-018 | Regulatory | P0 | Legal prerequisite | Does the school require a radio possession authorisation under the 2026 Rules? | `NEEDS_VENDOR` / regulator clarification. | ADM | Written regulator reply | SH-RAD-001 | YES | | | UNANSWERED | | |
| SH-RAD-019 | Regulatory | P1 | Legal prerequisite | Who holds Equipment Type Approval for the exact module purchased? | ETA must be supplier-held. | INT | Supplier ETA certificate | SH-RAD-006 | NO | | | UNANSWERED | | |
| SH-RAD-020 | Permission | P1 | School approval | Does school policy permit any additional RF device on the network or premises? | Institutional layer above regulation. | ADM / IT | Written approval, `NEEDS_SCHOOL_PERM` | SH-RAD-018 | NO | | | UNANSWERED | | |

---

# E. CCTV, computer vision, occupancy estimation and privacy — `SH-CCTV`

> **Design constraints that apply to every question in this category.**
> No facial recognition. No identity recognition. No biometric identification. No
> persistent tracking of identifiable persons. No automatic misconduct
> accusations. No raw video stored by default. No bypassing CCTV authentication.
> No real student footage during research. Human review before any incident
> report. If the feed fails or confidence is low, corridor status is `UNKNOWN`,
> **not** `CLEAR`. **CCTV must never be the robot's only obstacle-avoidance system.**
>
> **Research finding.** ONVIF Profile M v1.1 makes *Metadata Streaming*
> **mandatory** and *Video Streaming* **conditional** — a conformant analytics
> device can stream zero video. But Profile M contains **no occurrence of
> "occupancy"**; it mandates only line-crossing counters. An integrator must call
> `GetSupportedRules` per device rather than assume. Separately, **OpenCV moved
> HOG and Haar people detection to `opencv_contrib` in OpenCV 5.0.0**, noting
> *"Modern deep learning-based detectors are both faster and more accurate."*
> **No published accuracy benchmark for OpenCV people detection in cluttered
> school corridors was found (NOT FOUND).** A **mmWave presence sensor** is
> likely the stronger option: ₹2,000–11,000, no image of any person ever formed.

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-CCTV-001 | Inventory | P1 | Determines whether integration is possible at all | What CCTV brand, model and management system does the school operate? | ONVIF and RTSP support vary widely by vendor. | CCTV / IT | System documentation | — | YES | | | UNANSWERED | | |
| SH-CCTV-002 | Inventory | P1 | Determines integration surface | Does the system expose an ONVIF-conformant interface? | ONVIF is the standard integration path. | CCTV | Conformance statement | SH-CCTV-001 | YES | | | UNANSWERED | | |
| SH-CCTV-003 | Inventory | P1 | Determines integration surface | Does the NVR expose RTSP streams, and is the URL format documented? | RTSP is the most common practical path. | CCTV | URL scheme documentation | SH-CCTV-001 | YES | | | UNANSWERED | | |
| SH-CCTV-004 | Inventory | P2 | Common failure mode | Do NVR channels re-stream, or is each camera point-to-point only? | Many NVRs do not re-stream; this breaks naive integration. | CCTV | Confirmation | SH-CCTV-003 | NO | | | UNANSWERED | | |
| SH-CCTV-005 | Inventory | P2 | Authentication path | What authentication does the system use — ONVIF Digest, RTSP Basic, HTTP Digest? | Determines credential handling. | CCTV | Documentation | SH-CCTV-002 | NO | | | UNANSWERED | | |
| SH-CCTV-006 | Coverage | P1 | Determines whether the approach is viable | Do cameras cover all relevant corridor segments on candidate routes? | Uncovered segments would need dedicated sensors. | CCTV / FAC | Coverage map | SH-CCTV-001 | YES | | | UNANSWERED | | |
| SH-CCTV-007 | Coverage | P2 | Quality | What are each camera's resolution, frame rate and field of view? | Determines detection feasibility at corridor scale. | CCTV | Specification | SH-CCTV-001 | NO | | | UNANSWERED | | |
| SH-CCTV-008 | Coverage | P2 | Real-world quality | Are there glare, low-light, or occlusion problems in the corridors? | Strongly affects any vision-based count. | CCTV / FAC | Site observation | SH-CCTV-007 | NO | | | UNANSWERED | | |
| SH-CCTV-009 | Analytics | P1 | Standards-native path | Does any camera emit occupancy or line-crossing **metadata events** without video? | ONVIF Profile M mandates metadata streaming; occupancy is device-dependent. | CCTV | `GetSupportedRules` output | SH-CCTV-002 | YES | | | UNANSWERED | | |
| SH-CCTV-010 | Analytics | P2 | Preferred architecture | Does the school already run any people-counting analytics? | Avoids duplicate effort. | CCTV | Product documentation | SH-CCTV-001 | NO | | | UNANSWERED | | |
| SH-CCTV-011 | Analytics | P2 | Effort estimate | If analytics must be built, who would build and maintain it? | A capability gap, not just a build task. | ADM | Named owner | SH-CCTV-009 | NO | | | UNANSWERED | | |
| SH-CCTV-012 | Analytics | P1 | Privacy-critical | Confirm the design computes **anonymous counts only**, never identities. | Hard design constraint. | ADM / CCTV | Written confirmation | — | YES | | | UNANSWERED | | |
| SH-CCTV-013 | Analytics | P0 | Privacy-critical | Confirm **no facial recognition or biometric identification** will be performed. | Hard prohibition. | ADM / CCTV | Written confirmation | — | YES | | | UNANSWERED | | |
| SH-CCTV-014 | Analytics | P0 | Privacy-critical | Confirm frames are processed in a stream and **discarded**, with no raw video written to disk by default. | Core privacy guarantee. | ADM / CCTV | Design confirmation | — | YES | | | UNANSWERED | | |
| SH-CCTV-015 | Analytics | P1 | Legal requirement | What retention and deletion requirements apply to derived occupancy data? | Derived data may still be personal data under some regimes. | ADM | Policy | SH-CCTV-012 | YES | | | UNANSWERED | | |
| SH-CCTV-016 | Legal | P0 | Legal prerequisite | Does school policy permit **any** analytics access to CCTV feeds? | Without this, the entire category is closed. | ADM / CCTV | Written policy, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-CCTV-017 | Legal | P0 | Legal prerequisite | Who is the Data Fiduciary for CCTV under applicable law, and have they approved this use? | DPDP Act 2023 assigns this role. | ADM | Written approval | SH-CCTV-016 | YES | | | UNANSWERED | | |
| SH-CCTV-018 | Legal | P1 | Compliance check | Is a privacy impact assessment required before processing? | Many institutional policies require one. | ADM | Policy | SH-CCTV-016 | YES | | | UNANSWERED | | |
| SH-CCTV-019 | Legal | P2 | Note audio | Do the CCTV streams carry **audio**? | The pipeline must explicitly not decode it. | CCTV | Specification | SH-CCTV-001 | NO | | | UNANSWERED | | |
| SH-CCTV-020 | Legal | P1 | Compliance check | Is processing of identifiable-person video by a school covered by an exemption, and does corridor routing qualify? | DPDP Rules 2025 exempt educational institutions for educational activity or child safety — corridor routing may be neither. | ADM | Legal/policy opinion | SH-CCTV-016 | YES | | | UNANSWERED | | |
| SH-CCTV-021 | Governance | P0 | Safety requirement | Confirm the robot will **never** rely on CCTV for obstacle avoidance. | Independence requirement. | ADM / INT | Design confirmation | — | YES | | | UNANSWERED | | |
| SH-CCTV-022 | Governance | P1 | Operational | Who may view occupancy results, and are they ever shown to students? | Results must not enable identification. | ADM | Access policy | SH-CCTV-016 | NO | | | UNANSWERED | | |
| SH-CCTV-023 | Governance | P1 | Accountability | Confirm occupancy data will **never** generate an automatic misconduct or disciplinary report. | Hard prohibition. | ADM | Written confirmation | SH-CCTV-012 | YES | | | UNANSWERED | | |
| SH-CCTV-024 | Alternative | P1 | Likely better option | Has a **mmWave or ToF people-presence sensor** been considered instead of CCTV? | Same CLEAR/MODERATE/BUSY answer with no image ever formed. | INT / ADM | Options comparison | SH-CCTV-016 | NO | | | UNANSWERED | | |

<!-- CONTINUED-IN-PART-2 -->---

# F. Corridor congestion and dynamic route selection — `SH-CONG`

> **Design constraint.** A route planner may only use a **verified graph of
> approved corridor segments**. It must not infer routes, must not treat graph
> edges as permission to enter unverified areas, and must not choose stairs or
> lifts merely because an edge exists. **Do not claim SHANA can avoid a busy
> corridor unless both the alternate route and the navigation capability have
> been verified.** If every safe route is unavailable, the correct response is to
> stop at a safe location and request human assistance.

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-CONG-001 | Measurement | P1 | Determines whether congestion awareness is possible | How will corridor occupancy be measured, and by what method? | Options range from CCTV analytics to presence sensors to manual input. | INT | Written design | SH-CCTV-016 | YES | | | UNANSWERED | | |
| SH-CONG-002 | Measurement | P2 | Feasibility of source | What is the expected update interval for an occupancy estimate? | Determines whether stale data dominates. | INT | Requirement | SH-CONG-001 | NO | | | UNANSWERED | | |
| SH-CONG-003 | Thresholds | P1 | Determines routing behaviour | What occupancy thresholds define CLEAR, MODERATE, BUSY and UNKNOWN? | Must be experimentally set, not guessed. | INT | Threshold table | SH-CONG-001 | YES | | | UNANSWERED | | |
| SH-CONG-004 | Thresholds | P1 | Safety requirement | Confirm that low confidence yields `UNKNOWN`, never `CLEAR`. | Prevents treating a failed sensor as an empty corridor. | INT | Design confirmation | SH-CONG-003 | YES | | | UNANSWERED | | |
| SH-CONG-005 | Persistence | P2 | Affects route stability | How long does a corridor remain marked busy after the last positive reading? | Determines hysteresis and flapping. | INT | Design parameter | SH-CONG-003 | NO | | | UNANSWERED | | |
| SH-CONG-006 | Staleness | P1 | Safety requirement | How is stale occupancy data handled — expired, ignored, or treated as UNKNOWN? | Stale data must never authorise traversal. | INT | Design | SH-CONG-005 | YES | | | UNANSWERED | | |
| SH-CONG-007 | Graph | P0 | No graph, no routing | What is the verified corridor segment graph, with nodes, edges and floors? | Must be surveyed and human-verified before any routing. | FAC / INT | Annotated floor plan | — | YES | | | UNANSWERED | | |
| SH-CONG-008 | Graph | P1 | Determines whether alternates exist | For each segment, is a physically verified alternate route available? | If not, a busy corridor must become a hard block, not a preference. | FAC / INT | Route survey | SH-CONG-007 | YES | | | UNANSWERED | | |
| SH-CONG-009 | Graph | P0 | Authorisation | Which corridor segments are authorised for robot access at all? | Restricted and fire-route areas must be excluded. | ADM / FAC | Authorised segment list | SH-CONG-007 | YES | | | UNANSWERED | | |
| SH-CONG-010 | Graph | P1 | Safety requirement | Confirm the planner must never route through a fire or emergency evacuation route. | Legal and safety constraint. | FAC | Confirmation | SH-CONG-009 | YES | | | UNANSWERED | | |
| SH-CONG-011 | Routing | P1 | Routing behaviour | Does a BUSY route become blocked, or merely less desirable? | Determines whether alternates must always exist. | INT | Design decision | SH-CONG-008 | YES | | | UNANSWERED | | |
| SH-CONG-012 | Routing | P2 | Stability | How is oscillation between two routes prevented? | Requires hysteresis or a commitment policy. | INT | Design | SH-CONG-011 | NO | | | UNANSWERED | | |
| SH-CONG-013 | Routing | P0 | Safety requirement | Confirm that a route becoming unsafe **during** traversal stops the robot safely. | Pre-emptive checking is insufficient. | INT | Design | SH-CONG-011 | YES | | | UNANSWERED | | |
| SH-CONG-014 | Routing | P1 | Mid-mission change | May the planner reroute mid-mission, or only between missions? | Mid-mission rerouting in an unverified area is a risk. | INT | Design decision | SH-CONG-013 | YES | | | UNANSWERED | | |
| SH-CONG-015 | Prediction | P2 | Optional enhancement | Can school timetable data be used to predict congestion? | Corridors peak at class change, not uniformly. | ADM | Timetable export | SH-TIME-001 | NO | | | UNANSWERED | | |
| SH-CONG-016 | Prediction | P2 | Known failure mode | Where do students congregate unpredictably? | Lifts, canteens, notice boards, stairwells. | TCH / FAC | Observation notes | SH-CONG-007 | NO | | | UNANSWERED | | |
| SH-CONG-017 | Human override | P1 | Operational requirement | Can an authorised staff member close a corridor from the dashboard? | Manual closure must be available regardless of sensing. | INT | Design | SH-CONG-007 | YES | | | UNANSWERED | | |
| SH-CONG-018 | Human override | P1 | Operational requirement | Can an authorised person override congestion routing entirely? | During a demonstration, manual control must be possible. | INT | Design | SH-CONG-017 | YES | | | UNANSWERED | | |
| SH-CONG-019 | Degraded mode | P1 | Safety requirement | What does SHANA do when no safe route exists? | Must stop at a safe location and request assistance. | INT | Design | SH-CONG-013 | YES | | | UNANSWERED | | |
| SH-CONG-020 | Validation | P2 | Evidence quality | How will occupancy accuracy be measured, and against what ground truth? | Requires manual counts during a controlled trial. | INT | Test plan | SH-CONG-003 | NO | | | UNANSWERED | | |

---

# G. Wet floors, rain, lighting and environmental hazards — `SH-ENV`

> **Design constraint.** RGB and depth cameras **cannot** determine surface
> friction. Visual detection of water is **not** proof that a floor is safe.
> Any hazard design must be built around *declared* hazard states from
> authorised humans, with sensing used only as an advisory prompt for human
> confirmation.

### Proposed hazard response matrix

| State | Proposed safe behaviour | Notification | Routing consequence | Resume condition |
|---|---|---|---|---|
| `CONFIRMED_SAFE` | Proceed at reduced speed | None | Route allowed | Ongoing |
| `POSSIBLE_HAZARD` | **Stop.** Do not proceed | Alert custodian, request human confirmation | Segment excluded pending confirmation | Human confirms safe, with time limit |
| `CONFIRMED_CLOSED` | Stop and hold or return | Notify facilities + teachers | Segment excluded | Named person re-opens |
| `UNKNOWN` | **Treat as `POSSIBLE_HAZARD`.** Never as safe | Advisory only | Segment excluded if staleness threshold exceeded | Fresh confirming reading or human report |
| `SENSOR_FAILURE` | Degrade to human-only reporting | Fault alert to custodian | Conservative: exclude unverified segments | Sensor restored and human-confirmed |

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-ENV-001 | Governance | P0 | Authority to declare a hazard | **Who is authorised to declare a corridor closed or hazardous?** | Without this, hazard handling has no basis. | ADM / FAC | Written authority list, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-ENV-002 | Governance | P1 | Primary hazard source | Can housekeeping staff publish a cleaning/wet-floor schedule SHANA can read? | The single most useful hazard input in a school. | FAC | Schedule format | SH-ENV-001 | YES | | | UNANSWERED | | |
| SH-ENV-003 | Governance | P1 | Manual input path | Can staff report a hazard from a phone or the dashboard? | Faster than waiting for a schedule update. | FAC / INT | Process design | SH-ENV-001 | YES | | | UNANSWERED | | |
| SH-ENV-004 | Governance | P1 | Signal integration | Can wet-floor warning signs be integrated, or are they purely visual to humans? | Signs are not machine-readable. | FAC | Observation | SH-ENV-001 | NO | | | UNANSWERED | | |
| SH-ENV-005 | Sensing | P1 | Determine what is detectable | What floor hazards can the onboard camera realistically detect, and at what distance? | Water reflections are unreliable evidence. | INT | Test plan | SH-GO2-015 | YES | | | UNANSWERED | | |
| SH-ENV-006 | Sensing | P1 | Determine what is detectable | What floor hazards can the depth sensor detect, and what does it miss? | Transparent or specular surfaces defeat depth sensors. | INT | Test plan | SH-GO2-004 | YES | | | UNANSWERED | | |
| SH-ENV-007 | Sensing | P0 | Safety limitation | Confirm that **no** camera-based method may be treated as proof of floor safety. | Hard design constraint. | INT / ADM | Design confirmation | SH-ENV-005 | YES | | | UNANSWERED | | |
| SH-ENV-008 | Sensing | P2 | Reduces false alarms | Can wet-floor signs be detected visually as a *prompt to confirm*? | Advisory use only. | INT | Test plan | SH-ENV-007 | NO | | | UNANSWERED | | |
| SH-ENV-009 | Environment | P2 | Outdoor exposure | Are any routes open to rain, wind, or direct sun? | Go2 operating range is 5–35 °C; not weatherproof. | FAC | Site survey | SH-CONG-007 | NO | | | UNANSWERED | | |
| SH-ENV-010 | Environment | P1 | Safety decision | Should exposed or outdoor routes be disabled entirely? | Prevents a foreseeable hazard class. | ADM / INT | Decision record | SH-ENV-009 | YES | | | UNANSWERED | | |
| SH-ENV-011 | Environment | P1 | Operating limit | What is the corridor lighting like at all times, including emergency lighting only? | Low light degrades every sensor. | FAC | Survey | SH-CONG-007 | YES | | | UNANSWERED | | |
| SH-ENV-012 | Environment | P2 | Detects outages | Can a lighting outage be detected and reported to the custodian? | A useful safety trigger. | INT | Design | SH-ENV-011 | NO | | | UNANSWERED | | |
| SH-ENV-013 | Environment | P2 | Known hazard | Are there construction materials, temporary cables, or damaged flooring on any route? | Physical, not optical, hazard. | FAC | Survey | SH-CONG-007 | NO | | | UNANSWERED | | |
| SH-ENV-014 | Route policy | P1 | Fallback definition | For each segment, is a hazard-free alternate route verified? | Without one, a hazard must block the mission. | FAC / INT | Route survey | SH-CONG-008 | YES | | | UNANSWERED | | |
| SH-ENV-015 | Route policy | P0 | Safety requirement | Confirm the robot **stops** rather than attempting an unverified detour when a hazard is detected. | A detour through an unverified area is a hazard itself. | INT | Design | SH-ENV-014 | YES | | | UNANSWERED | | |
| SH-ENV-016 | Route policy | P1 | All-affected case | What happens when every route is affected by hazards? | Must resolve to safe-stop and human request. | INT | Design | SH-ENV-015 | YES | | | UNANSWERED | | |
| SH-ENV-017 | Humans | P1 | Assistance pathway | Can the robot request help from a nearby teacher, and by what mechanism? | Human assistance is the intended fallback. | ADM / TCH | Process design | SH-ENV-001 | YES | | | UNANSWERED | | |
| SH-ENV-018 | Validation | P2 | Evidence quality | How will false positives and false negatives be measured? | Required before trusting any sensing. | INT | Test plan | SH-ENV-005 | NO | | | UNANSWERED | | |
| SH-ENV-019 | Validation | P1 | Supervised testing | What supervised hazard tests are required, and who approves them? | No unsupervised hazard testing in an occupied school. | ADM / LAB | Test plan, `NEEDS_SCHOOL_PERM` | SH-ENV-018 | YES | | | UNANSWERED | | |
| SH-ENV-020 | Governance | P1 | Resume authority | Who approves resuming a route after a hazard is cleared? | Prevents automatic resumption. | ADM / FAC | Written authority list | SH-ENV-001 | YES | | | UNANSWERED | | |

---

# H. Obstacles, construction, maintenance and blocked corridors — `SH-BLOCK`

> **Design constraint.** Authorised staff must be able to mark a segment
> unavailable through an approved workflow. The planner must reject closed edges,
> must not resend the robot into unverified areas, must treat stale closure data
> conservatively, must stop safely when no route remains, must notify a responsible
> person, and **must not permit automatic resumption**.

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-BLOCK-001 | Governance | P0 | Authority to close a route | Who is authorised to mark a corridor segment unavailable? | Must be named, not implied. | ADM / FAC | Authority list, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-BLOCK-002 | Governance | P1 | Operational workflow | Through what mechanism will closures be published — dashboard, app, or schedule? | Determines the operator's actual workflow. | FAC / INT | Process design | SH-BLOCK-001 | YES | | | UNANSWERED | | |
| SH-BLOCK-003 | Governance | P1 | Closed-loop requirement | Does closure require an expiry time, or persist until explicitly reopened? | Expiry risks auto-reopening a still-closed route. | INT | Design decision | SH-BLOCK-002 | YES | | | UNANSWERED | | |
| SH-BLOCK-004 | Governance | P0 | Safety requirement | Confirm **no automatic resumption** after a closure is cleared. | Only a named person may reopen. | INT / ADM | Design confirmation | SH-BLOCK-003 | YES | | | UNANSWERED | | |
| SH-BLOCK-005 | Inventory | P2 | Known obstructions | Which permanent obstructions exist — cleaning trolleys, fire equipment, furniture? | Must be in the route graph as hazards. | FAC | Survey | SH-CONG-007 | NO | | | UNANSWERED | | |
| SH-BLOCK-006 | Inventory | P2 | Known obstructions | Which temporary obstructions recur — deliveries, bins, stacked chairs? | Informs the baseline obstruction map. | FAC | Survey | SH-BLOCK-005 | NO | | | UNANSWERED | | |
| SH-BLOCK-007 | Inventory | P1 | Behavioural hazard | Where do student groups routinely gather? | Congestion source not visible to a single sensor. | TCH / FAC | Observation notes | SH-CONG-016 | YES | | | UNANSWERED | | |
| SH-BLOCK-008 | Routing | P0 | Planner constraint | Confirm the planner rejects any closed edge, even if it is the shortest path. | Core routing invariant. | INT | Design | SH-BLOCK-002 | YES | | | UNANSWERED | | |
| SH-BLOCK-009 | Routing | P1 | Unknown-area handling | What happens if the planner reaches a node outside all verified maps? | Must stop, not explore. | INT | Design | SH-BLOCK-008 | YES | | | UNANSWERED | | |
| SH-BLOCK-010 | Routing | P1 | Stale data | How long may a closure remain unconfirmed before it expires? | Expiry risks premature reopening. | INT | Design parameter | SH-BLOCK-003 | YES | | | UNANSWERED | | |
| SH-BLOCK-011 | Routing | P1 | Dead-end handling | What is the safe behaviour when no route remains at any point? | Must resolve to safe-stop and notification. | INT | Design | SH-BLOCK-008 | YES | | | UNANSWERED | | |
| SH-BLOCK-012 | Routing | P0 | Safety requirement | Confirm mid-mission route changes are only permitted between verified segments. | Prevents unplanned entry into unverified space. | INT | Design | SH-BLOCK-011 | YES | | | UNANSWERED | | |
| SH-BLOCK-013 | Construction | P2 | Realistic scenario | Is construction or maintenance planned during the project period? | Affects the map's validity. | ADM / FAC | Programme | SH-MAP-012 | NO | | | UNANSWERED | | |
| SH-BLOCK-014 | Construction | P2 | Map invalidation | Who notifies the team when the building layout changes? | Without this, maps silently become unsafe. | ADM | Process | SH-BLOCK-013 | NO | | | UNANSWERED | | |
| SH-BLOCK-015 | Fire safety | P0 | Legal requirement | Which routes are fire evacuation routes and are absolutely excluded? | Legal and safety exclusion. | FAC | Fire plan | — | YES | | | UNANSWERED | | |
| SH-BLOCK-016 | Fire safety | P0 | Safety requirement | How will the robot behave on a fire alarm or evacuation? | Evacuation may be announced unexpectedly. | ADM / FAC | Procedure | SH-BLOCK-015 | YES | | | UNANSWERED | | |
| SH-BLOCK-017 | Fire doors | P1 | Physical constraint | Are there fire doors on candidate routes, and are they normally held open? | Fire doors may close and block or trap the robot. | FAC | Site survey | SH-CONG-007 | YES | | | UNANSWERED | | |
| SH-BLOCK-018 | Notification | P1 | Accountability | Who is notified when the robot stops due to a blocked route? | A blocked robot unattended is a hazard. | INT | Design | SH-BLOCK-011 | YES | | | UNANSWERED | | |
| SH-BLOCK-019 | Notification | P2 | Response model | What is the expected response time for a human to attend a stopped robot? | Determines whether stopping is safe. | ADM | Agreement | SH-BLOCK-018 | NO | | | UNANSWERED | | |
| SH-BLOCK-020 | Validation | P2 | Evidence quality | How will blocked-route handling be tested? | Requires a supervised obstruction trial. | INT | Test plan | SH-BLOCK-011 | NO | | | UNANSWERED | | |

---

# I. Elevators, lift control, authorised interfaces and service support — `SH-LIFT`

> **Standing constraints.** Do **not** contact the manufacturer or service
> provider. Do **not** open panels, connect to lift wiring, modify the controller,
> bypass interlocks, attach a servo, or build a button pusher. **Do not assume a
> lift supports remote control merely because it has an electronic controller or a
> maintenance application.** Any physical button actuator would require explicit
> permission from both the manufacturer **and** the school, and an approved
> engineering procedure. Assume that permission will not be granted.
>
> **Research findings that must be carried into this section.** No Johnson public
> API, developer portal, SDK or robot-integration product was found (high-confidence
> negative). Johnson's **EyeRIS** is a QR-code contactless call system — the only
> sanctioned path found. Johnson's **WATCH** IoT is monitoring-only with no inbound
> command path. Statutory position: Maharashtra Act 2017 s.10, Delhi/Bombay Rules
> 6(v)/(xv) and Tamil Nadu Act 1997 s.15 all require prior written permission of an
> **Electrical Inspector (Lifts)** for any addition or alteration. **Otis v. Rashmi
> Handa**, Supreme Court of India, 29 July 2026, apportioned 5% liability to the
> occupier for failure of oversight.
>
> **Do not assume permission for a physical button actuator. If an approved
> external actuator is not supported, the alternatives are: (a) human-assisted lift
> operation, (b) staff-controlled lift calls, (c) manufacturer-supported integration,
> (d) avoiding vertical transitions in the initial pilot.**

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-LIFT-001 | Identity | P0 | Determines the counterparty | What is the exact lift manufacturer and model? | Common confusion exists between related entities. | FAC / LIFT | Nameplate photo | — | YES | | | UNANSWERED | | |
| SH-LIFT-002 | Identity | P0 | Determines the applicable standard | Which Indian standard were the lifts built and certified to? | IS 14447 and IS 6914 are **not** lift standards; the family is IS 17900. | LIFT | Type approval certificate | SH-LIFT-001 | YES | | | UNANSWERED | | |
| SH-LIFT-003 | Identity | P1 | Determines documentation availability | What is the installation date and is documentation available? | Older installations may have fewer integration options. | FAC | Installation record | SH-LIFT-001 | NO | | | UNANSWERED | | |
| SH-LIFT-004 | Identity | P1 | Determines who is approached | Who is the current authorised maintenance contractor? | All technical questions must route through them. | FAC | AMC reference | — | YES | | | UNANSWERED | | |
| SH-LIFT-005 | Physical | P0 | Determines whether the robot fits | What are the car internal dimensions and the door clear opening width? | Robot is 700 × 310 mm; a school lift door is typically ~900 mm. | LIFT / FAC | Measured dimensions, drawings | SH-LIFT-001 | YES | | | UNANSWERED | | |
| SH-LIFT-006 | Physical | P1 | Often the real barrier | What is the threshold sill gap and levelling accuracy? | Small uneven gaps cause trips. | LIFT | Specification | SH-LIFT-005 | YES | | | UNANSWERED | | |
| SH-LIFT-007 | Physical | P1 | Manoeuvring feasibility | Is there enough car depth to enter, **turn around, and exit facing the door**? | A quadruped must reverse out or turn in a confined space. | LIFT / FAC | Measured dimensions | SH-LIFT-005 | YES | | | UNANSWERED | | |
| SH-LIFT-008 | Physical | P1 | Load check | What is the rated car load and current typical occupancy? | 15 kg robot against an 8–10 person car. | LIFT | Specification | SH-LIFT-005 | YES | | | UNANSWERED | | |
| SH-LIFT-009 | Interfaces | P0 | Core feasibility question | Does the manufacturer offer **any** supported interface for a third-party device or robot to place a hall call or destination call? | This determines whether autonomous lift use is even conceivable. | LIFT | Product documentation, `NEEDS_VENDOR` | SH-LIFT-001 | YES | | | UNANSWERED | | |
| SH-LIFT-010 | Interfaces | P1 | Distinguish control from monitoring | Does the lift expose a building-management or monitoring interface? | Monitoring is not control. | LIFT | Product documentation | SH-LIFT-009 | YES | | | UNANSWERED | | |
| SH-LIFT-011 | Interfaces | P1 | Do not assume maintenance app = control | Does a remote-control or maintenance application exist, and is it **maintenance-staff only**? | A maintenance app is not a third-party API. | LIFT | Application documentation | SH-LIFT-009 | YES | | | UNANSWERED | | |
| SH-LIFT-012 | Interfaces | P1 | Determine integration surface | Is there a documented SDK or integration protocol available to third parties? | Not expected, but must be confirmed. | LIFT | Documentation | SH-LIFT-009 | YES | | | UNANSWERED | | |
| SH-LIFT-013 | Interfaces | P2 | Security posture | Does the lift have network access, and is that network isolated or security-restricted? | Affects any integration feasibility. | LIFT / IT | Network description | SH-LIFT-009 | NO | | | UNANSWERED | | |
| SH-LIFT-014 | Interfaces | P0 | Cheapest possible win | Is the manufacturer's **EyeRIS** contactless QR call system available for retrofit on these lifts? | The only sanctioned integration path identified. | LIFT | Written response, `NEEDS_VENDOR` | SH-LIFT-001 | YES | | | UNANSWERED | | |
| SH-LIFT-015 | Interfaces | P0 | Determines whether EyeRIS is usable at all | Is the EyeRIS QR element on the landing panel a passive label or an active electronic component? | An active element would make fitting it a modification. | LIFT | Written response | SH-LIFT-014 | YES | | | UNANSWERED | | |
| SH-LIFT-016 | Interfaces | P0 | Determines robot usability | Can EyeRIS be provisioned to a **non-human client** such as an unattended device? | EyeRIS is documented as authenticating a human phone. | LIFT | Written response, `NEEDS_VENDOR` | SH-LIFT-014 | YES | | | UNANSWERED | | |
| SH-LIFT-017 | Interfaces | P1 | Would enable verification | Does the lift expose car position, door state or direction of travel as readable status? | Needed to confirm arrival and safe exit. | LIFT | Documentation | SH-LIFT-009 | NO | | | UNANSWERED | | |
| SH-LIFT-018 | Permission | P0 | Legal prerequisite | Does the school permit **any** form of lift integration, and under what written authority? | Electrical Inspector permission may be required by statute. | ADM | Written permission, `NEEDS_SCHOOL_PERM` | SH-LIFT-009 | YES | | | UNANSWERED | | |
| SH-LIFT-019 | Permission | P0 | Explicitly must not be assumed | **Does the manufacturer and school explicitly permit a purpose-built, externally mounted actuator to press the existing lift buttons?** | Do not assume permission. Do not build one. | LIFT / ADM | Written statement, `NEEDS_VENDOR` | — | YES | | | UNANSWERED | | |
| SH-LIFT-020 | Permission | P0 | If actuator is refused | If an approved external actuator is **not** supported, which alternatives are acceptable? | Human-assisted, staff-controlled, or avoid vertical transit. | ADM / LIFT | Written decision | SH-LIFT-019 | YES | | | UNANSWERED | | |
| SH-LIFT-021 | Operation | P0 | Safety requirement | Is unattended robot travel in the lift permitted in any circumstances? | The defining safety question for vertical transit. | ADM / LIFT | Written decision | SH-LIFT-018 | YES | | | UNANSWERED | | |
| SH-LIFT-022 | Operation | P1 | Most likely Phase 2 design | Is a human escort required for every trip, and who is the named escort? | Every comparable operator has converged on this model. | ADM | Protocol agreement | SH-LIFT-021 | YES | | | UNANSWERED | | |
| SH-LIFT-023 | Operation | P1 | Occupancy interaction | Can the robot wait safely in a lift lobby without obstructing passengers? | Lobby space is usually the binding constraint. | FAC / LIFT | Site observation | SH-LIFT-005 | YES | | | UNANSWERED | | |
| SH-LIFT-024 | Operation | P1 | Door interaction | Can the lift's door and passenger sensors accommodate a 15 kg quadruped? | Door closing on the robot is a documented failure mode. | LIFT | Confirmation | SH-LIFT-005 | YES | | | UNANSWERED | | |
| SH-LIFT-025 | Fault handling | P0 | Highest-consequence risk | What is the approved emergency or fault workflow if the lift stops with the robot inside? | Battery is dead in ≤4 h; rescue takes far longer. | ADM / LIFT | Procedure, `NEEDS_SCHOOL_PERM` | SH-LIFT-021 | YES | | | UNANSWERED | | |
| SH-LIFT-026 | Assurance | P1 | Risk and compliance | Would lift integration affect insurance, warranty, or statutory certification? | The occupier inherits liability per *Otis v. Rashmi Handa*. | ADM / LIFT | Written confirmation | SH-LIFT-018 | YES | | | UNANSWERED | | |

---

# J. Stairs, ramps and vertical navigation — `SH-STAIR`

> **Research findings that must be carried into this section.**
> ✅ The Go2's documented maximum step height is **160 mm**.
> ✅ Unitree's manual: *"it is forbidden to use the robot on the occasions where the
> terrain is complicated… steps higher than 16cm."*
> ✅ `SwitchGait`, the **only** documented stair-specific API, existed on firmware
> **< V1.1.6** and is **absent** from the current motion interface. On current
> firmware, stair capability is an emergent property of `ClassicWalk` with no
> documented step-height parameter or safety envelope.
> ✅ Independent reports consistently describe **descent as the worse case**.
> ✅ At low battery the robot **crouches within 10 minutes** — *a crouch on a
> staircase is a fall.*
> **No stair test may occur during this research-only phase.**

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-STAIR-001 | Geometry | P0 | Decides whether stairs are feasible at all | **Measure the riser height (mm), tread depth (mm) and step count for every staircase on any candidate route.** | The Go2 limit is 160 mm. A 170 mm riser is outside the manufacturer envelope. | FAC | Tape measurements, photos | — | YES | | | UNANSWERED | | |
| SH-STAIR-002 | Geometry | P0 | Manoeuvring feasibility | What is the clear width of each staircase, accounting for handrail projection? | The robot is 310 mm wide but needs swing room for leg placement. | FAC | Tape measurement | SH-STAIR-001 | YES | | | UNANSWERED | | |
| SH-STAIR-003 | Geometry | P1 | Turning feasibility | Is there sufficient landing space at each floor for the robot to turn? | Turning on a landing is a common failure point. | FAC | Measurement | SH-STAIR-001 | YES | | | UNANSWERED | | |
| SH-STAIR-004 | Geometry | P2 | Surface condition | What is the stair surface material and edge (nosing) condition? | Polished terrazzo is the worst case; the manual names slip as a failure mode. | FAC | Site survey, photos | SH-STAIR-001 | NO | | | UNANSWERED | | |
| SH-STAIR-005 | Geometry | P2 | Visual condition | Is stair edge contrast adequate for the robot's cameras? | Low-contrast edges impede perception. | FAC | Site survey | SH-STAIR-004 | NO | | | UNANSWERED | | |
| SH-STAIR-006 | Environment | P1 | Safety assessment | What is the lighting on each staircase? | Low light degrades all sensing. | FAC | Survey | SH-STAIR-001 | YES | | | UNANSWERED | | |
| SH-STAIR-007 | Environment | P1 | Traffic conflict | What is normal pedestrian traffic on each staircase, and at what times? | People on stairs is the dominant hazard. | FAC / TCH | Observation | SH-STAIR-001 | YES | | | UNANSWERED | | |
| SH-STAIR-008 | Environment | P1 | Legal restriction | Are any staircases emergency exits or fire evacuation routes? | Absolutely excluded from autonomous use. | FAC | Fire plan | — | YES | | | UNANSWERED | | |
| SH-STAIR-009 | Environment | P2 | Physical constraint | Is there handrail clearance sufficient for the robot's body? | Affects turning and lifting. | FAC | Survey | SH-STAIR-002 | NO | | | UNANSWERED | | |
| SH-STAIR-010 | Ramp | P2 | Alternative to stairs | Are there ramps or gently sloped corridors on any required route? | A ramp avoids the stair problem entirely. | FAC | Site survey | SH-STAIR-001 | NO | | | UNANSWERED | | |
| SH-STAIR-011 | Ramp | P2 | Slope check | What is the gradient of any candidate ramp? | Go2 is rated to 40°, but steep ramps with a payload are untested. | FAC | Measurement | SH-STAIR-010 | NO | | | UNANSWERED | | |
| SH-STAIR-012 | Restriction | P0 | Scope question | Does the route survey show **any** route to the required destinations that avoids all stairs and lifts? | If yes, the preferred architecture avoids vertical transit entirely. | FAC / INT | Route survey | SH-CONG-007 | YES | | | UNANSWERED | | |
| SH-STAIR-013 | Restriction | P0 | Legal prerequisite | Does school policy permit robot operation on staircases at all? | May be prohibited outright. | ADM | Written policy, `NEEDS_SCHOOL_PERM` | SH-STAIR-012 | YES | | | UNANSWERED | | |
| SH-STAIR-014 | Restriction | P0 | Manufacturer guidance | Does manufacturer guidance permit the proposed stair use with payload? | Manufacturer forbids complex terrain; payload derating is unpublished. | VENDOR | Written response, `NEEDS_VENDOR` | SH-STAIR-013 | YES | | | UNANSWERED | | |
| SH-STAIR-015 | Capability | P1 | Determine testability | Is any stair gait reachable from the approved SDK on the installed firmware? | `SwitchGait` is absent on ≥1.1.6. | INT | Source citation | SH-GO2-002 | YES | | | UNANSWERED | | |
| SH-STAIR-016 | Testing | P0 | Approval required | What school and insurer approval is required before **any** stair test? | No test may occur in this phase. | ADM | Written approval, `NEEDS_SCHOOL_PERM` | SH-STAIR-013 | YES | | | UNANSWERED | | |
| SH-STAIR-017 | Testing | P1 | Test design | What supervised, unloaded stair test protocol would the school approve? | Test ladder with hold points; unloaded only initially. | ADM / LAB | Protocol, `NEEDS_SCHOOL_PERM` | SH-STAIR-016 | YES | | | UNANSWERED | | |
| SH-STAIR-018 | Testing | P1 | Gating capability | If a stair test were approved, which capability is the pass/fail gate? | Forward descent is the hardest and least verified. | INT | Test plan | SH-STAIR-017 | NO | | | UNANSWERED | | |
| SH-STAIR-019 | Alternative | P1 | Safer fallback | Is a human-assisted alternative available where stairs cannot be used? | May be the permanent design. | ADM | Process design | SH-STAIR-012 | YES | | | UNANSWERED | | |
| SH-STAIR-020 | Decision | P0 | Scope decision | On current evidence, is stair operation **excluded from the initial pilot**? | Default recommendation: yes. | ADM / INT | Decision record | SH-STAIR-012 | YES | | | UNANSWERED | | |---

# K. Maps, named routes, localisation, checkpoints and ArUco markers — `SH-MAP`

> **Research finding that must be carried into this section.** ✅ **The Go2 EDU has
> no sensor that can determine which floor of a building it is on.** Every message
> field across the platform was enumerated: there is **no altitude, no pressure, no
> barometer, and no floor field**. `position[3]` is dead-reckoned odometry.
> `body_height` is the commanded stance height above ground, not altitude.
> `HeightMap.origin` is `float32[2]` — **XY only, no Z datum.** The built-in LiDAR
> is 360° × 96°, so vertical geometry is partially observable, but that is not an
> absolute floor reference. **Multi-floor operation therefore requires an external
> floor-identity source.**

### The five distinct things that are routinely confused

| Concept | What it is | What it does **not** give |
|---|---|---|
| **1. Named destination** | A database record: `C3_AI_LAB`, display name, block, floor | Does not imply the location exists or is reachable |
| **2. Route-graph node** | A verified graph vertex in the approved corridor network | Does not imply the robot can localise there |
| **3. Localisation landmark** | A physical or visual feature used for pose correction | Does not imply a safe approach pose exists |
| **4. ArUco marker ID** | An integer read by the camera | Does not imply correct identification without validation |
| **5. Verified navigation goal** | A pose on a validated map, physically reached and confirmed by a human | — only this is a safe goal |

> **Do not assume that placing ArUco markers automatically provides safe navigation.**
> A marker can identify a floor; it cannot make a corridor traversable.

**Provisional destination names** (names only — **not** verified navigation
coordinates): `C3_AI_LAB`, `A1_MAKERSPACE`, `C0_GRADE12`, `B1_GRADE7`, `B2_GRADE8`,
`CP-C3`, `CP-A1`, `CP-B0`, `CP-C0`, `CP-CB`.

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-MAP-001 | Workflow | P1 | Defines the mapping method | What official mapping workflow will be used to build the school map? | Vendor SLAM and self-built SLAM differ in outputs and dependencies. | INT | Written method | SH-GO2-013 | YES | | | UNANSWERED | | |
| SH-MAP-002 | Workflow | P1 | Determinism | Will the map be built once and reused, or rebuilt per session? | Persistence is essential for a named-destination system. | INT | Design decision | SH-MAP-001 | YES | | | UNANSWERED | | |
| SH-MAP-003 | Persistence | P1 | Core requirement | Where are maps stored, and what version control applies? | A changed map must not silently replace a verified one. | INT | Storage plan | SH-MAP-002 | YES | | | UNANSWERED | | |
| SH-MAP-004 | Persistence | P1 | Safety requirement | Must a map change invalidate all previously verified destinations? | Otherwise a stale goal could be reused. | INT | Design | SH-MAP-003 | YES | | | UNANSWERED | | |
| SH-MAP-005 | Localisation | P1 | Core requirement | What localisation accuracy is achievable, and how will it be measured? | Determines whether a destination can be confirmed. | INT | Test plan | SH-MAP-001 | YES | | | UNANSWERED | | |
| SH-MAP-006 | Localisation | P0 | Safety requirement | What is the defined behaviour when localisation is lost mid-mission? | Must stop and request assistance, not improvise. | INT | Design | SH-MAP-005 | YES | | | UNANSWERED | | |
| SH-MAP-007 | Localisation | P1 | Common confusion | How will the system distinguish *relocalisation within a floor* from *determining which floor it is*? | They are different problems; only the second is unsolved. | INT | Design | SH-MAP-005 | YES | | | UNANSWERED | | |
| SH-MAP-008 | Coordinate frame | P2 | Implementation requirement | What coordinate frame convention will be used, and how is it documented? | Prevents silent frame errors. | INT | Convention doc | SH-MAP-003 | NO | | | UNANSWERED | | |
| SH-MAP-009 | Floors | P0 | Multi-floor blocker | How will separate floors be represented — separate maps with switching, or one 3D map with slices? | Both are workable; floor identity is the harder problem. | INT | Design decision | SH-MAP-005 | YES | | | UNANSWERED | | |
| SH-MAP-010 | Floors | P0 | Safety requirement | What external signal identifies the current floor, given the robot has no altitude sensor? | ArUco per floor is the recommended mechanism. | INT | Design | SH-MAP-009 | YES | | | UNANSWERED | | |
| SH-MAP-011 | Floors | P0 | Safety requirement | Must a human confirm every floor transition? | Recommended regardless of the sensor used. | ADM | Process decision | SH-MAP-010 | YES | | | UNANSWERED | | |
| SH-MAP-012 | Validation | P0 | Authorisation | Who is authorised to approve a map or destination as verified? | No unverified goal may become operational. | ADM / FAC | Named approver, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-MAP-013 | Validation | P0 | Authorisation | What physical verification must a destination complete before use? | Teleoperation by a person, reaching the pose, confirming safety. | FAC / INT | Verification protocol | SH-MAP-012 | YES | | | UNANSWERED | | |
| SH-MAP-014 | Validation | P1 | Prevent stale data | Does every verified destination carry a `verified_by` and `verified_date`? | A six-month-old verified pose is not trustworthy. | INT | Schema requirement | SH-MAP-013 | YES | | | UNANSWERED | | |
| SH-MAP-015 | Validation | P2 | Expiry policy | After what period does a destination verification expire? | Building changes over time. | INT / ADM | Policy | SH-MAP-014 | NO | | | UNANSWERED | | |
| SH-MAP-016 | ArUco | P1 | Purpose must be explicit | Will ArUco markers be used for **floor identity, pose correction, checkpoint confirmation, docking, or a combination**? | Each purpose has different placement and accuracy requirements. | INT | Design decision | SH-MAP-010 | YES | | | UNANSWERED | | |
| SH-MAP-017 | ArUco | P1 | Detection reliability | What marker size, mounting height and standoff distance will be used? | Detection rate depends strongly on these. | INT | Specification | SH-MAP-016 | YES | | | UNANSWERED | | |
| SH-MAP-018 | ArUco | P1 | Environmental limit | What are the lighting conditions at each marker location? | School corridors vary widely. | FAC | Survey | SH-MAP-017 | YES | | | UNANSWERED | | |
| SH-MAP-019 | ArUco | P2 | Detection reliability | What viewing-angle tolerance does the robot's camera allow? | Off-axis detection degrades sharply. | INT | Test plan | SH-MAP-017 | NO | | | UNANSWERED | | |
| SH-MAP-020 | ArUco | P1 | Failure handling | What happens when a marker is missing, damaged, or misidentified? | Must degrade to `UNKNOWN`, never to a wrong floor. | INT | Design | SH-MAP-016 | YES | | | UNANSWERED | | |
| SH-MAP-021 | ArUco | P0 | Permission required | Does the school permit markers to be fixed to walls, doors, or lift lobbies? | Physical installation, not just software. | ADM / FAC | Written approval, `NEEDS_SCHOOL_PERM` | SH-MAP-016 | YES | | | UNANSWERED | | |
| SH-MAP-022 | ArUco | P1 | Supply and identity | How will marker identity and location be recorded and validated? | A mis-numbered marker is a wrong-floor bug. | INT | Inventory process | SH-MAP-021 | YES | | | UNANSWERED | | |
| SH-MAP-023 | Checkpoints | P1 | Recovery design | Which locations are approved checkpoints, and how many exist per floor? | Determines battery recovery options. | ADM / FAC | Checkpoint list | SH-CONG-007 | YES | | | UNANSWERED | | |
| SH-MAP-024 | Checkpoints | P1 | Safety requirement | Is each checkpoint physically secure against student interference? | The brief explicitly warns that a robot at a checkpoint is **not automatically secure**. | ADM / FAC | Confirmation | SH-MAP-023 | YES | | | UNANSWERED | | |

---

# L. Battery monitoring, charging, recovery and emergency return — `SH-BAT`

> **Research findings that must be carried into this section.** ✅ Battery
> telemetry is on **`rt/lowstate` → `LowState.bms_state`**, providing `soc` (uint8
> percent), `current` (int32 signed mA), `cell_vol[15]`, `bq_ntc`, `status`, `cycle`.
> `SportModeState` contains **no battery fields at all**. ✅ At low battery the
> robot **crouches within 10 minutes** — *a crouch on a staircase is a fall.*
> ✅ There is **no autonomous return-to-dock** in the documented service list; the
> charging pile is ArUco-marker-based and requires a command from outside.
> ✅ Manufacturer runtime figures span **1–5 hours** across four official documents.
> ⚠️ **Thresholds must be recorded as TBD** until manufacturer confirmation and
> authorised testing establish them. **Do not invent numeric thresholds.**

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-BAT-001 | Telemetry | P1 | Core requirement | Through which official interface is battery state of charge read? | Must be `LowState.bms_state`, not the sport topic. | INT | Source citation | SH-GO2-019 | YES | | | UNANSWERED | | |
| SH-BAT-002 | Telemetry | P2 | Improves planning | Are pack voltage, current and per-cell voltage available? | Enables pack health and imbalance detection. | INT | Source citation | SH-BAT-001 | NO | | | UNANSWERED | | |
| SH-BAT-003 | Telemetry | P1 | Calibration | How accurate is state of charge, and how will it be calibrated? | Percentage alone may be misleading. | INT | Calibration plan | SH-BAT-001 | YES | | | UNANSWERED | | |
| SH-BAT-004 | Telemetry | P1 | Failure handling | What happens if battery telemetry becomes unavailable mid-mission? | Must force a safe response, not continue blind. | INT | Design | SH-BAT-001 | YES | | | UNANSWERED | | |
| SH-BAT-005 | Endurance | P1 | Planning requirement | How does battery consumption change with payload, speed, terrain and gait? | Manual lists payload and low body height as reducers. | INT | Test plan | SH-BAT-001 | YES | | | UNANSWERED | | |
| SH-BAT-006 | Endurance | P0 | Safety threshold | What is the experimentally validated **reserve** the robot must maintain? | **TBD until tested.** Must cover the return route plus margin. | INT / LAB | Test results | SH-BAT-005 | YES | | | UNANSWERED | | |
| SH-BAT-007 | Endurance | P0 | Planning rule | Confirm the rule: before starting, check that the task **and** the return are feasible on the remaining charge. | Core safety invariant. | INT | Design confirmation | SH-BAT-006 | YES | | | UNANSWERED | | |
| SH-BAT-008 | Endurance | P1 | Operational rule | At what state of charge does the robot stop accepting new tasks? | **TBD until tested.** | INT / ADM | Policy | SH-BAT-006 | YES | | | UNANSWERED | | |
| SH-BAT-009 | Recovery | P1 | Recovery design | How is the return feasibility calculation performed? | Must be conservative and route-verified. | INT | Design | SH-BAT-006 | YES | | | UNANSWERED | | |
| SH-BAT-010 | Recovery | P1 | Checkpoint selection | How is the nearest **reachable and safe** checkpoint selected? | Not simply the nearest by distance. | INT | Design | SH-MAP-023 | YES | | | UNANSWERED | | |
| SH-BAT-011 | Recovery | P0 | Safety requirement | If return is not safely feasible, does the robot enter the documented safe-stop and human-assistance workflow? | Must, and must not improvise a route. | INT / ADM | Design | SH-BAT-009 | YES | | | UNANSWERED | | |
| SH-BAT-012 | Recovery | P0 | Explicit prohibition | Confirm a low battery may **not** cause the robot to take an unverified stair route, enter a lift without approval, or cross a closed or hazardous corridor. | Hard design constraint. | INT / ADM | Design confirmation | SH-BAT-011 | YES | | | UNANSWERED | | |
| SH-BAT-013 | Recovery | P1 | Edge case | What happens if the nearest checkpoint is inaccessible or occupied? | Requires an ordered fallback. | INT | Design | SH-BAT-010 | YES | | | UNANSWERED | | |
| SH-BAT-014 | Recovery | P1 | Timing | How long may a delivery continue once the abort threshold is reached mid-task? | Depends on whether completion or recovery takes priority. | ADM / INT | Policy | SH-BAT-008 | YES | | | UNANSWERED | | |
| SH-BAT-015 | Charging | P0 | Permission and safety | Is charging attended, unattended, or overnight — and what does the school permit? | Manufacturer temperature limits are strict. | ADM / FAC | Written policy, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-BAT-016 | Charging | P0 | Safety requirement | What is the official charging procedure and thermal limit? | Discharge −20 to 60 °C; charge 0 to 55 °C. Exceeding may ignite or explode. | ROB / VENDOR | Manufacturer documentation | SH-BAT-015 | YES | | | UNANSWERED | | |
| SH-BAT-017 | Charging | P1 | Operational requirement | Where will charging occur, on what surface, and with what supervision? | Non-flammable surface, supervised in an occupied building. | FAC | Location plan | SH-BAT-015 | YES | | | UNANSWERED | | |
| SH-BAT-018 | Charging | P1 | Swap procedure | Is hot-swapping supported or prohibited, and what is the documented swap procedure? | The pack latches from the side; the robot must lie on a flat surface. | ROB / VENDOR | Manufacturer documentation | SH-BAT-015 | YES | | | UNANSWERED | | |
| SH-BAT-019 | Charging | P1 | Spare capacity | How many spare batteries exist, and who owns charging responsibility? | Determines turnaround between missions. | ADM / LAB | Logged allocation | SH-BAT-018 | NO | | | UNANSWERED | | |
| SH-BAT-020 | Health | P2 | Lifecycle | How are battery health, cycle count and cell imbalance logged? | Needed for long-term ownership. | INT / LAB | Logging design | SH-BAT-002 | NO | | | UNANSWERED | | |
| SH-BAT-021 | Health | P1 | Quarantine | What is the procedure for a damaged, swollen or dropped pack? | Must exist before the first pack is charged. | FAC | Procedure, `NEEDS_SCHOOL_PERM` | SH-BAT-015 | YES | | | UNANSWERED | | |
| SH-BAT-022 | Docking | P2 | Optional capability | Is the ArUco-based charging pile present, and is there a documented recharge interface? | The service page exists in documentation but its body has not been read. | ROB / INT | Documentation | SH-GO2-007 | NO | | | UNANSWERED | | |

---

# M. School schedules, lunch breaks and operating windows — `SH-TIME`

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-TIME-001 | Timetable | P1 | Determines operating windows | What is the official school timetable, and who maintains it? | Corridors peak at class change; timing depends on this. | ADM | Timetable document | — | YES | | | UNANSWERED | | |
| SH-TIME-002 | Timetable | P2 | Accuracy requirement | Does the timetable change on examination days or special events? | A stale schedule mispredicts congestion. | ADM | Change process | SH-TIME-001 | NO | | | UNANSWERED | | |
| SH-TIME-003 | Timetable | P1 | Data source | Can the timetable be obtained in a machine-readable format? | Manual re-entry is a reliability risk. | ADM | File format | SH-TIME-001 | YES | | | UNANSWERED | | |
| SH-TIME-004 | Timetable | P2 | Fallback | If no machine-readable source exists, who updates SHANA manually? | Human error becomes a single point of failure. | ADM | Named owner | SH-TIME-003 | NO | | | UNANSWERED | | |
| SH-TIME-005 | Windows | P1 | Defines availability | What are the exact lunch start and end times, including any staggering by grade? | Determines the shutdown cutoff. | ADM | Schedule | SH-TIME-001 | YES | | | UNANSWERED | | |
| SH-TIME-006 | Windows | P0 | Safety requirement | What lead time is required for the robot to reach an approved checkpoint before lunch? | Must be measured, not estimated. | ADM / INT | Requirement | SH-BAT-006 | YES | | | UNANSWERED | | |
| SH-TIME-007 | Windows | P1 | Workflow rule | Should a delivery be rejected when it cannot finish before the cutoff? | Recommended behaviour. | ADM | Policy decision | SH-TIME-006 | YES | | | UNANSWERED | | |
| SH-TIME-008 | Windows | P1 | Edge case | What happens to a delivery already in progress at the cutoff? | Must complete safely or abort, per policy. | ADM | Policy decision | SH-TIME-007 | YES | | | UNANSWERED | | |
| SH-TIME-009 | Windows | P1 | Failure case | What happens if the robot cannot return before the cutoff? | Requires an escalation path, not a silent overrun. | ADM / INT | Procedure | SH-TIME-008 | YES | | | UNANSWERED | | |
| SH-TIME-010 | Security | P0 | Safety requirement | Is a checkpoint secure, authorised, and protected from student access? | A robot at a checkpoint is not automatically secure. | ADM / FAC | Confirmation | SH-MAP-024 | YES | | | UNANSWERED | | |
| SH-TIME-011 | Security | P1 | Practical control | Must the robot be powered down, attended, or physically secured during lunch? | Determines the shutdown workflow. | ADM | Policy decision | SH-TIME-010 | YES | | | UNANSWERED | | |
| SH-TIME-012 | Security | P1 | Access control | How is student access to the checkpoint area prevented? | Physical control, not just signage. | FAC | Procedure | SH-TIME-010 | YES | | | UNANSWERED | | |
| SH-TIME-013 | Operation | P1 | Manual control | Is there a manual disable control, and who may operate it? | Must be reachable without the dashboard. | INT / ADM | Design | SH-TIME-011 | YES | | | UNANSWERED | | |
| SH-TIME-014 | Operation | P1 | Resume authority | Who is authorised to resume deliveries after the shutdown? | Must not be automatic. | ADM | Named authority | SH-TIME-013 | YES | | | UNANSWERED | | |
| SH-TIME-015 | Operation | P2 | Edge cases | How are holidays, assemblies, exams and emergency closures handled? | Each is a distinct operating mode. | ADM | Calendar policy | SH-TIME-014 | NO | | | UNANSWERED | | |
| SH-TIME-016 | Operation | P1 | Data failure | What happens if timetable data becomes unavailable or incorrect? | Must fail safe, not silently continue. | INT | Design | SH-TIME-003 | YES | | | UNANSWERED | | |
| SH-TIME-017 | Operation | P2 | Congestion prediction | Does the timetable feed the congestion predictor? | Optional enhancement. | INT | Design | SH-CONG-015, SH-TIME-003 | NO | | | UNANSWERED | | |
| SH-TIME-018 | Operation | P1 | Shift pattern | Are there periods when no trained supervisor is available? | Missions must not run unsupervised. | ADM | Staffing roster | SH-TIME-013 | YES | | | UNANSWERED | | |
| SH-TIME-019 | Operation | P2 | Testing windows | During which periods may controlled testing take place? | Outside student hours where possible. | ADM | Window agreement | SH-TIME-018 | NO | | | UNANSWERED | | |
| SH-TIME-020 | Operation | P1 | Operating windows | Which operating windows does the school authorise for live deliveries? | Distinct from testing windows. | ADM | Written approval, `NEEDS_SCHOOL_PERM` | SH-TIME-019 | YES | | | UNANSWERED | | |

---

# N. Delivery workflow, teachers, recipients and item restrictions — `SH-DEL`

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-DEL-001 | Need | P0 | Determines whether the project solves a real problem | What is the single most valuable task teachers would delegate? | Building the wrong product is a common failure. | TCH | Ranked list | — | YES | | | UNANSWERED | | |
| SH-DEL-002 | Items | P0 | Safety-critical | What items may SHANA carry, and what items are **absolutely prohibited**? | Glass, liquids, chemicals, sharps and heated apparatus are excluded. | ADM / LAB | Written item policy, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-DEL-003 | Items | P0 | Payload limit | What is the maximum permitted payload weight and container size? | Rated ≈8 kg, but real-world usable is substantially lower. | ADM / INT | Specification | SH-DEL-002 | YES | | | UNANSWERED | | |
| SH-DEL-004 | Items | P1 | Dependability | What container types are approved, and must items be sealed or lidded? | Retention design depends on this. | ADM / LAB | Container list | SH-DEL-003 | YES | | | UNANSWERED | | |
| SH-DEL-005 | Items | P1 | Tamper evidence | Must the container be tamper-evident or chained to custody? | Relevant to exam papers or valuables. | ADM | Policy | SH-DEL-004 | NO | | | UNANSWERED | | |
| SH-DEL-006 | Workflow | P1 | Core interaction | How will teachers submit a delivery request? | Determines the interface and training burden. | TCH | Process design | SH-DEL-001 | YES | | | UNANSWERED | | |
| SH-DEL-007 | Workflow | P1 | Safety rule | Confirm the destination must be selected from an approved allow-list, never typed freely. | Prevents navigation to unintended places. | INT | Design confirmation | SH-DEL-006 | YES | | | UNANSWERED | | |
| SH-DEL-008 | Workflow | P2 | Usability | How will teachers be told a delivery is ready, and what notification channels are acceptable? | Dashboard, screen, email, or spoken announcement. | TCH | Preference ranking | SH-DEL-006 | NO | | | UNANSWERED | | |
| SH-DEL-009 | Workflow | P0 | Completion integrity | Confirm a delivery is **never** marked complete solely because the robot arrived. | Explicit recipient confirmation is required. | ADM / INT | Design confirmation | SH-DEL-006 | YES | | | UNANSWERED | | |
| SH-DEL-010 | Workflow | P1 | Recipients | Who is authorised to confirm receipt, and how is their identity established? | Determines whether confirmation is trustworthy. | ADM | Policy | SH-DEL-009 | YES | | | UNANSWERED | | |
| SH-DEL-011 | Workflow | P1 | Edge case | What happens if no recipient is present at the destination? | Must not be marked complete. | INT / ADM | Policy decision | SH-DEL-010 | YES | | | UNANSWERED | | |
| SH-DEL-012 | Workflow | P1 | Exception | What happens if the recipient refuses the delivery? | Must be a recorded outcome, not a failure. | ADM | Policy | SH-DEL-010 | YES | | | UNANSWERED | | |
| SH-DEL-013 | Controls | P1 | Duplicate prevention | How are duplicate or repeated requests detected and rejected? | Prevents accidental double submission. | INT | Design | SH-DEL-006 | YES | | | UNANSWERED | | |
| SH-DEL-014 | Controls | P2 | Abuse prevention | Are there rate limits on request submission? | Protects the queue and the robot. | INT | Design parameter | SH-DEL-013 | NO | | | UNANSWERED | | |
| SH-DEL-015 | Controls | P1 | Cancellation | Who may cancel a delivery, and at what stages? | Cancellation must be safe at any point. | ADM | Policy | SH-DEL-006 | YES | | | UNANSWERED | | |
| SH-DEL-016 | Controls | P2 | Auditability | What audit trail is required for every delivery lifecycle event? | Required for incident review. | ADM | Requirement | SH-SRV-016 | NO | | | UNANSWERED | | |
| SH-DEL-017 | Adoption | P1 | Determines real value | How many teachers are expected to use SHANA, and how often? | Determines whether daily reliability matters. | ADM / TCH | Survey | SH-DEL-001 | YES | | | UNANSWERED | | |
| SH-DEL-018 | Adoption | P2 | Feedback | How will teacher feedback be collected and acted upon? | Closes the loop without surveying students. | TCH | Process design | SH-DEL-017 | NO | | | UNANSWERED | | |
| SH-DEL-019 | Adoption | P1 | Acceptance threshold | What would make teachers stop using the system? | Defines the reliability bar. | TCH | Survey | SH-DEL-017 | YES | | | UNANSWERED | | |
| SH-DEL-020 | Adoption | P1 | Safety perception | Would teachers accept a 15 kg robot in their corridor during lessons? | Their threshold may be stricter than the technical one. | TCH | Survey | SH-DEL-017 | YES | | | UNANSWERED | | |
| SH-DEL-021 | Scope | P2 | Initial limits | Which rooms are candidates for the initial pilot, and which are excluded? | Start small. | ADM / TCH | Room list | SH-DEL-002 | NO | | | UNANSWERED | | |
| SH-DEL-022 | Scope | P0 | Design principle | Confirm teachers must never receive direct motor-control functionality. | Hard constraint. | ADM / INT | Design confirmation | SH-SRV-011 | YES | | | UNANSWERED | | |

---

# O. Safety, emergency stops, failure handling and human intervention — `SH-SAFE`

> **Research findings that must be carried into this section.**
> ✅ Go2 manual: *"Hand may be pinched at the motion joints, such as at the knee
> joint."*
> ✅ *"Keep it under control in the users' view… keep the robot maintain a safe
> distance of at least 2 meters from obstacles, complex ground, crowds, water."*
> ✅ *"This product is not a toy and is not intended for use by persons under the
> age of 14."*
> ✅ `Damp()` is the documented emergency stop: joints go limp and the robot
> collapses under gravity.
> ✅ `RecoveryStand()` recovers *"regardless of whether it is fallen or not"* —
> **unsafe with payload.**
> ✅ Field reports include a Go2 *"powered off completely in the middle of its
> movement"* and one that *"run into a door … while it was supposed to be in
> autonomous navigation and obstacle avoidance mode."*
> ✅ Built-in obstacle avoidance is **on by default**. ✅ `FreeAvoid` may produce an
> **uncommanded lateral dodge when stationary**.

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-SAFE-001 | Emergency stop | P0 | Mandatory before any powered run | What is the approved emergency-stop arrangement, and who holds it? | A named person with a working method, tested before every run. | ADM / LAB | Written procedure, `NEEDS_SCHOOL_PERM` | — | YES | | | UNANSWERED | | |
| SH-SAFE-002 | Emergency stop | P0 | Must be validated | Has the emergency stop been physically tested on the robot? | A method that has not been tested is not a control. | LAB | Test record | SH-SAFE-001 | YES | | | UNANSWERED | | |
| SH-SAFE-003 | Emergency stop | P1 | Immediate safety | Where is the nearest hard power switch, and can it be reached within seconds? | Backup for software failure. | FAC | Site plan | SH-SAFE-001 | YES | | | UNANSWERED | | |
| SH-SAFE-004 | Emergency stop | P1 | Method ambiguity | Which documented emergency method applies — `Damp()`, the physical controller, or hard power? | `Damp()` causes a collapse; hard power causes a fall. | INT / ROB | Method definition | SH-SAFE-001 | YES | | | UNANSWERED | | |
| SH-SAFE-005 | Spotter | P0 | Mandatory at every stage | Who is the named spotter, and what are their exact duties? | The spotter is a load-bearing control, not an observer. | ADM / LAB | Named person + duties | SH-SAFE-001 | YES | | | UNANSWERED | | |
| SH-SAFE-006 | Spotter | P0 | Authority | Confirm the spotter has **unilateral stop authority** with no justification required. | Removes hesitation under social pressure. | ADM | Written confirmation | SH-SAFE-005 | YES | | | UNANSWERED | | |
| SH-SAFE-007 | Spotter | P1 | Continuity | Are there at least two trained spotters, and a backup? | Absence of a spotter must stop operations. | ADM | Staffing record | SH-SAFE-005 | YES | | | UNANSWERED | | |
| SH-SAFE-008 | Exclusion zone | P0 | Primary physical control | How is the exclusion zone defined, marked, and enforced? | Distance alone is insufficient. | ADM / FAC | Procedure | SH-SAFE-005 | YES | | | UNANSWERED | | |
| SH-SAFE-009 | Exclusion zone | P2 | Speed control | What maximum operating speed is approved in each context? | Slower than a walking child. | ADM | Speed policy | SH-SAFE-008 | NO | | | UNANSWERED | | |
| SH-SAFE-010 | Speed | P1 | Design constraint | Confirm the software must clamp all velocity requests to the approved maximum. | Software control, not operator discipline alone. | INT | Design confirmation | SH-SAFE-009 | YES | | | UNANSWERED | | |
| SH-SAFE-011 | Interference | P1 | Realistic school scenario | What happens if students approach, touch, or ride the robot? | Interference must be detected or excluded by the zone. | ADM / TCH | Procedure | SH-SAFE-008 | YES | | | UNANSWERED | | |
| SH-SAFE-012 | Interference | P0 | Prohibited modification | What is the response if someone attempts to modify, mount on, or ride the robot? | Tampering must stop operations immediately. | ADM | Procedure | SH-SAFE-011 | YES | | | UNANSWERED | | |
| SH-SAFE-013 | Payload | P0 | Falling-object hazard | How is the payload secured, and against what deceleration? | Two independent restraints required. | INT | Design | SH-DEL-003 | YES | | | UNANSWERED | | |
| SH-SAFE-014 | Payload | P1 | Tilt test | Must the carrier be tilt-tested before use? | Cheap, fast, and catches most retention failures. | INT | Test record | SH-SAFE-013 | YES | | | UNANSWERED | | |
| SH-SAFE-015 | Comms loss | P0 | Safety requirement | What is the defined behaviour when the robot loses communication with the server? | Must be a safe hold, never autonomous improvisation. | INT | Design | — | YES | | | UNANSWERED | | |
| SH-SAFE-016 | Server crash | P0 | Safety requirement | What is the defined behaviour when the server crashes mid-mission? | There is no software watchdog; physical backstops apply. | INT | Design | SH-SAFE-015 | YES | | | UNANSWERED | | |
| SH-SAFE-017 | Localisation loss | P0 | Safety requirement | What is the defined behaviour when localisation is lost or the map does not match? | Must stop, not re-localise indefinitely. | INT | Design | SH-MAP-006 | YES | | | UNANSWERED | | |
| SH-SAFE-018 | Sensor failure | P0 | Safety requirement | What is the defined behaviour when a camera or depth sensor fails? | Must degrade conservatively, never assume clear. | INT | Design | SH-CCTV-021 | YES | | | UNANSWERED | | |
| SH-SAFE-019 | Sensor failure | P0 | Independence requirement | Confirm no single sensor, including CCTV, may be the sole obstacle-avoidance system. | Independence requirement. | INT / ADM | Design confirmation | SH-CCTV-021 | YES | | | UNANSWERED | | |
| SH-SAFE-020 | Obstacles | P1 | Known failure mode | How is the built-in avoidance behaviour verified and limited? | A field report describes a collision while avoidance was active. | INT / LAB | Test record | SH-SAFE-018 | YES | | | UNANSWERED | | |
| SH-SAFE-021 | Obstacles | P0 | Documented hazard | Confirm the robot must not rely on the default `FreeAvoid` behaviour while stationary near people. | Uncommanded lateral dodge risk. | INT | Design confirmation | SH-SAFE-008 | YES | | | UNANSWERED | | |
| SH-SAFE-022 | Recovery | P0 | Unsafe with payload | Confirm `RecoveryStand()` and auto-recovery are **disabled** during payload missions. | Documented as rising regardless of state. | INT | Design confirmation | SH-BAT-006 | YES | | | UNANSWERED | | |
| SH-SAFE-023 | Recovery | P1 | Manual procedure | What is the manual recovery procedure if the robot immobilises itself? | Must be taught to spotters. | LAB | Written procedure | SH-SAFE-005 | YES | | | UNANSWERED | | |
| SH-SAFE-024 | Evacuation | P0 | Life safety | What happens on a fire alarm or evacuation? | The robot must not obstruct an evacuation route. | ADM / FAC | Procedure | SH-BLOCK-016 | YES | | | UNANSWERED | | |
| SH-SAFE-025 | Reporting | P1 | Accountability | How are incidents, faults and near-misses reported and reviewed? | Near-misses are the most valuable data collected. | ADM | Procedure | SH-SAFE-001 | YES | | | UNANSWERED | | |
| SH-SAFE-026 | Assurance | P0 | Final safety gate | What evidence is required before any student-facing operation is authorised? | A defined gate, not a judgement call. | ADM | Gate criteria | SH-SAFE-001 | YES | | | UNANSWERED | | |---

# P. Permissions, school policies, student privacy and data retention — `SH-PRIV`

> **Legal position established by research.** ✅ Neither the **DPDP Act 2023** nor
> the **DPDP Rules 2025** mentions CCTV, surveillance, biometrics or facial
> recognition in their text. ✅ The operative sections, including §9 on children,
> **do not come into force until approximately May 2027** per G.S.R. 843(E)(c).
> ✅ DPDP Rules 2025 Rule 12 and Fourth Schedule Part A grant educational
> institutions an exemption — **but conditioned on processing for educational
> activities or "in the interests of safety of children", and corridor routing is
> not obviously either.** ✅ IT Act 2000 §66E is narrow: it criminalises images of
> *"genitals, public area, buttocks or female breast"*, not corridor CCTV.
> ✅ **CBSE clause 4.7.10 (21 July 2025) already mandates audio-visual CCTV in
> corridors with 15-day retention** — so in a CBSE school the footage already
> exists and the school is already the Data Fiduciary. **The stream contains
> audio, which the SHANA pipeline must explicitly not decode.**

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-PRIV-001 | Governance | P0 | No authority, no operation | Who is the named Data Fiduciary for student data at this school? | Assigns accountability and approval authority. | ADM | Named role | — | YES | | | UNANSWERED | | |
| SH-PRIV-002 | Governance | P0 | Permission prerequisite | Has the Data Fiduciary authorised this project in principle? | Required before any data processing. | ADM | Written approval, `NEEDS_SCHOOL_PERM` | SH-PRIV-001 | YES | | | UNANSWERED | | |
| SH-PRIV-003 | Governance | P1 | Policy alignment | What existing school policies govern robot operation, data capture and student privacy? | SHANA must comply with existing rules. | ADM | Policy documents | SH-PRIV-001 | YES | | | UNANSWERED | | |
| SH-PRIV-004 | Governance | P0 | Hard prohibition | Confirm SHANA will perform **no facial recognition, identity recognition or biometric identification**. | Absolute design constraint. | ADM | Written confirmation | SH-PRIV-002 | YES | | | UNANSWERED | | |
| SH-PRIV-005 | Governance | P0 | Hard prohibition | Confirm SHANA will **never** automatically accuse, identify, rank or report individual students as suspicious. | Explicit project constraint. | ADM | Written confirmation | SH-PRIV-002 | YES | | | UNANSWERED | | |
| SH-PRIV-006 | Governance | P0 | Hard prohibition | Confirm SHANA will **not** collect or process actual student footage during research. | Research-phase constraint. | ADM | Written confirmation | SH-PRIV-004 | YES | | | UNANSWERED | | |
| SH-PRIV-007 | CCTV | P0 | Permission prerequisite | Is access to CCTV analytics authorised by the CCTV owner and the Data Fiduciary? | Without this, occupancy-from-CCTV is closed. | CCTV / ADM | Written approval, `NEEDS_SCHOOL_PERM` | SH-CCTV-016 | YES | | | UNANSWERED | | |
| SH-PRIV-008 | CCTV | P0 | Legal qualification | Does any available exemption cover corridor-routing analytics, given the exemption covers educational activity or child safety? | **This must be resolved by a qualified adviser, not assumed.** | ADM | Legal or policy opinion | SH-PRIV-007 | YES | | | UNANSWERED | | |
| SH-PRIV-009 | CCTV | P1 | Technical safeguard | Confirm the pipeline will **not decode audio** from CCTV streams. | Streams may carry audio; decoding it is unnecessary risk. | INT | Design confirmation | SH-CCTV-019 | YES | | | UNANSWERED | | |
| SH-PRIV-010 | CCTV | P0 | Design safeguard | Confirm frames are processed in-stream and discarded, with no raw video written to disk by default. | Core privacy guarantee. | INT / ADM | Design confirmation | SH-PRIV-007 | YES | | | UNANSWERED | | |
| SH-PRIV-011 | CCTV | P1 | If retention is required | If any debug capture is unavoidable, who authorises it, for how long, and how is it deleted? | Must be an explicit, logged decision. | ADM | Procedure | SH-PRIV-010 | YES | | | UNANSWERED | | |
| SH-PRIV-012 | Occupancy data | P0 | Derived data may still be personal data | How long is derived occupancy data retained, and is it accessible to non-technical staff? | Count data can become identifying in combination. | ADM | Retention policy | SH-PRIV-007 | YES | | | UNANSWERED | | |
| SH-PRIV-013 | Occupancy data | P1 | Minimisation | Will occupancy data be stored at all, or only acted upon and discarded? | Discard-only is the strongest posture. | INT / ADM | Design decision | SH-PRIV-012 | YES | | | UNANSWERED | | |
| SH-PRIV-014 | Occupancy data | P0 | Prohibited use | Confirm occupancy data will **never** be used for individual assessment, attendance or behaviour monitoring. | Extends the misuse prohibition to derived data. | ADM | Written confirmation | SH-PRIV-012 | YES | | | UNANSWERED | | |
| SH-PRIV-015 | Incidents | P0 | Accountability requirement | Confirm any incident report requires **human review and school approval** before any action. | No automated enforcement output. | ADM | Procedure | SH-PRIV-014 | YES | | | UNANSWERED | | |
| SH-PRIV-016 | Cameras | P1 | Role limitation | May students be told that anonymous counting occurs? | Transparency policy question for the school. | ADM | Policy decision | SH-PRIV-007 | NO | | | UNANSWERED | | |
| SH-PRIV-017 | Photos | P1 | Necessary limitation | May photographs of the robot be taken for the exhibition, and may students appear in them? | Consent and image-rights question. | ADM | Policy | — | NO | | | UNANSWERED | | |
| SH-PRIV-018 | Robot camera | P1 | Permitted use | What may the robot's **own** camera capture, and is recording permitted? | A separate question from CCTV. | ADM | Policy | SH-PRIV-003 | YES | | | UNANSWERED | | |
| SH-PRIV-019 | Notifications | P1 | Data minimisation | Do delivery notifications expose any personal information to other teachers? | Role-based visibility must be enforced. | ADM | Policy | SH-SRV-012 | NO | | | UNANSWERED | | |
| SH-PRIV-020 | Credentials | P0 | Hard constraint | Confirm no Wi-Fi keys, CCTV credentials, tokens, passwords or private network details will be committed to Git. | Repository hygiene requirement. | INT | Design confirmation | SH-NET-018 | YES | | | UNANSWERED | | |
| SH-PRIV-021 | Credentials | P1 | Operational | Where will production credentials be stored, and how rotated? | Must be outside the repository. | IT / INT | Policy | SH-PRIV-020 | YES | | | UNANSWERED | | |
| SH-PRIV-022 | Breach | P1 | Governance | What is the school's breach-notification process if SHANA mishandles data? | Required even for a school project. | ADM / IT | Procedure | SH-PRIV-001 | NO | | | UNANSWERED | | |

---

# Q. Exhibition demonstration, evaluation and acceptance testing — `SH-EXPO`

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-EXPO-001 | Scope | P0 | Bounds what may be demonstrated | What exactly will the exhibition demonstrate, and what is explicitly excluded? | A constrained demonstration is safer than an ambitious one. | ADM / EXPO | Written scope | — | YES | | | UNANSWERED | | |
| SH-EXPO-002 | Scope | P0 | Safety gate | Must any student-facing demonstration be **separately authorised** from the pilot? | Yes. Student presence changes the risk profile entirely. | ADM | Written approval, `NEEDS_SCHOOL_PERM` | SH-EXPO-001 | YES | | | UNANSWERED | | |
| SH-EXPO-003 | Environment | P0 | Determines feasibility | Where will the exhibition be held, and is that space verified as safe? | An unverified venue invalidates all prior route work. | EXPO / FAC | Site survey | SH-EXPO-001 | YES | | | UNANSWERED | | |
| SH-EXPO-004 | Environment | P1 | Network at venue | Is network and power available at the venue, or must it be temporary? | Determines whether a temporary isolated network is required. | IT / EXPO | Site services | SH-NET-024 | YES | | | UNANSWERED | | |
| SH-EXPO-005 | Environment | P1 | Crowd control | How will spectators be kept outside the exclusion zone during the demonstration? | Crowds are the primary hazard at an event. | EXPO / ADM | Plan | SH-SAFE-008 | YES | | | UNANSWERED | | |
| SH-EXPO-006 | Environment | P0 | Supervision | Who will supervise the robot at the exhibition, and for the whole duration? | No unsupervised operation, ever. | ADM / EXPO | Named supervisor | SH-SAFE-005 | YES | | | UNANSWERED | | |
| SH-EXPO-007 | Demonstration | P1 | Repeatability | How many consecutive successful demonstrations are required before the event? | A repeatable workflow matters more than a single good run. | EXPO / INT | Acceptance criteria | SH-EXPO-001 | YES | | | UNANSWERED | | |
| SH-EXPO-008 | Demonstration | P2 | Contingency | Is there a fallback demonstration if live navigation fails on the day? | A static or simulated fallback protects the event. | EXPO / INT | Plan | SH-EXPO-007 | NO | | | UNANSWERED | | |
| SH-EXPO-009 | Demonstration | P0 | Abort criteria | What are the abort criteria during the exhibition, and who decides? | Must be agreed before the day, not during it. | ADM / EXPO | Written criteria | SH-SAFE-001 | YES | | | UNANSWERED | | |
| SH-EXPO-010 | Evaluation | P1 | Evidence quality | Which metrics will be published, and how will they be measured? | Unmeasured claims are not evidence. | INT / EXPO | Metric definitions | SH-EXPO-001 | YES | | | UNANSWERED | | |
| SH-EXPO-011 | Evaluation | P1 | Honest reporting | Confirm failures and near-misses will be reported alongside successes. | Reporting only successes undermines the whole exercise. | INT / ADM | Written confirmation | SH-EXPO-010 | YES | | | UNANSWERED | | |
| SH-EXPO-012 | Evaluation | P2 | Baseline | Is there an existing manual process whose time cost can serve as a comparison baseline? | Provides a meaningful efficiency claim. | TCH / ADM | Baseline data | SH-EXPO-010 | NO | | | UNANSWERED | | |
| SH-EXPO-013 | Evaluation | P1 | Acceptance criteria | Who judges acceptance of the exhibition build, and against what criteria? | Prevents last-minute pressure to ship. | ADM / EXPO | Acceptance criteria | SH-EXPO-010 | YES | | | UNANSWERED | | |
| SH-EXPO-014 | Constraints | P0 | Safety during event | Confirm no stairs, lifts, or congested corridors will be attempted during the exhibition. | The exhibition is not the time to test unverified transitions. | ADM / INT | Written confirmation | SH-EXPO-001 | YES | | | UNANSWERED | | |
| SH-EXPO-015 | Constraints | P1 | Battery at event | What is the charging plan before, during and after the event? | Event downtime is the most common failure. | EXPO / LAB | Plan | SH-BAT-015 | YES | | | UNANSWERED | | |
| SH-EXPO-016 | Constraints | P1 | Payload at event | Will live deliveries carry real items, or demonstration payloads? | Live items add risk with no demonstration value. | ADM / EXPO | Decision | SH-DEL-002 | YES | | | UNANSWERED | | |
| SH-EXPO-017 | Constraints | P0 | Incident readiness | What is the incident response plan during the event, including first aid? | An event has a different audience and a different risk profile. | EXPO / ADM | Plan | SH-SAFE-025 | YES | | | UNANSWERED | | |
| SH-EXPO-018 | Communications | P2 | Publicity | May the demonstration be photographed, filmed, or published? | Affects what may be visible on dashboards. | ADM / EXPO | Policy | SH-PRIV-017 | NO | | | UNANSWERED | | |
| SH-EXPO-019 | Logistics | P2 | Transport | How is the robot transported to and from the venue, and who handles it? | Transport damage and handling risk. | EXPO / LAB | Plan | SH-EXPO-003 | NO | | | UNANSWERED | | |
| SH-EXPO-020 | Post-event | P2 | Sustainability | What happens to the robot and data after the exhibition? | Determines whether the project continues or ends. | ADM | Decision record | SH-EXPO-001 | NO | | | UNANSWERED | | |

---

# R. Budget, installation, maintenance and long-term ownership — `SH-BUD`

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-BUD-001 | Budget | P0 | Bounds everything | What budget exists for this project, over what period? | Determines hardware, sensors, and infrastructure options. | ADM | Written budget | — | YES | | | UNANSWERED | | |
| SH-BUD-002 | Budget | P1 | Distinguishes options | Is there separate capital funding for hardware and consumable funding for operations? | Sensors and batteries are consumables. | ADM | Budget breakdown | SH-BUD-001 | YES | | | UNANSWERED | | |
| SH-BUD-003 | Budget | P2 | Operational cost | Who pays for ongoing consumables — batteries, markers, printing? | Sustainability. | ADM | Decision record | SH-BUD-002 | NO | | | UNANSWERED | | |
| SH-BUD-004 | Hardware | P1 | Purchase decision | May new hardware be purchased, or must everything be sourced from existing stock? | Determines whether Route A or Route B navigation is used. | ADM | Decision record | SH-BUD-001 | YES | | | UNANSWERED | | |
| SH-BUD-005 | Hardware | P2 | Optional path | If the expansion compute dock could be purchased, would it be approved? | Vendor SLAM requires it. | ADM / ROB | Decision record | SH-BUD-004 | NO | | | UNANSWERED | | |
| SH-BUD-006 | Hardware | P2 | Optional path | Could a MID-360 or XT16 LiDAR be purchased if vendor navigation were chosen? | Required by `slam_operate`. | ADM | Decision record | SH-BUD-004 | NO | | | UNANSWERED | | |
| SH-BUD-007 | Infrastructure | P1 | Physical dependency | Is cabling permitted and budgeted for the robot network segment? | Preferred over Wi-Fi for the robot link. | ADM / IT | Cost and approval | SH-NET-003 | YES | | | UNANSWERED | | |
| SH-BUD-008 | Infrastructure | P1 | Physical dependency | Is cabling and mounting for access points permitted and budgeted? | Determines whether per-floor coverage is achievable. | ADM / IT | Cost and approval | SH-NET-012 | YES | | | UNANSWERED | | |
| SH-BUD-009 | Infrastructure | P2 | Physical dependency | Is budget available for ArUco marker production, printing and mounting? | Low cost, high value. | ADM | Cost | SH-MAP-021 | NO | | | UNANSWERED | | |
| SH-BUD-010 | Infrastructure | P2 | Optional | Is budget available for a spare battery and a charging station? | Directly affects mission availability. | ADM | Cost | SH-BAT-019 | NO | | | UNANSWERED | | |
| SH-BUD-011 | Installation | P0 | Permission prerequisite | Which installations require school approval, and who approves each? | Markers, cabling, APs, signage all differ. | ADM | Approval matrix | — | YES | | | UNANSWERED | | |
| SH-BUD-012 | Installation | P1 | Physical constraint | Who installs markers and cabling, and under whose supervision? | School trades may be required. | ADM | Named installer | SH-BUD-011 | YES | | | UNANSWERED | | |
| SH-BUD-013 | Installation | P1 | Safety constraint | Are corridors modified during installation, and does that disrupt the school? | Timing matters. | ADM | Plan | SH-BUD-011 | NO | | | UNANSWERED | | |
| SH-BUD-014 | Maintenance | P1 | Sustainability | Who maintains the robot hardware, and what training do they have? | Distinguishes school capability from ours. | ADM | Named maintainer | — | YES | | | UNANSWERED | | |
| SH-BUD-015 | Maintenance | P1 | Sustainability | Who maintains the software, maps and server? | Long-term continuity. | ADM | Named owner | SH-SRV-018 | YES | | | UNANSWERED | | |
| SH-BUD-016 | Maintenance | P2 | Risk management | What is the contingency if the robot is damaged beyond repair? | A single point of failure for the project. | ADM | Plan | SH-GO2-024 | NO | | | UNANSWERED | | |
| SH-BUD-017 | Ownership | P0 | Legal requirement | Who is the legal owner of the robot, the server and all data? | Determines liability and access rights. | ADM | Ownership record | — | YES | | | UNANSWERED | | |
| SH-BUD-018 | Ownership | P1 | Continuity | Does the project continue after the exhibition, and who funds it? | Determines whether durability matters now. | ADM | Decision record | SH-EXPO-020 | YES | | | UNANSWERED | | |
| SH-BUD-019 | Procurement | P2 | Compliance | Are there school procurement rules for hardware purchases? | Affects lead times. | ADM | Policy | SH-BUD-001 | NO | | | UNANSWERED | | |
| SH-BUD-020 | Risk | P1 | Financial exposure | Does the school's insurance cover the robot in operation and in transit? | *Otis v. Rashmi Handa* apportioned liability to the occupier. | ADM | Written confirmation | SH-BUD-017 | YES | | | UNANSWERED | | |

---

# S. Git repository, documentation, configuration and deployment — `SH-GIT`

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-GIT-001 | Ownership | P0 | Cannot commit without authority | Who owns the repository, and who may approve commits? | Required before any version control is used. | ADM / INT | Named owner | — | YES | | | UNANSWERED | | |
| SH-GIT-002 | Ownership | P0 | Do not push without authorisation | Is the repository private, and may it be hosted on a public service at all? | School policy may prohibit external hosting. | ADM / IT | Written decision | SH-GIT-001 | YES | | | UNANSWERED | | |
| SH-GIT-003 | Ownership | P1 | Practical hosting | Where will the repository be hosted — school Git, a personal account, or local only? | Determines backup and access control. | ADM / INT | Decision record | SH-GIT-002 | YES | | | UNANSWERED | | |
| SH-GIT-004 | Secrets | P0 | Hard constraint | Confirm that no credentials, keys, tokens or private network details may be committed. | Absolute repository hygiene rule. | INT | Design confirmation | SH-PRIV-020 | YES | | | UNANSWERED | | |
| SH-GIT-005 | Secrets | P1 | Practical need | How will secrets be managed if the project progresses to implementation? | Git cannot hold them. | INT / IT | Policy | SH-GIT-004 | NO | | | UNANSWERED | | |
| SH-GIT-006 | Documentation | P1 | Continuity | Is documentation expected in a particular format or location? | School or team conventions. | ADM | Convention | SH-GIT-001 | NO | | | UNANSWERED | | |
| SH-GIT-007 | Documentation | P1 | Traceability | How will design decisions be recorded and justified? | A decision log prevents re-litigation. | INT | Process | SH-GIT-001 | NO | | | UNANSWERED | | |
| SH-GIT-008 | Documentation | P2 | Reuse | Should the documentation be structured for handover to future maintainers? | Affects effort now versus later. | ADM / INT | Decision record | SH-GIT-006 | NO | | | UNANSWERED | | |
| SH-GIT-009 | Configuration | P1 | Reproducibility | How will environment and dependency versions be pinned and reproduced? | Required for a reliable rebuild. | INT | Process | SH-GIT-001 | NO | | | UNANSWERED | | |
| SH-GIT-010 | Configuration | P1 | Device-specific | How will robot-specific and site-specific configuration be separated from code? | Prevents a site-specific fix being shipped as a default. | INT | Design | SH-GIT-009 | NO | | | UNANSWERED | | |
| SH-GIT-011 | Data | P1 | Privacy by design | What is the policy for maps, routes and telemetry data in version control? | Route data may reveal school layout. | ADM / INT | Policy | SH-PRIV-022 | YES | | | UNANSWERED | | |
| SH-GIT-012 | Data | P2 | Sensitivity | May floor plans or CCTV-derived data be stored in the repository at all? | Layout data is sensitive even without personal data. | ADM | Policy | SH-GIT-011 | NO | | | UNANSWERED | | |
| SH-GIT-013 | Deployment | P2 | Future phase | Will there be a deployment process, or is the system developer-run? | Affects documentation effort. | INT | Decision record | SH-GIT-001 | NO | | | UNANSWERED | | |
| SH-GIT-014 | Deployment | P2 | Future phase | Is a staging environment separate from the production server required? | Standard practice for safety-relevant systems. | INT / IT | Requirement | SH-GIT-013 | NO | | | UNANSWERED | | |
| SH-GIT-015 | Quality | P1 | Evidence | What test and review practice is expected before any change is accepted? | Governs how code will be assessed later. | ADM / INT | Process | SH-GIT-001 | NO | | | UNANSWERED | | |
| SH-GIT-016 | Quality | P2 | Traceability | Should every future release record which robot firmware it was verified against? | Firmware differences silently change behaviour. | INT | Process | SH-GIT-009 | NO | | | UNANSWERED | | |
| SH-GIT-017 | Licensing | P2 | Legal hygiene | Under what licence will any future code or documentation be released? | Must be settled before external release. | ADM | Decision record | SH-GIT-002 | NO | | | UNANSWERED | | |
| SH-GIT-018 | Licensing | P2 | Dependency review | Must third-party dependencies be reviewed for licence compatibility before use? | Some robotics dependencies are GPL or unlicensed. | INT | Process | SH-GIT-017 | NO | | | UNANSWERED | | |
| SH-GIT-019 | History | P2 | Continuity | What is the retention and archival policy for project documentation? | Supports handover. | ADM | Policy | SH-GIT-006 | NO | | | UNANSWERED | | |
| SH-GIT-020 | Communication | P2 | Coordination | How will the school and the team communicate and record decisions between gates? | Prevents informal approvals being assumed later. | ADM | Process | SH-GIT-001 | NO | | | UNANSWERED | | |

---

# T. Architecture decisions and project go/no-go criteria — `SH-DEC`

| Question ID | Subcategory | Priority | Priority Rationale | Question | Why It Matters | Intended Respondent | Evidence Required | Dependency | Blocks? | Answer | Evidence / Source / Attachment | Status | Follow-up Required | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SH-DEC-001 | Scope | P0 | Defines the deliverable | What is the **minimum viable safe first demonstration**? | Scope must be bounded before building anything. | ADM / INT | Written scope | — | YES | | | UNANSWERED | | |
| SH-DEC-002 | Scope | P0 | Bounded ambition | Confirm the first demonstration involves **one floor only**. | Multi-floor depends on unsolved floor identification. | ADM / INT | Decision record | SH-DEC-001 | YES | | | UNANSWERED | | |
| SH-DEC-003 | Scope | P0 | Bounded ambition | Confirm the first demonstration involves **no stairs and no lift use**. | Neither is established or permitted today. | ADM / INT | Decision record | SH-DEC-002 | YES | | | UNANSWERED | | |
| SH-DEC-004 | Scope | P1 | Bounded ambition | Confirm the first demonstration uses a **fixed, pre-verified route** only. | No exploration during a demonstration. | ADM / INT | Decision record | SH-DEC-002 | YES | | | UNANSWERED | | |
| SH-DEC-005 | Scope | P1 | Bounded ambition | How many destinations will be in the first pilot? | Four to six is a realistic target. | ADM / INT | Decision record | SH-DEC-001 | YES | | | UNANSWERED | | |
| SH-DEC-006 | Scope | P2 | Explicitly excluded | Which proposed features are **postponed or excluded**? | Prevents scope creep from ambiguity. | ADM / INT | Decision record | SH-DEC-001 | NO | | | UNANSWERED | | |
| SH-DEC-007 | Navigation | P1 | Core architecture | Is the navigation stack self-built, vendor-provided, or both? | Determines procurement, licensing and effort. | INT | Decision record | SH-GO2-013 | YES | | | UNANSWERED | | |
| SH-DEC-008 | Network | P1 | Core architecture | Is the robot network wired Ethernet, Wi-Fi, or a hybrid? | Ethernet is the only manufacturer-supported path. | IT / INT | Decision record | SH-NET-003 | YES | | | UNANSWERED | | |
| SH-DEC-009 | Server | P1 | Core architecture | Where does the server run, and who operates it? | Determines resilience and ownership. | ADM / INT | Decision record | SH-SRV-001 | YES | | | UNANSWERED | | |
| SH-DEC-010 | Occupancy | P1 | Core architecture | Is occupancy awareness in scope at all for the first pilot? | A **manual** corridor state may be sufficient and far cheaper. | ADM / INT | Decision record | SH-CONG-001 | YES | | | UNANSWERED | | |
| SH-DEC-011 | Occupancy | P2 | Prefer simplicity | Could occupancy be supplied manually by staff initially, avoiding all sensing? | Removes the entire CCTV and privacy burden. | ADM / INT | Decision record | SH-DEC-010 | NO | | | UNANSWERED | | |
| SH-DEC-012 | AI | P0 | Safety architecture | Confirm no language model may reach the robot, and the demonstration works with it disabled. | Hard design principle. | ADM / INT | Design confirmation | — | YES | | | UNANSWERED | | |
| SH-DEC-013 | AI | P1 | Safety architecture | Confirm the task validator is a deterministic, auditable function. | Prevents unreviewable behaviour. | ADM / INT | Design confirmation | SH-DEC-012 | YES | | | UNANSWERED | | |
| SH-DEC-014 | Safety | P0 | Non-negotiable | Confirm the system cannot operate without a human spotter present. | Design principle. | ADM / INT | Design confirmation | — | YES | | | UNANSWERED | | |
| SH-DEC-015 | Safety | P1 | Architecture invariant | Confirm the robot can always be stopped without the dashboard. | Software must never be the only control. | ADM / INT | Design confirmation | SH-DEC-014 | YES | | | UNANSWERED | | |
| SH-DEC-016 | Gates | P0 | Prevents premature progress | Are the go/no-go gates accepted as the control on project progression? | Without gates, pressure builds to skip them. | ADM | Written acceptance | SH-DEC-001 | YES | | | UNANSWERED | | |
| SH-DEC-017 | Gates | P1 | Governance | Who has authority to declare a gate passed or failed? | Gates must have an owner. | ADM | Named authority | SH-DEC-016 | YES | | | UNANSWERED | | |
| SH-DEC-018 | Gates | P1 | Correctness | Confirm a gate may be marked **BLOCKED** or **NOT APPLICABLE**, but never VERIFIED without evidence. | Prevents false assurance. | ADM | Written confirmation | SH-DEC-016 | YES | | | UNANSWERED | | |
| SH-DEC-019 | Go/no-go | P0 | Final authority | What is the explicit condition under which the project is **stopped**? | A stop criterion must exist and be agreed in advance. | ADM | Written criteria | SH-DEC-016 | YES | | | UNANSWERED | | |
| SH-DEC-020 | Go/no-go | P1 | Continued funding | What evidence justifies continuing investment in later phases? | Defines success beyond the exhibition. | ADM | Written criteria | SH-DEC-019 | YES | | | UNANSWERED | | |

---

# Prioritised decision queue

## 1. Questions that must be answered immediately (P0, highest dependency)

| # | IDs | Theme | Why first |
|---|---|---|---|
| 1 | `SH-GO2-008` | Written authorisation for secondary development | **Nothing can proceed without this.** Technical capability is not permission. |
| 2 | `SH-GO2-001`, `-002` | Robot identity and firmware | Determines everything downstream |
| 3 | `SH-PRIV-002`, `SH-ADM` | Data Fiduciary approval in principle | Gates all data work |
| 4 | `SH-SAFE-001`, `-005`, `-008` | Emergency stop, spotter, exclusion zone | Gates any powered operation |
| 5 | `SH-NET-003`, `SH-SRV-001` | Network segment and server host | Gates all technical work |
| 6 | `SH-MAP-012`, `-013` | Who may verify a destination, and how | Gates all navigation |
| 7 | `SH-STAIR-001` | **Measure the stair risers** | May eliminate stairs entirely, in an afternoon |
| 8 | `SH-LIFT-014`, `-016` | EyeRIS retrofit and robot provisioning | Longest lead time; one email |
| 9 | `SH-STAIR-012`, `SH-DEL-001`, `SH-DEC-002` | Is there a single-floor route; what do teachers actually need | May eliminate vertical transit entirely |
| 10 | `SH-DEC-001`, `-016`, `-019` | Minimum demo scope and gate authority | Prevents scope creep |

## 2. Questions researchable in parallel without school input

| IDs | Activity |
|---|---|
| `SH-GO2-010` to `-022` | Read SDK source; document interfaces, battery fields, recovery semantics |
| `SH-CCTV-024` | Compare mmWave against CCTV for the same CLEAR/MODERATE/BUSY answer |
| `SH-RAD-001` to `-017` | Build a link budget and site-survey plan; decide whether LoRa is justified |
| `SH-MAP-007`, `-016` | Design the floor-identity mechanism and marker failure handling |
| `SH-DEC-007` to `-015` | Draft architecture decisions and their reversal conditions |
| `INT` | Run `unitree_mujoco` — **the entire software stack can be built with no robot and no school answer** |

## 3. Questions requiring school administration

`SH-GIT-001` to `-003`, `SH-PRIV-001` to `-006`, `SH-SAFE-001`, `-005` to `-012`,
`SH-ENV-001`, `SH-BLOCK-001`, `SH-MAP-009` (checkpoint approval), `SH-MAP-021`,
`SH-STAIR-013`, `SH-LIFT-018` to `-026`, `SH-DEC-001` to `-006`, `SH-BUD-001`,
`SH-BUD-011`, `SH-BUD-017`

## 4. Questions requiring IT

`SH-NET-001` to `-024`, `SH-CCTV-001` to `-003`, `SH-SRV-010`, `SH-SRV-014`,
`SH-RAD-020`

## 5. Questions requiring the robot owner or Unitree support

`SH-GO2-001` to `-024`, `SH-STAIR-014`, `SH-STAIR-015`, `SH-BAT-016`, `SH-BAT-018`

## 6. Questions requiring the lift manufacturer or authorised service provider

`SH-LIFT-001` to `-026` — **all of them.** In particular `SH-LIFT-014`, `-015`,
`-016` (EyeRIS), `-009` (any sanctioned third-party interface), and `-019`
(button actuator permission).

## 7. Questions requiring facilities and housekeeping

`SH-CONG-005` to `-010`, `SH-ENV-001` to `-004`, `SH-STAIR-001` to `-011`,
`SH-MAP-023`, `-024`, `SH-BLOCK-005` to `-017`, `SH-BAT-015` to `-021`

## 8. Questions requiring a privacy or school-policy decision

`SH-PRIV-001` to `-022`, `SH-CCTV-012` to `-023`, `SH-NET-018`, `SH-DEL-002`

## 9. Questions that can wait until after a controlled pilot

`SH-RAD-003` to `-017`, `SH-CCTV-007` to `-011`, `SH-CONG-015` to `-017`,
`SH-MAP-019`, `SH-BAT-020`, `SH-TIME-015` to `-017`, `SH-EXPO-008`, `SH-EXPO-012`,
`SH-EXPO-018` to `-020`, `SH-BUD-003`, `SH-BUD-005`, `SH-BUD-006`, `SH-BUD-009`,
`SH-BUD-010`, `SH-GIT-008`, `SH-GIT-012` to `-020`

---

# Question count summary

| Category | Prefix | Questions | P0 | P1 | P2 | P3 |
|---|---|---|---|---|---|---|
| A. Go2 hardware, SDK, interfaces | `SH-GO2` | 24 | 5 | 14 | 5 | 0 |
| B. School network, Wi-Fi, LAN | `SH-NET` | 24 | 5 | 10 | 8 | 1 |
| C. Local server, dashboard, auth | `SH-SRV` | 20 | 5 | 8 | 7 | 0 |
| D. LoRa, ESP32, telemetry | `SH-RAD` | 20 | 3 | 7 | 9 | 1 |
| E. CCTV, CV, occupancy, privacy | `SH-CCTV` | 24 | 5 | 12 | 7 | 0 |
| F. Congestion, dynamic routing | `SH-CONG` | 20 | 3 | 11 | 6 | 0 |
| G. Wet floors, rain, lighting | `SH-ENV` | 20 | 3 | 12 | 5 | 0 |
| H. Obstacles, construction, blockage | `SH-BLOCK` | 20 | 6 | 8 | 6 | 0 |
| I. Elevators | `SH-LIFT` | 26 | 12 | 13 | 1 | 0 |
| J. Stairs, ramps | `SH-STAIR` | 20 | 7 | 8 | 5 | 0 |
| K. Maps, routes, checkpoints, ArUco | `SH-MAP` | 24 | 7 | 14 | 3 | 0 |
| L. Battery, charging, recovery | `SH-BAT` | 22 | 6 | 13 | 3 | 0 |
| M. Schedules, lunch, windows | `SH-TIME` | 20 | 2 | 13 | 5 | 0 |
| N. Delivery workflow | `SH-DEL` | 22 | 5 | 12 | 5 | 0 |
| O. Safety, e-stop, failure | `SH-SAFE` | 26 | 16 | 9 | 0 | 1 |
| P. Permissions, privacy, retention | `SH-PRIV` | 22 | 12 | 10 | 0 | 0 |
| Q. Exhibition, evaluation | `SH-EXPO` | 20 | 7 | 8 | 5 | 0 |
| R. Budget, installation, ownership | `SH-BUD` | 20 | 3 | 10 | 6 | 1 |
| S. Git, docs, configuration | `SH-GIT` | 20 | 3 | 8 | 8 | 1 |
| T. Architecture decisions, go/no-go | `SH-DEC` | 20 | 7 | 11 | 2 | 0 |
| **TOTAL** | | **434** | **122** | **211** | **99** | **2** |

**Read the totals honestly.** 122 questions are safety, permission, or
fundamental-feasibility blockers. 211 are architecture blockers. **Only 99 are
pilot requirements and 2 are genuinely optional.** That distribution is the
honest shape of this project: most of the work ahead is establishing what is
permitted and what is possible, not building.

Three categories are dominated by P0 by design — elevators (12 of 26),
safety (16 of 26) and permissions/privacy (12 of 22). In these areas a wrong
assumption is not an inconvenience; it is a legal exposure or an injury.

---

# Structural gaps and limitations — stated deliberately

This section exists so that a reader can see where the questionnaire is
deliberately thinner, and why. **No padding was added to reach a target count.**

| Category | Depth | Explanation |
|---|---|---|
| **D. LoRa / ESP32** | 20 questions rather than more | The research conclusion is that **this hardware should probably not be built at all.** `SH-RAD-001` is a P0 justification gate: if no demonstrated problem exists, the remaining 19 questions are moot. Adding depth here would imply a commitment the evidence does not support. |
| **S. Git / documentation** | 20 questions | Many are answered by us (`INT`) rather than by the school. Kept at 20 because repository ownership and secrets policy genuinely require school decisions. |
| **I. Elevators** | 26 questions — deepest category | Justified. This is the highest-consequence unknown: a statutory prohibition, a Supreme Court liability finding, an unverified manufacturer interface, and a documented physical safety scenario. |
| **O. Safety** | 26 questions, 18 of them P0 | Justified. Student safety is the primary constraint, and every hazard class in the brief maps to at least one question. |
| **Ramp / wet-floor sensing** | Covered within `SH-STAIR` and `SH-ENV` rather than as separate categories | The brief's categories are preserved; ramps sit naturally with stairs and hazards with environment. Splitting them would have created artificial sub-boundaries without improving coverage. |
| **No question asks a school to answer something they cannot know** | — | For example, no question asks facilities to assess whether the Go2 can climb stairs. That is a measurement for us to take once permission exists. |

---

# Closing statement

**434 questions. Zero answers. Zero evidence supplied.**

That is the correct state of this document. Every row is a genuine unknown that
either only the school can answer, or that requires a physical measurement, a
written approval, or a supervised test to resolve.

**No capability described anywhere in the SHANA documentation is operational.**
Nothing in this repository has been connected to, commanded on, or tested
against a robot, a lift, a network, or a camera.

The project must not advance to implementation until the P0 questions in the
decision queue are answered or explicitly accepted as constraints by an
authorised school decision-maker.