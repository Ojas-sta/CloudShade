# SHANA — Data Flow

**Status: PROPOSED. No data has been collected, stored, or transmitted.**

---

## What flows, and what must never flow

| Data | Direction | Sensitivity | Notes |
|---|---|---|---|
| Delivery request | Teacher → server | Low | Must reference allow-listed IDs only |
| Task state | Server → dashboard | Low | No personal data |
| Robot telemetry | Robot → server | **Medium** | Position reveals school layout |
| Occupancy state | Sensors/staff → server | **High if derived from CCTV** | ⚠️ Only if separately approved |
| Corridor status | Staff → server | Low | Named author only |
| Battery and fault state | Robot → server | Low | |
| Movement commands | Server → robot | **Critical** | ⚠️ Only path that causes motion |
| Identity/authentication | Server-internal | **Critical** | ⛔ Never in the repository |

---

## Flow 1 — Delivery request

```
Teacher browser
     │  POST (destination_id, item_id)
     ▼
Server intake ──▶ validate schema ──▶ resolve IDs against registry
     │                                    │
     │ invalid                    not found │  found
     ▼                                    ▼
  REJECTED                    ┌──────────────────────┐
  (reason shown)              │  TASK VALIDATOR      │
                              │  (pure function)     │
                              └──────────┬───────────┘
                                         │ reject
                                         ▼
                              ┌──────────────────────┐
                              │  Specific reason:    │
                              │  · not approved      │
                              │  · route unavailable  │
                              │  · battery insufficient│
                              │  · outside hours     │
                              │  · no supervisor     │
                              └──────────────────────┘

                                         │ accept
                                         ▼
                                     QUEUED
```

⚠️ **Rejection always carries a specific reason.** "Request failed" is a bug, not
acceptable UX. A teacher who cannot understand why a request was refused will
assume the system is broken.

⚠️ **The registry lookup is closed.** If a name does not exist, it is rejected. It
is never fuzzy-matched and never invented.

---

## Flow 2 — Task execution

```
Server
  │
  ├─ Read battery ──────────── rt/lowstate → LowState.bms_state
  ├─ Read position, faults ─── rt/sportmodestate
  ├─ Read corridor states ──── corridor store
  ├─ Plan route ────────────── route planner (excludes non-AVAILABLE)
  │
  ▼
Safety supervisor
  ├─ Heartbeat present?
  ├─ Battery above threshold?
  ├─ Speed within cap?
  ├─ Within approved zone?
  │     │
  │   reject ──▶ STOP, notify, log
  │
  ▼ approve
Robot adapter
  │
  ├─ Motion command ──────────▶ [WIRED ETHERNET] ──▶ SportClient / ObstaclesAvoidClient
  │                                                          │
  │                                                          ▼
  │                                                    Go2 EDU
  │                                              moves / stops / faults
  │                                                          │
  │   Telemetry ◀────────────────────────────────────────────┘
  │
  ▼
Log every transition with timestamp, reason, battery, and position
```

⚠️ **Every arrow crossing the network boundary is logged.** A complete audit trail
is a prerequisite for debugging, for incident review, and for trust.

---

## Flow 3 — Telemetry

| Field | Source topic | Frequency | Retention | Sensitivity |
|---|---|---|---|---|
| State of charge | `rt/lowstate` → `bms_state.soc` | ~1 Hz | Mission | Low |
| Pack current (signed) | `bms_state.current` | ~1 Hz | Mission | Low |
| Pack voltage | `LowState.power_v` | ~1 Hz | Mission | Low |
| Battery temperature | `bms_state.bq_ntc` | ~0.1 Hz | Mission + trend | Low |
| Cell voltages | `bms_state.cell_vol[15]` | ~0.1 Hz | Trend | Low |
| Cycle count | `bms_state.cycle` | Per session | Long-term | Low |
| Position, velocity | `rt/sportmodestate` | ~5 Hz | Mission | **Medium** |
| Foot forces | `sportmodestate.foot_force` | ~5 Hz | Mission | Low |
| Gait type | `sportmodestate.gait_type` | ~1 Hz | Mission | Low |
| IMU | `rt/lowstate` | ~50 Hz | Ring buffer | Low |
| Error code | `sportmodestate.error_code` | On change | Long-term | **High** — drives fault handling |
| LiDAR cloud | `rt/utlidar/cloud_deskewed` | ~10 Hz | **Not retained** | Medium |
| Height map | `rt/utlidar/height_map_array` | ~5 Hz | **Not retained** | Low |

⚠️ **Point clouds and height maps are not retained.** They exist only to support
localisation in real time. Retaining them would create a record of the school's
internal layout at high resolution.

⚠️ **Position data is treated as sensitive** because it reveals the building's
layout and, over time, could reveal staff movement patterns. It is retained per
mission and then discarded or aggregated.

---

## Flow 4 — Corridor status

```
Sources (mutually substitutable):
  ├── Facilities manual declaration  (authorised named person)
  ├── Housekeeping schedule           (cleaning windows)
  ├── Wet-floor sign detection        (ADVISORY ONLY → prompts human confirmation)
  └── Occupancy sensor                (IF SEPARATELY APPROVED)
              │
              ▼
        Corridor state store
   AVAILABLE | BUSY | HAZARD | CLOSED | UNKNOWN
              │
              ▼
        Route planner reads state
              │
              ▼
   Only AVAILABLE segments are traversable
   ⚠️ UNKNOWN is never treated as AVAILABLE
```

⚠️ **Sensors are advisory. Authority is human.** No sensor may close or open a
corridor. A sensor may raise a prompt; a named person decides.

⚠️ **Every state change is logged with who made it, when, and on what basis.**

---

## Flow 5 — Notifications

| Event | To | Method | Timing |
|---|---|---|---|
| Request accepted | Requesting teacher | Dashboard | Immediate |
| Request rejected | Requesting teacher | Dashboard + reason | Immediate |
| Task started | Requesting teacher; supervisor | Dashboard | On transition |
| Robot waiting | Supervisor | Dashboard + audible | Immediate |
| Arrived at destination | Recipient | Dashboard + audible | Immediate |
| Delivery confirmed | Requesting teacher; recipient | Dashboard | On confirmation |
| Delivery failed | Requesting teacher; supervisor | Dashboard + reason | Immediate |
| Corridor closed | Supervisor; facilitator | Dashboard | On closure |
| Low battery | Supervisor | Audible + dashboard | At threshold |
| Recovery required | Supervisor | Audible + dashboard + escalation | Immediate |
| Fault | Supervisor; custodian | Audible + dashboard | Immediate |
| Safety veto | Supervisor; administrator | Audible + dashboard | Immediate |

⚠️ **Failures and faults always carry a reason.** A notification without a reason
is a notification the recipient must investigate, which is worse than no
notification.

---

## Data retention

| Data | Retention | Rationale |
|---|---|---|
| Delivery records and audit log | Per school policy — **TBD** | Required for dispute resolution and incident review |
| Mission telemetry | Per mission | Required for measurement |
| Point clouds and height maps | **Not retained** | Avoids a high-resolution layout record |
| Battery health trends | Long-term | Lifecycle management |
| Fault history | Long-term | Reliability analysis |
| Camera or CCTV imagery | **Not retained by default** | Privacy |
| Derived occupancy data | **TBD — `SH-PRIV-006`** | ⚠️ Unresolved; may be personal data |
| Manual corridor reports | Per school policy | Audit of closures |

⚠️ **Retention periods are deliberately marked TBD.** They are a school policy
decision, not a technical one. See `SH-PRIV-006`, `SH-PRIV-012`, `SH-SRV-016`.

---

## Prohibited flows

| Prohibited | Why |
|---|---|
| ⛔ Model output → robot command | The AI boundary |
| ⛔ Teacher input → velocity | Role separation |
| ⛔ Free-text destination → motion | Unverifiable target |
| ⛔ Any flow → public network | Design principle |
| ⛔ Credentials → repository | Absolute rule |
| ⛔ CCTV frame → disk | Privacy, unless separately approved |
| ⛔ Camera footage → student-facing display | Privacy |
| ⛔ Occupancy count → individual assessment | Prohibited use |
| ⛔ Sensor → corridor closure without a human | Sensors are advisory only |

---

## The data flow invariant

> **Every byte that can cause physical motion passes through exactly one
> deterministic function — the task validator — and one independent safety
> supervisor.**

Everything else in the system is either telemetry flowing the other way, or
information a human acts on.