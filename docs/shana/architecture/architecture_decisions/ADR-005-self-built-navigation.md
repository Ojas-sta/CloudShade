# ADR-005: Self-built navigation rather than vendor SLAM

**Status:** `Provisional`
**Date:** 2026-10-10
**Decides:** Whether the navigation stack is built in-house or uses the vendor's
`slam_operate` service
**Reversal condition:** Reversed if `ROB-01` establishes that the school already
owns an expansion dock **and** the vendor-issued YSN is obtainable. ⚠️ Even then,
Route B is expected to remain preferable — see "Alternatives".
**Evidence:** `docs/shana/feasibility/decision_log.md` `D-009` ·
`docs/shana/hardware/sdk_and_interfaces.md`

---

## Context

Two routes to navigation capability:

| Route | Description |
|---|---|
| **A — Vendor SLAM** | Use `slam_operate` |
| **B — Self-built** | Build navigation on the robot's existing perception stack |

---

## Decision

**Route B — self-built navigation — is primary. Route A is not used.**

⚠️ **This is `Provisional`. It depends on `ROB-01`, which is unanswered.**

---

## Evidence

| Fact | Class | Source |
|---|---|---|
| `slam_operate` requires EDU **and** an expansion dock **and** separately purchased MID-360/XT16 **and** a Unitree-issued YSN | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | `sdk_and_interfaces.md` |
| `slam_operate` cannot run inside a ROS graph | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | same |
| The built-in L2 already publishes point clouds on DDS in a ROS 2 `PointCloud2`-compatible type | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | same |
| 🔴 Route B needs **no purchase** | `VERIFIED` | no purchase required |
| Whether the school owns an expansion dock | `UNKNOWN` | `ROB-01` |

---

## Rationale

**Why the YSN dependency is decisive.** A vendor-issued serial binding the
navigation stack to a specific hardware purchase makes the project's viability
depend on a procurement decision nobody has made. A route that requires no purchase
removes that dependency entirely.

**Why the existing L2 output matters.** If the robot already publishes
ROS-compatible point clouds, Route B is an engineering problem rather than a
hardware problem — a very different risk profile.

**Why this is nonetheless provisional.** Route B is **more engineering**. It is
chosen for dependency reasons, not because it is technically superior. If the
hardware turns out to be available, the comparison must be redone.

⚠️ **Recorded limitation:** choosing self-built navigation raises implementation
risk and it is **unvalidated**. No simulation of Route B exists. See ADR-011.

---

## Consequences

**Accepted** — no procurement dependency; engineering remains ours; ROS-compatible
perception data is available.

**Rejected** — Route A as primary; any plan assuming `slam_operate` is available.

---

## Alternatives considered

| Alternative | Why rejected as primary |
|---|---|
| **Route A — vendor SLAM** | Requires four separate conditions including a Unitree-issued YSN. None confirmed. |
| Both routes in parallel | ⚠️ Doubles the engineering surface for a project with no implementation. Deferred until Route B is proven in simulation. |

⚠️ **If `ROB-01` reveals the expansion dock is already owned, Route A must be
re-evaluated before Route B is built.** Route B's advantage would then be reduced
to the YSN dependency and ROS compatibility, and the implementation risk may not
justify it.

---

## Related

- [`../../hardware/sdk_and_interfaces.md`](../../hardware/sdk_and_interfaces.md)
- [`ADR-011`](ADR-011-simulation-before-hardware.md)
- [`../../feasibility/decision_log.md`](../../feasibility/decision_log.md) — `D-009`
- [`../../handoff/unresolved_dependencies.md`](../../handoff/unresolved_dependencies.md) — `ROB-01`