# SHANA — School Route Survey

**Status: NOT STARTED. No survey has been carried out. No route has been walked,
measured, or approved.**

---

## Why a route cannot be designed from a floor plan

⚠️ **A route graph may only be built from a person having physically walked the
route.** A floor plan shows corridors; it does not show furniture, cables, rugs,
storage, cleaning equipment, or where students actually stand.

Every segment in the route graph exists because someone recorded it after walking
it. No segment exists because a planner would like it to.

---

## Known and unknown

### Known (from the project brief — ⚠️ **unverified, not surveyed**)

| Item | Detail |
|---|---|
| Blocks | A, B, C — connected by indoor corridors |
| Levels | Approximately 3–4 per block; A Block has no second floor |
| C↔B transition | ⚠️ Believed available **only via basement and ground floor** |
| B↔A route | ⚠️ **NOT SURVEYED. Does not exist in any document** |
| C Block basement | Sports facilities and other laboratories |
| A Block first floor | HR, DNT lab, **Makerspace Lab** (proposed checkpoint CP-A1) |
| C Block third floor | Grade 9 classrooms and the **AI Lab** (proposed checkpoint CP-C3) |
| Lifts | One in C Block, one in B Block |
| A Block ground floor | ⚠️ **"Areas that still need to be documented"** |

⚠️ **Every item above is a brief input, not a measured fact.** The first survey
task is to confirm all of it.

### Unknown — and required before any route exists

| # | Unknown | Question |
|---|---|---|
| 1 | Stair riser heights, treads, widths | `SH-STAIR-001`, `-002` |
| 2 | Corridor clear widths | `FAC-02` |
| 3 | Floor surface types | `FAC-03` — ⚠️ polished terrazzo is the worst case |
| 4 | Thresholds, ramps, level changes, cable runs | `FAC-04` |
| 5 | The B↔A route | `FAC-05` — ⚠️ **has never been surveyed** |
| 6 | A Block ground floor layout | ⚠️ **not documented** |
| 7 | Permanent and recurring obstructions | `FAC-06` |
| 8 | Which corridors are authorised for robot access | `SH-CONG-009` |
| 9 | Fire and evacuation routes to exclude | `SH-BLOCK-015` |
| 10 | Checkpoint security and power | `SH-MAP-023`, `-024` |

---

## Survey protocol (proposed)

⚠️ **To be approved by the school before any access.** `SH-BLOCK-011`, `FAC-P1`.

### Phase 1 — Desk review (no building access needed)

- [ ] Collect floor plans for all three blocks
- [ ] Collect room allocation lists
- [ ] Produce a draft block/floor/room inventory
- [ ] **Identify gaps in the documentation** — especially A Block ground floor and the B↔A connection
- [ ] Produce a proposed survey route for approval

### Phase 2 — Physical survey (requires approval and escort)

For every candidate route:

| Record | Unit | Notes |
|---|---|---|
| Segment ID | — | Stable identifier |
| From → to | — | Named nodes |
| Clear width | mm | Narrowest point |
| Floor surface | — | Tile / terrazzo / vinyl / carpet / concrete |
| Step or threshold heights | mm | Every one |
| Ramps and gradients | ° or % | |
| Lighting | — | Adequate / poor / emergency-only |
| Permanent obstructions | — | Furniture, extinguishers, storage |
| Recurring obstructions | — | Cleaning trolleys, bins |
| Cable runs | — | ⚠️ **A trip hazard at any speed** |
| Stairs | mm riser | ⚠️ **>160 mm excludes the segment** |
| Fire doors | — | ⚠️ **Normally held open?** |
| Wi-Fi coverage | — | If surveyed alongside |
| Photographs | — | Every segment |
| Surveyed by / date | — | ⚠️ **Required for every segment** |

### Phase 3 — Destination verification

⚠️ **A destination is not verified because it appears on a floor plan.**

For each candidate destination:

- [ ] A named person has **physically walked to the location**
- [ ] The **approach pose** has been recorded and checked for clearance
- [ ] The robot has **physically reached** that pose under supervision
- [ ] The pose is safe: no cables, no doors to open, no people required to move
- [ ] `verified_by` and `verified_pose_date` recorded
- [ ] An ArUco marker is approved, installed, and its ID recorded

⚠️ **Status defaults to `NEEDS_SURVEY`.** It becomes `ACTIVE` only when every box
above is complete.

---

## Obstruction recording

| Type | Example | Treatment |
|---|---|---|
| **Permanent** | Fire extinguishers, wall-mounted equipment, fixed furniture | Record in the graph as a clearance constraint |
| **Recurring** | Cleaning trolleys, waste bins, stacked chairs | Record; define a response |
| **Temporary** | Deliveries, maintenance, events | ⚠️ **Closure mechanism** — see `SH-BLOCK-001` |
| **Dynamic** | Student groups | ⚠️ **Congestion handling** — see `SH-CONG-007` |

⚠️ **Cables deserve special attention.** Field reports describe a quadruped
colliding with a door while in autonomous navigation. A cable across a corridor
is a trip hazard at any speed and is frequently invisible to sensors below the
camera's resolution.

---

## The route graph

⚠️ **Built from survey data only.** See
`docs/shana/architecture/route_graph.md` for the full model.

| Property | Requirement |
|---|---|
| Source | Physical survey, recorded by a named person |
| Every edge | Carries state, authorisation, constraints, `last_updated` |
| Authorisation | **Only segments explicitly approved for robot access may appear** |
| Fire routes | ⛔ **Excluded** |
| Unverified segments | ⛔ **Must not exist in the graph** |
| Stair edges | ⚠️ May exist as physical connections but are **excluded from routing** |
| Lift edges | ⚠️ Same — excluded from routing |

⚠️ **A graph edge means "physically connected", not "permitted for this robot."**
The planner must not select a staircase or lift merely because an edge exists.

---

## The survey that could eliminate the project

⚠️ **`FAC-01` — measuring stair risers — is the single most decisive measurement
in the whole project.**

```
Go2 documented maximum step height: 160 mm

Riser ≤ 160 mm  → stairs *might* be feasible, subject to test and approval
Riser > 160 mm  → stairs are PERMANENTLY OUT OF SCOPE
                  before payload, before supervision, before wear
```

**A tape measure and an afternoon could settle it.**

And the survey may reveal something better:

- ✅ A **single-floor route** to every needed room → stairs never arise
- ✅ A **ramp** that avoids the stair problem entirely
- ✅ A **ground-floor connection** between blocks that removes the need to change floors

⚠️ **A survey that produces "we do not need stairs" has been a success, not a
failure.**

---

## Survey questions

| ID | Question | Respondent | Priority |
|---|---|---|---|
| `FAC-01` | **Stair riser heights, treads, widths** | Facilities | **P0** |
| `FAC-02` | Corridor and stairwell clear widths | Facilities | **P0** |
| `FAC-03` | Floor surface types | Facilities | **P0** |
| `FAC-04` | Thresholds, ramps, cable runs, level changes | Facilities | P1 |
| `FAC-05` | **B↔A route survey** | Facilities | **P0** |
| `FAC-06` | Permanent and recurring obstructions | Facilities | P1 |
| `FAC-07` | Checkpoint confirmation, security, power | Facilities | P1 |
| `SH-CONG-007` | **Verified corridor segment graph** | Facilities + team | **P0** |
| `SH-CONG-008` | Verified alternate routes | Facilities + team | P1 |
| `SH-CONG-009` | **Authorised segments** | Administration | **P0** |
| `SH-BLOCK-015` | Fire and evacuation routes to exclude | Facilities | **P0** |
| `SH-MAP-012` | Who approves a destination as verified | Administration | **P0** |
| `SH-MAP-013` | What verification a destination must complete | Facilities + team | **P0** |

---

⚠️ **Nothing in this document has been surveyed, measured, or walked. No floor
plan has been consulted. The route graph does not exist.**