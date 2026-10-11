# SHANA — System Context

## The system in one picture

```
┌───────────────────────────────────────────────────────────────────────────┐
│  SCHOOL ENVIRONMENT                                                     │
│                                                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Teachers │  │ Custodian│  │    IT    │  │Facilities│  │  Safety  │   │
│  │ (request)│  │ (physical│  │ (network)│  │ (routes, │  │ (policy, │   │
│  │          │  │  handling│  │          │  │  hazards)│  │  e-stop) │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │             │             │             │             │         │
└───────┼─────────────┼─────────────┼─────────────┼─────────────┼─────────┘
        │             │             │             │             │
        │        APPROVALS     PERMISSION       ROUTE DATA    SAFETY RULES
        │             │             │             │             │
┌───────▼─────────────▼─────────────▼─────────────▼─────────────▼─────────┐
│                                                                          │
│   ┌────────────────────────────────────────────────────────────────┐    │
│   │  LOCAL APPLICATION SERVER                        (AI Lab, CP-C3)│    │
│   │                                                                │    │
│   │   Teacher dashboard  ·  Delivery queue  ·  Route planner      │    │
│   │   Safety supervisor  ·  Corridor state  ·  Task validator      │    │
│   │   Telemetry & audit  ·  Notifications   ·  Admin controls     │    │
│   └───────────┬────────────────────────────────────┬───────────────┘    │
│               │ ROS 2 (Humble)                     │ REST / WebSocket │
│               │ over school LAN                    │ to teachers      │
│  ┌────────────▼───────────────────┐   ┌───────────▼─────────────┐     │
│  │  NAVIGATION & PERCEPTION      │   │  TEACHER BROWSERS       │     │
│  │  Mapping · Localisation       │   │  Any device on the      │     │
│  │  Path planning · Speed limit  │   │  school network         │     │
│  │  Marker detection · Corridor  │   └─────────────────────────┘     │
│  │  state ingestion             │                                    │
│  └───────────────┬───────────────┘                                    │
│                  │                                                     │
└──────────────────┼─────────────────────────────────────────────────────┘
                   │  Cyclone DDS over WIRED ETHERNET
                   │  Isolated 192.168.123.0/24 subnet
┌──────────────────▼─────────────────────────────────────────────────────┐
│  UNITREE GO2 EDU                                                       │
│                                                                        │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────────────┐  │
│  │ Motion &   │ │ Obstacle   │ │ 4D LiDAR   │ │ Battery / BMS      │  │
│  │ gait       │ │ avoidance  │ │ L2 → DDS   │ │ telemetry          │  │
│  │ (onboard,  │ │ (onboard,  │ │ point cloud│ │                    │  │
│  │  closed)   │ │  on by     │ │            │ │                    │  │
│  │            │ │  default)  │ │            │ │                    │  │
│  └────────────┘ └────────────┘ └────────────┘ └────────────────────┘  │
│                                                                        │
│  [ payload carrier ]  [ ArUco markers at checkpoints — proposed ]      │
│  [ human supervisor — always present ]                                 │
└────────────────────────────────────────────────────────────────────────┘
```

---

## System boundary

### Inside SHANA's control

| Component | Notes |
|---|---|
| Local application server | Full control. Owns all task logic. |
| Navigation and perception | Runs on the server host. |
| Route graph and destination directory | Human-verified, version controlled. |
| Safety supervisor | Independent; can veto any command. |
| Payload carrier | Our design. |
| Dashboard | Full control. |

### Outside SHANA's control

| Component | Owner | Implication |
|---|---|---|
| Robot gait, balance, low-level motion | Unitree, closed on-board binaries | Cannot be modified; must be used as documented |
| Robot obstacle avoidance | Unitree, closed | Can be enabled/disabled, not reconfigured |
| LiDAR driver | Unitree, closed | Point cloud is consumed read-only |
| School network | School IT | Subject to approval; must be isolated |
| Physical environment | Facilities | Routes, hazards, and closures are external inputs |
| Lift systems | Service provider | **No modification permitted** |
| CCTV | School | **Not accessed unless separately approved** |

⚠️ **The boundary between "controlled" and "external" is where every serious
failure mode lives.** The robot's own motion and avoidance systems are closed
and not ours to change; the environment is not ours to control. SHANA's job is
to work correctly within a boundary it does not own.

---

## Trust boundaries

| Boundary | Crossed by | Control |
|---|---|---|
| **Teacher → server** | Delivery requests | Authentication; allow-listed destinations; deterministic validation |
| **Server → robot** | Motion commands | Isolated subnet; authenticated; speed clamp; heartbeat |
| **Robot → server** | Telemetry | Read-only topics |
| **Staff → system** | Corridor status, closures | Named authority only; logged |
| **Environment → system** | Hazards, obstructions | Conservative default; `UNKNOWN` never treated as clear |
| **CCTV → system** | Occupancy metadata | **Not crossed.** No CCTV access unless separately approved |
| **Radio → system** | Telemetry backup | **Not crossed.** Telemetry only; never commands |
| **Repository → public** | Documentation | No credentials, no network details, no student data |

---

## External dependencies

| Dependency | Status | Failure impact |
|---|---|---|
| Unitree SDK and robot services | Available, closed | **Total loss of function** |
| School network availability | Unknown | Server unreachable → robot holds position |
| School electricity | Unknown | Server down → robot holds position; robot battery depletes |
| Human supervisor availability | Unknown | **No operation permitted** |
| Facilities corridor status | Unknown | Stale state → conservative routing |
| Approved item and destination lists | Unknown | Tasks rejected |
| Lift service provider *(later phases)* | Unknown | No vertical transit |

⚠️ **No external dependency may be relied upon without a defined fallback.**
Every one above has a safe degraded behaviour, and that behaviour is a
requirement, not an aspiration.

---

## Actors and their authority

| Actor | May | May not |
|---|---|---|
| Teacher | Create a request; confirm receipt; give feedback | Command motion; bypass validation; extend the destination list |
| Room administrator | All teacher actions; manage their block's destinations | Command motion directly |
| Robot custodian | All above; physically handle the robot; record relocation events | Override safety supervisor |
| Safety officer | **Stop anything at any time** | — |
| Facilities | Close/open corridors; declare hazards | Command motion |
| IT | Manage network; revoke access | Command motion |
| Data Fiduciary | Approve or refuse any data processing | — |
| Server safety supervisor | **Veto any command** | Initiate motion |
| Human supervisor | **Stop at any time, unilaterally** | — |
| The robot | Nothing | Everything |

⚠️ **There are five independent ways to stop the robot and only one way to start
it.** That asymmetry is intentional.

---

## System phases

| Phase | Depends on | Status |
|---|---|---|
| **0 — Feasibility** | Nothing | ✅ Complete |
| **1 — Read-only connection** | Gate 0, Gate 1 | ⛔ Not started |
| **2 — Simulation validation** | Nothing | ⛔ Not started — **can begin immediately** |
| **3 — Supervised movement** | Gate 8 | ⛔ Not started |
| **4 — Minimal end-to-end task** | Phase 3 | ⛔ Not started |
| **5 — Dashboard and queue** | Phase 4 | ⛔ Not started |
| **6 — Perception and autonomy** | Phase 5, approvals | ⛔ Not started |
| **7 — Testing and measurement** | Phase 6 | ⛔ Not started |
| **8 — Pilot** | Gates 0–9 passed | ⛔ Not started |
| **9 — Exhibition** | Gate 10 | ⛔ Not started |

⚠️ **Phase 2 requires no permission and should begin now.** Everything else is
blocked on an answer, an approval, or a measurement.