# SHANA — Architecture Decision Records

An ADR records a decision, the evidence behind it, and — critically — **the
condition under which it would be reversed**.

⚠️ **A decision without a recorded reversal condition becomes a decision nobody
dares revisit.** Every ADR here carries one.

---

## Index

| ADR | Title | Status | Reversal condition |
|---|---|---|---|
| [ADR-001](ADR-001-no-ai-reaches-the-robot.md) | 🔴 No AI model output ever reaches the robot | **Accepted** | Not reversible by engineering — governance decision |
| [ADR-002](ADR-002-single-floor-first.md) | Single-floor first; staged deployment | **Accepted** | Single-floor routes proven + cross-floor demand confirmed + vertical transit permitted |
| [ADR-003](ADR-003-stairs-and-lifts-excluded.md) | Stairs and lifts excluded from the pilot | **Provisional** | Per-modality — see ADR-003 |
| [ADR-004](ADR-004-wired-ethernet-robot-link.md) | 🔴 Robot-to-server link is wired Ethernet | **Accepted** | School sanctions a wireless control path + latency budget met + fail-safe preserved |
| [ADR-005](ADR-005-self-built-navigation.md) | Self-built navigation rather than vendor SLAM | **Provisional** | `ROB-01` shows expansion dock owned **and** YSN obtainable |
| [ADR-006](ADR-006-aruco-floor-identity.md) | ArUco markers encode floor identity | **Provisional** | Markers unreliable at school scale |
| [ADR-007](ADR-007-corridor-state-human-declared.md) | Corridor state is human-declared, not sensed | **Accepted** | Local privacy-compliant sensing proves reliable as a *supplement* |
| [ADR-008](ADR-008-no-lift-modification.md) | 🔴 No lift modification, ever | **Accepted** | 🔴 **None** — legal constraint, not an engineering preference |
| [ADR-009](ADR-009-manual-occupancy-before-sensing.md) | Manual occupancy before sensing | **Provisional** | Local on-robot occupancy validated. 🔴 A peer feed would **never** satisfy this |
| [ADR-010](ADR-010-no-raw-video-retention.md) | 🔴 No raw video ever leaves the robot | **Accepted** | Not reversible by engineering — governance decision |
| [ADR-011](ADR-011-simulation-before-hardware.md) | Simulation before hardware | **Accepted** | Simulator unrepresentative of platform dynamics |
| [ADR-012](ADR-012-radio-telemetry-deferred.md) | Radio telemetry deferred | **Provisional** | `NET-05` names a specific uncovered room |

### ✅ The owed-ADR gap is CLOSED (2026-10-11)

Ten ADRs were previously indexed as *"recorded in the decision log but never
written as files"*. **All ten now exist as files** and resolve.

⚠️ **An ADR index entry without a file behind it is a defect** — it makes a
reader believe a decision has been documented when it has not
(`AGENTS.md` §M, `CONF-04`). Every row above now points to a real file.

### 🔴 Cited across the subsystem boundary

CloudLight's safety argument rests on three of these. They are cited as
*"not CloudLight's to relax"*:

| ADR | Cited by CloudLight as |
|---|---|
| **ADR-001** — no AI reaches the robot | An accepted architecture decision CloudLight cannot relax |
| **ADR-010** — no raw video retention | Privacy invariant |
| **ADR-011** — simulation before hardware | Development ordering |

⚠️ **Before this ADR set, all three citations resolved to nothing.** A reader
following them had to decide for themselves whether the safety invariants held.
They do. ⚠️ **All three remain `DESIGN_ONLY` — no code implements them.**

---

## Status values

| Status | Meaning |
|---|---|
| `Proposed` | Drafted, not agreed |
| `Accepted` | Agreed; may be implemented |
| `Provisional` | Agreed subject to a stated condition |
| `Deprecated` | No longer applies |
| `Superseded` | Replaced by another ADR |

⚠️ **An `Accepted` decision is not a shipped feature.** ⚠️ **None of these may be
implemented yet** — the go/no-go gates have not been passed.

---

## Evidence classes used throughout

| Label | Meaning |
|---|---|
| `VERIFIED` | Confirmed in official manufacturer documentation or source code, quoted |
| `DOCUMENTED_NOT_HARDWARE_VERIFIED` | Vendor-documented; not tested on our equipment |
| `PLAUSIBLE_REQUIRES_TESTING` | Technically plausible; feasibility not established |
| `UNKNOWN` | Insufficient information |
| `NOT FOUND` | 🔴 A valid and valuable finding — the source could not be accessed |
| `NOT_PERMITTED` | Prohibited by law, manufacturer terms, or safety judgement |

---

## Source decisions

Each ADR derives from a decision already made in
[`../../feasibility/decision_log.md`](../../feasibility/decision_log.md).

| ADR | Decision |
|---|---|
| ADR-001 | `D-005` — LLM behind a deterministic validator |
| ADR-004 | `D-003` — wired Ethernet over Cyclone DDS |
| ADR-005 | `D-009` — self-built navigation |
| ADR-006 | `D-018` — ArUco floor identity; `D-019` — relocation invalidates pose |
| ADR-008 | `D-015` — no lift modification; `D-016` — no autonomous lift assumed |
| ADR-011 | `D-010` — simulation-first |
| ADR-012 | `D-017` — LoRa deferred |

ADR-002 and ADR-003 predate the decision log.

⚠️ **ADR-007, ADR-009 and ADR-010 have no numbered decision in the log.** They
record positions stated directly in the operations, integration and privacy
documents. See the defect note in
[ADR-009](ADR-009-manual-occupancy-before-sensing.md).

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

## Rationale
Why, in terms that would convince a sceptic.

## Evidence
Source ID, class, and what it actually says.

## Consequences
What this commits us to. What becomes easier. What becomes harder.

## Alternatives considered
What else was on the table, and why not.

## Follow-up tasks
| # | Task | Owner | Status |

## What this decision does NOT settle
The questions that remain open even after this decision.
```