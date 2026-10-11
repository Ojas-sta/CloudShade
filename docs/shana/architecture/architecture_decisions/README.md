# SHANA — Architecture Decision Records

An ADR records a decision, the evidence behind it, and — critically — **the
condition under which it would be reversed**.

⚠️ **A decision without a recorded reversal condition becomes a decision nobody
dares revisit.** Every ADR here carries one.

---

## Index

### Written ADRs

| ADR | Title | Status | Reversal condition |
|---|---|---|---|
| [ADR-002](ADR-002-single-floor-first.md) | Single-floor first; staged deployment | **Accepted** | Single-floor route to all destinations proven **and** cross-floor demand confirmed **and** vertical transit permitted |
| [ADR-003](ADR-003-stairs-and-lifts-excluded.md) | Stairs and lifts excluded from the pilot | **Provisional** | Per-modality conditions — see ADR-003 |

### Decisions recorded in the decision log but **not yet written as an ADR**

⚠️ **These are real decisions with recorded reversal conditions.** They are
currently recorded in
[`../../feasibility/decision_log.md`](../../feasibility/decision_log.md) and are
indexed here so they are not lost. **An ADR is owed for each.**

| Would be | Decision | Log ID | Status | Where recorded |
|---|---|---|---|---|
| ADR-001 | No AI model may reach the robot | `D-005` | Accepted | `decision_log.md` §1 |
| ADR-004 | Wired Ethernet for the robot link | `D-003` | Accepted | `decision_log.md` §1 |
| ADR-008 | No lift modification, ever | `D-015` | Accepted | `decision_log.md` §1 |
| ADR-010 | No raw video retention | — | Accepted | `../../safety/student_privacy.md` |
| ADR-011 | Simulation before hardware | `D-010` | Accepted | `decision_log.md` §1 |
| ADR-005 | Self-built navigation over vendor SLAM | `D-009` | Provisional | `decision_log.md` §1 |
| ADR-006 | ArUco markers for floor identity | `D-018` | Provisional | `decision_log.md` §1 |
| ADR-007 | Corridor state is human-declared | — | Provisional | `../../operations/congestion_aware_routing.md` |
| ADR-009 | Manual occupancy before sensing | `D-031` | Provisional | `decision_log.md` §2 |
| ADR-012 | Radio telemetry deferred | `D-017` | Provisional | `decision_log.md` §1 |

⚠️ **This gap is recorded rather than hidden.** The handoff agent and any future
contributor should treat writing these ADRs as outstanding work — see
`../../handoff/implementation_inventory.md`.

---

## Status values

| Status | Meaning |
|---|---|
| `Proposed` | Drafted, not agreed |
| `Accepted` | Agreed; may be implemented |
| `Provisional` | Agreed subject to a stated condition |
| `Deprecated` | No longer applies |
| `Superseded` | Replaced by another ADR |

⚠️ **Most of these decisions cannot be implemented yet**, because the gates have
not been passed. An accepted decision is **not** a shipped feature.

---

## Evidence classes used throughout

| Label | Meaning |
|---|---|
| `VERIFIED` | Confirmed in official manufacturer documentation or source code, quoted |
| `DOCUMENTED_NOT_HARDWARE_VERIFIED` | Vendor-documented; not tested on our equipment |
| `PLAUSIBLE_REQUIRES_TESTING` | Technically plausible; feasibility not established |
| `UNKNOWN` | Insufficient information |
| `NOT_PERMITTED` | Prohibited by law, manufacturer terms, or safety judgement |

---

## Template for new ADRs

Keep it short. A long ADR is usually an architecture in disguise.

```markdown
# ADR-NNN: Title

**Status:** Proposed | Accepted | Provisional
**Date:** YYYY-MM-DD
**Decides:** What this settles
**Reversal condition:** What would cause this to be revisited
**Evidence:** Source IDs from ../../research/source_register.md

## Context
What forces were at play? What made a decision necessary?

## Decision
What was decided, in one unambiguous statement.

## Evidence
Source ID, class, and what it actually says.

## Consequences
What this commits us to. What becomes easier. What becomes harder.

## Alternatives considered
What else was on the table, and why not.

## What this decision does NOT settle
The questions that remain open even after this decision.
```