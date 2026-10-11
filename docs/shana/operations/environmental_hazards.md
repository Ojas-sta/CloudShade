# Environmental Hazards

⚠️ **`DESIGN_ONLY` — no environmental monitoring exists. School-specific
conditions have never been observed.**

---

## 1. 🔴 Status

> Nothing here has been measured, sensed, or tested. A school is an
> **uncontrolled environment** — rain tracked indoors, spilled drinks, cleaning
> chemicals, crowding, and glare are all plausible and none has been surveyed.

---

## 2. Hazard categories

| # | Hazard | Effect | Proposed response | Status |
|---|---|---|---|---|
| 2.1 | **Wet floor** | Reduced traction; slip risk to people and robot | Slow, increase distance, report | `DESIGN_ONLY` |
| 2.2 | **Cleaning chemicals** | Corrosion; damage to sensors | Wipe-down protocol; avoid mopped zones | `DESIGN_ONLY` |
| 2.3 | **Glare / bright light** | Depth-camera degradation | Reduce speed; re-localise | `DESIGN_ONLY` |
| 2.4 | **Low light** | Degraded perception | Slow or refuse to proceed | `DESIGN_ONLY` |
| 2.5 | **Heat** | Battery and thermal limits | Thermal policy; rest periods | `DESIGN_ONLY` |
| 2.6 | **Rain tracked indoors** | Same as wet floor | Avoid entry points | `DESIGN_ONLY` |
| 2.7 | **Clutter / bags / bins** | Obstacles changing without warning | Slower speed; stop on uncertainty | `DESIGN_ONLY` |
| 2.8 | **Doorways** | Pinch and collision risk | Slow through doorways; yield | `DESIGN_ONLY` |
| 2.9 | **Crowding** | High density near people | Speed limits; hold position | `DESIGN_ONLY` |
| 2.10 | **Noise** | No effect on sensors; affects people around it | — | — |

⚠️ **Environmental conditions in this specific school are entirely unknown.** No
visit has occurred.

---

## 3. 🔴 Fail-closed principle

> **When conditions are uncertain, the robot must slow, stop, or refuse — never
> proceed on assumption.**

| Condition | Response |
|---|---|
| Visibility degraded | Slow, then hold |
| Traction questionable | Reduce speed and increase following distance |
| Chemical hazard suspected | **Avoid the zone entirely** |
| Heat above limit | Return / rest |
| Route conditions unknown | 🔴 **Do not enter** |

⚠️ This mirrors Invariant 4.7 — an unknown route is never traversed.

---

## 4. ⚠️ Unevidenced assumptions

| # | Assumption | Risk if false |
|---|---|---|
| 4.1 | Indoor temperature suits the robot | Summer shutdowns |
| 4.2 | Corridors are wide enough | 🔴 **Cannot traverse at all** |
| 4.3 | Flooring is navigable | Traction and sensor issues |
| 4.4 | Lighting is adequate | Perception failures |
| 4.5 | Chemicals present are non-corrosive to sensors | Equipment damage |
| 4.6 | Cleaners will not obstruct routes | Blocked paths |

⚠️ **30 of 34 project assumptions are unverified.** These six are among the most
operationally significant.

---

## 5. 🔴 Required before operation

| # | Requirement | Who |
|---|---|---|
| 5.1 | Environmental survey during school hours | SHANA + school |
| 5.2 | Seasonal assessment — a hot-month trial | School |
| 5.3 | Cleaning schedule and chemical inventory | School facilities |
| 5.4 | Lighting measurement at corridors and doorways | Survey |
| 5.5 | Fire-route and evacuation path identification | School |
| 5.6 | Confirmation that floors are navigable | Survey |

---

## 6. Related

- [`blocked_corridors.md`](blocked_corridors.md) — temporary obstructions
- [`../safety/hazard_analysis.md`](../safety/hazard_analysis.md) — formal hazard register
- [`../safety/operating_restrictions.md`](../safety/operating_restrictions.md) — 57 restrictions
- [`../feasibility/assumptions.md`](../feasibility/assumptions.md) — unverified assumptions