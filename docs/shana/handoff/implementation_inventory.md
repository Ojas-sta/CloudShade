# Implementation Inventory

🔴 **The authoritative list of what exists, what does not, and what is needed.**

Every proposed component of SHANA, with a truthful status. If a component is
not on this list, it was not assessed and should not be assumed.

---

## Status vocabulary — use these labels exactly

| Label | Meaning |
|---|---|
| `IMPLEMENTED_AND_TESTED` | Code exists and tests pass with recorded output |
| `IMPLEMENTED_NOT_TESTED` | Code exists, never executed |
| `PARTIALLY_IMPLEMENTED` | Some components exist, critical path does not |
| `DESIGN_ONLY` | Fully specified, no code written |
| `RESEARCH_ONLY` | Investigated, no design committed |
| `BLOCKED` | Cannot proceed until a named dependency clears |
| `UNKNOWN` | Insufficient information; **do not assume** |
| `DEPRECATED` | Superseded; retained for history only |

⚠️ **No component in this project holds `IMPLEMENTED_AND_TESTED`.**
⚠️ **No component holds `IMPLEMENTED_NOT_TESTED`.**

---

## Summary

| Status | Count |
|---|---|
| `IMPLEMENTED_AND_TESTED` | **0** |
| `IMPLEMENTED_NOT_TESTED` | **0** |
| `PARTIALLY_IMPLEMENTED` | **0** |
| `DESIGN_ONLY` | 14 |
| `RESEARCH_ONLY` | 6 |
| `BLOCKED` | 5 |
| `UNKNOWN` | 8 |
| `DEPRECATED` | 3 |

---

## 1. Task and delivery pipeline

| # | Component | Status | Notes |
|---|---|---|---|
| 1.1 | Task intake / request parsing | `DESIGN_ONLY` | Input shape specified in `data/schemas/` |
| 1.2 | Delivery task state machine | `DESIGN_ONLY` | States defined in [`architecture/task_state_machine.md`](../architecture/task_state_machine.md) |
| 1.3 | Destination validation | `DESIGN_ONLY` | Depends on route graph existing |
| 1.4 | Carry / tray verification | `DESIGN_ONLY` | **No arm — carry-only by design** |
| 1.5 | Handover confirmation | `BLOCKED` | Needs human confirmation interface |
| 1.6 | Retry and abandonment policy | `DESIGN_ONLY` | — |

⚠️ **SHANA cannot pick up objects, open doors, press buttons, or place items on a
shelf.** It carries items on a tray. Any claim to the contrary is false.

## 2. Navigation and localisation

| # | Component | Status | Blocking dependency |
|---|---|---|---|
| 2.1 | Mapping | `BLOCKED` | 🔴 **Floor survey never conducted** |
| 2.2 | Floor identity / localisation | `UNKNOWN` | 🔴 **No altitude, pressure, or barometer sensor on the robot.** It cannot tell which floor it is on |
| 2.3 | Obstacle avoidance | `DESIGN_ONLY` | — |
| 2.4 | Route graph construction | `BLOCKED` | Depends on 2.1 |
| 2.5 | Self-built navigation stack | `DESIGN_ONLY` | Decision `D-009`; vendor SLAM not used |
| 2.6 | ArUco floor markers | `RESEARCH_ONLY` | Marker placement needs survey |
| 2.7 | Localisation fallback | `UNKNOWN` | Undefined |

## 3. Safety and motion control

| # | Component | Status | Blocking dependency |
|---|---|---|---|
| 3.1 | Obstacle stop | `DESIGN_ONLY` | 🔴 **Safety case has zero evidence** |
| 3.2 | Emergency stop | `DESIGN_ONLY` | School protocol required |
| 3.3 | Speed limiting in occupied space | `DESIGN_ONLY` | — |
| 3.4 | Safe-return-on-low-battery | `DESIGN_ONLY` | No autonomous dock exists — human-assisted |
| 3.5 | Stair prevention | `DESIGN_ONLY` | 🔴 **Stair gait setter removed from SDK (≥1.1.6)** |
| 3.6 | Human intervention protocol | `DESIGN_ONLY` | — |

## 4. Infrastructure integration

| # | Component | Status | Blocking dependency |
|---|---|---|---|
| 4.1 | Robot network interface | `RESEARCH_ONLY` | Wired Ethernet required; ADR-004 |
| 4.2 | School network integration | `BLOCKED` | 102 permissions, **0 granted** |
| 4.3 | CCTV occupancy feed | `DEPRECATED` | 🔴 **Recommended against on privacy grounds** |
| 4.4 | Elevator integration | `BLOCKED` | 🔴 **No sanctioned lift API exists; modification unlawful** |
| 4.5 | Radio / LoRa telemetry | `RESEARCH_ONLY` | Deferred by ADR-012; India RF regulation |
| 4.6 | Onboard ESP32 sensing | `RESEARCH_ONLY` | Not required for pilot |

## 5. CloudLight interface

| # | Component | Status | Notes |
|---|---|---|---|
| 5.1 | Interface definition | `DESIGN_ONLY` | [`../architecture/cloudlight_integration.md`](../architecture/cloudlight_integration.md) |
| 5.2 | CloudLight implementation | `UNKNOWN` | 🔴 **CloudLight is owned by a different agent. SHANA has no visibility into it** |
| 5.3 | Shared data schema | `DESIGN_ONLY` | `data/schemas/` — never validated against a real partner |
| 5.4 | Integration testing | `BLOCKED` | Cannot test against a partner that does not exist yet |

## 6. Data, schemas, and tooling

| # | Component | Status | Notes |
|---|---|---|---|
| 6.1 | YAML schemas | `DESIGN_ONLY` | 4 files in `data/schemas/`; **never parsed by any code** |
| 6.2 | Blank data templates | `DESIGN_ONLY` | `data/` — empty by design |
| 6.3 | Simulation environment | `DEPRECATED` | ❌ **Never created.** ADR-011 mandates simulation first — not done |
| 6.4 | Build / dependency tooling | `DEPRECATED` | ❌ Never created |
| 6.5 | CI pipeline | `DEPRECATED` | ❌ Never created |

## 7. Research artefacts (documents, not software)

| # | Artefact | Status |
|---|---|---|
| 7.1 | Source register | `IMPLEMENTED_AND_TESTED` — in the only sense that applies: it exists and was written |
| 7.2 | Research log | `IMPLEMENTED_AND_TESTED` — append-only, exists |
| 7.3 | Claim–evidence matrix | ✅ exists |
| 7.4 | Risk register | ✅ exists, **0 risks mitigated** |
| 7.5 | Safety case | `DESIGN_ONLY` — draft, no evidence |
| 7.6 | Go/No-Go gates | ✅ exists, **0 of 11 passed** |
| 7.7 | Permission checklist | ✅ exists, **0 of 102 granted** |
| 7.8 | Feasibility questionnaire | ✅ exists, **0 of 434 answered** |
| 7.9 | Exhibition evidence inventory | ✅ exists |
| 7.10 | Image assets | `DEPRECATED` — ⚠️ 8 unrelated display-panel photographs, unusable |

---

## The gaps that matter, ranked

| Rank | Gap | Why it blocks everything else |
|---|---|---|
| 1 | 🔴 **No answers to the 434-question questionnaire** | Almost every design decision is provisional on these answers |
| 2 | 🔴 **No permissions granted** | The robot cannot be touched, the network cannot be joined, nothing can be tested |
| 3 | 🔴 **No route survey** | No map, no route graph, no navigation design |
| 4 | 🔴 **No simulation, despite ADR-011 requiring it** | The first permitted software work has not begun |
| 5 | 🔴 **No Go/No-Go gate passed** | By the project's own rules, implementation is not yet permitted |
| 6 | 🔴 **Robot identity unconfirmed** | Model, firmware, and SDK licence status unknown — gates 1–2 |
| 7 | ⚠️ **10 ADRs owed** | See [`../architecture/architecture_decisions/README.md`](../architecture/architecture_decisions/README.md) |

---

## Honest first software task

If and only if a gate is cleared and permissions are granted, the first useful
software deliverable is:

> A **simulation-only** proof of concept: a virtual Go2 in simulation, navigating
> a surveyed single-floor map, with obstacle avoidance and a human-in-the-loop
> stop, emitting `data/schemas/`-conformant messages.

**No hardware. No network. No student data. No physical operation.** That is what
ADR-011 already requires, and it has not been started.

See [`../testing/future_pilot_plan.md`](../testing/future_pilot_plan.md).