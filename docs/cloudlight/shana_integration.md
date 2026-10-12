# CloudLight — SHANA Integration Boundary

**Phase 0 · Documentation only · 2026-10-11**

This document defines the boundary between CloudLight and the SHANA robotics
subsystem. It is an **interface proposal**, not an implementation. No interface
has been agreed with SHANA's authors, no message schema has been validated by
code, and no joint test session has occurred.

⚠️ **Any assumption that SHANA has accepted a schema described here is
unsupported.** This is CloudLight's view of where the line should go. If SHANA's
own authors draw it differently, theirs wins.

---

## 1. The single rule

> **CloudLight may request a delivery. CloudLight may never command the robot.**

Concretely, CloudLight's entire vocabulary toward SHANA is limited to:

- *deliver this approved item to this approved destination, within this window*

CloudLight has **no** representation of, and **no** ability to express:

- motor commands · gait selection · velocity or acceleration
- waypoints, paths, checkpoints, or route segments
- localisation, map corrections, or floor identity
- battery thresholds, charging, or docking
- obstacle avoidance, hazard suppression, or sensor behaviour
- emergency stop, safe stop, or recovery behaviour
- any override, bypass, or "continue anyway" semantics

A low-level command path existing anywhere in the chain — even "just for
debugging" or "just for the exhibition" — converts a software defect or an
intrusion into physical motion in an occupied corridor. This is why the boundary
is expressed as an **absence of capability**, not as a policy.

---

## 2. Who owns what

| Concern | Owner | Rationale |
|---|---|---|
| Navigation | **SHANA** | Only the system with live sensor data can decide what is traversable |
| Localisation and floor identity | **SHANA** | Requires the robot's own sensor fusion |
| Route safety | **SHANA** | Depends on live corridor state the robot observes |
| Battery reserve | **SHANA** | Only the robot knows its own state of charge |
| Hazard handling | **SHANA** | Physical world, not a database |
| Safe stopping | **SHANA** | Must not depend on any network peer |
| Emergency behaviour | **SHANA** | Must never require a partner to be reachable |
| Physical delivery execution | **SHANA** | — |
| Robot-specific state | **SHANA** | — |
| Teacher identity | **CloudLight** | CloudLight owns the staff directory |
| Request authorisation and policy | **CloudLight** | CloudLight owns RBAC |
| Request record and audit | **CloudLight** | Both, but CloudLight holds the authoritative request log |
| Display of status to users | **CloudLight** | — |
| Timetable, substitution, resources | **CloudLight** | Entirely CloudLight's domain |
| UI, search, assistant, updates | **CloudLight** | — |

⚠️ **Safety ownership does not transfer, and cannot be delegated, and is not
subject to a fallback implemented on the CloudLight side.** There is no
"if CloudLight is unreachable, CloudLight will tell the robot to stop" design —
because CloudLight being unreachable is precisely the case in which it cannot
tell the robot anything.

---

## 3. Direction of communication

```
   ┌─────────────────────────────────────────────────────────────┐
   │  CLOUDLIGHT                                              │
   │   • authenticates the teacher                             │
   │   • checks policy (who may request what, where, when)      │
   │   • creates a DeliveryRequest  (idempotent)                │
   │   • persists it in the request log BEFORE sending          │
   │   • sends: high-level request + idempotency key            │
   │   • polls / receives status and displays it               │
   │   • may request cancellation where safe                   │
   └───────────────────────────┬─────────────────────────────────┘
                               │  outbound: request only
                               │  inbound:  status only
                               ▼
   ┌─────────────────────────────────────────────────────────────┐
   │  SHANA  (independent validator)                           │
   │   • authenticates the caller                             │
   │   • validates destination exists and is permitted          │
   │   • validates the request against its OWN safety rules    │
   │   • decides feasibility: battery, corridor, window, load  │
   │   • ACCEPTS, REJECTS, or QUEUES                           │
   │   • executes navigation and delivery itself               │
   │   • reports status transitions                            │
   │   • completes only on RECIPIENT CONFIRMATION              │
   └─────────────────────────────────────────────────────────────┘
```

**Direction matters.** CloudLight sends commands and receives status. It does not
receive a command channel in the reverse direction.

**SHANA may reject any request.** Acceptance is not a right. A rejection is a
normal outcome, not an error condition, and CloudLight must display it as such.

---

## 4. Request contract (proposed, unagreed)

⚠️ **`DESIGN_ONLY`. Not agreed by SHANA. Never validated.** See
[`api_contracts.md`](architecture/api_contracts.md) §2 for the
recorded divergence between the two independently drafted proposals.

A request carries the minimum needed for SHANA to make its own decision:

| Field | Purpose | Notes |
|---|---|---|
| `delivery_id` | Client-generated unique identifier | 🔴 **Aligned to SHANA's field name**, not `request_id` — see R11 below |
| `idempotency_key` | Deduplicates retries of the same intent | 🔴 **NOT PRESENT IN SHANA'S SCHEMA.** Open gap |
| `origin_id` | Source location | SHANA's name; a location SHANA recognises |
| `destination_id` | Target location | Never free text — by design on both sides |
| `item_category` | Class of item, not contents | 🔴 **Weight is deliberately NOT transmitted** — see below |
| `requested_by` | 🔴 **Role ID only** (e.g. `teacher`) | Not credentials; not a secret; **not an identity reference** |
| `requested_window` | Earliest / latest acceptable | SHANA may refuse outside its operating hours |
| `schema_version` | Version of this message | Enables graceful degradation |

### 4.1 🔴 Item weight — conflict recorded, not resolved

| CloudLight's original position | SHANA's schema |
|---|---|
| Weight **forbidden**: *"payload dimensions, weights"* | `item_weight_kg` is **required** |

⚠️ **Both positions are defensible and this is not CloudLight's to settle.**
SHANA's is the safer physical position — a robot attempting to carry an item it
cannot lift is a hazard, and only the robot should know its own limits. CloudLight's
is the cleaner boundary — a teacher must not be able to influence a physical
parameter.

**CloudLight's position:** 🔴 **category only on the wire**, with SHANA applying
its own payload policy from onboard sensing and its own data. If SHANA requires a
weight field, CloudLight will supply a **category-derived conservative bound, not a
teacher-entered figure**.

🔴 **Requires both owners.** Recorded as `IC-03` in
`docs/integration/unresolved_conflicts.md`. Unresolved.

### 4.2 `requested_by` — role ID accepted

⚠️ **CloudLight accepts Agent 3's recommendation: transmit a role ID, not an
identity reference.** A role ID (`teacher`) carries no personal data, whereas a
directory object ID is a personal identifier travelling to a second system for no
operational benefit. This is consistent with SHANA's own `prohibited_fields`, which
bars *"any student or staff identifier beyond the requesting role id"*.

### 4.3 What is deliberately absent

Motor parameters · route hints · speed preferences · any "priority" that could
override safety · teacher-entered weight · any free-text destination · any field
that would let a caller influence SHANA's navigation.

**The request is persisted before transmission.** If SHANA never receives it, the
record still exists. If SHANA receives it and CloudLight crashes, reconciliation
recovers the state. Send-then-record loses requests silently, and a lost request
in a delivery workflow is a failed delivery nobody notices.

---

## 4.4 🔴 API shape — divergence is larger than a base path

⚠️ **Agent 3 framed this as `/v1` vs `/api/v1`. That framing is too small.**

| | CloudLight drafted | SHANA drafted |
|---|---|---|
| Style | REST (`POST /v1/deliveries`) | JSON-RPC-style messages (`task.assign`, `status.en_route`) |
| Addressable resource | A delivery object with a URL | No URL; a task handle plus a status stream |
| Status delivery | Polling `GET .../{id}` | Published `status.*` messages |

⚠️ **No base path reconciles these.** They are two different interaction models.

**CloudLight's position:** 🔴 **REST over HTTP/JSON, with `status.*` names
reused verbatim as the `status` enum values.** This keeps SHANA's vocabulary —
the part ADR-016 protects — while letting CloudLight use ordinary HTTP. SHANA
would implement a thin REST adapter over its existing state machine.

⚠️ **Requires both owners.** SHANA may reasonably prefer its message model
outright. Recorded as `IC-05`. **Unresolved.**

---

## 5. Idempotency, retries and duplicates

A teacher tapping a button twice, a proxy retrying a request, or a browser
retrying after a timeout must **not** produce two robot tasks.

Rules:

1. The client generates a **`delivery_id`** before the first transmission
   attempt.
2. Retries of the same intent reuse the same `delivery_id`.
3. SHANA treats a repeated `delivery_id` as the same request, returns the
   existing state, and **does not create a second task**.
4. CloudLight reconciles on startup: any request in `SENT` with no terminal
   state is re-queried by `delivery_id` rather than re-sent as a new request.

### 5.1 🔴 Identifier decision — one identifier, not two

⚠️ An earlier draft carried **both** `request_id` and `idempotency_key`. That
gives one intent two identities and invites exactly the desynchronisation that
causes duplicate tasks.

**CloudLight's position:** 🔴 **one identifier, `delivery_id`, which doubles as
the idempotency key.** This also reconciles the field-name divergence with SHANA,
whose schema already uses `delivery_id`.

🔴 **`delivery_id` must be a non-sequential opaque identifier (UUIDv4/v7).** A
guessable sequential ID is not a safe idempotency key: it leaks delivery volume
and lets one caller address another's task.

⚠️ Requires SHANA's confirmation that its `delivery_id` is already treated as an
idempotency key. Recorded as `IC-04`. **Unresolved.**

⚠️ **Two physical tasks from one tap is a physical safety event, not a data
bug** — a second robot moving through a corridor carrying an item nobody asked
for. Idempotency is treated as a P0 control, not an optimisation
(`CL-SEC-016`, `CL-SHANA-018`).

---

## 6. Status vocabulary — a declared projection of SHANA's state machine

🔴 **SHANA owns the state machine.** CloudLight does not define a parallel
vocabulary. CloudLight holds only a **read-only projection**: a mapping from
SHANA's reported states to display strings, defined in
[`ADR-016`](../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md).

**Source of truth:** SHANA's `delivery.schema.yaml` state enum and
`docs/shana/architecture/task_state_machine.md`.

### 6.1 🔴 Correction — `PAUSED` was missing

⚠️ **This section previously mapped every executing state to `IN_PROGRESS`,
displayed as "On the way". SHANA can enter `PAUSED` from *any* state.** The
result was that a robot halted by an operator, holding position and waiting for
a human, would have been shown to a teacher as **"On the way"** — a false
statement about the physical state of a machine carrying an item through a
school, and a direct contradiction of this document's own invariant 8.

That defect is now corrected. `PAUSED` is explicit.

### 6.2 The projection — keyed to messages SHANA actually publishes

🔴 **This maps the messages SHANA really publishes**, taken from SHANA's
`task_state_machine.md` §State → published message mapping. An earlier draft of
this section invented display states (`MOVING_TO_PICKUP`, `IN_TRANSIT`) that
**SHANA never emits** — precisely the kind of parallel vocabulary ADR-016
forbids. Corrected.

| SHANA publishes | Covers state(s) | CloudLight shows | Teacher must act? |
|---|---|---|---|
| *(nothing yet)* | not yet sent | "Requesting…" | No |
| `status.assigned` | `QUEUED` · `VALIDATING` · `READY` | "Accepted — preparing" | No |
| `status.en_route` | `NAVIGATING_TO_PICKUP` · `NAVIGATING_TO_DESTINATION` | "On the move" | No |
| `status.awaiting_loading` | `AWAITING_LOADING_CONFIRMATION` | "Waiting for the item to be loaded" | 🔴 **Yes — a person must load** |
| `status.awaiting_recipient` | `AWAITING_RECIPIENT` | "At the destination — waiting for confirmation" | 🔴 **Yes — go to the robot now** |
| `status.paused` | `PAUSED` | "Paused — a person is needed" | 🔴 **Yes — operator action** |
| `status.blocked` | overlay on any state | "Route blocked — a person is needed" | 🔴 **Yes — operator action** |
| `status.estop` | out-of-band, takes precedence | "Emergency stop" | 🔴 **Yes** |
| `status.delivered` | `COMPLETED` | "Delivered" | No |
| `status.cancelled` | `CANCELLED` | "Cancelled" | No |
| `status.error` | `FAILED` | Failure reason in plain language | Depends on reason |
| `status.ready` | ⚠️ **not a delivery state** | Availability indicator only | No |
| *(no update / unrecognised)* | unknown | "Status unavailable" **+ age** | Re-request later |

### 6.3 🔴 Known, accepted projection loss

⚠️ **CloudLight cannot tell the pickup leg from the delivery leg.** SHANA
publishes `status.en_route` for both `NAVIGATING_TO_PICKUP` and
`NAVIGATING_TO_DESTINATION`, so "going to collect it" is not observable from the
wire.

**CloudLight accepts this loss.** It passes the homogeneity test: in both states
the robot is moving and **nothing is required of the teacher**. Had the two
different teacher obligations, CloudLight would object.

⚠️ If the school later wants "going to collect it" shown, that is **SHANA's
change to make** — SHANA would have to publish the legs separately. CloudLight
must not infer it.

### 6.4 🔴 States that must never be collapsed

⚠️ **These are the states where a human must act. Merging any of them into a
coarse "in progress" tells a teacher the opposite of the truth.** This is SHANA's
rule set `M-1`…`M-4` applied on the CloudLight side:

| Must stay distinct | Because |
|---|---|
| **`status.paused`** vs `status.en_route` | "Paused, a person is needed" and "on the move" demand **opposite** responses |
| `status.awaiting_recipient` vs `status.en_route` | One means **wait**; the other means **go there now** |
| `status.awaiting_loading` vs `status.en_route` | A person is needed at the pickup |
| `status.estop` | Out-of-band; 🔴 takes precedence over **every** state |
| unknown / stale vs anything live | A stale status must never be rendered as current |

**Projection rule:** several SHANA states may share one display string **only if**
the collapsed set is homogeneous with respect to *what the teacher must do*.
`QUEUED`/`VALIDATING`/`READY` and the two travel legs pass that test. The rows
above do not.

### 6.4 `REJECTED` — unresolved

🔴 **`REJECTED` has no SHANA counterpart.** SHANA's machine routes a validator
rejection to `CANCELLED`, so today a refusal and a withdrawal are
indistinguishable to CloudLight.

⚠️ CloudLight **will not invent a terminal state SHANA cannot emit.** Until SHANA
either adds a distinct rejection state or supplies a mapping rule, `REJECTED`
stays in the design as `DESIGN_ONLY` and is **not** displayed. This is recorded as
a live conflict in `docs/integration/unresolved_conflicts.md` (`CONF-02`).

### 6.5 Rules

1. 🔴 **Never render a state SHANA did not report.** No inference, ever.
2. 🔴 **`UNKNOWN` and `UNAVAILABLE` are always shown with the age of the last
   known state.** A bare stale "On the way" causes a teacher to wait for a robot
   that already stopped.
3. 🔴 **`AWAITING_RECIPIENT` is never shown as delivered.** Completion requires
   explicit recipient confirmation and is never inferred from arrival. An item
   left in a closed room is not a delivery. See SHANA's `delivery.schema.yaml`
   validation rule.
4. **The projection is versioned and reviewable.** It changes through the
   interface-change procedure (`AGENTS.md` §G), never by editing one subsystem
   silently.
5. 🔴 **No integration may be implemented until this projection is agreed by both
   owners.** There is nothing to build against an ambiguous contract.

---

## 7. Failure behaviour in both directions

### 7.1 SHANA is offline or unreachable

| Aspect | Behaviour |
|---|---|
| CloudLight dashboard | **Unaffected.** Full timetable, substitutions, resources, search. |
| Robot panel | Neutral unavailable state, with the age of the last known status |
| New requests | Rejected locally with a clear message: robot subsystem unavailable |
| In-flight requests | Left in their last known state, shown with age, never assumed complete |
| Human spotter | SHANA's own protocol applies; CloudLight has no role |

**This is the single most important availability property in the document.** A
robot failure must never blank a classroom screen.

### 7.2 CloudLight is offline or unreachable

| Aspect | Behaviour |
|---|---|
| Robot behaviour | **Undefined by CloudLight, and must be.** SHANA decides; it must not require a peer. |
| Safety | Must never depend on a network peer's availability |
| Task completion | Continues or safely stops **according to SHANA's own documented policy** |
| Recipient confirmation | Sourced from the destination, not from CloudLight |

⚠️ **CloudLight must not specify SHANA's behaviour when CloudLight is down.** That
is SHANA's decision to make and document, and it may reasonably differ from what
CloudLight would prefer. The only CloudLight-side requirement is that **SHANA
retains its authority**, and that nothing in CloudLight's design creates
pressure on SHANA to degrade safety to keep a request moving.

### 7.3 Server restart

| Aspect | Behaviour |
|---|---|
| CloudLight | On startup, reconcile all non-terminal requests by `delivery_id` |
| SHANA | On restart, report the true state of in-flight tasks |
| Orphan detection | A request with no counterpart on either side is flagged for a human, never auto-retried as new |

### 7.4 Robot disconnection mid-task

CloudLight shows the last known state **with a visible age**. It does not
re-send. It does not cancel. It does not guess. A human decides.

---

## 8. Cancellation semantics

Cancellation is a **physical action with preconditions**, not a database update.

- Cancellation is requested by CloudLight but **decided by SHANA**.
- SHANA may refuse a cancellation, and refusal is a legitimate outcome.
- A cancellation arriving after pickup may be refused, or honoured with an
  **item return**, which is a second physical movement. Which one applies is
  SHANA's policy, not CloudLight's preference.
- A cancellation accepted by CloudLight's UI is not a cancellation until SHANA
  confirms it. The UI must show `CANCEL_REQUESTED`, not `CANCELLED`, until then.

⚠️ **CloudLight must never display "Cancelled" before SHANA has confirmed
cancellation.** Showing a cancelled state for a robot still carrying an item is
a safety-relevant lie.

Only authorised roles may request cancellation (`CL-SHANA-006`, `CL-SHANA-013`,
`CL-SHANA-014`).

---

## 9. Request authentication

| Aspect | Requirement |
|---|---|
| Caller authentication | CloudLight must be authenticated to SHANA, and vice versa |
| Basis | Mutual TLS, or a signed request token with a short expiry |
| Why not a shared static key alone | Single-key systems are single points of compromise |
| Non-negotiable | An unauthenticated task endpoint lets **any host on the network move the robot** |
| Identity of the teacher | Travels as a *reference*, resolved by CloudLight; never as a credential SHANA must trust |

⚠️ **This is a P0 control.** Without mutual authentication, any device that can
reach SHANA's interface can create tasks. Combined with the absent motor-command
path, the worst case is still "an unauthorised delivery request" — which is
serious, but bounded. It is far better than the alternative.

See `CL-SHANA-015` and `CL-SHANA-017`.

---

## 10. What must never happen

These are the invariants. Each one, if violated, invalidates the safety argument
of the entire system.

1. **No AI model output reaches the robot**, directly or indirectly. CloudLight's
   assistant may *prepare* a proposal; a deterministic backend validates it.
2. **No motor, path, speed or actuator command path exists in CloudLight**, in
   any mode, including debug, admin and exhibition.
3. **CloudLight cannot bypass, delay, or override a SHANA stop.** Stop semantics
   are SHANA's alone and are honoured unconditionally.
4. **SHANA's safety checks are never skipped, sampled, or made conditional** on
   CloudLight's state.
5. **CloudLight is never in SHANA's safety path.** If CloudLight is down, no
   safety function may be affected.
6. **No raw robot video ever leaves the robot**, and no person is ever
   identified. See `docs/safety/student_privacy.md`.
7. **Delivery completion requires explicit recipient confirmation**, never
   inference from arrival.
8. **CloudLight never presents an inferred or stale status as current.**

⚠️ Invariants 1–3 are inherited from SHANA's own accepted architecture
decisions — SHANA's [ADR-001](../shana/architecture/architecture_decisions/ADR-001-no-ai-reaches-the-robot.md) (no AI model reaches the robot) and [ADR-003](../shana/architecture/architecture_decisions/ADR-003-stairs-and-lifts-excluded.md) — and are **not** CloudLight's to relax.

---

## 11. Open questions

| # | Question | Blocks | Who can answer | If unanswered |
|---|---|---|---|---|
| 1 | What robot interface is actually available and permitted? | The integration existing at all | Robot custodian + vendor | No integration; CloudLight ships without it |
| 2 | What identity must accompany a request? | Request schema | Robot custodian + vendor | Send a non-personal reference |
| 3 | Is there a shared identity store across CloudLight and SHANA? | Offboarding correctness | IT | Two stores; drift risk accepted and logged |
| 4 | What is SHANA's policy when CloudLight is unreachable? | Failure semantics | Robot custodian | SHANA decides; CloudLight adapts |
| 5 | How does SHANA report in-flight tasks after restart? | Reconciliation | Robot custodian | Manual reconciliation; slower recovery |
| 6 | What are SHANA's own safety-critical operating windows? | Request validation | Robot custodian | CloudLight refuses all requests outside school hours |
| 7 | Is cancellation permitted mid-task, and what happens after pickup? | Cancellation design | Robot custodian | No cancellation offered in CloudLight |
| 8 | Is SHANA's network segment reachable from the CloudLight segment? | Deployment topology | IT | SHANA isolated; integration deferred |

Full question set: `questionnaires/Additional_CloudLight_Questionnaire.md`,
category `CL-SHANA` (`CL-SHANA-001` … `CL-SHANA-024`).

---

## 12. Recommendation

> **Build CloudLight without the SHANA integration, and add it last.**

The timetable, substitution, resource and search features are useful on their own,
carry most of the school's likely value, and depend on none of the eight open
questions above. Building them first also means that when SHANA integration
eventually happens, it is a bounded addition to a working system rather than a
load-bearing component discovered to be unreachable.

⚠️ **A SHANA integration that is unreachable, rejected, or refused would take
every classroom screen down with it if the dependency runs the other way.** The
dependency arrow must point from CloudLight to SHANA, and it must be optional.

---

## Sources

This document is derived from internal architecture analysis of this repository.
Its two cross-referenced SHANA documents are cited below.

| # | Document | Location | Class | What it supports |
|---|---|---|---|---|
| 1 | SHANA architecture | `docs/architecture/architecture.md` | INTERNAL | Evidence-label conventions; local-server principle; the deterministic-validator rule |
| 2 | Delivery state machine | `docs/architecture/delivery_state_machine.md` | INTERNAL | Recipient-confirmation rule; the "last known position is not current position" principle |
| 3 | Student privacy | `docs/safety/student_privacy.md` | INTERNAL | No-identification invariant; no raw video retention |
| 4 | No AI model may reach the robot | SHANA [ADR-001](../shana/architecture/architecture_decisions/ADR-001-no-ai-reaches-the-robot.md) `D-005` |
| 5 | School network | `docs/infrastructure/school_network.md` | INTERNAL | The robot link is a separate wired Ethernet path, not the CloudLight LAN |

⚠️ **No external vendor or robot-documentation source is cited in this document.**
The SHANA-facing interface described in §4 is CloudLight's own proposal, derived
from CloudLight's requirements. It has **not** been checked against any vendor
SDK documentation and must not be treated as compatible with one.