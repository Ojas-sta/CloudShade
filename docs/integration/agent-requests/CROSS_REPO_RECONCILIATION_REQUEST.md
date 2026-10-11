# Reconciliation Request — Agent 1 (SHANA) and Agent 2 (CloudLight)

**From:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Status:** 🔴 **`PENDING` — not delivered, not acknowledged**
**Audience:** Agent 1 **and** Agent 2 — this is a joint task

---

## 🔴 Why this document exists

Two agents each wrote a complete, careful integration document **without seeing
the other's**. Both are internally sound. 🔴 **They are mutually incompatible.**

The result is [`../unresolved_conflicts.md`](../unresolved_conflicts.md):
10 open conflicts, of which one is safety-relevant.

🔴 **This is not a failing by either agent.** Under
[`ADR-015`(../../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md)
they had no mechanism to see each other. The fix is procedural, not personal.

⚠️ **Workflow requests** (`AGENT_1_SHANA_GIT_WORKFLOW_REQUEST.md`,
`AGENT_2_CLOUDLIGHT_GIT_WORKFLOW_REQUEST.md`) cover Git and ownership. **This
document covers only the contract.** Read this one if you only have time for one.

---

## The one-sentence job

> **Read the other subsystem's integration document, then produce one agreed
> contract at [`../interface_contracts.md`](../interface_contracts.md).**

---

## Step 1 — Read both documents (30 minutes)

| Document | Owner | Read for |
|---|---|---|
| [`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md) | Agent 1 | SHANA's interface proposal |
| [`../../cloudlight/shana_integration.md`](../../cloudlight/shana_integration.md) | Agent 2 | CloudLight's interface proposal |
| [`../interface_contracts.md`](../interface_contracts.md) | Agent 3 | 🔴 **The reconciliation. 11 entries, 0 `AGREED`** |

⚠️ **The third document is Agent 3's draft** — a merge of both positions, with
the conflicts marked. 🔴 **It is a proposal. Neither owner has reviewed it, and it
binds no one.** Its value is that everything you need to argue about is on one
page.

---

## Step 2 — 🔴 Agree the state vocabulary first

**Everything else is downstream of this.**

### The problem

`grep -rn "PAUSED" docs/cloudlight/` → **0 matches.**

SHANA's machine can enter `PAUSED` from **any** state. CloudLight maps every
executing state to `IN_PROGRESS`, shown as **"On the way"**.

> **A paused robot would be shown to a teacher as "On the way."**

This violates CloudLight's own invariant 8 and its own §6 warning about stale
status. ⚠️ **CloudLight's reasoning is correct; its state table contradicts it**,
because `PAUSED` has nowhere to go.

And a second collapse: `AWAITING_RECIPIENT` also maps to `IN_PROGRESS`.

| SHANA state | Reality | Teacher must |
|---|---|---|
| `NAVIGATING_TO_DESTINATION` | Moving | Wait |
| `AWAITING_RECIPIENT` | Holding, at destination | 🔴 **Go now** |

### Decide

| # | Question | Owner |
|---|---|---|
| 1 | Is SHANA's machine the single source of truth? | Both |
| 2 | Does CloudLight expose the full machine, or a projection? | Both |
| 3 | What does `PAUSED` display as? | Agent 2 |
| 4 | Does `AWAITING_RECIPIENT` get its own display? | Agent 2 |
| 5 | Is `REJECTED` a SHANA state or a mapping rule? *(SHANA routes rejection to `CANCELLED`)* | Agent 1 |
| 6 | Does `QUEUED` mean the same thing on both sides? | Both |

### Review

[`ADR-016`(../../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md)
proposes a resolution. 🔴 **`Proposed` — Agent 3 cannot decide it.** Accept,
reject with a private-machine argument, or amend.

⚠️ **A private machine is a legitimate position.** But it must still distinguish
`PAUSED` from moving, or the defect survives any design.

---

## Step 3 — Settle the remaining conflicts

| ID | Question | Owner | Priority |
|---|---|---|---|
| `CONF-06` | Does `request_id` equal `delivery_id`? Where does idempotency live? 🔴 **P0 — two tasks from one tap is a physical safety event** | Both | 🔴 High |
| `CONF-08` | Publish a `state → status.*` mapping. 🔴 `status.blocked` and `status.estop` have **no state** | Agent 1 | 🔴 High |
| `IC-03` | Item weight: SHANA requires it, CloudLight forbids sending it | Both | 🟠 Med |
| `IC-06` | Authentication between subsystems — CloudLight proposes mTLS; SHANA has not weighed in | Both | 🔴 High |
| `IC-10` | Error model: SHANA publishes reason codes, CloudLight expects prose | Both | 🟠 Med |
| `CONF-09` | Should occupancy ever be a CloudLight input to SHANA? 🔴 **Agent 3 recommends no** | Both | 🟠 Med |
| `CONF-10` | `requested_by` — role ID vs identity reference 🔴 **recommend role ID** | Agent 2 | 🟡 Low |
| `CONF-07` | `/v1/` vs `/api/v1/` | Agent 2 | 🟡 Low |

### Non-negotiable while reconciling

🔴 **None of these may be weakened, relaxed, or traded for simplicity.** They are
not interface decisions; they are the safety case.

1. No AI output reaches the robot — `ADR-001` *(owed — write it, Agent 1)*
2. No motor/path/speed/actuator path in CloudLight, **any mode** —
   [`ADR-014`(../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md),
   🔴 **not reversible**
3. `system.stop` honoured unconditionally, with **no authorisation check**
4. No raw video leaves the robot — `ADR-010` *(owed)*
5. No student identification, ever
6. A blocked or unknown route is never traversed — fail closed
7. `COMPLETED` requires explicit recipient confirmation, **never** arrival
8. CloudLight never presents inferred or stale status as current
9. SHANA's safety never depends on CloudLight's availability
10. A location SHANA does not recognise is **rejected**, never improvised

⚠️ **Where a proposal conflicts with an invariant, the invariant wins and the
proposal is rejected.** Schedule pressure is not an argument.

---

## Step 4 — Write the outcome

### Update the register

In [`../interface_contracts.md`](../interface_contracts.md), each entry moves:

`PROPOSED` → **`AGREED`** — only when **both** owners have signed off.

🔴 **Agent 3 does not mark anything `AGREED`.** Agent 3 publishes what the
owners have agreed.

### Where disagreements survive

Record them as an **ADR** with a reversal condition. A recorded disagreement with
a named owner is a better outcome than a silent unilateral fix, because it stays
visible.

### Update

| File | Who |
|---|---|
| [`../interface_contracts.md`](../interface_contracts.md) | Agent 3, from owners' decisions |
| [`../unresolved_conflicts.md`](../unresolved_conflicts.md) | Agent 3 |
| [`../TASK_BOARD.md`](../TASK_BOARD.md) | Agent 3 |
| `docs/shana/**` / `docs/cloudlight/**` | 🔴 **Each owner — only their own** |

---

## Step 5 — 🔴 Do not begin implementing

⚠️ **Even after the contract is agreed, no code may be written.**

| Blocker | Owner |
|---|---|
| Go/no-go gates — **0 of 20 passed** | School / administration |
| No code exists in either subsystem | Agent 1 + 2 |
| ≈1,000 questions asked, **0 answered** | School |

🔴 **Agreement on a contract does not authorise building against it.** See
`AGENTS.md` §J and §L.

---

## 🔴 Three things both agents should know

### 1. You got the boundary right, independently

Without sight of each other, both reproduced the same four positions:

| Agreement | SHANA | CloudLight |
|---|---|---|
| CloudLight assigns tasks; SHANA decides how to move | ✅ | ✅ |
| Communication is one-way | ✅ | ✅ |
| SHANA may reject — a normal outcome, not an error | ✅ | ✅ |
| CloudLight must not specify SHANA's behaviour when CloudLight is down | ✅ | ✅ |

🔴 **Independent convergence on a safety boundary is stronger evidence than
negotiated agreement** — it means the boundary follows from the problem, not
from negotiation pressure. [`ADR-014`(../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md)
is corroborated after the fact.

### 2. 🔴 The `PAUSED` defect is the whole lesson

Both documents are *well-argued*. Both are *incomplete in the same way* — they
each describe the whole system as if they owned it. Neither could see the gap.

🔴 **That is precisely what a shared written contract is for**, and why
`AGENTS.md` §G requires one:

> *"If it is not written in `docs/integration/`, it does not exist."*

### 3. ⚠️ None of this blocks the project

🔴 **≈1,000 questions have been asked and 0 answered.** The contract will matter
a great deal — eventually. **Right now, no answer to it enables anything.**

The things that would enable things:

| Action | Cost |
|---|---|
| Principal: may a local service run? | ~10 min |
| IT: can devices reach it? Is isolation on? | ~15 min |
| Timetable: who approves a substitution? | ~15 min |
| Custodian: robot model, firmware, battery? | ~30 min |
| Lift provider: any sanctioned interface? | ~1 week (external) |

⚠️ **Reconcile the contract — and do not mistake it for progress on the thing
that actually blocks this project.**