# ADR-002: Single-floor first; staged deployment

**Status:** Accepted
**Date:** 2026-10-10
**Decides:** The deployment progression and the scope of the first pilot
**Reversal condition:** A verified single-floor route to every required
destination is demonstrated to exist AND cross-floor demand is confirmed by
teachers AND stairs or lifts are formally permitted

---

## Context

The project vision spans three blocks and three to four floor levels. The
robot, however, **has no sensor capable of determining which floor of a building
it is on.**

This was established by enumerating every accessor across every Go2 message
field. There is no altitude, no pressure, no barometer, and no floor field
anywhere in the platform. `SportModeState.position[3]` is dead-reckoned
odometry. `body_height` is the commanded stance height above ground. The
`HeightMap` message origin is `float32[2]` — XY only, with no Z datum.

The built-in LiDAR has a 360° × 96° field of view, so vertical geometry is
partially observable, but partial observation is not an absolute floor
reference.

---

## Decision

SHANA will deploy in four stages, and **Stage 1 is single-floor**.

| Stage | Scope | Depends on |
|---|---|---|
| 0 — Observe | Single floor, no actuation, no autonomy | Gates 0, 1 |
| 1 — Single floor | One floor, human-docked, supervisor present | Gates 0–2, 5, 6, 8, 9 |
| 2 — Vertical | Multi-floor, **human-assisted transit only** | Stage 1 + floor identity + lift permission |
| 3 — Multi-block | Cross-block | Stage 2 + verified routes |

**Each stage must independently earn its own permission.** No stage depends on an
unverified capability of a later one.

---

## Evidence

| Claim | Class | Source |
|---|---|---|
| Only EDU-and-above models may be programmed | VERIFIED | `https://support.unitree.com/home/en/developer/faq` |
| Development is wired-only | VERIFIED | same |
| No altitude/pressure/barometer field exists | VERIFIED (negative) | `unitree_sdk2` Go2 message definitions |
| Built-in LiDAR is 360° × 96° | VERIFIED | `https://support.unitree.com/home/en/developer/about_Go2` |
| Corridor layouts immediately outside lifts are near-identical across floors | PEER-REVIEWED | McAree et al. (Sheffield) |

⚠️ **There is no published body of work on quadruped multi-floor navigation.**
This is not a solved problem we are choosing to defer; it is an unsolved problem.

---

## Consequences

**Easier:**
- Stage 1 avoids the hardest unsolved problem in the project
- Floor identification complexity disappears entirely
- Every verification result feeds a system that can actually be built

**Harder:**
- Reach is limited to one floor initially
- If teachers genuinely need cross-floor delivery, Stage 1 does not serve them
- We must actively ask whether they do — which is why `SH-DEC-002` is a P0

---

## Alternatives considered

**Build for multi-floor from the start.**
Rejected. It would require floor-identification infrastructure before any
measurement, and the most likely outcome would be a system that cannot reliably
tell which floor it is on — the silent failure described in
`docs/shana/architecture/route_graph.md`.

**Use a human to tell the robot which floor it is on, with no sensor.**
Rejected. Better than guessing, but it depends entirely on a human noticing and
acting correctly, and provides no verification.

---

## What this decision does NOT settle

- Whether multi-floor is ever viable. It may require hardware Unitree does not
  offer.
- What the actual demand for cross-floor delivery is. `SH-DEC-002`,
  `SH-TIME-005`, `SH-DEL-001` must answer it.
- Whether the B↔A route exists as believed. `SH-STAIR-012`, `FAC-02`.