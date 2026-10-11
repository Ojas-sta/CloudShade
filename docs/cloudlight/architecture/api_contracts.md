# CloudLight — API Contracts and the SHANA Interface Gap

**Date:** 2026-10-11 · **Status: `DESIGN_ONLY` · nothing implemented · nothing tested**

> **🔴 No API exists.** No endpoint, no route, no handler, no client, no test.
> This document records a *proposal*, and — more importantly — a **material
> finding about a mismatch between two proposals that have never been reconciled.**

---

## 1. What exists today

| Artefact | Owner | State |
|---|---|---|
| CloudLight's SHANA proposal | CloudLight | [`../shana_integration.md`](../shana_integration.md) §4 |
| SHANA's interface proposal | SHANA | `docs/shana/architecture/cloudlight_integration.md` §3 (read-only) |
| Four YAML schemas | SHANA | `data/schemas/*.yaml` — headers read `# PROPOSED — not implemented. Documentation only.` |

⚠️ **All three were authored without sight of each other.** No joint session has
occurred. SHANA's own document states this: *"Shared schema agreement — Not
agreed. No joint session has occurred."*

⚠️ **The four schemas are documentation.** They have never been parsed by any
code, because there is no code. They are a design sketch, not a contract in force.

---

## 2. 🔴 The material finding: the two proposals diverge

### 2.1 Side-by-side

| Aspect | CloudLight's proposal | SHANA's proposal / schema | Assessment |
|---|---|---|---|
| **Primary identifier** | `request_id` **plus** a dedicated `idempotency_key` | `delivery_id` only | 🔴 **GAP** |
| **Deduplication** | Required — a retry is the same request, and no second task may be created | **No idempotency field exists in `delivery.schema.yaml`** | 🔴 **CRITICAL GAP** |
| **Declined vs failed** | Distinct `REJECTED` (policy) and `FAILED` (operational) | Only `FAILED` / `CANCELLED`; `status.error` for all errors | 🟠 Gap — the reason cannot be conveyed |
| **Inbound message set** | A delivery request, plus a cancellation *request* | `task.assign`, `task.cancel`, `task.pause`, `task.resume`, `system.stop`, `system.pause` | 🟠 SHANA anticipates commands CloudLight must never send |
| **Authentication** | 🔴 **Mutual authentication required (P0)** | **Not mentioned anywhere in the proposal** | 🔴 **CRITICAL GAP** |
| **Versioning** | `schema_version` field required | `schema_version: 1` at document level, `status: DRAFT` | ✅ Consistent in intent |
| **Requester identity** | A reference resolved by CloudLight, never credentials | `requested_by` — "teacher role id"; prohibited fields ban any identifier beyond it | ✅ Consistent |
| **Destination** | `destination_id`, never free text | *"no free-text destination field exists"* by design | ✅ Consistent |
| **Waiting for recipient** | Explicit `AWAITING_CONFIRMATION` | `AWAITING_RECIPIENT` in the schema state enum | ✅ Consistent |
| **Completion** | Requires recipient confirmation; never inferred from arrival | `validation: state == COMPLETED requires recipient_confirmed == true` | ✅ **Strongly consistent — both honour the invariant** |
| **AI-derived work** | Model may propose; deterministic code decides | `model_derived == true requires human_confirmed == true before READY` | ✅ Consistent |
| **Student/staff privacy** | No student-identifying data anywhere | `prohibited_fields` bars staff/student identifiers, behavioural and attendance attributes | ✅ Consistent |
| **When CloudLight is down** | SHANA decides; CloudLight must not pressure it | "Continue current task if safe; otherwise hold; do not start new work" | ✅ Consistent |

### 2.2 Why the two critical gaps matter

**Gap A — no idempotency (🔴 safety-relevant).**

CloudLight's design explicitly requires that a double-tap, a proxy retry, or a
browser retry must **not** produce two robot tasks. SHANA's schema has no field in
which to express that intent. Without a field, a receiver cannot distinguish a
retry from a genuinely new request.

> A second robot task carrying a real load through an occupied corridor, created
> by one tap, is a **physical safety event**, not a data-integrity bug.

This cannot be fixed unilaterally. It needs either a new field in SHANA's schema
or a CloudLight-side guarantee strong enough that retries cannot happen.

**Gap B — no stated authentication (🔴 safety-relevant).**

CloudLight's design requires mutual authentication. SHANA's proposal does not
mention authentication at all.

> An unauthenticated task endpoint means **any host that can reach SHANA's
> interface can request a robot movement.**

Bounded by the absent motor-command path — the worst outcome is an unauthorised
*delivery request*, not motion — but that is still serious and must be closed
before anything is built.

### 2.3 🟠 Divergence C — command surface

SHANA's proposed inbound set includes `task.pause`, `task.resume`, `system.stop`
and `system.pause`. CloudLight's design says it must never express emergency-stop
or safe-stop semantics.

⚠️ **These are reconcilable but must be agreed explicitly.** SHANA is entitled to
offer an emergency-stop channel; CloudLight is entitled to decline to use it. What
must not happen is CloudLight acquiring a pause/resume channel that later becomes
a way to interfere with SHANA's own safety decisions. SHANA's invariant *"no
command bypasses SHANA's own safety validation"* covers this, and CloudLight's
ADR-014 forbids the capability from existing.

**Recommendation:** CloudLight sends **only** `task.assign` and `task.cancel`.
Everything else is either SHANA-internal or out of scope.

---

## 3. 🔴 A hard blocker that is not a schema problem

`data/schemas/destination.schema.yaml` declares:

```
status: { type: enum, required: true,
          values: [ACTIVE, BLOCKED, NEEDS_SURVEY],
          default: NEEDS_SURVEY }
```

with validation:

```
- if status == ACTIVE then verified_by and verified_pose_date are required
- approach_pose.verified must be true before status may become ACTIVE
```

> **Consequence: SHANA currently has zero usable delivery destinations by
> construction.** A destination cannot become `ACTIVE` until it has been
> physically surveyed, its approach pose verified, and the verification dated.

No amount of schema agreement, interface work or CloudLight development changes
this. **No live robot delivery can be demonstrated until a site survey produces at
least one verified destination.** This is SHANA-owned, physical, and on the long
lead-time path.

---

## 4. CloudLight's own API surface (proposed, not built)

⚠️ **None of this exists.** Listed so the third agent can see the intended shape,
not to imply any implementation.

| Endpoint | Purpose | Auth | Notes |
|---|---|---|---|
| `GET /v1/display/{room}/now` | Current and next lesson for a room | None | **Intentionally unauthenticated** — the classroom display |
| `GET /v1/display/{room}/snapshot` | Last-synchronised metadata | None | Powers the staleness indicator |
| `POST /v1/admin/timetable/import` | Upload a timetable document | Admin | Deterministic parse; quarantines on validation failure |
| `GET /v1/admin/substitutions/{id}` | Review a proposed substitution | Timetable staff | Approval gate |
| `POST /v1/admin/substitutions/{id}/approve` | Publish an approved substitution | Timetable staff + second approver where required | 🔴 The only path to publication |
| `GET /v1/resources/search` | Permission-filtered search | Authenticated | Re-verified at open time |
| `POST /v1/auth/challenge` | Begin authentication | — | Individual accounts |
| `POST /v1/auth/action-code` | Authorise **one** action | Authenticated session | Never authenticates a person |
| `POST /v1/deliveries` | Request a SHANA delivery | Authenticated + authorisation | Generates the idempotency key |
| `GET /v1/deliveries/{id}` | Delivery status | Authenticated | Shows last-known status **with its age** |
| `GET /health` | Liveness | None | Feeds the health check |
| `GET /version` | Build identity | None | Feeds update verification |

**Design rules these would obey:**

- `/v1/` prefix from day one, so `/v2/` is possible without breaking deployed
  displays.
- A cached frontend bundle must work against its backend version for a stated
  window, or a backend rollback strands classrooms. See `../ota_updates.md` §5.
- `POST /v1/deliveries` persists the request **before** transmission, so a crash
  cannot lose it.
- No endpoint returns a document the caller cannot open.
- No endpoint exposes robot state beyond what SHANA publishes.

---

## 5. What must happen before any of this is implemented

| # | Action | Owner | Blocks |
|---|---|---|---|
| 1 | **Joint session** between both owners to agree schemas, error codes and timeout semantics | Both | Everything |
| 2 | Add an idempotency mechanism to the contract | Both | Duplicate-task risk |
| 3 | Specify mutual authentication | Both | 🔴 Unauthorised task creation |
| 4 | Agree which inbound commands CloudLight may send | Both | Command-surface creep |
| 5 | Agree a distinct "rejected on policy" outcome | Both | Error clarity |
| 6 | Agree reconciliation semantics after restart | Both | Orphaned or duplicated tasks |
| 7 | Site-survey and verify at least one destination | SHANA | 🔴 Any live delivery |
| 8 | Rotate or re-issue the API key material | Both | Trust establishment |
| 9 | Write the agreed contract into a **shared** file both owners reference | Both | Drift prevention |

⚠️ **SHANA's own document makes recommendation 5: *"Write the invariant list into
the shared contract, not only here."* That has not been done.** The invariants
currently exist in two documents that have never been compared.

---

## 6. Status summary

| Item | Status |
|---|---|
| CloudLight proposal | `DESIGN_ONLY` |
| SHANA proposal | `DESIGN_ONLY` |
| Schemas | Draft, never parsed by code |
| Schema agreement | 🔴 **Not agreed** |
| Authentication requirement | 🔴 **Unstated on SHANA's side** |
| Idempotency | 🔴 **Not expressible in SHANA's schema** |
| Integration testing | 🔴 **`BLOCKED` — cannot test against nothing** |
| Verified destinations | 🔴 **Zero** |
| CloudLight's own API | 🔴 **Does not exist** |

---

## Related

- [`../shana_integration.md`](../shana_integration.md) — CloudLight's full proposal
- [`implementation_inventory.md`](../handoff/implementation_inventory.md) — §4 covers the integration status
- [`unresolved_dependencies.md`](../handoff/unresolved_dependencies.md) — `DEP-SHANA-01` … `11`
- [`ADR-014`](../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md) — the boundary
- `docs/shana/architecture/cloudlight_integration.md` — SHANA's side (read-only)