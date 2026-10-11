# Repository State Audit — Phase A Baseline

**Auditor:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Method:** direct command inspection of the working tree. Every value below was
read from a command, not recalled.

> This file is the evidence record for Phase A of the integration procedure. It
> exists so that later claims about "the repository state" can be checked against
> a dated, reproducible observation rather than against memory.

---

## 1. Git baseline as found

| Check | Command | Result |
|---|---|---|
| Repository root | `git rev-parse --show-toplevel` | `/Users/roopalisingh/Documents/Default Project` |
| Branch | `git branch --show-current` | `main` |
| Commits | `git log --oneline` | 🔴 **none — "No commits yet on main"** |
| Remotes | `git remote -v` | 🔴 **empty — 0 remotes** |
| Branches | `git branch -a` | `main` only |
| Tags | `git tag` | 🔴 **empty** |
| Worktrees | `git worktree list` | 1 — the root only |
| In-progress merge/rebase | directory inspection | none |
| Staged changes | `git status --short` | none (no HEAD to diff against) |
| Untracked content | `git status --short --branch` | 13 untracked top-level entries |

## 2. 🔴 Topological finding — the brief did not match reality

The operating brief for Agent 3 describes **three repositories**: a shared
integration repository plus one for SHANA owned by Agent 1 and one for
CloudLight owned by Agent 2, with cross-repository pull requests between them.

🔴 **There is one repository, and it contains both subsystems.**

| Expected by the brief | Actually found |
|---|---|
| Shared integration repo | ✅ this directory |
| Separate SHANA repo | ⛔ **does not exist** |
| Separate CloudLight repo | ⛔ **does not exist** |
| Cross-repository PRs | ⛔ **impossible — no second repo, no remote, no hosting** |
| Source code to integrate | ⛔ **0 implementation files** |
| Tests to run | ⛔ **0 tests exist** |

This is **not** a failure of the brief. It is a material fact that changes what
"integration" can mean here. `docs/shana/` and `docs/cloudlight/` are two
subsystems documented **inside one tree**, and they have therefore been authored
**without a shared interface-agreement step** — which is precisely how the
conflicts in [`unresolved_conflicts.md`](unresolved_conflicts.md) arose.

Recorded as
[`ADR-015`](../../docs/architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md).

## 3. Content inventory as found

169 files excluding `.git/` and `.obsidian/`.

| Category | Count | Notes |
|---|---|---|
| `docs/shana/**` | ~100 markdown | SHANA feasibility study |
| `docs/cloudlight/**` | ~30 markdown | CloudLight feasibility study |
| `docs/architecture/**` | 2 ADR files | ADR-013, ADR-014 |
| `questionnaires/**` | 6 markdown | SHANA + CloudLight instruments |
| `data/schemas/**` | 4 YAML + README | proposed, never parsed by code |
| `data/templates/**` | 7 markdown | operational record templates |
| Root governance | 7 files | README, CONTRIBUTING, SECURITY, CODE_OF_CONDUCT, LICENSE_REVIEW, `.editorconfig`, `.gitignore` |
| `.github/**` | 7 files | issue templates + PR template |
| **Source code** | **0** | no `.py .js .ts .tsx .go .c .cpp .java .rs` |
| **Tests** | **0** | no test directory |
| **CI workflows** | **0** | no `.github/workflows/` |
| **Build config** | **0** | no `package.json`, `pyproject.toml`, `Makefile`, `CMakeLists.txt` |

### 3.1 The only non-documentation file

`start_opencode_server.sh` at the repository root. Unrelated to SHANA or
CloudLight. **Contains a hardcoded plaintext administrative password** and
starts a server bound to `0.0.0.0:4096`.

Actions taken: it was **already** excluded by `.gitignore` before this audit, and
was **not** committed. Its credential value was not read into, or reproduced
in, any document. 🔴 **It still requires rotation by its owner** — being
gitignored does not un-expose a cleartext password sitting on disk.

## 4. Files deliberately excluded from version control

Verified by `git add --dry-run` before staging. Each was confirmed **absent**
from the staged set.

| Excluded | Reason | Already ignored before audit? |
|---|---|---|
| `start_opencode_server.sh` | 🔴 hardcoded plaintext credential | yes |
| `Image.jpeg`, `Image 2.jpeg` … `Image 8.jpeg` (8 files) | 🔴 photographs of **LCD/display panels**, unrelated to this project. Committing them risks their later being mistaken for project evidence | yes |
| `.DS_Store` | macOS noise | yes |
| `.obsidian/` (4 files) | **machine-specific Obsidian vault/workspace state**, incl. open-pane session data — not project content | ⛔ **no — added by this audit** |

> `.obsidian/workspace.json` records one person's editor session. It is local
> tooling state, has no place in a shared repository, and changes on every
> keystroke. It was added to `.gitignore` in the baseline commit.

## 5. Actions performed

| # | Action | Verification |
|---|---|---|
| 1 | Inspected git state, tree, and content before modifying anything | §1, §3 |
| 2 | Ran an internal-link integrity scan across all markdown | §6 |
| 3 | Added `.obsidian/` to `.gitignore` | dry-run confirms exclusion |
| 4 | Ran a credential-pattern scan over every file proposed for staging | no live secret found |
| 5 | Staged explicitly by path — **never** `git add .` blindly | `git diff --cached --name-only` = 159 files |
| 6 | Created the baseline commit | `3a01fa9` |

🔴 **Not performed:** `git init` (already initialised), any destructive Git
command, any remote creation, any push, any history rewrite. **No existing work
was discarded** — the repository had no history to discard.

## 6. 🔴 Defect found by inspection — broken internal links

A scan of **1097 internal markdown links** found **6 broken targets**.

| Broken target | Cited from | Correct target |
|---|---|---|
| `../architecture/architecture.md` | `docs/cloudlight/requirements.md` | `docs/shana/architecture/architecture.md` |
| `../architecture/data_flow.md` | `docs/cloudlight/timetable_and_substitutions.md` | `docs/shana/architecture/data_flow.md` |
| `../infrastructure/school_network.md` | `docs/cloudlight/requirements.md`, `resource_browser.md`, `resource_search.md` | `docs/shana/infrastructure/school_network.md` |
| `../safety/risk_register.md` | `docs/cloudlight/requirements.md`, `risk_register.md` | `docs/shana/safety/risk_register.md` |
| `../safety/student_privacy.md` | `docs/cloudlight/ai_assistant.md`, `requirements.md` | `docs/shana/safety/student_privacy.md` |
| `../../data/schemas/` | `docs/shana/handoff/unresolved_dependencies.md` | `data/schemas/` (needs one more `../`) |

**Root cause, established by inspection:** SHANA's documentation originally sat
at `docs/architecture/`, `docs/safety/` and `docs/infrastructure/`. It was later
moved under `docs/shana/`. CloudLight was written against the **pre-move**
layout. CloudLight's own documentation noticed the breakage and recorded it
rather than fixing it — correctly, under the ownership rule — see
`docs/cloudlight/README.md:204` and `docs/cloudlight/research/source_register.md:933`.

**Owner:** Agent 2, for the CloudLight-side links. **Agent 3** has issued a
corrected mapping table rather than editing Agent 2's files. See
[`agent-requests/AGENT_2_CLOUDLIGHT_GIT_WORKFLOW_REQUEST.md`](agent-requests/AGENT_2_CLOUDLIGHT_GIT_WORKFLOW_REQUEST.md) R3.

## 7. Second defect class — citations to files that do not exist

Separate from broken links, and **more serious**, because it misrepresents the
evidence base rather than merely the navigation:

🔴 CloudLight documents cite SHANA **architecture decisions that were never
written as ADR files** — `ADR-001` (no AI model reaches the robot), `ADR-010`
(no raw video retention) and `ADR-011` (simulation before hardware). They are
recorded in `docs/shana/feasibility/decision_log.md` and indexed as *owed* in
`docs/shana/architecture/architecture_decisions/README.md`, but no such files
exist.

🔴 CloudLight also cites `docs/architecture/delivery_state_machine.md`, which
does not exist under any path. The nearest real file is
`docs/shana/architecture/task_state_machine.md`.

Only **4 ADR files exist** in the entire repository: ADR-002, ADR-003
(SHANA-internal) and ADR-013, ADR-014 (cross-project).

This matters because CloudLight's safety argument rests on `ADR-001` and
`ADR-003`. The *decisions* are real and their reversal conditions are recorded.
The *citations* are unresolvable. Tracked as `CONF-04` in
[`unresolved_conflicts.md`](unresolved_conflicts.md).

## 8. State after the baseline commit

| Check | Value |
|---|---|
| Branch | `main` |
| Commit | `3a01fa9` |
| Commits | **1** |
| Remotes | **0** (unchanged — creating one requires owner authorisation) |
| Working tree | **clean** (`git status --short` empty) |
| Files tracked | 159 |
| Source code | **0** |
| Tests | **0** |

## 9. What Phase A could not do, and why

| Blocker | Consequence | Owner |
|---|---|---|
| No remote exists | 🔴 **No push is possible.** No PR can be opened, reviewed or merged | Repository owner |
| No hosting platform configured | 🔴 The PR and stacked-PR procedures are `SPECIFIED`, not `IMPLEMENTED` | Repository owner |
| No second subsystem repository | 🔴 Cross-repository PR review is `NOT APPLICABLE` today | — (topology is ADR-015) |
| Zero source code | 🔴 No checks can be configured — there is nothing to build, lint or test | Agent 1 / Agent 2, after gates pass |
| No school answers | 🔴 Gates cannot pass; no capability can be verified | School administration |

## 10. Phase A conclusion

**Phase A is `DONE`** under the §Q definition of done: the state was inspected,
recorded, existing work was preserved and committed without loss, exclusions were
verified, and every blocker is named with an owner.

🔴 **The repository now has version control and a recorded baseline. It does not
have a remote, code, tests, CI, or any approved capability.** Those are separate
gaps owned by separate people, tracked in
[`TASK_BOARD.md`](TASK_BOARD.md).