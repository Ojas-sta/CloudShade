# Unresolved Dependencies

🔴 **What SHANA is actually waiting for. Read this before starting any work.**

Most SHANA tasks are not blocked by engineering difficulty. They are blocked by
**facts that only other people and organisations can supply.**

---

## Tier 1 — Blocking everything

Nothing meaningful can proceed until these clear.

| # | Dependency | Why it blocks | Who answers | Status |
|---|---|---|---|---|
| 1 | 🔴 **Go2 model number, firmware version, SDK licence eligibility** | If it is not EDU, **no secondary development is possible** and the project is dead on arrival | School / robot owner | `UNKNOWN` |
| 2 | 🔴 **Written answers to the 434-question questionnaire** | Almost every design decision is provisional on these | School administration | **0 of 434 answered** |
| 3 | 🔴 **Permissions** | Without any of them nothing can be tested | School administration + IT | **0 of 102 granted** |
| 4 | 🔴 **Go/No-Go gate decisions** | The project's own rules forbid implementation until gates pass | Project owner | **0 of 11 passed** |
| 5 | 🔴 **Route survey access** | No map, no route graph, no navigation design | School administration | Not started |

---

## Tier 2 — Blocking specific subsystems

| # | Dependency | Blocks | Who answers |
|---|---|---|---|
| 6 | 🔴 **Lift manufacturer response** | Any multi-floor scope | Johnson, via school |
| 7 | 🔴 **Privacy assessment** | Any camera or CCTV use | School / legal |
| 8 | **Network topology and robot subnet policy** | Robot connectivity | School IT |
| 9 | **Corridor width, door widths, ramp gradients** | Whether the robot physically fits | Survey |
| 10 | **Battery behaviour under school duty cycle** | Operational feasibility | Bench test |
| 11 | **Charging location and power access** | Recovery workflow | School facilities |
| 12 | **Destination authority** | Who may order a delivery, and who may open a door | School administration |

---

## Tier 3 — Needed before exhibition

| # | Dependency | Blocks | Who answers |
|---|---|---|---|
| 13 | 🔴 **Is a live robot demonstration mandatory?** | 🔴 **The entire exhibit approach** | Exhibition organiser |
| 14 | **May a diagram-only proposal be exhibited?** | Non-demo exhibit | Exhibition organiser |
| 15 | **Is documentary evidence acceptable?** | Whole exhibit approach | Exhibition organiser |

⚠️ **Dependency 13 is decisive and should be resolved first.** If a live
demonstration is mandatory, the honest answer today is that SHANA cannot meet it.
That should be stated plainly rather than substituted with a mock-up or a
simulation presented as real.

---

## Technical unknowns — `UNKNOWN`, not blocked

These are unknown rather than blocked. They need investigation, not permission.

| # | Unknown | How to resolve |
|---|---|---|
| 16 | Exact floor area and corridor topology | Survey |
| 17 | Whether destinations are reachable on one floor | Survey |
| 18 | Locally-sourced rule under Indian Information Technology Act | Legal review — ⚠️ **not** a substitute for counsel |
| 19 | Vendor SLAM capability on EDU firmware | Documentation review |
| 20 | Whether ArUco floor identity is viable at school scale | Simulation test |
| 21 | Realistic simultaneous-request load | Estimate |
| 22 | Whether cloud scheduling adds value for a single school | Design judgement |

---

## Internal work owed — no external dependency

🔴 **These are SHANA's own outstanding debts and can be done right now.**

| # | Owed work | Where tracked |
|---|---|---|
| 23 | ⚠️ **Write 10 owed ADRs** — real decisions already recorded in the decision log | [`../architecture/architecture_decisions/README.md`](../architecture/architecture_decisions/README.md) |
| 24 | 🔴 **Build the simulation proof of concept** that ADR-011 already mandates | [`../testing/future_pilot_plan.md`](../testing/future_pilot_plan.md) |
| 25 | **Write the route survey protocol** so it is ready to execute | [`../infrastructure/school_route_survey.md`](../infrastructure/school_route_survey.md) |
| 26 | **Finalise the technical assessment request** for IT, CCTV, and lift contacts | [`../administration/technical_assessment_request.md`](../administration/technical_assessment_request.md) |
| 27 | **Verify the schema YAML parses** — it has never been checked | [`../../../data/schemas/`](../../../data/schemas/) |
| 28 | 🔴 **Rotate the exposed credential** and gitignore `start_opencode_server.sh` | [`../overview/current_project_state.md`](../overview/current_project_state.md) |

⚠️ **Item 24 is the highest-value unblocked task.** It is the only route to
producing a working artefact under current rules, because simulation requires no
robot, no network, no school, and no student data.

⚠️ **Item 28 is urgent and belongs to the repository owner**, not SHANA. The
password in that script must be treated as compromised.

---

## Summary

| Tier | Count | Resolvable by SHANA alone? |
|---|---|---|
| Tier 1 — blocking everything | 5 | ❌ **No** |
| Tier 2 — blocking subsystems | 7 | ❌ **No** |
| Tier 3 — exhibition | 3 | ❌ **No** |
| Technical unknowns | 7 | ⚠️ Partly |
| Internal debts | 6 | ✅ **Yes — 6 of these can start today** |

> ## The uncomfortable conclusion
>
> **20 of 22 external dependencies can only be answered by other people.**
>
> A future agent reading this should understand that the most valuable thing they
> can do is **not** more engineering — it is converting the six internal debts
> above into finished work, and helping the owner chase the external answers.