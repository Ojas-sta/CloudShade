# Agent Branch and PR Workflow

**Owner:** Agent 3 (Integration Lead)
**Status:** 🔴 **`IMPLEMENTED`** — the infrastructure exists and is verified
**Last updated:** 2026-10-11

> **This supersedes the "no remote / no PR" limitations recorded earlier in
> this directory.** Those statements were true when written on 2026-10-11 and
> were made public on the same day. They are now corrected here.

---

## 1. 🔴 What changed, and when

| Fact | Before | **Now** |
|---|---|---|
| Remotes | 0 | 🔴 **1** — `origin https://github.com/Ojas-sta/CloudShade.git` |
| Remote branches | 0 | 🔴 **3** — `main`, `SHANA_main`, `CloudLight_Main` |
| Visibility | — | 🔴 **PUBLIC** |
| Worktrees | 1 | 🔴 **3** — one per agent, plus the integration root |
| Branch protection | none | 🔴 **all 3 protected** — 1 approving review, no force-push, no deletion |
| PRs | ⛔ impossible | ✅ **available and required** |

⚠️ **Consequence: neither agent can push directly to `main`, `SHANA_main` or
`CloudLight_Main`.** All three require a PR and one approving review. That is
deliberate — it is the gate this document exists to operate.

---

## 2. Topology

```
                      origin (GitHub, PUBLIC)
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
   main            SHANA_main          CloudLight_Main
  (protected)       (protected)            (protected)
        ▲                    ▲                    ▲
        │                    │                    │
   ┌────┴────────────────────┴────────────────────┴────┐
   │            THE INTEGRATION ROOT                    │
   │  ~/Documents/Default Project    │
   │  branch: main     — Agent 3 works here            │
   └────────────────────────────────────────────────────┘

   ┌──────────────────────┐        ┌──────────────────────┐
   │ AGENT 1 — SHANA     │        │ AGENT 2 — CLOUDLIGHT │
   │ ~/Documents/SHANA_main │        │ ~/Documents/CloudLight_│
   │ branch: SHANA_main  │        │ Main                │
   │                     │        │ branch: CloudLight_  │
   │ ✅ write access     │        │ Main                │
   │ ✅ may open PRs     │        │ ✅ write access       │
   │ ✅ may push features│        │ ✅ may open PRs       │
   └──────────────────────┘        │ ✅ may push features  │
                                   └──────────────────────┘
```

⚠️ **Each agent must work in their own worktree.** They share one machine and
one disk; two agents cannot `git checkout` different branches in the same
directory without colliding.

---

## 3. 🔴 Agent 1 — SHANA

```bash
cd ~/Documents/SHANA_main
git status                      # confirm you are on SHANA_main
git fetch origin && git pull
```

### Everyday loop

```bash
# 1. Branch OFF your lane branch — never work directly on it
git checkout -b docs/shana-something

# 2. Work. Commit with the §Q status convention (AGENTS.md §Q)
git add docs/shana/          # ONLY your own paths
git commit -m "docs(shana): correct X

Refs: TASK-ID | ADR-NNN
Status: DESIGN_ONLY"

# 3. Push the FEATURE branch (feature branches are unprotected — this works)
git push -u origin docs/shana-something

# 4. Open a PR into your lane
gh pr create --base SHANA_main --head docs/shana-something \
  --title "docs(shana): correct X" \
  --body "See .github/pull_request_template.md"
```

### 🔴 What you may and may not push

| Branch | May push? |
|---|---|
| `main` | 🔴 **No — protected** |
| `SHANA_main` | 🔴 **No — protected** |
| `CloudLight_Main` | 🔴 **No — and not yours** |
| `feature/…`, `docs/…`, `fix/…` | ✅ **Yes** |

---

## 4. 🔴 Agent 2 — CloudLight

Identical, substituting your lane.

```bash
cd ~/Documents/CloudLight_Main
git checkout -b docs/cloudlight-something
# ... work, commit, push ...
git push -u origin docs/cloudlight-something
gh pr create --base CloudLight_Main --head docs/cloudlight-something \
  --title "docs(cloudlight): correct X" --body "..."
```

🔴 **Agent 2 may only stage `docs/cloudlight/**` and their questionnaire.**
🔴 Agent 1 may only stage `docs/shana/**`, their questionnaire, and
`data/schemas/**` via the interface-change procedure.

---

## 5. PR flow

```
  feature branch                lane branch              main
  (unprotected)                 (protected)              (protected)
        │                            │                      ▲
        │  push                      │                      │
        ├───────────────────────────►│                      │
        │                            │                      │
        │   PR  ── approve ── merge  │                      │
        │                            │                      │
        │                            │   PR ─ approve ─ merge
        └────────────────────────────┼─────────────────────►│
```

| Step | Actor | Gate |
|---|---|---|
| 1 | Agent | Push feature branch — no gate |
| 2 | Agent | Open PR → their lane |
| 3 | Owner | **Review** |
| 4 | Owner | **Approve** (1 required) |
| 5 | Owner | Merge |
| 6 | Agent 3 | Merge lane → `main` when the lane is releasable |
| 7 | Agent 3 | Merge requires its own PR + approval |

⚠️ **Step 6 is deliberate.** `main` is the combined project. A lane merges into
it only when it is coherent — not after every PR.

---

## 6. Stacked PRs

Use a stack when work genuinely layers. 🔴 **Keep it to 3–4 PRs.**

```bash
# PR 1 of 3 — base is the lane
git checkout -b integration/delivery-contract
git push -u origin integration/delivery-contract
gh pr create --base SHANA_main --head integration/delivery-contract

# PR 2 of 3 — branches OFF PR 1, not off the lane
git checkout -b integration/delivery-contract-impl
git push -u origin integration/delivery-contract-impl
gh pr create --base integration/delivery-contract --head integration/delivery-contract-impl \
  --body "Depends on #<PR1>. Review only the delta."

# PR 3 of 3 — depends on PR 2
git checkout -b integration/delivery-contract-docs
git push -u origin integration/delivery-contract-docs
gh pr create --base integration/delivery-contract-impl --head integration/delivery-contract-docs \
  --body "Depends on #<PR2>."
```

### 🔴 Rules

| Rule | Detail |
|---|---|
| Base PR identified | PR 1 says *"This is the base of a 3-PR stack"* |
| Dependencies declared | Every dependent PR links its base PR **by number** |
| Downstream refresh | When the base changes, dependents merge the base down. 🔴 **Never force-push a shared branch** |
| 🔴 **Merge order** | **Strictly base-first.** A dependent must never merge before its base |
| Blocked PRs | Mark explicitly. Do not leave a dependent looking ready when its base is unresolved |
| Base changed significantly | 🔴 **The prior approval no longer applies.** Re-review |
| Stack review | Review only the delta. 🔴 Do not review inherited changes as new |

⚠️ GitHub renders cross-PR dependencies automatically from the base branch —
you do not need a native stacking feature.

---

## 7. 🔴 Review and approval rules

### Reviewer checklist

- [ ] Correct lane? (`SHANA_main` for Agent 1, `CloudLight_Main` for Agent 2)
- [ ] 🔴 **Does it stage only its owner's paths?**
- [ ] Only the delta reviewed, not inherited changes
- [ ] Every claim carries an `AGENTS.md` §Q status label
- [ ] Test claims are **real output**, or explicitly `NONE — none exist`
- [ ] Links in changed files resolve
- [ ] No secret, no school-identifying detail added
- [ ] Interface change? → §G procedure followed
- [ ] 🔴 **No claim of implementation where only documentation changed**
- [ ] 🔴 **Not approved solely because it is small**

### 🔴 Never do

- ❌ **Approve your own PR.** Different agent, or the owner.
- ❌ **Bypass protection.** It exists to stop exactly that.
- ❌ **Force-push a shared branch** without telling every affected reviewer.
- ❌ **Merge a dependent before its base.**
- ❌ **Merge on green checks alone** — there are no checks yet; the review *is* the gate.

---

## 8. 🔴 Cross-lane visibility

The lanes will diverge. When an agent needs the other's work:

```bash
# Agent 2 needs SHANA's agreed state machine
git fetch origin
git merge origin/SHANA_main        # into a feature branch, never your lane
```

| Need | Action |
|---|---|
| Read the other's lane | `git fetch origin && git log origin/SHANA_main` |
| Incorporate their work | Merge into a **feature branch**, open a PR |
| 🔴 Push to their lane | **Never.** Protected, and not yours |
| 🔴 Edit their files | Never. Use `agent-requests/` |

⚠️ **A lane that is far behind is a real risk** — CloudLight could build
against a SHANA state machine that has since changed. 🔴 **Mitigation: both
agents should rebase on their lane at the start of a task**, and `main` should
be merged into both lanes periodically by Agent 3.

---

## 9. Merge authority

| Action | Who |
|---|---|
| Merge a feature PR into a lane | 🔴 **The owner approves. Agent 3 executes.** |
| Merge a lane into `main` | 🔴 **The owner approves. Agent 3 executes.** |
| 🔴 Enable/disable protection | 🔴 **The owner only** |
| 🔴 Delete a branch | 🔴 **The owner only** — protected branches refuse deletion |
| Push a feature branch | The owning agent |

⚠️ **No agent merges their own work.** `AGENTS.md` §11 of the workflow spec.

---

## 10. 🔴 What is still blocked

Setting up Git does **not** unblock engineering.

| Still blocked | Why |
|---|---|
| Writing implementation code | 🔴 0 of 20 go/no-go gates passed |
| Operating a robot | 🔴 0 of 104 permissions granted |
| Testing | 🔴 0 source files, 0 tests |
| CI status checks | 🔴 Nothing to check. Protection requires **reviews**, not checks |
| Running any `cloudlight/**` check | 🔴 No test suite exists |

🔴 **A pull request mechanism is not authorisation to build.** `AGENTS.md` §J
and §L.

---

## 11. 🔴 Committing to `main` directly is now impossible

For anyone following older documentation:

> *"Do not commit or push without explicit instruction"* — still true for
> anything outside a PR.

But 🔴 **the Git topology has changed**: `main` is protected, so a direct push
is refused by GitHub rather than by convention. That is an improvement —
convention relied on goodwill, protection does not.

⚠️ **The one caveat:** protection blocks *GitHub*. A local commit is still
possible anywhere. 🔴 **Local commits are still expected and encouraged** — the
gate is at the remote.

---

## 12. Verification

| Property | Value | Verified by |
|---|---|---|
| Remote | `origin` → `Ojas-sta/CloudShade` | `git remote -v` |
| Visibility | `PUBLIC` | `gh repo view --json visibility` |
| Branches on remote | `main`, `SHANA_main`, `CloudLight_Main` | `git ls-remote --heads origin` |
| Worktrees | 3 | `git worktree list` |
| Protection on all 3 | 1 review, admins enforced, no force-push, no delete | `gh api .../branches/{b}/protection` |
| 🔴 Source files | **0** | filesystem scan |
| 🔴 Tests | **0** | no test directory |
| 🔴 CI workflows | **0** | no `.github/workflows/` |
| 🔴 Gates passed | **0 of 20** | both `go_no_go_gates.md` |

---

## 13. Quick reference

```bash
# Agent 1
cd ~/Documents/SHANA_main
git checkout -b docs/shana-my-change && git push -u origin HEAD
gh pr create --base SHANA_main --head docs/shana-my-change --title "…" --body "…"

# Agent 2
cd ~/Documents/CloudLight_Main
git checkout -b docs/cloudlight-my-change && git push -u origin HEAD
gh pr create --base CloudLight_Main --head docs/cloudlight-my-change --title "…" --body "…"

# Owner approves, then Agent 3 merges
gh pr list --base SHANA_main --state open
gh pr merge <N> --squash
```

⚠️ **Before opening any PR: re-read the checklist in §7.**