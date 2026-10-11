# Exhibition Risk Register

**Owner:** Agent 3 (Integration Lead)
**Date:** 2026-10-11

**Scope:** risks to **the exhibition itself** — the proposal, the demo, the
judgement made. Not the engineering risk register (owned by each subsystem).

⚠️ Subsystem risk registers: `docs/shana/safety/risk_register.md` (21 risks) ·
`docs/cloudlight/risk_register.md` (46 risks). 🔴 Neither is duplicated here.

**Rating:** `L` × `S`, each 1–5. 🔴 High 15–25 · 🟠 Medium 8–14 · 🟡 Low 1–7

---

## 🔴 The dominant risk

> **R-00: a proposal that overclaims, and is then caught.**

🔴 **This is the highest-likelihood, highest-impact risk in the entire project**
— higher than any robotics risk, because no robot will be operated at the
exhibition.

An evaluator who reads ≈1,000 questions and 20 gates may still be told "the robot
autonomously delivers supplies". If they ask to see it, **there is nothing to
show**, and the prior good impression converts into a lasting credibility
failure.

| | |
|---|---|
| Likelihood | **5** — overclaim is the default failure mode of a feasibility project shown to judges |
| Impact | **5** — total loss of credibility |
| Rating | 🔴 **25** |
| Owner | Proposal author |
| Mitigation | [`claim_evidence_matrix.md`](claim_evidence_matrix.md) governs every claim; [`evidence_index.md`](evidence_index.md) states what exists |
| Verification | Every claim traced to a row in the matrix before submission |
| Status | 🟡 **MITIGATION AVAILABLE — NOT YET APPLIED** |

---

## A. 🔴 Credibility risks

| ID | Risk | L | S | Rating | Owner | Mitigation | Status |
|---|---|---|---|---|---|---|---|
| R-01 | A capability is described in the present tense and read as existing | 5 | 5 | 🔴 25 | Proposal author | Use `AGENTS.md` §Q labels; state `DESIGN_ONLY` | 🟡 Available |
| R-02 | 🔴 The 8 root JPEGs are mistaken for project output | 3 | 5 | 🔴 15 | Proposal author | 🔴 **They are LCD panels.** Excluded from Git. Must be named explicitly | 🟡 Available |
| R-03 | A citation points at a file that does not exist (ADR-001/010/011) | 4 | 3 | 🟠 12 | Agent 1 | Write the owed ADRs; interim: cite the decision log | 🔴 Open |
| R-04 | A broken link is followed by an evaluator and 404s | 4 | 2 | 🟠 8 | Agent 2 | Apply the verified 6-link mapping | 🔴 Open |
| R-05 | Statistics are invented to sound credible | 3 | 5 | 🔴 15 | Proposal author | 🔴 **Only numbers in §7 of the matrix** | 🟡 Available |
| R-06 | A partnership or permission is implied that does not exist | 3 | 5 | 🔴 15 | Proposal author | 🔴 0 of 104 permissions granted — say so | 🟡 Available |
| R-07 | The proposal implies the school has endorsed the project | 3 | 4 | 🔴 12 | Proposal author | State the approval position explicitly | 🟡 Available |

---

## B. 🔴 Safety and privacy risks

| ID | Risk | L | S | Rating | Owner | Mitigation | Status |
|---|---|---|---|---|---|---|---|
| R-10 | 🔴 The proposal implies the robot will operate near students | 3 | 5 | 🔴 15 | Proposal author | 🔴 **0 gates passed.** Technical feasibility ≠ authorisation | 🟡 Available |
| R-11 | 🔴 Any suggestion of facial recognition or student tracking | 1 | 5 | 🟠 5 → **5** | Proposal author | 🔴 **Prohibited outright.** State the privacy position as a feature | 🟢 Position strong |
| R-12 | Lift integration is implied as a solved problem | 3 | 4 | 🟠 12 | Proposal author | 🔴 Unlawful to modify; no sanctioned interface | 🟡 Available |
| R-13 | Stair capability is implied | 3 | 4 | 🟠 12 | Proposal author | 🔴 160 mm limit; risers unmeasured; gait API removed | 🟡 Available |
| R-14 | CCTV occupancy estimation is presented as available | 2 | 4 | 🟠 8 | Proposal author | 🔴 Not implemented **and recommended against** | 🟡 Available |
| R-15 | 🔴 A live demo attempts physical robot operation | 2 | 5 | 🔴 10 | Demonstration lead | 🔴 **No demo may involve hardware.** See `demonstration_plan.md` | 🟢 Plan forbids |

---

## C. 🟠 Scope and expectation risks

| ID | Risk | L | S | Rating | Owner | Mitigation | Status |
|---|---|---|---|---|---|---|---|
| R-20 | The proposal promises a delivery date | 4 | 3 | 🟠 12 | Proposal author | 🔴 **20 gates, 0 passed.** Phases with **exit criteria**, not dates | 🟡 Available |
| R-21 | The audience expects a working robot and is disappointed | 4 | 3 | 🟠 12 | Proposal author | Frame as a **feasibility study**, up front | 🟡 Available |
| R-22 | The scope looks too small to justify the effort | 3 | 3 | 🟠 9 | Proposal author | 🔴 **≈1,000 questions is the value.** Show the rigour | 🟡 Available |
| R-23 | 🔴 A future phase is presented as funded or approved | 2 | 4 | 🟠 8 | Proposal author | Label every future phase `PROPOSED` | 🟡 Available |
| R-24 | The two subsystems are presented as one finished product | 3 | 4 | 🟠 12 | Proposal author | 🔴 They are 2 documented designs + 0 `AGREED` contracts | 🟡 Available |

---

## D. 🟠 Integration-review risks — created by this phase

| ID | Risk | L | S | Rating | Owner | Mitigation | Status |
|---|---|---|---|---|---|---|---|
| R-30 | 🔴 A paused robot is shown to a teacher as "On the way" | **4** | 4 | 🔴 16 | Agent 2 | 🔴 `CONF-02` / `ADR-016`. **Fix before any integration is built** | 🔴 **OPEN** |
| R-31 | 🔴 A duplicate delivery request creates two physical tasks | 3 | 5 | 🔴 15 | Agent 1 + 2 | 🔴 `CONF-06` idempotency is a **P0 safety control** | 🔴 **OPEN** |
| R-32 | An unauthenticated host could create robot tasks | 2 | 5 | 🟠 10 | Agent 1 | 🔴 `IC-06`. ⚠️ Bounded by the absent motion path | 🔴 **OPEN** |
| R-33 | Two contracts are implemented independently and diverge | 4 | 3 | 🟠 12 | Both | The reconciliation request has been issued | 🔴 **PENDING** |
| R-34 | CloudLight's availability becomes coupled to the robot | 3 | 4 | 🟠 12 | Agent 2 | 🔴 The dependency must stay optional | 🟡 Design stated |

⚠️ **R-30 is the one to watch.** It is the only defect found in this review that
produces a *false statement about a robot's physical state*.

---

## E. 🟡 Process risks

| ID | Risk | L | S | Rating | Owner | Mitigation | Status |
|---|---|---|---|---|---|---|---|
| R-40 | 🔴 The exhibition shows a chart or diagram implying data that does not exist | 3 | 4 | 🟠 12 | Proposal author | Label every figure `SPECIFIED` or `MEASURED` | 🟡 Available |
| R-41 | Generated imagery is presented as a photograph | 3 | 4 | 🟠 12 | Proposal author | 🔴 Label clearly: *"Schematic — design, not photograph"* | 🟡 Available |
| R-42 | The proposal is written from a summary rather than the repository | 3 | 4 | 🟠 12 | Proposal author | 🔴 `ANTIGRAVITY_EXHIBITION_HANDOFF.md` §15 requires repository inspection | 🟡 Available |
| R-43 | The agent requests are never delivered and coordination stalls | **4** | 3 | 🟠 12 | Repository owner | 🔴 **No delivery channel exists.** Needs a human | 🔴 **OPEN** |
| R-44 | Work continues without owner visibility | 3 | 3 | 🟠 9 | All | Task board + handoff protocol | 🟢 Active |

---

## F. Register summary

| Metric | Value |
|---|---|
| Risks registered here | **24** |
| Subsystem risks (not duplicated) | **67** |
| 🔴 High risks | **9** |
| 🔴 **Highest-rated risk** | **R-00 — overclaiming (25)** |
| Mitigations available | **19** |
| 🔴 **Mitigations applied** | **0** — no exhibition exists yet |
| 🔴 **Open integration risks** | **4** (`R-30`…`R-33`) |
| 🔴 Mitigations requiring another agent | **4** |

---

## G. 🔴 The one conclusion

> **Every risk in this register is either already mitigated by available
> discipline, or blocked on a person.**

🔴 **Nothing here is a technical unknown.** The dominant risks are
**communication** — overclaiming, and coordination that has not happened.

⚠️ **The cheapest action that reduces this register most is a sentence in the
proposal stating what has not been done.** It costs one line and protects
everything else.