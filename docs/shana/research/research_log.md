# Research Log

⚠️ **Append-only. Never edit or delete an entry. Never reorder.**

This is the chronological record of research activity. It exists so that a
future agent can reconstruct how conclusions were reached, and so that a
changed conclusion is visibly a *change* rather than a silent rewrite.

**Append new entries at the bottom.**

---

## Entry format

```
## YYYY-MM-DD — Title

**Activity:** What was done
**Sources:** Which sources were consulted (link to source_register.md)
**Findings:** What was learned
**Impact on decisions:** Which decisions, assumptions, or risks changed
**Status change:** Any component whose implementation status changed
**Still unknown:** What remains unresolved
```

---

## 2026-10-11 — Platform selection completed

**Activity:** Comparative study of the Unitree G1 humanoid, the G1 humanoid EDU,
and the Go2 quadruped against the school delivery requirement.

**Sources:** Unitree vendor documentation; school requirement framing.

**Findings:** The Go2 EDU best matches an indoor delivery task. The G1 humanoid
carries a manipulation and cost burden the task does not need. A humanoid arm
would not help, since delivery does not require handling beyond a tray.

**Impact on decisions:** Platform narrowed to the Go2 family. G1 excluded.

**Still unknown:** 🔴 The school robot's **exact model, firmware, and SDK
licence eligibility were never confirmed** — this remains Gate 1.

---

## 2026-10-11 — Secondary development interface verified

**Activity:** Verified the programming eligibility of Go2 variants.

**Sources:** Unitree vendor documentation, quoted verbatim.

**Findings:** *"Only EDU or above versions can be developed, other versions such
as PRO and AIR versions do not open the secondary development interface."*

**Impact on decisions:** Established as a **hard constraint**. If the school's
robot is not EDU-or-above, the project cannot proceed at all. This is recorded
as claim A1 in the claim–evidence matrix.

**Still unknown:** Which variant the school owns.

---

## 2026-10-11 — Floor identity identified as a hard blocker

**Activity:** Reviewed the documented Go2 sensor suite.

**Sources:** Unitree vendor sensor documentation.

**Findings:** The robot has **no altitude, pressure, or barometer sensor**. It
therefore **cannot determine which floor it is on**.

**Impact on decisions:** Multi-floor operation is impossible without external
marking. ADR-006 (ArUco floor identity) was created as a *provisional* response.
Stairs were excluded (ADR-003). The lift dependency became Tier 1.

**Still unknown:** Whether marker-based identity is viable at school scale.

---

## 2026-10-11 — Stair support found to be removed

**Activity:** Reviewed the current SDK release notes.

**Sources:** Unitree SDK changelog.

**Findings:** The stair gait setter was **removed from the SDK at firmware
≥1.1.6**.

**Impact on decisions:** Stairs permanently excluded for the pilot. Recorded as
ADR-003 and claim A4.

---

## 2026-10-11 — Lift integration found to have no sanctioned path

**Activity:** Assessed the installed Johnson lifts for a sanctioned API.

**Sources:** Johnson lift documentation; general integration assessment.

**Findings:** **No sanctioned integration API exists** for the installed lifts.
Achieving one would require physical modification, which is **unlawful**.

**Impact on decisions:** Cross-floor scope permanently excluded for the pilot.
`infra/elevator_integration.md` retained to explain *why* exclusion is correct.

**Still unknown:** 🔴 The lift manufacturer was **never contacted**. This remains
an open Tier 1 dependency.

---

## 2026-10-11 — CCTV occupancy assessed and deprecated

**Activity:** Assessed using existing school CCTV as a corridor occupancy source.

**Sources:** ONVIF/CCTV technical references; privacy analysis.

**Findings:** Technically feasible, but student privacy exposure is
**disproportionate** to the benefit, particularly given that occupancy can be
declared by staff far more cheaply and reliably.

**Impact on decisions:** 🔴 CCTV marked `DEPRECATED` and **recommended against**.
Drove the privacy position that no student may ever be identified. The
knowledge is retained only to explain why the option was rejected.

**Still unknown:** 🔴 No formal privacy assessment was obtained.

---

## 2026-10-11 — India RF regulation researched

**Activity:** Researched regulatory constraints on radio telemetry.

**Sources:** Gazette citations recorded in
[`appendix_india_rf_research.md`](appendix_india_rf_research.md).

**Findings:** Unlicensed radio operation is regulated; frequency allocation
carries licensing conditions.

**Impact on decisions:** Radio/LoRa telemetry **deferred** (ADR-012). Not
required for the pilot.

**Still unknown:** ⚠️ Whether the locally-sourced rule under the Indian
Information Technology Act applies to this deployment. **Requires legal
counsel — research is not legal advice.**

---

## 2026-10-11 — Safety case drafted with no evidence

**Activity:** Drafted the safety argument structure.

**Sources:** General robotics safety practice; hazard analysis.

**Findings:** A structure exists but **zero evidence has been gathered** for any
safety property. Every mitigation is currently unverified.

**Impact on decisions:** Safety case status set to `DESIGN_ONLY`. Recorded as an
explicit gap — the **human is the safety system**
([`../operations/human_intervention.md`](../operations/human_intervention.md)).

**Still unknown:** 🔴 Stop distance, stopping reliability, and false-negative
rates are all unmeasured.

---

## 2026-10-11 — Questionnaire and gate register established

**Activity:** Generated the feasibility questionnaire and the go/no-go gate
structure.

**Findings:** 434 questions across 20 categories. 11 gates defined.

**Impact on decisions:** Created the project's own rules: implementation is not
permitted until gates pass.

**Still unknown:** 🔴 **0 of 434 questions answered. 0 of 11 gates passed.**

---

## 2026-10-11 — Repository audit: no implementation exists

**Activity:** Direct filesystem audit of the repository to verify whether the
documentation's framing matched reality.

**Sources:** Filesystem scan of all source extensions; `git status`, `git log`,
`git remote -v`.

**Findings:**
- **0 implementation code files.** No `.py`, `.js`, `.ts`, `.cpp`, `.go`, `.rs`.
- **0 tests.** No test files, no test directories, no frameworks.
- **0 build configuration.** No `package.json`, `pyproject.toml`, `Makefile`.
- **0 CI configuration.** No `.github/workflows/`.
- **0 commits, 0 remotes.** All work untracked.

**Impact on decisions:** Created `overview/current_project_state.md` and
`handoff/implementation_inventory.md` to make the gap impossible to miss.

**Still unknown:** Nothing — this was a determinate finding.

---

## 2026-10-11 — 🔴 Image assets found to be unrelated

**Activity:** Visually inspected every image file in the repository.

**Findings:** All 8 root JPEGs (`Image.jpeg`, `Image 2.jpeg` … `Image 8.jpeg`)
are **close-up photographs of LCD and display panel assemblies** — flex ribbon
cables, LCD modules, driver boards. **None relates to a robot, a school, or any
SHANA subject.**

**Impact on decisions:** Created `handoff/exhibition_evidence_inventory.md` with
an explicit prohibition on using these as evidence. Recorded as defect 3.1 in
`testing/known_failures.md`.

**Still unknown:** ⚠️ Why these images are present. **Recommend the owner move or
delete them** — they are a mis-presentation risk at an exhibition.

---

## 2026-10-11 — 🔴 Security defect found

**Activity:** Inspected `start_opencode_server.sh`.

**Findings:** The script contains a **hardcoded plaintext administrative
password** and binds the OpenCode server to `0.0.0.0:4096`, exposing it on all
network interfaces. It is not in `.gitignore`.

**Impact on decisions:** Recorded as a **critical open defect** requiring owner
rotation. The credential value was deliberately **not copied into any SHANA
document**. Created `safety/security.md` to codify credential rules.

**Still unknown:** 🔴 Whether the credential has already been exposed. **It must
be treated as compromised and rotated.**

---

## 2026-10-11 — Documentation restructured and defects corrected

**Activity:** Moved the SHANA documentation under `docs/shana/` to separate it
from the CloudLight namespace; renamed files to match their content; validated
all links.

**Findings:**
- 12 broken relative Markdown links existed after renames — **now 0**.
- The ADR index **claimed 12 ADRs but only 2 existed** — the index now records
  the gap honestly and points to the decision log.
- `cctv_research_evidence.md` referenced a non-existent source register — fixed.

**Impact on decisions:** Documentation integrity restored. ⚠️ **10 ADRs remain
owed** — tracked in `handoff/unresolved_dependencies.md` item 23.

**Still unknown:** Nothing — these were determinate defects.

---

<!-- APPEND NEW ENTRIES BELOW THIS LINE -->