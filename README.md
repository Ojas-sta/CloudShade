# CloudLight by SHANA

> **A school-local digital platform — with SHANA, an autonomous delivery robot, as one subsystem within it.**

---

## The two systems in this repository

| System | What it is | Documentation |
|---|---|---|
| **CloudLight** | The **platform**. Classroom displays, timetable and substitution dashboard, resource browsing and search, AI assistant, teacher authentication, administration, and updates — running on the school's own network and its own hardware | `docs/cloudlight/` |
| **SHANA** — School Autonomous Notification & Assist | The **robotics subsystem**. Navigation, localisation, route safety, battery reserve, hazards, safe stopping, and physical delivery execution, on a school-owned Unitree Go2 EDU | `docs/shana/` |

**CloudLight is the platform. SHANA is one subsystem of it. Both are documented,
researched and versioned in this one repository.**

Two rules govern the boundary, and both are accepted decisions rather than
preferences:

1. **CloudLight may request a delivery. It may never command the robot.** There is
   no motor, path or actuator command path anywhere in CloudLight — see
   [`ADR-014`](docs/architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md).
2. **CloudLight must operate fully when SHANA is offline.** A robot failure must
   never blank a classroom screen.

**Read [`docs/cloudlight/overview.md`](docs/cloudlight/overview.md) first** if
you are new. SHANA's own overview is
[`docs/shana/overview/`](docs/shana/overview/).

---

## ⚠️ STATUS: FEASIBILITY AND REQUIREMENTS DISCOVERY

**Nothing in this repository is operational.**

No robot has been operated, connected to, or commanded. No network has been
probed or scanned. No lift has been approached. No camera or CCTV feed has been
accessed. No production code has been written. No hardware has been installed,
flashed, or configured.

This repository contains **research, analysis, and questions**. That is deliberate.
The project's architecture depends on answers that only the school, its IT
department, its facilities team, its robot custodian, and its lift service
provider can give — and on measurements that require supervised physical tests
which have not been authorised.

---

## 🔴 Three things to know before reading further

| # | Fact |
|---|---|
| 1 | 🔴 **There is no code and no test.** 0 implementation files, 0 tests, 0 builds, 0 CI, 0 commits, 0 remotes. See `docs/shana/overview/current_project_state.md` |
| 2 | 🔴 **There is no visual evidence of SHANA.** The 8 JPEG files at the repository root are photographs of **LCD/display panels** — unrelated to this project. They must never be presented as robot, school, or system evidence. See `docs/shana/handoff/exhibition_evidence_inventory.md` |
| 3 | 🔴 **`start_opencode_server.sh` contains a hardcoded plaintext password** and binds a server to `0.0.0.0`. It is now gitignored, but **the password must be rotated** by its owner. See `docs/shana/safety/security.md` |
| 4 | 🔴 **The CloudLight architecture rests on unverified school facts.** Windows editions, administrator rights, network segmentation, permission to run a persistent service, the existence of shared storage, an identity directory, and AI policy are **all unknown**. Each is a P0 question in `questionnaires/Additional_CloudLight_Questionnaire.md` |
| 5 | 🔴 **Nine of sixteen proposed CloudLight capabilities are not in the first release**, and four of those are deferred because **the thing they depend on has not been confirmed to exist**. See `docs/cloudlight/first_release_scope.md` |

**Counts:** 434 questions asked · **0 answered** · 102 permissions sought · **0
granted** · 11 gates defined · **0 passed** · 34 assumptions · 30 unverified ·
21 risks · 0 mitigated.

---

## What SHANA is proposed to be

An autonomous indoor delivery robot that carries **approved items only** between
**authorised school locations**, requested by teachers through a local dashboard,
and reports progress and delivery confirmation.

### What SHANA is proposed **not** to be

| Not | Why |
|---|---|
| Not a student surveillance system | No facial recognition. No identity recognition. No biometric identification. No persistent tracking of individuals. No automatic misconduct detection or reporting. |
| Not a stair-climbing robot | Stair capability is not established for this hardware in this building. See `docs/shana/feasibility/feasibility_report.md`. |
| Not an autonomous lift user | No sanctioned third-party lift interface has been identified for the installed lifts, and physical modification is prohibited by law. |
| Not a networked-by-default device | Core operation requires no internet. The server is never exposed to a public network. |
| Not a remote-controlled toy | It carries items, in a school, among children. It requires supervision at every stage. |

---

## The single most important thing to know

**Both the robot's capability and the school's permission are unverified.**

- Unitree documents secondary development for **EDU-and-above models only**.
- Unitree's Go2 documentation states the robot supports **wired** development
  connections only.
- **The Go2 EDU has no sensor that can determine which floor of a building it is
  on** — no altitude, no pressure, no barometer exists anywhere in its message
  set.
- **The documented maximum step height is 160 mm**, and the manufacturer's manual
  forbids operation on complex terrain.
- **No autonomous return-to-dock capability exists** in the documented service list.

Until those are resolved by evidence and by written permission, every capability
described in this repository is **proposed, not proven**.

---

## Repository map

```
SHANA/
├── README.md                     ← you are here
├── CONTRIBUTING.md · SECURITY.md · CODE_OF_CONDUCT.md · LICENSE_REVIEW.md
├── docs/
│   ├── shana/                    ← SHANA documentation (start at docs/shana/README.md)
│   │   ├── overview/             what SHANA is, goals, context, glossary,
│   │   │                         🔴 current_project_state.md
│   │   ├── architecture/         system design, components, navigation,
│   │   │                         communication, data flow, task state machine,
│   │   │                         route graph, 🔴 cloudlight_integration.md,
│   │   │                         architecture_decisions/
│   │   ├── feasibility/          feasibility report, 🔴 constraints.md,
│   │   │                         questionnaire, risk register, open questions,
│   │   │                         assumptions, verification plan, go/no-go gates,
│   │   │                         decision log
│   │   ├── hardware/             🔴 go2_edu_inventory.md, sdk_and_interfaces.md,
│   │   │                         network_interfaces.md, cameras, battery,
│   │   │                         radio, markers
│   │   ├── infrastructure/       network, CCTV, elevator, stairs, route survey,
│   │   │                         mapping_and_localisation.md
│   │   ├── safety/               safety case, hazards, emergencies, privacy,
│   │   │                         restrictions, risk register, 🔴 security.md
│   │   ├── operations/           delivery workflow, congestion, environmental
│   │   │                         hazards, blocked corridors, lunch shutdown,
│   │   │                         battery recovery, human intervention
│   │   ├── research/             🔴 research_index.md, research_log.md,
│   │   │                         source_register.md, 🔴 claim_evidence_matrix.md,
│   │   │                         RF and CCTV source registers
│   │   ├── administration/       teacher introduction, permission checklist,
│   │   │                         technical assessment request, exhibition plan
│   │   ├── testing/              test strategy, 🔴 test_inventory.md,
│   │   │                         🔴 known_failures.md, acceptance criteria,
│   │   │                         pilot plan
│   │   ├── handoff/              🔴 agent_handoff.md, 🔴 implementation_inventory.md,
│   │   │                         🔴 unresolved_dependencies.md,
│   │   │                         🔴 exhibition_evidence_inventory.md
│   │   └── platform_selection/   G1 vs Go2 study
│   └── cloudlight/                 ← the platform (see docs/cloudlight/overview.md)
│       ├── overview.md             🔴 START HERE — what CloudLight is, scope
│       ├── architecture.md         modular monolith, modules, data flow, failure
│       ├── feasibility_report.md   🔴 conclusions classified by evidence strength
│       ├── requirements.md · deployment_options.md · first_release_scope.md
│       ├── next_actions.md         🔴 the next ten actions, in priority order
│       ├── classroom_dashboard.md · timetable_and_substitutions.md
│       ├── windows_startup.md · network_and_hosting.md
│       ├── resource_browser.md · resource_search.md · ai_assistant.md
│       ├── authentication_and_authorisation.md · ota_updates.md
│       ├── security_and_privacy.md · shana_integration.md · sources.md
│       └── risk_register.md · open_questions_register.md · decision_log.md
├── questionnaires/                 576-question CloudLight questionnaire
│                                  + 434-question SHANA questionnaire
│                                  + one Gemini conversion prompt for each
└── data/                         schemas and templates (documentation only)
```

---

## Start here

| If you are… | Read |
|---|---|
| 🔴 **Anyone new to this repository** | **`docs/cloudlight/overview.md` first** — it explains both systems and the relationship between them |
| 🔴 **Deciding whether this is feasible** | `docs/cloudlight/feasibility_report.md`, then `open_questions_register.md` |
| 🔴 **The Principal** | `docs/cloudlight/requirements.md`, then `docs/cloudlight/decision_log.md` |
| 🔴 **School IT / network** | `docs/cloudlight/network_and_hosting.md`, then `windows_startup.md`, then questionnaire category `CL-NET` |
| 🔴 **Timetable / administration** | `docs/cloudlight/timetable_and_substitutions.md`, then `classroom_dashboard.md` |
| 🔴 **Teachers** | `docs/cloudlight/classroom_dashboard.md`, then `resource_search.md` |
| 🔴 **Deciding what to do next** | `docs/cloudlight/next_actions.md` — ten actions, cheapest-first |
| A school administrator (SHANA) | `docs/shana/administration/teacher_introduction.md`, then `permission_checklist.md` |
| A teacher (SHANA) | `docs/shana/administration/teacher_introduction.md` |
| IT or network staff (SHANA) | `docs/shana/infrastructure/school_network.md`, then `questionnaires/SHANA_Feasibility_QA.md` category `SH-NET` |
| The robot custodian | `docs/shana/hardware/sdk_and_interfaces.md`, then category `SH-GO2` |
| Facilities / housekeeping | `docs/shana/infrastructure/school_route_survey.md`, `docs/shana/infrastructure/stairs_and_ramps.md` |
| The lift service provider | `docs/shana/administration/technical_assessment_request.md` |
| The CCTV owner | `docs/shana/infrastructure/cctv_feasibility.md` |
| 🔴 **The third agent / new contributor** | **`docs/shana/handoff/agent_handoff.md` — start here** |
| 🔴 Anyone checking claims | `docs/shana/research/claim_evidence_matrix.md` |
| 🔴 Exhibition / evaluation | `docs/shana/handoff/exhibition_evidence_inventory.md` |
| A new contributor | `CONTRIBUTING.md`, then `docs/cloudlight/decision_log.md` |
| Evaluating SHANA feasibility | `docs/shana/feasibility/feasibility_report.md`, then `go_no_go_gates.md` |

---

## Current state: what blocks everything

Ten questions block the architecture. All are P0. None can be answered by
research — each needs a measurement, a decision, or a signature.

| # | Question | Who answers |
|---|---|---|
| 1 | Is secondary development access authorised **in writing**? | School administration |
| 2 | What is the exact robot model, firmware, and installed options? | Robot custodian |
| 3 | Which battery is fitted — 8,000 mAh or 15,000 mAh? | Robot custodian |
| 4 | Is a dedicated robot network segment permitted? | IT |
| 5 | Is there a Linux host with a spare Ethernet port available? | AI Lab / IT |
| 6 | **What are the actual stair riser heights?** | Facilities |
| 7 | Are access points and wall-mounted markers permitted? | Administration + IT |
| 8 | **Is there a single-floor route to the rooms teachers actually need?** | Facilities + teachers |
| 9 | Does the lift manufacturer permit any integration, or robot use of EyeRIS? | Lift service provider |
| 10 | Who has authority to declare a go/no-go gate passed? | Administration |

⚠️ **Questions 6, 8 and 9 together determine whether vertical transit is in
scope at all.** If any riser exceeds 160 mm, stairs are permanently out. If
teachers do not need cross-floor delivery, the project becomes single-floor and
substantially more likely to succeed. **That is a good outcome, not a failure.**

### What blocks CloudLight

Five questions block the CloudLight architecture. All are P0. None can be
answered by research.

| # | Question | Who answers | If unanswered |
|---|---|---|---|
| 1 | **May a persistent local service run on school premises?** | Administration + IT | The local-first premise fails; the project cannot proceed |
| 2 | **Can classroom devices actually reach that service?** | IT | The project cannot proceed — and if client isolation is on, it fails *silently* |
| 3 | **Is client/AP isolation enabled on the school network?** | IT | The most likely quiet failure in the whole project |
| 4 | **Who approves a substitution, and who owns the master timetable?** | Timetable / administration | The core feature has no publication path |
| 5 | **What Windows edition is on the classroom boards?** | IT | The kiosk lockdown design may be impossible |

⚠️ **An access point cannot host this platform.** It is a Layer-2 bridge. "All
smartboards can reach the same access point" establishes at most a shared
network segment — it does not establish that a backend can exist, nor that one
device can reach another. See `docs/cloudlight/network_and_hosting.md`.

Full set: [`docs/cloudlight/open_questions_register.md`](docs/cloudlight/open_questions_register.md)
· 576 questions in `questionnaires/Additional_CloudLight_Questionnaire.md`.

---

## What can be done right now without any school answer

- **Run the Go2 simulator.** `unitree_mujoco` simulates the Go2 on the same DDS
  channels as the hardware, CPU-only, no GPU. The entire software stack can be
  built and demonstrated with no robot and no cost.
- **Build the link budget and site-survey plan** for any future radio work.
- **Draft architecture decisions** with their reversal conditions.
- **Design the floor-identity mechanism** and its failure handling.
- **Build the timetable and substitution data model** against synthetic fixtures.
  No school data is needed to model a period grid, a substitution record, or an
  approval trail.
- **Prototype deterministic timetable parsing** against a synthetic spreadsheet.
  This is the highest-value, lowest-risk piece of CloudLight engineering and
  depends on **no** unanswered school question.
- **Model the degraded states** — what a display shows with no network, no
  snapshot, and a quarantined document — with synthetic data.
- **Draft the timetable, substitution and approval workflow** for timetable staff
  to review, without building anything.

---

## Ground rules this project follows

1. **No capability is described as operational before it is demonstrated.**
2. **No AI model may control the robot.** A deterministic validator sits between
   every request and every motion command.
3. **Teachers never receive motor-control access.**
4. **Delivery completion requires explicit recipient confirmation** — never
   inferred from arrival.
5. **Loss of communications produces a defined safe behaviour**, never a
   best-effort continuation.
6. **A human spotter is required at every stage**, with unilateral stop authority.
7. **No lift modification.** Ever.
8. **No student data is collected, and no person is ever identified.**
9. **No credentials or private school network details are committed to Git.**
10. **A gate is never marked VERIFIED without evidence.**

### Additionally, for CloudLight

11. **CloudLight never controls robot motion** — the boundary is an *absence of
    capability*, not a policy. See
    [`ADR-014`](docs/architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md).
12. **A shared six-digit code is never proof of teacher identity.** It may only
    ever authorise one action, or act as a second factor.
13. **Deterministic parsing precedes AI**, and AI never invents a missing value.
14. **No AI may publish timetable changes, approve substitutions, grant
    permissions, or issue robot commands.** It may only propose.
15. **Publication always requires human approval.** No automated path reaches a
    classroom display.
16. **A photograph of a whiteboard is never an authoritative record** — it is an
    input to a proposal.
17. **Stale data is shown only with a visible marker.** A blank screen is better
    than a confidently wrong answer.
18. **Search never returns a file the user cannot open**, re-verified at open time.
19. **No external AI service receives school data** without written authorisation.
20. **A school LAN is not trusted.** Updates are signed; administration surfaces
    are separated; secrets live nowhere near Git, URLs, logs or browser storage.

---

## Licence

Not yet determined. See [`LICENSE_REVIEW.md`](LICENSE_REVIEW.md) — this is an open
decision and no licence has been chosen.