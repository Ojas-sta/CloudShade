# Shared Interface Contracts

**Owner:** Agent 3 (Integration Lead)
**Status:** 🔴 **`ALL ENTRIES ARE `PROPOSED`. NOTHING HERE IS AGREED. NOTHING IS
IMPLEMENTED.`**
**Last updated:** 2026-10-11
**Change procedure:** `AGENTS.md` §G · **Register template:** below

---

## 🔴 Status warning

This register exists because two agents independently wrote integration
documents without seeing each other's, and produced two incompatible contracts.
[`unresolved_conflicts.md`](unresolved_conflicts.md) holds the findings.

🔴 **Reading this file must not lead anyone to believe an integration exists.**
There is **0 source files**, **0 tests**, and no API of any kind. Every entry
below describes *paper*.

| Label | Meaning here |
|---|---|
| `PROPOSED` | Drafted by Agent 3 from two conflicting sources. **Not agreed.** |
| `AGREED` | Both owners have reviewed and accepted. 🔴 **None currently.** |
| `IMPLEMENTED` | Code exists. 🔴 **None currently.** |

---

## Sources this register reconciles

| # | Document | Owner | Position |
|---|---|---|---|
| S1 | `docs/shana/architecture/cloudlight_integration.md` | Agent 1 | SHANA's proposed interface |
| S2 | `docs/cloudlight/shana_integration.md` | Agent 2 | CloudLight's proposed interface |
| S3 | `docs/shana/architecture/task_state_machine.md` | Agent 1 | SHANA's state machine |
| S4 | `data/schemas/delivery.schema.yaml` | Agent 1 | Proposed delivery schema |
| S5 | `docs/cloudlight/architecture/api_contracts.md` | Agent 2 | CloudLight's API surface |
| S6 | `docs/shana/safety/operating_restrictions.md` | Agent 1 | 57 numbered restrictions |
| S7 | `docs/cloudlight/security_and_privacy.md` | Agent 2 | CloudLight security position |

---

## IC-01 — Direction of communication

| | |
|---|---|
| **Status** | 🔴 `PROPOSED` — ⚠️ **the two sources actually agree here** |
| **Sources** | S1 §3, S2 §3 |

**Current behaviour.** CloudLight sends high-level **requests** and receives
**status**. The reverse direction carries no command channel.

**Agreed by both sources**, in compatible terms:

- CloudLight → SHANA: `task.assign`, `task.cancel`, `task.pause` / `task.resume`,
  `system.stop`, `system.pause`
- SHANA → CloudLight: `status.*` — see `IC-04`, where the two disagree

**Why this is the strongest entry in the register.** S1 §4.5 and S2 §1 state the
same boundary independently, and
[`ADR-014`](../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md)
records it as an accepted, non-reversible decision.

🔴 **This is the one place where the two subsystems arrived at the same answer
without talking to each other.** It is the strongest available evidence that
the boundary is well-founded.

**Deliberately absent from any request** (S2 §4): payload dimensions, weights,
motor parameters, route hints, speed preferences, safety-overriding priority.

---

## IC-02 — Primary identifier and idempotency

| | |
|---|---|
| **Status** | 🔴 `PROPOSED` — ⚠️ **the sources conflict** |
| **Conflict** | `CONF-06` |
| **Sources** | S2 §4–§5, S4 |

**CloudLight proposes** two fields:

| Field | Purpose |
|---|---|
| `request_id` | Client-generated unique identifier |
| `idempotency_key` | Deduplicates retries of the same intent |

**SHANA's schema proposes** one: `delivery_id` (required, string). 🔴 **No
`idempotency_key` field exists.**

### 🔴 Why this is a safety control, not a nicety

S2 §5:

> ⚠️ **Two physical tasks from one tap is a physical safety event, not a data
> bug** — a second robot moving through a corridor carrying an item nobody asked
> for. Idempotency is treated as a P0 control, not an optimisation
> (`CL-SEC-016`, `CL-SHANA-018`).

S2 §5 also requires: *"The request is persisted before transmission."*

### Open question for both owners

Does SHANA's `delivery_id` **equal** CloudLight's `request_id`, or does SHANA
issue its own? The second reading breaks the idempotency guarantee across the
boundary. **Not resolved. Owner: Agent 1 + Agent 2.**

---

## IC-03 — Request payload fields

| | |
|---|---|
| **Status** | 🔴 `PROPOSED` |
| **Sources** | S2 §4, S4 |

| Field | CloudLight (S2) | SHANA schema (S4) | Reconciled proposal |
|---|---|---|---|
| `request_id` / `delivery_id` | ✅ | ✅ `delivery_id` | ⚠️ `IC-02` |
| `idempotency_key` | ✅ | ⛔ **absent** | 🔴 required |
| `origin` | ✅ | ✅ `origin_id` | naming drift |
| `destination` | ✅ | ✅ `destination_id` | naming drift |
| `item_category` | ✅ *"class of item, not contents"* | ⛔ absent | ✅ CloudLight's is better — it avoids leaking contents |
| `item_weight_kg` | 🔴 **explicitly forbidden** — *"payload dimensions, weights"* | ✅ **required** | 🔴 **direct conflict** |
| `requested_by` | ✅ *identity reference* | ✅ *"teacher role id"* | `CONF-10` — prefer the **role id** |
| `requested_window` | ✅ | ⛔ absent | ✅ |
| `schema_version` | ✅ | ⛔ absent | 🔴 **required** — without it there is no version negotiation |
| `planned_route` | 🔴 **forbidden** | ⚠️ optional array | 🔴 should be **removed** — it is a route hint |
| `loaded_confirmed` | ⛔ absent | ✅ optional | robot-side |
| `recipient_confirmed` | ⛔ absent | ✅ optional | 🔴 **the only route to `COMPLETED`** |

### 🔴 The weight conflict

S4 requires `item_weight_kg` on every delivery. S2 forbids transmitting it.

⚠️ **Both positions are defensible and the owners must choose.** SHANA's is
safer: a robot carrying an item it cannot lift is a hazard, and only the robot's
own policy should decide what it accepts. CloudLight's is cleaner: a teacher
should not be able to influence a physical parameter.

**Reconciliation proposal:** SHANA applies **its own** item policy from its own
onboard weight sensing. CloudLight transmits a **category only**, never a weight.
This satisfies both intents. 🔴 **Requires both owners' review.**

---

## IC-04 — 🔴 Delivery state vocabulary

| | |
|---|---|
| **Status** | 🔴 `PROPOSED` — ⚠️ **unreconciled. The most serious conflict found.** |
| **Conflict** | `CONF-02`, `CONF-08` |
| **Proposed resolution** | [`ADR-016`](../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md) — **`Proposed`, not `Accepted`** |

| SHANA state (S3, S4) | CloudLight display state (S2 §6) | Problem |
|---|---|---|
| `QUEUED` | `QUEUED` | ⚠️ ambiguous — see below |
| `VALIDATING` | — | ⛔ no representation |
| `READY` | — | ⛔ no representation |
| `NAVIGATING_TO_PICKUP` | `IN_PROGRESS` | merged |
| `AWAITING_LOADING_CONFIRMATION` | `IN_PROGRESS` | ⚠️ **a human must act** |
| `NAVIGATING_TO_DESTINATION` | `IN_PROGRESS` | ⚠️ |
| `AWAITING_RECIPIENT` | `IN_PROGRESS` / `AWAITING_CONFIRMATION` | 🔴 **two states, contradictory mappings** |
| `COMPLETED` | `DELIVERED` | naming drift |
| `PAUSED` | 🔴 **absent — `grep` returns 0** | 🔴 **CRITICAL** |
| `CANCELLED` | `CANCELLED` | ok |
| `FAILED` | `FAILED` | ok |
| — | `REQUESTED` | CloudLight-only |
| — | `ACCEPTED` | CloudLight-only |
| — | `REJECTED` | 🔴 **SHANA has no such state** — it routes rejection to `CANCELLED` |
| — | `UNKNOWN` | CloudLight-only |

### 🔴 The critical defect

SHANA can enter `PAUSED` from **any** state (S3:70). CloudLight has **no**
`PAUSED`. A paused robot is therefore displayed as `IN_PROGRESS` — **"On the
way"** — to a teacher.

This violates CloudLight's own invariant 8 (*"never presents an inferred or stale
status as current"*) and its own §6 warning that a stale *"On the way" causes a
teacher to wait for a robot that already stopped*.

### ⚠️ A second collapse, less often noticed

`AWAITING_RECIPIENT` also maps to `IN_PROGRESS`. To a teacher these mean
**opposite** things:

- `NAVIGATING_TO_DESTINATION` → **wait**
- `AWAITING_RECIPIENT` → 🔴 **go to the robot now**

### Proposal (ADR-016)

1. **S3 is the single source of truth.** CloudLight consumes; it does not redefine.
2. **CloudLight's display model must include `PAUSED`.**
3. **No lossy collapse into `IN_PROGRESS`.**
4. `REJECTED` needs a SHANA state or an explicit mapping rule.
5. CloudLight-only states (`REQUESTED`, `ACCEPTED`, `UNKNOWN`) are **declared**
   as request-lifecycle states, not robot states.
6. 🔴 **Until agreed, no integration may be implemented.**

---

## IC-05 — SHANA's outbound `status.*` messages

| | |
|---|---|
| **Status** | 🔴 `PROPOSED` — ⚠️ **internally inconsistent** |
| **Conflict** | `CONF-08` |

S1 §3 publishes eight messages; S3 defines eleven states; **no mapping between
them is stated.**

| `status.*` (S1) | Plausible state (S3) | Explicit? |
|---|---|---|
| `status.ready` | `READY`? `QUEUED`? | ⛔ ambiguous |
| `status.assigned` | — | ⛔ no state |
| `status.en_route` | `NAVIGATING_TO_DESTINATION` | ⛔ inferred |
| `status.blocked` | `FAILED`? | ⛔ no `BLOCKED` state exists |
| `status.arrived` | `AWAITING_RECIPIENT` | ⛔ inferred |
| `status.delivered` | `COMPLETED` | ⛔ inferred |
| `status.error` | `FAILED` | ⛔ inferred |
| `status.estop` | — | ⛔ no `ESTOP` state exists |

⚠️ 🔴 **`status.blocked` and `status.estop` have no corresponding state in
SHANA's own machine.** An implementer would have to invent the mapping, and
would likely reintroduce `CONF-02`.

**Requested from Agent 1:** an explicit `state → status.*` table.

---

## IC-06 — Authentication between subsystems

| | |
|---|---|
| **Status** | 🔴 `PROPOSED` — **P0, unanswered** |

S2 §9 requires mutual authentication: mTLS, or a signed short-expiry token.

> 🔴 **P0 control. Without it, any host on the network can move the robot.**

S2 §9 also requires teacher identity to travel as a *reference* resolved by
CloudLight, never as a credential SHANA must trust.

🔴 **SHANA has not weighed in on this.** S1 §3 specifies no authentication for
either direction. **Unreconciled.**

---

## IC-07 — Location and destination naming

| | |
|---|---|
| **Status** | 🔴 `PROPOSED` |

| Source | Term |
|---|---|
| S1 §3 | "destination" — "Task assigned to unknown destination → **Reject; never improvise a destination**" |
| S2 §4 | `origin` / `destination` — "Must be a location SHANA recognises" |
| S4 | `origin_id` / `destination_id` |

⚠️ Both sources agree on the **behaviour** — unknown locations are rejected, and
a destination may never be free text (S4 explicitly prohibits a free-text
destination field). ⚠️ They disagree on the **identifier format**.

**Source of truth:** `data/schemas/destination.schema.yaml` (Agent 1 authored).
CloudLight must resolve names to SHANA identifiers before transmitting.

🔴 **No location registry exists.** S1 §5 requires rejection of unknown
destinations — with no registry, **every** request would be rejected.

---

## IC-08 — Failure semantics

| | |
|---|---|
| **Status** | ⚠️ **Substantially aligned** — the one encouraging finding |

| Failure | SHANA (S1 §5) | CloudLight (S2 §7) | Compatible? |
|---|---|---|---|
| Peer unreachable | *"Continue current task if safe; otherwise hold; do not start new work"* | *"CloudLight dashboard unaffected"* | ✅ yes |
| SHANA offline | — | Reject new requests locally; show age of last known | ✅ |
| CloudLight offline | 🔴 **"must degrade safely"** | 🔴 **"must not specify SHANA's behaviour when CloudLight is down"** | ✅ **explicitly aligned** |
| Restart | — | Reconcile non-terminal requests by `request_id` | ⚠️ needs SHANA input |
| Blocked mid-task | Stop at nearest safe point; `status.blocked` | Show last known **with age**; do not re-send | ✅ |

🔴 S2 §7.2 is notable:

> ⚠️ **CloudLight must not specify SHANA's behaviour when CloudLight is down.**
> That is SHANA's decision to make and document, and it may reasonably differ
> from what CloudLight would prefer.

**Two agents, working alone, produced the same boundary from opposite sides.**
This corroborates `ADR-014` independently.

---

## IC-09 — Cancellation

| | |
|---|---|
| **Status** | 🔴 `PROPOSED` |

S2 §8:

- Cancellation is **requested** by CloudLight, **decided** by SHANA
- SHANA may refuse; refusal is legitimate
- 🔴 **"CloudLight must never display 'Cancelled' before SHANA has confirmed
  cancellation."** The UI must show `CANCEL_REQUESTED` until then.

⚠️ `CANCEL_REQUESTED` is a **tenth CloudLight-only state**, not in S2's own table
of 10 states — an internal inconsistency in S2 as well as a cross-subsystem one.

**Consistent with SHANA's safety ownership.** 🔴 Requires an explicit SHANA
policy on post-pickup cancellation (refuse vs. item return, which is a **second
physical movement**).

---

## IC-10 — Error handling

| | |
|---|---|
| **Status** | 🔴 `PROPOSED` — **no shared error model exists** |

| Source | Error handling |
|---|---|
| S1 §5 | `status.error` "with reason code" — 🔴 **no reason codes are defined** |
| S2 §6 | `FAILED` displayed with *"Failure reason, in plain language"* |
| S2 §7.4 | *"It does not guess. A human decides."* |

⚠️ **Two incompatible error philosophies.** SHANA publishes machine reason
codes; CloudLight expects prose. Neither is wrong alone — they are two layers.

**Needed:** a reason-code taxonomy on the wire, plus a CloudLight prose mapping.
🔴 **No such taxonomy exists. Requested from Agent 1 + Agent 2.**

---

## IC-11 — 🔴 Safety invariants (not negotiable through this register)

| | |
|---|---|
| **Status** | ⚠️ **NOT `PROPOSED`. These are settled and outside interface negotiation.** |

🔴 **Recorded here so that no future contract revision can weaken them.** See
`AGENTS.md` §L and
[`security_boundaries.md`](security_boundaries.md).

| # | Invariant | Origin |
|---|---|---|
| 1 | No AI model output reaches the robot, directly or indirectly | ADR-001 *(owed — see `CONF-04`)* |
| 2 | No motor, path, speed or actuator command path in CloudLight, **any mode** | [`ADR-014`](../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md) |
| 3 | `system.stop` honoured unconditionally and immediately | S1 §4.2 |
| 4 | No raw video ever leaves the robot | ADR-010 *(owed)* |
| 5 | No student identification, ever | `AGENTS.md` §L.9 |
| 6 | A blocked or unknown route is never traversed — fail closed | `AGENTS.md` §L.5 |
| 7 | No command bypasses SHANA's own safety validation | S1 §4.8 |
| 8 | Delivery completion requires explicit recipient confirmation | S4 `validation`; S2 §6 |
| 9 | CloudLight never presents inferred or stale status as current | S2 §10.8 |
| 10 | SHANA's safety never depends on CloudLight's availability | S1 §4.6; S2 §7.2 |

🔴 **Where a contract proposal conflicts with an invariant, the invariant wins
and the proposal is rejected.** No amount of schedule pressure changes this.

---

## Entry template

```markdown
## IC-NN — Title
| | |
|---|---|
| **Status** | PROPOSED / AGREED / IMPLEMENTED |
| **Sources** | S1, S4, … |
| **Conflict** | CONF-NN, or "none" |

**Current behaviour** — quoted, with file path.
**Proposed behaviour** — what it becomes.
**Reason** — why.
**Backward-compatibility impact** — who breaks.
**Required change — SHANA.**
**Required change — CloudLight.**
**Tests required** — what would demonstrate it.
**Migration strategy** — or "none".
**Rollback implications.**
**Decision status** — PROPOSED / ACCEPTED / REJECTED / SUPERSEDED.
**Reviewers** — who must accept.
```

---

## Register status

| Metric | Value |
|---|---|
| Entries | 11 |
| 🔴 `AGREED` | **0** |
| 🔴 `IMPLEMENTED` | **0** |
| `PROPOSED` | 10 |
| Settled by policy, not negotiation | 1 (`IC-11`) |
| Conflicts referenced | 6 |
| 🔴 Implementable today | **0 — no code exists, and gates have not passed** |