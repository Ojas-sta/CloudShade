# School Science & Innovation Exhibition — Project Submission Form

---

## Part A: Project Identification & Classification

1. **Project Title:**  
   **CloudLight & SHANA: A Grounded Feasibility Study for a School-Local Information Platform and Autonomous Single-Floor Quadruped Delivery Subsystem** `[SPECIFIED]`
2. **Subsystems:**  
   - **CloudLight:** School-local digital platform (classroom schedules, substitutions, authorized resource retrieval) `[SPECIFIED]`  
   - **SHANA:** School Autonomous Notification & Assist (single-floor physical delivery on school-owned Unitree Go2 EDU) `[SPECIFIED]`
3. **Project Category:** Engineering & Systems Architecture / Autonomous Robotics Feasibility Study `[RESEARCHED]`
4. **Current Implementation Status:** `DESIGN_ONLY` — Formal Pre-Implementation Feasibility Study `[RESEARCHED]`
5. **Verified Codebase Inventory:** **0 Source Files** · **0 Tests Executed** · **0 CI Pipelines** · **0 Deployments** · **0 Robots Operated** `[RESEARCHED]`

---

## Part B: Project Specification & Engineering Feasibility

### 1. Problem Statement `[RESEARCHED]`
Academic institutions operate on fragmented operational data dispersed across disconnected media: static daily timetables, morning substitution notices, distributed lesson materials, and immediate classroom notifications. Concurrently, transporting physical items between campus departments—such as laboratory supplies, administrative files, and emergency classroom equipment—relies entirely on manual courier transit by faculty and staff. Crucially, the institutional cost, delay, and workload impact of these routine interruptions have **not yet been formally measured** `[UNKNOWN]`. This project addresses this systemic operational fragmentation through rigorous pre-implementation engineering and risk analysis, establishing verified physical constraints before writing code.

### 2. Innovative System Architecture & Safety Boundary `[SPECIFIED]`
The architecture separates digital information services from robotic actuation across an absolute, asymmetric safety boundary governed by formal decision record **ADR-014** `[RESEARCHED]`:
- **Asymmetric One-Way Request Path:** CloudLight acts purely as an informational requester (`delivery_id`, pickup room, destination room, payload type). It possesses **zero interface, API, or RPC path to command robot locomotion, path coordinates, velocities, or joint torques** `[SPECIFIED]`.
- **Independent Robotic Autonomy:** SHANA evaluates requests via a deterministic onboard safety validator and retains unilateral authority to reject, pause, or abort tasks. Crucially, **CloudLight operates with full availability when SHANA is powered off or offline**; robot faults never degrade classroom displays (Journey 8) `[SPECIFIED]`.
- **Privacy as an Absolute Constraint:** The design strictly excludes facial recognition, biometric student tracking, misconduct surveillance, and raw video offloading; camera feeds are processed strictly for local spatial obstacle avoidance `[SPECIFIED]`. Recipient handover is human-assisted as the robot has no manipulator arm `[RESEARCHED]`.

### 3. The Technical Turning Point & Sourced Feasibility Rigour `[RESEARCHED]`
Rather than assuming omnidirectional navigation, static analysis of the Unitree Go2 SDK message definitions (`IMUState.msg`) revealed that the robot reports exactly **5 IMU fields**: orientation, angular velocity, linear acceleration, roll-pitch-yaw, and temperature `[RESEARCHED]`. **With zero altimeter, pressure, or barometer data, the quadruped cannot sense which floor of a building it is on** `[RESEARCHED]`. Combined with a manufacturer-documented 160 mm obstacle step ceiling `[RESEARCHED]`, the removal of stair-gait APIs at firmware V1.1.6 `[RESEARCHED]`, and the legal prohibition of unsanctioned lift controller splicing `[RESEARCHED]`, the team formally **scoped the robotics subsystem strictly to single-floor horizontal transit** `[SPECIFIED]`. Furthermore, an internal integration self-audit uncovered **10 cross-project conflicts** (including defect CONF-02 where paused states collapsed into "on the way"), 6 broken links, and 3 missing ADR citations, recording them openly rather than concealing them `[RESEARCHED]`.

---

## Part C: Verified Feasibility Audit & Governance Matrix

| Governance Axis | Verified Count | Repository Source & Status |
|---|---|---|
| **Sourced Research Entries** | **106 entries** | 68 SHANA entries (`source_register.md`) + 38 CloudLight entries (`sources.md`) `[RESEARCHED]` |
| **Structured Institutional Questions** | **986 asked (0 answered)** | 576 CloudLight questions + 410 SHANA / 434 questionnaire items; none answered `[RESEARCHED]` |
| **Formal Go/No-Go Gates** | **20 gates (0 passed)** | 9 platform gates + 11 robotics gates; zero gates cleared `[RESEARCHED]` |
| **Registered System Risks** | **68 registered (0 retired)** | 46 CloudLight risks + 22 SHANA risks; all active, none retired `[RESEARCHED]` |
| **Administrative Permissions Sought** | **104 sought (0 granted)** | Cross-project permission checklist; all status `NOT REQUESTED` `[RESEARCHED]` |
| **System Requirements Specified** | **186 requirements (0 agreed)** | Platform and robotics requirements; none formally signed off `[SPECIFIED]` |

---

## Part D: Limitations Statement & Proportionate Next Step

> **Verbatim Statement of Limitations:**  
> *This work is a feasibility study. No system has been built, no robot has been operated, and no school infrastructure has been accessed. Approximately 1,000 structured questions have been asked and none has yet been answered. Twenty formal go/no-go gates are defined and none has been passed. Every capability described is proposed, not demonstrated. The research is grounded in manufacturer documentation and source-code analysis, with access limitations and unresolved contradictions recorded rather than concealed.* `[RESEARCHED]`

**The Proportionate Next Step:**  
No code should be written and no hardware operated until institutional clearance is established. The remaining blockers are institutional rather than technical: approximately **25 minutes of School Leadership (Principal) consultation** and **40 minutes of Campus IT consultation** are required to review the 14 critical gating criteria and determine whether a simulated physics test (`unitree_mujoco`) may proceed `[RESEARCHED]`.
