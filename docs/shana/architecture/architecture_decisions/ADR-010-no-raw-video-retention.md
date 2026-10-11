# ADR-010: No raw video ever leaves the robot

**Status:** `Accepted`
**Date:** 2026-10-10
**Decides:** Whether raw video, still imagery, or any footage of identifiable
people may leave the robot's compute boundary
**Reversal condition:** 🔴 **Not reversible by engineering.** A reversal requires a
demonstrated, independently reviewed method that provably prevents identification
from the transmitted data, *and* a school decision authorising it. Neither exists.
Absence of identification is a sufficient and non-negotiable condition; presence of
useful signal is not.
**Evidence:** `docs/shana/safety/student_privacy.md` ·
`docs/shana/feasibility/decision_log.md` `D-006` · `AGENTS.md` §L.9, §L.10 ·
`docs/shana/architecture/cloudlight_integration.md` invariant 4.3

---

## Context

A robot moving through a school encounters children. Its depth camera necessarily
observes them. Several design proposals would transmit that observation
somewhere — for debugging, for occupancy analysis, for "seeing why the robot
stopped", or for the teacher-facing interface.

Transmission of video is the single highest-risk privacy action available to this
project. The subjects are children, in a school, routinely and without any
practical expectation of being filmed for analysis.

---

## Decision

🔴 **No raw video ever leaves the robot's compute boundary.**

| # | Rule |
|---|---|
| 1 | No video stream is transmitted from the robot, on any interface, in any mode |
| 2 | No still frames are transmitted from the robot |
| 3 | No footage is retained on the robot after processing |
| 4 | No video is ever used to identify, track, or profile any person |
| 5 | 🔴 The prohibition binds **debug and exhibition modes equally** |

**What may leave the robot is a derived abstraction, never pixels.**

---

## Rationale

**Why "only in debug" fails.** Debug access is where raw data habitually escapes,
and it is exactly the mode used for demos and troubleshooting — the modes with the
widest audience and the least scrutiny. A rule that holds in production and fails
in debug has no safety value.

**Why identification is the decisive concern.** Even without facial recognition,
footage of children in a school is a serious privacy matter. The system's own
operating rules prohibit identification of any person (`AGENTS.md` §L.9). Making
identification *technically impossible* by never transmitting the evidence is
stronger than promising not to process it.

**Why derived abstractions are sufficient.** The system's actual needs are
narrow: *is the path clear*, *is someone in the corridor*, *why did the robot
stop*. None of these require pixels leaving the device. A stop reason and an
occupancy boolean answer the operational question completely.

**Why retention is prohibited even locally.** Data that never leaves still
accumulates. Footage on a robot that gets stolen, borrowed, or repaired is a
lifetime exposure with no owner to consent.

---

## Evidence

| Fact | Class | Source |
|---|---|---|
| No raw video retention is an operating restriction | `DESIGN_ONLY` | `safety/operating_restrictions.md` |
| 🔴 Student data is never collected; no person is ever identified | `DESIGN_ONLY` | `AGENTS.md` §L.9 |
| CCTV as an occupancy source was deprecated on privacy grounds | `RESEARCH_ONLY` | `research/cctv_source_register.md`; `infrastructure/cctv_feasibility.md` |
| 🔴 No implementation exists, so no control is verified | `VERIFIED` | repository audit 2026-10-11 |

---

## Consequences

**Accepted**

- Privacy risk is bounded at the device boundary, independent of every other
  control in the system.
- Debugging works from derived state — stop reasons, occupancy flags, route
  segment, battery — never from video.
- The exclusion of CCTV occupancy (ADR-009 family) is consistent rather than
  anomalous.

**Rejected**

- Streaming to a teacher interface.
- "Sample frames on demand" for support.
- Storing footage for post-incident review. 🔴 **An incident report is a written
  account; it does not require retaining footage of a corridor full of children.**
- Any exception granted for a specific investigation. 🔴 **There are no
  exceptions.** The subjects cannot consent.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Transmit video with faces blurred at source | 🔴 Blur is a processing step that can fail, and faces at 10 m in a corridor are already too small to reliably detect. A failed blur is an identification of a child. |
| Transmit only on explicit human request | Debug access is exactly the unmonitored path. Rejected. |
| Short retention on the robot, deleted after upload | Retention is the risk. Rejected. |
| Use video for occupancy only, no storage | 🔴 Occupancy is available by human declaration (ADR-009) and by depth-based presence detection. Video is not required for it. Rejected as unnecessary risk. |

---

## Follow-up tasks

| # | Task | Owner | Status |
|---|---|---|---|
| 1 | Verify no video-capable interface exists on the robot link | Agent 1 | ⛔ `BLOCKED` — no hardware |
| 2 | Specify the derived-abstraction interface that replaces video | Agent 1 | ⛔ `BLOCKED` — depends on R8 simulation PoC |
| 3 | Obtain written school acknowledgement of this invariant | School | ⛔ `REQUESTED` |
| 4 | Confirm CloudLight's contract contains no video field | Agent 2 | ⛔ `REQUESTED` — see `CONF-06` |

---

## Related

- [`../../safety/student_privacy.md`](../../safety/student_privacy.md)
- [`../../infrastructure/cctv_feasibility.md`](../../infrastructure/cctv_feasibility.md)
- [`../../architecture/cloudlight_integration.md`](../../architecture/cloudlight_integration.md) — invariant 4.3
- [`../../research/claim_evidence_matrix.md`](../../research/claim_evidence_matrix.md)
- `AGENTS.md` §L.9, §L.10 — 🔴 *"No student data is collected, and no person is ever identified. No raw video ever leaves the robot."*