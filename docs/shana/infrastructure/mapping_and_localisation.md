# Mapping and Localisation

⚠️ **`DESIGN_ONLY` and `BLOCKED`. No map exists. No survey has been conducted.**

---

## 1. 🔴 Status

| Question | Answer |
|---|---|
| Has the school been surveyed? | ❌ **No** |
| Does a map exist? | ❌ **No** |
| Has any mapping been run? | ❌ **No — never connected to hardware** |
| Is floor identity solved? | ❌ **No — no sensor exists** |
| Can this be started now? | ⚠️ **Protocol can be written; execution needs access** |

> **A mapping document without a map is a plan for mapping. That is what this
> is.**

---

## 2. Map model — proposed

⚠️ Never validated against a real building.

| Layer | Contents |
|---|---|
| Topology | Corridors, junctions, doorways, dead ends |
| Metrics | Widths, lengths, turning radii |
| Destinations | Named delivery points |
| Vertical | 🔴 **Single floor only** — see §4 |
| Markers | Floor-identity markers |
| Prohibited zones | Stairs, lifts, fire routes, restricted areas |

⚠️ **Building layouts are sensitive.** A committed floor plan may constitute a
security disclosure. Maps must not enter Git without school approval.

---

## 3. Mapping approach

| Step | Approach | Status |
|---|---|---|
| 3.1 | Manual survey as the **primary** source | ✅ Protocol written — see [`school_route_survey.md`](school_route_survey.md) |
| 3.2 | Robot-assisted mapping as **secondary** | `DESIGN_ONLY` — needs hardware |
| 3.3 | Reconcile the two | `DESIGN_ONLY` |

⚠️ **The manual survey is primary, deliberately.** Robot-assisted mapping cannot
run without hardware access, and a hand-measured map is more trustworthy than a
map built by an unvalidated SLAM stack on a first run.

---

## 4. 🔴 Floor identity — the unsolved problem

Constraint **C-2**: no altitude, no pressure, no barometer.

> **The robot cannot determine which floor it is on.**

| Option | Status |
|---|---|
| Onboard sensing | ⛔ **Impossible** |
| ArUco floor markers (ADR-006) | ⚠️ **Provisional — never validated** |
| Network-derived floor | ⛔ **Rejected** — fails when the network fails |
| Human declaration | ⚠️ Fallback; requires staff |
| Scene recognition | ⚠️ Unvalidated; privacy cost |

⚠️ **Marker-based identity is the only serious candidate and it has never been
tested.** It depends on markers being placed, maintained, and not obstructed —
maintenance is a real operational cost the school must accept.

---

## 5. 🔴 Single-floor conclusion

Constraints **C-2**, **C-3** (stairs removed), and **C-4** (no lift API) together
make multi-floor operation **physically impermissible**:

- Cannot sense the floor → does not know where it is
- Cannot climb stairs → cannot change floor that way
- Cannot use lifts lawfully → cannot change floor that way

> **SHANA must be single-floor. This is not a scoping choice.**

See [`../feasibility/constraints.md`](../feasibility/constraints.md).

---

## 6. 🔴 Survey prerequisites

Nothing can be mapped until these are available:

| # | Requirement | Status |
|---|---|---|
| 6.1 | Physical access to the school | ❌ Not granted |
| 6.2 | Guide from school staff | ❌ Not arranged |
| 6.3 | Floor plan from the school | ❌ Unknown if it exists |
| 6.4 | List of delivery destinations | ❌ Not provided |
| 6.5 | Knowledge of prohibited zones and fire routes | ❌ Not provided |
| 6.6 | Permission to record measurements | ❌ Not granted |

⚠️ **The survey is the single highest-value unblocked-by-permission task.** The
protocol is written; only execution is missing.

---

## 7. Related

- [`school_route_survey.md`](school_route_survey.md) — the protocol
- [`../architecture/navigation_architecture.md`](../architecture/navigation_architecture.md) — navigation design
- [`../feasibility/constraints.md`](../feasibility/constraints.md) — C-2, C-3, C-4
- [`../architecture/route_graph.md`](../architecture/route_graph.md) — route representation