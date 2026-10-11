# System Architecture — Cross-Project View

**Owner:** Agent 3 (Integration Lead)
**Status:** 🔴 **`DESIGN_ONLY` — nothing in this document is implemented.**
**Last updated:** 2026-10-11
**Authority:** this document *reconciles* the two subsystems' architectures. It
does not override either.

---

## 🔴 Read this first

**There is no system.** 0 source files · 0 tests · 0 builds · 0 CI · 0
deployments · 0 capabilities demonstrated.

This document shows how the two subsystems *would* fit together, and — more
usefully — **where they currently disagree**. A diagram of a system that does
not exist is easy to mistake for one that does. That is why every box below
carries a status.

---

## 1. Whole-system view

```
┌──────────────────────────────────────────────────────────────────────────┐
│  CLOUDLIGHT — school platform                      STATUS: DESIGN_ONLY    │
│  📄 0 lines implemented                                               📄  │
│                                                                          │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐            │
│  │ Classroom  │ │ Timetable &│ │  Resource  │ │  Authn /   │            │
│  │ dashboards │ │ substitution│ │  browse &  │ │  Authz     │            │
│  │ 📄 spec    │ │ 📄 spec    │ │ 🔴 deferred│ │ 🔴 identity│            │
│  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └─────┬──────┘            │
│        └──────────────┴──────────────┴──────────────┘                   │
│                           modular monolith (ADR-013)                    │
│                        🟡 provisional · no code                        │
│                                                                          │
│   ✋✋✋ NO MOTOR · PATH · SPEED · ACTUATOR COMMAND PATH ✋✋✋              │
└────────────────────────────────┬─────────────────────────────────────────┘
                                 │
                    OUTBOUND: delivery REQUEST only
                    INBOUND:  status only, never a command
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────────┐
│  SHANA — robotics subsystem                 STATUS: DESIGN_ONLY          │
│  📄 0 lines implemented                                               📄  │
│                                                                          │
│  ┌──────────────┐  ┌───────────────┐  ┌──────────────┐                 │
│  │ Deterministic│─▶│ Task state    │─▶│ Navigation / │                 │
│  │ VALIDATOR    │  │ machine       │  │ localisation  │                 │
│  │ 🔴 AI never │  │ 📄 11 states  │  │ 🔴 no floor  │                 │
│  │   reaches it │  │              │  │   sensing    │                 │
│  └──────────────┘  └───────────────┘  └──────────────┘                 │
│          ▲                                                      │        │
│          └────── route safety · battery · hazards · safe stop ◀──┘        │
│                                                                          │
│   🛑 system.stop honoured unconditionally and immediately                │
└──────────────────────────────────────────────────────────────────────────┘
                                 │
                    DEPENDENCY IS OPTIONAL 🔴
                    CloudLight ships and is useful with this absent
```

---

## 2. What the two subsystems actually agree on

🔴 **Worth stating plainly: more than expected.**

Both agents, working **independently and without sight of each other**, arrived
at the same boundary from opposite sides.

| Agreement | SHANA's words | CloudLight's words |
|---|---|---|
| CloudLight assigns **tasks**, SHANA decides **how to move** | *"CloudLight assigns tasks; SHANA decides how to move"* (S1 §4.5) | *"deliver this approved item to this approved destination, within this window"* (S2 §1) |
| Direction is one-way | Outbound: commands. Inbound: status | *"It does not receive a command channel in the reverse direction"* (S2 §3) |
| SHANA may **reject** | *"Reject; never improvise a destination"* (S1 §5) | *"Acceptance is not a right… a normal outcome, not an error"* (S2 §3) |
| CloudLight must not specify SHANA's offline behaviour | *"SHANA must degrade safely without CloudLight"* (S1 §4.6) | *"CloudLight must not specify SHANA's behaviour when CloudLight is down"* (S2 §7.2) |
| Arrival ≠ delivery | *"COMPLETED requires recipient_confirmed == true"* (S4) | *"AWAITING_CONFIRMATION must not be shown as delivered"* (S2 §6) |
| No AI to the robot | Invariant 4.1 (S1 §4) | Invariant 1 (S2 §10) |

🔴 **Independent convergence on a safety boundary is stronger evidence than
agreement reached after negotiation** — it means the boundary is the natural
consequence of the problem, not of negotiation pressure.

This corroborated [`ADR-014`](../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md)
after the fact.

---

## 3. Where they disagree

| # | Area | Conflict | Severity |
|---|---|---|---|
| 1 | Delivery states | 🔴 `PAUSED` unrepresentable in CloudLight | `CONF-02` |
| 2 | Identifiers | `request_id`/`idempotency_key` vs `delivery_id` | `CONF-06` |
| 3 | Item weight | SHANA requires it; CloudLight forbids sending it | `IC-03` |
| 4 | Authentication | CloudLight specifies mTLS; SHANA has not weighed in | `IC-06` |
| 5 | Errors | SHANA: reason codes. CloudLight: prose | `IC-10` |
| 6 | `status.*` messages | Do not match SHANA's own state machine | `CONF-08` |
| 7 | Occupancy | SHANA: human-declared. CloudLight: building intelligence | `CONF-09` |
| 8 | `requested_by` | Role ID vs identity reference | `CONF-10` |

Full detail: [`unresolved_conflicts.md`](unresolved_conflicts.md) ·
[`interface_contracts.md`](interface_contracts.md).

---

## 4. Data flow

### 4.1 What happens when a teacher requests a delivery

🔴 **This is `DESIGN_ONLY`. No step below has been executed.**

| # | Step | Owner | ⚠️ Status |
|---|---|---|---|
| 1 | Teacher authenticates | CloudLight | 🔴 identity source **unknown** (`CL-AUTH-001`) |
| 2 | CloudLight checks policy — who may request what, where, when | CloudLight | 📄 spec; 🔴 no approver named (`CL-SUB-001`) |
| 3 | CloudLight resolves room name → SHANA location ID | CloudLight | 🔴 **no location registry exists** |
| 4 | CloudLight persists the request **before** transmitting | CloudLight | 📄 spec; no database exists |
| 5 | CloudLight sends the request + `idempotency_key` | CloudLight | 🔴 no transport, no auth agreed |
| 6 | **SHANA authenticates the caller** | SHANA | 📄 proposed |
| 7 | **SHANA validates destination exists and is permitted** | SHANA | 📄 proposed |
| 8 | **SHANA applies its own safety rules** — battery, corridor, window, load | SHANA | 📄 proposed |
| 9 | SHANA `ACCEPTS`, `REJECTS`, or `QUEUES` | SHANA | 📄 proposed |
| 10 | SHANA navigates and delivers **itself** | SHANA | 🔴 no code, no hardware |
| 11 | Recipient confirms receipt | Human | 📄 spec |
| 12 | SHANA reports status transitions | SHANA | ⚠️ **vocabulary unreconciled** |

🔴 **Steps 1, 3 and 5 are blocked on unanswered school questions. Steps 4 and 5
have no implementation. Step 11 has no mechanism — there is no arm, no
manipulator, and no confirmed way to detect a recipient.**

### 4.2 ⚠️ Step 11 deserves its own note

SHANA's own documentation is blunt
(`docs/shana/handoff/agent_handoff.md` §3):

> It is **not** an arm or a manipulator. **It cannot pick up, place, or open
> anything.**

So "explicit recipient confirmation" is currently a **human standing at a
stationary robot** pressing something. 🔴 **No mechanism for that interaction
has been designed or validated.** This is a genuine open design question, not
a detail — and it is the state that makes a delivery *a delivery* rather than an
arrival.

---

## 5. Network and hosting

🔴 **Every network fact in this project is `UNKNOWN`.** Neither agent has
contacted school infrastructure, and neither may.

| Question | Status | Consequence |
|---|---|---|
| May a persistent local service run on school premises? | 🔴 Unknown — `CL-SRV-001` | **Gate 1. Everything depends on it.** |
| Can classroom devices reach it? | 🔴 Unknown — `CL-NET-003` | 🔴 The project may fail *silently* |
| Is client/AP isolation on? | 🔴 Unknown — `CL-NET-008` | *"The most likely quiet failure in the whole project"* |
| Is a dedicated robot segment permitted? | 🔴 Unknown | Robot link and classroom LAN must not share a broadcast domain |
| Does a shared identity directory exist? | 🔴 Unknown — `CL-AUTH-001` | 🔴 Authentication has no source |
| Does a NAS exist at all? | 🔴 Unknown — `CL-NAS-001` | 🔴 The entire resource track may close permanently |

⚠️ **A verified fact worth carrying forward:** an access point is a Layer-2
bridge and cannot host an application. *"All smartboards can reach the same
access point"* establishes at most a shared network segment — **not** that a
backend can exist, nor that one device can reach another.

---

## 6. 🔴 What blocks the architecture — and it is not engineering

| # | Blocker | Owner | Gates |
|---|---|---|---|
| 1 | Secondary-development access authorised **in writing**? | School administration | SHANA 1–2 |
| 2 | Exact robot model, firmware, installed options? | Robot custodian | SHANA 1–2 |
| 3 | Which battery — 8,000 or 15,000 mAh? | Robot custodian | SHANA battery |
| 4 | Dedicated robot network segment permitted? | IT | SHANA net |
| 5 | Linux host with a spare Ethernet port? | AI Lab / IT | SHANA net |
| 6 | **Actual stair riser heights?** | Facilities | SHANA vertical |
| 7 | Access points and wall markers permitted? | Administration + IT | SHANA mapping |
| 8 | **Is there a single-floor route to the rooms actually needed?** | Facilities + teachers | SHANA scope |
| 9 | Does the lift manufacturer permit *any* integration? | Johnson, via school | SHANA vertical |
| 10 | May a persistent local service run at all? | Principal + IT | **CloudLight 1** |
| 11 | Can classroom devices reach a host? | IT | CloudLight 2 |
| 12 | Is client isolation enabled? | IT | CloudLight 2 |
| 13 | Who approves a substitution? Who owns the timetable? | Timetable / admin | CloudLight 4 |
| 14 | What Windows edition is on the boards? | IT | CloudLight 2 |

⚠️ **Questions 6, 8 and 9 together determine whether vertical transit is in
scope at all.** If teachers do not need cross-floor delivery, the project
becomes single-floor and substantially more likely to succeed. **That is a good
outcome, not a failure.**

⚠️ **The estimated cost of answering all fourteen: roughly 25 minutes of a
Principal's time and 40 minutes of IT's time.** That is the entire remaining
cost of learning whether this project is possible.

---

## 7. 🔴 Architectural facts that constrain everything

From SHANA's hardware research — all `DOCUMENTED_NOT_HARDWARE_VERIFIED`:

| # | Fact | Consequence |
|---|---|---|
| 1 | Only **EDU-and-above** Go2 models open a development interface | 🔴 PRO and AIR do not |
| 2 | No altitude, pressure, or barometer sensor exists | 🔴 **The robot cannot determine which floor it is on** |
| 3 | Stair gait setter removed from SDK ≥ 1.1.6 | Stairs harder than assumed |
| 4 | Documented max step height **160 mm** | Any riser above this → stairs permanently out |
| 5 | No sanctioned lift API exists | 🔴 Modification would be unlawful |
| 6 | No autonomous return-to-dock | 🔴 Recovery is human-assisted |
| 7 | Go2 documentation states **wired** development connections only | Affects robot link design |

🔴 **Fact 2 is the most consequential and the least discussed.** A robot that
cannot know which floor it is on cannot safely be told to deliver to another
floor — because it cannot verify it arrived on the correct one. This is not a
software problem and no amount of engineering removes it.

---

## 8. Architecture status summary

| Layer | Status | Notes |
|---|---|---|
| System boundary | ✅ **Agreed independently** | Strongest area — see §2 |
| Data flow | 📄 `DESIGN_ONLY` | 12 steps; 3 blocked on school answers, 2 on absent code |
| Contract | 🔴 `PROPOSED` | 0 entries `AGREED` |
| State machine | 📄 `DESIGN_ONLY`, 🔴 unreconciled across subsystems | `CONF-02` |
| Network / hosting | 🔴 `UNKNOWN` | 6 unanswered questions |
| Authentication | 🔴 `UNKNOWN` | No identity source |
| Robot hardware | 📄 `DOCUMENTED_NOT_HARDWARE_VERIFIED` | 🔴 Cannot sense floor |
| Safety architecture | ✅ **Well-argued** | 🔴 **No evidence gathered** |
| Privacy architecture | ✅ **Well-argued** | 🔴 Not legally reviewed |
| Modular monolith | 🟡 `Provisional` | ADR-013 |
| Never command the robot | ✅ **`Accepted`, not reversible** | ADR-014 |
| 🔴 **Implementation** | ⛔ **None** | **0 files** |
| 🔴 **Testing** | ⛔ **None** | **0 tests** |
| 🔴 **Deployment** | ⛔ **None** | **0 systems** |