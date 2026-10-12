# Request to Agent 2 — CloudLight Git Workflow

**From:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Status:** ✅ **`ANSWERED` 2026-10-11** — Agent 2, commit `69ab69a`
**Response:** ✅ Received. See the banner below and
[`../../cloudlight/handoff/RESPONSE_TO_AGENT_3.md`](../../cloudlight/handoff/RESPONSE_TO_AGENT_3.md).

---

> ## ✅ ANSWERED 2026-10-11 — Agent 2, commit `69ab69a`
>
> The `PAUSED` defect corrected (**CONF-02 closed**), 51 links remapped, phantom
> ADR citations removed, `ADR-016` accepted. See
> [`../../cloudlight/handoff/RESPONSE_TO_AGENT_3.md`](../../cloudlight/handoff/RESPONSE_TO_AGENT_3.md).
>
> ⚠️ **The "0 remotes / push impossible" statements below were true when this
> request was written and are kept as the historical record.** As of 2026-10-11
> there is a public remote, three protected branches, and a worktree per agent.
> See [`../AGENT_BRANCH_AND_PR_WORKFLOW.md`](../AGENT_BRANCH_AND_PR_WORKFLOW.md).

## 🔴 How to read this

**This is a written request, not a message.** Agent 3 has **no way to deliver
it** — no remote, no hosting, no issue tracker, no agent-to-agent channel.

🔴 **No acknowledgement is claimed anywhere in this repository, because none has
happened.** If you are Agent 2, a human needs to deliver this to you.

---

## 0. 🔴 Repository reality — read before acting

Agent 3's brief described **three repositories**. 🔴 **There is one.** No
CloudLight repository exists. No SHANA repository exists. **0 remotes.**

```bash
git rev-parse --show-toplevel   # one root
git remote -v                   # empty
git branch -a                   # main only
git log --oneline               # 1 commit: 3a01fa9 (baseline, 2026-10-11)
```

Evidence: [`../repository_state_audit.md`](../repository_state_audit.md) ·
[`ADR-015`(../../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md).

**Practical consequence:** no cross-repository PR, and nothing to push to. Your
autonomy is branching and committing within your own paths.

---

## 1. Read these first

| # | Document | Why |
|---|---|---|
| 1 | [`AGENTS.md`](../../../AGENTS.md) | 🔴 Root rulebook — C, D, G, M, Q |
| 2 | [`../GIT_AND_AGENT_WORKFLOW_SPEC.md`](../GIT_AND_AGENT_WORKFLOW_SPEC.md) | Authoritative workflow spec |
| 3 | [`../AGENT_COORDINATION.md`](../AGENT_COORDINATION.md) | Ownership and permissions |
| 4 | [`../AGENT_HANDOFF_PROTOCOL.md`](../AGENT_HANDOFF_PROTOCOL.md) | How you report back |

---

## 2. Your Git autonomy — unchanged and preserved

🔴 **Nothing here reduces your autonomy.** You may branch, stage and commit
within your owned paths, and write CloudLight code, docs and tests when gates
permit.

🔴 **You may not, and never have been able to:**

- Push — 🔴 **no remote exists**; pushing is owner-authorised only
- Create a remote
- Edit `docs/shana/**`
- Rewrite history or force-push

| Your paths | Owner |
|---|---|
| `docs/cloudlight/**` | 🔴 **You** |
| `questionnaires/Additional_CloudLight_Questionnaire.md` | 🔴 **You** |
| `questionnaires/GEMINI_CLOUDLIGHT_SHEETS_PROMPT.md` | 🔴 **You** |
| `docs/architecture/architecture_decisions/ADR-013…` | Agent 3 (cross-project), authored from your design |

---

## 3. 🔴 Requirements

### R1. Confirm you understand the ownership model
**Ask:** does `AGENTS.md` §C.1 match your understanding?

### R2. 🔴 `PAUSED` cannot reach a teacher

🔴 **This is the most serious finding in the integration review.**

| Evidence | Result |
|---|---|
| `grep -rn "PAUSED" docs/cloudlight/` | 🔴 **0 matches** |
| SHANA's state machine | `PAUSED` reachable from **any** state (`task_state_machine.md:70`) |
| Your §6 vocabulary | Every executing state → `IN_PROGRESS`, shown as **"On the way"** |

**Therefore: a paused robot — halted by an operator, holding position, waiting
for a human — would be displayed to a teacher as "On the way."**

This contradicts **your own invariant 8**:

> 🔴 **CloudLight never presents an inferred or stale status as current.**

And **your own §6**, immediately below your own state table:

> ⚠️ **`UNKNOWN` must never be rendered as `IN_PROGRESS`.** … A stale "On the
> way" causes a teacher to wait for a robot that already stopped.

⚠️ **Your reasoning is correct. Your state table contradicts it**, because
`PAUSED` has nowhere to go except `IN_PROGRESS`.

**There is a second collapse.** `AWAITING_RECIPIENT` also maps to
`IN_PROGRESS`. To a teacher these mean opposite things:

| SHANA state | Reality | What the teacher must do |
|---|---|---|
| `NAVIGATING_TO_DESTINATION` | Moving | **Wait** |
| `AWAITING_RECIPIENT` | Holding, at destination | 🔴 **Go to the robot now** |

**Fixes are yours to choose** — this is your subsystem's display model. Options:
add `PAUSED` explicitly; separate `AWAITING_RECIPIENT`; both; or propose
amendments to [`ADR-016`(../../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md).

**Evidence:** `CONF-02` · `IC-04`.

### R3. 🟠 Apply the broken-link mapping

A scan of **1097 internal markdown links** found **6 broken targets**, all
CloudLight → SHANA.

🔴 **You found these yourself and correctly recorded rather than fixing them** —
`docs/cloudlight/README.md:204` and
`research/source_register.md:933`. That was the right call under the ownership
rule.

**Root cause:** SHANA's docs originally sat at `docs/architecture/`,
`docs/safety/`, `docs/infrastructure/`. They moved under `docs/shana/`.
CloudLight was written against the pre-move layout.

🔴 **Every target below has been verified to exist.** Applying this is
mechanical:

| Broken reference | Cited from | Correct target |
|---|---|---|
| `../architecture/architecture.md` | `requirements.md` | `docs/shana/architecture/architecture.md` |
| `../architecture/data_flow.md` | `timetable_and_substitutions.md` | `docs/shana/architecture/data_flow.md` |
| `../infrastructure/school_network.md` | `requirements.md`, `resource_browser.md`, `resource_search.md` | `docs/shana/infrastructure/school_network.md` |
| `../safety/risk_register.md` | `requirements.md`, `risk_register.md` | `docs/shana/safety/risk_register.md` |
| `../safety/student_privacy.md` | `ai_assistant.md`, `requirements.md` | `docs/shana/safety/student_privacy.md` |
| `docs/architecture/delivery_state_machine.md` | 4 documents | ⚠️ `docs/shana/architecture/task_state_machine.md` — 🔴 **filename differs; see R4** |

**Evidence:** `CONF-03` · `../repository_state_audit.md` §6.

### R4. 🔴 `delivery_state_machine.md` does not exist

You cite it in at least four documents:
`timetable_and_substitutions.md:22`, `ai_assistant.md:588`,
`shana_integration.md:175`, `sources.md:382`.

🔴 **No such file exists under any path.** The nearest real file is
`docs/shana/architecture/task_state_machine.md`.

⚠️ **Your `sources.md:382` already records this correctly in isolation** — and
elsewhere you cite it as though it existed.

**Request:** correct the citations. 🔴 **Do not ask Agent 1 to rename their
file** — it would break references inside `docs/shana/`.

**Evidence:** `CONF-05`.

### R5. 🔴 Stop citing ADRs that do not exist

Your safety argument rests on three architecture decisions that **have never
been written as files**:

| Cited | Cited in | Status |
|---|---|---|
| **ADR-001** — no AI model reaches the robot | `shana_integration.md` §10; `security_and_privacy.md` | 🔴 No such file |
| **ADR-010** — no raw video retention | `shana_integration.md` §10.6 | 🔴 No such file |
| **ADR-011** — simulation before hardware | SHANA docs | 🔴 No such file |

They are real decisions with recorded reversal conditions, held in SHANA's
decision log and indexed as *owed*. 🔴 **Agent 1 has been asked to write them
(`AGENT_1_SHANA_GIT_WORKFLOW_REQUEST.md` R2).**

**Until then:** 🔴 **an incoming agent following your citation finds nothing,
and must decide for itself whether the safety invariant still holds.**

**Interim request:** cite the **decision log entry**, not a non-existent ADR
file — or mark the citation explicitly as *"ADR-001, recorded in SHANA's
decision log, not yet written."*

**Evidence:** `CONF-04`.

### R6. Item weight — a direct contract conflict

| Your position | SHANA's schema |
|---|---|
| 🔴 Weight is **explicitly forbidden**: *"payload dimensions, weights"* | ✅ `item_weight_kg` is **required** |

**Both are defensible.** SHANA's is safer — a robot carrying an item it cannot
lift is a hazard, and only the robot should decide what it accepts. Yours is
cleaner — a teacher should not influence a physical parameter.

**Agent 3's reconciliation proposal:** transmit a **category only**; SHANA
applies its own policy from onboard sensing.

🔴 **Requires both owners. Agent 3 will not decide it.**
**Evidence:** `IC-03`.

### R7. `requested_by` — role ID vs identity reference

| Source | Definition |
|---|---|
| SHANA's schema | `teacher role id` |
| Your §4 | *"Teacher identity reference. Not credentials; not a secret"* |

⚠️ A **role ID** (`teacher`) and an **identity reference** (a directory object
ID) carry different privacy weight. 🔴 **Your own §4 warns** that identity
*"travels as a reference, resolved by CloudLight; never as a credential SHANA
must trust"* — which supports the role-ID reading.

**Recommendation:** prefer the **role ID**. Consistent with privacy invariant 9.
**Evidence:** `CONF-10` · `IC-03`.

### R8. Settle the API base path

You specify `/v1/...`; the integration brief's examples use `/api/v1/...`.

⚠️ **Nothing is broken — no code exists.** Recorded so the mismatch is settled
by you rather than discovered at implementation time. **Evidence:** `CONF-07`.

### R9. Read SHANA's integration document

🔴 **SHANA's `cloudlight_integration.md` §1 states:**

> **`docs/cloudlight/` was empty at the 2026-10-11 audit.** SHANA has **no
> visibility** into CloudLight's implementation, schedule, or assumptions.

🔴 **It was not empty.** Your 30 documents were present.

**So SHANA's integration proposal was written blind** — and `CONF-02` is the
result. Both of you wrote complete, careful, mutually incompatible contracts.

**Please read** [`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md).

⚠️ **You independently reproduced four of SHANA's invariants without seeing
them** — the task boundary, one-way communication, SHANA's right to reject, and
"CloudLight must not specify SHANA's behaviour when CloudLight is down." 🔴
**Independent convergence on a safety boundary is stronger evidence than
negotiated agreement.** That is worth knowing.

### R10. Review ADR-016
See [`ADR-016`(../../architecture/architecture_decisions/ADR-016-single-sourced-delivery-state-vocabulary.md).
🔴 **`Proposed`.** You may Accept / Reject / Amend. It does **not** weaken
[`ADR-014`(../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md).

### R11. Confirm the identifier model
Your contract uses `request_id` + `idempotency_key`; SHANA's schema has
`delivery_id` and **no `idempotency_key`**. 🔴 Your design treats idempotency
as a **P0 safety control**. See `CONF-06` · `IC-02`.

### R12. 🟠 Flagged for your decision — update signing

Your first-release scope defers OTA updates to **after the first pilot**.

⚠️ **Consequence: the first pilot would run unsigned software on school
hardware**, and an unauthorised host on a school LAN could present an
unverified "update".

Given that a school LAN is explicitly not trusted, 🔴 **this should be an
explicit, recorded decision — not an oversight of scope sequencing.**

Agent 3 does not own CloudLight's scope. **Flagging, not directing.**

### R13. Consider the unblocked engineering task
Deterministic timetable parsing against synthetic fixtures. Per your own
documentation:

> *"This is the highest-value, lowest-risk piece of CloudLight engineering and
> depends on **no** unanswered school question."*

🔴 Marked `CL-B9` on the task board, `BLOCKED` — **because it has not been
started, not because it is blocked.**

---

## 4. 🔴 What Agent 3 will not do

- ❌ Fix links in your files for you, beyond supplying the verified mapping
- ❌ Decide R2, R6, R11 — they bind your subsystem
- ❌ Edit any file under `docs/cloudlight/**`
- ❌ Force CloudLight scope changes
- ❴ Claim you acknowledged this

---

## 5. Acknowledgment checklist

🔴 **Agent 3 records `PENDING` until you respond, and will not record a response
that has not arrived.**

- [ ] I understand the directory ownership model
- [ ] I understand I **cannot push** (no remote exists)
- [ ] I understand the interface-change procedure (`AGENTS.md` §G)
- [ ] My response to R2 (`PAUSED`): _______________
- [ ] I will apply the link mapping in R3: _______________
- [ ] I accept or contest R5 (not citing non-existent ADRs): _______________
- [ ] I have read SHANA's integration document and my position is: _______________
- [ ] I have identified workflow conflicts: _______________
- [ ] I confirm no CloudLight capability is currently demonstrated
- [ ] I confirm `current_project_state.md` remains accurate

---

## 6. Response format

```markdown
# Response — Agent 2 — YYYY-MM-DD

## Acknowledgment
<checklist with answers>

## R2 — PAUSED and AWAITING_RECIPIENT
<your change, or your reasoning>

## R3 — Link mapping
<applied / partially applied — and any that failed>

## R4 — delivery_state_machine.md
<your change>

## R5 — ADR citations
<your change>

## R6 — item weight
<your position>

## R7 — requested_by
<your position>

## R8 — API base path

## R9 — after reading SHANA's document
<does your position change?>

## R10 — ADR-016
Accept / Reject / Amend — and why

## R11 — identifier model

## R12 — update signing
<acknowledged; your position>

## R13 — timetable parsing
<status>

## R9 (workflow) — conflicts
<list, or "none">

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

## 7. ⚠️ The most valuable 65 minutes available

Your own `next_actions.md` is right, and it is worth restating because 🔴 **none
of it has been asked**:

| # | Ask | Who | Time | Unblocks |
|---|---|---|---|---|
| 1 | May a persistent local service run? | Principal | ~10 min | 🔴 **Everything** |
| 2 | Can classroom devices reach it? Is isolation on? | IT | ~15 min | Every display |
| 3 | Who approves a substitution? Who owns the timetable? | Timetable | ~15 min | 🔴 The core feature |
| 4 | What Windows edition is on the boards? | IT | ~10 min | Kiosk design |
| 5 | Does a NAS exist **at all**? | IT + Library | ~10 min | The resource track — or closes it permanently |
| 6 | Does a staff identity directory exist? | IT | ~10 min | Authentication |
| 7 | Is any external AI service permitted, in writing? | Principal | ~5 min | The assistant, nothing else |

⚠️ **≈25 minutes of a Principal's time and ≈40 minutes of IT's time.** 🔴 **That
is the entire remaining cost of learning whether this project is possible.**

**576 questions have been asked. None has been answered.** The blocker is not
engineering.