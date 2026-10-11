# Constraints

🔴 **Hard constraints that cannot be traded off, negotiated away, or engineered
around.**

A constraint is not a risk, and not a task. A constraint **removes options**.
Anything proposed in this repository that violates a constraint here is invalid
by definition.

---

## The eight

| # | Constraint | Source | Effect |
|---|---|---|---|
| **C-1** | 🔴 **EDU-or-above only** | Vendor, verbatim | If the unit is PRO or AIR, **no secondary development is possible.** Project ends |
| **C-2** | 🔴 **No floor-identity sensor** | Verified sensor suite | The robot **cannot determine which floor it is on.** No software fixes a missing sensor |
| **C-3** | 🔴 **Stair gait setter removed** (≥1.1.6) | Verified SDK changelog | **Stairs are permanently excluded** for the pilot |
| **C-4** | 🔴 **No sanctioned lift API** | Verified — none exists | Cross-floor transit requires **unlawful modification.** Excluded |
| **C-5** | 🔴 **No autonomous return-to-dock configured** | No dock integration | Recovery is **human-assisted by design** |
| **C-6** | ⛔ **No AI model may reach the robot** | ADR-001 | Deterministic safety layer is **always** in the loop |
| **C-7** | ⛔ **No student identification, ever** | Privacy position | No face recognition, no persistent identifiers, no raw video retention |
| **C-8** | 🔴 **Wired Ethernet for the robot link** | ADR-004 | Wireless is not an acceptable control path |

---

## The consequences, stated plainly

### C-1 — The project may not exist

This is the **first** question, and it is unanswered. If the school's robot is
PRO or AIR, there is no development interface and every document in this
repository is moot.

🔴 **Confirm this before doing anything else.**

### C-2 + C-3 + C-4 together force a single-floor scope

Each constraint alone limits the project. Together they are decisive:

- Cannot sense the floor → cannot know where it is vertically
- Cannot climb stairs → cannot change floor by stairs
- Cannot use lifts lawfully → cannot change floor by lift

> **Therefore SHANA must be single-floor. This is not a scoping preference — it
> is the only physically permissible configuration.**

If the school requires multi-floor delivery, the honest answer is that **SHANA
cannot deliver it**, and no engineering work will change that.

### C-5 — Battery recovery is an operational cost, not a feature

Every low-battery event requires a human. That is a real staffing cost the
school must accept. It should be raised explicitly rather than discovered
during operation.

### C-6 — Architecture is permanently bounded

No amount of model improvement may push inference onto the robot. This is
deliberate: a probabilistic system must not hold the final say on physical
motion around children.

### C-7 — Privacy is not tunable

No amount of educational value justifies identifying children. If a future
proposal weakens this, it is not a SHANA proposal.

---

## ⚠️ Constraints vs. risks vs. assumptions

| Category | Meaning | Can it change? |
|---|---|---|
| **Constraint** | Physics, law, or manufacturer terms | ❌ **No** |
| Risk | Something that might happen | ⚠️ Mitigable |
| Assumption | Something believed but unverified | ⚠️ By verification |

⚠️ **Registering a constraint as an assumption is how projects talk themselves
into impossible plans.** All eight above are hard.

---

## 🔴 Confirm or die

| # | Question | If the answer is bad |
|---|---|---|
| 1 | Is the unit EDU-or-above? | **Project cannot proceed at all** |
| 2 | Is the school willing to accept single-floor scope? | 🔴 Scope must be reduced or the project dropped |
| 3 | Does the school accept human-assisted battery recovery? | 🔴 Operational model is unworkable |
| 4 | Does the school accept wired Ethernet? | 🔴 Control link design changes fundamentally |

⚠️ **These four answers determine whether SHANA is viable at all.** They are
recorded as Gate 1 in
[`../feasibility/go_no_go_gates.md`](../feasibility/go_no_go_gates.md).

---

## Related

- [`../hardware/go2_edu_inventory.md`](../hardware/go2_edu_inventory.md) — C-1, C-2, C-3, C-5
- [`../infrastructure/elevator_feasibility.md`](../infrastructure/elevator_feasibility.md) — C-4
- [`../safety/student_privacy.md`](../safety/student_privacy.md) — C-7
- [`../architecture/architecture_decisions/README.md`](../architecture/architecture_decisions/README.md) — C-6, C-8