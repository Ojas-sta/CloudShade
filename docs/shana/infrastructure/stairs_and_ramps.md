# SHANA — Stairs and Ramps

**Status: RESEARCH ONLY. No stair test has been performed, scheduled, or
approved. No measurement has been taken.**

**Current position: stairs are excluded from the initial pilot.** See
`docs/shana/architecture/architecture_decisions/ADR-003-stairs-and-lifts-excluded.md`.

---

## The manufacturer's documented position

| Claim | Source | Class |
|---|---|---|
| Maximum climb/drop height **160 mm** | `about_Go2` | VERIFIED |
| *"not currently recommended that robot walk stairs higher than 16cm, otherwise it is likely to **trip over the foot**"* | Go2 User Manual §5 | VERIFIED |
| *"it is **forbidden** to use the robot on the occasions where the terrain is complicated… steps higher than 16cm"* | Go2 User Manual §9 | VERIFIED |
| Rated for slopes ≤40°; **not specified for stairs** | `about_Go2` | VERIFIED |
| Max incline for legged G1 | **NOT FOUND** | — |

⚠️ **The manufacturer's documented failure mode is a foot trip, not a clean fall.**
A leading leg catches the riser nose and the body pitches forward.

---

## ⚠️ The stair gait was removed from the SDK

| Firmware | Dedicated stair API | Evidence |
|---|---|---|
| **< 1.1.6** | ✅ `SwitchGait(3)` = forward climbing; `SwitchGait(4)` = reverse climbing | VERIFIED — `sports_services` doc, verbatim |
| **≥ 1.1.6** | ⛔ **`SwitchGait` is ABSENT** from Motion Services Interface V2.0 and from `SportClient` | VERIFIED |

`SwitchGait` still exists in the **B2-W** client, which confirms the removal from
Go2 was deliberate rather than an omission.

**Consequence:** on current firmware the only stair-capable method is
`ClassicWalk`, described as *"AI-classic gait, with strong adaptability to complex
terrain, supports climbing stairs, gravel, meadows, and slippery surfaces."*

⚠️ **That is a terrain-adaptation claim, not a stair mode** — with no
step-height parameter, no dedicated engagement, and no safety envelope.

The stair gaits still exist in the **consumer firmware**: the User Manual
documents two remote-control modes — *"go upstairs forwards and downstairs
backwards"* and *"go downstairs forwards"*. ⚠️ **Whether these are reachable from
the EDU SDK is UNKNOWN** (`SH-STAIR-015`).

---

## Ascent and descent are not symmetric

⚠️ Unitree provides **two** stair modes precisely because backing down is the
fallback, and every independent report describes **descent as the worse case**:

| Source | Report | Class |
|---|---|---|
| DroneBlocks, 2024-10 | 8-inch (203 mm) steps — **27% above spec** — visible struggle; *"loses balance a little bit on both stairs"* | UNVERIFIED |
| Miraenda, 2024-03 | *"can walk upstairs but has an issue walking downstairs"* | UNVERIFIED |
| mybotshop forum, 2025-09 | Post-update, stair mode removed from the remote; *"the robot just stopped in front of the stairs"* | UNVERIFIED |
| `go2-blind-stairs-newton` | *"blind proprioceptive locomotion is the wrong tool for clean stair climbing… **stair climbing is a perception problem**"* | UNVERIFIED |

⚠️ **There is no report anywhere of a Go2 EDU descending stairs carrying 8 kg
with a person present.** That is exactly the case we would need.

---

## The step-height arithmetic — the decisive measurement

| Typical riser | vs 160 mm | Verdict |
|---|---|---|
| 150 mm | **+10 mm (6.7%)** | Inside spec — **10 mm of tolerance** |
| 160 mm | **0 mm** | Exactly at limit. **Zero margin** |
| 170 mm | **−10 mm** | **EXCEEDS** |
| 180 mm | **−20 mm** | **EXCEEDS** |

### Aggravating factors

- The 160 mm figure is a **no-load** figure. ⚠️ **No derated figure under payload
  is published.**
- Descent margin is normally **smaller** than ascent in legged robots — impact
  loading, lower traction at ground contact. No figure is published.
- ⚠️ **At low battery the robot crouches within 10 minutes.** *A crouch on a
  staircase is a fall.* On stairs, the 10-minute warning is the entire reaction
  budget.

---

## Physical survey required before any conclusion

**None of this has been measured.**

| ID | Measurement | Why |
|---|---|---|
| `SH-STAIR-001` | **Riser height, tread depth, step count — every staircase, every candidate route** | ⚠️ **May end the question outright** |
| `SH-STAIR-002` | Clear width, accounting for handrail projection | Robot is 310 mm wide but needs swing room for leg placement |
| `SH-STAIR-003` | Landing space at each floor | Turning on a landing is a common failure point |
| `SH-STAIR-004` | Surface material and nosing condition | ⚠️ Polished terrazzo is the worst case; the manual names slip as a failure mode |
| `SH-STAIR-005` | Edge contrast | Low contrast impedes perception |
| `SH-STAIR-006` | Lighting | Degrades all sensing |
| `SH-STAIR-007` | Pedestrian traffic and timing | People on stairs is the dominant hazard |
| `SH-STAIR-008` | Which staircases are fire exits | ⚛️ **Absolutely excluded** |
| `SH-STAIR-009` | Handrail clearance | Affects turning and lifting |

⚠️ **If any required riser exceeds 160 mm, stairs are outside the manufacturer
envelope before payload, before supervision, and before wear. No test is needed.**

---

## The safest routes are the ones that avoid stairs entirely

⚠️ **The preferred approach is not to solve the stair problem but to avoid it.**

| Option | Action |
|---|---|
| **Single-floor scope** | Restrict the pilot to one floor ✅ *recommended* |
| **Find a single-floor route** | `SH-STAIR-012` — if every needed room is reachable on one floor, stairs never arise |
| **Ramps** | `SH-STAIR-010`, `-011` — a ramp avoids the stair problem entirely |
| **Human-assisted** | `SH-STAIR-019` — may be the permanent design |
| **Ground-floor connectivity** | The C↔B transition is believed to exist at ground and basement level |

⚠️ **If teachers do not need cross-floor delivery, this entire section becomes
moot.** That is why `SH-STAIR-012` and `SH-DEC-002` are P0 questions.

---

## If stairs were ever pursued

### Preconditions — all required

- [ ] **Measured** risers ≤160 mm on every candidate route
- [ ] Written **manufacturer** confirmation that the proposed use is permitted
      (`SH-STAIR-014`)
- [ ] Written **school** permission (`SH-STAIR-013`)
- [ ] Written **insurer** approval
- [ ] Approved supervised test protocol

### Test ladder — advance only on a clean pass

| Stage | Condition | Pass criterion |
|---|---|---|
| T1 | Flat corridor, 10 m, **unloaded** | 3/3 clean |
| T2 | Single step at measured height, **ascent only** | 5/5, no foot trip |
| T3 | **Descent — backwards** | 5/5 |
| T4 | **Descent — forwards** | 5/5 |
| T5 | Full flight, ascent | 3/3 |
| T6 | Full flight, both descents | 3/3 |
| T7 | Repeat ×10 | ≥29/30, **zero** unrecovered falls |
| T8 | **2 kg payload** | Repeat T5–T7 |
| T9 | 4 kg, then 8 kg | Repeat T5–T7 |

⚠️ **Gating capability: forward descent (T4). If it fails, the project does not
proceed to T5.**

⚠️ **Stairs would be descended unloaded only. Always.** No exceptions.

### Test preconditions

- [ ] Non-public staircase, outside school hours
- [ ] Signed off by administration **and** insurer
- [ ] **Two people: one operator on the remote, one spotter with hands free**
- [ ] Remote-control manual mode — **obstacle avoidance OFF**
- [ ] Hard abort reachable by the spotter within 0.5 s
- [ ] **Unloaded.** No payload, no tray, nothing
- [ ] Video from two angles including a full-staircase wide shot
- [ ] Battery above 40% at every stage

### Abort criteria

- Any unrecovered fall
- Any audible foot contact with a riser nose
- Spotter loses sight of any part of the robot
- Any person enters the staircase footprint
- **Battery below 40%** — ⚠️ because a crouch on a staircase is a fall

⚠️ **A test pass proves the envelope on one staircase, on one day, with a trained
operator and a spotter. It does not prove unsupervised operation, which the
manufacturer forbids. Write that conclusion down before starting, so the test
cannot later be misquoted as product validation.**

---

## Questions

| ID | Question | Priority |
|---|---|---|
| `SH-STAIR-001` | **Riser heights — the decisive measurement** | **P0** |
| `SH-STAIR-002` | Clear widths | **P0** |
| `SH-STAIR-012` | **Is there a single-floor route to every required destination?** | **P0** |
| `SH-STAIR-013` | Does school policy permit stair operation? | **P0** |
| `SH-STAIR-014` | Does manufacturer guidance permit the proposed use? | **P0** |
| `SH-STAIR-015` | Is a stair gait reachable from the approved SDK? | P1 |
| `SH-STAIR-016` | What approval is required before any stair test? | **P0** |
| `SH-STAIR-019` | Is a human-assisted alternative available? | P1 |
| `SH-STAIR-020` | **Confirm stairs are excluded from the pilot** | **P0** |