# Demonstration Plan

**Owner:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Basis:** [`../integration/evidence_index.md`](evidence_index.md) ·
[`claim_evidence_matrix.md`](claim_evidence_matrix.md)

---

## 🔴 The governing constraint

> **There is nothing to demonstrate.**

0 source files · 0 tests · 0 builds · 0 deployments · 0 capabilities · 0
photographs of the robot · 0 screenshots · 0 measurements · 0 permissions
granted · 0 of 20 gates passed.

🔴 **Therefore this is not a "which demo?" question. It is a "what may be
shown, and how do we show that honestly?" question.**

⚠️ The three plans below are **present-tense options**, not a menu to choose
between after building something. 🔴 **PLAN C is the only one executable
today**, and choosing it is not a failure — it is the accurate answer.

---

## 🔴 Absolute prohibitions

Applies to **all** plans.

| ❌ Prohibited | Why |
|---|---|
| 🔴 Operating a robot, live or recorded | 0 permissions. `AGENTS.md` §L.12 |
| 🔴 Any physical demonstration near people | No gate passed |
| 🔴 Any generated image presented as a photograph or screenshot | 🔴 Never |
| 🔴 A mockup of a robot presented as the robot | Must be labelled `SCHEMATIC — proposed design` |
| 🔴 Connecting to school infrastructure | 0 permissions |
| 🔴 Contacting CCTV, a lift, or a network | `AGENTS.md` §K |
| 🔴 Using the 8 root JPEGs | 🔴 They are **LCD panels** |
| 🔴 Any "live demo" of CloudLight | 🔴 No code exists |

⚠️ **There is no live software demo available in any plan**, because there is no
software.

---

## PLAN C — 🟡 The only executable plan today

> **A software-design prototype presentation, with the research and the safety
> architecture as the substance.**

### What is shown

| # | Element | Evidence backing it |
|---|---|---|
| 1 | 🔴 **The problem** — school-local information and delivery | `requirements.md` (186 reqs) |
| 2 | 🔴 **The architecture** — CloudLight as platform, SHANA as subsystem | Both subsystem designs |
| 3 | 🔴 **The safety boundary** — CloudLight requests; SHANA decides | 🔴 **Independently derived by both agents.** The single strongest intellectual result |
| 4 | 🔴 **The research** — vendor docs + source-code analysis | 106 sourced entries |
| 5 | 🔴 **The critical finding** — the robot cannot sense its floor | `IMUState.msg`: 5 fields, no altitude |
| 6 | 🔴 **The rigour** — 20 gates, 68 risks, ≈1,000 questions | All in-repo |
| 7 | 🔴 **The blockers, honestly** — with costs | ~25 min Principal + ~40 min IT |
| 8 | 🔴 **The roadmap** — phased, with exit criteria | `demonstration_plan.md` §6 |

### Why this is strong, not weak

🔴 **An evaluator who knows what a feasibility study should contain will find
this project passes every one of those checks**, and finds them unusually well
documented:

- Sourced research with **recorded contradictions** rather than resolution by
  assertion
- **Access limitations disclosed** (WAF-blocked vendor site; proxied retrieval)
- **Static source-code analysis** with the decisive negative result stated
- A **proposal rejected** where the evidence did not support it
- Privacy treated as an **absolute constraint**, not a feature to trade off

⚠️ **That is engineering judgement, which is harder to fake than working
software and considerably rarer.** A class that built and shipped an unsafe
robot would be more alarming than one that refused.

### 🔴 Honest framing — required

> *"We completed a feasibility study. We have not built the system. Here is what
> we established, what we could not, and what must be answered first."*

---

## PLAN B — 🟠 Conditional, and gated

> **PLAN C + a controlled demonstration of verified robot capabilities.**

### 🔴 Gate condition

| # | Prerequisite | Owner | Status |
|---|---|---|---|
| 1 | Written secondary-development authorisation | School administration | 🔴 **Not obtained** |
| 2 | Go2 model, firmware, SDK licence confirmed | Robot custodian | 🔴 **Unknown** |
| 3 | Gates 1–2 passed | School + custodian | 🔴 0 of 20 |
| 4 | A safe, supervised, non-student test area | School | 🔴 Not obtained |
| 5 | 🔴 A human spotter with unilateral stop authority | Named person | 🔴 Not named |
| 6 | Battery reserve validated by measurement | Agent 1 | 🔴 No measurements |

🔴 **6 prerequisites, 0 met.** PLAN B is 🔴 **not available** and must not be
promised in any proposal.

### ⚠️ If it ever became available

The demonstration would be **a robot walking a short supervised route in an
empty area**, with 🔴 **no delivery**, 🔴 **no autonomous navigation claim**,
and 🔴 **no interaction with the platform**. ⚠️ Even then it would demonstrate
*vendor capability*, not *project capability*.

---

## PLAN A — 🔴 Not available, and not foreseeable

> **A functional demonstration of implemented, tested features.**

| Required | Status |
|---|---|
| Working software | 🔴 0 source files |
| Passing tests | 🔴 0 run |
| A deployed system | 🔴 0 hosts |
| A robot operated | 🔴 never |
| Passed gates | 🔴 0 of 20 |
| Granted permissions | 🔴 0 of 104 |

🔴 **PLAN A does not exist. Do not describe it as a future option without
saying it requires 20 gates to pass first.**

---

## 🔴 Decision

> **Execute PLAN C.**

| Reason | |
|---|---|
| It is the only plan with available evidence | |
| It requires **no permission**, **no hardware**, **no school contact** | |
| It cannot be falsified by a question the team cannot answer | |
| 🔴 **Its content is genuinely strong** — see §3 of this document | |
| 🔴 Presenting PLAN A or B would create an expectation the project cannot meet | |

⚠️ **This is not a downgrade. It is the only honest option available**, and the
underlying work supports it well.

---

## 🔴 What makes PLAN C credible — six specific things to show

### 1. 🔴 The floor-finding problem

> *"We analysed the robot's own message definitions. `IMUState` has exactly five
> fields: quaternion, gyroscope, accelerometer, rpy, temperature. **There is no
> altitude, no pressure, no barometer.** The robot cannot determine which floor
> of a building it is on."*

🔴 **This is the most impressive single item in the project.** It was found by
reading source, not by reading marketing, and it **constrains the entire design**
— it means cross-floor delivery cannot be safely verified by the robot.

### 2. 🔴 The boundary found independently

> *"Two agents, working separately and unable to see each other's documents,
> both arrived at the same rule: the platform may request a delivery, and the
> robot decides how to move. Neither was told the answer."*

⚠️ **Independent convergence is stronger evidence than agreement reached after
negotiation.**

### 3. 🔴 A decision based on negative evidence

> *"Unitree documents 33 motion functions. The stair gait setter is not among
> them for firmware V1.1.6 and above. We therefore excluded stairs from scope —
> not because stairs are hard, but because the interface we would need does not
> appear to exist on the version we might own."*

### 4. 🔴 Contradictions shown, not hidden

> *"Our research records where sources disagree — battery runtime, camera frame
> rate, whether the stair gait exists in a given firmware — rather than picking
> one silently."*

### 5. 🔴 Privacy as a hard constraint

> *"We decided the system will not identify people, will not track individuals,
and will not send raw video off the robot. That removes a great deal of
functionality. It was not a trade-off we made lightly, and we would make it
again."*

### 6. 🔴 The honest blocker

> *"We asked approximately ≈1,000 questions. None has been answered yet. Roughly
25 minutes of the Principal's time and 40 minutes of IT's time would tell us
whether this project is possible at all. We think that is a reasonable next
step."*

---

## 🔴 What must NOT be in the demonstration

| ❌ | Why |
|---|---|
| A robot, even a photo of one | 🔴 None exists in the repository. Do not source one and imply it is ours |
| Any screenshot | 🔴 None exists. 🔴 **Never generate one** |
| A performance figure | 🔴 No measurement exists |
| A delivery success rate | 🔴 0 deliveries |
| A timeline with dates | 🔴 20 gates, 0 passed |
| A "partnership with the school" | 🔴 0 permissions granted |
| A mockup presented as a working screen | Must be labelled `SCHEMATIC — proposed design` |
| 🔴 The 8 root JPEGs | 🔴 **They are LCD panels** |

---

## 🔴 Recommended format

| Element | Form |
|---|---|
| Presentation | Slides, ~15–20 |
| Architecture | A **clearly labelled schematic** — 🔴 not a screenshot |
| State machine | A **diagram of a proposal** |
| Research | Quoted vendor text + the message-definition finding |
| Roadmap | Phases with 🔴 **exit criteria, not dates** |
| Handout | The evidence index + claim matrix, so claims are checkable |

⚠️ **Hand the audience the limitations.** It converts the project's biggest
weakness into evidence of judgement.

---

## 🔴 Checklist before presenting

- [ ] Every claim traced to a row in [`claim_evidence_matrix.md`](claim_evidence_matrix.md)
- [ ] 🔴 No claim marked 🔴 `NOT USABLE` or ⛔ `FORBIDDEN` appears
- [ ] Every figure labelled `SPECIFIED`, `MEASURED`, or `RESEARCHED`
- [ ] No generated image presented as a photograph
- [ ] 🔴 The 8 root JPEGs confirmed absent
- [ ] Limitations slide present and unedited
- [ ] No date, percentage, or approval implied without evidence
- [ ] 🔴 **Someone has read it against the matrix, not just written it**
- [ ] 🔴 "What have you built?" has an answer ready: *"Nothing yet — here is
      what we established instead"*