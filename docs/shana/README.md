# SHANA — Documentation Index

**SHANA** = the **S**chool **H**ousehold **A**utonomous **N**avigation **A**ssistant.
A proposed autonomous indoor school delivery robotics subsystem, operating as
part of **CloudLight**.

> ## ⚠️ NOTHING HERE IS OPERATIONAL
>
> No code has been written. No robot has been operated, connected to, or
> commanded. No network has been probed. No lift, camera, or CCTV system has been
> approached. **Every capability in this documentation is proposed, not proven.**
>
> **Tests actually executed to date: zero.** See
> [`testing/test_inventory.md`](testing/test_inventory.md).

---

## Read this first

| If you are… | Read, in this order |
|---|---|
| **The third agent preparing a handoff** | [`handoff/agent_handoff.md`](handoff/agent_handoff.md) |
| **A judge or exhibition proposer** | [`handoff/exhibition_evidence_inventory.md`](handoff/exhibition_evidence_inventory.md) |
| **A developer** | [`overview/current_project_state.md`](overview/current_project_state.md) → [`handoff/implementation_inventory.md`](handoff/implementation_inventory.md) |
| **CloudLight agent** | [`architecture/cloudlight_integration.md`](architecture/cloudlight_integration.md) |
| **Reviewing claims** | [`research/claim_evidence_matrix.md`](research/claim_evidence_matrix.md) |
| **Just scanning** | This page |

---

## 1. Overview

| Document | Purpose |
|---|---|
| [`overview/introduction.md`](overview/introduction.md) | What SHANA is and the problem it addresses |
| [`overview/goals_and_non_goals.md`](overview/goals_and_non_goals.md) | Objectives and **12 explicit non-goals** |
| [`overview/current_project_state.md`](overview/current_project_state.md) | 🔴 **What is actually built, now. Read this before anything else.** |
| [`overview/system_context.md`](overview/system_context.md) | System diagram, boundaries, actors |
| [`overview/glossary.md`](overview/glossary.md) | Terms; common confusions called out |
| [`overview/phase_summary.md`](overview/phase_summary.md) | Phase 0 outcome and next 10 actions |

## 2. Architecture

| Document | Purpose |
|---|---|
| [`architecture/architecture.md`](architecture/architecture.md) | End-to-end design, staged deployment |
| [`architecture/component_architecture.md`](architecture/component_architecture.md) | Layers and the AI/deterministic boundary |
| [`architecture/navigation_architecture.md`](architecture/navigation_architecture.md) | Navigation, localisation, mapping approach |
| [`architecture/communication_architecture.md`](architecture/communication_architecture.md) | Network topology and protocol selection |
| [`architecture/cloudlight_integration.md`](architecture/cloudlight_integration.md) | 🔴 **SHANA ↔ CloudLight interface boundary** |
| [`architecture/data_flow.md`](architecture/data_flow.md) | What flows, and what must never flow |
| [`architecture/task_state_machine.md`](architecture/task_state_machine.md) | Task states, entry conditions, abort paths |
| [`architecture/route_graph.md`](architecture/route_graph.md) | Verified route graph and destination model |
| [`architecture/architecture_decisions/`](architecture/architecture_decisions/) | ADRs with reversal conditions |

## 3. Hardware

| Document | Purpose |
|---|---|
| [`hardware/go2_edu_inventory.md`](hardware/go2_edu_inventory.md) | 🔴 **What the school robot actually is — and what is unconfirmed** |
| [`hardware/sdk_and_interfaces.md`](hardware/sdk_and_interfaces.md) | Verified SDK surfaces, topics, licences |
| [`hardware/network_interfaces.md`](hardware/network_interfaces.md) | Robot connectivity options and constraints |
| [`hardware/cameras_and_depth_sensors.md`](hardware/cameras_and_depth_sensors.md) | Cameras, depth, and what they cannot do |
| [`hardware/battery_and_charging.md`](hardware/battery_and_charging.md) | Telemetry, thresholds, charging, recovery |
| [`hardware/checkpoint_markers.md`](hardware/checkpoint_markers.md) | ArUco markers, floor identity, checkpoints |
| [`hardware/esp32_and_lora.md`](hardware/esp32_and_lora.md) | Radio telemetry — **deferred**, Indian regulation |

## 4. Research

| Document | Purpose |
|---|---|
| [`research/research_index.md`](research/research_index.md) | **Central index — every research activity** |
| [`research/source_register.md`](research/source_register.md) | Every source, with class, claims, limitations |
| [`research/research_log.md`](research/research_log.md) | **Append-only chronological log** |
| [`research/claim_evidence_matrix.md`](research/claim_evidence_matrix.md) | 🔴 **Claim → evidence, for exhibition statements** |
| [`research/cctv_source_register.md`](research/cctv_source_register.md) | CCTV / ONVIF / occupancy sources |
| [`research/appendix_india_rf_research.md`](research/appendix_india_rf_research.md) | India RF regulation, gazette citations, link budgets |

## 5. Feasibility

| Document | Purpose |
|---|---|
| [`feasibility/feasibility_report.md`](feasibility/feasibility_report.md) | Sourced technical findings and confidence |
| [`feasibility/constraints.md`](feasibility/constraints.md) | 🔴 **Hard constraints that cannot be traded off** |
| [`feasibility/assumptions.md`](feasibility/assumptions.md) | Every unverified assumption + reversal |
| [`feasibility/decision_log.md`](feasibility/decision_log.md) | 20 decisions made, 10 open, 16 rejected |
| [`feasibility/verification_plan.md`](feasibility/verification_plan.md) | How each unknown gets resolved |
| [`feasibility/go_no_go_gates.md`](feasibility/go_no_go_gates.md) | 🔴 **11 gates; 0 passed** |
| [`feasibility/open_questions_register.md`](feasibility/open_questions_register.md) | Blocking questions and who answers them |
| [`feasibility/questionnaire.md`](feasibility/questionnaire.md) | Legacy 75-question summary questionnaire |

## 6. Infrastructure

| Document | Purpose |
|---|---|
| [`infrastructure/school_network.md`](infrastructure/school_network.md) | LAN, Wi-Fi, robot subnet |
| [`infrastructure/mapping_and_localisation.md`](infrastructure/mapping_and_localisation.md) | Map model, per-floor maps, floor identity |
| [`infrastructure/cctv_feasibility.md`](infrastructure/cctv_feasibility.md) | 🔴 **CCTV: recommended NOT to use** |
| [`infrastructure/elevator_feasibility.md`](infrastructure/elevator_feasibility.md) | 🔴 **Lifts: no sanctioned integration exists** |
| [`infrastructure/stairs_and_ramps.md`](infrastructure/stairs_and_ramps.md) | Stairs: **not established**, excluded |
| [`infrastructure/school_route_survey.md`](infrastructure/school_route_survey.md) | Survey protocol — **not started** |

## 7. Safety

| Document | Purpose |
|---|---|
| [`safety/safety_case.md`](safety/safety_case.md) | Draft safety argument — **no evidence yet** |
| [`safety/hazard_analysis.md`](safety/hazard_analysis.md) | Failure modes and mitigations |
| [`safety/risk_register.md`](safety/risk_register.md) | 21 risks, 102 outstanding permissions |
| [`safety/emergency_procedures.md`](safety/emergency_procedures.md) | Stop, recovery, fire, injury |
| [`safety/student_privacy.md`](safety/student_privacy.md) | 🔴 **No identification. Ever.** |
| [`safety/operating_restrictions.md`](safety/operating_restrictions.md) | 57 numbered restrictions |
| [`safety/security.md`](safety/security.md) | Security model and credential rules |

## 8. Operations

| Document | Purpose |
|---|---|
| [`operations/delivery_workflow.md`](operations/delivery_workflow.md) | End-to-end delivery journey |
| [`operations/congestion_aware_routing.md`](operations/congestion_aware_routing.md) | Corridor states and routing rules |
| [`operations/environmental_hazards.md`](operations/environmental_hazards.md) | Wet floor, lighting, weather |
| [`operations/blocked_corridors.md`](operations/blocked_corridors.md) | Closures, construction, fire routes |
| [`operations/battery_recovery.md`](operations/battery_recovery.md) | Reserve rule, checkpoint selection |
| [`operations/lunch_shutdown.md`](operations/lunch_shutdown.md) | High-traffic shutdown workflow |
| [`operations/human_intervention.md`](operations/human_intervention.md) | 🔴 **The human is the safety system** |

## 9. Testing

| Document | Purpose |
|---|---|
| [`testing/test_strategy.md`](testing/test_strategy.md) | Five-tier test plan |
| [`testing/test_inventory.md`](testing/test_inventory.md) | 🔴 **What tests exist and which have been run: none** |
| [`testing/known_failures.md`](testing/known_failures.md) | ⚠️ **Known defects, contradictions, unproven behaviour** |
| [`testing/acceptance_criteria.md`](testing/acceptance_criteria.md) | 52 criteria, 0 met |
| [`testing/future_pilot_plan.md`](testing/future_pilot_plan.md) | Phased plan to first safe operation |

## 10. Handoff — read these for the exhibition proposal

| Document | Purpose |
|---|---|
| [`handoff/agent_handoff.md`](handoff/agent_handoff.md) | 🔴 **Entry point for the third agent** |
| [`handoff/implementation_inventory.md`](handoff/implementation_inventory.md) | 🔴 **Every component and its real status** |
| [`handoff/unresolved_dependencies.md`](handoff/unresolved_dependencies.md) | 🔴 **What must be answered before progress** |
| [`handoff/exhibition_evidence_inventory.md`](handoff/exhibition_evidence_inventory.md) | 🔴 **What evidence exists — and what must NOT be claimed** |

## 11. Administration

| Document | Purpose |
|---|---|
| [`administration/teacher_introduction.md`](administration/teacher_introduction.md) | For teachers and administration |
| [`administration/permission_checklist.md`](administration/permission_checklist.md) | 102 permissions, **0 granted** |
| [`administration/technical_assessment_request.md`](administration/technical_assessment_request.md) | Prepared questions for IT, CCTV, lift, robot owner |
| [`administration/exhibition_plan.md`](administration/exhibition_plan.md) | Proposed demonstration plan |

## 12. Platform selection (prior phase)

[`platform_selection/`](platform_selection/) — the study that chose the Go2 EDU
over the G1 humanoid. Retained because several findings are load-bearing.

---

## Related work in this repository

| Path | Owner | Status |
|---|---|---|
| `docs/shana/` | **SHANA agent** | This documentation set |
| `docs/cloudlight/` | **CloudLight agent** | 🔴 **Not owned by SHANA. Not modified.** Empty at time of writing. |
| `../../questionnaires/` | SHANA | 434-question feasibility questionnaire + Gemini Sheets prompt |
| `../../data/` | SHANA | Proposed schemas and blank templates |

---

## The five facts that decide everything

1. 🔴 **Only EDU-and-above Go2 models may be programmed.** Verbatim: *"Only EDU or
   above versions can be developed, other versions such as PRO and AIR versions do
   not open the secondary development interface."*
2. 🔴 **The robot has no altitude, pressure, or barometer sensor.** It **cannot
   determine which floor it is on.**
3. 🔴 **The stair gait setter was removed** from the current SDK (firmware
   ≥1.1.6). Stairs are excluded.
4. 🔴 **No sanctioned lift integration exists** for the installed Johnson lifts,
   and modification is prohibited by law.
5. 🔴 **There is no autonomous return-to-dock.** Recovery is human-assisted by
   design.

---

## The honest position

> SHANA is a genuinely useful idea on hardware that is well suited to **delivery**
> and poorly suited to **manipulation**. It carries items on a tray. It has no arm.
> It operates among children, which sets a very high bar.
>
> **434 questions asked. 0 answered. 102 permissions sought. 0 granted. 11 gates
> defined. 0 passed. 0 tests run.**
>
> That is the accurate state of the project. The next step is not building — it is
> answering questions that only the school, its staff, and one lift manufacturer
> can answer.