# Exhibition Evidence Inventory

🔴 **Purpose: to state exactly what evidence exists, so that nothing is
fabricated and nothing is overclaimed.**

This document exists because SHANA is being proposed for exhibition or
demonstration, and the pressure to show *something* is real. This file is the
control against that pressure.

---

## 1. Headline finding

> ## ⚠️ THERE IS NO VISUAL EVIDENCE OF SHANA IN THIS REPOSITORY.

Eight JPEG files exist at the repository root. **All eight were visually
inspected. All eight are close-up photographs of LCD and display panel
assemblies** — flex ribbon cables, LCD modules, and driver boards.

| File | Content | Usable as SHANA evidence? |
|---|---|---|
| `Image.jpeg` | LCD / display panel assembly with ribbon cable | ❌ **No** |
| `Image 2.jpeg` | LCD / display panel assembly with ribbon cable | ❌ **No** |
| `Image 3.jpeg` | Display / electronics assembly | ❌ **No** |
| `Image 4.jpeg` | Display / electronics assembly | ❌ **No** |
| `Image 5.jpeg` | LCD / display panel assembly | ❌ **No** |
| `Image 6.jpeg` | Display / electronics assembly | ❌ **No** |
| `Image 7.jpeg` | Display / electronics assembly | ❌ **No** |
| `Image 8.jpeg` | LCD / display panel assembly | ❌ **No** |

🔴 **None of these may be captioned as a robot, a school, a prototype, or SHANA
output. They must be excluded from any exhibition material.**

---

## 2. Evidence that does exist

| Artefact | Type | What it proves | What it does **not** prove |
|---|---|---|---|
| ~100 markdown documents | Design | The project has been researched and designed | That anything was built |
| `data/schemas/*.yaml` (4) | Specification | Message shapes were considered | That any code reads or writes them |
| `questionnaires/SHANA_Feasibility_QA.md` | Questionnaire | 434 relevant questions were identified | That any were answered |
| `questionnaires/GEMINI_SHEETS_ONLY_PROMPT.md` | Tooling | A conversion workflow exists | — |
| Platform selection study | Research | A documented decision was made | That the decision is implementable |
| Source register | Citation | Sources were genuinely read | That claims were hardware-tested |

**All available evidence is documentary. There is no experimental evidence of
any kind.**

---

## 3. ⚠️ Hard prohibitions

### 3.1 Never fabricate

| ❌ Prohibited | Why |
|---|---|
| Generating a "photo of the robot" | The robot was never operated or accessed |
| Generating mock screenshots of a dashboard | No dashboard exists; a mock would imply one does |
| Rendering a 3D model and presenting it as the actual robot | Misrepresents a design as hardware |
| Claiming test results | **No tests have ever been run** |
| Inventing a school name, room number, or staff name | Unknown and potentially identifying |
| Presenting a simulated run as a real run | No simulation exists either |

⚠️ **If an exhibit needs visual material, it must be labelled a diagram, a
render, or a schematic — and described as proposed.**

### 3.2 Never overclaim what the project *does* claim

| ❌ Do not claim | Because |
|---|---|
| "The robot navigates the school" | 🔴 It has never been there; no survey exists |
| "It detects people using cameras" | 🔴 No implementation; privacy position opposes it |
| "It uses the lift" | 🔴 No sanctioned API exists; modification is unlawful |
| "It works on multiple floors" | 🔴 It cannot determine which floor it is on |
| "It returns to its dock autonomously" | 🔴 No dock integration exists |
| "The system is tested" | 🔴 0 tests run |
| "The AI makes navigation decisions" | 🔴 Explicitly prohibited by ADR-001 — the AI never reaches the robot |

---

## 4. What a truthful exhibit could contain

⚠️ **All of the following are documentary and require no fabrication.**

| Exhibit item | Source | Honest framing |
|---|---|---|
| The problem statement | [`../overview/introduction.md`](../overview/introduction.md) | "A real school problem we researched" |
| The system architecture | [`../architecture/architecture.md`](../architecture/architecture.md) | "Proposed architecture" |
| The 434 questions | `questionnaires/SHANA_Feasibility_QA.md` | "We identified 434 questions needing answers" |
| The gate register | [`../feasibility/go_no_go_gates.md`](../feasibility/go_no_go_gates.md) | "We defined 11 gates, none yet passed" |
| The risk register | [`../safety/risk_register.md`](../safety/risk_register.md) | "21 risks identified and registered" |
| The platform selection | [`../platform_selection/`](../platform_selection/) | "We chose a platform and recorded why" |
| The source register | [`../research/source_register.md`](../research/source_register.md) | "Our claims are sourced" |
| A schematic diagram | From architecture docs | "Proposed design" — **legitimate** |
| The *absence* of a robot | — | "We are at the stage before hardware access" |

⚠️ **A project that says "we have designed this rigorously and cannot build it
yet, and here is exactly why" is a stronger and more honest exhibit than a
simulation dressed up as a finished system.**

---

## 5. 🔴 Blocking questions for exhibition

| # | Question | Blocks | Who answers |
|---|---|---|---|
| 1 | Will the school grant robot access at all? | Any demonstration | School administration |
| 2 | Can any simulation be shown at all? | Any visual demo | Nobody — we would build it ourselves |
| 3 | May the exhibit show a **diagram-only** proposal? | Non-demo exhibit | Exhibition organiser |
| 4 | Is a live demo a requirement? | Whole exhibit approach | Exhibition organiser |
| 5 | May the evaluation accept documentary evidence only? | Whole exhibit approach | Exhibition organiser |

⚠️ **Question 4 is decisive and should be asked before any work is done.** If a
live robot demonstration is mandatory, the honest answer now is that this
project cannot meet it — and that should be said plainly rather than papered
over with a video, a mock-up, or a simulation presented as real.

---

## 6. Verification record

| Check | Method | Result |
|---|---|---|
| Image inventory | `ls` of repository root | 8 JPEGs found |
| Image content | Visual inspection of all 8 | **All are LCD/display panel assemblies** |
| Code existence | Filesystem scan for all common language extensions | **None found** |
| Test existence | Filesystem scan for test files and test directories | **None found** |
| CI existence | Check for `.github/workflows/` | **None found** |
| Screenshots | Visual inspection of all image files | **None** |
| Simulation artefacts | Full file listing | **None** |
| Visual evidence of SHANA | — | 🔴 **None** |

**Audit date:** 2026-10-11

---

## 7. Statement of integrity

> As of the audit date, SHANA has produced no experimental, visual, or
> performance evidence of any kind. Every capability it describes is a proposal.
>
> The only images in this repository are unrelated photographs of LCD and
> display hardware, and they are excluded from any claim about this project.
>
> **Any future exhibit material must be verifiable against
> [`../research/claim_evidence_matrix.md`](../research/claim_evidence_matrix.md),
> or it must not be presented as fact.**