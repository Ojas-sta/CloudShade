# CloudLight — Agent Handoff

**Date:** 2026-10-11 · **From:** the CloudLight agent · **To:** whoever picks this up next

> **🔴 THE ONE THING TO KNOW BEFORE YOU READ ANYTHING ELSE:**
> **There is no CloudLight code. There are no tests. Nothing runs. Nothing has
> been deployed. The school has never been contacted.**
> Every capability is a proposal. See
> [`implementation_inventory.md`](implementation_inventory.md).

---

## 1. Read these, in this order

| # | Document | Why it is next |
|---|---|---|
| 1 | [`exhibition_evidence_inventory.md`](exhibition_evidence_inventory.md) | ⚠️ **If you are writing an exhibition proposal, this is the constraint document.** It lists 12 claims that must never be made |
| 2 | [`implementation_inventory.md`](implementation_inventory.md) | The factual state of every component |
| 3 | [`../overview/current_project_state.md`](../overview/current_project_state.md) | Condensed state, including what does *not* exist |
| 4 | [`../feasibility_report.md`](../feasibility_report.md) | Every conclusion, classified by evidence strength |
| 5 | [`../research/claim_evidence_matrix.md`](../research/claim_evidence_matrix.md) | 62 claims, each with a verdict on whether it may be exhibited |
| 6 | [`../open_questions_register.md`](../open_questions_register.md) | The blocking questions, sequenced by who can answer |
| 7 | [`../next_actions.md`](../next_actions.md) | The next ten actions, cheapest-first |

---

## 2. Where things are

CloudLight's documents are **flat files plus four subfolders**. The flat layout is
intentional and predates the subfolder convention; a README index maps everything.

| Area | Location |
|---|---|
| Overview, vision, state, glossary | `overview/` |
| Architecture, data model, context | `architecture/` (plus flat `architecture.md`) |
| Feasibility, assumptions, constraints, gates | `feasibility/` (plus flat `feasibility_report.md`) |
| Testing | `testing/` |
| Research and claims | `research/` |
| Handoff | `handoff/` |
| Dashboard, timetable, resources, search, AI, auth, OTA, security, hosting, SHANA | flat files at this level |
| Questionnaire | `../../../questionnaires/Additional_CloudLight_Questionnaire.md` |

⚠️ **Flat files have links pointing at SHANA docs under `docs/architecture/`,
`docs/safety/` and `docs/infrastructure/`.** Those paths are currently correct for
the pre-revert layout. SHANA is mid-way through reverting `docs/shana/` back to
`docs/`. **After that revert lands, re-run a link check** — this is a known,
recorded, unresolved issue (see [`unresolved_dependencies.md`](unresolved_dependencies.md)).

---

## 3. The 90-second summary

**What CloudLight is proposed to be:** a local-first school platform running on
Windows classroom smartboards — timetable and substitution display, teacher
resource browsing and search, authentication, and a request interface to a robot.

**What SHANA is:** the robotics subsystem within it, on a school-owned Unitree Go2
EDU, documented separately in `docs/shana/`.

**The two rules that define everything:**

1. **CloudLight may request a delivery. It may never command the robot.**
   ([`ADR-014`](../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md))
2. **CloudLight must work fully when SHANA is offline.**

**The three findings that shape the design:**

| Finding | Consequence |
|---|---|
| An access point is a Layer-2 bridge and cannot host a platform | The brief's premise conflated connectivity with compute |
| A shared six-digit code is not proof of identity | Codes may only authorise one action or act as a second factor |
| A photograph of a whiteboard is not a record | It is an input to a proposal; humans publish |

**The recommended architecture:** a **modular monolith**, browser-first, one host,
one database, no extra datastores until a *measured* need appears.

**The recommended first release:** one or two smartboards showing a correct,
honest timetable, with human approval before anything publishes — and it depends
on **none** of the open questions.

---

## 4. Ownership boundaries in this repository

| Path | Owner |
|---|---|
| `docs/cloudlight/**` | **CloudLight** |
| `docs/shana/**`, `data/**` | **SHANA** — read only for this work |
| `questionnaires/SHANA_Feasibility_QA.md`, `questionnaires/GEMINI_SHEETS_ONLY_PROMPT.md` | **SHANA** — left unchanged |
| `questionnaires/Additional_CloudLight_Questionnaire.md`, `questionnaires/GEMINI_CLOUDLIGHT_SHEETS_PROMPT.md` | **CloudLight** |
| `README.md` | **Shared** — contains sections for both |
| `docs/architecture/architecture_decisions/` | **Shared ADR index** |

⚠️ **Before writing to any shared file, check whether the other agent is mid-edit.**
Two subagents in this project restructured or rewrote files they did not own; both
were detected and reverted.

---

## 5. What must not be done

| ⛔ | Why |
|---|---|
| Claim any capability works | Nothing is built |
| Claim school approval exists | There has been no contact at all |
| Present `docs/shana/**` content as CloudLight evidence | Different system, different owner, different safety case |
| Let CloudLight send anything SHANA would treat as a motion command | Safety boundary; see ADR-014 |
| Present an unapproved substitution as confirmed | Core correctness invariant |
| Mark a gate passed without evidence | Project rule 10 |
| Present the root JPEG images as project evidence | They are LCD panel photographs, unrelated |
| Run any test and report a result | No tests exist; a reported result would be fabricated |
| Use a six-digit code as identity | Security finding already documented |
| Assume a SHANA API exists | It does not; the two proposals have never been reconciled |

---

## 6. Honest account of how this work was produced

⚠️ **Recorded deliberately, because a future agent needs to know the reliability
profile of these documents.**

- Subagents were used to draft most CloudLight documents in parallel.
- **Drafted content contained fabricated cross-reference paths** (references to a
  `docs/shana/` layout that did not exist). All were detected by link checking and
  repaired.
- **One subagent made a false factual claim** — that a credential-bearing script
  was not gitignored. It **is** gitignored. The claim was corrected.
- **One subagent created a stray pointer file** outside its brief. It was removed.
- One subagent's output was truncated and had to be recovered and re-verified.
- **Parallel drafting risks internal inconsistency.** Every `CL-*`, `FR-*`, `D-0nn`
  and `R-nn` ID cited was verified by grep against its source file, and no
  duplicate or invented ID remains — but cross-document *narrative* consistency
  has not been machine-checked.
- The questionnaire's own 576 questions were generated the same way and then
  validated: 576 unique IDs, 0 duplicates, 0 collisions with `SH-*`, 0 answered
  rows, all 16 columns intact.

Full record: [`../testing/known_failures.md`](../testing/known_failures.md).

---

## 7. State of the questionnaire

| Property | Value |
|---|---|
| CloudLight questions | **576** across 26 categories |
| Answered | **0** |
| Duplicate IDs | **0** |
| Collisions with `SH-*` | **0** |
| SHANA questions (untouched) | 434 |
| Total unanswered across both | **1,010** |

Both Google Sheets conversion prompts exist and are deliberately **separate** —
one per questionnaire. Neither merges them, and neither is permitted to answer
anything.

---

## 8. If you are the third agent

Your task is to produce an exhibition proposal. The honest path:

1. Read [`exhibition_evidence_inventory.md`](exhibition_evidence_inventory.md) §2 —
   the 12 forbidden claims.
2. Use [`../research/claim_evidence_matrix.md`](../research/claim_evidence_matrix.md)
   as your source of truth for what may be said. It has an explicit
   "may appear in an exhibition proposal?" verdict per claim.
3. Lead with the **reasoning exhibits**, not features: the safety boundary, the
   access-point correction, the six-digit-code finding, the human approval gate,
   the honest failure behaviour, and the 576 questions.
4. Present the proposed architecture as a diagram, clearly labelled
   *"proposed — not implemented"*.
5. State the limitations plainly. A proposal that discloses nine deferred
   capabilities and 13 unknowns is **more** credible than one that implies a
   finished product.
6. Do not invent screenshots, test results, school approval or user feedback.
   There is none.

---

## Related

- [`../overview.md`](../overview.md) — CloudLight in one page
- [`../architecture.md`](../architecture.md) — architecture and the modular-monolith argument
- [`../decision_log.md`](../decision_log.md) — decisions with reversal conditions
- [`../risk_register.md`](../risk_register.md) — risks by domain
- [`unresolved_dependencies.md`](unresolved_dependencies.md) — what is blocked and on whom
- [`../shana_integration.md`](../shana_integration.md) — CloudLight's side of the boundary