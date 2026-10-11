# SHANA — Checkpoints and ArUco Markers

**Status: PROPOSED. No marker has been designed, produced, installed, or tested.**

---

## Two distinct jobs — do not conflate them

| Job | Marker role | Failure consequence |
|---|---|---|
| **Floor identity** | Marker ID encodes the floor | Robot cannot determine which map applies |
| **Checkpoint confirmation** | Marker ID confirms the robot is at a known place | Arrival cannot be verified |

⚠️ **A marker can identify a place. It cannot make that place safe.** Placing
markers does not create a traversable route, does not clear an obstruction, and
does not authorise a destination.

---

## Why markers rather than general vision

| Reason | Detail |
|---|---|
| Low frame rate is tolerated | The camera may be 15 fps. Markers are large, static, high-contrast |
| Works in poor light | A marker is readable where object recognition fails |
| Absolute identity | An integer, not a similarity score |
| **Already proven on this robot** | The charging pile mechanism is ArUco-based |
| Verifiable | A human can confirm the expected ID at each location |

⚠️ **The last point matters most.** A marker system is *auditable* — a person can
stand at a location and check that the right marker is mounted. A neural
detector cannot be checked that way.

---

## Marker specification (proposed)

| Parameter | Proposed value | Rationale |
|---|---|---|
| Family | ArUco or AprilTag | Open, well-documented |
| Size | **≥ 200 mm** | Distance and angle tolerance |
| Mounting height | Set by camera geometry — **to be measured** | Robot camera is ~0.4 m high |
| Standoff distance | **To be measured** (`SH-MAP-017`) | |
| Viewing angle | **To be measured** (`SH-MAP-019`) | Off-axis detection degrades sharply |
| Contrast | Maximum against surroundings | Plain wall, no glare |
| Illumination | Consistent with corridor lighting | A dark corridor is a real constraint |
| ID assignment | **Floor encoded in the ID range** | So a marker identifies its floor |

⚠️ **Detection rate must be measured at every marker location** before the system
relies on it. `SH-MAP-017`, `SH-MAP-018`, `SH-MAP-019`.

---

## Floor identity encoding

⚠️ **The Go2 has no sensor that can determine which floor it is on.** Verified
negative: every accessor across every Go2 message field was enumerated, and there
is no altitude, pressure, barometer, or floor field.

Therefore multi-floor operation requires **three independent layers**:

| Layer | Mechanism | Why it alone is insufficient |
|---|---|---|
| 1 | **ArUco marker per floor**, ID encodes floor | Fails if missing, damaged, or obscured |
| 2 | **Human confirmation on every floor transition** | Depends entirely on a human noticing |
| 3 | **Relocation event** whenever anyone manually handles the robot | A carried robot looks identical to one that walked there |

⚠️ **All three are required. Any one alone is insufficient.**

### The silent wrong-floor failure

> A person carries the robot up a flight of stairs and sets it down on floor 3.

1. Odometry is destroyed — recoverable.
2. Scan matching against **floor 3** succeeds.
3. Scan matching against **floor 1 also succeeds**, because school corridors look
   alike.
4. **The robot does not know it is on the wrong map.** Nothing crashes, nothing
   logs an error. It plans confidently against floor 1 while standing on floor 3.

⚠️ **This is the most dangerous failure mode in the system because it is silent.**
Independent work on floor-ambiguity in lifts confirms it: *"the areas immediately
adjacent to the lift are identical."*

---

## Marker failure handling

| Situation | Required behaviour |
|---|---|
| Marker not detected | `UNKNOWN` — **never assume a floor** |
| Marker detected but ID unknown | `UNKNOWN` — reject, do not guess |
| Marker ID contradicts the expected floor | **Abort the task.** Do not proceed. |
| Multiple markers with conflicting IDs | `UNKNOWN` — requires human resolution |
| Marker damaged or missing | Human reports it; robot treats location as `UNVERIFIED` |
| No marker at a checkpoint | Checkpoint cannot be confirmed; use an alternative method or mark unverified |

⚠️ **A failed marker read must never default to "the floor I was on last".** That
is precisely the silent failure above.

---

## Relocation events

Any of the following sets `pose_valid = false`:

- A person lifts or carries the robot
- A person repositions it within a floor
- The battery is removed for a swap
- The robot is transported anywhere

**Required behaviour while `pose_valid` is false:**

- No new task accepted
- Must re-localise before resuming
- Dashboard must show the state prominently
- **Entering the event must be unmissable in the console** — an unrecorded
  relocation is the most likely way the robot ends up somewhere it believes it is
  not

⚠️ **The dashboard must also display `last_pose_age` next to any position.** A
40-minute-old position shown beside a physical robot invites a person to look in
the wrong place.

---

## Checkpoints

| Property | Requirement |
|---|---|
| Identity | ArUco marker for reliable identification |
| Status | Reachable from every operating segment via `AVAILABLE` edges |
| Power | Mains for charging, or a documented spare-battery policy |
| Security | ⚠️ **Physical control against student access must be agreed, not assumed** |
| Storage | Approved arrangement when not operating |
| Approval | Named authority approves each checkpoint |

### Provisional checkpoint names

⚠️ **Names only. None is a verified coordinate. None has been surveyed.**

| ID | Location | Block | Floor | Status |
|---|---|---|---|---|
| `CP-C3` | AI Lab — proposed primary base | C | 3 | **Unverified** |
| `CP-A1` | Makerspace Lab — proposed secondary | A | 1 | **Unverified** |
| `CP-B0` | B Block basement | B | 0 | **Unverified** |
| `CP-C0` | C Block ground | C | 0 | **Unverified** |
| `CP-CB` | C↔B transition — believed basement/ground only | C/B | 0/−1 | **Unverified** |

⚠️ **A robot sitting at a checkpoint is not necessarily still at the checkpoint.**
The system must not assume it is, and a human who moves it must record that.

---

## Marker installation

Physical installation requires approval.

- [ ] School authorisation obtained (`SH-MAP-021`, `ADM-A5`)
- [ ] Marker inventory with ID and location recorded
- [ ] Every marker verified in place by a named person
- [ ] Detection rate measured at every location
- [ ] **Expected ID confirmed visually by a person at each location** — a
      mis-numbered marker is a wrong-floor bug

⚠️ **A marker inventory that disagrees with reality is worse than no markers,
because it produces false confidence.**

---

## Questions

| ID | Question | Priority |
|---|---|---|
| `SH-MAP-016` | What is the marker's specific purpose — floor identity, pose correction, checkpoint confirmation, or a combination? | P1 |
| `SH-MAP-017` | Marker size, mounting height, standoff distance | P1 |
| `SH-MAP-018` | Lighting conditions at each marker location | P1 |
| `SH-MAP-020` | What happens when a marker is missing or misidentified? | P1 |
| `SH-MAP-021` | **Does the school permit markers to be fixed to walls, doors, or lift lobbies?** | **P0** |
| `SH-MAP-022` | How will marker identity and location be validated? | P1 |
| `SH-MAP-023` | Which locations are approved checkpoints? | P1 |
| `SH-MAP-024` | **Is each checkpoint secure against student interference?** | **P0** |