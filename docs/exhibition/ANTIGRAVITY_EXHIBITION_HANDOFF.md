# Antigravity Exhibition Proposal — Handoff Prompt

**From:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Status:** `DESIGN_ONLY` — 🔴 **a specification, not a proposal**

---

# ⬇️⬇️⬇️ THE PROMPT BEGINS ⬇️⬇️⬇️

You are producing a **school exhibition proposal** for a project called
**CloudLight**, which has a robotics subsystem called **SHANA**.

🔴 **Read the entire "MANDATORY GROUND RULES" section before anything else.** It
contains the constraints that determine whether this proposal is credible or
ruined. Getting this wrong costs more than an unimpressive proposal.

---

## 1. 🔴 MANDATORY GROUND RULES

### 1.1 🔴 There is no software

Before you write a word, internalise this:

| Fact | Value |
|---|---|
| **Source files** | 🔴 **0** |
| **Tests written / run** | 🔴 **0 / 0** |
| **Builds / CI** | 🔴 **0 / none** |
| **Deployments** | 🔴 **0** |
| **Robots operated** | 🔴 **never** |
| **Screenshots of any system** | 🔴 **none exist** |
| **Photographs of the robot** | 🔴 **none exist** |
| **Measurements** | 🔴 **0** |
| **Go/no-go gates passed** | 🔴 **0 of 20** |
| **Permissions granted** | 🔴 **0 of 104** |
| **Questions answered by the school** | 🔴 **0 of ≈1,000** |

🔴 **You are writing about a completed feasibility study.** Not a product. Not
a prototype. A study.

### 1.2 🔴 Prohibited without exception

- ❌ Claiming any capability is implemented, tested, working, operational or
  demonstrated
- ❌ Inventing statistics, percentages, timelines, costs, metrics or results
- ❌ Inventing screenshots, photographs, mockups presented as real output
- ❌ Implying school approval, partnership or endorsement
- ❌ Implying any robot has been operated
- ❌ Implying lift, stair, CCTV or wet-floor capability
- ❌ Suggesting facial recognition, student tracking or misconduct detection
- ❌ Presenting a generated illustration as a photograph of real hardware
- ❌ Writing anything in the present tense that implies a built system

### 1.3 🔴 A specific trap — the eight JPEGs

Eight JPEG files sit at the repository root (`Image.jpeg`, `Image 2.jpeg` …
`Image 8.jpeg`).

🔴 **They were visually inspected. They are photographs of LCD and display
panel assemblies — ribbon cables, LCD modules, driver boards.** 🔴 **They have
no relationship to this project.**

⚠️ **Do not use them. Do not caption them as project output. If you encounter
them, understand that they are irrelevant and exclude them.**

### 1.4 🔴 Every claim must be traceable

For every factual statement, identify which of these it is:

| Class | Meaning |
|---|---|
| `RESEARCHED` | Supported by a source in the repository's registers |
| `SPECIFIED` | Designed on paper. **Not built** |
| `PROPOSED` | Intended for the future. **Not planned, not funded** |
| `UNKNOWN` | Not established. **A valid and valuable answer** |

🔴 **If you cannot classify a claim, do not make it.**

---

## 2. What this project is

| | |
|---|---|
| **CloudLight** | A **school-local digital platform**: classroom displays, timetable and substitution handling, resource browsing and search, teacher authentication, administration, updates |
| **SHANA** | **School Autonomous Notification & Assist** — a **robotics subsystem** on a school-owned Unitree Go2 EDU: navigation, localisation, route safety, battery reserve, hazard handling, safe stopping, physical delivery |

🔴 **CloudLight is the platform. SHANA is one subsystem within it.**

Two rules govern the boundary, both **accepted decisions**:

1. 🔴 **CloudLight may request a delivery. It may never command the robot.**
2. 🔴 **CloudLight must operate fully when SHANA is offline.** A robot failure
   must never blank a classroom screen.

### 🔴 A clarification that matters

> **"SHANA" is the project name, not a second project.** You are writing about
> **one** school-technology project with two named parts. Do not present them as
> two competing products, and do not present the robot as the centre of the
> proposal — **the platform is.**

⚠️ This matters because the robot cannot be shown and the platform's *design*
can be explained. Lead with the platform; present SHANA as the ambitious
subsystem it is.

---

## 3. 🔴 What you must do first

**Inspect the repository before writing.** Do not work from this document alone.

```bash
git log --oneline
find . -type f -name "*.py" -o -name "*.js" -o -name "*.ts" | head
find . -type d -name "*test*"
ls .github/workflows 2>/dev/null || echo "NO CI"
```

### Reading order

| # | Document | Why |
|---|---|---|
| 1 | [`../integration/evidence_index.md`](evidence_index.md) | 🔴 **Start here.** What exists |
| 2 | [`claim_evidence_matrix.md`](claim_evidence_matrix.md) | 🔴 **What you may claim** |
| 3 | [`../integration/repository_state_audit.md`](../integration/repository_state_audit.md) | 🔴 Verified state |
| 4 | [`../cloudlight/overview/current_project_state.md`](../cloudlight/overview/current_project_state.md) | Platform reality |
| 5 | [`../shana/overview/current_project_state.md`](../shana/overview/current_project_state.md) | Robotics reality |
| 6 | [`../integration/system_architecture.md`](../integration/system_architecture.md) | Cross-project view |
| 7 | [`../integration/unresolved_conflicts.md`](../integration/unresolved_conflicts.md) | 🔴 Where the subsystems disagree |
| 8 | [`../shana/research/source_register.md`](../shana/research/source_register.md) | 🔴 The strongest asset |
| 9 | [`../shana/hardware/sdk_and_interfaces.md`](../shana/hardware/sdk_and_interfaces.md) | Vendor constraints |
| 10 | [`demonstration_plan.md`](demonstration_plan.md) | What may be shown |
| 11 | [`../integration/architecture_decisions/ADR-014…`](../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md) | The safety boundary |
| 12 | [`../integration/architecture_decisions/ADR-015…`](../architecture/architecture_decisions/ADR-015-single-integration-repository-not-three.md) | Why there is one repo |

⚠️ **You do not need to read all ~170 documents.** These twelve cover what you
need. `docs/integration/README.md` indexes the rest.

---

## 4. 🔴 What to verify before you write

Check each against the repository. If any fails, **say so in the proposal**.

| # | Claim | Verify by |
|---|---|---|
| 1 | 0 source files | `find` for source extensions |
| 2 | 0 tests | no test directory |
| 3 | 0 CI workflows | no `.github/workflows/` |
| 4 | ≈1,000 questions, 0 answered | Both questionnaires |
| 5 | 20 gates, 0 passed | Both `go_no_go_gates.md` |
| 6 | 104 permissions, 0 granted | Permission checklist |
| 7 | 68 risks, 0 retired | Both risk registers |
| 8 | 3 ADRs cited but non-existent | `ls docs/architecture/…` + shana ADRs |
| 9 | 6 broken internal links | Follow them |
| 10 | The 8 JPEGs are LCD panels | Inspect them |
| 11 | `PAUSED` absent from CloudLight | `grep -rn "PAUSED" docs/cloudlight/` → 0 |
| 12 | `start_opencode_server.sh` is gitignored | `git check-ignore -v` |

⚠️ **If a verification fails, report it.** 🔴 You must not write the proposal
as though verification succeeded when it did not.

---

## 5. Audience

| Audience | They care about | Watch for |
|---|---|---|
| **Teachers** | Will it save time? Is it easy? | 🔴 **No efficiency claim can be evidenced** |
| **Leadership** | Cost, risk, will it work, who approves | 🔴 **25 min of their time is the unblock.** Say so |
| **Technical evaluators** | Is the engineering real? | 🔴 **Lead with the research.** They will respect it |
| **Students** | What can I learn / build? | This is where educational value lands |
| **Parents** | Is it safe? Is it private? | 🔴 Privacy is a **strength**. Lead with it |
| **Exhibition visitors** | A 60-second version | Must not overclaim even here |

⚠️ **Technical evaluators are the audience that matters most**, and they are the
ones most likely to spot an overclaim. 🔴 **Do not optimise the proposal for a
general audience at the cost of a technical evaluator's trust.**

---

## 6. 🔴 Narrative

### The problem — grounded, not inflated

🔴 **Do not invent school inefficiency.** No time study, no workload figure, no
accident statistic exists.

What you **can** say:

- A school runs on information that lives in several places: the timetable, the
  substitution sheet, shared files, and what a teacher needs to know right now.
- Getting a physical item to another part of a building takes a person to walk
  with it.
- ⚠️ **The size of these problems has not been measured.** Say so.

### What the project did

🔴 **The honest and impressive version:**

> *"Rather than build first and ask permission later, this project asked ≈1,000
> structured questions, defined 20 formal go/no-go gates, registered 68 risks,
> and grounded its hardware feasibility in manufacturer documentation and
> source-code analysis — before writing a line of robot code."*

### The turning point

> *"Analysing the robot's own message definitions, the team found that the
> Go2 reports exactly five IMU fields: orientation, angular velocity,
> acceleration, roll-pitch-yaw, and temperature. There is no altitude, no
> pressure, no barometer. **The robot cannot determine which floor of a building
> it is on.** That single finding reshaped the project's scope."*

### 🔴 The scope decision this forced

> *"Multi-floor delivery requires the robot to know where it is — and to be
> able to verify it arrived. It cannot. Unitree also documents a maximum step
> height of 160 mm, and the stair-gait interface appears to have been removed at
> firmware V1.1.6. **The project scoped itself to a single floor**, and recorded
> the conditions under which that decision would be revisited."*

⚠️ **Present this as engineering judgement, not limitation.** 🔴 A team that
found this and *changed scope accordingly* has demonstrated something a team
that built a stair-climbing robot has not.

### The safety boundary

> *"The platform may request a delivery. The robot decides how to move. Two
> agents designed the two subsystems independently, unable to see each other's
> documents, and both arrived at the same rule. **Neither was told the answer.**"*

### The present state

> *"No system has been built. No robot has been operated. No school permission
> has been obtained. Approximately 1,000 questions have been asked and none has
> been answered. Twenty gates are defined and none has been passed."*

### The next step

> *"Roughly 25 minutes of the Principal's time and 40 minutes of IT's time
> would establish whether this project is possible at all. We believe that is a
> proportionate next step."*

---

## 7. Architecture

### 🔴 Label everything `PROPOSED`

```
┌──────────────────────────────────────────────────────────┐
│  CLOUDLIGHT — school platform        STATUS: SPECIFIED    │
│  displays · timetable · substitutions · resources ·      │
│  search · authentication · updates                       │
│                                                          │
│  ✋ NO MOTOR / PATH / SPEED / ACTUATOR PATH EXISTS ✋     │
└───────────────────────┬──────────────────────────────────┘
                        │ request only →   ← status only
                        ▼
┌──────────────────────────────────────────────────────────┐
│  SHANA — robotics subsystem          STATUS: SPECIFIED    │
│  navigation · localisation · route safety · battery ·    │
│  hazards · safe stop · physical delivery                 │
│  🔴 must remain safe when CloudLight is unreachable      │
└──────────────────────────────────────────────────────────┘
```

⚠️ **Label every diagram `PROPOSED ARCHITECTURE — NOT IMPLEMENTED`.**

### Requested diagrams

| # | Diagram | Note |
|---|---|---|
| 1 | System architecture | Must show the one-way boundary |
| 2 | 🔴 **The safety boundary** | 🔴 The most important diagram. Show the *absence* of a motion path |
| 3 | Teacher-to-delivery workflow | 🔴 Label every step `SPECIFIED` |
| 4 | Delivery state machine | 🔴 11 states. Show `AWAITING_RECIPIENT` |
| 5 | 🔴 **The floor-finding constraint** | Why multi-floor is out of scope |
| 6 | Network architecture | 🔴 Mark every network fact `UNKNOWN` |
| 7 | Safety and failure handling | 🔴 Show the degraded states |
| 8 | Roadmap | Phases with **exit criteria**, not dates |

⚠️ **Do not produce a UI mockup that reads as a screenshot.** If you include
one, label it `SCHEMATIC — proposed design, not a working screen`.

---

## 8. User journeys

🔴 **Present these as designed journeys, not working features.**

| # | Journey | Label |
|---|---|---|
| 1 | A teacher checks the next class on a classroom display | `SPECIFIED` — no display exists |
| 2 | A teacher finds an authorised resource | 🔴 **Gated — a shared store may not exist** |
| 3 | Timetable staff review and approve a substitution | `SPECIFIED` — 🔴 **no approver has been named** |
| 4 | A teacher requests an approved delivery | `SPECIFIED` — 🔴 deferred to last |
| 5 | SHANA validates the request independently | `SPECIFIED` — 🔴 **SHANA may reject** |
| 6 | The recipient confirms delivery | ⚠️ **No mechanism designed.** SHANA has no arm; a human stands at the robot |
| 7 | 🔴 A blocked route pauses the task | `SPECIFIED` |
| 8 | 🔴 The robot is offline — the platform still works | `SPECIFIED` — **the most important journey** |

🔴 **Journey 8 deserves emphasis.** It is the design principle most likely to be
appreciated by a technical evaluator: 🔴 *the platform never depends on the
robot.*

⚠️ **Journey 6 is honest but unflattering** — a humanoid-free robot cannot pick
anything up. ⚠️ If the item can be loaded by a person and unloaded by a person,
say exactly that. 🔴 Do not imply autonomous handover.

---

## 9. Safety, privacy, security

### 🔴 Present privacy as a feature, not a disclaimer

> *"This system will not identify people. It will not recognise faces. It will
> not track individuals. It will not send raw video off the robot. It will not
> detect misconduct. These were treated as absolute constraints, not as
> features to be added later if there is time."*

⚠️ **These were independently adopted by both subsystems**, and they cost real
functionality. That is the point.

### Safety position

| Item | Statement |
|---|---|
| Human supervision | 🔴 Required at every stage, with **unilateral stop authority** |
| Emergency stop | Honoured **unconditionally and immediately** — 🔴 cannot be overridden by a peer |
| No AI to the robot | 🔴 A deterministic validator sits in the loop. Never |
| Route safety | 🔴 A blocked or unknown route is **never** traversed. Fails closed |
| Lifts | 🔴 **No modification, ever.** Unlawful, and no sanctioned interface exists |
| Stairs | 🔴 Out of scope for the reasons above |
| Battery | 🔴 **No autonomous return to dock.** Recovery is human-assisted |

### 🔴 Authenticate honestly

Security is **designed, not deployed**. Say:

> *"No system is deployed, so there is nothing currently exposed. The security
> architecture is documented, including a requirement for mutual authentication
> between the platform and the robot, because an unauthenticated endpoint would
> let any host on the network create delivery tasks."*

⚠️ 🔴 **Do not claim any security control is in place.**

---

## 10. 🔴 Limitations — state them

🔴 **A proposal that omits these is hiding its own weakness.** Include them.

| # | Limitation | Why it strengthens the proposal |
|---|---|---|
| 1 | **Nothing has been built** | Shows process discipline |
| 2 | **The robot cannot sense its floor** | The finding drove the scope |
| 3 | **The exact robot model is unknown** | Gates everything |
| 4 | **No sanctioned lift interface exists** | Explains single-floor scope |
| 5 | **No stair gait API at the likely firmware** | Found by source analysis |
| 6 | **≈1,000 questions, 0 answered** | The work is the question set |
| 7 | **0 of 20 gates passed** | Shows rigour |
| 8 | **0 of 104 permissions granted** | Honest |
| 9 | 🔴 **10 cross-project conflicts remain open** | 🔴 **Integration review found real defects in our own design** |
| 10 | 🔴 **6 broken links; 3 citations to non-existent ADRs** | 🔴 **We audited ourselves and found these** |

⚠️ **Items 9 and 10 are unusual and valuable.** Most teams do not audit their
own work. 🔴 **Disclosing it demonstrates exactly the rigour being claimed.**

### 🔴 Recommended verbatim limitations statement

> *This is a feasibility study, not a delivered system. No software has been
> written, no robot operated, and no school infrastructure accessed.
> Approximately 1,000 structured questions have been asked and none has been
> answered; twenty formal go/no-go gates are defined and none has been passed;
> sixty-seven risks are registered and none retired. Every capability described
> is proposed rather than demonstrated. An integration review conducted on this
> codebase identified ten unresolved conflicts between the two subsystems,
> including a state-machine defect that would have caused a paused robot to be
> reported to staff as "on the way". We have recorded these rather than
> concealed them.*

---

## 11. Educational value

⚠️ **Do not fabricate outcome measurements.** 🔴 No educational outcome has been
measured.

🔴 **What can honestly be said** — the disciplines genuinely engaged:

| Discipline | Real content |
|---|---|
| **Robotics** | DDS interfaces, gait control, sensor fusion, the limits of IMU-only state estimation |
| **Embedded systems** | Real-time control, vendor SDK constraints, firmware-version gating |
| **Software engineering** | ⚠️ Not yet — 🔴 **say so, and present it as the next phase** |
| **Networking** | L2 vs L3, client isolation, segment separation, wired robot links |
| **AI** | 🔴 The project's position is **deterministic-first**, with AI assistive only. ⚠️ Present this as a considered position, not an AI system |
| **Computer vision** | ⚠️ **Only as research.** No CV pipeline exists |
| **Testing** | 🔴 0 tests. Present the *plan* |
| **Safety engineering** | 🔴 **This is where the project is strongest** |
| **Integration** | 🔴 **Contract design, conflict analysis, cross-subsystem ownership** |
| **Research method** | 🔴 **Source citation, evidence classification, recording `NOT FOUND`** |
| **Project management** | 🔴 **≈1,000 questions, gates, risk registers** |

⚠️ **The strongest educational story is not "we built a robot".** It is:

> *"Students learn that the hardest part of an ambitious project is often
> proving what is *not* possible — and that saying so early is worth more than
> building something unsafe and apologising later."*

---

## 12. Roadmap

🔴 **No dates. No funding claims. No approvals.**

Each phase needs: **deliverable · dependencies · risks · exit criteria.**

| Phase | Content | 🔴 Exit criteria |
|---|---|---|
| **0. Unblock** | Answer the ~14 gating questions | 🔴 Permission to proceed granted in writing |
| **1. Simulate** | 🔴 **`unitree_mujoco` — no robot, no cost** | 🔴 Software runs on the real DDS interface |
| **2. Platform first** | Deterministic timetable parsing against synthetic fixtures | 🔴 **No school data needed** |
| **3. Bench test** | CloudLight on authorised hardware | 🔴 Gates 1–3 passed |
| **4. Single board** | One classroom display, supervised | 🔴 Anonymised teacher confirmation |
| **5. Robot** | 🔴 Supervised, no students, closed area | 🔴 Gates passed; spotter protocol proven |
| **6. Integration** | 🔴 **Last.** SHANA integration added | 🔴 Contract tests pass |
| **7. Scale** | 🔴 Conditional | 🔴 Evidence at every prior phase |

⚠️ **Phase 1 and Phase 2 require no permission from anyone.** 🔴 They are the
right next steps and you should say so — it shows the team knows how to
sequence.

⚠️ 🔴 **Never state a date, a budget or an approval.**

---

## 13. Visual design

| Requirement | |
|---|---|
| **Typography** | One family, 2–3 weights. 🔴 No more |
| **Colour** | 🔴 **One accent.** Neutral base. 🔴 No red/green as the only signal |
| **Hierarchy** | One focal point per slide |
| **Diagrams** | 🔴 **Consistent, labelled, `PROPOSED` where applicable** |
| **Density** | 🔴 If it does not fit, it does not go in |
| **Jargon** | 🔴 "E-stop" and "DDS" need a word of explanation on first use |
| 🔴 **Imagery** | ⚠️ **No stock robot photos.** 🔴 **No photographs of a robot** — 🔴 **we have none, and stock would imply ours** |
| ✅ **Allowed imagery** | 🔴 **Diagrams and schematics only**, clearly labelled |
| ⚠️ **If** you use an AI-generated illustration | ⚠️ It must be labelled *"Illustration — not our hardware"* 🔴 **Never** presented as a photograph |

⚠️ **A stock photograph of a robot dog is the single most likely way this
proposal misleads.** It will read as *our* robot. 🔴 **Do not include one.**

---

## 14. 🔴 Final quality control

Run **every** check. Report failures — 🔴 **do not silently fix them.**

### Claims

- [ ] Every claim carries `RESEARCHED` / `SPECIFIED` / `PROPOSED` / `UNKNOWN`
- [ ] 🔴 Zero claims marked 🔴 `NOT USABLE` or ⛔ `FORBIDDEN` in
      [`claim_evidence_matrix.md`](claim_evidence_matrix.md)
- [ ] No present-tense construction implying a built system
- [ ] No invented statistic, date, cost, percentage or approval

### Numbers

- [ ] Every number traced to [`claim_evidence_matrix.md`](claim_evidence_matrix.md) §7
- [ ] Every number cross-checked against the repository
- [ ] 🔴 **No number appearing that is not in the matrix**

### Diagrams

- [ ] Every diagram labelled `PROPOSED` where applicable
- [ ] 🔴 No diagram implies a screenshot
- [ ] Architecture matches `system_architecture.md`
- [ ] State machine shows all 11 SHANA states

### Safety and privacy

- [ ] No privacy claim is false
- [ ] No lift, stair, CCTV or wet-floor capability implied
- [ ] Human supervision stated
- [ ] Limitations present and unedited

### Assets

- [ ] 🔴 The 8 root JPEGs excluded
- [ ] 🔴 No generated image presented as a photograph
- [ ] 🔴 No stock robot photograph used

### Consistency

- [ ] Implementation statuses match `current_project_state.md` ×2
- [ ] Cross-references resolve
- [ ] Terminology consistent: CloudLight, SHANA, the robot

### 🔴 The evaluator test

> **Could a sceptical technical evaluator find anything in this proposal that
> the repository does not support?**
>
> 🔴 **If yes, it must be fixed or removed before delivery.**

---

## 15. Execution sequence

1. **Inspect the repository** — `git log`, source-file census, test directory,
   CI check
2. **Read** the 12 documents in §3, in order
3. **Verify** the 12 items in §4
4. **Read** `claim_evidence_matrix.md` and note what is 🔴 forbidden
5. **Identify gaps** — anything you could not verify, say so explicitly
6. **Create the proposal** — following §§2–12
7. **Validate** against §14 — every check
8. **Report** the files created and the unresolved issues

### 🔴 Constraints on your behaviour

- ❌ **Do not alter unrelated source code** to make the proposal appear complete
- ❌ **Do not commit or push** unless separately authorised
- ❌ **Do not create documentation in `docs/shana/**` or `docs/cloudlight/**`** —
  🔴 those are owned by other agents
- ❌ **Do not invent evidence to fill a gap** — 🔴 report the gap
- ❌ **Do not soften the limitations to make the proposal more appealing** —
  🔴 they are the proposal's strongest asset

---

## 16. Deliverable

| Property | Value |
|---|---|
| **Format** | 🔴 **Inspect the repository first.** No format is mandated. **Slides plus a written document** is appropriate |
| **Location** | 🔴 **Ask, or place where agreed.** Agent 3 owns `docs/exhibition/**` |
| **Do not** | Create redundant formats without reason |
| **Handout** | ⚠️ **Include** [`claim_evidence_matrix.md`](claim_evidence_matrix.md) so an evaluator can check your claims |

⚠️ **Including the evidence matrix in the handout is a strong move.** It says:
*"check our work."* 🔴 **Most teams will not.**

---

## 17. 🔴 Final instruction

> **Do not make this project sound more complete than it is.**

This has been the rule throughout, from the feasibility documents to this
handoff. It is not caution — it is strategy.

🔴 A judge comparing this against a team that built and shipped a stair-
climbing delivery robot for children will still see the difference. **A team
that knew it should not build that, and said so, has shown the harder skill.**

⚠️ **The proposal's job is not to prove the system works. It is to prove the
team knows exactly what it does not know** — and knows what to do about it.

🔴 **If you are ever uncertain whether a sentence overclaims, it overclaims.**

---

# ⬆️⬆️⬆️ THE PROMPT ENDS ⬆️⬆️⬆️

---

## Appendix — Reference

| Need | Read |
|---|---|
| What may be claimed | [`claim_evidence_matrix.md`](claim_evidence_matrix.md) |
| What evidence exists | [`evidence_index.md`](evidence_index.md) |
| What to show | [`demonstration_plan.md`](demonstration_plan.md) |
| What may be measured | [`measurement_plan.md`](measurement_plan.md) |
| Exhibition risks | [`risk_register.md`](risk_register.md) |
| Verified repo state | [`../integration/repository_state_audit.md`](../integration/repository_state_audit.md) |
| Cross-project conflicts | [`../integration/unresolved_conflicts.md`](../integration/unresolved_conflicts.md) |
| The safety boundary | [`../architecture/architecture_decisions/ADR-014…`](../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md) |
| Platform reality | [`../cloudlight/overview/current_project_state.md`](../cloudlight/overview/current_project_state.md) |
| Robotics reality | [`../shana/overview/current_project_state.md`](../shana/overview/current_project_state.md) |
| Governance | [`../../AGENTS.md`](../../AGENTS.md) |

### 🔴 Document status

| | |
|---|---|
| This handoff | `DESIGN_ONLY` |
| 🔴 Proposals created | **0** |
| 🔴 Claims verified by an author | **0** |
| 🔴 Open conflicts | **10** |
| 🔴 Outstanding requests | **3**, none acknowledged |
| 🔴 **Source files** | **0** |