# Agent Handoff Protocol

**Owner:** Agent 3 (Integration Lead)
**Status:** `IMPLEMENTED` (document) · **Applies from 2026-10-11**

---

## 🔴 The rule

**A handoff must make the work understandable without access to the previous
conversation.**

An incoming agent has no memory of what was discussed, what was decided, or
what was tried and abandoned. They have only the repository. If the reasoning
exists only in a chat log, it does not exist.

⚠️ **This repository already contains a document proving the cost of getting
that wrong.** Two integration documents were written independently — see
[`unresolved_conflicts.md`](unresolved_conflicts.md) `CONF-01`. Each states that
it has no visibility of the other. Neither is wrong. Both are incomplete in a way
that only a *shared* written record could have prevented.

---

## Required sections

Copy this structure. Every heading is required; use **`NONE`** where nothing
applies.

```markdown
# Handoff — <Agent> — <YYYY-MM-DD>

## 1. Objective
What this piece of work was for. One paragraph.

## 2. Work completed
What was actually done. Not what was intended.

## 3. Files changed
| Path | Change | Owner |

## 4. Implementation status
`AGENTS.md` §Q labels. `DESIGN_ONLY` / `UNKNOWN` are complete answers.

## 5. Tests executed
The exact commands. "NONE — no runnable code exists" is complete and correct.

## 6. Actual test results
🔴 The real output. Never a summary. Never invented.
If none ran: "NONE — 0 tests exist in this repository."

## 7. Research sources
Title · publisher · URL · date · claim supported · limitations ·
whether it applies to our exact hardware revision.
"`NOT FOUND`" is a valid and valuable entry.

## 8. Decisions made
ADR IDs, or "none". Note anything that needs the other owner's review.

## 9. Dependencies
On the other subsystem, on the school, on a gate. Name the owner.

## 10. Known limitations
What this work does not do.

## 11. Unresolved questions
Open items, with an owner for each.

## 12. Breaking changes
Or "none".

## 13. Required permissions
Who must approve what. Or "none".

## 14. Relevant PRs / commits
Or "NONE — no PR opened yet" (verify with `gh pr list`).

## 15. Recommended next action
One concrete, actionable step.
```

---

## 🔴 Prohibited in a handoff

| ❌ Never | Why | ✅ Write instead |
|---|---|---|
| Invent a test result | Fabricated evidence | `"NONE — 0 tests exist"` |
| Claim a PR was opened | 🔴 Verify with `gh pr list` | `"NONE — no PR opened yet"` |
| Claim an agent acknowledged | 🔴 **No delivery channel exists** | `"PENDING — not yet delivered"` |
| Claim a gate passed | Gates need evidence | `"PENDING — evidence not gathered"` |
| Describe `DESIGN_ONLY` in the present tense | 🔴 The core failure mode | `"SPECIFIED — not implemented"` |
| Copy a credential into the handoff | 🔴 Secret handling | `"A credential-bearing file exists; see SECURITY.md"` |
| Cite an ADR ID that does not resolve | 🔴 Evidence integrity | Verify the file exists, or name the decision log |
| Say "works" | Unfalsifiable | `"DOCUMENTED_NOT_HARDWARE_VERIFIED"` |
| Present a generated image as a photograph | 🔴 Never | `"Schematic diagram — design, not photograph"` |

---

## Status vocabulary reminder

`AGENTS.md` §Q. The most common honest answers in this project today:

| Label | Correct use here |
|---|---|
| `DESIGN_ONLY` | Specified or argued. No code. Nothing built |
| `RESEARCH_ONLY` | Investigated. No design, no code |
| `BLOCKED` | Cannot proceed. **Blocker and owner must be named** |
| `UNKNOWN` | Insufficient information. **A valid, valuable answer** |

⚠️ `UNKNOWN` is not failure. With ≈1,000 questions asked and 0 answered, `UNKNOWN`
is the **most accurate** label available for most of this project.

---

## Current state — required honesty for any handoff today

🔴 **State this before anything else, because it changes what every other
section means:**

| Fact | Value |
|---|---|
| Source files | **0** |
| Tests written / executed | **0 / 0** |
| Build system | **none** |
| CI | **none** |
| Remotes | **1** — `Ojas-sta/CloudShade`, public |
| Remote branches | **3** — `main`, `SHANA_main`, `CloudLight_Main`, all protected |
| Commits | **6** |
| Open PRs | **0** |
| Capabilities demonstrated | **0** |
| Go/no-go gates passed | **0 of 20** |
| Questions answered | **0 of ≈1,000** |
| Permissions granted | **0 of 104** |
| Outstanding agent requests | **4**, none delivered, none acknowledged |
| Open cross-project conflicts | **10** |

---

## Template — Agent 3 example (real, not hypothetical)

```markdown
# Handoff — Agent 3 (Integration) — 2026-10-11

## 1. Objective
Establish a maintainable coordination system across SHANA and CloudLight,
record what actually exists, and produce a self-contained handoff for the
exhibition proposal author.

## 2. Work completed
Phases A–L. Established the Git baseline, created the agentic rules, wrote the
workflow specification, reconciled the two subsystems' architectures into a
single contract register, identified 11 cross-project conflicts, and produced
the exhibition handoff.

## 3. Files changed
| Path | Change | Owner |
|---|---|---|
| `AGENTS.md` | Created | Agent 3 |
| `docs/integration/**` (13 files) | Created | Agent 3 |
| `docs/exhibition/**` (6 files) | Created | Agent 3 |
| `docs/architecture/architecture_decisions/ADR-015…` | Created | Agent 3 |
| `docs/architecture/architecture_decisions/ADR-016…` | Created — `Proposed` | Agent 3 |
| `.gitignore` | `.obsidian/` added | Agent 3 |
| `docs/shana/**`, `docs/cloudlight/**` | **None. Not modified.** | Agent 1 / Agent 2 |

## 4. Implementation status
Every deliverable is `DESIGN_ONLY`. 🔴 **No implementation was produced** —
that was not the task, and no code exists to implement.

## 5. Tests executed
`git status`, `git log`, `git remote -v`, `git branch -a`, `git worktree list`,
a 1097-link internal scan, and a credential-pattern scan over every file
proposed for staging.

## 6. Actual test results
- Internal links: 1097 checked, **6 broken targets** → recorded in `CONF-03`
- Credential scan: **no live secret staged**
- Source-file census: **0**
- 🔴 No unit, integration or system test was run — **none exists to run.**

## 7. Research sources
None added. The existing registers
(`docs/shana/research/source_register.md`, `docs/cloudlight/research/source_register.md`)
were audited, not duplicated. **No new external source was consulted in this
phase**, and none is claimed.

## 8. Decisions made
- `ADR-015` — one repository, not three (`Accepted`)
- `ADR-016` — single-sourced delivery state vocabulary (**`Proposed`** —
  🔴 requires Agent 1 and Agent 2 review before it binds anyone)

## 9. Dependencies
- **Agent 1**: ADR-001/010/011 must be written; `status.*` mapping table
- **Agent 2**: apply the link mapping; review `ADR-016`
- **Repository owner**: decide on a remote
- **School**: answer the 14 blocking questions

## 10. Known limitations
No PR was opened, no merge performed, no gate passed, no capability verified.
No agent was contacted — 🔴 **no delivery channel exists.**

## 11. Unresolved questions
10 conflicts, all `BLOCKED` on agent action. See `unresolved_conflicts.md`.

## 12. Breaking changes
None.

## 13. Required permissions
⚠️ *At time of writing:* repository owner — remote + hosting. ⚠️ **Both granted
2026-10-11.** School: 14 answers. 🔴 Still outstanding.

## 14. Relevant PRs / commits
At time of writing: **NONE — no remote existed, so no PR was possible.**
Local commits: `3a01fa9` (baseline).

⚠️ **Superseded 2026-10-11:** the owner authorised a public remote
(`Ojas-sta/CloudShade`), three protected branches, and a worktree per agent.
🔴 **Still no PR has been opened.** See
[`AGENT_BRANCH_AND_PR_WORKFLOW.md`](AGENT_BRANCH_AND_PR_WORKFLOW.md).

## 15. Recommended next action
Deliver the three requests in `docs/integration/agent-requests/` to Agent 1 and
Agent 2 — `AGENT_1_SHANA_GIT_WORKFLOW_REQUEST.md`,
`AGENT_2_CLOUDLIGHT_GIT_WORKFLOW_REQUEST.md`, and the joint
`CROSS_REPO_RECONCILIATION_REQUEST.md`. Everything else waits on that.
```

---

## Checklist before submitting

- [ ] Statuses use `AGENTS.md` §Q labels
- [ ] Test results are **real output**, or explicitly `NONE`
- [ ] No ADR cited that does not resolve to a file
- [ ] No credential copied
- [ ] No agent response claimed without evidence
- [ ] No capability described as implemented
- [ ] Links resolve
- [ ] The state table is current
- [ ] A reader with no chat history could act on §15