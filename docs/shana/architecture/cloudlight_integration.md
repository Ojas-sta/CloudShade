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

🔴 **`docs/cloudlight/` was empty at the 2026-10-11 audit.** SHANA has **no
visibility** into CloudLight's implementation, schedule, or assumptions. Any
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
| Occupancy and scheduling intelligence | ⚠️ Currently human-declared — see `D-031` |
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
| 6.3 | CloudLight counterpart | `UNKNOWN` — owned elsewhere |
| 6.4 | Shared schema agreement | ⚠️ **Not agreed.** No joint session has occurred |
| 6.5 | Integration testing | `BLOCKED` — cannot test against nothing |

⚠️ **This is a proposal made in isolation.** A real integration requires a joint
session with the CloudLight owner to agree schemas, error codes, and timeout
behaviour. **That has not happened.**

---

## 7. Recommendations to whoever coordinates both agents

| # | Recommendation |
|---|---|
| 1 | Agree the message schema in a joint session before either side writes code |
| 2 | Define timeout and retry semantics explicitly |
| 3 | Define a version-negotiation strategy — schemas will change |
| 4 | Define who is authoritative for occupancy data (`D-031` is provisional) |
| 5 | 🔴 Write the invariant list above into the shared contract, not only here |
| 6 | Keep the SHANA-side validation non-bypassable regardless of partner pressure |

---

## Related

- [`../safety/operating_restrictions.md`](../safety/operating_restrictions.md) — 57 numbered restrictions
- [`../architecture/architecture_decisions/README.md`](../architecture/architecture_decisions/README.md) — ADR-001, ADR-010
- [`../operations/human_intervention.md`](../operations/human_intervention.md) — why the human is the safety system
- [`unresolved_dependencies.md`](../handoff/unresolved_dependencies.md) — interface agreement is an open dependency