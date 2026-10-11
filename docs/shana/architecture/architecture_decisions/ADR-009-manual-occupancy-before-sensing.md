# ADR-009: Manual occupancy before sensing

**Status:** `Provisional`
**Date:** 2026-10-10
**Decides:** Whether SHANA may accept occupancy information from CloudLight as
an input to route safety
**Reversal condition:** Reversed only if a **local, on-robot** occupancy method is
demonstrated to be privacy-compliant and reliable at school scale. ⚠️ **A
CloudLight-derived occupancy feed would never satisfy this condition**, because
rule 3 places a network peer in the safety path — see "Alternatives".
**Evidence:** `docs/shana/operations/congestion_aware_routing.md` ·
[`ADR-007`](ADR-007-corridor-state-human-declared.md) ·
`docs/integration/unresolved_conflicts.md` `CONF-09`

---

## Context

CloudLight holds timetable and substitution data, and is described in
`cloudlight_integration.md` §2 as owner of "occupancy and scheduling
intelligence". It is therefore reasonable to ask: should CloudLight tell SHANA
which corridors will be busy?

The two positions are compatible today only by accident. SHANA's route safety
assumes SHANA knows the corridor state itself. An occupancy feed would insert a
network peer into that input.

⚠️ **This ADR exists to make the current position explicit rather than accidental.**

---

## Decision

🔴 **Occupancy is never a CloudLight input to SHANA route safety.**

| # | Rule |
|---|---|
| 1 | Corridor state is **human-declared** or **locally sensed** — never peer-derived |
| 2 | CloudLight may **display** occupancy and may use it for **scheduling** |
| 3 | 🔴 **A CloudLight occupancy feed may never declare a corridor passable** |
| 4 | Rule 3 is an invariant, not a preference — it is not relaxable by agreement |

---

## Rationale

**Why this is safety-critical, not architectural tidiness.** `AGENTS.md` §L.6
requires loss of communications to produce a *defined safe behaviour*. If
occupancy is a peer input, then link loss removes a safety input — and the
temptation under schedule pressure is to continue on stale data rather than fail
closed. That is precisely the "best-effort continuation" §L.6 forbids.

**Why rule 4 has no exception.** A peer that can declare a corridor passable can
cause the robot to enter an occupied or closed corridor. The school, not CloudLight,
holds the authority to say a corridor is safe.

**Why CloudLight may still display and schedule.** Timetable data is genuinely
better at predicting *when* a corridor will be busy. Using it to plan the order
of deliveries is safe. Using it to decide *whether to enter* is not.

**Why `Provisional` rather than `Accepted`.** ⚠️ A **local** on-robot occupancy
method — depth-based presence detection, no identification — could supplement
this. That has not been validated. The prohibition on *peer-derived* occupancy is
permanent; the prohibition on *any* automation is not.

---

## Consequences

**Accepted** — no network peer in SHANA's safety inputs; fail-closed behaviour is
preserved under link loss; occupancy data stays useful for scheduling.

**Rejected** — CloudLight as an authority on corridor passability; any design
where a peer decision gates robot motion.

⚠️ **If CloudLight derives occupancy for its own reasons, `D-031` and the
"human-declared" position become silently wrong.** ⚠️ **Note: `D-031` does not
exist in the decision log — see "Known defect" below.**

---

## Known defect in the current record

🔴 **`D-031` is cited in four SHANA documents but does not exist in
`docs/shana/feasibility/decision_log.md`.** The decision log ends at `D-020`.
The citation is a dangling reference introduced when occupancy documents were
written.

| Cited in | Line context |
|---|---|
| `operations/blocked_corridors.md` | "Manual declaration — current basis (`D-031`, provisional)" |
| `architecture/navigation_architecture.md` | "Corridor states — 4 states, human-declared (`D-031`)" |
| `architecture/cloudlight_integration.md` | "currently human-declared — see `D-031`" |
| `architecture/architecture_decisions/README.md` | ADR-009 → `D-031` |

**This ADR is now the authority** for the occupancy position. The `D-031`
references should be repointed here.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| CloudLight supplies occupancy; SHANA treats it as advisory | 🔴 Advisory inputs in safety paths drift into authoritative use. Rejected. |
| CloudLight occupancy, but only when link healthy | 🔴 Still a peer in the safety path. Link health is a network property, not a safety guarantee. |
| Full automation from vision | 🔴 Requires identifying or tracking people. Prohibited (`AGENTS.md` §L.9). |

---

## Related

- [`ADR-007`](ADR-007-corridor-state-human-declared.md) — the operative implementation decision
- [`../../architecture/cloudlight_integration.md`](../../architecture/cloudlight_integration.md)
- [`../../operations/congestion_aware_routing.md`](../../operations/congestion_aware_routing.md)
- `docs/integration/unresolved_conflicts.md` — `CONF-09`
- `AGENTS.md` §L.6, §L.9