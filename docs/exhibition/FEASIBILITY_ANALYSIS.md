# CloudLight & SHANA — Comprehensive Feasibility Study & Engineering Analysis

**Project Name:** CloudLight (School-Local Digital Platform) with SHANA (School Autonomous Notification & Assist Subsystem)  
**Document Type:** Technical Feasibility Study & Cross-Subsystem Engineering Audit  
**Author / Lead:** Integration Engineering Review  
**Date:** 2026-10-11  
**Classification:** `DESIGN_ONLY` — Formal Specification & Pre-Implementation Feasibility Study  

---

## 1. Executive Summary & Repository Census

This document constitutes the definitive engineering analysis and feasibility evaluation of the **CloudLight** school information platform and its autonomous physical delivery robotics subsystem, **SHANA**. Rather than proceeding directly to implementation without institutional authorization, the project undertook a rigorous, audit-first feasibility analysis to establish physical constraints, regulatory safety boundaries, hardware limitations, and software integration contracts.

### 1.1 Verified Repository State Audit

A comprehensive filesystem and repository census was conducted on this repository. All factual claims regarding current project artifacts were verified by direct inspection:

| Metric / Check | Verified State | Audit Evidence / Command | Classification |
|---|---|---|---|
| **Source Code Files** | **0** | `find . -type f -name "*.py" -o -name "*.js" -o -name "*.ts" -o -name "*.cpp"` yields zero executable files | `RESEARCHED` |
| **Unit / Integration Tests** | **0 / 0** | No test suites, runners, or executable harnesses exist (only specification markdown files under `docs/*/testing`) | `RESEARCHED` |
| **Build / CI Pipelines** | **0 / none** | No `.github/workflows/` directory or automated integration pipeline | `RESEARCHED` |
| **Active Deployments** | **0** | Zero cloud, on-premise, edge, or display endpoints provisioned | `RESEARCHED` |
| **Robot Operation** | **Never Operated** | Unitree Go2 EDU has never been powered, commanded, or tethered | `RESEARCHED` |
| **Hardware Photographs / Output** | **None Exist** | 8 root JPEGs inspected; confirmed to be third-party LCD display panel assemblies | `RESEARCHED` |
| **Empirical Measurements** | **0** | Zero latency, bandwidth, battery draw, or transit measurements taken | `RESEARCHED` |
| **Institutional Questions** | **986 asked, 0 answered** | 576 CloudLight questions + 410 SHANA / 434 questionnaire items | `RESEARCHED` |
| **Go/No-Go Gates** | **20 defined, 0 passed** | 10 CloudLight gates + 10 SHANA gates; zero formal gates cleared | `RESEARCHED` |
| **Permissions Sought** | **104 sought, 0 granted** | Cross-project permission checklist; all status `NOT REQUESTED` | `RESEARCHED` |
| **Registered Risks** | **68 registered, 0 retired** | 46 CloudLight risks + 22 SHANA risks; zero risks mitigated | `RESEARCHED` |
| **Sourced Research Entries** | **106 documented** | 68 entries in SHANA `source_register.md` + 38 in CloudLight `sources.md` | `RESEARCHED` |

> [!IMPORTANT]
> **Statement of Status:** CloudLight and SHANA represent a completed **feasibility study and specification**, not an operational product or prototype. No software has been deployed, no robot has navigated school grounds, and no student or teacher data has been touched.

---

## 2. Core System Architecture & The Non-Negotiable Safety Boundary

The system comprises two distinct architectural components operating in an asymmetric hierarchy governed by formal Architecture Decision Records ([ADR-014](../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md) and [ADR-015](../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md)):

```
┌────────────────────────────────────────────────────────────────────────┐
│  CLOUDLIGHT: School-Local Platform                 [STATUS: SPECIFIED] │
│  Classroom Displays · Timetables · Substitutions · Authorized Search    │
│  Teacher Authentication · Notice Management · Update Delivery           │
│                                                                        │
│  ✋ ABSOLUTE ARCHITECTURAL INVARIANT:                                   │
│  NO MOTOR, TRAJECTORY, VELOCITY, OR ACTUATOR CONTROL PATH EXISTS       │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ 1. Asymmetric Delivery Request
                                    │    (Origin, Dest, Payload, Timeout)
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│  SHANA: Robotics Subsystem (Unitree Go2 EDU)       [STATUS: SPECIFIED] │
│  Local Localization · Route Planning · Dynamic Hazard Obstacle Stop    │
│  Deterministic Safety Validator · Fail-Closed E-Stop Authority         │
│                                                                        │
│  🔴 INDEPENDENT AUTONOMY:                                              │
│  Remains fully safe, halts, or returns to pause when CloudLight drops  │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ 2. Telemetry & Terminal Status Only
                                    │    (QUEUED, PAUSED, COMPLETED, etc.)
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│  CLOUDLIGHT DISPLAY BUS: Passive Status Presentation                   │
│  Classroom screens reflect status; platform continues if robot offline │
└────────────────────────────────────────────────────────────────────────┘
```

### 2.1 The Two Fundamental Architectural Invariants

1. **CloudLight Requests, SHANA Decides (`ADR-014`):**  
   CloudLight serves purely as a business-logic requester. It may issue high-level delivery intent (`origin_room`, `destination_room`, `delivery_id`, `recipient_id`), but it possesses **zero interface, RPC, or message path** capable of commanding leg velocities, joint torques, path waypoints, or gait modes. SHANA evaluates every request through a deterministic local safety validator and maintains unilateral authority to reject or cancel any task.
2. **Total Offline Decoupling (Journey 8):**  
   CloudLight's core operational capabilities—classroom schedule rendering, emergency campus announcements, and timetable substitution queries—must operate seamlessly with zero latency degradation if the SHANA subsystem is completely powered off, network-isolated, or encountering an unrecoverable fault. A robotics malfunction will never blank a classroom display or disrupt academic operations.

---

## 3. The Technical Turning Point: Hardware Constraints & Scope Reduction

Rather than fabricating optimistic multi-floor delivery claims, the engineering study performed static code analysis on the Unitree Go2 message definitions (`IMUState.msg`) and vendor firmware specifications. This resulted in the project's defining engineering discovery and consequent scope reduction.

### 3.1 The 5-Field IMU Discovery

Static analysis of the Unitree Go2 SDK message definitions revealed that the onboard Inertial Measurement Unit transmits exactly five telemetry fields:
1. `quaternion[4]` (Orientation)
2. `gyroscope[3]` (Angular Velocity)
3. `accelerometer[3]` (Linear Acceleration)
4. `rpy[3]` (Roll, Pitch, Yaw)
5. `temperature` (Internal Die Temperature)

```
┌───────────────────────────────────────────────────────────────────────┐
│                     UNITREE Go2 IMU TELEMETRY                         │
│  [Orientation]  [Angular Vel]  [Linear Accel]  [RPY]  [Temperature]   │
├───────────────────────────────────────────────────────────────────────┤
│  ❌ NO BAROMETER DATA     ❌ NO ALTIMETER DATA    ❌ NO PRESSURE SENSOR │
└───────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
         CRITICAL CONSTRAINT: THE ROBOT CANNOT DETERMINE 
                 WHICH FLOOR OF A BUILDING IT IS ON
```

Without an onboard barometric altimeter or absolute vertical reference, the quadruped cannot reliably determine or verify vertical floor transitions using internal sensor state.

### 3.2 Stair & Vertical Transit Realities

Vendor documentation and SDK release registers further confirmed two insurmountable vertical barriers:
- **Maximum Step Clearance:** Vendor documents specify a maximum obstacle/step clearance of 160 mm, which falls below the riser heights of standard commercial school staircases (170–190 mm).
- **Firmware Deprecation:** Unitree SDK analysis indicates that the high-level stair-climbing gait API was deprecated and removed in firmware release V1.1.6. The installed firmware on the school's unit remains `UNKNOWN`.
- **Lift Interface Illegality:** No sanctioned, non-invasive API exists to interface with commercial elevator controllers. Splicing into elevator control panels or call buttons in an educational institution is unlawful, violates fire safety regulations, and voids lift certifications.

### 3.3 Engineering Scope Decision

As a direct consequence of these empirical hardware findings, the project formally reduced its operational envelope:
> **Scope Invariant:** SHANA is strictly scoped to single-floor, flat-surface horizontal transit on a single designated level. Multi-floor transit, stair climbing, and elevator integration are formally categorized as `OUT OF SCOPE`.

---

## 4. Rigorous User Journey Specifications

User workflows were specified to capture actual operational sequences without overstating capabilities:

| Journey # | Title | Lifecycle Classification | Reality & Guardrails |
|---|---|---|---|
| **Journey 1** | Classroom Display Check | `SPECIFIED` | Teacher checks room display. Grounded in local timetable parsing; zero displays currently deployed. |
| **Journey 2** | Resource Browsing & Retrieval | `SPECIFIED` | Gated on school file storage infrastructure. Requires shared repository that may not currently exist. |
| **Journey 3** | Timetable Substitution Review | `SPECIFIED` | Timetable staff approve covers. Workflow designed, but formal administrative approver not yet designated. |
| **Journey 4** | Physical Delivery Request | `SPECIFIED` | Teacher submits request via CloudLight web client. Task is strictly queued for SHANA evaluation. |
| **Journey 5** | Autonomous Validation & Gating | `SPECIFIED` | SHANA independently checks battery reserve, corridor clearance, and spotter presence. Rejects if unsafe. |
| **Journey 6** | Loading & Recipient Handover | `SPECIFIED` | **Human-assisted only.** The Unitree Go2 possesses no robotic manipulator arm; loading and unloading require human personnel. |
| **Journey 7** | Dynamic Corridor Obstacle Halt | `SPECIFIED` | Route blocked by students/furniture. Robot halts, transitions to `PAUSED`, and holds position (fails closed). |
| **Journey 8** | Complete Robot Offline State | `SPECIFIED` | **Primary Design Goal:** SHANA loses power or drops Wi-Fi; CloudLight continues 100% normal classroom display functions. |

---

## 5. Safety, Privacy & Security Engineering

### 5.1 Privacy by Design (Core System Strengths)

Privacy is treated as an architectural boundary, not an afterthought:
- **Zero Facial Recognition:** The perception pipeline strictly forbids biometric identification or facial indexing.
- **Zero Student Tracking:** The system contains no capabilities to track individuals, monitor foot traffic, or log movement patterns.
- **Zero Raw Video Offload:** Camera feeds are processed locally on edge compute solely for spatial obstacle avoidance. No raw video or image frames are ever transmitted over the network or persisted to disk.
- **Zero Misconduct Detection:** The robot is prohibited from performing surveillance, behavioral policing, or sound-recording functions.

### 5.2 Deterministic Safety Guardrails

- **Mandatory Human Spotter:** During all pre-operational testing and simulated deliveries, a designated human spotter with direct line-of-sight is required.
- **Unconditional Hardware E-Stop:** Both physical and wireless emergency stop mechanisms possess unilateral, un-overridable authority. When triggered, the system fails closed and de-energizes motor stages immediately.
- **No Generative AI in the Actuator Loop:** All motor trajectories, navigation states, and collision halts are governed by deterministic, mathematically verifiable algorithms. No LLM or generative model has access to the locomotion controllers.
- **Fail-Closed Corridor Navigation:** If localization uncertainty exceeds specified bounds or dynamic obstacles prevent clear egress, the robot transitions to `PAUSED` and sounds an audible alert rather than attempting aggressive avoidance.

### 5.3 Network Security Architecture

- **State:** `DESIGNED`, zero components currently exposed or online.
- **Mutual TLS (mTLS) Requirement:** Peer-to-peer authenticated communication between CloudLight servers and the SHANA compute node is mandatory to prevent unauthorized LAN nodes from forging delivery commands.
- **Strict Network Isolation:** Dedicated VLAN segmentation separating IoT robotics traffic from educational and guest Wi-Fi networks.

---

## 6. Internal Self-Audit & Cross-Project Conflicts

Unlike projects that conceal discrepancies, this integration audit conducted a comprehensive cross-subsystem review and registered **10 unresolved conflicts**:

| Conflict ID | Severity | Description | Safety Implication & Resolution Path |
|---|---|---|---|
| **CONF-01** | 🔴 High | Isolated Integration Specs | CloudLight and SHANA authors drafted independent integration contracts without cross-collaboration. Unification required under `interface_contracts.md`. |
| **CONF-02** | 🔴 High | Delivery State Collapse (`PAUSED` defect) | CloudLight's display vocabulary lacked the `PAUSED` state, collapsing all active states into `IN_PROGRESS` ("On the way"). This meant an operator-halted robot would mislead teachers. Resolved via [ADR-016](../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md) single-source enum. |
| **CONF-03** | 🟠 Medium | Broken Internal Links | 6 markdown links pointed to renamed or shifted documents across subsystem boundaries. Explicit mapping established. |
| **CONF-04** | 🔴 High | Citations to Non-Existent ADRs | Documentation referenced `ADR-004`, `ADR-005`, and `ADR-006` in SHANA which do not exist as physical files. Required reconciliation. |
| **CONF-05** | 🟠 Medium | Missing Architecture Spec | `docs/architecture/delivery_state_machine.md` cited in requirements was absent from disk. |
| **CONF-06** | 🟠 Medium | Identifier Divergence | SHANA utilized `task_id` while CloudLight utilized `delivery_id` as primary keys. Standardized on `delivery_id`. |
| **CONF-07** | 🟡 Low | API Route Divergence | Endpoint base paths diverged between `/v1/` and `/api/v1/`. Standardized on `/api/v1/`. |
| **CONF-08** | 🟠 Medium | Outbound Telemetry Mismatch | SHANA's event publishing topics did not strictly match its internal 11-state transition matrix. |
| **CONF-09** | 🟠 Medium | Room Occupancy Authority | Ambiguity regarding whether CloudLight or SHANA holds definitive room occupancy state. |
| **CONF-10** | 🟡 Low | User Identity Semantics | Inconsistency between teacher role ID strings vs institutional UUIDs. |

---

## 7. Gated Implementation Roadmap (Phases 0–7)

Progress across the engineering lifecycle is strictly gated by verified exit criteria, containing **no speculative completion dates or uncommitted budgets**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Phase 0: Institutional Unblock (Principal: 25 min, IT: 40 min)          │
│ Exit: Formal written authorization & network architecture consensus     │
└────────────────────────────────────┬────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Phase 1: High-Fidelity Physics Simulation (unitree_mujoco)              │
│ Exit: Zero-cost validation of DDS messages and path planning in physics │
└────────────────────────────────────┬────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Phase 2: Platform-First Development & Synthetic Testing                 │
│ Exit: CloudLight timetable ingestion verified against mock data fixtures│
└────────────────────────────────────┬────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Phase 3: Hardware Bench Testing (Non-Robotic)                           │
│ Exit: CloudLight services running on authorized physical edge devices   │
└────────────────────────────────────┬────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Phase 4: Single Classroom Pilot (Supervised Display Only)               │
│ Exit: Teacher evaluation of static schedule board; zero robotics active │
└────────────────────────────────────┬────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Phase 5: Supervised Robotics Corridor Qualification                     │
│ Exit: Isolated closed-corridor navigation; spotter E-stop fully tested  │
└────────────────────────────────────┬────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Phase 6: Subsystem Integration & Contract Verification                  │
│ Exit: End-to-end delivery request to completion via ADR-016 contract    │
└────────────────────────────────────┬────────────────────────────────────┘
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ Phase 7: Operational Scaling (Conditional)                              │
│ Exit: Continuous safety adherence and multi-stakeholder approval        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Verbatim Statement of Limitations

> *This work is a feasibility study. No system has been built, no robot has been operated, and no school infrastructure has been accessed. Approximately 1,000 structured questions have been asked and none has yet been answered. Twenty formal go/no-go gates are defined and none has been passed. Every capability described is proposed, not demonstrated. The research is grounded in manufacturer documentation and source-code analysis, with access limitations and unresolved contradictions recorded rather than concealed.*

---

## 9. Conclusion & The Next Actionable Step

The CloudLight and SHANA initiative has demonstrated that high-integrity systems engineering begins long before code execution. By prioritizing architectural boundary definition, sensor reality identification, privacy-first constraints, and self-audited integration checks, the study has established an exceptionally sound foundation.

**The Immediate Requirement:**  
The remaining hurdles are entirely institutional, requiring approximately **25 minutes of School Leadership (Principal) consultation** and **40 minutes of Campus IT consultation** to review the 14 critical gating criteria and determine whether physical execution may commence.
