# Navigation Architecture

⚠️ **`DESIGN_ONLY` — no navigation code exists, and navigation cannot begin until
a route survey is conducted.**

---

## 1. The blocker that shapes everything

> 🔴 **The school has never been surveyed.**
>
> There is no map, no corridor topology, no door widths, no turning radii, no
> destination list. Every design below is **conditional on a survey that has not
> happened**.

Navigation design without a surveyed building is guesswork. This document records
the intended approach so it is ready, not because it has been validated.

---

## 2. Approach — self-built, not vendor SLAM

| Decision | Value | Status |
|---|---|---|
| Navigation stack | Self-built (ADR-005) | ⚠️ **Provisional** |
| Vendor SLAM | Not used | Deliberate |
| Rationale | Full control, determinism, and auditability of the safety layer | — |

⚠️ **Choosing a self-built stack over vendor SLAM raises risk.** It has not been
validated. This is recorded as an open assumption, not a settled decision.

---

## 3. Pipeline

```text
  ┌──────────┐   ┌──────────┐   ┌───────────┐   ┌──────────┐
  │ Perceive │──►│ Localise │──►│   Plan    │──►│  Execute │
  └──────────┘   └──────────┘   └───────────┘   └──────────┘
       ▲              ▲             ▲              │
       │              │             │              ▼
  depth camera   floor markers   route graph   🔴 safety
  + state        (C-2 problem)   + corridor    layer ALWAYS
                                   states       in the loop
```

⚠️ **The safety layer sits outside and below the planner. The planner never has
unilateral authority over motion** — C-6 / ADR-001.

---

## 4. 🔴 Localisation — the unsolved problem

Constraint **C-2**: the robot has no altitude, pressure, or barometer sensor. It
**cannot determine which floor it is on**.

| Approach | Status |
|---|---|
| Onboard floor sensing | ⛔ **Impossible — no sensor** |
| Visual floor markers (ArUco) | ⚠️ ADR-006 **provisional, unvalidated** |
| Barometric differential | ⛔ **Impossible — no barometer** |
| Network-derived floor | ⛔ **Rejected** — fails when the network fails |
| Human-declared floor | ⚠️ Fallback only; requires staff action |
| Camera-based scene recognition | ⚠️ Unvalidated; adds privacy risk |

> **This is the hardest unsolved problem in the project.** It is solvable only by
> an external mechanism, and the candidate mechanism has never been tested at
> school scale.

⚠️ If marker-based identity proves unworkable, **single-floor scoping becomes
mandatory rather than preferred** — see
[`../feasibility/constraints.md`](../feasibility/constraints.md).

---

## 5. Planning

| Element | Approach | Status |
|---|---|---|
| Route representation | Route graph over surveyed corridors | `DESIGN_ONLY` |
| Corridor states | 4 states, **human-declared** (`D-031`) | `DESIGN_ONLY` |
| Congestion avoidance | Alternate routing on `BUSY` | `DESIGN_ONLY` |
| Dynamic replanning | On `BLOCKED` | `DESIGN_ONLY` |
| Stairs | ⛔ **Excluded** — C-3 | — |
| Lifts | ⛔ **Excluded** — C-4 | — |
| Unknown route | 🔴 **Never traversed. Fail closed** | Invariant 4.7 |

⚠️ **Occupancy is human-declared, not sensed.** The system depends on staff
discipline with no automated backup. If staff do not declare congestion, routing
degrades silently. This is a known weakness.

---

## 6. Safety layer — always in the loop

| # | Function | Behaviour |
|---|---|---|
| 6.1 | Obstacle detection | Stop before contact |
| 6.2 | Speed limiting | Reduced speed near people |
| 6.3 | Emergency stop | 🔴 Immediate, unconditional, hardware-level |
| 6.4 | Battery reserve | Hold at safe point at threshold |
| 6.5 | Network loss | 🔴 **Degrade safely — never continue blind into uncertainty** |
| 6.6 | Localisation loss | 🔴 **Stop. Never navigate while lost** |

🔴 **6.5 and 6.6 are fail-closed by design.** When confidence drops, motion
stops. Continuing on guesswork around children is not acceptable.

---

## 7. 🔴 What is missing

| # | Missing | Blocks |
|---|---|---|
| 7.1 | 🔴 **Route survey** | Everything |
| 7.2 | 🔴 **Floor identity validation** | Any multi-floor claim |
| 7.3 | **Corridor measurements** | Feasibility of traversal |
| 7.4 | **Destination list** | Task definition |
| 7.5 | **Simulation validation** | All of the above |
| 7.6 | **Stop-distance measurement** | Any safety claim |

---

## 8. 🔴 Status

> **Nothing in this document has been built, tested, or validated.**
>
> Navigation cannot begin until a survey is conducted and Gate 3 passes.
>
> The honest current statement is: *"A navigation approach has been proposed. It
> has never been run anywhere."*