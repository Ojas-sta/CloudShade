# AGENTS.md — CloudLight / SHANA

**Operating manual for every agent and human contributor working in this
repository.**

This file is the root rulebook. Where it conflicts with any other document,
this file wins. Where it conflicts with a subsystem's own document, the
subsystem document wins **for subsystem-internal matters only** — cross-project
interface, safety and evidence rules remain governed here.

> **Read [`docs/integration/README.md`](docs/integration/README.md) after this
> file, and [`docs/shana/overview/current_project_state.md`](docs/shana/overview/current_project_state.md)
> or [`docs/cloudlight/overview/current_project_state.md`](docs/cloudlight/overview/current_project_state.md)
> before you touch anything at all.**

---

## A. Project identity

Two systems, one repository.

| System | What it is | Where it lives | Owner |
|---|---|---|---|
| **CloudLight** | The **platform**. Classroom displays, timetable and substitution handling, resource browsing, authentication, updates. School-facing, runs on school-local hardware. | `docs/cloudlight/**` | Agent 2 |
| **SHANA** — School Autonomous Notification & Assist | The **robotics subsystem**. Navigation, localisation, route safety, battery reserve, safe stopping, physical delivery. Runs on a school-owned Unitree Go2 EDU. | `docs/shana/**` | Agent 1 |

**CloudLight is the platform. SHANA is one subsystem within it.**

The two governing boundary rules are **accepted decisions, not preferences**:

1. **CloudLight may request a delivery. It may never command the robot.**
   ([`ADR-014`](docs/architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md))
2. **CloudLight must operate fully when SHANA is offline.**
   ([`README.md`](README.md))

🔴 **The single most important fact about this repository, stated once so it
cannot be misread later: there is no source code.** 0 implementation files, 0
tests, 0 builds, 0 CI. Everything in `docs/` is research, design and questions.
**Nothing here is operational, demonstrable, or deployable.** ⚠️ Git has since
been set up — that changed the workflow, **not** the software.

---

## B. System architecture

Full detail: [`docs/integration/system_architecture.md`](docs/integration/system_architecture.md).
Interfaces: [`docs/integration/interface_contracts.md`](docs/integration/interface_contracts.md).

```
   ┌──────────────────────────────────────────────────────────────┐
   │  CLOUDLIGHT  — school platform                               │
   │  displays · timetable · substitutions · resources · search  │
   │  authentication · authorisation · updates                   │
   │                                                              │
   │   ✋ NO motor / path / speed / actuator command path exists  │
   └───────────────────────────┬──────────────────────────────────┘
                               │  outbound: delivery REQUEST only
                               │  inbound:  status only
                               ▼
   ┌──────────────────────────────────────────────────────────────┐
   │  SHANA  — robotics subsystem (independent validator)        │
   │  navigation · localisation · route safety · battery          │
   │  hazard handling · safe stop · physical delivery             │
   │                                                              │
   │  Must remain safe when CloudLight is unreachable.            │
   └──────────────────────────────────────────────────────────────┘
```

**The dependency arrow points CloudLight → SHANA, and it is optional.**
CloudLight must ship and be useful with the integration absent.

---

## C. Repository ownership

### C.1 Directory ownership

| Path | Owner |
|---|---|
| `docs/shana/**` | **Agent 1** |
| `data/schemas/**` | **Agent 1 authored**, shared — changes require the interface-change procedure (§H) |
| `questionnaires/SHANA_Feasibility_QA.md`, `questionnaires/SHANA_*` | **Agent 1** |
| `docs/cloudlight/**` | **Agent 2** |
| `questionnaires/Additional_CloudLight_Questionnaire.md`, `questionnaires/GEMINI_CLOUDLIGHT*` | **Agent 2** |
| `AGENTS.md`, `docs/integration/**`, `docs/exhibition/**` | **Agent 3** |
| `docs/architecture/**` | **Agent 3** (cross-project ADRs). Subsystem-internal ADRs live under the owning subsystem |
| Root governance files (`README`, `CONTRIBUTING`, `SECURITY`, `.gitignore`) | **Agent 3** coordinates; changes affecting a subsystem require that owner's review |

The full, detailed ownership table — including who may change what, and who
must be consulted — is
[`docs/integration/AGENT_COORDINATION.md`](docs/integration/AGENT_COORDINATION.md).

### C.2 Agent responsibilities

| Agent | Owns | May commit/push/PR in this repo |
|---|---|---|
| **Agent 1 — SHANA** | SHANA source, tests, research, hardware, safety, operations docs | ✅ yes, within `docs/shana/**` and its questionnaire/data-schemas |
| **Agent 2 — CloudLight** | CloudLight source, tests, research, architecture, security docs | ✅ yes, within `docs/cloudlight/**` and its questionnaire |
| **Agent 3 — Integration** | `AGENTS.md`, `docs/integration/**`, `docs/exhibition/**`, cross-project ADRs, task board, evidence audit, Antigravity handoff | ✅ yes, within its owned paths |

### C.3 Hard ownership rules

- 🔴 **An agent must not silently modify another agent's subsystem.** Not
  "fixing a typo". If you find an error in another owner's files, **write a
  request** (`docs/integration/agent-requests/`) or open a PR comment. An
  unrequested edit destroys the owner's ability to trust their own directory.
- **Exceptions that do NOT require a request:** fixing a broken internal link
  whose target unambiguously exists, fixing a typo with zero semantic content,
  correcting a path to a file you have verified. In all three cases: state the
  change in the commit body and notify the owner in the handoff.
- **Exceptions that ALWAYS require a request:** changing a status label,
  changing a claim, changing an evidence class, changing a boundary or
  invariant, adding or removing a requirement.
- 🔴 **Agent 3 must not assume it has write access to any other repository.**
  This project currently has **no other repository** (§E).

---

## D. Git boundaries

🔴 **Do not add remotes, change branch protection, publish tags, or merge a
dependent PR before its base without explicit owner authorisation.**

✅ **Pushing feature branches and opening PRs is now expected, not exceptional.**
See [`docs/integration/AGENT_BRANCH_AND_PR_WORKFLOW.md`](docs/integration/AGENT_BRANCH_AND_PR_WORKFLOW.md).

| Rule | Detail |
|---|---|
| Remote | ✅ **1** — `origin` → `Ojas-sta/CloudShade`, **public**, authorised by the owner 2026-10-11 |
| Protected branches | `main`, `SHANA_main`, `CloudLight_Main` — 🔴 all require a PR + **1 approving review** |
| Local commits | Expected and encouraged |
| Pushing a feature branch | ✅ Allowed |
| 🔴 **Pushing to a lane or `main`** | **Refused by GitHub.** PR only |
| Force-push | 🔴 Refused by protection on all 3 branches |
| Deleting a branch | 🔴 Refused by protection |
| Removing protection | Owner only |
| History rewriting | Never. **A decision without a recorded reversal condition becomes a decision nobody dares revisit** — that applies to Git history too |
| Destructive cleanup | `git reset --hard`, `git clean -fd`, `git checkout .` are **not** routine cleanup. Use them only to discard *your own* work, and only after confirming nothing else is in the tree |
| Never discard others' work | If the tree has changes you did not make, leave them and say so |

Full procedure: [`docs/integration/GIT_AND_AGENT_WORKFLOW_SPEC.md`](docs/integration/GIT_AND_AGENT_WORKFLOW_SPEC.md).

---

## E. Repository topology — the actual situation

🔴 **Correct a common misreading.** There is **one repository** containing both
subsystems' documentation. There is **no** SHANA repository and **no**
CloudLight repository. ⚠️ **As of 2026-10-11 there is one public remote with
three protected branches, and one worktree per agent** — so the two subsystems
no longer collide, but they are still **one repository, not three**.

| Fact | Value | Verified by |
|---|---|---|
| Repository root | this directory | `git rev-parse --show-toplevel` |
| Branches | `main`, `SHANA_main`, `CloudLight_Main` | `git branch -a` |
| Remotes | **1** — `Ojas-sta/CloudShade`, **public** | `git remote -v` |
| Remote branches | **3**, all protected | `git ls-remote --heads origin` |
| Worktrees | **3** — one per agent, plus the root | `git worktree list` |
| Subsystem repositories | **none exist** | filesystem inspection |
| CI / hosting | hosting ✅ · CI **none** | no `.github/workflows/` |
| Branch protection | ✅ 1 approving review, all 3 branches | `gh api .../branches/{b}/protection` |
| Source code | **0 files** | filesystem scan for `.py .js .ts .tsx .go .c .cpp .java .rs` |
| Tests | **0** | no test directory anywhere |

⚠️ **Updated 2026-10-11:** the owner has since authorised a remote and per-agent
worktrees. There is still **one** repository — 🔴 **not three** — but the two
subsystems now have **separate long-lived branches and separate working
directories**, so they no longer collide. See
[`ADR-015`](docs/architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md)
and [`docs/integration/AGENT_BRANCH_AND_PR_WORKFLOW.md`](docs/integration/AGENT_BRANCH_AND_PR_WORKFLOW.md).

**Therefore:** the "three agents, separate repositories, cross-repository PRs"
model does **not** match reality. Agent 3 has documented what *is* true in
[`docs/integration/GIT_AND_AGENT_WORKFLOW_SPEC.md`](docs/integration/GIT_AND_AGENT_WORKFLOW_SPEC.md)
§2, and specified what the workflow becomes **if and when** separate repositories
or a hosting platform are introduced. That work is `SPECIFIED`, not `IMPLEMENTED`.

See [`ADR-015`](docs/architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md).

---

## F. Pull-request and stacked-PR conventions

Applicable **when** a hosting platform exists (§E). Until then, the same
discipline is applied through commits plus
[`docs/integration/AGENT_HANDOFF_PROTOCOL.md`](docs/integration/AGENT_HANDOFF_PROTOCOL.md).

**Every meaningful change must state:** summary · motivation · scope · files
affected · dependencies · **tests run and their actual output** · documentation
changed · known limitations · security/safety implications · breaking changes ·
reviewer requirements.

🔴 **A PR must never claim a feature is tested because the code compiles, and
must never claim it is implemented because a document describes it.**

**Stacked PRs** are used when a change genuinely layers — typically a shared
contract first, then each subsystem's implementation, then integration tests
and documentation. The base PR must be identified, dependencies declared
explicitly, and the stack merged in dependency order. Never force-push a shared
branch without informing reviewers. Full procedure in the workflow spec §7.

---

## G. Cross-repository specifications

Any change to a **shared interface** — the delivery state machine, the API
surface, authentication between subsystems, location naming, the data model,
error handling, or the safety boundary — requires the **interface-change
procedure** before implementation begins.

The proposer must document:

1. Current behaviour
2. Proposed behaviour
3. Reason for the change
4. Backward-compatibility impact
5. Required change in **each** affected subsystem
6. Tests required
7. Migration strategy, if any
8. Rollback implications
9. Decision status (`PROPOSED` / `ACCEPTED` / `REJECTED` / `SUPERSEDED`)

🔴 **Do not proceed on an assumption that lives only in one agent's
conversation.** If it is not written in `docs/integration/`, it does not exist.

Template and register:
[`docs/integration/interface_contracts.md`](docs/integration/interface_contracts.md).

---

## H. Documentation requirements

- 🔴 **Documentation must match implementation.** Where a document describes
  behaviour, it must state whether that behaviour is `DESIGN_ONLY` or
  `IMPLEMENTED`. Phrasing in the present tense is *not* evidence of existence.
  Both `current_project_state.md` files exist specifically because the
  documentation otherwise reads as though a system exists.
- **Record uncertainty rather than resolving it silently.** Where two sources
  conflict, show the conflict.
- **Distinguish absence of permission from absence of capability.** They need
  different fixes and different owners.
- **Keep internal links resolving.** A broken link is a defect. Before
  committing, verify links in any file you touch.
- Use UK-influenced English consistently, matching existing documents.
- Keep tables narrow enough to read in a diff.

---

## I. Research requirements

- **Prefer primary sources** — official vendor documentation, standards,
  specifications, published papers.
- **Every significant claim carries an evidence label:**
  `VERIFIED` · `DOCUMENTED_NOT_HARDWARE_VERIFIED` · `PLAUSIBLE_REQUIRES_TESTING`
  · `UNKNOWN` · `REQUIRES_SCHOOL_PERMISSION` ·
  `REQUIRES_MANUFACTURER_OR_SERVICE_PROVIDER` ·
  `NOT_RECOMMENDED_OR_NOT_PERMITTED_WITHOUT_APPROVAL`
- **Citations record:** title · publisher · URL · publication/update date if
  available · date accessed · the specific claim supported · limitations ·
  whether the source applies to *our exact hardware revision*.
- 🔴 **`NOT FOUND` is a valid and valuable finding.** If you could not access
  a source, say so. Do not write as though you read it.
- 🔴 **Never use a marketing claim as proof of safety or capability.**
- 🔴 **A vendor documenting a feature proves the feature exists — not that it
  behaves the same way on the equipment this school happens to own.**
- **Do not add technology because it looks impressive.** Every proposed
  component needs a named problem it solves.

---

## J. Testing expectations

🔴 **There are currently 0 tests and 0 test runs. Nothing in this repository
has ever been executed.**

The moment implementation exists:

- Test claims must match the code that was actually run. Record the command and
  its real output.
- Integration claims require an integration test that exercises **both**
  subsystems. There is currently no such test and therefore no such claim may
  be made.
- A documented strategy that has never been executed is not evidence of
  testing. `docs/shana/testing/` and `docs/cloudlight/testing/` currently
  describe strategies only.
- 🔴 **"Tests pass" is never a statement about safety.** Safety requires
  physical, supervised validation — see §L and the go/no-go gates.

---

## K. Security and secret handling

🔴 **Never commit, and never paste into an issue or document:**

Wi-Fi keys or passwords · CCTV credentials or stream URLs · API tokens · robot
or server credentials · private network topology, IP assignments or VLAN
configuration · student or staff personal data · floor plans or building
layouts not approved for circulation.

Known exposure requiring owner action:
[`start_opencode_server.sh`](docs/shana/safety/security.md) contains a
**hardcoded plaintext administrative password** and binds a server to
`0.0.0.0`. It is gitignored and excluded from every commit. **The credential
must still be rotated** — it is on disk in cleartext and must be assumed
exposed. **No agent may quote or copy its value into any document.**

🔴 **Do not probe, scan, or connect to school infrastructure.** No network
scanning, no CCTV connection, no lift-system contact, no device enumeration.
Every school fact is a **question with a named owner**, never a finding.

If you discover sensitive material already committed, stop and tell the
repository owner immediately. See [`SECURITY.md`](SECURITY.md).

---

## L. Safety and privacy

These are **not** style preferences. Violating them makes the work worthless or
unsafe.

1. 🔴 **No AI model output ever reaches the robot** — directly or indirectly.
   A deterministic validator sits in the loop.
2. 🔴 **No motor, path, speed or actuator command path exists in CloudLight**,
   in any mode — including debug, admin or exhibition.
3. 🔴 **`system.stop` is honoured unconditionally and immediately.** No peer can
   talk SHANA out of a stop.
4. 🔴 **Delivery completion requires explicit recipient confirmation** — never
   inferred from arrival.
5. 🔴 **A blocked or unknown route is never traversed.** Fail closed.
6. 🔴 **Loss of communications produces a defined safe behaviour**, never a
   best-effort continuation.
7. 🔴 **A human spotter is required at every stage**, with unilateral stop
   authority.
8. 🔴 **No lift modification. Ever.**
9. 🔴 **No student data is collected, and no person is ever identified.** No
   facial recognition, no identity recognition, no biometrics, no persistent
   individual tracking, no automated misconduct detection.
10. 🔴 **No raw video ever leaves the robot.**
11. **A shared six-digit code is never proof of identity.** It may only
    authorise one action, or act as a second factor.
12. **A gate is never `PASSED` without evidence attached.**

🔴 **Successful software development does not authorise robot operation near
students.** Technical feasibility and institutional authorisation are
different gates, owned by different people.

---

## M. Architecture decision records

ADRs live in `docs/architecture/architecture_decisions/` (cross-project) and
`docs/shana/architecture/architecture_decisions/` (SHANA-internal).

Every ADR records: ID · title · date · status · context · options considered ·
decision · rationale · evidence · consequences · rejected alternatives ·
follow-up tasks · **reversal condition**.

- 🔴 **Never record a proposal as an accepted decision until the affected
  owners have reviewed it.** Status values: `Proposed` · `Accepted` ·
  `Provisional` · `Deprecated` · `Superseded`.
- ⚠️ **An `Accepted` decision is not a shipped feature.**
- ⚠️ **Referencing an ADR that does not exist is a defect.** Several documents
  currently cite ADR IDs that are recorded in a decision log but never written
  as ADR files. Verify an ADR file exists before citing it.
  See [`docs/integration/unresolved_conflicts.md`](docs/integration/unresolved_conflicts.md).

---

## N. Integration procedure

1. Identify the task and its **owner** (check §C.1).
2. Identify cross-project dependencies.
3. Agree the interface contract where needed (§G).
4. Create the branch (§F).
5. Run the checks that **actually exist** — currently: none. Do not claim a
   check you have not run.
6. Review the change against this file.
7. Resolve feedback.
8. Merge dependencies in dependency order.
9. Update `docs/integration/TASK_BOARD.md`.
10. Update evidence records if status changed.
11. Produce a handoff (§P).

---

## O. Conflict resolution

1. **Identify the conflict** and write it in
   `docs/integration/unresolved_conflicts.md` with both positions quoted.
2. **Assign an owner** for resolution — the subsystem that must change.
3. **Raise a request** in `docs/integration/agent-requests/`. Do not edit the
   other subsystem's files to force compatibility.
4. **Prefer the safety-conservative reading** while unresolved.
5. **Record the resolution** as an ADR, with a reversal condition.

🔴 **The safe reading wins by default.** When two documents disagree about
safety, privacy or permissions, assume the more restrictive one until proven
otherwise.

---

## P. Agent handoff requirements

🔴 **A handoff must make the work understandable without access to the previous
conversation.** Template:
[`docs/integration/AGENT_HANDOFF_PROTOCOL.md`](docs/integration/AGENT_HANDOFF_PROTOCOL.md).

Required: objective · work completed · files changed · **implementation status
using §Q vocabulary** · tests executed · **actual test results** · research
sources · decisions made · dependencies · known limitations · unresolved
questions · breaking changes · required permissions · relevant PRs ·
recommended next action.

🔴 **Never fabricate** a test result, an agent response, a Git operation, a
permission, a photograph or a quotation. `"No tests run"` is a complete and
acceptable answer. `"PENDING — not yet delivered"` is a complete and
acceptable answer.

---

## Q. Status vocabulary — use these labels, exactly

| Label | Means |
|---|---|
| `IMPLEMENTED_AND_TESTED` | Code exists, and a named test has been run against it and passed |
| `IMPLEMENTED_NOT_TESTED` | Code exists; no test has been run |
| `PARTIALLY_IMPLEMENTED` | Some of the described behaviour exists; the rest does not |
| `DESIGN_ONLY` | Specified or argued. No code. Nothing built |
| `RESEARCH_ONLY` | The subject was investigated. No design, no code |
| `BLOCKED` | Cannot proceed. Blocker and owner must be named |
| `UNKNOWN` | Insufficient information. **This is a valid, valuable answer** |
| `DEPRECATED` | No longer applies |

🔴 **Use these labels in inventories, handoffs, the task board, and evidence
records.** Do not substitute prose like "working", "nearly done", or "basically
complete". 🔴 **Never describe a `DESIGN_ONLY` component in the present tense
without its label** — that is the specific failure mode
`current_project_state.md` exists to prevent.

### Definition of done

A task is `DONE` only when **all** of these hold:

- [ ] Acceptance criteria are satisfied
- [ ] Required implementation exists (or the task was documentation-only)
- [ ] Relevant tests have been run where applicable — **and the real output recorded**
- [ ] Documentation matches the actual result, using §Q labels
- [ ] Dependencies and limitations are recorded
- [ ] Required reviews are complete
- [ ] Interface changes have been communicated to affected owners
- [ ] No known critical blocker remains undisclosed

⚠️ A documentation-only task may be `DONE` without implementing a feature — but
it **must not** claim the feature is implemented.

---

## R. Reporting requirements

Report honestly, separating what happened from what is proposed.

| Always distinguish | |
|---|---|
| `SPECIFIED` | required behaviour documented |
| `REQUESTED` | another agent has been asked |
| `IMPLEMENTED` | the change exists |
| `VERIFIED` | the change has been inspected or tested |
| `BLOCKED` | implementation cannot proceed |
| `NOT APPLICABLE` | the requirement does not fit |

🔴 **Never describe an unexecuted action as completed.** Never claim a PR was
opened, approved or merged without evidence. Never claim the working tree is
clean without running `git status`.

---

## Related documents

| Document | Purpose |
|---|---|
| [`docs/integration/README.md`](docs/integration/README.md) | Index of the integration layer |
| [`docs/integration/GIT_AND_AGENT_WORKFLOW_SPEC.md`](docs/integration/GIT_AND_AGENT_WORKFLOW_SPEC.md) | Git + cross-project workflow specification |
| [`docs/integration/AGENT_COORDINATION.md`](docs/integration/AGENT_COORDINATION.md) | Ownership model in detail |
| [`docs/integration/AGENT_HANDOFF_PROTOCOL.md`](docs/integration/AGENT_HANDOFF_PROTOCOL.md) | Handoff template |
| [`docs/integration/TASK_BOARD.md`](docs/integration/TASK_BOARD.md) | Cross-project task tracking |
| [`docs/integration/interface_contracts.md`](docs/integration/interface_contracts.md) | Shared interface register |
| [`docs/integration/unresolved_conflicts.md`](docs/integration/unresolved_conflicts.md) | Open cross-project conflicts |
| [`docs/exhibition/claim_evidence_matrix.md`](docs/exhibition/claim_evidence_matrix.md) | What may be claimed, and on what evidence |
| [`docs/exhibition/ANTIGRAVITY_EXHIBITION_HANDOFF.md`](docs/exhibition/ANTIGRAVITY_EXHIBITION_HANDOFF.md) | Handoff to the proposal author |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Contribution rules and evidence labels |
| [`SECURITY.md`](SECURITY.md) | Security reporting and secret handling |