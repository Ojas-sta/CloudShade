# Research Index

**The central index for every research activity in SHANA.**

Start here to find what was researched, where it is recorded, and what is still
missing.

---

## 1. Research artefacts

| Document | Purpose | Status |
|---|---|---|
| This file | Central index | ✅ Exists |
| [`research_log.md`](research_log.md) | ⚠️ **Append-only chronological record** | ✅ Exists |
| [`source_register.md`](source_register.md) | Every source, with class and limitations | ✅ Exists |
| [`claim_evidence_matrix.md`](claim_evidence_matrix.md) | 🔴 Claim → evidence, for exhibition statements | ✅ Exists |
| [`cctv_source_register.md`](cctv_source_register.md) | CCTV / ONVIF / occupancy sources | ✅ Exists |
| [`appendix_india_rf_research.md`](appendix_india_rf_research.md) | India RF regulation and link budgets | ✅ Exists |

---

## 2. Research by area

### 2.1 Platform and hardware

| Question | Where recorded | Evidence class | Answered? |
|---|---|---|---|
| Go1 vs Go2 vs G1 selection | [`../platform_selection/`](../platform_selection/) | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | ✅ **Yes — decision made** |
| Secondary development interface eligibility | [`../hardware/sdk_and_interfaces.md`](../hardware/sdk_and_interfaces.md) | `VERIFIED` — vendor wording quoted | ✅ **Yes** |
| Robot sensor suite | [`../hardware/go2_edu_inventory.md`](../hardware/go2_edu_inventory.md) | `VERIFIED` | ✅ **Yes** |
| Stair support in current SDK | [`../platform_selection/`](../platform_selection/) | `VERIFIED` — removed ≥1.1.6 | ✅ **Yes** |
| Battery behaviour under duty cycle | [`../hardware/battery_and_charging.md`](../hardware/battery_and_charging.md) | `PLAUSIBLE_REQUIRES_TESTING` | ❌ **No** |
| Depth sensor capability in corridors | [`../hardware/cameras_and_depth_sensors.md`](../hardware/cameras_and_depth_sensors.md) | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | ⚠️ Partly |
| Radio telemetry feasibility | [`../hardware/esp32_and_lora.md`](../hardware/esp32_and_lora.md) | `RESEARCH_ONLY` — deferred | ✅ Deferred |

### 2.2 Infrastructure

| Question | Where recorded | Evidence class | Answered? |
|---|---|---|---|
| School network requirements | [`../infrastructure/school_network.md`](../infrastructure/school_network.md) | `PLAUSIBLE_REQUIRES_TESTING` | ⚠️ Partly — needs IT |
| Lift API availability | [`../infrastructure/elevator_feasibility.md`](../infrastructure/elevator_feasibility.md) | `NOT_PERMITTED` | ✅ **Yes — no sanctioned API exists** |
| Stairs and ramps | [`../infrastructure/stairs_and_ramps.md`](../infrastructure/stairs_and_ramps.md) | `UNKNOWN` | 🔴 **No — never surveyed** |
| CCTV occupancy feasibility | [`../infrastructure/cctv_feasibility.md`](../infrastructure/cctv_feasibility.md) | `DEPRECATED` | ✅ **Yes — recommended against** |
| Corridor topology | [`../infrastructure/school_route_survey.md`](../infrastructure/school_route_survey.md) | — | 🔴 **No — survey never conducted** |

### 2.3 Safety, privacy, and law

| Question | Where recorded | Evidence class | Answered? |
|---|---|---|---|
| Principal hazards | [`../safety/hazard_analysis.md`](../safety/hazard_analysis.md) | `PLAUSIBLE_REQUIRES_TESTING` | ⚠️ Identified, not verified |
| Risk register | [`../safety/risk_register.md`](../safety/risk_register.md) | — | ⚠️ 21 risks, **0 mitigated** |
| Student privacy position | [`../safety/student_privacy.md`](../safety/student_privacy.md) | Design judgement | ✅ Position taken |
| Surveillance law applicability | [`appendix_india_rf_research.md`](appendix_india_rf_research.md) + privacy doc | `UNKNOWN` | ❌ **Needs legal counsel** |
| Safety case evidence | [`../safety/safety_case.md`](../safety/safety_case.md) | — | 🔴 **None gathered** |
| Credential handling rules | [`../safety/security.md`](../safety/security.md) | `VERIFIED` practice | ✅ **One live violation found** |

### 2.4 Operations

| Question | Where recorded | Answered? |
|---|---|---|
| Corridor occupancy states | [`../operations/congestion_aware_routing.md`](../operations/congestion_aware_routing.md) | ⚠️ Design only, human-declared |
| Environmental hazards | [`../operations/environmental_hazards.md`](../operations/environmental_hazards.md) | ⚠️ Design only |
| Blocked corridor handling | [`../operations/blocked_corridors.md`](../operations/blocked_corridors.md) | ⚠️ Design only |
| Battery recovery | [`../operations/battery_recovery.md`](../operations/battery_recovery.md) | ⚠️ Design only |
| Lunch shutdown | [`../operations/lunch_shutdown.md`](../operations/lunch_shutdown.md) | ⚠️ Design only |

---

## 3. 🔴 Research gaps

| # | Gap | Why it matters | Who can close it |
|---|---|---|---|
| 3.1 | 🔴 **Robot model, firmware, licence unconfirmed** | 🔴 Blocks everything | School / robot owner |
| 3.2 | 🔴 **Route survey never conducted** | No map, no route graph | School |
| 3.3 | 🔴 **Lift manufacturer never contacted** | Cross-floor scope | Johnson via school |
| 3.4 | 🔴 **Privacy assessment never obtained** | Any camera use | School / legal |
| 3.5 | **34 questions in the questionnaire carry unresearched specifics** | Unknowns remain | Multiple parties |
| 3.6 | **No hardware testing of any kind** | All vendor-documented claims unverified by us | School, once permitted |
| 3.7 | **No independent review of the research** | Single-author risk | A second reader |

⚠️ **Gap 3.7 is structural.** All research was authored by one agent without
external review. A second reader — especially a domain expert — should review the
hardware and safety conclusions before they are relied upon.

---

## 4. Adding new research

| Rule | Why |
|---|---|
| Add the source to `source_register.md` | One place to find every citation |
| Append a dated entry to `research_log.md` | Chronological continuity across agents |
| Use the evidence classes consistently | A reader must be able to trust the confidence level |
| Never upgrade a claim without a source | Prevents confidence from outrunning evidence |
| Record what you **could not** find | Negative findings are findings |

Evidence classes: `VERIFIED` · `DOCUMENTED_NOT_HARDWARE_VERIFIED` ·
`PLAUSIBLE_REQUIRES_TESTING` · `UNKNOWN` · `NOT_PERMITTED`