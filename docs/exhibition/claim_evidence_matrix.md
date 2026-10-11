# Claim–Evidence Matrix

**Owner:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Rule:** 🔴 **No claim may appear in exhibition material unless it appears here
with an acceptable status.**

---

## 🔴 How to read the Status column

| Status | Meaning | Exhibition use |
|---|---|---|
| ✅ **USABLE** | Directly supported by evidence in this repository | May be stated plainly |
| 🟡 **QUALIFIED** | True, but only with its stated limitation attached | May be stated **with** the qualification |
| 🔴 **NOT USABLE** | No evidence, or the evidence contradicts it | ⛔ **Must not be claimed** |
| ⛔ **FORBIDDEN** | Safety/privacy/legal violation regardless of evidence | ⛔ **Never** |

---

## 🔴 Tier 1 — Forbidden regardless of evidence

| # | Claim | Why forbidden |
|---|---|---|
| F-01 | ⛔ "The robot has been operated" | 🔴 Never. No hardware access |
| F-02 | ⛔ "The system recognises students" | 🔴 Privacy invariant — prohibited in any form |
| F-03 | ⛔ "The robot uses lifts / climbs stairs" | 🔴 ADR-002/003; no sanctioned interface |
| F-04 | ⛔ "The lift was integrated" | 🔴 Unlawful; no permission |
| F-05 | ⛔ "CCTV provides corridor occupancy" | 🔴 Not implemented, and **recommended against** |
| F-06 | ⛔ "Wet floor detection" | 🔴 Not implemented. 🔴 **No evidence it is possible** |
| F-07 | ⛔ "The robot returns to its dock autonomously" | 🔴 **No documented service exists** (see E-13) |
| F-08 | ⛔ "X% delivery success rate" | 🔴 Zero deliveries performed |
| F-09 | ⛔ "Saves teachers N hours per week" | 🔴 No measurement, no baseline exists |
| F-10 | ⛔ "Tested", "verified", "working", "operational" | 🔴 0 tests, 0 runs, 0 demos |
| F-11 | ⛔ Any image from the repository root as project output | 🔴 **They are LCD panels** |
| F-12 | ⛔ "Teachers use this daily" | 🔴 0 deployments |

---

## 🔴 Tier 2 — Not usable

| # | Claim | Reality |
|---|---|---|
| N-01 | "CloudLight is deployed" | 🔴 0 hosts, 0 permission |
| N-02 | "SHANA navigates autonomously" | 🔴 No code, no simulator |
| N-03 | "The subsystems are integrated" | 🔴 0 `AGREED` contracts |
| N-04 | "The delivery workflow is implemented" | 🔴 11 states on paper only |
| N-05 | "The school has approved this" | 🔴 0 of 104 permissions |
| N-06 | "Gates have been passed" | 🔴 0 of 20 |
| N-07 | "The architecture is validated" | 🔴 Design review only |
| N-08 | "Risks have been mitigated" | 🔴 0 of 67 retired |
| N-09 | "Assumptions have been verified" | 🔴 1 of 58 |
| N-10 | "Requirements have been agreed" | 🔴 0 of 186 |

---

## 🟡 Tier 3 — Usable only with qualification

| # | Claim | Required qualification | Evidence |
|---|---|---|---|
| Q-01 | *"Documented by Unitree: only EDU-and-above models open a development interface"* | 🔴 **Vendor documentation. Not tested on our hardware.** | Verbatim Unitree FAQ |
| Q-02 | *"Unitree documents a maximum step height of 160 mm"* | 🔴 **Vendor figure. Actual risers unmeasured.** | Developer docs |
| Q-03 | *"Unitree's docs state the robot supports **wired** development connections only"* | 🔴 **Vendor documentation.** | Verbatim Unitree FAQ |
| Q-04 | *"Source-code analysis shows no altitude, pressure or barometer message"* | 🔴 **Static analysis. 🔴 Consequence — the robot cannot know its floor — is not tested.** | `IMUState.msg` |
| Q-05 | *"The stair gait API was removed at firmware V1.1.6"* | 🔴 **The installed firmware is `UNKNOWN`.** ⚠️ Finding 6 vs 7 is an unresolved conflict | `sport_client.hpp` |
| Q-06 | *"No charging or dock service appears in the documented service list"* | 🔴 **Negative finding from documentation. 🔴 An unread ArUco recharge page may contradict it.** | `RobotStateClient` |
| Q-07 | *"CloudLight must operate fully when SHANA is offline"* | 🔴 **A design principle. Nothing enforces it — there is no code.** | `ADR-014`, §6 |
| Q-08 | *"CloudLight has no capability to command robot motion"* | 🔴 **True by absence — there is no code at all.** ⚠️ Must be stated as *designed*, not *implemented* | `ADR-014` |
| Q-09 | *"An access point cannot host an application"* | 🔴 **General networking fact, not about this school.** | Microsoft / L2 networking |
| Q-10 | *"Assigned Access and Shell Launcher are edition-dependent and cannot be combined"* | 🔴 **General Windows fact.** 🔴 **The boards' editions are `UNKNOWN`.** | Microsoft docs |
| Q-11 | *"A Windows Service cannot display UI"* | 🔴 General Windows fact | Microsoft docs |
| Q-12 | *"TUF is a published specification naming rollback, freeze, fast-forward and mix-and-match"* | 🔴 **No TUF implementation exists here.** | TUF specification |
| Q-13 | *"SMS/PSTN is a restricted authenticator under NIST SP 800-63B"* | 🔴 General standards fact | NIST SP 800-63B |
| Q-14 | *"OCR dominates document-conversion cost by 1–2 orders of magnitude"* | 🔴 From vendor-published per-stage figures. 🔴 **Not reproduced on this school's documents** | Docling published figures |

---

## ✅ Tier 4 — Usable

| # | Claim | Evidence | Confidence |
|---|---|---|---|
| U-01 | *"A structured feasibility study has been completed: 986 structured questions, 20 go/no-go gates, 68 registered risks and 106 sourced research entries."* | Measured this audit — see `evidence_index.md` §3a | ✅ High |
| U-02 | *"Zero implementation code, zero tests and zero deployments exist."* | Filesystem inspection | ✅ High |
| U-03 | *"The research is grounded in manufacturer documentation and source-code analysis, with access limitations and contradictions recorded rather than hidden."* | Both source registers | ✅ High |
| U-04 | *"Two subsystems were designed independently and converged on the same safety boundary — CloudLight requests, SHANA decides."* | Both integration documents; `ADR-014` | ✅ High |
| U-05 | *"The robot cannot determine which floor it is on: there is no altitude, pressure or barometer measurement anywhere in its message set."* | `IMUState.msg` static analysis | ✅ High on the finding |
| U-06 | *"Every school-specific fact is an open question with a named owner — none has been assumed."* | Questionnaires; 0 answered | ✅ High |
| U-07 | *"The remaining blockers are institutional, not technical. Approximately 25 minutes of a Principal's time and 40 minutes of IT's time would establish whether the project is feasible at all."* | `next_actions.md`; cross-checked | ✅ High |
| U-08 | *"No student data is collected, no person is identified, and no raw video leaves the robot."* | Design position; **not implemented** | 🟡 Position, not behaviour |
| U-09 | *"No school infrastructure has been probed, scanned or contacted."* | Process fact | ✅ High |
| U-10 | *"10 cross-project conflicts were found and recorded during integration review."* | `unresolved_conflicts.md` | ✅ High |

---

## 🔴 Tier 5 — Findings that must accompany the project wherever it is shown

⚠️ **These are not defects. They are the most credible part of the study.** A
proposal that omits them is hiding its own value.

| # | Finding | Why it must be stated |
|---|---|---|
| D-01 | 🔴 The robot **cannot sense which floor it is on** | The single most important technical limitation |
| D-02 | 🔴 Secondary development requires an **EDU** model; 🔴 **the actual model is `UNKNOWN`** | Gates everything |
| D-03 | 🔴 No sanctioned lift interface exists; modification would be unlawful | Explains single-floor scope |
| D-04 | 🔴 **No documented autonomous return-to-dock** | Affects operations and recovery |
| D-05 | 🔴 **≈1,000 questions asked, 0 answered** | Explains why nothing is built |
| D-06 | 🔴 **0 of 20 gates passed; 0 of 104 permissions granted** | The rigour *and* the limit |
| D-07 | 🔴 **0 of 68 risks retired** | Honest risk position |
| D-08 | ⚠️ Stair gait API was **removed** at firmware V1.1.6, and the installed firmware is unknown | A live technical conflict |
| D-09 | ⚠️ An ArUco recharge documentation page **exists but was never read** and may contradict D-04 | Outstanding work, honestly recorded |
| D-10 | 🔴 **6 broken internal links** and **3 citations to non-existent ADRs** | Self-audited; shows the audit is real |

---

## 6. 🔴 Prohibited exhibition phrasing

| ❌ Never write | ✅ Write instead |
|---|---|
| "The robot delivers…" | *"SHANA is specified to deliver…"* |
| "Our platform…" | *"A platform has been designed…"* |
| "The system works…" | *"The design specifies…"* |
| "Tested successfully" | *"No tests have been run — none exist to run"* |
| "Proven in a pilot" | *"No pilot has been conducted"* |
| "Students can find…" | *"Resource access is specified, gated on storage that may not exist"* |
| "Improves staff efficiency by…" | *"No measurement exists. A measurement plan is documented"* |
| "Approved by the school" | *"No permission has been granted"* |
| "Autonomous navigation" | *"Navigation is specified; no navigation code exists"* |
| "Safe" (unqualified) | *"Safety has been argued rigorously. No safety test has been run"* |

---

## 7. 🔴 Numbers that may be used

| Number | Source | Safe to state? |
|---|---|---|
| 434 / 576 / ≈1,000 questions | Questionnaires | ✅ |
| 0 answered | Questionnaires | ✅ |
| 20 gates, 0 passed | `go_no_go_gates.md` ×2 | ✅ |
| 104 permissions sought, 0 granted | Permission checklist — all `NOT REQUESTED` | ✅ |
| 68 risks, 0 retired | Risk registers ×2 (22 SHANA + 46 CloudLight) | ✅ |
| 186 requirements, 0 agreed | `requirements.md` | ✅ |
| 106 research entries | Source registers ×2 (68 + 38) | ✅ |
| 160 mm max step | Vendor-documented | 🟡 **With qualification** |
| 8,000 / 15,000 mAh battery options | Vendor-documented | 🟡 **Which is fitted is `UNKNOWN`** |
| 25 min Principal + 40 min IT | `next_actions.md` | ✅ **Approximate** |
| 6 broken links · 11 conflicts | This audit | ✅ |
| 0 code · 0 tests · 0 deployments | Filesystem inspection | ✅ |

⚠️ **Every other number must be sourced or omitted.**

---

## 8. 🔴 Statement of limitations — recommended verbatim inclusion

> *This work is a feasibility study. No system has been built, no robot has
> been operated, and no school infrastructure has been accessed. Approximately
> ≈1,000 structured questions have been asked and none has yet been answered.
> Twenty formal go/no-go gates are defined and none has been passed. Every
> capability described is proposed, not demonstrated. The research is grounded
> in manufacturer documentation and source-code analysis, with access
> limitations and unresolved contradictions recorded rather than concealed.*

⚠️ **Including this costs nothing and buys everything.** An evaluator who reads
it sees judgement. An evaluator who catches an overclaim sees a credibility
problem that never goes away.