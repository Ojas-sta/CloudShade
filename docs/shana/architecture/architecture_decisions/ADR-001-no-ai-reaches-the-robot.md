# ADR-001: No AI model output ever reaches the robot

**Status:** `Accepted`
**Date:** 2026-10-10
**Decides:** Whether any output of a language model may influence robot motion,
directly or indirectly
**Reversal condition:** 🔴 **This ADR is not reversible by engineering.** Reversing
it requires a different institutional model — a school that accepts probabilistic
control of a physical agent among children. That is a governance decision, not a
technical one. No demonstration success, model improvement, or schedule pressure
may be cited as grounds for reversal.
**Evidence:** `docs/shana/feasibility/decision_log.md` `D-005` · `AGENTS.md` §L.1 ·
`docs/shana/architecture/task_state_machine.md` invariant `I-05` and entry
condition `E-15`

---

## Context

The system brief invites natural-language interaction: a teacher asks for
something to be delivered. A language model is an obvious way to parse that
request.

The failure mode is not hypothetical. A language model is a probabilistic system
whose output depends on a prompt, a context window, and a sampled token
sequence. If such output can reach an actuator, then the physical behaviour of a
robot carrying an item through a school is, in part, a function of a stochastic
process — with no bound on how wrong that function can be, and no guarantee of
reproducibility between two identical requests.

This project operates a physical agent in an environment containing children.

---

## Decision

**No output of a machine-learning model may reach the robot — directly or
indirectly, on any path, in any mode including debug and exhibition.**

Concretely, four rules:

| # | Rule | Enforcement |
|---|---|---|
| 1 | The model emits JSON to a pipe. It has **no network route** to the robot. | Architectural: no route exists to misconfigure |
| 2 | The validator is a **pure function with no I/O**. It cannot act. | Architectural |
| 3 | Unknown entities are **rejected, never guessed**. | Deterministic |
| 4 | 🔴 **Every model-derived task requires human confirmation** before `READY`. | `task_state_machine.md` `E-15` |

🔴 **The demonstration must work with the LLM layer switched off.**

---

## Rationale

**Why the safety argument must not depend on the model.** A safety property
derived from *"the model is usually right"* is not a safety property. It is a
statistical tendency that degrades under exactly the conditions this system
operates in — a novel request, an unfamiliar corridor, an ambiguous request from a
teacher who is in a hurry.

**Why a validator is not sufficient on its own.** A validator that is
well-designed catches malformed output. It does not catch a *well-formed but
wrong* instruction — a real classroom, a plausible destination, a valid-looking
time. That is precisely why rule 4 exists.

**Why the human confirmation is mandatory rather than advisory.** Without it,
rule 1 through 3 protect against malformed input but not against
well-formed-but-wrong input. With it, a human is the last decision point before
motion — which is the only position from which "probably right" is good enough.

**Why route 1 matters more than the validator.** A validator can be misconfigured
away. The absence of a network route cannot. Defence in depth is only real when
one layer is not merely "stronger" but **structural**.

---

## Evidence

| Fact | Class | Source |
|---|---|---|
| The model has no network route to the robot | `DESIGN_ONLY` | this ADR; unimplemented |
| Validator is pure, no I/O | `DESIGN_ONLY` | `decision_log.md` `D-005` |
| `model_derived == true` requires `human_confirmed == true` before `READY` | `DESIGN_ONLY` | `delivery.schema.yaml` `validation` |
| No state transition may be caused by model output | `DESIGN_ONLY` | `task_state_machine.md` `I-05` |
| 🔴 No implementation exists to verify any of this | `VERIFIED` | repository audit 2026-10-11 |

⚠️ **Every rule above is `DESIGN_ONLY`.** There is no code. The evidence that this
holds is the decision itself, not a test result.

---

## Consequences

**Accepted**

- The model can fail without failing the demonstration.
- The safety case is independent of model capability and of model vendor.
- Every delivery is traceable to a human decision.

**Rejected**

- Any direct natural-language-to-motion path.
- Any "assistive" model output reaching an actuator — including speed
  suggestions, path ranking, or hazard scores presented to the executor.
- Any proposal to relax rule 4 for low-risk destinations. ⚠️ **There is no
  low-risk destination.** The robot carries an item through a corridor full of
  children in every case.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Model output validated, then passed to the executor | Validation catches malformed input, not well-formed-but-wrong input. Rule 4 is required regardless. |
| Model used only for destination *ranking*, deterministic filter applies | A ranked list is still a suggestion into the safety path. Rejected as indirect. |
| Model permitted in simulation only | 🔴 **Accepted.** Simulation is a legitimate and encouraged use — it is how the navigation stack is developed at all (ADR-011). The prohibition is on the *live* path. |
| Remove the model entirely | Rejected — natural-language request is a genuine usability goal, and the validator architecture makes it safe enough for its actual role. |

---

## Follow-up tasks

| # | Task | Owner | Status |
|---|---|---|---|
| 1 | Test: inject model output as a transition source; must be rejected | Agent 1 | ⛔ `BLOCKED` — no code |
| 2 | Test: demonstration runs with LLM layer disabled | Agent 1 | ⛔ `BLOCKED` — no code |
| 3 | Verify no network route from model host to robot | Agent 1 | ⛔ `BLOCKED` — no hardware |
| 4 | Obtain written school acknowledgement of this invariant | School | ⛔ `REQUESTED` |

---

## Related

- [`../../safety/operating_restrictions.md`](../../safety/operating_restrictions.md)
- [`../../architecture/task_state_machine.md`](../../architecture/task_state_machine.md) — `I-05`, `E-15`
- [`../../feasibility/decision_log.md`](../../feasibility/decision_log.md) — `D-005`, `D-006`
- [`ADR-011`](ADR-011-simulation-before-hardware.md) — the permitted model use
- `AGENTS.md` §L.1 — 🔴 *"No AI model output ever reaches the robot"*
- [`../../testing/acceptance_criteria.md`](../../testing/acceptance_criteria.md)