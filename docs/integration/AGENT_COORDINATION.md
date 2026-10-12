# Agent Coordination Model

**Owner:** Agent 3 (Integration Lead)
**Last updated:** 2026-10-11
**Authority:** this document elaborates `AGENTS.md` §C. Where they conflict,
`AGENTS.md` wins.

---

## 1. The model in one paragraph

Three agents share **one repository**. 🔴 There is **no** SHANA repository and
**no** CloudLight repository. ⚠️ **Since 2026-10-11 there is also one remote and
three protected branches** — see
[`AGENT_BRANCH_AND_PR_WORKFLOW.md`](AGENT_BRANCH_AND_PR_WORKFLOW.md). See
[`ADR-015`](../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md).
Coordination therefore happens through **directory ownership**, **written
requests**, and a **single shared contract** — not through branch isolation or
cross-repository pull requests.

---

## 2. Directory ownership

### 2.1 Authoritative table

| Path | Owner | May change | Consult before changing |
|---|---|---|---|
| `docs/shana/**` | **Agent 1** | Any SHANA material | — |
| `docs/shana/architecture/architecture_decisions/**` | **Agent 1** | SHANA-internal ADRs | — |
| `questionnaires/SHANA_Feasibility_QA.md` | **Agent 1** | Questions, ID rules | — |
| `questionnaires/GEMINI_SHEETS_ONLY_PROMPT.md` | **Agent 1** | Prompt | — |
| `data/schemas/**` | **Agent 1 authored**, shared | 🔴 via §5 interface-change | **Agent 2** — CloudLight consumes |
| `data/templates/**` | **Agent 1** | Templates | — |
| `docs/cloudlight/**` | **Agent 2** | Any CloudLight material | — |
| `questionnaires/Additional_CloudLight_Questionnaire.md` | **Agent 2** | Questions, ID rules | — |
| `questionnaires/GEMINI_CLOUDLIGHT_SHEETS_PROMPT.md` | **Agent 2** | Prompt | — |
| `AGENTS.md` | **Agent 3** | Agentic rules | **All** — it binds everyone |
| `docs/integration/**` | **Agent 3** | Coordination | Both owners for contract content |
| `docs/exhibition/**` | **Agent 3** | Exhibition + handoff | Both owners for claims |
| `docs/architecture/**` | **Agent 3** | Cross-project ADRs | Both owners before `Accepted` |
| `README.md`, `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `LICENSE_REVIEW.md`, `.gitignore`, `.editorconfig` | **Agent 3** coordinates | Governance | The affected subsystem's owner |
| `.github/**` | **Agent 3** | Templates | Both owners |

### 2.2 Why `data/schemas/**` is special

It is the only shared *artefact*. SHANA authored it; CloudLight's proposed
contract depends on it. 🔴 Changing a schema silently would change CloudLight's
inputs without CloudLight knowing. It therefore requires the interface-change
procedure (§5) and **both** owners' awareness — it is not Agent 1's alone.

---

## 3. Who may do what

| Action | Agent 1 | Agent 2 | Agent 3 |
|---|---|---|---|
| Edit own subsystem docs | ✅ | ✅ | ❌ (requests instead) |
| Edit `docs/integration/**` | ❌ (requests instead) | ❌ (requests instead) | ✅ |
| Edit `data/schemas/**` | ✅ via §5 | ✅ via §5 | ⚠️ contract text only |
| Commit locally, own paths | ✅ | ✅ | ✅ |
| Commit locally, another's paths | 🔴 **Never** | 🔴 **Never** | 🔴 **Never** |
| **Push** | 🔴 owner authorisation | 🔴 owner authorisation | 🔴 owner authorisation |
| **Create a remote** | 🔴 Never | 🔴 Never | 🔴 Never — owner decision |
| **Push a feature branch** | ✅ to `origin` | ✅ to `origin` | ✅ to `origin` |
| **Open a PR** | ✅ → `SHANA_main` | ✅ → `CloudLight_Main` | ✅ |
| **Push to `main` / a lane** | 🔴 Protected | 🔴 Protected | 🔴 Protected — needs owner |
| **Merge** | Owner-driven | Owner-driven | ⚠️ advises; owner merges |
| Rewrite history / force-push | 🔴 Never | 🔴 Never | 🔴 Never |
| Publish a decision as `Accepted` | SHANA-internal ADRs | — | Cross-project: 🔴 **only after both owners review** |
| Define the shared contract | Proposes | Proposes | **Publishes, but does not invent** |

---

## 4. Cross-boundary changes

### 4.1 The rule

🔴 **An agent must not silently modify another agent's subsystem.** Not "fixing a
typo". Not "tidying". Not "obvious".

The reason is not territorial. It is that **the owner is the only party who
knows which claims are load-bearing.** An innocuous edit to a safety document
can quietly weaken an argument that has been checked against hardware
documentation.

### 4.2 The three permitted exceptions

`AGENTS.md` §C.3. All three require stating the change in the commit body and
notifying the owner in the handoff.

| # | Exception | Test |
|---|---|---|
| 1 | Broken internal link whose target unambiguously exists | Have you opened the target and confirmed it is the intended document? |
| 2 | Typo with zero semantic content | Could the correction change anyone's understanding? |
| 3 | Correcting a path to a file you have verified | Does the file exist, and is it the right one? |

⚠️ **Exception 1 failed for 1 of the 6 broken links.** `delivery_state_machine.md`
does not exist; the intended target is a *differently named* file. That is not
unambiguous, so it went to the owner. See `CONF-03`.

### 4.3 What always requires a request

🔴 Changing a status label · changing a claim · changing an evidence class ·
changing a boundary or invariant · adding or removing a requirement.

### 4.4 The request mechanism

```
docs/integration/agent-requests/
├── AGENT_1_SHANA_GIT_WORKFLOW_REQUEST.md      Agent 1 — Git workflow + SHANA-owned actions
├── AGENT_2_CLOUDLIGHT_GIT_WORKFLOW_REQUEST.md Agent 2 — Git workflow + CloudLight-owned actions
└── CROSS_REPO_RECONCILIATION_REQUEST.md       Both — joint contract reconciliation
```

⚠️ The reconciliation request is **one joint document**, not two. Reconciling a
contract requires both owners reading the same text; splitting it would recreate
the isolation that produced the conflicts.

A request must be **actionable**: it states what is wrong, where, what the
correct state is, and what evidence supports it. "Please review X" is not a
request.

---

## 5. The interface-change procedure

Triggered by any change to: the delivery state machine · the API surface ·
authentication between subsystems · location naming · the data model · error
handling · the safety boundary.

| Step | Actor | Output |
|---|---|---|
| 1. Propose | Any agent | An entry in [`interface_contracts.md`](interface_contracts.md) |
| 2. Assess impact | Both owners | Subsystem-specific required changes |
| 3. Review | Both owners | `ACCEPTED` / `REJECTED` / `AMENDEND` per entry |
| 4. Record | Agent 3 | ADR with a **reversal condition** |
| 5. Update task board | Agent 3 | Linked task with dependencies |
| 6. Handoff | The changing agent | Uses [`AGENT_HANDOFF_PROTOCOL.md`](AGENT_HANDOFF_PROTOCOL.md) |

🔴 **Steps 1–5 are documentation. Step 6 is not.** No step implies that code
exists — today, none does.

---

## 6. Handoffs

Every handoff must stand alone without access to the previous conversation. Full
protocol: [`AGENT_HANDOFF_PROTOCOL.md`](AGENT_HANDOFF_PROTOCOL.md).

Minimum viable handoff in this repository's current state:

```markdown
## Objective
## Work completed
## Files changed
## Implementation status   (AGENTS.md §Q labels)
## Tests executed          ("None — no runnable code exists")
## Actual test results     ("None — 0 tests exist")
## Decisions made          (ADR IDs, or "none")
## Dependencies            (on the other subsystem, or "none")
## Known limitations
## Unresolved questions
## Required permissions
## Recommended next action
```

🔴 `"No tests run"` is a **complete and acceptable** answer. `"PENDING"` is a
complete and acceptable answer. An invented test result is not.

---

## 7. Communication

🔴 **There is no agent-to-agent messaging channel in this repository.** Agent 1
and Agent 2 do not talk to each other through Agent 3 automatically, and Agent 3
cannot deliver a message to them.

Available mechanisms:

| Mechanism | Status |
|---|---|
| Repository-hosted issues / PRs | ✅ `Ojas-sta/CloudShade` — PRs + protection live |
| Written request in `agent-requests/` | ✅ Used — 🔴 **delivery and acknowledgement are `PENDING`** |
| Task board entry | ✅ Used |
| Conflict register entry | ✅ Used |
| Shared contract entry | ✅ Used |

🔴 **Agent 3 has written three requests. None has been delivered. None has been
acknowledged.** Every one is marked `PENDING`. No acknowledgement is claimed
anywhere in this documentation.

---

## 8. What Agent 3 will not do

| Refusal | Why |
|---|---|
| Resolve a subsystem conflict by editing the owner's files | §4 |
| Publish a cross-project ADR as `Accepted` unilaterally | `AGENTS.md` §M |
| Create a remote, push, or open a PR | `AGENTS.md` §D — owner authorisation only |
| Mark a gate `PASSED` | Gates need evidence, and evidence needs the school |
| Claim a capability is demonstrated | Nothing has been demonstrated |
| Claim an agent acknowledged a request | No delivery mechanism exists |
| Infer test results from passing documentation | `AGENTS.md` §J |
| Collapse two state machines into one by fiat | `ADR-016` is `Proposed`, pending both owners |

---

## 9. Escalation

| Situation | Goes to |
|---|---|
| Ownership dispute | Repository owner |
| Safety-boundary dispute | 🔴 **The more restrictive reading applies immediately**, pending review |
| A gate cannot be passed | The named gate owner in the relevant `go_no_go_gates.md` |
| An agent requests a change in another's subsystem | The change is **specified** by Agent 3; the other agent implements it |
| A capability is claimed without evidence | `docs/exhibition/claim_evidence_matrix.md`; the claim is struck |

---

## 10. Summary of the current coordination state

| Question | Answer |
|---|---|
| How many repositories? | **1** |
| How many remotes? | **1** — `Ojas-sta/CloudShade`, public |
| Remote branches? | **3** — `main`, `SHANA_main`, `CloudLight_Main`, all protected |
| Worktrees? | **3** — one per agent plus the integration root |
| Can agents push feature branches? | ✅ **Yes** |
| Can agents open PRs? | ✅ **Yes** → their own lane |
| Can agents push to a lane or `main`? | 🔴 **No — protected** |
| Can agents commit? | ✅ Yes, within their own paths |
| How many conflicts are open? | **10** |
| How many requests are outstanding? | **3**, all `PENDING` |
| How many have been acknowledged? | 🔴 **0** |
| How many source files exist? | **0** |
| How many tests have run? | **0** |
| How many gates have passed? | **0** |
| How many capabilities are demonstrated? | **0** |

**The coordination system is real. The project is not yet.**