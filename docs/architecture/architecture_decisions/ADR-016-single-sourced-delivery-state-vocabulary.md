# ADR-016: The delivery state vocabulary must be single-sourced

**Status:** `Proposed` 🔴 — **not `Accepted`**. Requires review by Agent 1 and
Agent 2.
**Date:** 2026-10-11
**Decides:** where the delivery state machine is defined, and what CloudLight is
permitted to display
**Decided by:** Agent 3, on evidence gathered by inspection
**Reversal condition:** SHANA and CloudLight agree a single vocabulary that
expresses both the robot's internal states and the display states without
loss — **or** SHANA's own review establishes that CloudLight legitimately needs a
reduced, independent view. In that case this ADR is superseded by a documented
projection with an explicit, tested lossy-mapping rule.
**Evidence:** `docs/shana/architecture/task_state_machine.md`,
`docs/shana/architecture/cloudlight_integration.md`,
`docs/cloudlight/shana_integration.md` §6, `data/schemas/delivery.schema.yaml`

---

## Context

SHANA and CloudLight each defined a delivery status vocabulary, **without ever
seeing the other's**.

🔴 Under [ADR-015](ADR-015-single-integration-repository-not-three.md) they were
authored in isolation in one tree, with no interface-agreement step. This ADR
records the resulting defect, which is a **safety-relevant correctness problem**,
not merely a tidiness issue.

### SHANA's vocabulary — 11 states, robot-side

From `data/schemas/delivery.schema.yaml` and
`docs/shana/architecture/task_state_machine.md`:

`QUEUED` · `VALIDATING` · `READY` · `NAVIGATING_TO_PICKUP` ·
`AWAITING_LOADING_CONFIRMATION` · `NAVIGATING_TO_DESTINATION` ·
`AWAITING_RECIPIENT` · `COMPLETED` · `PAUSED` · `CANCELLED` · `FAILED`

### CloudLight's vocabulary — 10 states, display-side

From `docs/cloudlight/shana_integration.md` §6:

`REQUESTED` · `ACCEPTED` · `QUEUED` · `IN_PROGRESS` · `AWAITING_CONFIRMATION` ·
`DELIVERED` · `FAILED` · `CANCELLED` · `REJECTED` · `UNKNOWN`

### 🔴 The defect

**SHANA can transition from *any* state to `PAUSED`. CloudLight has no `PAUSED`
state. `grep -rn "PAUSED" docs/cloudlight/` returns **zero matches**.

CloudLight maps every executing SHANA state onto `IN_PROGRESS`, displayed to a
teacher as **"On the way"**. Therefore:

> **A paused robot — halted by an operator, holding position, waiting for a
> human — would be displayed to a teacher as "On the way".**

This directly contradicts **CloudLight's own invariant 8**:

> *"CloudLight never presents an inferred or stale status as current."*

And it contradicts its own §6 warning immediately below the state table:

> *"`UNKNOWN` must never be rendered as `IN_PROGRESS`. When the link drops, the
> last known state is shown with its age. A stale 'On the way' causes a teacher
> to wait for a robot that already stopped."*

⚠️ The reasoning is correct. The implementation of the state table contradicts
it, because `PAUSED` has nowhere to go except `IN_PROGRESS`.

### Secondary conflicts

| # | Conflict | Detail |
|---|---|---|
| a | `REJECTED` has no SHANA counterpart | SHANA routes validator rejection to **`CANCELLED`** (`task_state_machine.md:28`). CloudLight would display `REJECTED`; SHANA emits `CANCELLED`. A rejection and a withdrawal become indistinguishable |
| b | `QUEUED` is ambiguous | SHANA: *accepted, waiting for a slot*. CloudLight: *accepted but not yet started*. Adjacent but not identical, and `ACCEPTED` sits between them on only one side |
| c | `ACCEPTED` / `REQUESTED` are CloudLight-only | They are request-lifecycle states, not robot states. Legitimate — but unlabelled as such |
| d | `UNKNOWN` has no SHANA counterpart | Legitimate display-only state. Should be explicitly declared as such |
| e | Terminology drift | `DELIVERED` vs `COMPLETED`, `AWAITING_CONFIRMATION` vs `AWAITING_RECIPIENT` |
| f | `status.delivered` vs `AWAITING_RECIPIENT` | SHANA's outbound message table names `status.arrived` and `status.delivered`; neither appears in its state machine under those names |

---

## Decision

**There is exactly one state machine, owned by SHANA. CloudLight may not define
a parallel vocabulary. CloudLight may hold a documented *projection* — a
lossless, read-only mapping from SHANA states to display strings — but the
projection must be declared lossy in writing if it is lossy, and it must never
present a state SHANA did not report.**

Concretely:

1. **Single source of truth:** SHANA's state machine
   (`docs/shana/architecture/task_state_machine.md`) plus
   `data/schemas/delivery.schema.yaml`. CloudLight consumes; it does not
   redefine.
2. **`PAUSED` must appear in CloudLight's display model.** It is a state SHANA
   can be in. It has no safe default.
3. **No lossy collapse into `IN_PROGRESS`.** Where CloudLight genuinely wants a
   coarser display, it maps several SHANA states to one string **only if** the
   collapsed set is homogeneous with respect to what the teacher would need to
   do. `NAVIGATING_TO_DESTINATION` and `AWAITING_RECIPIENT` are **not**
   homogeneous — one means "wait", the other means "go there".
4. **`REJECTED` requires a SHANA state or an explicit mapping rule.** CloudLight
   must not invent a terminal state SHANA cannot emit.
5. **The projection is a versioned, reviewable artefact** in
   `docs/integration/interface_contracts.md`, changed through the
   interface-change procedure (`AGENTS.md` §G) — never edited silently in one
   subsystem.
6. 🔴 **Until both owners agree this, no integration may be implemented.** There
   is nothing to build against an ambiguous contract.

---

## Rationale

**Why is this not merely a naming disagreement?**

Because one specific collapse produces a **false statement to a teacher about
the physical state of a robot carrying an item through a school**. "On the way"
is not an approximation of "paused and waiting for a human". It invites a
teacher to send someone to a corridor where nothing is coming.

**Why must SHANA own it, not CloudLight?**

SHANA is the only party that knows the robot's actual state, and it owns the
physical consequence (`ADR-015` §8.3). A second vocabulary defined by the party
that does not observe the robot is, by construction, guesswork.

**Why forbid a coarse display entirely?**

It is not forbidden — it is *constrained*. A coarser display is good design
when it hides detail without hiding meaning. The defect here is hiding **meaning**.
`AWAITING_RECIPIENT` means *"a person must go to the robot now"*;
`IN_PROGRESS` means *"nothing is required of you yet"*. Merging them tells a
teacher the opposite of the truth.

**Why `Proposed` and not `Accepted`?**

🔴 Both subsystems must review this before it binds them. Agent 3 may propose the
contract; it may not impose it on either owner. SHANA may reasonably argue its
internal machine should stay private and expose a coarser API — that is a
legitimate position, and this ADR anticipates it in its reversal condition.

---

## Evidence

| Fact | Source | Class |
|---|---|---|
| SHANA defines 11 states incl. `PAUSED` | `data/schemas/delivery.schema.yaml`; `task_state_machine.md` §States | `DESIGN_ONLY` |
| "Any state ──► PAUSED" | `docs/shana/architecture/task_state_machine.md:70` | `DESIGN_ONLY` |
| CloudLight defines 10 states, none `PAUSED` | `docs/cloudlight/shana_integration.md` §6 | `DESIGN_ONLY` |
| `PAUSED` absent from all CloudLight documents | `grep -rn "PAUSED" docs/cloudlight/` → **0 matches** | `VERIFIED` |
| CloudLight invariant 8 forbids presenting inferred status as current | `docs/cloudlight/shana_integration.md` §10.8 | `DESIGN_ONLY` |
| CloudLight's own §6 warns against stale "On the way" | `docs/cloudlight/shana_integration.md` §6 | `DESIGN_ONLY` |
| SHANA rejects to `CANCELLED`, not a `REJECTED` state | `docs/shana/architecture/task_state_machine.md:28` | `DESIGN_ONLY` |
| Both documents written without sight of each other | SHANA doc: "`docs/cloudlight/` was empty at the 2026-10-11 audit"; CloudLight doc: "no joint session has occurred" | `VERIFIED` |

⚠️ **Every behavioural claim above is `DESIGN_ONLY`.** There is no code, no
test, and no implemented API. This ADR is about a contract defect, not about a
running defect.

---

## Consequences

### If accepted

- CloudLight gains `PAUSED`, and likely `REJECTED`, as explicit states.
- CloudLight's display table becomes a *declared projection* rather than an
  independent vocabulary — auditable, reviewable, versioned.
- 🔴 **Some CloudLight UI text changes.** This is the point.
- A test becomes possible later: *"for every SHANA state, CloudLight shows a
  status that does not understate urgency."* Today that test cannot exist
  because there is no code.
- 🔴 **No implementation may begin** until the projection is agreed.

### If rejected

- CloudLight keeps an independent vocabulary.
- 🔴 **Then the mapping must be written down, reviewed by both owners, and
  tested for every SHANA state** — including `PAUSED`. An undocumented
  independent vocabulary is not an option.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| **Keep both vocabularies, document the difference** | 🔴 This is the status quo. It has already produced a false-status defect. Documenting a known lossy collapse without fixing it is not reconciliation. |
| **Let CloudLight own the vocabulary** | Rejected. CloudLight does not observe the robot; it cannot know which state the robot is in. Ownership would guarantee drift. |
| **Collapse SHANA's machine to CloudLight's 10 states** | Rejected. It would delete `AWAITING_LOADING_CONFIRMATION` and `AWAITING_RECIPIENT` — the two states where a **human must act**. Those are the most safety-relevant states in the machine. |
| **SHANA exposes a private machine; CloudLight sees only `IN_PROGRESS`/`DONE`** | 🔴 Legitimate, and explicitly preserved in the reversal condition. But it still must distinguish `PAUSED` from `IN_PROGRESS`, or a paused robot is misreported. |
| **Resolve it now by fiat** | Rejected. Agent 3 does not own either subsystem. `AGENTS.md` §G requires owner review. |

---

## What this decision does NOT settle

1. Whether SHANA exposes its full state machine to CloudLight, or a coarser
   public API. That is SHANA's call.
2. Whether `AWAITING_LOADING_CONFIRMATION` needs a CloudLight representation at
   all — a teacher may have no business seeing it.
3. Authentication between the subsystems (`docs/cloudlight/shana_integration.md`
   §9 remains `P0` and unanswered).
4. Timeouts, retries, and reconciliation after restart.
5. 🔴 **All of the above are moot until the go/no-go gates pass.** No
   integration may be implemented regardless.

---

## Follow-up tasks

| # | Task | Owner | Status |
|---|---|---|---|
| 1 | Review this ADR: is SHANA's machine the right single source? | Agent 1 | ⛔ `REQUESTED` |
| 2 | Review this ADR: accept the projection rule? | Agent 2 | ⛔ `REQUESTED` |
| 3 | Publish the agreed projection in `interface_contracts.md` | Agent 3 | ⛔ blocked on 1, 2 |
| 4 | Write the `PAUSED` state into CloudLight's display model | Agent 2 | ⛔ blocked on 1, 2 |
| 5 | Define whether `REJECTED` becomes a SHANA state or a mapping | Agent 1 + Agent 2 | ⛔ blocked on 1, 2 |
| 6 | Specify the test that proves no state is understated | Agent 3 | ⛔ blocked on 3 |
| 7 | Implement | — | ⛔ `BLOCKED` — no code exists; gates not passed |

---

## Related

- [ADR-015](ADR-015-single-integration-repository-not-three.md) — the topology that allowed this
- [ADR-014](ADR-014-cloudlight-never-controls-robot-motion.md) — 🔴 this ADR must not weaken it
- [`docs/integration/unresolved_conflicts.md`](../../integration/unresolved_conflicts.md) — `CONF-02`
- [`docs/integration/interface_contracts.md`](../../integration/interface_contracts.md)
- `docs/shana/architecture/task_state_machine.md` — the source of truth
- `docs/cloudlight/shana_integration.md` §6 — the projection under review
- `AGENTS.md` §G — the interface-change procedure