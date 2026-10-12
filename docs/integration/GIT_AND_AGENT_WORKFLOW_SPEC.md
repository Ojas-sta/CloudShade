# GIT AND AGENT WORKFLOW SPECIFICATION

**Status:** `SPECIFIED` — implemented in part (see the status column throughout)
**Owner:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Applies to:** this repository, now; any future subsidiary repository, later

> 🔴 **Read this before assuming anything about Git in this project.**
> The operating brief described three repositories. **There is one.** This
> specification documents reality first (§2), then specifies what happens when
> that changes (§3+). Anything described as `SPECIFIED` here is a *requirement*,
> not a report that something exists.

---

## 0. Status vocabulary for this document

| Label | Meaning |
|---|---|
| `SPECIFIED` | The required behaviour is documented here. Not yet true of the repository. |
| `IMPLEMENTED` | The change exists and has been verified |
| `VERIFIED` | Inspected or tested |
| `BLOCKED` | Cannot proceed. Blocker named in §12 |
| `NOT APPLICABLE` | Does not fit the current topology |

---

## 1. Purpose

Agents 1 and 2 must be able to work **independently** on two subsystems while
remaining **compatible**. This document specifies the Git and workflow discipline
that makes that possible without either agent needing permission to touch the
other's files.

It is **not** a style guide. It exists to prevent three specific failures that
have already been observed or are structurally likely:

1. Two agents inventing incompatible interfaces and discovering it late.
2. One agent rewriting another's subsystem to force compatibility.
3. Claims of completion that do not match what was built or run.

---

## 2. Repository topology — what is actually true

Verified by command inspection. Full evidence:
[`repository_state_audit.md`](repository_state_audit.md).

```
┌──────────────────────────────────────────────────────────────────┐
│  ONE REPOSITORY  —  <repo root>                              │
│                                                                  │
│  ├── docs/shana/**         Agent 1   (SHANA)                     │
│  ├── docs/cloudlight/**    Agent 2   (CloudLight)                │
│  ├── docs/architecture/**  Agent 3   (cross-project ADRs)        │
│  ├── docs/integration/**   Agent 3   (coordination)               │
│  ├── docs/exhibition/**    Agent 3   (exhibition + handoff)      │
│  ├── data/schemas/**       Agent 1 authored, shared               │
│  ├── questionnaires/**      Agent 1 + Agent 2                     │
│  └── root governance files  Agent 3 coordinates                   │
└──────────────────────────────────────────────────────────────────┘
```

| Fact | Value | Verified by |
|---|---|---|
| Repositories | **1** | `git rev-parse --show-toplevel` |
| Branches | `main` only | `git branch -a` |
| Remotes | **1** — `origin` → `Ojas-sta/CloudShade` | `git remote -v` |
| Remote branches | **3** — `main`, `SHANA_main`, `CloudLight_Main` | `git ls-remote --heads origin` |
| Visibility | `PUBLIC` | `gh repo view --json visibility` |
| Worktrees | **3** | `git worktree list` |
| Branch protection | **all 3 protected** — 1 review, admins enforced, no force-push, no deletion | `gh api .../branches/{b}/protection` |
| CI / hosting | hosting ✅ · CI still **none** | no `.github/workflows/` |
| Source code | **0 files** | filesystem scan |
| Tests | **0** | no test directory |
| Commits | **1** (`3a01fa9`, the Phase A baseline) | `git log --oneline` |

### 2.1 Consequences of a single repository

🔴 **There is no cross-repository PR to open, because there is no second
repository.** The brief's model — Agent 1 PRs into the integration repo, Agent 2
PRs into the integration repo — reduces to:

> Both agents work in one tree, under **directory ownership**, and reconcile
> through **written requests and a shared contract**, not through branch
> isolation.

This makes the ownership rule (§5 of `AGENTS.md`) *more* important, not less.
With no branch boundary to stop an agent, the directory boundary is the only
thing preventing one subsystem's documents from being edited by the other's
owner. **Do not treat "it's the same repo" as licence to edit across the line.**

### 2.2 Authority per subsystem

| Subsystem | Authoritative for | Not authoritative for |
|---|---|---|
| SHANA | Robot behaviour, safety, navigation, localisation, battery, hazards, delivery state machine, hardware, the robot's own vocabulary | Teacher identity, timetable, resources, UI |
| CloudLight | Timetable, substitutions, resources, search, teacher identity, authorisation, UI, update mechanism, display | Robot motion, route safety, battery, floor identity, hazard handling |
| Integration | The **contract between them**, the task board, evidence, the exhibition handoff | Either subsystem's internal design |

🔴 **Where the two disagree about who owns a concept, the subsystem that owns
the physical or safety consequence wins.** See §8.

---

## 3. Git autonomy — what each agent may do

### 3.1 Today (single repository)

| Agent | May | Must not |
|---|---|---|
| **Agent 1** | Stage, commit and branch **within `docs/shana/**`, `questionnaires/SHANA_*`, `data/schemas/**`** | Commit another agent's paths; push; add a remote; rewrite history |
| **Agent 2** | Stage, commit and branch **within `docs/cloudlight/**`, `questionnaires/*CLOUDLIGHT*`** | Same |
| **Agent 3** | Commit within `docs/integration/**`, `docs/exhibition/**`, `docs/architecture/**`, `AGENTS.md`; coordinate merges | Edit `docs/shana/**` or `docs/cloudlight/**` substantively without a request |

**Everyone:** 🔴 **`main`, `SHANA_main` and `CloudLight_Main` are protected.** A
direct push is refused by GitHub. Changes reach them only via a PR carrying
one approving review.

🔴 **Still owner-authorised, and not enforced by Git:** tag publication, remote
creation, force-push, deletion, and changing branch protection itself.

⚠️ **The owner granted the remote on 2026-10-11.** Phase A predates that; its
"0 remotes" finding remains a correct record of that moment. See
[`AGENT_BRANCH_AND_PR_WORKFLOW.md`](AGENT_BRANCH_AND_PR_WORKFLOW.md).

### 3.2 If separate repositories are ever created

`SPECIFIED` — this is a standing design, not a current instruction.

| Concern | Rule |
|---|---|
| Agent 1 autonomy | ✅ full commit + push + PR + stacked PR in the SHANA repo |
| Agent 2 autonomy | ✅ full commit + push + PR + stacked PR in the CloudLight repo |
| Integration repo | Agent 3 maintains it; Agents 1 and 2 are **not** required to commit or push there as part of their normal workflow |
| Cross-repo change | Agent 3 raises a **specification** or a **request**; it never claims to have made a change it did not make |
| Contract changes | Land in the integration repo **first**, as a versioned contract, before either implementation proceeds |
| Agent 3 write access | 🔴 Never assumed. Agent 3 works through review, not direct edit |

⚠️ **Making both agents push to a shared integration repo is a mistake.** It
couples two release cycles and turns every documentation merge into a
coordination event. The integration repo should hold **contracts and evidence**,
not code.

---

## 4. Branch naming

### 4.1 Scheme

`<type>/<scope>-<subject>`, lower-case, hyphenated.

| Type | Use |
|---|---|
| `feature/` | new capability |
| `fix/` | defect in existing work |
| `docs/` | documentation only |
| `research/` | investigation, sources, evidence |
| `integration/` | cross-project contract, task board, evidence |
| `chore/` | tooling, config, repo hygiene |

`<scope>` is **mandatory** and is `shana`, `cloudlight`, `integration`, or
`exhibition`.

### 4.2 Examples

```
docs/shana-hardware-inventory-corrected
docs/cloudlight-integration-boundary
fix/cloudlight-broken-shana-cross-references
research/shana-lift-manufacturer-response
integration/delivery-state-vocabulary
docs/exhibition-antigravity-handoff
chore/repo-observe-gitignore
```

### 4.3 Existing branches

🔴 **Never rename an existing branch to fit this scheme.** There is one branch,
`main`. It is the integration baseline and it keeps its name. Renaming is
history rewriting for no benefit.

If a future branch predates this scheme and is **unmerged**, renaming it is
acceptable **only** with owner authorisation. If it is **merged**, leave it and
apply the scheme going forward.

---

## 5. Commit conventions

### 5.1 Format

```
<type>(<scope>): <imperative summary, ≤ 72 chars>

<why the change was needed — not what the diff shows>

Refs: <TASK-ID | ADR-NNN | QUESTION-ID>
Status: <IMPLEMENTED_AND_TESTED | DESIGN_ONLY | RESEARCH_ONLY | UNKNOWN>
```

### 5.2 Examples

```
docs(integration): single-source the delivery state vocabulary

CloudLight's status vocabulary could not represent SHANA's PAUSED state,
so a paused delivery would display as "On the way" — contradicting
CloudLight's own invariant 8. Recorded as CONF-02 and fixed by ADR-016.

Refs: ADR-016, CONF-02
Status: DESIGN_ONLY
```

### 5.3 Rules

| Rule | Why |
|---|---|
| One logical change per commit | A revert must be possible |
| The body explains **why** | The diff already shows what changed |
| Every commit declares a `Status:` | 🔴 Makes over-claiming structurally difficult |
| Reference task/ADR/question IDs | Traceability across agents |
| 🔴 **Do not rewrite history to match this style** | Existing history has value; rewriting destroys other agents' context |
| 🔴 **Never** stage another agent's paths to "tidy up" a commit | It hides the change from its owner |

⚠️ Conventional Commits are a *convention*, not a gate. No tool enforces this and
none should be added until CI exists.

---

## 6. Pull requests

### 6.1 Availability

✅ **`IMPLEMENTED`.** The remote is `Ojas-sta/CloudShade` (public). PRs can be
opened against `SHANA_main`, `CloudLight_Main` or `main`, and all three are
protected so a PR plus one approving review is required.

🔴 **Status checks remain `NONE`** — there is no CI and nothing to check, so
the review *is* the gate. See
[`AGENT_BRANCH_AND_PR_WORKFLOW.md`](AGENT_BRANCH_AND_PR_WORKFLOW.md).

`SPECIFIED` — the following applies the moment a platform exists.

### 6.2 Required PR body

Every meaningful PR states:

| Field | Content |
|---|---|
| **Summary** | What changes, in one sentence |
| **Motivation** | Why it is needed now |
| **Scope** | What it explicitly does *not* do |
| **Files/components** | Paths |
| **Dependencies** | Base PR / branch; issue IDs |
| **Tests run** | The exact commands |
| **Test results** | 🔴 The **actual output**. "Not run — no runnable code exists" is a valid and complete answer |
| **Documentation changed** | Paths |
| **Known limitations** | What remains unaddressed |
| **Security / safety implications** | Or "none", explicitly |
| **Screenshots / evidence** | Only real evidence |
| **Reviewer requirements** | Who must read it, and why |
| **Breaking changes** | Or "none" |
| **Rollback** | How to undo safely |

### 6.3 Rules

- 🔴 **A PR may not claim a feature is tested because the code compiles.**
- 🔴 **A PR may not claim a feature is implemented because a document describes
  it.** The doc is `DESIGN_ONLY`; the PR does not change that unless code lands.
- A PR touching `data/schemas/**`, the state machine, the API surface, or any
  safety invariant requires **the other subsystem's owner** to review.
- 🔴 **Do not approve on green CI alone.** CI proves the checks passed, not that
  the change is correct, safe, or honestly described.

### 6.4 Available templates

`.github/pull_request_template.md` exists in the repository. 🔴 **It has never
been used**, because no PR has ever been opened. It should be reconciled with
§6.2 before first use.

---

## 7. Stacked pull requests

### 7.1 When to stack

Stack only when a change genuinely layers and cannot be reviewed as one unit.

```
PR 1  integration/delivery-state-contract     ← the contract, lands first
PR 2  docs/shana-state-machine-alignment      ← SHANA side of PR 1
PR 3  docs/cloudlight-status-mapping          ← CloudLight side of PR 1
PR 4  docs/exhibition-evidence-and-handoff    ← depends on PR 2 + PR 3
```

⚠️ This is an **illustration of the shape**, not a mandated sequence. Do not
manufacture a stack to look rigorous.

### 7.2 Rules

| Rule | Detail |
|---|---|
| Base PR identified | The first PR names itself the base in its description |
| Dependencies declared | Every dependent PR links its base by number |
| Stack stays small | 🔴 **3–4 PRs.** Beyond that, review quality collapses |
| No duplicate changes | A dependent PR contains only its own delta. Inherited changes are not re-reviewed as new |
| Downstream refresh | When the base PR changes, dependents rebase or merge the base — 🔴 **never force-push a shared branch** without telling every affected reviewer |
| Merge order | **Strictly base-first.** A dependent PR must never merge before its base |
| Blocked PRs | Mark explicitly; do not leave a dependent PR looking ready when its base is unresolved |
| Upstream changes significantly | Re-review the dependent PR. The original approval no longer applies |
| No native stack support? | Use ordinary branches with **explicit dependency links in the PR body**. Do not assume GitHub's draft-PR chaining exists |

### 7.3 Merge readiness classification

Applied to every PR before any merge discussion. 🔴 **These must reflect
repository evidence, not an agent's assertion.**

| Status | Meaning |
|---|---|
| `READY_FOR_REVIEW` | Submitted; review not complete |
| `CHANGES_REQUESTED` | Reviewer asked for changes |
| `BLOCKED` | Waiting on a dependency, a gate, or a permission |
| `READY_TO_MERGE` | Reviews satisfied, checks green, dependencies merged, no conflicts |
| `MERGED` | Repository confirms the merge |
| `UNKNOWN` | 🔴 The repository state could not be inspected. **Never guess** |

---

## 8. Shared interface changes — the interface-change procedure

A change to **any** of the following requires this procedure **before**
implementation begins:

- the delivery state machine or its vocabulary
- the API surface or its payloads
- authentication between the subsystems
- location / destination naming and identifiers
- the shared data model (`data/schemas/**`)
- error handling and failure semantics
- the safety boundary itself

### 8.1 The procedure

The proposer writes an entry in
[`interface_contracts.md`](interface_contracts.md) containing:

1. **Current behaviour** — quoted, with the file path
2. **Proposed behaviour** — what it becomes
3. **Reason** — why the change is needed
4. **Backward-compatibility impact** — who breaks
5. **Required change in each subsystem** — separately for SHANA and CloudLight
6. **Tests required** — what would demonstrate it works
7. **Migration strategy** — if applicable
8. **Rollback implications** — how to undo safely
9. **Decision status** — `PROPOSED` / `ACCEPTED` / `REJECTED` / `SUPERSEDED`

### 8.2 The rule that prevents the observed failure

🔴 **Do not proceed on an assumption that lives only in one agent's
conversation.** If it is not written in `docs/integration/`, it does not exist.

This is not theoretical. Two integration documents already exist —
`docs/shana/architecture/cloudlight_integration.md` and
`docs/cloudlight/shana_integration.md` — and **neither agent had seen the
other's**. They disagree on the state vocabulary, and each states in terms that
the other's contract is unagreed. SHANA's own document says it *"has no
visibility into CloudLight"*, written after CloudLight's document already
existed. See `CONF-01` and `CONF-02`.

### 8.3 Conflict ownership

| Conflict about | Decided by |
|---|---|
| Robot behaviour, safety, navigation, battery, hazards | **SHANA (Agent 1)** — it owns the physical consequence |
| Teacher identity, authorisation, timetable, resources, UI | **CloudLight (Agent 2)** |
| The **wire format** between them | **Agent 3**, after both owners review |
| A safety invariant | 🔴 **The more restrictive reading wins**, immediately, pending review |

---

## 9. Required checks

### 9.1 Today

🔴 **None. There is nothing to check.** No code, no build, no tests, no CI.

| Candidate check | Status | Why not |
|---|---|---|
| Unit tests | ⛔ | 0 source files |
| Integration tests | ⛔ | 0 source files |
| Type checking | ⛔ | no `package.json` / `pyproject.toml` |
| Linting | ⛔ | no code |
| Formatting | ⚠️ `.editorconfig` exists | Real; applies to the docs being written. `prettier`/`markdownlint` **not** configured |
| Markdown link check | ✅ **performed manually** | See §9.2 |
| Secret scanning | ✅ **performed manually** | See §9.3 |
| Frontend build | ⛔ | no frontend |
| API schema validation | ⛔ | no schema tooling; the 4 YAML schemas are unparsed prose-adjacent |
| Firmware compilation | ⛔ | no firmware |
| Dependency checks | ⛔ | no manifest |

### 9.2 Checks that were actually run in Phase A

| Check | Command | Result |
|---|---|---|
| Internal link integrity | custom Python scan over all `*.md` | 1097 links checked, **6 broken targets** — recorded, not silently fixed |
| Credential pattern scan | `grep` over every file proposed for staging | **No live secret staged** |
| Staged-file review | `git diff --cached --name-only` | 159 files reviewed before commit |
| Code-file census | `find` over source extensions | **0** |

⚠️ **These were manual, one-off, and are not automated.** They will not run
again unless someone repeats them. Recording that honestly matters more than
implying a safety net exists.

### 9.3 What to add, and when

`SPECIFIED` — add only when there is something to check:

| When | Add |
|---|---|
| First source file lands | Language-appropriate formatter + linter |
| First test lands | The test runner, in CI |
| First PR is opened | Branch protection + required review |
| Documents exceed ~250 | Markdown link checker in CI — 🔴 **6 links were already broken and nobody noticed** |

🔴 **Do not add tooling to look rigorous.** A CI badge on a repository with no
code is theatre.

---

## 10. Integration sequence

`SPECIFIED` for the future. Today, only steps 1–3 and 9–12 are reachable.

| # | Step | Owner | Reachable today |
|---|---|---|---|
| 1 | Identify the task and its **owner** | any | ✅ |
| 2 | Identify cross-project dependencies | any | ✅ |
| 3 | Agree the interface contract | Agent 3 + both owners | ✅ |
| 4 | Create the branch | the owner | ✅ (no push) |
| 5 | Run subsystem checks | the owner | ⛔ none exist |
| 6 | Review the change | the other owner | ⛔ no PR facility |
| 7 | Resolve feedback | the owner | ⚠️ via handoff |
| 8 | Merge dependencies **in order** | the owner | ✅ PR + 1 approval |
| 9 | **Update the integration repo** | Agent 3 | ✅ |
| 10 | Run cross-project tests | both | ⛔ no code |
| 11 | Update shared documentation | Agent 3 | ✅ |
| 12 | Update task board + evidence inventory | Agent 3 | ✅ |

---

## 11. Merge authority

### 11.1 Before any merge, verify

- [ ] Correct repository **and** correct PR
- [ ] Correct base branch
- [ ] Required reviews satisfied
- [ ] Required checks green (or: none exist, stated explicitly)
- [ ] Dependency PRs merged
- [ ] No unresolved conflicts
- [ ] Security and safety implications assessed
- [ ] Interface compatibility confirmed
- [ ] Within authorised scope

### 11.2 Rules

- Agent 3 **assists** the owner with review, sequencing, and readiness. 🔴 **The
  owner merges.**
- 🔴 **Never merge because an agent said it was ready.** Verify.
- 🔴 **Never bypass** branch protection, required reviews, or security controls.
- If the owner asks for a merge and a check fails → **name the specific failing
  check** and propose the smallest safe resolution. Do not work around it.
- 🔴 **Never claim a PR was merged unless the repository confirms it.**
- After a verified merge: record the merge commit, update shared docs, update
  dependencies, run integration checks, **and record failures honestly.**
  🔴 **Never claim integration succeeded when checks have not passed.**

---

## 12. Blockers

| # | Blocker | Blocks | Owner | Severity |
|---|---|---|---|---|
| 1 | ~~No remote~~ | — | — | ✅ **RESOLVED 2026-10-11** |
| 2 | ~~No hosting~~ | — | — | ✅ **RESOLVED 2026-10-11** — PRs and protection live |
| 13 | **No CI** → no status checks; review is the only gate | automated verification | Owner, once code exists | 🟠 Medium |
| 3 | **No second subsystem repo** | cross-repository PR model | — (topology is ADR-015) | ⚠️ Structural |
| 4 | **0 source files** | every automated check | Agent 1 / Agent 2, post-gate | 🔴 High |
| 5 | **0 tests** | every test claim | Agent 1 / Agent 2, post-gate | 🔴 High |
| 6 | **No CI** | automated verification | Repository owner | 🟠 Medium |
| 7 | **6 broken internal links** | navigation, credibility at exhibition | Agent 2 (mapping supplied) | 🟠 Medium |
| 8 | **Cited-but-nonexistent ADRs** | the safety argument's traceability | Agent 1 | 🔴 High |
| 9 | **State vocabularies unreconciled** | any implementation of the integration | Agent 1 + Agent 2 | 🔴 High |
| 10 | **No school answers** | all gates; any capability claim | School administration | 🔴 Blocking |

---

## 13. Rollback and recovery

| Situation | Action |
|---|---|
| A commit needs undoing | `git revert`. 🔴 **Never** `reset --hard` on shared history |
| A bad commit already pushed | New commit that undoes it. Never force-push a shared branch |
| A merge needs undoing | `git revert -m 1 <merge-sha>`, then explain in the handoff |
| Someone else's work is in the tree | **Leave it.** Commit only your own paths, and say so |
| The tree is a mess | ⚠️ **Do not clean it.** Report the state and ask |
| A branch is far behind | Merge the base, or rebase **only** a branch you own and nobody has reviewed |

🔴 **Discarding uncommitted work is almost never "cleanup."** If unsure, leave
it and ask.

---

## 14. Acceptance criteria for this specification

| # | Criterion | Status |
|---|---|---|
| 1 | Repository topology documented from actual inspection | ✅ `VERIFIED` |
| 2 | Agent autonomy preserved and stated | ✅ `VERIFIED` |
| 3 | Branch, commit, PR and stack conventions specified | ✅ `IMPLEMENTED` (document) |
| 4 | Interface-change procedure specified | ✅ `IMPLEMENTED` (document) |
| 5 | Checks specified **without claiming unconfigured ones exist** | ✅ `VERIFIED` |
| 6 | Merge authority and safety rules stated | ✅ `IMPLEMENTED` (document) |
| 7 | Every blocker named with an owner | ✅ `IMPLEMENTED` |
| 8 | ✅ Remotes created so PRs work | ✅ `IMPLEMENTED` |
| 9 | 🔴 CI configured | ⛔ `BLOCKED` — no code to check |
| 10 | ✅ Branch protection enabled | ✅ `IMPLEMENTED` — 1 review on all 3 branches |
| 11 | Agent 1 reviewed and acknowledged this spec | ⛔ `PENDING` — request issued |
| 12 | Agent 2 reviewed and acknowledged this spec | ⛔ `PENDING` — request issued |

🔴 **This document is a specification. Satisfying criteria 3–7 means the
*document* exists — not that any PR has been opened, reviewed or merged. None
has.**