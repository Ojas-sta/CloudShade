# CloudLight — Documentation Index

**Phase 0 · Research and design only · 2026-10-11**

> **🔴 NOTHING HERE IS BUILT.** There is no CloudLight source code, no test, and
> no deployment. Every capability is a proposal. Start with
> [`handoff/implementation_inventory.md`](handoff/implementation_inventory.md).

---

## ⚠️ Read this first

| If you are… | Read, in order |
|---|---|
| 🔴 **The next agent** | [`handoff/agent_handoff.md`](handoff/agent_handoff.md) → [`handoff/exhibition_evidence_inventory.md`](handoff/exhibition_evidence_inventory.md) |
| **Anyone new** | [`overview.md`](overview.md) → [`architecture.md`](architecture.md) |
| **Writing an exhibition proposal** | 🔴 [`handoff/exhibition_evidence_inventory.md`](handoff/exhibition_evidence_inventory.md) — it lists 12 claims that must **never** be made |
| **Deciding feasibility** | [`feasibility_report.md`](feasibility_report.md) → [`open_questions_register.md`](open_questions_register.md) |
| **The Principal** | [`requirements.md`](requirements.md) → [`decision_log.md`](decision_log.md) |
| **School IT** | [`network_and_hosting.md`](network_and_hosting.md) → [`windows_startup.md`](windows_startup.md) |
| **Timetable staff** | [`timetable_and_substitutions.md`](timetable_and_substitutions.md) → [`classroom_dashboard.md`](classroom_dashboard.md) |
| **Teachers** | [`classroom_dashboard.md`](classroom_dashboard.md) → [`resource_search.md`](resource_search.md) |

---

## 📁 Structure, and why

⚠️ **This directory uses a hybrid layout.** 22 documents sit at the top level and
39 more sit in nine subfolders. This is deliberate:

- The **flat files** were written first, as a single body of analysis. They are
  complete and interdependent.
- The **subfolders** were added later to house artefacts that do not fit that
  shape — research registers, test plans, handoff material, and additive
  feasibility/architecture records.

A single flat structure would mean moving 22 substantial documents and rewriting
every cross-reference, for no analytical gain. The mapping below makes the
organisation discoverable instead.

---

## Document map

### Overview and product

| Document | Contents |
|---|---|
| [`overview.md`](overview.md) | **Start here.** What CloudLight is; the 16 proposed capabilities and their real status; the relationship to SHANA; the three findings that shape everything |
| [`overview/current_project_state.md`](overview/current_project_state.md) | The factual state: what exists, what does not, what is running (nothing), what is verified (nothing on school hardware) |
| [`overview/product_vision.md`](overview/product_vision.md) | Why it exists, who it is for, what success and failure look like |
| [`overview/goals_and_non_goals.md`](overview/goals_and_non_goals.md) | What it will and will not do, each with a reason |
| [`overview/glossary.md`](overview/glossary.md) | Plain-English terms |
| [`requirements.md`](requirements.md) | Requirements classed as CONFIRMED / PROPOSED / PENDING ANSWER / OUT OF SCOPE |
| [`first_release_scope.md`](first_release_scope.md) | The smallest useful first release, and what is postponed |

### Architecture

| Document | Contents |
|---|---|
| [`architecture.md`](architecture.md) | Modules, data flow, failure behaviour, and the modular-monolith argument |
| [`architecture/system_context.md`](architecture/system_context.md) | C4-style context view and trust boundaries |
| [`architecture/data_model.md`](architecture/data_model.md) | Proposed entities — provisional, because the real period structure is UNKNOWN |
| [`architecture/api_contracts.md`](architecture/api_contracts.md) | 🔴 Proposed API, **plus the material finding that CloudLight's and SHANA's interface proposals diverge** |
| [`deployment_options.md`](deployment_options.md) | Hosting and client options compared; baseline deployment; progression |

> The proposed structure also listed `component_architecture.md`, `network_architecture.md` and `data_flow.md` as separate files. Those subjects are covered inside [`architecture.md`](architecture.md) §3, §6 and §4 respectively. Splitting them would duplicate content without adding anything.

### Classroom

| Proposed path | Actual location |
|---|---|
| `classroom/dashboard.md` | [`classroom_dashboard.md`](classroom_dashboard.md) — display content hierarchy, countdowns, the scheduled-vs-confirmed contract, degraded states |
| `classroom/timetable.md` + `substitutions.md` | [`timetable_and_substitutions.md`](timetable_and_substitutions.md) — one pipeline document, because the two are inseparable |
| `classroom/smartboard_startup.md` | [`windows_startup.md`](windows_startup.md) |
| `classroom/touchscreen_usability.md` | [`classroom_dashboard.md`](classroom_dashboard.md) §10 and [`testing/acceptance_criteria.md`](testing/acceptance_criteria.md) |
| `classroom/schedule_synchronisation.md` | Not written. **Deliberately deferred** — it presupposes a school scheduling system that has not been identified |

### Resources and search

| Proposed path | Actual location |
|---|---|
| `resources/nas_integration.md` | [`resource_browser.md`](resource_browser.md) §1–§3 |
| `resources/folder_browser.md` | [`resource_browser.md`](resource_browser.md) §5–§8 |
| `resources/resource_indexing.md` | [`resource_search.md`](resource_search.md) §1–§3 |
| `resources/search.md` | [`resource_search.md`](resource_search.md) |
| `resources/permissions.md` | [`resource_search.md`](resource_search.md) §3 — the permission-aware retrieval model, and [`security_and_privacy.md`](security_and_privacy.md) §7 |

### AI

| Proposed path | Actual location |
|---|---|
| `ai/assistant_architecture.md` | [`ai_assistant.md`](ai_assistant.md) §1, §5, §10 |
| `ai/retrieval_and_citations.md` | [`ai_assistant.md`](ai_assistant.md) §3, §4, §7 |
| `ai/document_ingestion.md` | [`timetable_and_substitutions.md`](timetable_and_substitutions.md) — ingestion is the timetable pipeline |
| `ai/model_hosting.md` | [`ai_assistant.md`](ai_assistant.md) §5–§6 |
| `ai/limitations.md` | [`ai_assistant.md`](ai_assistant.md) §15 and [`risk_register.md`](risk_register.md) |

### Security

| Proposed path | Actual location |
|---|---|
| `security/authentication.md` + `authorisation.md` + `session_management.md` | [`authentication_and_authorisation.md`](authentication_and_authorisation.md) §2–§3, §5–§9, §11 |
| `security/threat_model.md` | [`security_and_privacy.md`](security_and_privacy.md) §3 |
| `security/privacy_and_data_retention.md` | [`security_and_privacy.md`](security_and_privacy.md) §4, §8 |
| `security/secrets_management.md` | [`security_and_privacy.md`](security_and_privacy.md) §5 |
| `security/update_security.md` | [`ota_updates.md`](ota_updates.md) §2–§4, §9, §14 |

### Integrations

| Document | Contents |
|---|---|
| [`shana_integration.md`](shana_integration.md) | CloudLight's side of the robot boundary |
| [`integrations/identity_provider.md`](integrations/identity_provider.md) | Three identity models; none chosen, none contacted |
| [`integrations/timetable_sources.md`](integrations/timetable_sources.md) | Candidate input sources; **no school timetable has been seen** |

### Operations

| Proposed path | Actual location |
|---|---|
| `operations/hosting.md` | [`network_and_hosting.md`](network_and_hosting.md) §3–§4 |
| `operations/windows_startup.md` | [`windows_startup.md`](windows_startup.md) |
| `operations/ota_updates.md` | [`ota_updates.md`](ota_updates.md) |
| `operations/backup_and_recovery.md` | [`network_and_hosting.md`](network_and_hosting.md) §9 and [`requirements.md`](requirements.md) §3 |
| `operations/monitoring.md` | [`architecture.md`](architecture.md) §5 |
| `operations/incident_response.md` | [`security_and_privacy.md`](security_and_privacy.md) §10 |

### Feasibility

| Document | Contents |
|---|---|
| [`feasibility_report.md`](feasibility_report.md) | **The primary answer document.** 24 questions answered, every conclusion classified |
| [`feasibility/assumptions.md`](feasibility/assumptions.md) | Every load-bearing assumption, what breaks if it is false, and how to test it cheaply |
| [`feasibility/constraints.md`](feasibility/constraints.md) | Binding constraints and what each forbids |
| [`feasibility/go_no_go_gates.md`](feasibility/go_no_go_gates.md) | Gates that must pass before each phase. **None has been passed** |
| [`open_questions_register.md`](open_questions_register.md) | Blocking questions, sequenced by who can answer |
| [`decision_log.md`](decision_log.md) | Decisions, provisional defaults, rejected options, reversal conditions |
| [`risk_register.md`](risk_register.md) | Risks by domain with rating, mitigation and residual risk |
| [`next_actions.md`](next_actions.md) | The next ten actions, cheapest-first |
| [`sources.md`](sources.md) | Bibliography, sources deliberately not used, unsourced judgement |

### Testing

| Document | Contents |
|---|---|
| [`testing/test_strategy.md`](testing/test_strategy.md) | Plan for tests that do not yet exist |
| [`testing/test_inventory.md`](testing/test_inventory.md) | **92 planned tests, every one `NOT WRITTEN` / `NOT RUN`** |
| [`testing/acceptance_criteria.md`](testing/acceptance_criteria.md) | First-release and exhibition acceptance criteria |
| [`testing/known_failures.md`](testing/known_failures.md) | 🔴 Failures of this research process itself, recorded openly |

### Research

| Document | Contents |
|---|---|
| [`research/research_index.md`](research/research_index.md) | Index and how to extend the register |
| [`research/source_register.md`](research/source_register.md) | 36 research threads, each with 17 recorded fields |
| [`research/research_log.md`](research/research_log.md) | Chronological record of research performed |
| [`research/claim_evidence_matrix.md`](research/claim_evidence_matrix.md) | 🔴 62 claims, each with an exhibition verdict |

### Handoff

| Document | Contents |
|---|---|
| [`handoff/agent_handoff.md`](handoff/agent_handoff.md) | 🔴 **Start here for the next agent** |
| [`handoff/implementation_inventory.md`](handoff/implementation_inventory.md) | 🔴 Component-by-component status; what does not exist |
| [`handoff/exhibition_evidence_inventory.md`](handoff/exhibition_evidence_inventory.md) | 🔴 **12 claims that must never be made**; what may be shown |
| [`handoff/unresolved_dependencies.md`](handoff/unresolved_dependencies.md) | 43 dependencies and who owns each |

---

## The questionnaire

| File | Contents |
|---|---|
| [`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md) | 🔴 **576 questions**, 26 categories, **0 answered** |
| [`../../questionnaires/GEMINI_CLOUDLIGHT_SHEETS_PROMPT.md`](../../questionnaires/GEMINI_CLOUDLIGHT_SHEETS_PROMPT.md) | Conversion prompt — CloudLight file only |

The SHANA questionnaire (434 questions) is separate and unchanged.

---

## Ground rules this project follows

1. No capability is described as operational before it is demonstrated.
2. No AI may publish timetable data, approve substitutions, grant permissions, or
   issue robot commands.
3. Deterministic parsing precedes AI; AI never invents a missing value.
4. The display never shows an authentication secret, and never shows an
   unapproved substitution as a confirmed teacher.
5. Stale data is shown only with a visible marker; a blank screen beats a
   confident wrong answer.
6. Search never returns a file the user cannot open.
7. No external AI service receives school data without written authorisation.
8. A school LAN is not trusted.
9. CloudLight never controls robot motion; SHANA's safety authority is never delegated.
10. A gate is never marked verified without evidence.

---

## Known documentation issues

| Issue | Status |
|---|---|
| 9 links point at SHANA docs under `docs/architecture/`, `docs/safety/`, `docs/infrastructure/` | 🟡 **Temporarily broken** pending the other agent's revert of `docs/shana/` → `docs/`. Correct paths for the pre-revert layout; will resolve when the revert lands |
| ADR-013 and ADR-014 are not listed in the shared ADR index | 🟡 The index is mid-revert; left to its owner |
| The repository has **0 commits** | 🔴 Nothing is version-controlled. No history, no rollback |
| Flat vs subfolder layout | 🟡 Deliberate; see §Structure above |