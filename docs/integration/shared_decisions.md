# Shared Decisions — Cross-Project Register

**Owner:** Agent 3 (Integration Lead)
**Last updated:** 2026-10-11

Subsystem-internal decisions stay with their owners:
`docs/shana/feasibility/decision_log.md` ·
`docs/cloudlight/decision_log.md`.
This file holds only decisions that **bind both** subsystems.

---

## ADR index (all files — verified to exist)

🔴 **Only four ADR files exist in this repository.** Documents citing ADR-001,
ADR-010 or ADR-011 cite decisions that were never written as files. See
[`unresolved_conflicts.md`](unresolved_conflicts.md) `CONF-04`.

| ADR | Title | Status | Owner | Reversal condition |
|---|---|---|---|---|
| [`ADR-002`](../shana/architecture/architecture_decisions/ADR-002-single-floor-first.md) | Single-floor first; staged deployment | `Accepted` | Agent 1 | Single-floor route proven **and** cross-floor demand confirmed **and** vertical transit permitted |
| [`ADR-003`](../shana/architecture/architecture_decisions/ADR-003-stairs-and-lifts-excluded.md) | Stairs and lifts excluded from the pilot | `Provisional` | Agent 1 | Per-modality conditions in ADR-003 |
| [`ADR-013`](../architecture/architecture_decisions/ADR-013-modular-monolith-not-microservices.md) | Modular monolith, not microservices | `Provisional` | Agent 2 | Scale or team evidence |
| [`ADR-014`](../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md) | CloudLight never controls robot motion | **`Accepted` — not reversible** | Agent 3 | 🔴 **None.** The absence of a motion path is the control |
| [`ADR-015`](../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md) | One integration repository, not three | `Accepted` | Agent 3 | Owner splits repositories or a hosting platform enables cross-repo PRs |
| [`ADR-016`](../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md) | Single-sourced delivery state vocabulary | 🔴 **`Proposed`** | Agent 3 | Owners agree a lossless vocabulary, **or** SHANA argues for a private machine with a documented projection |

### Owed — decisions recorded in a log, never written as files

🔴 Per `docs/shana/architecture/architecture_decisions/README.md`. 🔴 **Three
of these are cited across the subsystem boundary.**

| Would be | Decision | Status | Cited by CloudLight? |
|---|---|---|---|
| **ADR-001** | No AI model may reach the robot | `Accepted` | 🔴 **Yes** — as a safety invariant "not CloudLight's to relax" |
| **ADR-004** | Wired Ethernet for the robot link | `Accepted` | No |
| **ADR-005** | Self-built navigation over vendor SLAM | `Provisional` | No |
| **ADR-006** | ArUco markers for floor identity | `Provisional` | No |
| **ADR-007** | Corridor state is human-declared | `Provisional` | No |
| **ADR-008** | No lift modification, ever | `Accepted` | No |
| **ADR-009** | Manual occupancy before sensing | `Provisional` | ⚠️ Relates to `CONF-09` |
| **ADR-010** | No raw video retention | `Accepted` | 🔴 **Yes** — privacy invariant |
| **ADR-011** | Simulation before hardware | `Accepted` | ⚠️ Cited in SHANA docs |
| **ADR-012** | Radio telemetry deferred | `Provisional` | No |

🔴 **10 ADRs are owed. Agent 1 owns writing them. This is the highest-priority
cross-project integrity task**, because two of the cited ones carry safety
weight.

---

## D-01 — CloudLight never commands robot motion

**Status:** `Accepted` · **Reversal condition:** 🔴 **none.**
Recorded in
[`ADR-014`](../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md).

CloudLight's entire vocabulary toward SHANA is limited to *"deliver this approved
item to this approved destination, within this window."* It has **no
representation of** motor commands, gait, velocity, waypoints, paths,
checkpoints, localisation, map corrections, floor identity, battery thresholds,
docking, obstacle avoidance, emergency stop, safe stop, recovery, or any
override or "continue anyway" semantics.

🔴 **The boundary is an absence of capability, not a policy.** A policy can be
bypassed by a bug or an intruder. An absent code path cannot be called.

⚠️ **Corroborated independently.** CloudLight, working without sight of
SHANA's ADR, reached the same conclusion from the opposite direction
(`shana_integration.md` §1). 🔴 **Independent convergence is stronger evidence
than negotiated agreement** — it means the boundary follows from the problem.

---

## D-02 — The dependency arrow points CloudLight → SHANA, and is optional

**Status:** `Accepted` — 🟡 **inferred from both subsystems' documents, not
formally recorded.** Recommend promotion to an ADR.

🔴 **CloudLight must operate fully when SHANA is offline.** A robot failure must
never blank a classroom screen.

> *"A SHANA integration that is unreachable, rejected, or refused would take
> every classroom screen down with it if the dependency ran the other way."*
> — `docs/cloudlight/shana_integration.md` §12

Both subsystems agree. 🔴 **Neither has written it as a numbered ADR**, which
means it is currently protected by convention rather than by record.

---

## D-03 — One repository, not three

**Status:** `Accepted` — [`ADR-015`](../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md)
· **Reversal condition:** owner splits repositories, or hosting enables
cross-repo PRs.

Recorded because the operating brief described three repositories and
inspection found one. 🔴 **A brief, an ADR and any future summary will
otherwise disagree**, and the brief's version is what an incoming agent is most
likely to act on.

---

## D-04 — The delivery state vocabulary must be single-sourced

**Status:** 🔴 **`Proposed`** — [`ADR-016`](../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md)
· 🔴 **Not binding until Agent 1 and Agent 2 review.**

SHANA's state machine is the single source of truth. CloudLight holds a
**declared projection**. `PAUSED` must be representable. No lossy collapse into
`IN_PROGRESS`.

⚠️ Recorded as `Proposed` because **Agent 3 does not own either subsystem.**
Publishing a cross-project decision as `Accepted` unilaterally is precisely what
`AGENTS.md` §M forbids.

---

## D-05 — ⚠️ Occupancy must never be a CloudLight input to SHANA

**Status:** 🔴 **`Proposed`** — Agent 3's recommendation. **Not agreed.**

🔴 **No ADR currently settles this**, and the two subsystems' documents are
compatible only by accident.

| Source | Statement |
|---|---|
| `cloudlight_integration.md` §2 | "Occupancy and scheduling intelligence" → **CloudLight**, ⚠️ *"currently human-declared — see `D-031`"* |
| ADR-009 / `D-031` (SHANA, `Provisional`) | "Manual occupancy **before** sensing" |
| `shana_integration.md` §2 | "Higher-level building intelligence" → **CloudLight** |

**Why it matters.** SHANA's invariant is *"a blocked or unknown route is never
traversed"* — which assumes SHANA knows corridor state itself. 🔴 **A
CloudLight occupancy feed would insert a network peer into SHANA's safety
inputs**, the exact dependency `AGENTS.md` §L.6 forbids.

⚠️ If CloudLight ever *derives* occupancy from timetable and substitution data,
`D-031` and ADR-009 both become wrong without anyone noticing.

**Recommendation:** SHANA's route safety uses **only locally sensed state**.
CloudLight may *display* occupancy. This should be a numbered invariant, not an
implicit assumption. See `CONF-09`.

---

## D-06 — ⚠️ Release signing is deferred past the first pilot

**Status:** 🔴 **`Proposed`** — 🔴 **flagged, not endorsed.**

CloudLight's first-release scope defers the over-the-air update mechanism to
*after the first pilot*.

⚠️ **Consequence: the first pilot would run unsigned software on school
hardware, and an unauthorised host on the school LAN could present a "signed"
update that is not verified.**

Given that the school LAN is explicitly not trusted, 🔴 this should be an
explicit, recorded, owner-accepted decision — **not an oversight of scope
sequencing**. Agent 3 does not own CloudLight's scope and is flagging, not
directing.

---

## Decisions deliberately NOT taken

| Not decided | Why |
|---|---|
| Whether SHANA exposes its full state machine or a coarser API | Agent 1's call — preserved in `ADR-016`'s reversal condition |
| The identifier model (`request_id` vs `delivery_id`) | Both owners — `CONF-06` |
| Whether item weight crosses the interface | Both owners — `IC-03` |
| The error taxonomy | Both owners — `IC-10` |
| The API base path | Agent 2 — `CONF-07` |
| Whether a remote should exist | 🔴 **The repository owner** |
| Any go/no-go gate | Its named owner, with evidence |

---

## Register status

| Metric | Value |
|---|---|
| ADR files existing | **6** |
| ADRs owed | **10** |
| 🔴 ADRs cited but nonexistent | **3** |
| Decisions binding both subsystems | **4** (2 `Accepted`, 2 `Proposed`) |
| 🔴 Decisions Agent 3 may impose unilaterally | **0** |
| 🔴 Cross-project ADRs awaiting both owners' review | **1** (`ADR-016`) |