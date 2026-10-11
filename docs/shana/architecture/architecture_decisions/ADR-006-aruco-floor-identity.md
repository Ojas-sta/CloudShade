# ADR-006: ArUco markers encode floor identity

**Status:** `Provisional`
**Date:** 2026-10-10
**Decides:** How the robot determines which floor it is on
**Reversal condition:** Reversed if marker-based floor identity proves unreliable
at school scale — see below. ⚠️ If reversed **and** single-floor scoping
(ADR-002) is not adopted, the project has **no** floor-identity mechanism and
multi-floor operation becomes impossible.
**Evidence:** `docs/shana/feasibility/decision_log.md` `D-018`, `D-019` ·
`docs/shana/hardware/checkpoint_markers.md` ·
`docs/shana/feasibility/constraints.md` `C-2`

---

## Context

🔴 **The robot cannot determine which floor it is on.** There is no altitude,
pressure, or barometer sensor (`constraints.md` `C-2`). This is a verified
hardware absence, not a software gap.

Stairs are excluded (ADR-003) and no lift API exists (`ADR-003`), so if lifts are
ever reintroduced, the robot must know its floor. More immediately: **if a human
carries the robot up a flight of stairs**, it must not silently believe it is
still on the floor it started from.

---

## Decision

⚠️ **`Provisional`:** floor identity is encoded by **four ArUco markers, one per
floor**, detected by the robot's existing camera pipeline.

| # | Rule |
|---|---|
| 1 | 🔴 **Every floor transition requires mandatory human confirmation**, regardless of marker detection |
| 2 | Markers make the mechanism work; **human confirmation makes the residual risk acceptable** |
| 3 | 🔴 **Any manual relocation event invalidates the robot's pose** (`D-019`) |

---

## Evidence

| Fact | Class | Source |
|---|---|---|
| 🔴 No altitude, pressure or barometer sensor exists | `VERIFIED` | `go2_edu_inventory.md`; `constraints.md` `C-2` |
| The charging pile mechanism is **already ArUco-based** | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | `hardware/checkpoint_markers.md` |
| Therefore the marker-detection pipeline is **proven on this robot** | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | same |
| Marker approach pending `ADM-04`, `TEC-08` | `UNKNOWN` | `decision_log.md` `D-018` |
| 🔴 Marker approach **never validated at school scale** | `VERIFIED` | repository audit 2026-10-11 |

---

## Rationale

**Why the charging-pile precedent matters.** The strongest argument for this
approach is that the sensor pipeline is already proven on this exact robot. One
investment yields two capabilities — floor identity and docking.

**Why human confirmation is mandatory, not advisory.** Markers fail silently: a
marker can be removed, obscured, or mis-detected, and the robot will plan
confidently against the wrong floor. Rule 1 converts a silent failure into a
visible one.

**Why rule 3 exists — the most dangerous failure in the system.** If a human
carries the robot up stairs, scan-matching **succeeds against both floors' maps**.
The robot plans correctly against the wrong map and **raises no error**. Any
manual handling therefore sets `pose_valid = false`, forcing re-localisation
before any new task.

⚠️ **Marker maintenance is a real operational cost**: someone must keep markers
present, clean and unobstructed. That is a commitment the school must make, not a
technical detail.

---

## Consequences

**Accepted** — a floor-identity mechanism that requires no purchase and reuses a
proven pipeline.

**Rejected** — on-board floor sensing (impossible, `C-2`); network-derived floor
(🔴 fails when the network fails — a network peer must not enter the safety path);
camera-only scene recognition (adds privacy cost, unvalidated).

⚠️ **Single-floor operation makes this ADR unnecessary.** Under ADR-002 the
marker scheme is belt-and-braces, not load-bearing.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Barometric differential | ⛔ No barometer. Impossible. |
| Network-reported floor | 🔴 A network peer in the safety path; fails closed in the wrong direction. |
| Vision-based scene recognition | Unvalidated, privacy cost, more failure modes. |
| Manual declaration only | ⚠️ Weak — a human error is unrecoverable and undetected. Retained as a **required complement** (rule 1), rejected as a sole mechanism. |

---

## Follow-up tasks

| # | Task | Owner | Status |
|---|---|---|---|
| 1 | Measure whether markers are permitted and maintainable at school scale | School | ⛔ `REQUESTED` — `ADM-04` |
| 2 | Validate marker detection in simulation | Agent 1 | ⛔ blocked on ADR-011 PoC |
| 3 | Implement relocation-event detection | Agent 1 | ⛔ blocked on 1, 2 |

---

## Related

- [`../../hardware/checkpoint_markers.md`](../../hardware/checkpoint_markers.md)
- [`../../feasibility/constraints.md`](../../feasibility/constraints.md) — `C-2`
- [`ADR-002`](ADR-002-single-floor-first.md) · [`ADR-003`](ADR-003-stairs-and-lifts-excluded.md)
- [`../../feasibility/decision_log.md`](../../feasibility/decision_log.md) — `D-018`, `D-019`