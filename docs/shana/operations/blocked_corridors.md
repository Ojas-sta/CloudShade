# Blocked Corridors

⚠️ **`DESIGN_ONLY` — no blockage detection exists. Occupancy is human-declared.**

---

## 1. 🔴 Core principle

> ## Fail closed. Never traverse an unknown or unverified route.
>
> Invariant 4.7 is absolute. If SHANA does not know a corridor is passable, it
> does not go there — regardless of how much delay that causes.

⚠️ **A missed delivery is an inconvenience. A collision with a child is not
recoverable.** The asymmetry governs every decision here.

---

## 2. Corridor states

| State | Meaning | May SHANA traverse? |
|---|---|---|
| `CLEAR` | Declared passable | ✅ Yes |
| `BUSY` | Occupied but passable | ⚠️ Slowed, alerted |
| `BLOCKED` | Impassable | ⛔ **No** |
| `UNKNOWN` | 🔴 **Not declared** | ⛔ **No — fail closed** |

⚠️ **`UNKNOWN` is the most common state in practice.** At start-up, before any
declaration, and whenever a declaration is missing, the corridor is `UNKNOWN` and
therefore untraversable.

> **This means SHANA cannot move anywhere until staff declare the routes.** That
> is a genuine operational dependency, not a formality.

---

## 3. 🔴 Occupancy is human-declared

| Attribute | Status |
|---|---|
| Automated blockage detection | 🔴 **Not implemented** |
| CCTV-based detection | ⛔ **Deprecated — privacy grounds** |
| Camera-based detection | ⚠️ Proposed; adds privacy cost |
| Manual declaration | ⚠️ **Current basis** ([ADR-009](../architecture/architecture_decisions/ADR-009-manual-occupancy-before-sensing.md), provisional) |

⚠️ **The system depends on staff discipline with no automatic backup.** If
declarations are not made, the robot simply stops. This is safe but
unproductive — and it is a real risk to project acceptance.

⚠️ **Staff compliance is an organisational dependency, not a technical one.** It
has never been discussed with any staff member.

---

## 4. Blocking causes

| # | Cause | Typical | Clearance |
|---|---|---|---|
| 4.1 | Cleaning in progress | Daily | Facilities confirmation |
| 4.2 | Furniture moved | Occasional | Manual |
| 4.3 | Construction or maintenance | Periodic | Facilities |
| 4.4 | Fire evacuation / drill | Periodic | 🔴 **Always treat as blocking** |
| 4.5 | Deliveries or events | Occasional | Manual |
| 4.6 | Wet floor | Daily | Facilities |
| 4.7 | Congestion | Daily | Automatic via declaration |
| 4.8 | Robot fault on another unit | Rare | Technical |

---

## 5. 🔴 Absolute prohibitions

| # | Prohibition | Rationale |
|---|---|---|
| 5.1 | 🔴 **Never traverse a fire route when blocked** | Life safety outranks delivery |
| 5.2 | 🔴 **Never enter a `BLOCKED` corridor to "check"** | The corridor is unsafe by definition |
| 5.3 | 🔴 **Never improvise a route around a blockage** | Unverified routes are forbidden |
| 5.4 | 🔴 **Never continue while localisation is degraded** | Stop instead |
| 5.5 | ⚠️ **Never self-clear a blockage** | Requires human judgement |

⚠️ **5.3 deserves emphasis.** An intuitive design would route around obstacles.
Under Invariant 4.7, that is exactly what is forbidden — an alternative path
through an unverified corridor is more dangerous than waiting.

---

## 6. 🔴 Operational dependency

| Question | Answer |
|---|---|
| Who declares corridors? | ⚠️ **Unassigned** |
| How is declaration made? | ⚠️ **No interface exists** |
| How quickly must it happen? | `UNKNOWN` |
| What if no one declares? | 🔴 **Robot does not move** |
| Has any staff member been consulted? | ❌ **No** |

> **A system that cannot move until a human declares it safe is a system whose
> real dependency is human attention, not autonomy.** This must be presented to
> the school honestly, not as "autonomous delivery."

---

## 7. Related

- [`congestion_aware_routing.md`](congestion_aware_routing.md) — routing under `BUSY`
- [`environmental_hazards.md`](environmental_hazards.md) — wet floor and chemicals
- [`../architecture/route_graph.md`](../architecture/route_graph.md) — route representation
- [`../architecture/cloudlight_integration.md`](../architecture/cloudlight_integration.md) — Invariant 4.7