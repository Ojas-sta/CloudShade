# ADR-007: Corridor state is human-declared, not sensed

**Status:** `Accepted`
**Date:** 2026-10-10
**Decides:** The source of truth for whether a corridor is passable
**Reversal condition:** Reversed if a local, privacy-compliant sensing method
proves reliable enough to be a **supplement** — never a sole source. Human
declaration remains the authority. A sensing feed may add advisory information; it
may not be allowed to declare a corridor passable when no human has.
**Evidence:** `docs/shana/operations/congestion_aware_routing.md` ·
`docs/shana/operations/blocked_corridors.md` · `docs/shana/operations/environmental_hazards.md`

---

## Context

Route safety requires knowing which corridors are passable. Four states are
defined: `CLEAR`, `BUSY`, `BLOCKED`, `UNKNOWN`.

The obvious design is automatic detection — cameras, the school's existing CCTV,
or the building's own data. Each has a problem:

| Source | Problem |
|---|---|
| Robot cameras | Works, but only where the robot is; a closed corridor behind you is not seen |
| 🔴 School CCTV | **Deprecated on privacy grounds** — see ADR-010 |
| Building/timetable data | 🔴 A network peer in the safety input path |

---

## Decision

🔴 **Corridor state is declared by a human. It is never inferred from a network
peer.**

| # | Rule |
|---|---|
| 1 | A corridor is `CLEAR`/`BUSY`/`BLOCKED` **only when declared by an authorised person** |
| 2 | 🔴 **`UNKNOWN` is the default and is untraversable.** Fail closed |
| 3 | 🔴 **A CloudLight occupancy feed may inform a display, never route safety** |
| 4 | The robot never self-clears a blockage |

---

## Rationale

**Why fail-closed is non-negotiable.** `AGENTS.md` §L.5: *"A blocked or unknown
route is never traversed."* If `UNKNOWN` were traversable, every undeclared
corridor would be assumed safe — which is the opposite of the invariant.

**Why rule 3 is the safety-critical one.** SHANA's route safety must rest on
**locally sensed and locally declared** state. A network peer in that input path
means the safety decision depends on a system SHANA does not control, cannot
audit, and may lose — and `AGENTS.md` §L.6 requires loss of communications to
produce a defined safe behaviour, not a best-effort continuation. ⚠️ **This is the
substance behind `CONF-09`.**

**Why this is an accepted operational cost.** The system depends on staff
discipline. ⚠️ **That dependency has never been discussed with any staff member.**
It must be presented honestly: this is a system whose real dependency is human
attention.

---

## Consequences

**Accepted** — no surveillance of students; no network peer in the safety path;
privacy preserved by construction.

**Rejected** — CCTV occupancy (privacy); any automatic `CLEAR` declaration;
CloudLight-derived occupancy as a route-safety input.

⚠️ **Consequence to state plainly to the school:** a system that cannot move until
a human declares it safe is **not autonomous in the way the proposal might
imply**.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| CCTV occupancy | 🔴 Deprecated on privacy grounds (ADR-010). See `infrastructure/cctv_feasibility.md`. |
| Robot camera occupancy | 🔴 Only detects what it can see; cannot detect a closure behind it. Insufficient as a sole source. |
| CloudLight occupancy | 🔴 Network peer in the safety path. See rule 3. |
| Full automation | 🔴 Requires identifying or tracking people to be useful. Prohibited (`AGENTS.md` §L.9). |

---

## Related

- [`../../operations/congestion_aware_routing.md`](../../operations/congestion_aware_routing.md)
- [`../../operations/blocked_corridors.md`](../../operations/blocked_corridors.md)
- [`ADR-009`](ADR-009-manual-occupancy-before-sensing.md)
- [`ADR-010`](ADR-010-no-raw-video-retention.md)
- `AGENTS.md` §L.5, §L.6, §L.9