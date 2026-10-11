# Claim → Evidence Matrix

🔴 **Use this before making any statement about SHANA to anyone.**

For every claim someone might reasonably make about this project, this file
gives the evidence class and an explicit **verdict**.

**Verdicts:**

| Verdict | Meaning |
|---|---|
| ✅ `SUPPORTED` | Evidence exists. Say it. |
| ⚠️ `QUALIFIED` | True only with the stated qualification. Say it *with* the qualification. |
| ❌ `UNSUPPORTED` | 🔴 **Do not say this.** No evidence exists. |
| ⛔ `PROHIBITED` | 🔴 **Do not say this. It is also prohibited** by decision or law. |

**Evidence classes:** `VERIFIED` (confirmed in official vendor documentation,
quoted) · `DOCUMENTED_NOT_HARDWARE_VERIFIED` (vendor-documented, not tested by
us) · `PLAUSIBLE_REQUIRES_TESTING` (feasibility unestablished) · `UNKNOWN`
(insufficient information) · `NOT_PERMITTED` (prohibited by law, terms, or
safety judgement)

---

## A. Claims about the robot itself

| # | Claim | Evidence | Verdict | Qualification required |
|---|---|---|---|---|
| A1 | "Only EDU-or-above Go2 models expose a development interface" | `VERIFIED` — vendor wording: *"Only EDU or above versions can be developed, other versions such as PRO and AIR versions do not open the secondary development interface."* | ✅ `SUPPORTED` | — |
| A2 | "The robot has no altitude, pressure, or barometer sensor" | `VERIFIED` — documented sensor list | ✅ `SUPPORTED` | — |
| A3 | "Therefore it cannot determine which floor it is on" | `VERIFIED` — direct consequence of A2 | ✅ `SUPPORTED` | — |
| A4 | "The stair gait setter was removed from the current SDK" | `VERIFIED` — firmware ≥1.1.6 | ✅ `SUPPORTED` | — |
| A5 | "Our school has a Go2 EDU" | `UNKNOWN` — model number **never confirmed** | ❌ `UNSUPPORTED` | 🔴 Model, firmware, and licence eligibility are all unconfirmed |
| A6 | "We have tested the robot" | No test exists | ❌ `UNSUPPORTED` | — |
| A7 | "We have operated the robot" | Never accessed | ❌ `UNSUPPORTED` | — |
| A8 | "The robot supports autonomous return-to-dock" | `DOCUMENTED_NOT_HARDWARE_VERIFIED` — vendor documents docking behaviour, but no dock integration is configured | ⚠️ `QUALIFIED` | Only ever as "vendor-documented; not integrated by us" |

## B. Claims about SHANA's capabilities

| # | Claim | Evidence | Verdict | Qualification required |
|---|---|---|---|---|
| B1 | "SHANA delivers items to classrooms" | Design only | ❌ `UNSUPPORTED` | 🔴 Never operated. Say "proposes to" |
| B2 | "SHANA navigates corridors" | Design only | ❌ `UNSUPPORTED` | 🔴 Never operated |
| B3 | "SHANA can pick up objects" | Not a capability | ⛔ `PROHIBITED` | 🔴 No arm. Carry-only, by design |
| B4 | "SHANA works across floors" | Blocked by A2, A4 | ⛔ `PROHIBITED` | 🔴 Stairs excluded; no lift API |
| B5 | "SHANA uses the school lifts" | No sanctioned API | ⛔ `PROHIBITED` | 🔴 Modification would be unlawful |
| B6 | "SHANA detects people via cameras" | Not implemented; opposed on privacy grounds | ❌ `UNSUPPORTED` | — |
| B7 | "SHANA uses CCTV for occupancy" | `DEPRECATED` — recommended **against** | ⛔ `PROHIBITED` | 🔴 Contrary to our own privacy position |
| B8 | "The AI system makes navigation decisions on the robot" | Explicitly prohibited | ⛔ `PROHIBITED` | 🔴 ADR-001: no AI reaches the robot |
| B9 | "SHANA is fully tested" | 0 tests run | ❌ `UNSUPPORTED` | — |
| B10 | "SHANA is ready for deployment" | 0 of 11 gates passed | ❌ `UNSUPPORTED` | — |

## C. Claims about the project process

| # | Claim | Evidence | Verdict | Qualification required |
|---|---|---|---|---|
| C1 | "We researched the platform thoroughly" | Source register with honest classes | ✅ `SUPPORTED` | — |
| C2 | "We chose the Go2 EDU over the G1" | [`../platform_selection/`](../platform_selection/) | ✅ `SUPPORTED` | — |
| C3 | "We identified 434 questions needing answers" | Questionnaire, 434 unique IDs | ✅ `SUPPORTED` | — |
| C4 | "We defined 11 go/no-go gates" | [`../feasibility/go_no_go_gates.md`](../feasibility/go_no_go_gates.md) | ✅ `SUPPORTED` | — |
| C5 | "**0 of 11 gates have been passed**" | Gate register | ✅ `SUPPORTED` | — |
| C6 | "**0 of 102 permissions have been granted**" | [`../administration/permission_checklist.md`](../administration/permission_checklist.md) | ✅ `SUPPORTED` | — |
| C7 | "**0 of 434 questions have been answered**" | Questionnaire status | ✅ `SUPPORTED` | — |
| C8 | "**We have run zero tests**" | [`../testing/test_inventory.md`](../testing/test_inventory.md) | ✅ `SUPPORTED` | — |
| C9 | "We have built a prototype" | No code exists | ❌ `UNSUPPORTED` | 🔴 **Zero code. Do not use the word prototype** |
| C10 | "We have a working system" | Nothing built | ❌ `UNSUPPORTED` | — |

## D. Claims about evidence and assets

| # | Claim | Reality | Verdict |
|---|---|---|---|
| D1 | "We have photographs of the robot" | 🔴 **The 8 images are unrelated LCD/display panels** | ❌ `UNSUPPORTED` |
| D2 | "We have photographs of the school" | None exist | ❌ `UNSUPPORTED` |
| D3 | "We have screenshots of the system" | No system, no screenshots | ❌ `UNSUPPORTED` |
| D4 | "We have a simulation" | 🔴 **Never created**, though ADR-011 requires one | ❌ `UNSUPPORTED` |
| D5 | "We have videos of testing" | None exist | ❌ `UNSUPPORTED` |

---

## 🔴 The six sentences never to say

1. ❌ "We've tested the robot."
2. ❌ "Our system works across floors."
3. ❌ "The AI drives the robot."
4. ❌ "We built a prototype."
5. ❌ "We demonstrated it."
6. ❌ "We integrated with the lifts."

---

## ✅ The honest version

> "SHANA is a feasibility study for an indoor school delivery robot. We chose
> the Unitree Go2 EDU over the alternatives and documented why. We verified from
> vendor documentation that the robot cannot sense which floor it is on and that
> stair support was removed from the SDK — which is why our scope is single-floor
> and excludes lifts. We registered 21 risks, defined 11 go/no-go gates, and
> identified 434 questions requiring answers from the school, its staff, and one
> lift manufacturer. **None of those gates have been passed and none of those
> questions have been answered, because we have not yet been granted access.**
> Our next step is a simulation-only proof of concept, which requires no robot
> and no student data."

That statement is **fully supported** by this repository. It is also more
credible than any overclaim, because it is true.

---

## Maintaining this file

When new evidence arrives — a survey, a test result, a written answer — update
the relevant row and **append an entry** to
[`research_log.md`](research_log.md) recording the date, what changed, and the
source.

**Never upgrade a verdict without a source.** A claim moving from
`UNSUPPORTED` to `SUPPORTED` requires evidence, not confidence.