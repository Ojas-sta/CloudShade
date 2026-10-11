# SHANA ↔ CloudLight Integration

🔴 **Ownership boundary. Read before touching anything under `docs/cloudlight/`.**

This document defines what SHANA is responsible for, what CloudLight is
responsible for, and where the two meet.

---

## 1. Ownership rules

| Path | Owner | SHANA may… |
|---|---|---|
| `docs/shana/**` | **SHANA** | ✅ Read, write, extend |
| `docs/cloudlight/**` | **CloudLight agent** | ⚠️ **Read only. Never modify.** |
| `data/schemas/**` | Shared — SHANA authored | ⚠️ Propose changes via interface revision |
| `questionnaires/**` | SHANA | ✅ Read, write |

⚠️ **`docs/cloudlight/` was empty when SHANA wrote this document** (audit of
2026-10-11). 🔴 **That is no longer true — Agent 2's documentation now exists
(47 documents).**

**This document was therefore written in isolation, and a partner proposal has
since appeared.** Agent 2's own
[`shana_integration.md`](../../cloudlight/shana_integration.md) states the same
thing about SHANA. 🔴 **Neither document was aware of the other. That isolation
is the defect recorded as `CONF-01` and `CONF-02`.**

⚠️ **This file has now been read against Agent 2's document.** Where they
disagree — chiefly on the status vocabulary — see
[`../handoff/RESPONSE_TO_AGENT_3.md`](../handoff/RESPONSE_TO_AGENT_3.md) R4 and
§3 below. Any
statement about CloudLight in SHANA documents is a *proposal*, not a description
of something that exists.

---

## 2. Responsibility split

### SHANA owns — and must not delegate

| Responsibility | Why it stays with SHANA |
|---|---|
| Robot motion and navigation | Direct physical consequence. Cannot be handed to another system |
| Localisation and floor identity | 🔴 The robot cannot sense altitude — a safety-critical gap SHANA must manage |
| Obstacle avoidance and safe stop | Safety-critical, real-time, must not depend on a network peer |
| Battery management and safe return | Safety-critical |
| Route validation | Safety-critical |
| Physical emergency stop response | Must function with all networks down |

⚠️ **The principle: any component whose failure could injure a person must not
depend on an external system's availability.** SHANA must remain safe when
CloudLight is unreachable, late, or wrong.

### CloudLight owns

| Responsibility | Notes |
|---|---|
| Higher-level building intelligence | Outside SHANA's remit |
| Occupancy and scheduling intelligence | 🔴 **Never** a route-safety input — see [ADR-009](architecture_decisions/ADR-009-manual-occupancy-before-sensing.md) |
| User-facing interfaces | Out of SHANA scope |
| Aggregated reporting | Out of SHANA scope |

---

## 3. Proposed interface

⚠️ **`DESIGN_ONLY` — never implemented, never validated against a real partner.**

SHANA proposes to consume commands and publish status. Proposed shapes are in
`data/schemas/`. They have **never been parsed by code**.

### Inbound — commands SHANA would accept

| Message | Meaning | SHANA's obligation |
|---|---|---|
| `task.assign` | Deliver item to destination | Validate; reject unsafe routes |
| `task.cancel` | Abandon current task | Stop safely, hold position |
| `task.pause` / `task.resume` | Hold / continue | Hold in a safe location |
| `system.stop` | 🛑 **Emergency stop** | ⚠️ **Honour immediately and unconditionally** |
| `system.pause` | Cease new tasks | Complete or safely abort current |

### Outbound — status SHANA would publish

| Message | Meaning |
|---|---|
| `status.ready` | Idle, available for work |
| `status.assigned` | Task accepted |
| `status.en_route` | Navigating |
| `status.blocked` | 🔴 Obstacle or unsafe state; needs human help |
| `status.arrived` | At destination |
| `status.delivered` | Handover confirmed |
| `status.error` | Failure, with reason code |
| `status.estop` | 🛑 Emergency stop engaged |
| `status.awaiting_loading` | 🔴 Robot holding; **a person must load** |
| `status.awaiting_recipient` | 🔴 Robot holding at destination; **a person must collect** |
| `status.paused` | ⚠️ Halted by an operator; robot is holding |
| `status.cancelled` | Task ended without completion |

#### 🔴 Authoritative mapping

⚠️ **This list is derived from, and subordinate to, the state machine.**
The authoritative `state → status.*` mapping is in
[`task_state_machine.md`](task_state_machine.md) §State → published message
mapping. If this table and that one disagree, **the state machine wins**.

⚠️ `status.arrived` is retained for backward compatibility but is **redundant
with `status.awaiting_recipient`** — a robot that has arrived *is* awaiting its
recipient. 🔴 `status.arrived` must **never** be rendered as "delivered":
completion requires explicit recipient confirmation (`D-007`, `AGENTS.md` §L.4).

#### Three states previously unpublished

⚠️ Before 2026-10-11 this list omitted `PAUSED`, `AWAITING_LOADING_CONFIRMATION`
and `AWAITING_RECIPIENT` — the states in which **a human must act**. That was the
defect recorded as `CONF-02` and `CONF-08`. All three are now published.
🔴 **`PAUSED` in particular may never be collapsed into "on the way."**

---

## 4. 🔴 Invariants SHANA must never violate

These are not negotiable and are not subject to interface convenience.

| # | Invariant | Rationale |
|---|---|---|
| 4.1 | 🔴 **No AI model reaches the robot** | ADR-001. Deterministic safety layer is in the loop |
| 4.2 | 🔴 **`system.stop` is honoured unconditionally and immediately** | A peer must never be able to talk SHANA out of a stop |
| 4.3 | 🔴 **No raw video ever leaves the robot** | ADR-010 / [`../safety/student_privacy.md`](../safety/student_privacy.md) |
| 4.4 | 🔴 **No student identification, ever** | Privacy position is absolute |
| 4.5 | 🔴 **CloudLight cannot command physical motion** | CloudLight assigns *tasks*; SHANA decides *how* to move |
| 4.6 | 🔴 **SHANA must degrade safely without CloudLight** | No peer dependency on safety functions |
| 4.7 | 🔴 **A blocked or unknown route is never traversed** | Fail closed, never fail open |
| 4.8 | 🔴 **No command bypasses SHANA's own safety validation** | Convenience never overrides safety |

⚠️ **Invariant 4.5 is the most important architectural boundary in this
document.** CloudLight may say *"deliver this to room 12"*. It may **never** say
*"drive at 1.5 m/s"*, *"skip the checkpoint"*, or *"ignore that corridor"*. The
how of movement is SHANA's alone.

---

## 5. Failure behaviour

| Failure | SHANA's behaviour |
|---|---|
| CloudLight unreachable | ⚠️ Continue current task if safe; otherwise hold; do not start new work |
| Malformed message | Reject, log, report `status.error` |
| Conflicting commands | Obey the most restrictive |
| Repeated `system.stop` | Idempotent; already stopped |
| Task assigned to unknown destination | Reject; never improvise a destination |
| Route becomes blocked mid-task | Stop at nearest safe point; report `status.blocked` |

---

## 6. Integration status

| # | Item | Status |
|---|---|---|
| 6.1 | Interface concept | `DESIGN_ONLY` |
| 6.2 | Proposed message schemas | `DESIGN_ONLY` — never validated |
| 6.3 | CloudLight counterpart | ✅ **Now exists** — 47 documents, incl. Agent 2's own `shana_integration.md` |
| 6.4 | Shared schema agreement | 🔴 **Not agreed.** Both proposals exist; neither has been reconciled |
| 6.5 | Integration testing | `BLOCKED` — cannot test against nothing |

⚠️ **This proposal and Agent 2's were written independently and disagree on the
status vocabulary.** Agent 1 has reviewed Agent 2's document and recorded a
position in [`../handoff/RESPONSE_TO_AGENT_3.md`](../handoff/RESPONSE_TO_AGENT_3.md)
R4. 🔴 **Agreement has not occurred.** The reconciliation is Agent 3's to
publish.

---

## 7. Recommendations to whoever coordinates both agents

| # | Recommendation |
|---|---|
| 1 | Agree the message schema in a joint session before either side writes code |
| 2 | Define timeout and retry semantics explicitly |
| 3 | Define a version-negotiation strategy — schemas will change |
| 4 | Define who is authoritative for occupancy data ([ADR-007](architecture_decisions/ADR-007-corridor-state-human-declared.md), [ADR-009](architecture_decisions/ADR-009-manual-occupancy-before-sensing.md)) |
| 5 | 🔴 Write the invariant list above into the shared contract, not only here |
| 6 | Keep the SHANA-side validation non-bypassable regardless of partner pressure |

---

## Related

- [`../safety/operating_restrictions.md`](../safety/operating_restrictions.md) — 57 numbered restrictions
- [`../architecture/architecture_decisions/README.md`](../architecture/architecture_decisions/README.md) — ADR-001, ADR-010
- [`../operations/human_intervention.md`](../operations/human_intervention.md) — why the human is the safety system
- [`unresolved_dependencies.md`](../handoff/unresolved_dependencies.md) — interface agreement is an open dependency