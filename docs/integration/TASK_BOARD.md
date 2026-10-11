# Integration Task Board

**Owner:** Agent 3 (Integration Lead)
**Last updated:** 2026-10-11

**Statuses:** `TODO` · `IN_PROGRESS` · `BLOCKED` · `IN_REVIEW` · `DONE`
**Implementation statuses** use the `AGENTS.md` §Q vocabulary and are recorded
separately, because a task can be `DONE` while the thing it tracks is `UNKNOWN`.

---

## Summary

| Status | Count |
|---|---|
| `DONE` | **7** |
| `TODO` | 6 |
| `IN_PROGRESS` | 2 |
| `BLOCKED` | 22 |
| `IN_REVIEW` | 0 |
| **Total** | **37** |

🔴 **All 7 completed tasks are documentation or coordination.** Not one produced
code, a test, a capability, or a demonstration.

---

## Legend

| Column | Meaning |
|---|---|
| **Impl status** | `AGENTS.md` §Q label for the *tracked thing*, not the task |
| **Tests** | `NONE` = no runnable code exists 🔴 |

---

## A. Integration coordination — owned by Agent 3

| ID | Task | Owner | Status | Impl | Priority | Depends on | Acceptance | Files | Tests |
|---|---|---|---|---|---|---|---|---|---|
| INT-01 | Audit repository state; record baseline evidence | Agent 3 | ✅ `DONE` | `IMPLEMENTED_AND_TESTED` | 🔴 P0 | — | State inspected, recorded, committed, exclusions verified | `repository_state_audit.md` | `VERIFIED` by command |
| INT-02 | Create baseline commit of existing work | Agent 3 | ✅ `DONE` | `IMPLEMENTED_AND_TESTED` | 🔴 P0 | INT-01 | 159 files reviewed and staged explicitly; no secret committed | — | `VERIFIED` |
| INT-03 | Add `.obsidian/` to `.gitignore` | Agent 3 | ✅ `DONE` | `IMPLEMENTED_AND_TESTED` | 🟡 P3 | INT-01 | Machine-specific state excluded | `.gitignore` | `VERIFIED` |
| INT-04 | Create root `AGENTS.md` | Agent 3 | ✅ `DONE` | `IMPLEMENTED_AND_TESTED` | 🔴 P0 | INT-01 | Sections A–R present; ownership, status vocabulary, done criteria defined | `AGENTS.md` | `VERIFIED` |
| INT-05 | Create Git + cross-project workflow spec | Agent 3 | ✅ `DONE` | `IMPLEMENTED_AND_TESTED` | 🔴 P0 | INT-04 | Topology documented from inspection, not assumption | `GIT_AND_AGENT_WORKFLOW_SPEC.md` | `VERIFIED` |
| INT-06 | Record cross-project conflicts | Agent 3 | ✅ `DONE` | `IMPLEMENTED_AND_TESTED` | 🔴 P0 | INT-05 | Each conflict has both positions quoted, an owner, evidence | `unresolved_conflicts.md` | `VERIFIED` |
| INT-07 | Establish agent requests | Agent 3 | ⛔ `BLOCKED` | `DESIGN_ONLY` | 🔴 P0 | INT-05 | 🔴 **No delivery channel exists** | `agent-requests/*` | `NONE` |
| INT-08 | Reconcile the two integration contracts | Agent 1 + 2 | 🔴 `BLOCKED` | `DESIGN_ONLY` | 🔴 P0 | INT-07, CONF-01 | One contract, both owners `ACCEPTED` | `interface_contracts.md` | `NONE` |
| INT-09 | Establish a remote and hosting | Repository owner | 🔴 `BLOCKED` | `BLOCKED` | 🔴 P0 | owner decision | Remote exists; PRs possible | — | `NONE` |
| INT-10 | Configure CI | Repository owner | 🔴 `BLOCKED` | `BLOCKED` | 🟠 P2 | first code lands | 🔴 Nothing to check yet | — | `NONE` |
| INT-11 | Enable branch protection | Repository owner | 🔴 `BLOCKED` | `BLOCKED` | 🟠 P2 | INT-09 | Required review + checks | — | `NONE` |
| INT-12 | Automate the markdown link check | Agent 3 | 🔴 `BLOCKED` | `BLOCKED` | 🟡 P3 | INT-10 | 🔴 6 links broke unnoticed | — | `NONE` |
| INT-13 | Produce the Antigravity exhibition handoff | Agent 3 | ⛔ `IN_PROGRESS` | `DESIGN_ONLY` | 🔴 P0 | INT-06 | Self-contained; evidence-bound | `../exhibition/…` | `NONE` |

---

## B. Conflicts — owned by the named agent

| ID | Conflict | Owner | Status | Priority | Blocks | Decision |
|---|---|---|---|---|---|---|
| CONF-01 | Two integration docs written blind | Agent 1 + 2 | 🔴 `BLOCKED` | 🔴 P0 | INT-08 | Joint session → `IC-01`…`IC-11` |
| CONF-02 | 🔴 `PAUSED` unrepresentable | Agent 1 + 2 | 🔴 `BLOCKED` | 🔴 P0 | any integration | [`ADR-016`](../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md) — **`Proposed`** |
| CONF-03 | 6 broken internal links | Agent 2 | 🔴 `BLOCKED` | 🟠 P2 | navigation, credibility | Mapping **supplied**; needs application |
| CONF-04 | Cited ADRs do not exist | Agent 1 | 🔴 `BLOCKED` | 🔴 P0 | evidence integrity | Write ADR-001, ADR-010, ADR-011 |
| CONF-05 | `delivery_state_machine.md` does not exist | Agent 2 | 🔴 `BLOCKED` | 🟠 P2 | CONF-02 | Correct citations; **do not rename** (Agent 1's file) |
| CONF-06 | `request_id` vs `delivery_id` | Agent 1 + 2 | 🔴 `BLOCKED` | 🟠 P2 | IC-02, idempotency | One identifier model |
| CONF-07 | `/v1/` vs `/api/v1/` | Agent 2 | 🔴 `BLOCKED` | 🟡 P3 | nothing today | Owner decides |
| CONF-08 | `status.*` vs state machine | Agent 1 | 🔴 `BLOCKED` | 🟠 P2 | IC-05 | Publish a mapping table |
| CONF-09 | Occupancy authority | Agent 1 + 2 | 🔴 `BLOCKED` | 🟠 P2 | route safety | 🔴 Recommend: never a CloudLight safety input |
| CONF-10 | `requested_by` semantics | Agent 2 | 🔴 `BLOCKED` | 🟡 P3 | IC-03 | Prefer **role ID** |
| CONF-11 | `.obsidian/` not ignored | Agent 3 | ✅ `DONE` | 🟡 P3 | — | Resolved in baseline commit |

⚠️ **All 10 open conflicts are `BLOCKED` on agent action, and every one of those
agents is itself unblocked to act** — the block is coordination, not permission.
🔴 **No acknowledgement has been received from Agent 1 or Agent 2.**

---

## C. SHANA — 🔴 owned by Agent 1. Listed for dependency tracking only.

🔴 Agent 3 does not track these as actionable work. They appear so the
cross-project dependencies are visible. Full detail:
`docs/shana/handoff/unresolved_dependencies.md`.

| ID | Task | Status | Blocks | Notes |
|---|---|---|---|---|
| SH-B1 | Go2 model, firmware, SDK licence | 🔴 `BLOCKED` | Hardware gates 1–2 | Custodian |
| SH-B2 | Battery fitted — 8,000 or 15,000 mAh | 🔴 `BLOCKED` | Battery reserve | Custodian |
| SH-B3 | Written secondary-development authorisation | 🔴 `BLOCKED` | **Everything** | Administration |
| SH-B4 | Stair riser heights | 🔴 `BLOCKED` | Vertical transit | Facilities |
| SH-B5 | Single-floor route to rooms actually needed | 🔴 `BLOCKED` | **Project scope** | Facilities + teachers |
| SH-B6 | Lift manufacturer response | 🔴 `BLOCKED` | Cross-floor scope | Johnson via school |
| SH-B7 | Privacy assessment for camera/CCTV | 🔴 `BLOCKED` | Gate 8 | School / legal |
| SH-B8 | Network and robot access approvals | 🔴 `BLOCKED` | Any hardware work | School IT |
| SH-B9 | 🔴 Simulation-only proof of concept | 🔴 `BLOCKED` | — | ⚠️ **The only unblocked high-value engineering task.** No hardware, no school data needed. 🔴 **Not yet started** |

---

## D. CloudLight — 🔴 owned by Agent 2. Listed for dependency tracking only.

| ID | Task | Status | Blocks | Notes |
|---|---|---|---|---|
| CL-B1 | May a persistent local service run at all? | 🔴 `BLOCKED` | 🔴 **Everything** | Principal + IT. **~10 min** |
| CL-B2 | Can classroom devices reach that host? | 🔴 `BLOCKED` | Every display | IT. **~15 min** |
| CL-B3 | Is client/AP isolation enabled? | 🔴 `BLOCKED` | Silent failure | IT |
| CL-B4 | Who approves a substitution? Who owns the timetable? | 🔴 `BLOCKED` | 🔴 The core feature | Timetable / admin. **~15 min** |
| CL-B5 | Windows edition on classroom boards | 🔴 `BLOCKED` | Kiosk design | IT. **~10 min** |
| CL-B6 | Does a NAS exist at all? | 🔴 `BLOCKED` | The whole resource track | IT + Library. **~10 min** |
| CL-B7 | Does a staff identity directory exist? Is SSO permitted? | 🔴 `BLOCKED` | Authentication | IT. **~10 min** |
| CL-B8 | Is any external AI service permitted, in writing? | 🔴 `BLOCKED` | The assistant only | Principal. **~5 min** |
| CL-B9 | 🔴 Deterministic timetable parsing vs synthetic fixtures | 🔴 `BLOCKED` | — | ⚠️ **Highest-value, lowest-risk engineering available. Depends on NO unanswered question.** 🔴 **Not yet started** |

---

## E. Exhibition — owned by Agent 3

| ID | Task | Status | Impl | Priority | Depends on |
|---|---|---|---|---|---|
| EXH-01 | Exhibition claim-evidence matrix | ⛔ `IN_PROGRESS` | `DESIGN_ONLY` | 🔴 P0 | INT-06 |
| EXH-02 | Exhibition risk register | 🔴 `BLOCKED` | `DESIGN_ONLY` | 🔴 P0 | EXH-01 |
| EXH-03 | Evidence index — what actually exists | 🔴 `BLOCKED` | `RESEARCH_ONLY` | 🔴 P0 | INT-01 |
| EXH-04 | Demonstration plan (PLAN A / B / C) | 🔴 `BLOCKED` | `DESIGN_ONLY` | 🔴 P0 | EXH-01 |
| EXH-05 | Measurement plan | 🔴 `BLOCKED` | `DESIGN_ONLY` | 🟠 P1 | EXH-01 |
| EXH-06 | Antigravity handoff prompt | 🔴 `BLOCKED` | `DESIGN_ONLY` | 🔴 P0 | EXH-01…05 |

---

## F. 🔴 Repository-wide status

| Metric | Value | Source |
|---|---|---|
| Source files | **0** | filesystem scan |
| Tests written / run | **0 / 0** | no test directory |
| CI workflows | **0** | no `.github/workflows/` |
| Remotes | **0** | `git remote -v` |
| Commits | **1** | `git log` |
| Open conflicts | **10** | this document |
| Outstanding requests | **4** | `agent-requests/` |
| 🔴 Requests acknowledged | **0** | no delivery channel |
| Questions asked of the school | **≈1,000** | 434 SHANA + 576 CloudLight |
| Questions answered | **0** | questionnaire `Status` columns |
| Permissions sought / granted | 102 / **0** | SHANA permission checklist |
| Go/no-go gates defined / passed | 20 / **0** | 11 SHANA + 9 CloudLight |
| Risks registered / retired | 67 / **0** | 21 SHANA + 46 CloudLight |
| Assumptions verified | **1 of 58** | 34 SHANA + 24 CloudLight |
| Capabilities demonstrated | **0** | — |

---

## G. ⚠️ The board's honest summary

**Every unblocked action on this board produces documentation.** The two
genuinely unblocked engineering tasks — `SH-B9` (simulation proof of concept)
and `CL-B9` (deterministic timetable parsing) — are 🔴 **both marked `BLOCKED`
because they have not been started**, and both belong to other agents.

⚠️ **A task board in which the highest-value available work is unstarted is a
board describing a stalled project.** That is an accurate description of this
one. The blocker is not complexity; it is that ≈1,000 questions have been asked
and none has been answered, and the work that *could* proceed unblocked has not
been picked up.