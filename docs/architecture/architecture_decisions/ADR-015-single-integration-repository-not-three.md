# ADR-015: One integration repository, not three

**Status:** `Accepted`
**Date:** 2026-10-11
**Decides:** whether SHANA, CloudLight and integration work live in separate
repositories or in one
**Decided by:** Agent 3, from direct repository inspection
**Reversal condition:** the owner splits the subsystems into separate
repositories, or a hosting platform makes cross-repository PRs viable — in
which case this is superseded by the topology section of
`docs/integration/GIT_AND_AGENT_WORKFLOW_SPEC.md` §3.2
**Evidence:** `git rev-parse`, `git remote -v`, `git branch -a`, filesystem scan

---

## Context

Agent 3's operating brief describes **three repositories**: a shared integration
repository, a SHANA repository owned by Agent 1, and a CloudLight repository
owned by Agent 2, with cross-repository pull requests flowing between them.

Inspection found **one repository** containing documentation for both
subsystems, **zero remotes**, and **zero source files**.

| Expected | Found | Verified by |
|---|---|---|
| 3 repositories | **1** | `git rev-parse --show-toplevel` |
| SHANA repo separate | **absent** | filesystem inspection |
| CloudLight repo separate | **absent** | filesystem inspection |
| Remotes | **0** | `git remote -v` |
| Cross-repo PRs | **impossible** | no remote, no second repo |
| Code to integrate | **0 files** | `find` over source extensions |

A specification written for the assumed topology would have been fiction. This
ADR records what is true, and what follows from it.

---

## Decision

**There is one repository. The three-agent model operates through directory
ownership and written requests, not through branch isolation and cross-repository
pull requests.**

Specifically:

1. The repository is authoritative for **all** SHANA and CloudLight material.
   Neither subsystem has a separate home.
2. Ownership is enforced by **directory** (`AGENTS.md` §C.1), with Agent 3 as
   the coordinator of cross-project material.
3. 🔴 **There is no push, no remote, and no PR facility.** Any workflow document
   that implies otherwise is wrong about this repository.
4. The workflow for a *future* multi-repository arrangement is **specified but
   not implemented** in `GIT_AND_AGENT_WORKFLOW_SPEC.md` §3.2, and becomes
   active only if the topology changes.

---

## Rationale

**Why not create the repositories the brief assumed?**

The brief's model assumes Agent 1 and Agent 2 already have repositories with
their own remotes. **Neither exists.** Creating them would mean inventing
directory structure, choosing hosting, creating remotes, and moving content
across a boundary — none of which Agent 3 may do:

- 🔴 Remotes and publication are explicitly owner-authorised decisions
  (`AGENTS.md` §D).
- 🔴 No source code exists to separate. Both subsystems are documentation
  trees; splitting them creates two near-empty repositories and one shared
  contract repository with no contract to share.
- Both subsystems cite each other constantly. A hard split would immediately
  break cross-references that are currently one relative hop away.

**Why is a single repository not simply fine?**

Because it has a specific, observed failure mode: **with no branch boundary, the
directory boundary is the only protection.** Nothing mechanically stops Agent 2
from "helpfully" editing `docs/shana/`. That has not happened — both agents
recorded their boundaries explicitly in
`docs/shana/architecture/cloudlight_integration.md` §1 and
`docs/cloudlight/shana_integration.md` — but the risk is structural, so it is
now written into `AGENTS.md` §C.3.

**Why record it as an ADR rather than a note?**

Because the brief, this ADR and any future summary will otherwise disagree, and
the brief's version is the one an incoming agent is most likely to act on. The
discrepancy must be traceable, not just corrected.

---

## Evidence

| Fact | Source | Class |
|---|---|---|
| Single repository root | `git rev-parse --show-toplevel` | `VERIFIED` |
| Branch `main` only | `git branch -a` | `VERIFIED` |
| Zero remotes | `git remote -v` (empty) | `VERIFIED` |
| Zero source files | `find` for `.py .js .ts .tsx .go .c .cpp .java .rs` | `VERIFIED` |
| Zero tests | no test directory | `VERIFIED` |
| Zero CI | no `.github/workflows/` | `VERIFIED` |
| Subsystems co-located | `docs/shana/` and `docs/cloudlight/` both present | `VERIFIED` |
| Both agents documented boundaries independently | `docs/shana/architecture/cloudlight_integration.md`, `docs/cloudlight/shana_integration.md` | `VERIFIED` |

Full audit: [`docs/integration/repository_state_audit.md`](../../integration/repository_state_audit.md).

---

## Consequences

### What becomes easier

- Cross-references between subsystems resolve with one relative hop.
- A contract change is visible to both owners immediately, because there is one
  tree.
- One commit history records *why* a decision was taken, in context.
- No cross-repository synchronisation problem exists.

### What becomes harder

- 🔴 **Directory ownership is the only boundary.** It is a convention, not a
  control. Merge to the integration paths from an agent working in the wrong
  directory is possible.
- 🔴 **Two independent authors produce incompatible artefacts in the same tree,
  and find out late.** This is not hypothetical — it has already happened.
  `docs/shana/architecture/cloudlight_integration.md` says CloudLight was empty
  at its audit date, and `docs/cloudlight/shana_integration.md` says no joint
  session has occurred. Each was written without the other. See `CONF-01`.
- 🔴 **No branch isolation means no review boundary.** Today the only review
  mechanism is a written request.
- 🔴 **Every change is in one history**, so a careless rewrite is
  project-wide.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| **Create three repositories as the brief describes** | 🔴 Requires creating remotes and publishing — owner-authorised actions Agent 3 may not take. No code exists to justify the split. Would break the dense cross-referencing between the two subsystems. |
| **Assume the brief is right and write the spec against it** | 🔴 Produces a document describing a world that does not exist. Every command in it would fail. This is the specific failure mode `AGENTS.md` §E and §R exist to prevent. |
| **Keep one repository but separate by git subtree** | Rejected — premature. There is no code to separate, and subtree operations add failure modes for zero present benefit. |
| **Merge the three agents into one** | Rejected — the split between subsystems is real and valuable. It is enforced by domain, not by repository boundary. |

---

## What this decision does NOT settle

1. 🔴 **Who owns the merge.** With no remote, the owner merges locally or does
   not merge at all. Not specified.
2. **Whether the owner wants a remote at all.** Some single-owner projects
   deliberately never publish. Not the integration lead's decision.
3. **Whether CI should be configured.** There is nothing to check yet. See
   `GIT_AND_AGENT_WORKFLOW_SPEC.md` §9.3.
4. **Which agent arbitrates a disputed claim.** ADR-016 proposes a default; it
   is not accepted until both owners review it.

---

## Follow-up tasks

| # | Task | Owner | Status |
|---|---|---|---|
| 1 | Ask the owner whether a remote and hosting are wanted | Agent 3 → owner | ⛔ `BLOCKED` — needs owner decision |
| 2 | Re-confirm this ADR once a remote exists | Agent 3 | `PENDING` |
| 3 | Enforce directory ownership mechanically (pre-commit hook / CI path filter) once CI exists | Agent 3 | ⛔ `BLOCKED` — no CI |
| 4 | Reconcile the two integration documents against one contract | Agent 1 + Agent 2 | `REQUESTED` — see `agent-requests/` |

---

## Related

- [`ADR-016`](ADR-016-single-sourced-delivery-state-vocabulary.md) — the contract defect this topology allowed
- [`ADR-014`](ADR-014-cloudlight-never-controls-robot-motion.md) — the safety boundary that contract must not weaken
- [`docs/integration/unresolved_conflicts.md`](../../integration/unresolved_conflicts.md)
- [`docs/integration/repository_state_audit.md`](../../integration/repository_state_audit.md)
- [`AGENTS.md`](../../../AGENTS.md) §E