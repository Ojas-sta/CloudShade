# `questionnaires/` — how to use

This directory holds the feasibility questionnaire and its Google Sheets
conversion prompt.

---

## Files

| File | Purpose |
|---|---|
| `SHANA_Feasibility_QA.md` | **434 questions** across 20 categories. Every `Answer` field is empty and every `Status` is `UNANSWERED`. Self-contained — a reader needs nothing else to understand what is being asked. |
| `GEMINI_SHEETS_ONLY_PROMPT.md` | A conversion prompt that instructs Gemini to build a Google Sheets workbook from the questionnaire **without answering anything**. |

---

## Why the questionnaire exists

The feasibility study found that SHANA's architecture **cannot be finalised**
without answers from people outside the SHANA team. Specifically:

- The school must authorise development access to the robot.
- IT must confirm what network infrastructure is permitted.
- Facilities must measure the stair geometry.
- The lift service provider must confirm what integration, if any, is permitted.
- The Data Fiduciary must approve any data processing.

**This directory exists to gather those answers as evidence, not opinion.**

---

## How to work through it

1. **Start with `START HERE`** in the converted sheet — or the equivalent section
   at the top of the markdown if working from the file directly.
2. **Answer P0 questions first.** They are safety, permission, and
   fundamental-feasibility blockers.
3. **Answer only what you can support.** An answer of *"unknown"* is a valid and
   valuable response. A guess on a P0 question is worse than no answer.
4. **Attach evidence.** Photos, screenshots, policy documents, datasheets.
   `ANSWERED` means someone replied. `VERIFIED` means evidence exists.
5. **Do not answer for someone else.** School staff answer school questions.
   The SHANA team answers internal questions.

---

## Rules that must not be broken

- ❌ **No question is answered by the SHANA team on behalf of school staff.**
- ❌ **No question is answered by Gemini.** The conversion prompt is explicitly
  constrained to produce an empty form.
- ❌ **Nothing is marked `VERIFIED` without an attachment.**
- ❌ **No credentials, keys, tokens, CCTV details, or private network
  information** go into the `Evidence` column or anywhere else in this repository.

---

## Question ID prefixes

| Prefix | Category |
|---|---|
| `SH-GO2` | Go2 EDU hardware, software, SDK, interfaces |
| `SH-NET` | School network, Wi-Fi, wired LAN, connectivity |
| `SH-SRV` | Local server, dashboard, authentication, security |
| `SH-RAD` | LoRa, ESP32, gateways, antennas, long-range telemetry |
| `SH-CCTV` | CCTV, computer vision, occupancy, privacy |
| `SH-CONG` | Corridor congestion and dynamic route selection |
| `SH-ENV` | Wet floors, rain, lighting, environmental hazards |
| `SH-BLOCK` | Obstacles, construction, blocked corridors |
| `SH-LIFT` | Elevators, lift control, authorised interfaces |
| `SH-STAIR` | Stairs, ramps, vertical navigation |
| `SH-MAP` | Maps, routes, localisation, checkpoints, ArUco |
| `SH-BAT` | Battery, charging, recovery, emergency return |
| `SH-TIME` | Schedules, lunch breaks, operating windows |
| `SH-DEL` | Delivery workflow, teachers, recipients, items |
| `SH-SAFE` | Safety, emergency stops, failure handling |
| `SH-PRIV` | Permissions, privacy, data retention |
| `SH-EXPO` | Exhibition, evaluation, acceptance testing |
| `SH-BUD` | Budget, installation, maintenance, ownership |
| `SH-GIT` | Git repository, documentation, configuration, deployment |
| `SH-DEC` | Architecture decisions, go/no-go criteria |

Question IDs are **stable**. If a question is removed or reworded, it keeps its ID
and the change is noted — IDs are never reused for a different question.

---
---

# CloudLight — additional questionnaire

> **Added for the CloudLight platform.** CloudLight is the school-facing platform;
> SHANA is the robotics subsystem within it. The content above applies to SHANA
> and is unchanged. The content below applies to CloudLight and is additive.

## Additional CloudLight files

| File | Purpose |
|---|---|
| `Additional_CloudLight_Questionnaire.md` | **576 questions** across 26 categories (`CL-*`). Every `Answer` field is empty and every `Status` is `UNANSWERED`. Self-contained. |
| `GEMINI_CLOUDLIGHT_SHEETS_PROMPT.md` | A conversion prompt that builds a Google Sheets workbook from **the CloudLight questionnaire only**, **without answering anything**. |

⚠️ **The two Gemini prompts are separate and must not be merged.** Each converts
exactly one file. `GEMINI_SHEETS_ONLY_PROMPT.md` is unchanged and continues to
convert `SHANA_Feasibility_QA.md`.

## Why the CloudLight questionnaire exists

The CloudLight architecture study found that the platform **cannot be finalised**
without answers from the school, its IT department, its timetable and
administration staff, its library, and its facilities team.

The questionnaire is large because almost nothing about this school's
infrastructure has been confirmed — Windows editions, administrator rights,
network segmentation, server permission, identity directory, shared storage,
timetable ownership and AI policy are all unknown, and each unknown blocks a
specific architecture decision. **If a research phase answers them, the
questionnaire will shrink.** That is the intent.

## How to work through it

The method is identical to the SHANA questionnaire above, with one addition:

- ❌ **Never enter a real password, key or credential in any column.** Request the
  policy, or a screenshot with values redacted.
- ❌ **No photograph of a school document, timetable or whiteboard** belongs in
  this repository.

## CloudLight question ID prefixes

| Prefix | Category | Qty |
|---|---|---|
| `CL-GEN` | Product goals and school workflows | 22 |
| `CL-WIN` | Windows smartboards, kiosk startup and device management | 24 |
| `CL-NET` | School network connectivity, addressing and segmentation | 24 |
| `CL-SRV` | Server hosting, backend placement and operations | 22 |
| `CL-TIME` | Timetable data — sources, models and reconciliation | 24 |
| `CL-SUB` | Substitution sheet processing and approval | 24 |
| `CL-AIML` | AI/ML assisted extraction, validation and human review | 22 |
| `CL-DASH` | Classroom dashboard, countdowns and degraded states | 22 |
| `CL-AUTH` | Teacher identity and authentication | 24 |
| `CL-CODE` | Six-digit codes and single-action authorisation | 20 |
| `CL-NAS` | NAS and shared-folder access | 22 |
| `CL-RES` | Teacher resource directory and metadata | 20 |
| `CL-SEARCH` | Resource search, indexing and ranking | 24 |
| `CL-AI` | AI assistant and retrieval-augmented answers | 24 |
| `CL-REACT` | React frontend and backend | 22 |
| `CL-DATA` | Local database, persistence, backup and recovery | 20 |
| `CL-OTA` | OTA updates, integrity verification and rollback | 22 |
| `CL-SEC` | Security, privacy and threat model | 26 |
| `CL-SHANA` | SHANA integration boundary | 24 |
| `CL-ADMIN` | Administration, permissions and roles | 20 |
| `CL-OPS` | Operations, backups, maintenance and support | 22 |
| `CL-UX` | Accessibility and touchscreen usability | 20 |
| `CL-EXPO` | Exhibition, pilot and acceptance testing | 20 |
| `CL-BUD` | Budget, procurement and deployment | 20 |
| `CL-GIT` | Repository, testing and documentation | 20 |
| `CL-DEC` | Architecture decisions and go/no-go criteria | 22 |
| | **TOTAL** | **576** |

⚠️ **`SH-*` and `CL-*` are disjoint namespaces.** No question ID exists in both.
CloudLight IDs follow the same stability rule: an ID is never reused for a
different question.

## Where CloudLight answers are recorded

| Register | Location |
|---|---|
| Open questions | `docs/cloudlight/open_questions_register.md` |
| Decisions | `docs/cloudlight/decision_log.md` |
| Risks | `docs/cloudlight/risk_register.md` |
| Feasibility | `docs/cloudlight/feasibility_report.md` |
| Architecture | `docs/cloudlight/architecture.md` |
| Overview | `docs/cloudlight/overview.md` |

## Boundary note

Some questions overlap in subject between the two questionnaires — the school
network, the server, and the timetable are each asked from **both** systems' points
of view. **Both must be answered.** A single answer does not satisfy both, because
the two systems have different requirements, different network paths, and
different safety boundaries.