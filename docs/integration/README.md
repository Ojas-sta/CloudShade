# Integration Layer — Index

**Owner:** Agent 3 (Integration Lead)
**Last updated:** 2026-10-11

---

## 🔴 Read this first

**There is no source code in this repository.** 0 implementation files · 0 tests
· 0 builds · 0 CI · 0 capabilities demonstrated.

> ⚠️ **Since 2026-10-11 there is a public remote with three protected branches and
> one worktree per agent.** Git has been set up; 🔴 **no software has.** See
> [`AGENT_BRANCH_AND_PR_WORKFLOW.md`](AGENT_BRANCH_AND_PR_WORKFLOW.md).

Everything in `docs/` — including this directory — is research, design and
questions. See
[`../shana/overview/current_project_state.md`](../shana/overview/current_project_state.md)
and
[`../cloudlight/overview/current_project_state.md`](../cloudlight/overview/current_project_state.md).

---

## What this directory is

It is the **coordination layer** between two subsystems that were authored
independently, plus the evidence record for exhibition claims.

🔴 Its most consequential content is not architecture. It is the record of
**where the two subsystems disagree** — which, until now, nobody had written
down. See [`unresolved_conflicts.md`](unresolved_conflicts.md).

---

## Index

### Start here

| Document | What it answers |
|---|---|
| [`repository_state_audit.md`](repository_state_audit.md) | 🔴 What actually exists. Verified by command. |
| [`unresolved_conflicts.md`](unresolved_conflicts.md) | 🔴 Where the subsystems contradict each other |
| [`../architecture/architecture_decisions/ADR-015…`](../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md) | 🔴 There is one repository, not three |
| [`../architecture/architecture_decisions/ADR-016…`](../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md) | 🔴 Why `PAUSED` cannot reach a teacher |

### Coordination

| Document | Purpose |
|---|---|
| [`AGENT_COORDINATION.md`](AGENT_COORDINATION.md) | Ownership, permissions, cross-boundary rules, escalation |
| [`TASK_BOARD.md`](TASK_BOARD.md) | 37 tasks; what is `DONE`, what is blocked, and on whom |
| [`AGENT_HANDOFF_PROTOCOL.md`](AGENT_HANDOFF_PROTOCOL.md) | Handoff template — must stand alone |
| [`shared_decisions.md`](shared_decisions.md) | Cross-project decision register |
| [`agent-requests/`](agent-requests/) | 🔴 4 written requests. **0 delivered, 0 acknowledged** |

### Architecture

| Document | Purpose |
|---|---|
| [`system_architecture.md`](system_architecture.md) | Cross-project view, where the subsystems converge and diverge |
| [`interface_contracts.md`](interface_contracts.md) | 🔴 11 entries. **0 `AGREED`, 0 `IMPLEMENTED`** |
| [`security_boundaries.md`](security_boundaries.md) | Trust boundaries, auth, privacy, secrets |
| [`deployment_model.md`](deployment_model.md) | Hosting, network, topology options and constraints |
| [`integration_test_plan.md`](integration_test_plan.md) | 🔴 **No test can be run today.** What becomes possible, and when |
| [`AGENT_BRANCH_AND_PR_WORKFLOW.md`](AGENT_BRANCH_AND_PR_WORKFLOW.md) | 🔴 **Live.** Branches, worktrees, PRs, stacking, approval |

### Exhibition

| Document | Purpose |
|---|---|
| [`../exhibition/claim_evidence_matrix.md`](../exhibition/claim_evidence_matrix.md) | 🔴 What may be claimed, and on what evidence |
| [`../exhibition/evidence_index.md`](../exhibition/evidence_index.md) | 🔴 What evidence exists. 🔴 **Almost none** |
| [`../exhibition/risk_register.md`](../exhibition/risk_register.md) | Exhibition-specific risks |
| [`../exhibition/demonstration_plan.md`](../exhibition/demonstration_plan.md) | PLAN A / B / C |
| [`../exhibition/ANTIGRAVITY_EXHIBITION_HANDOFF.md`](../exhibition/ANTIGRAVITY_EXHIBITION_HANDOFF.md) | 🔴 **Principal deliverable** |

---

## 🔴 The five findings that matter most

Recorded in full in [`unresolved_conflicts.md`](unresolved_conflicts.md). In
brief:

| # | Finding | Why it matters |
|---|---|---|
| 1 | 🔴 **`PAUSED` appears zero times in CloudLight** while SHANA can pause from any state | A paused robot would show a teacher **"On the way"** — violating CloudLight's own invariant 8. `ADR-016` |
| 2 | 🔴 **The two integration documents were written without seeing each other** | Neither agent knew the other's contract existed. This produced #1. `ADR-015` |
| 3 | 🔴 **Documents cite ADR-001, ADR-010, ADR-011 — none of which exist as files** | The safety argument's citations do not resolve. `CONF-04` |
| 4 | 🟠 **6 broken internal links** across the subsystem boundary | Root cause: SHANA's docs moved under `docs/shana/`; CloudLight was written against the old layout. `CONF-03` |
| 5 | ⚠️ **Two state machines, two identifier models, two error philosophies** | An implementer would have to invent every mapping. `IC-02`…`IC-10` |

---

## 🔴 What this directory cannot do

| Cannot | Why |
|---|---|
| Verify any capability | Nothing has been built |
| Demonstrate anything | 0 source files, 0 hardware access |
| Open a pull request | ✅ → your lane; 1 approval required |
| Receive an agent response | ✅ PR comments and commits on `origin` |
| Pass a go/no-go gate | Gates need evidence; evidence needs the school |
| Fix a subsystem defect | 🔴 Agent 3 does not own either subsystem |

---

## The one-line summary

> **Two subsystems have documented their boundary well and their interface
> badly. The boundary is safe. The contract is not agreed. Nothing is built, and
> the questions that would tell us whether it can be built have not been
> answered.**