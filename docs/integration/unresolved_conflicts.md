# Unresolved Conflicts — Cross-Project Register

**Owner:** Agent 3 (Integration Lead)
**Last updated:** 2026-10-11
**Rule:** a conflict is recorded here **with both positions quoted**, assigned an
owner, and left open until the owning agent resolves it. Agent 3 does not
resolve a conflict by editing another subsystem's files.

> 🔴 **Every conflict below was found by direct inspection**, not reported by a
> participant. Each entry names the command or file that produced it, so it can
> be re-checked.

---

## Summary

| ID | Conflict | Severity | Owner | Status |
|---|---|---|---|---|
| [CONF-01](#conf-01) | Two integration documents written without sight of each other | 🔴 High | Agent 1 + Agent 2 | `REQUESTED` |
| [CONF-02](#conf-02) | Delivery state vocabularies incompatible; `PAUSED` unrepresentable | 🔴 High | Agent 1 + Agent 2 | `REQUESTED` — ADR-016 |
| [CONF-03](#conf-03) | 6 broken internal links across the SHANA/CloudLight boundary | 🟠 Medium | Agent 2 | `REQUESTED` — mapping supplied |
| [CONF-04](#conf-04) | Documents cite ADRs that do not exist as files | 🔴 High | Agent 1 | `REQUESTED` |
| [CONF-05](#conf-05) | `docs/architecture/delivery_state_machine.md` does not exist anywhere | 🟠 Medium | Agent 2 | `REQUESTED` |
| [CONF-06](#conf-06) | `request_id` vs `delivery_id` — divergent primary identifiers | 🟠 Medium | Agent 1 + Agent 2 | `REQUESTED` |
| [CONF-07](#conf-07) | Delivery API base path: `/v1/` vs `/api/v1/` | 🟡 Low | Agent 2 | `REQUESTED` |
| [CONF-08](#conf-08) | SHANA's outbound `status.*` set does not match its own state machine | 🟠 Medium | Agent 1 | `REQUESTED` |
| [CONF-09](#conf-09) | Occupancy authority unresolved (`D-031` provisional vs CloudLight as "building intelligence" owner) | 🟠 Medium | Agent 1 + Agent 2 | `REQUESTED` |
| [CONF-10](#conf-10) | `requested_by` semantics differ: teacher role ID vs teacher identity reference | 🟡 Low | Agent 2 | `REQUESTED` |
| [CONF-11](#conf-11) | `.gitignore` gap: `.obsidian/` was not ignored | 🟡 Low | Agent 3 | ✅ `RESOLVED` |

---

## CONF-01 — Two integration documents, neither aware of the other

**Severity:** 🔴 High · **Owner:** Agent 1 + Agent 2 · **Status:** `REQUESTED`
· **Related:** [ADR-015](../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md)

### Position A — SHANA

From `docs/shana/architecture/cloudlight_integration.md` §1:

> 🔴 **`docs/cloudlight/` was empty at the 2026-10-11 audit.** SHANA has **no
> visibility** into CloudLight's implementation, schedule, or assumptions.

§6.4:

> ⚠️ **This is a proposal made in isolation.** A real integration requires a
> joint session with the CloudLight owner… **That has not happened.**

### Position B — CloudLight

From `docs/cloudlight/shana_integration.md`, preamble:

> No interface has been agreed with SHANA's authors, no message schema has been
> validated by code, and **no joint session has occurred.**
>
> ⚠️ **Any assumption that SHANA has accepted a schema described here is
> unsupported.**

### Why this is a conflict

Both documents are candid about the isolation — that is to their credit. 🔴 But
**the isolation itself is the defect.** Both were written as proposals for a
contract that the other party had already, independently, proposed differently.
Neither cites the other.

The result is [CONF-02](#conf-02) and [CONF-06](#conf-06): two incompatible
contracts, both authoritative-looking, neither agreed.

### Resolution

1. Both owners read the other's document — [`shana_integration.md`](../cloudlight/shana_integration.md) and [`cloudlight_integration.md`](../shana/architecture/cloudlight_integration.md).
2. The union of both is drafted into a single contract at
   [`interface_contracts.md`](interface_contracts.md), owned by Agent 3.
3. Each owner records `ACCEPTED` / `REJECTED` / `AMENDEND` against each entry.
4. Any surviving disagreement becomes an ADR, not a chat thread.

---

## CONF-02 — Delivery state vocabularies are incompatible, and `PAUSED` cannot be represented

**Severity:** 🔴 High · **Owner:** Agent 1 + Agent 2 · **Status:** `REQUESTED`
· **Proposed resolution:** [ADR-016](../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md)
· 🔴 **This is the most safety-relevant conflict found.**

### Position A — SHANA's machine (11 states)

`docs/shana/architecture/task_state_machine.md` and
`data/schemas/delivery.schema.yaml`:

`QUEUED` `VALIDATING` `READY` `NAVIGATING_TO_PICKUP`
`AWAITING_LOADING_CONFIRMATION` `NAVIGATING_TO_DESTINATION`
`AWAITING_RECIPIENT` `COMPLETED` `PAUSED` `CANCELLED` `FAILED`

> ⚠️ **Any state ──► PAUSED ──► resume to prior state, or CANCELLED**
> (`task_state_machine.md:70`)

> ⚠️ **Only two states advance without a human action, and neither reaches a
> destination.** Every delivery requires at least two human confirmations —
> loading and receipt.

### Position B — CloudLight's display vocabulary (10 states)

`docs/cloudlight/shana_integration.md` §6:

`REQUESTED` `ACCEPTED` `QUEUED` `IN_PROGRESS` `AWAITING_CONFIRMATION`
`DELIVERED` `FAILED` `CANCELLED` `REJECTED` `UNKNOWN`

### 🔴 The defect

`grep -rn "PAUSED" docs/cloudlight/` → **0 matches.**

CloudLight maps all executing states to `IN_PROGRESS`, rendered to a teacher as
**"On the way"**. SHANA can be `PAUSED` from any state — halted by an operator,
holding position, waiting for a human.

> **A paused robot would be displayed to a teacher as "On the way."**

### Why that is a safety problem, not a naming problem

CloudLight's own invariant 8 forbids it
(`docs/cloudlight/shana_integration.md` §10):

> 🔴 **CloudLight never presents an inferred or stale status as current.**

And CloudLight's own §6 says:

> ⚠️ **`UNKNOWN` must never be rendered as `IN_PROGRESS`.** … A stale "On the
> way" causes a teacher to wait for a robot that already stopped.

🔴 **The document reasons correctly and then contradicts itself in the state
table**, because `PAUSED` has nowhere to go. The failure mode the document warns
against is the one its own vocabulary produces.

Worse, the collapse also merges `AWAITING_RECIPIENT` into `IN_PROGRESS`. Those
mean opposite things to a teacher:

| SHANA state | Physical reality | What the teacher must do |
|---|---|---|
| `NAVIGATING_TO_DESTINATION` | Robot is moving | **Wait** |
| `AWAITING_RECIPIENT` | Robot is holding, at destination | 🔴 **Go to the robot now** |

### Secondary conflicts

| # | Conflict | Detail |
|---|---|---|
| a | `REJECTED` has no SHANA counterpart | SHANA routes validator rejection to **`CANCELLED`** (`task_state_machine.md:28`). Rejection and withdrawal become indistinguishable |
| b | `QUEUED` is ambiguous | SHANA: *accepted, waiting for a slot*. CloudLight: *accepted but not yet started* |
| c | `ACCEPTED` / `REQUESTED` / `UNKNOWN` are CloudLight-only | Legitimate request-lifecycle states — but undeclared as such |
| d | Terminology drift | `DELIVERED` vs `COMPLETED`; `AWAITING_CONFIRMATION` vs `AWAITING_RECIPIENT` |

### Resolution proposed by ADR-016

SHANA's machine is the single source of truth. CloudLight holds a **declared
projection**, and:
- `PAUSED` **must** appear in CloudLight's display model
- no lossy collapse into `IN_PROGRESS`
- `REJECTED` requires a SHANA state or an explicit mapping rule
- the projection is versioned and changed only via the interface-change procedure

🔴 **`Proposed`, not `Accepted`.** Both owners must review. SHANA may reasonably
argue for a private machine with a coarser public API — that position is
preserved in ADR-016's reversal condition.

---

## CONF-03 — Six broken internal links across the subsystem boundary

**Severity:** 🟠 Medium · **Owner:** Agent 2 · **Status:** `REQUESTED`

### Finding

A scan of **1097 internal markdown links** found **6 broken targets**. All are
CloudLight → SHANA references.

### Root cause — established by inspection

SHANA's documents originally sat at `docs/architecture/`, `docs/safety/` and
`docs/infrastructure/`. They were later moved under `docs/shana/`. CloudLight
was written against the **pre-move** layout.

🔴 **CloudLight detected this itself and recorded rather than fixed it** —
correctly, under the ownership rule:
`docs/cloudlight/README.md:204` and
`docs/cloudlight/research/source_register.md:933`.

### Verified mapping

Agent 3 has confirmed every target exists. Agent 2 may apply this:

| Broken reference (as written) | Cited from | Correct target |
|---|---|---|
| `../architecture/architecture.md` | `requirements.md` | `docs/shana/architecture/architecture.md` |
| `../architecture/data_flow.md` | `timetable_and_substitutions.md` | `docs/shana/architecture/data_flow.md` |
| `../infrastructure/school_network.md` | `requirements.md`, `resource_browser.md`, `resource_search.md` | `docs/shana/infrastructure/school_network.md` |
| `../safety/risk_register.md` | `requirements.md`, `risk_register.md` | `docs/shana/safety/risk_register.md` |
| `../safety/student_privacy.md` | `ai_assistant.md`, `requirements.md` | `docs/shana/safety/student_privacy.md` |
| `docs/architecture/delivery_state_machine.md` | `timetable_and_substitutions.md`, `ai_assistant.md`, `shana_integration.md`, `sources.md` | `docs/shana/architecture/task_state_machine.md` ⚠️ **name differs, not just path** |

⚠️ The last row is more than a path fix — see
[CONF-05](#conf-05).

### Why Agent 3 did not fix it

🔴 These files are owned by Agent 2. `AGENTS.md` §C.3 permits Agent 3 to fix a
broken link "whose target unambiguously exists" and to notify the owner. 🔴 **The
sixth row does not qualify** — the intended filename does not exist, so the
target is not unambiguous. The first five are safe but are bundled with the
sixth into one CloudLight-side change, so all six are handed to Agent 2 with
this table.

---

## CONF-04 — Documents cite architecture decisions that do not exist

**Severity:** 🔴 High · **Owner:** Agent 1 · **Status:** `REQUESTED`

### Finding

Only **4 ADR files exist** in the entire repository: `ADR-002`, `ADR-003`
(SHANA-internal) and `ADR-013`, `ADR-014` (cross-project).

🔴 CloudLight's **safety argument** rests on three that were never written:

| Cited as | Cited in | Existence |
|---|---|---|
| `ADR-001` — *no AI model may reach the robot* | `docs/cloudlight/shana_integration.md` §10.1, §10 invariants 1–3; `security_and_privacy.md` | ⛔ **No such file** |
| `ADR-010` — *no raw video retention* | `docs/cloudlight/shana_integration.md` §10.6 | ⛔ **No such file** |
| `ADR-011` — *simulation before hardware* | cited in SHANA docs as an accepted decision | ⛔ **No such file** |

`docs/shana/architecture/architecture_decisions/README.md` confirms these are
indexed as **"owed"** — real decisions with recorded reversal conditions, held
in `docs/shana/feasibility/decision_log.md`, not yet written as files.

### Why this is high severity

🔴 **It is an evidence-integrity defect, not a navigation defect.** CloudLight
cites `ADR-001` as an *accepted* architecture decision that "is **not**
CloudLight's to relax." An incoming agent following that citation finds nothing,
and must then decide whether the safety invariant still holds. The *decisions* are
real and well-argued. The *citations* are unresolvable.

Nine further ADRs are owed: `ADR-001`, `ADR-004`, `ADR-005`, `ADR-006`,
`ADR-007`, `ADR-008`, `ADR-009`, `ADR-010`, `ADR-011`, `ADR-012`.

### Resolution

1. Agent 1 writes the owed ADR files, at minimum **ADR-001, ADR-010, ADR-011**
   which are cited cross-subsystem.
2. Agent 3 then updates `docs/integration/evidence_index.md` to point at files
   that exist.
3. 🔴 **Until written, no document may cite an ADR ID that does not resolve to a
   file.** `AGENTS.md` §M.

---

## CONF-05 — `docs/architecture/delivery_state_machine.md` does not exist anywhere

**Severity:** 🟠 Medium · **Owner:** Agent 2 · **Status:** `REQUESTED`

### Finding

Cited as existing in at least four CloudLight documents:

- `docs/cloudlight/timetable_and_substitutions.md:22`
- `docs/cloudlight/ai_assistant.md:588`
- `docs/cloudlight/shana_integration.md:175`, `:336`
- `docs/cloudlight/sources.md:382`

Nearest real file: `docs/shana/architecture/task_state_machine.md`.

🔴 **This is not a path error — it is a filename error.** An earlier CloudLight
document already noticed it:
`docs/cloudlight/sources.md:382` records "⚠️ **No such file.** The nearest is
`docs/architecture/task_state_machine.md`". CloudLight cited it correctly, in
isolation, and elsewhere cited it as if it existed.

### Resolution

Decide once: does SHANA's state machine file get renamed, or do CloudLight's
citations get pointed at the existing name? 🔴 **Renaming a SHANA file is
Agent 1's decision** — it would break references inside `docs/shana/`.
Recommendation: **correct the CloudLight citations**; do not rename.

⚠️ This is entangled with [CONF-02](#conf-02) — whichever way it resolves should
be decided together.

---

## CONF-06 — `request_id` vs `delivery_id`

**Severity:** 🟠 Medium · **Owner:** Agent 1 + Agent 2 · **Status:** `REQUESTED`

### Positions

**CloudLight** (`docs/cloudlight/shana_integration.md` §4):

> `request_id` — Client-generated unique identifier. **Also the idempotency key**
> `idempotency_key` — Deduplicates retries of the same intent

**SHANA** (`data/schemas/delivery.schema.yaml`):

```yaml
delivery_id:  { type: string, required: true }
```

🔴 **SHANA has no `idempotency_key` field at all.**

### Why it matters

CloudLight's design depends on idempotency as a **P0 safety control**, not an
optimisation:

> ⚠️ **Two physical tasks from one tap is a physical safety event, not a data
> bug** — a second robot moving through a corridor carrying an item nobody asked
> for. (`shana_integration.md` §5)

If SHANA has only `delivery_id`, either:
- **CloudLight's `request_id` maps to SHANA's `delivery_id`** — the contract must
  say so explicitly, or
- **SHANA issues its own `delivery_id`** — then the two are different values and
  the idempotency guarantee is broken across the boundary.

⚠️ Additionally, SHANA's schema requires `item_weight_kg` while CloudLight's
request contract explicitly forbids weight influencing SHANA
(*"payload dimensions, weights"*) — see
[interface_contracts.md](interface_contracts.md) `IC-03`.

### Resolution

Agree one identifier model. Recorded as `IC-02` and `IC-03`.

---

## CONF-07 — API base path differs

**Severity:** 🟡 Low · **Owner:** Agent 2 · **Status:** `REQUESTED`

CloudLight specifies `/v1/...`
(`docs/cloudlight/architecture/api_contracts.md`, e.g. `POST /v1/deliveries`,
`GET /v1/deliveries/{id}`). The integration brief's examples use `/api/v1/...`.

⚠️ **No implementation exists, so nothing is broken.** Recorded so the mismatch
is settled by the owner rather than discovered at implementation time.

---

## CONF-08 — SHANA's outbound `status.*` messages do not match its own state machine

**Severity:** 🟠 Medium · **Owner:** Agent 1 · **Status:** `REQUESTED`

### Positions

`docs/shana/architecture/cloudlight_integration.md` §3 publishes eight outbound
messages: `status.ready`, `status.assigned`, `status.en_route`, `status.blocked`,
`status.arrived`, `status.delivered`, `status.error`, `status.estop`.

`docs/shana/architecture/task_state_machine.md` defines eleven states:
`QUEUED` `VALIDATING` `READY` `NAVIGATING_TO_PICKUP`
`AWAITING_LOADING_CONFIRMATION` `NAVIGATING_TO_DESTINATION`
`AWAITING_RECIPIENT` `COMPLETED` `PAUSED` `CANCELLED` `FAILED`.

### The mismatch

| SHANA state | No corresponding `status.*` message |
|---|---|
| `VALIDATING` | ⛔ none |
| `READY` | ambiguous — `status.ready` plausible but unstated |
| `NAVIGATING_TO_PICKUP` | ⛔ none |
| `AWAITING_LOADING_CONFIRMATION` | ⛔ none |
| `NAVIGATING_TO_DESTINATION` | `status.en_route` plausible but unstated |
| `AWAITING_RECIPIENT` | `status.arrived` plausible but unstated |
| `PAUSED` | 🔴 **none** — same gap as CONF-02 |
| `CANCELLED` | ⛔ none |
| `COMPLETED` | `status.delivered` plausible but unstated |

🔴 **Within SHANA's own documentation.** The mapping is never stated, so an
implementer would have to invent it — and would likely reproduce CONF-02.

### Resolution

Agent 1 should publish an explicit `state → status.*` mapping table alongside the
state machine. Part of
[ADR-016](../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md).

---

## CONF-09 — Occupancy authority is unresolved

**Severity:** 🟠 Medium · **Owner:** Agent 1 + Agent 2 · **Status:** `REQUESTED`

**SHANA** (`cloudlight_integration.md` §2) lists "Occupancy and scheduling
intelligence" as **CloudLight's**, ⚠️ *currently human-declared — see `D-031`*.

**SHANA's own record** (`ADR-009` / `D-031`, provisional): "Manual occupancy
before sensing" — i.e. SHANA's position is that occupancy is **not yet sensed**.

**CloudLight** (`shana_integration.md` §2) assigns "higher-level building
intelligence" to CloudLight.

🔴 **These are compatible only by accident.** If CloudLight ever *derives*
occupancy (from timetable + substitution data), it becomes a live source
SHANA's route safety would need — and `D-031` becomes wrong, as does ADR-009.

🔴 **This matters for safety.** SHANA's invariant *"a blocked or unknown route
is never traversed"* assumes SHANA knows the corridor state itself. A CloudLight
occupancy feed would insert a network peer into SHANA's safety inputs — the exact
dependency `AGENTS.md` §L.6 forbids.

### Resolution

Decide explicitly whether occupancy is **ever** a CloudLight input to SHANA.
Recommendation: 🔴 **No.** SHANA's route safety must use only locally sensed
state. CloudLight may *display* occupancy. This should be stated as an
invariant, not left implicit.

---

## CONF-10 — `requested_by` means different things

**Severity:** 🟡 Low · **Owner:** Agent 2 · **Status:** `REQUESTED`

| Source | Definition |
|---|---|
| `data/schemas/delivery.schema.yaml` | `requested_by: { type: string, description: teacher role id }` |
| `docs/cloudlight/shana_integration.md` §4 | *"Teacher identity reference. Not credentials; not a secret"* |

⚠️ A **role ID** (e.g. `teacher`) and an **identity reference** (e.g. a
directory object ID) carry different privacy weight. SHANA's reading is the
more conservative; CloudLight's doc also warns *"Identity of the teacher travels
as a reference, resolved by CloudLight; never as a credential SHANA must
trust."*

🔴 **Consistent with privacy invariant 9** (`AGENTS.md` §L.9 — no person is ever
identified). Resolve in favour of the **role ID**; record in
`interface_contracts.md`.

---

## CONF-11 — `.gitignore` gap

**Severity:** 🟡 Low · **Owner:** Agent 3 · **Status:** ✅ `RESOLVED`

`.obsidian/` was not ignored. `git add --dry-run` showed four vault files —
including `workspace.json`, which records one person's editor session — would
have been committed.

🔴 Resolved in the Phase A baseline commit, as a machine-specific artefact
exclusion. No agent-owned path was touched.

---

## How to close a conflict

1. The **owning agent** decides. Agent 3 does not decide for them.
2. Record the resolution as an **ADR** with a reversal condition, or amend the
   existing ADR if one governs.
3. Update this file: status → `RESOLVED`, with the date and the ADR reference.
4. Update the task board.
5. 🔴 **Never close a conflict by deleting the record of it.** The conflict is
   the evidence that the two subsystems were written independently. That fact
   should remain visible.