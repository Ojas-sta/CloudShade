# Evidence Index — What Actually Exists

**Owner:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Method:** direct inspection of the working tree. Every count below was
measured, not estimated.

---

## 🔴 The headline

> **There is no evidence that either system exists, works, or has been
> demonstrated. There is one category of genuinely excellent evidence: a
> feasibility study.**

| Evidence class | Count | Exhibition value |
|---|---|---|
| 🔴 Code, tests, build artefacts | **0** | ⛔ none |
| 🔴 Robot photographs or video | **0** | ⛔ none |
| 🔴 Screenshots of any system | **0** | ⛔ none |
| 🔴 Test results | **0** | ⛔ none |
| 🔴 Measurements | **0** | ⛔ none |
| 🔴 Route surveys executed | **0** | ⛔ none |
| 🔴 Permissions granted | **0** | ⛔ none |
| ✅ **Sourced research** | **68 SHANA + 38 CloudLight = 106** | 🔴 **High — as research** |
| ✅ **Structured question sets** | **≈1,000** | 🔴 **High — as analysis** |
| ✅ **Risk registers** | **67** | 🟠 Moderate |
| ✅ **Go/no-go gates** | **20** | 🔴 **High — as rigour** |
| ✅ **Architecture design** | Documented both sides | 🟠 Moderate |
| ✅ **Data schemas** | 4 proposed YAML | 🟡 Low |

⚠️ **This is the evidence base for a well-executed feasibility study. It is
not the evidence base for a working system, and must never be presented as one.**

---

## 1. 🔴 Photographs — a specific hazard

Eight JPEG files sit at the repository root: `Image.jpeg`, `Image 2.jpeg`
through `Image 8.jpeg`.

🔴 **All eight were visually inspected. All eight are close-up photographs of
LCD and display panel assemblies — flex ribbon cables, LCD modules, driver
boards.** They have **no relationship** to a quadruped robot, a school, a map,
or any project subject.

🔴 **Therefore:**
- ⚠️ There is **no photograph of the Go2 robot** in this repository
- ⚠️ There is **no photograph of the school, corridors, or destinations**
- ⚠️ There are **no screenshots of any system**

**Action taken:** all eight are excluded from version control by `.gitignore`
(`*.jpg`, `*.jpeg`). They were excluded before this audit and remain excluded.

🔴 **These images must never be captioned as SHANA output, must never appear in
exhibition material, and must never be offered to a proposal author as project
evidence.** The proposal author should be told they exist and what they are, so
they are not mistaken for assets.

---

## 2. Research evidence — the strongest asset

### SHANA source register

🔴 **This is high-quality work and should be presented as such.** It is not
overclaimed, and its limitations are documented honestly.

| Attribute | Assessment |
|---|---|
| Sources | 68 numbered entries (the register's own numbering runs to 68) |
| Method limits documented | ✅ Yes — explicitly |
| Contradictions recorded | ✅ Yes — a dedicated §6 |
| Verbatim quotes | ✅ Yes |
| Vendor limitations noted | ✅ Yes |
| Hardware-version applicability noted | ✅ Yes |

**Documented access problems — good practice:**
- `support.unitree.com` is WAF-blocked to direct fetch; content retrieved via a
  text-render proxy. URLs cited are origin URLs.
- `api.github.com` rate-limits unauthenticated requests; listings obtained via
  authenticated CLI and `codeload` tarballs.

⚠️ **This transparency raises the evidence's credibility.** A proposal should
rely on it.

### Findings the research establishes, with sources

| # | Finding | Class | Source basis |
|---|---|---|---|
| 1 | 🔴 *"Only EDU or above versions can be developed, other versions such as PRO and AIR versions do not open the secondary development interface."* | **Verbatim vendor quote** | Unitree Developer FAQ |
| 2 | 🔴 *"Go2 only supports wired connection for development"* | **Verbatim** | Unitree Developer FAQ |
| 3 | 🔴 `IMUState.msg` has **exactly five fields** — `quaternion`, `gyroscope`, `accelerometer`, `rpy`, `temperature`. **No altitude. No pressure. No barometer.** | **Source-code analysis** | `unitree_ros2` `IMUState.msg` |
| 4 | 🔴 `HeightMap.msg` `origin` is `float32[2]` — **XY only, no Z datum** | Source-code analysis | `HeightMap.msg` |
| 5 | Max step height **16 cm**; slope ±40° | Vendor-documented | Unitree developer docs |
| 6 | `SwitchGait` **absent** from the Motion Services v2.0 interface for firmware ≥ V1.1.6 | Source-code + docs | `sport_client.hpp`; v2.0 docs |
| 7 | `SwitchGait` **exists** for firmware **< V1.1.6** — enum `3 = climb stair` | Source-code + docs | `High_motion_control` |
| 8 | `Damp()` — *"highest priority and is used for emergency stops"* | Vendor-documented | v2.0 interface |
| 9 | `RecoveryStand()` — recovers to standing regardless | Vendor-documented | v2.0 interface |
| 10 | No charging or dock service in the Go2 service list | Source-code analysis | `RobotStateClient`; `sport_client.hpp` |
| 11 | `SportModeState.msg` contains **no battery fields** — battery is in `BmsState.msg` | Source-code analysis | Message definitions |
| 12 | Extended electrical interfaces are **exclusive to Go2 EDU** | Vendor-documented | Payload docs |

🔴 **Finding 3 is the most consequential and least discussed.** A robot with no
altitude, pressure or barometer **cannot determine which floor of a building it
is on**. This is not a software gap and no engineering removes it.

⚠️ **Finding 6 vs 7 is a genuine conflict**, correctly recorded: stair gait was
**removed** at firmware V1.1.6. 🔴 **The installed firmware is `UNKNOWN`** —
which is why obtaining it is gate-blocking.

### 🔴 Known gaps in the SHANA register

| Gap | Detail |
|---|---|
| ArUco Recharge Service | ⚠️ A page **exists** in the navigation tree; **body would not render through 8 URL variants**. **NOT READ.** Recorded as `TEC-09` |
| Fps contradiction | Product page 15 fps vs developer docs 30/60 fps — recorded, not resolved |
| Battery runtime | Marketing contradicts developer docs — recorded |

⚠️ **`NOT READ` and `NOT FOUND` are recorded honestly.** That is correct
practice and should be preserved.

### CloudLight source register

| Class | Count |
|---|---|
| `VERIFIED` | 29 |
| `PLAUSIBLE_REQUIRES_TESTING` | 10 |
| `DOCUMENTED_NOT_HARDWARE_VERIFIED` | 10 |
| `UNKNOWN` | 31 |
| `INTERNAL` | 6 |

🔴 **Note the shape:** 11 verified claims, all about **the world** — `.xlsx` is
a ZIP of XML; text-layer PDFs yield tokens; OCR dominates cost; Assigned Access
and Shell Launcher are edition-dependent and cannot be combined; a Windows
Service cannot display UI; SMS/PSTN is a restricted authenticator under NIST
SP 800-63B; TUF names rollback, freeze, fast-forward and mix-and-match; SMBv1
is deprecated; an access point is a Layer-2 bridge.

🔴 **Zero school-specific facts are verified, because none has been checked.**

---

## 3. Structured analysis assets

| Asset | Count | Location |
|---|---|---|
| SHANA questions | **434** · 0 answered | `questionnaires/SHANA_Feasibility_QA.md` |
| CloudLight questions | **576** · 0 answered | `questionnaires/Additional_CloudLight_Questionnaire.md` |
| Total asked | 🔴 **≈1,000** · **0 answered** | — |
| Permissions sought | **104** · 0 granted | `docs/shana/administration/permission_checklist.md` |
| Go/no-go gates | **20** (11 SHANA + 9 CloudLight) · **0 passed** | — |
| Risks registered | **67** (22 SHANA + 46 CloudLight) · **0 retired** | — |
| Assumptions | **58** (34 SHANA + 24 CloudLight) · **1 verified** | — |
| Requirements | **186** · 0 agreed | `docs/cloudlight/requirements.md` |
| Conflicts found by integration review | **11** · 1 resolved | `docs/integration/unresolved_conflicts.md` |

⚠️ **These are the project's real assets.** A proposal should present them as
*what a rigorous feasibility phase produces* — which is genuinely impressive —
and not as delivery progress.

---

## 3a. 🔴 Count discrepancies found during this audit

🔴 **Agent 3's own figures initially did not match the repository.** This
section records that, because a proposal that cites a number the repository does
not support is exactly the failure this project exists to avoid.

| Metric | Subsystem documents claim | Measured by Agent 3 | Method |
|---|---|---|---|
| SHANA questions | **434** | 🔴 **410** | Unique `SH-*` IDs |
| CloudLight questions | **576** | ✅ **576** | Unique `CL-*` IDs |
| SHANA risks | **21** | 🔴 **22** | Unique `R-*` IDs |
| CloudLight risks | **46** | ✅ **46** | Unique `R-*` IDs |
| SHANA permissions | **102** | 🔴 **104** | `NOT REQUESTED` rows |
| SHANA research sources | — | **68** | Highest numbered row |
| CloudLight research entries | — | **38** | Unique `CR-*` IDs |
| Gates | 11 + 9 = **20** | ✅ **20** | `GATE 0`–`10`, `GATE 0`–`8` |
| Requirements | 133 + 53 = **186** | ✅ **186** | Unique `FR-*` / `NFR-*` |

### Why this matters more than it looks

⚠️ The discrepancies are **small** — the documents are not wrong in substance.
But:

1. 🔴 **A proposal author who copies "1,010 questions" from a summary and an
   evaluator who counts 986 rows has found a discrepancy.** 🔴 That reads as
   carelessness, and it undermines every other number.
2. ⚠️ The SHANA question count differs by 24. **Whether that is 24 omitted rows
   or 24 sub-questions folded into parents is `UNKNOWN`** — the questionnaire
   would need reading in full to determine.
3. ✅ **Using "≈1,000 questions" is accurate under either figure**, and is what
   this integration layer uses throughout.

### Recommendation

🔴 **Agent 1 and Agent 2 should reconcile their summary figures against their own
content**, or state the counting method alongside each figure.

⚠️ **This is a small finding with a disproportionate effect on credibility** —
and it was found by counting, not by reading.

---

## 4. 🔴 What does not exist

| Item | Status |
|---|---|
| Source code | 🔴 **0 files** |
| Tests | 🔴 **0 written, 0 run** |
| Build configuration | 🔴 **none** |
| CI configuration | 🔴 **none** |
| Database | 🔴 none created |
| CloudLight frontend | 🔴 none |
| CloudLight backend | 🔴 none |
| Robot code | 🔴 none |
| Simulation | 🔴 **none — 🔴 this is the unblocked task** |
| Hardware access | 🔴 **never** |
| Network access | 🔴 **never probed** |
| CCTV access | 🔴 **never** |
| Lift access | 🔴 **never** |
| AI service contacted | 🔴 **never** |
| School documents read | 🔴 **none** |
| Go/no-go gates passed | 🔴 **0 of 20** |
| Capabilities demonstrated | 🔴 **0** |
| Permissions granted | 🔴 **0 of 104** |

---

## 5. 🔴 Security exposure

| Item | Detail | Status |
|---|---|---|
| `start_opencode_server.sh` | 🔴 Hardcoded plaintext administrative password; binds `0.0.0.0:4096` | **Not committed** — gitignored |
| Credential quoted in any document? | 🔴 **No** | ✅ Verified |
| 🔴 **Rotation** | 🔴 **REQUIRED — outstanding** | ⛔ Owner action |

⚠️ Being gitignored does not un-expose a cleartext password on disk.

---

## 6. 🔴 Research gaps worth closing

Ranked by whether they change a decision. ⚠️ **None is answerable by Agent 3.**

| # | Gap | Why it matters | Who can answer |
|---|---|---|---|
| 1 | 🔴 Go2 model, **firmware version**, SDK licence eligibility | Firmware determines whether stair gait exists (finding 6 vs 7) | Robot custodian |
| 2 | 🔴 `ArUco Recharge Service` page body — never read | 🔴 **May contradict "no return-to-dock"** | Anyone with browser access |
| 3 | Lift manufacturer — any sanctioned interface? | Determines cross-floor scope | Johnson, via school |
| 4 | Stair riser heights | >160 mm → stairs permanently out | Facilities |
| 5 | Battery fitted — 8,000 or 15,000 mAh | Battery reserve | Custodian |
| 6 | Windows edition on classroom boards | Kiosk feasibility | IT |
| 7 | Does a NAS exist **at all**? | 🔴 May close the resource track permanently | IT + Library |

⚠️ **Gap 2 is the cheapest and highest-value.** It is a documentation lookup,
requires no permission and no hardware, and 🔴 **a positive finding would
materially improve SHANA's autonomy case.** If "no return-to-dock" is wrong,
recovery assumptions throughout the project need revisiting.

---

## 7. Index summary

| Class | Items | Exhibition use |
|---|---|---|
| 🔴 **Product evidence** | **0** | ⛔ **None. Nothing may be claimed as demonstrated.** |
| ✅ **Research evidence** | 106 sourced entries | 🔴 **Primary asset** — presented as research |
| ✅ **Analytical rigour** | ≈1,000 questions · 20 gates · 68 risks · 186 requirements | 🔴 **Primary asset** — presented as method |
| ✅ **Architecture** | Both subsystems, independently derived and convergent on the boundary | 🟠 Supporting |
| ⛔ **Absent** | code, tests, screenshots, photographs, measurements, permissions | 🔴 **Must be stated, not hidden** |

---

## 8. 🔴 The statement a proposal may truthfully make

> *This project has completed a rigorous feasibility study. It has produced
> approximately ≈1,000 structured questions, 20 formal go/no-go gates, 67
> registered risks and 106 sourced research entries — grounded in
> manufacturer documentation and source-code analysis. It has **not** built a
> system. It has **not** operated a robot, contacted school infrastructure, or
> obtained any permission. The current state is a well-evidenced assessment of
> what is feasible and what must be resolved first.*

🔴 **That sentence is defensible, evidence-backed, and honest.** It is also
substantially more impressive than an overclaim would be, because it
demonstrates judgement rather than optimism.