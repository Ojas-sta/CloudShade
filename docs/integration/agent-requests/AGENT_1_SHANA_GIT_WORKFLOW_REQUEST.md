# Request to Agent 1 — SHANA Git Workflow

**From:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Status:** 🔴 **`PENDING` — not delivered, not acknowledged**
**Response:** ⛔ **None received.** No delivery channel exists in this repository.

---

## 🔴 How to read this

**This is a written request, not a message.** Agent 3 has **no way to deliver
it** — there is no remote, no hosting, no issue tracker, no agent-to-agent
channel. It is a file you are reading.

🔴 **No acknowledgement is claimed anywhere in this repository, because none has
happened.**

If you are Agent 1, this document needs to be delivered to you by a human.

---

## 0. 🔴 Repository reality — read before acting

Agent 3's brief for this request described **three repositories**: yours, one for
Agent 2, and a shared integration repository, with cross-repository PRs.

🔴 **There is one repository.** No SHANA repository exists. No CloudLight
repository exists. There are **0 remotes**.

```bash
git rev-parse --show-toplevel   # one root
git remote -v                   # empty
git branch -a                   # main only
git log --oneline               # 1 commit: 3a01fa9 (baseline, 2026-10-11)
```

Full evidence: [`../repository_state_audit.md`](../repository_state_audit.md) ·
[`ADR-015`(../../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md).

🔴 **Practical consequence:** there is no cross-repository PR to open, and no
remote to push to. Your autonomy is **branching and committing within your own
paths in this repository.**

---

## 1. Read these first

| # | Document | Why |
|---|---|---|
| 1 | [`AGENTS.md`](../../../AGENTS.md) | 🔴 The root rulebook. Sections C, D, G, M, Q matter most to you |
| 2 | [`../GIT_AND_AGENT_WORKFLOW_SPEC.md`](../GIT_AND_AGENT_WORKFLOW_SPEC.md) | 🔴 The authoritative workflow specification |
| 3 | [`../AGENT_COORDINATION.md`](../AGENT_COORDINATION.md) | Ownership and permissions |
| 4 | [`../AGENT_HANDOFF_PROTOCOL.md`](../AGENT_HANDOFF_PROTOCOL.md) | How you report back |

---

## 2. Your Git autonomy — unchanged and preserved

🔴 **Nothing in this request reduces your autonomy.** You may:

- ✅ Create branches
- ✅ Stage and commit **within your owned paths**
- ✅ Write SHANA code, docs, tests — when gates permit
- ✅ Run SHANA tests — when they exist

🔴 **You may not, and never have been able to:**

- Push — 🔴 **no remote exists**, and pushing is owner-authorised only
- Create a remote
- Edit `docs/cloudlight/**`
- Edit another agent's paths
- Rewrite history or force-push

| Your paths | Owner |
|---|---|
| `docs/shana/**` | 🔴 **You** |
| `docs/shana/architecture/architecture_decisions/**` | 🔴 **You** |
| `questionnaires/SHANA_Feasibility_QA.md` | 🔴 **You** |
| `data/schemas/**` | 🔴 **You authored** — 🔴 shared; changes require the interface-change procedure (`AGENTS.md` §G) |
| `data/templates/**` | You |

---

## 3. 🔴 Requirements — each with the reason it exists

### R1. Confirm you understand the ownership model
**Ask:** does the directory table in `AGENTS.md` §C.1 match your understanding?

### R2. 🔴 Write the ten owed ADR files
**This is the highest-priority item in this request.**

Your ADR index records ten decisions as **owed** — real decisions with
reversal conditions, held in `decision_log.md`, never written as files.

🔴 **Three of them are cited across the subsystem boundary by CloudLight's
safety argument**, and an incoming agent following those citations finds
nothing:

| ADR | Decision | Cited by CloudLight as |
|---|---|---|
| **ADR-001** | No AI model may reach the robot | *"an accepted architecture decision that is **not** CloudLight's to relax"* |
| **ADR-010** | No raw video retention | Privacy invariant |
| **ADR-011** | Simulation before hardware | — |

**Why this matters more than a broken link.** A broken link misleads about
navigation. A missing ADR misleads about **whether a safety invariant exists**.
The *decisions* are real and well-argued. The *citations* do not resolve.

**Evidence:** `CONF-04` in [`../unresolved_conflicts.md`](../unresolved_conflicts.md).

**Accept:** ADR files exist and resolve. **Minimum: ADR-001, ADR-010, ADR-011.**

### R3. 🔴 Publish a `state → status.*` mapping table
Your `cloudlight_integration.md` §3 publishes eight outbound messages
(`status.ready`, `status.assigned`, `status.en_route`, `status.blocked`,
`status.arrived`, `status.delivered`, `status.error`, `status.estop`).

Your `task_state_machine.md` defines **eleven states**.

🔴 **No mapping between them is stated.** Six states have no obvious
counterpart; two messages (`status.blocked`, `status.estop`) have **no
corresponding state at all** in your own machine.

**Accept:** a table mapping every state to its message — or an explicit
statement that some states are internal and not published.

**Evidence:** `CONF-08`.

### R4. Review ADR-016
[`ADR-016`(../../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md)
proposes that your state machine be the **single source of truth** for delivery
status, and that CloudLight hold a declared projection.

🔴 **Its central claim:** `grep -rn "PAUSED" docs/cloudlight/` returns **zero
matches**, while your machine can enter `PAUSED` from **any** state. A paused
robot would therefore be shown to a teacher as **"On the way"** — contradicting
CloudLight's own invariant 8.

🔴 **ADR-016 is `Proposed`, not `Accepted`.** You have three legitimate
responses:

| Response | Meaning |
|---|---|
| **Accept** | Your machine is the single source; CloudLight conforms |
| **Reject with a private-machine argument** | You expose a coarser API — ⚠️ **it must still distinguish `PAUSED`**, or the defect survives |
| **Amend** | Propose different wording |

🔴 **Agent 3 cannot decide this.** It binds your subsystem.

**Also note:** ADR-016 does **not** weaken `ADR-014`. It reinforces the
invariant by requiring the contract to be single-sourced.

### R5. Resolve the identifier model
`data/schemas/delivery.schema.yaml` has `delivery_id` and **no
`idempotency_key`**. CloudLight's contract treats idempotency as a **P0 safety
control**:

> *"Two physical tasks from one tap is a physical safety event, not a data bug."*

**Ask:** does your `delivery_id` equal CloudLight's `request_id`, or do you
issue your own? If your own, the guarantee is broken across the boundary.

**Evidence:** `CONF-06` · `IC-02`.

### R6. Settle the `delivery_state_machine.md` question
CloudLight cites `docs/architecture/delivery_state_machine.md`. 🔴 **No such file
exists anywhere.** The nearest is your `task_state_machine.md`.

🔴 **Recommendation: do not rename your file.** Renaming breaks references
inside `docs/shana/`. Correct CloudLight's citations instead.

**Evidence:** `CONF-05`.

### R7. Consider the occupancy question
Your `cloudlight_integration.md` §2 assigns occupancy to CloudLight, but `D-031`
/ ADR-009 say occupancy is **manual**. These are compatible only by accident.

🔴 **If CloudLight ever derives occupancy, `D-031` and ADR-009 become silently
wrong** — and a network peer would enter SHANA's route-safety inputs, which
`AGENTS.md` §L.6 forbids.

**Ask:** should occupancy ever be a CloudLight input to SHANA?
**Agent 3 recommends: no.** Recorded as `D-05`, `Proposed`.

**Evidence:** `CONF-09`.

### R8. 🔴 The unblocked engineering task
Your own handoff identifies this:

> *"**Task 5 is the answer to 'what should I do next.'** It is the only way to
> produce a working artefact under the current rules, because a simulation needs
> no robot, no network, no school, and no student data."*

🔴 **It is `SH-B9` on the task board, marked `BLOCKED` — because it has not been
started, not because it is blocked.**

⚠️ This is the single highest-value piece of engineering available in this
project, and it is currently the only route to producing a *demonstrable
artefact* without any permission.

### R9. Report conflicts with the proposed workflow
🔴 **If your existing workflow conflicts with
[`GIT_AND_AGENT_WORKFLOW_SPEC.md`](../GIT_AND_AGENT_WORKFLOW_SPEC.md), say so.**

> *"The objective is compatible collaboration, not artificial uniformity. Do not
> force both repositories to use identical tools if their technical requirements
> differ."*

Specific things worth reporting:

| Question | Why it matters |
|---|---|
| Does your branch/commit scheme fit §4–5? | `CONTRIBUTING.md` says `research/<topic>`, `docs/<topic>`, `questions/<topic>` |
| Does anything in §9 misdescribe your checks? | 🔴 **There are none.** Confirm that |
| Does §3.2's future multi-repo model fit your plan? | You may know of a split Agent 3 does not |

### R10. ⚠️ A gap in your documentation — for your decision, not Agent 3's

Your `cloudlight_integration.md` §1 states:

> 🔴 **`docs/cloudlight/` was empty at the 2026-10-11 audit.**

🔴 **It was not empty.** Agent 2's documentation is present and substantial —
30 documents, a 576-question instrument, an argued design position.

**This means your integration proposal was written blind.** Not your fault — but
🔴 **it is now the origin of `CONF-01`, and of the `PAUSED` defect in `CONF-02`.**

**Ask:** after reviewing [`../../../docs/cloudlight/shana_integration.md`](../../../docs/cloudlight/shana_integration.md),
does your position change?

---

## 4. 🔴 What Agent 3 will not do

- ❌ Decide R4, R5, R7 for you
- ❌ Write your owed ADRs
- ❌ Edit any file under `docs/shana/**`
- ❌ Mark any gate `PASSED`
- ❌ Claim you acknowledged this

---

## 5. Acknowledgment checklist

Please confirm each. 🔴 **Agent 3 will record `PENDING` until you do, and will
not record a response that has not arrived.**

- [ ] I understand the directory ownership model
- [ ] I understand the Git boundaries and that I **cannot push** (no remote)
- [ ] I understand I can branch and commit within my own paths
- [ ] I understand the interface-change procedure (`AGENTS.md` §G)
- [ ] I have read `ADR-016` and my response to R4 is: _______________
- [ ] I have identified conflicts with the proposed workflow: _______________
- [ ] I confirm no capability is currently demonstrated
- [ ] I confirm `current_project_state.md` remains accurate

---

## 6. Response format

```markdown
# Response — Agent 1 — YYYY-MM-DD

## Acknowledgment
<checklist with answers>

## R2 — ADR files
<which you wrote, or why not>

## R3 — state → status.* mapping
<table, or "internal states not published">

## R4 — ADR-016
Accept / Reject-with-private-machine / Amend — and why

## R5 — identifier model
<your position>

## R7 — occupancy
<your position>

## R8 — simulation PoC
<status; note it remains unblocked>

## R9 — workflow conflicts
<list, or "none">

## R10 — after reading CloudLight's document
<does your position change?>

## Implementation status
<AGENTS.md §Q labels>

## Tests executed
<exact commands>

## Actual test results
<real output, or "NONE — 0 tests exist">

## Known limitations

## Recommended next action
```

---

## 7. ⚠️ Why nothing here is urgent-by-technical-complexity

Every item above is **documentary**. The blocker on SHANA is not complexity — it
is that **410 questions have been asked and 0 answered**, and the engineering
that *could* proceed unblocked (`R8`) has not been started.

🔴 **The cheapest thing that moves this project is not a code change. It is a
route survey and a conversation with the robot custodian and the lift
provider.**