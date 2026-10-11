# SHANA — Corridor Availability

**Status: PROPOSED. No corridor has been surveyed, declared, or monitored.**

---

## Corridor states

| State | Meaning | Routing effect |
|---|---|---|
| `AVAILABLE` | Verified, open, no reported hazard | Traversable |
| `BUSY` | Congested | Traversable **only** if a verified alternate exists |
| `HAZARD` | Wet floor, cleaning, damage, spill | **Blocked.** Confirmation required to clear |
| `CLOSED` | Construction, fire route, restricted | **Blocked.** Named person must reopen |
| `UNKNOWN` | Stale data, sensor failure, no recent report | **Blocked.** Conservative default |

⚠️ **`UNKNOWN` is never treated as `AVAILABLE`.** This is the single most
important rule in the corridor model.

⚠️ **`BUSY` is a hard block when no verified alternate exists.** The system must
not assume an alternate exists because the graph suggests one.

---

## Who may declare what

| Action | Who | Evidence needed |
|---|---|---|
| Declare `HAZARD` | Named authorised person | Reason recorded |
| Clear `HAZARD` | Same person, or another named authorised person | Confirmation recorded |
| Declare `CLOSED` | Named authorised person | Reason recorded |
| Reopen `CLOSED` | Same person, or another named authorised person | Confirmation recorded |
| Declare `BUSY` | Automated, **advisory only** | — |

⚠️ **Authority is defined by the school** (`SH-ENV-001`, `SH-BLOCK-001`). SHANA
has no opinion on who should hold it.

⚠️ **No closure expires automatically.** Auto-reopening a corridor that is still
closed is a safety failure. Only a named person reopens. (`SH-BLOCK-003`,
`SH-BLOCK-004`)

---

## Where corridor state comes from

| Source | Authoritative? | Notes |
|---|---|---|
| Facilities declaration | ✅ **Authoritative** | The primary mechanism |
| Housekeeping schedule | ✅ **Authoritative** | Cleaning windows |
| Manual staff report | ✅ **Authoritative** | Fastest mechanism |
| Occupancy sensor | ⛔ **Advisory only** | ⚠️ May raise a prompt; may **never** close a corridor |
| Wet-floor sign detection | ⛔ **Advisory only** | ⚠️ Visual detection of water is **not** proof of floor safety |

⚠️ **Sensors raise prompts. Humans decide.** This inverts the usual automation
instinct, and it is deliberate.

---

## The hazard state machine

| State | Safe behaviour | Notification | Routing | Resume condition |
|---|---|---|---|---|
| `CONFIRMED_SAFE` | Proceed at reduced speed | None | Allowed | Ongoing |
| `POSSIBLE_HAZARD` | **Stop** | Custodian alerted | Segment excluded | Human confirms, with time limit |
| `CONFIRMED_CLOSED` | Stop and hold or return | Facilities + teachers | Segment excluded | Named person reopens |
| `UNKNOWN` | **Treat as `POSSIBLE_HAZARD`** | Advisory only | Segment excluded if stale | Fresh reading or human report |
| `SENSOR_FAILURE` | Degrade to human-only | Fault alert | Conservative: exclude unverified segments | Sensor restored and human-confirmed |

⚠️ **Every state resolves to "stop" rather than "proceed".** A system that
occasionally proceeds on a guess is a system that will eventually guess wrong.

---

## What cannot be detected by any sensor

| Hazard | Why not |
|---|---|
| **Surface friction** | ⛔ Not detectable by any camera or depth sensor |
| **A thin cable** | Below sensor resolution |
| **Low-friction surface with no visible water** | Invisible |
| **A floor that is *about* to become wet** | No sensor predicts it |
| **Slippery polished terrazzo** | Only measurable by physical test |

⚠️ **The manufacturer's manual explicitly forbids wet and low-friction ground and
names foot slip as a failure mode. That prohibition exists because this cannot
be sensed.**

---

## Staleness

| Data | Expiry | On expiry |
|---|---|---|
| Occupancy estimate | Short — by sensor interval | `UNKNOWN` |
| Hazard report | Short — TBD | `UNKNOWN` |
| Closure | ⚠️ **No automatic expiry** | Remains `CLOSED` |
| Destination verification | Medium — TBD by policy | `NEEDS_SURVEY` |
| Map | On any layout change | Invalidated |

⚠️ **Stale data must never authorise traversal.** Expiry always degrades toward
`UNKNOWN`, never toward `AVAILABLE`.

---

## Congestion handling

⚠️ **Do not claim SHANA can avoid a busy corridor unless both the alternate route
and the navigation capability have been verified.**

| Step | Rule |
|---|---|
| Threshold selection | CLEAR / MODERATE / BUSY — set experimentally, not guessed |
| Confidence | Below threshold → `UNKNOWN`, never `CLEAR` |
| Hysteresis | A corridor stays busy for a defined period after the last positive reading |
| Oscillation | Prevented by commitment to a route once started |
| Mid-mission change | ⚠️ Only between verified segments. Never into unverified space |
| No alternate | Busy becomes a **hard block** |
| Predicted congestion | Optional, from timetable data. ⚠️ Students congregate unpredictably regardless |

---

## Route planning rules

1. Exclude every edge not `AVAILABLE`.
2. Exclude every edge not authorised for robot access.
3. ⚠️ Exclude stairs and lifts **regardless of graph edges** — a graph edge means
   "physically connected", not "permitted".
4. If no path exists → reject with the specific reason.
5. If no path is verified safe → **stop and request human assistance.**

⚠️ **A routing algorithm must never bypass barriers, enter restricted areas, or
select stairs or lifts simply because the graph contains an edge.**

---

## Operational questions

| ID | Question | Priority |
|---|---|---|
| `SH-ENV-001` | **Who may declare a corridor closed or hazardous?** | **P0** |
| `SH-ENV-002` | Can housekeeping publish a cleaning schedule SHANA can read? | P1 |
| `SH-ENV-003` | Can staff report a hazard from a phone? | P1 |
| `SH-ENV-015` | **Confirm the robot stops rather than attempting an unverified detour** | **P0** |
| `SH-ENV-016` | What happens when every route is affected? | P1 |
| `SH-ENV-020` | Who approves resuming after a hazard is cleared? | P1 |
| `SH-CONG-003` | What thresholds define CLEAR / MODERATE / BUSY? | P1 |
| `SH-CONG-006` | How is stale occupancy data handled? | P1 |
| `SH-CONG-011` | Does BUSY block, or merely deprioritise? | P1 |
| `SH-CONG-019` | What if no safe route exists? | P1 |
| `SH-BLOCK-001` | **Who may close a route?** | **P0** |
| `SH-BLOCK-004` | **Confirm no automatic resumption** | **P0** |
| `SH-BLOCK-008` | **Confirm closed edges are always rejected** | **P0** |
| `SH-BLOCK-012` | **Confirm mid-mission changes only between verified segments** | **P0** |
| `SH-BLOCK-015` | **Which routes are fire evacuation routes?** | **P0** |