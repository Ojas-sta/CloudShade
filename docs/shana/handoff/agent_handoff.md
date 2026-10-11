# Agent Handoff

**Purpose:** the single entry point for whoever picks SHANA up next.

You are inheriting a **feasibility study**, not a codebase. That single sentence
determines almost everything about how you should proceed.

---

## 1. Orient yourself — 30 minutes

| # | Read | Why |
|---|---|---|
| 1 | [`../overview/current_project_state.md`](../overview/current_project_state.md) | 🔴 The reality check. Start here, not anywhere else |
| 2 | This document | What you are picking up and what you must not do |
| 3 | [`implementation_inventory.md`](implementation_inventory.md) | Component-by-component truth |
| 4 | [`unresolved_dependencies.md`](unresolved_dependencies.md) | 🔴 What you are actually blocked on |
| 5 | [`../feasibility/go_no_go_gates.md`](../feasibility/go_no_go_gates.md) | The rules that say whether you may build |
| 6 | [`../research/claim_evidence_matrix.md`](../research/claim_evidence_matrix.md) | Which statements are defensible |
| 7 | [`../README.md`](../README.md) | Full index |

Total: roughly 100 markdown documents. **You do not need to read all of them.**
Read the seven above, then use the index to pull what your task needs.

---

## 2. What SHANA is

An indoor school delivery robot subsystem. A quadruped Go2 carries items on a
tray, navigates single-floor corridors, avoids people, and stops safely. It is
part of a wider **CloudLight** system.

## 3. What SHANA is not

| It is not | Consequence |
|---|---|
| An arm or manipulator | It cannot pick up, place, or open anything |
| A multi-floor system | Stairs excluded, no lift integration possible |
| Autonomous at the battery level | No return-to-dock; human recovery required |
| Capable of autonomous lifts | No sanctioned lift API exists and modification is unlawful |
| A finished system | 0 tests, 0 gates passed, 0 code |

---

## 4. 🔴 Hard rules — non-negotiable

These are not style preferences. Violating them makes the work worthless or
unsafe.

### 4.1 Never fabricate evidence

| ❌ Never | ✅ Do instead |
|---|---|
| Invent test results | Record "no tests run" and why |
| Create or edit photographs to look like results | State that no evidence exists |
| Generate mock screenshots and present them as the system | Use schematic diagrams, clearly labelled as design |
| Claim tests "passed" because code looks correct | Report `not tested` |
| Invent a robot serial, firmware, or school name | Mark `UNKNOWN` |

⚠️ The repository contains eight photographs. **All eight are unrelated
LCD/display panel assemblies.** They are not SHANA evidence and must never be
presented as such. See
[`exhibition_evidence_inventory.md`](exhibition_evidence_inventory.md).

### 4.2 Never operate hardware

No physical operation. No robot commands. No network probing. No CCTV or lift
system contact. **Zero permissions have been granted.**

### 4.3 Never touch CloudLight's files

`docs/cloudlight/` belongs to another agent. Read it if it exists; **never
modify it.** Coordinate changes through the interface contract in
[`../architecture/cloudlight_integration.md`](../architecture/cloudlight_integration.md).

### 4.4 Never commit, push, or add a remote

The repository has **0 commits and 0 remotes**. Do not change that without
explicit instruction from the owner.

### 4.5 Never commit secrets

🔴 `start_opencode_server.sh` contains a **hardcoded plaintext password** and
binds a server to `0.0.0.0`. Do not commit it. Do not copy its contents into any
document. Do not quote the credential. It must be rotated by its owner.

---

## 5. 🔴 What you are actually blocked on

**Not** the technology. **Not** your effort. These external facts:

| # | Blocker | Who answers |
|---|---|---|
| 1 | Go2 model number, firmware version, SDK licence eligibility | School / robot owner |
| 2 | Written answers to the school-critical questionnaire questions | School administration |
| 3 | Route survey access — corridors, lifts, stairs, destinations | School administration |
| 4 | Lift manufacturer response on sanctioned API | Johnson (via school) |
| 5 | Privacy assessment for any camera or CCTV use | School / legal |
| 6 | Network and robot access approvals | School IT |

⚠️ **Writing more software before these clear would be the single biggest
wasted-effort risk in this project.** The architecture is deliberately
conditional on all of them.

---

## 6. What you *can* do right now, without permission

| # | Task | Output |
|---|---|---|
| 1 | **Chase the blockers** | Fill answers in `questionnaires/SHANA_Feasibility_QA.md` |
| 2 | **Write the 10 owed ADRs** | Real decisions, already in `decision_log.md` |
| 3 | **Draft the route survey protocol** | Ready to execute the moment access is granted |
| 4 | **Prepare the technical assessment request** | Ready to send to IT, CCTV, and lift contacts |
| 5 | **Build a simulation-only proof of concept** | ⚠️ Permitted by ADR-011, needs no hardware, no data. **The single highest-value unblocked task** |
| 6 | **Improve diagrams and clarity** | Diagrams are allowed — photographs are not |

⚠️ **Task 5 is the answer to "what should I do next."** It is the only way to
produce a working artefact under the current rules, because a simulation needs
no robot, no network, no school, and no student data.

---

## 7. The five facts that shape everything

1. 🔴 **Only EDU-and-above Go2 models expose a development interface.** Verbatim:
   *"Only EDU or above versions can be developed, other versions such as PRO and
   AIR versions do not open the secondary development interface."*
2. 🔴 **No altitude, pressure, or barometer sensor.** The robot **cannot
   determine which floor it is on.**
3. 🔴 **The stair gait setter was removed** from the current SDK (≥1.1.6).
4. 🔴 **No sanctioned lift API.** Using the installed Johnson lifts would require
   modification, which is unlawful.
5. 🔴 **No autonomous return-to-dock.** Recovery is human-assisted.

---

## 8. How to keep this documentation honest

| Habit | Why |
|---|---|
| Add a dated entry to [`../research/research_log.md`](../research/research_log.md) on every significant change | Continuity across agents |
| Update [`implementation_inventory.md`](implementation_inventory.md) when any status changes | It is the truth source |
| Mark unknowns `UNKNOWN` — never guess | Guesses propagate as false confidence |
| Keep the evidence class on every claim | `VERIFIED` / `DOCUMENTED_NOT_HARDWARE_VERIFIED` / `PLAUSIBLE_REQUIRES_TESTING` / `UNKNOWN` / `NOT_PERMITTED` |
| Prefer a diagram labelled "proposed" to a fake screenshot | Honest and more useful |

---

## 9. If you only remember five things

1. **Zero code. Zero tests. Zero gates passed.** Read
   [`../overview/current_project_state.md`](../overview/current_project_state.md).
2. **You are blocked on external answers, not on engineering.** Chase the school.
3. **The only unblocked high-value task is the simulation proof of concept.**
4. **Never fabricate evidence.** The eight images in the repo are irrelevant
   display hardware.
5. **CloudLight is someone else's. Do not modify `docs/cloudlight/`.**