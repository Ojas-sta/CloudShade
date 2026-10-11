# SHANA — Route Graph and Destination Model

**Status: PROPOSED. No route has been surveyed, mapped, or verified.**

---

## The core distinction

Five things are routinely confused. Conflating them is how a delivery robot ends
up driving into a cupboard.

| # | Concept | What it is | What it does **not** give |
|---|---|---|---|
| 1 | **Named destination** | A database record: `C3_AI_LAB`, display name, block, floor | Does not imply the location exists or is reachable |
| 2 | **Route-graph node** | A verified intersection in the approved corridor network | Does not imply the robot can localise there |
| 3 | **Localisation landmark** | A feature used for pose correction | Does not imply a safe approach pose exists |
| 4 | **ArUco marker ID** | An integer the camera reads | Does not imply correct identification without validation |
| 5 | **Verified navigation goal** | A pose on a validated map, physically reached and confirmed by a person | — only this is a safe goal |

⚠️ **Only concept 5 may ever be the target of a motion command.** Concepts 1–4
are inputs to validation, never instructions to move.

⚠️ **Placing ArUco markers does not make a route safe.** A marker can tell the
robot it is on floor 3. It cannot tell it the corridor is clear, the door is
open, or the far end is not under repair.

---

## Provisional destination names

> ⚠️ **These are names only. None is a verified navigation coordinate. None has
> been surveyed.** They are placeholders awaiting a route survey.

| Name | Meaning | Block | Floor | Status |
|---|---|---|---|---|
| `C3_AI_LAB` | AI Lab — proposed primary base | C | 3 | **Unverified** |
| `A1_MAKERSPACE` | Makerspace Lab — proposed secondary | A | 1 | **Unverified** |
| `C0_GRADE12` | Grade 12 classrooms | C | 0 | **Unverified** |
| `B1_GRADE7` | Grade 7 classrooms | B | 1 | **Unverified** |
| `B2_GRADE8` | Grade 8 classrooms | B | 2 | **Unverified** |
| `CP-C3` | Checkpoint — AI Lab | C | 3 | **Unverified** |
| `CP-A1` | Checkpoint — Makerspace | A | 1 | **Unverified** |
| `CP-B0` | Checkpoint — B Block basement | B | 0 | **Unverified** |
| `CP-C0` | Checkpoint — C Block ground | C | 0 | **Unverified** |
| `CP-CB` | Checkpoint — C↔B transition | C/B | 0 or −1 | **Unverified** |

⚠️ **The C↔B transition is believed to exist only at basement and ground level.**
The B↔A route is **unsurveyed**. Neither may appear in any route graph until
measured.

---

## Destination record schema

Every destination carries verification metadata. A record without it is invalid.

```
destination_id        : immutable key, never reused
display_name          : "AI Lab"
block                 : C
floor                 : 3
floor_map_id          : c_block_f3          ← must reference an existing map

graph_node_id         : must exist in the verified route graph
approach_pose         : the pose the robot stops at
approach_pose_verified: true | false       ← default FALSE

arrival_fiducial_id   : ARUCO_0042          ← required for confirmation
allowed_origins       : [CP-C3, CP-C0]
allowed_items         : [STATIONERY_BOX, WORKSHEET_FOLDER]
max_payload_kg        : 2.0

floor_transition      : NONE | ESCORTED_LIFT | STAIRS   ← default NONE

status                : ACTIVE | BLOCKED | NEEDS_SURVEY
verified_by           : <name>              ← required for ACTIVE
verified_pose_date    : YYYY-MM-DD          ← required for ACTIVE
verification_expires  : YYYY-MM-DD          ← poses expire
```

⚠️ **`status` defaults to `NEEDS_SURVEY`.** A destination cannot become `ACTIVE`
without a named human having physically verified the approach pose.

---

## Route graph structure

```
     ┌─────────────────────────────────────────────────────────┐
     │  ROUTE GRAPH                                             │
     │                                                          │
     │  Nodes = verified corridor intersections                 │
     │  Edges = verified corridor segments                      │
     │  Edges carry constraints; they are never assumed          │
     │                                                          │
     │      ┌───────┐                                            │
     │      │CP-C3  │──── segment ────►┌───────────┐             │
     │      │node_1 │                 │ C3_AI_LAB │             │
     │      └───────┘                 │  node_7   │             │
     │          │                      └───────────┘             │
     │      segment                                              │
     │          │  constraints:                                   │
     │          │  · state: AVAILABLE|BUSY|HAZARD|CLOSED|UNKNOWN │
     │          │  · authorised_for_robot: true|false            │
     │          │  · floor_transition: NONE|STAIRS|ESCORTED_LIFT │
     │          │  · max_speed_mps                                │
     │          │  · last_updated                                 │
     │          ▼                                                │
     │      ┌───────┐                                            │
     │      │node_4 │───► ...                                     │
     │      └───────┘                                            │
     │                                                          │
     │  ⚠️ An edge exists in the graph only because a person      │
     │     walked it and recorded it. It does not exist because  │
     │     the algorithm would like it to.                        │
     └─────────────────────────────────────────────────────────┘
```

---

## Edge state model

| State | Meaning | Routing effect |
|---|---|---|
| `AVAILABLE` | Verified, open, no known hazard | Traversable |
| `BUSY` | Congested | Traversable if an alternate exists; otherwise a hard block |
| `HAZARD` | Wet floor, cleaning, damage, spill | **Blocked.** Confirmation required to clear |
| `CLOSED` | Construction, fire route, restricted | **Blocked.** Named person must reopen |
| `UNKNOWN` | Stale data, sensor failure, no recent report | **Blocked.** Conservative default |

⚠️ **`UNKNOWN` is never treated as `AVAILABLE`.** This is the single most
important rule in the route graph.

⚠️ **`BUSY` is a hard block when no verified alternate exists.** The system must
not assume an alternate exists simply because the graph suggests one.

---

## Staleness

| Data | Expiry | On expiry |
|---|---|---|
| Corridor occupancy estimate | Short — set by sensor interval | `UNKNOWN` |
| Hazard report | Short | `UNKNOWN` |
| Closure | **No automatic expiry** | Remains `CLOSED` until a named person reopens |
| Destination verification | Medium — set by policy | `NEEDS_SURVEY` |
| Map | On any layout change | Invalidated; all destinations revert |

⚠️ **Closures do not expire automatically.** Auto-reopening a corridor that is
still closed is a safety failure. Only a named person reopens.

---

## How a route is selected

```
1. Task references origin and destination by approved ID
2. Both must have status = ACTIVE and unexpired verification
3. Resolve origin and destination to graph nodes
4. Search the graph, excluding any edge that is not AVAILABLE
5. Also exclude any edge with authorised_for_robot = false
6. Also exclude any edge whose floor_transition is STAIRS or ESCORTED_LIFT
      ⚠️ unless the route's transit type is explicitly authorised
7. If no path exists → task rejected with the specific reason
8. Selected path is checked against battery reserve
9. If return route also unavailable → task rejected
```

⚠️ **Step 6 is a safety rule, not an optimisation.** The planner must not select
a staircase or lift merely because an edge exists. A route graph edge means
"physically connected", **not** "permitted for this robot".

---

## Multi-floor representation

⚠️ **The robot has no sensor that can determine which floor it is on.** Verified
negative across every message field in the platform.

**Consequences:**

| Operation | Single floor | Multi-floor |
|---|---|---|
| Localisation within a floor | Scan matching | Scan matching |
| Determining which map applies | Unnecessary | ⚠️ **Impossible with stock sensors** |
| Required extra infrastructure | None | **ArUco marker per floor, plus human confirmation** |
| Map representation | One map | One map per floor, switched explicitly |

### Floor identity: three required layers

1. **Unique ArUco marker per floor**, marker ID encoding the floor.
2. **Human confirmation on every floor transition.**
3. **A relocation event** whenever anyone manually handles the robot — it
   invalidates the pose and forces re-localisation.

⚠️ **Missing, damaged, or ambiguous marker → `UNKNOWN`, never a guess.** Guessing
produces the silent wrong-floor failure described below.

### The silent wrong-floor failure

> A person carries the robot up a flight of stairs and sets it down on floor 3.

1. Odometry is destroyed — expected, and recoverable.
2. Scan matching against **floor 3** succeeds.
3. Scan matching against **floor 1 also succeeds**, because school corridors
   look alike.
4. **The robot therefore does not know it is on the wrong map.** Nothing crashes.
   Nothing logs an error. It plans confidently against floor 1 while standing on
   floor 3.

⚠️ **This is the most dangerous failure mode in the entire system because it is
silent.** It is why floor identity is a P0 question and why the relocation event
is mandatory.

---

## Checkpoints

| Property | Requirement |
|---|---|
| Location | Approved, surveyed, verified |
| Reachability | Reachable from every operating segment on `AVAILABLE` edges |
| Power | Mains supply for charging, or a spare battery policy |
| Security | Physical control against student access |
| Identification | ArUco marker for reliable identification |
| Storage | Approved arrangement when not operating |

⚠️ **A robot sitting at a checkpoint is not automatically secure.** The system
must not assume the robot is still at the checkpoint because nothing told it
otherwise — which is why the relocation event exists.

---

## What is required before any graph is built

- [ ] Physical survey of every candidate corridor and intersection
- [ ] Measurement of clear widths, floor surfaces, thresholds, ramps
- [ ] Identification of fire routes and restricted areas
- [ ] Inventory of permanent and recurring obstructions
- [ ] Confirmation that the C↔B transition exists as believed
- [ ] **Survey of the B↔A route, which does not currently exist in any document**
- [ ] Named authority approving each segment for robot access
- [ ] Physical walk-and-verify of every destination approach pose
- [ ] Marker installation approved and inventoried

**Questions:** `SH-MAP-001` to `-024`, `SH-CONG-005` to `-010`,
`SH-STAIR-001` to `-011`

⚠️ **No route graph may be created from a floor plan alone.** It must come from
someone walking the building.