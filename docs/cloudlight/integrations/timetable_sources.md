# CloudLight — Timetable Sources

**Phase 0 · DESIGN ONLY · 2026-10-11**

The candidate inputs for timetable and substitution data, and what each would cost.

> ## ⛔ **No school timetable has been seen. No format is confirmed. No
> substitution sheet has been read. No document has been photographed.**
>
> **The tables below are categories of input we might have to handle — not
> assertions about what this school uses.**

> ### ⚠️ **Who the authoritative source is, is an unresolved school-policy
> question** (`CL-TIME-001`, `CL-SUB-001`). Until the school answers it,
> CloudLight **consumes and renders** whatever record the school designates. It
> does not become authoritative by being installed, by being on the school
> network, or by being the only system displaying the information.

⚠️ **This document does not describe the ingestion pipeline.** It does not repeat
the parsing tiers, the record schema, validation rules or the quarantine model —
those are in [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md),
which is the single authority. This document answers a narrower question: **which
input might arrive, and what does each one cost us?**

---

## 1. The governing rule

> **Deterministic parsing first, without exception. AI never fills a missing
> value with a plausible guess. A model never publishes.**

⚠️ **If the source has a stable cell structure, read it with a real spreadsheet or
CSV parser and do not involve a model.** A model reading an `.xlsx` cannot do
better than the cell addresses it was handed, is slower, non-deterministic, costs
money, works online, and cannot be reasoned about.

| Rule | Why it is absolute |
|---|---|
| Tier 1 deterministic is **always tried first, on every input** | It is fast, free, offline, exact and — critically — **repeatable**. Every property the review gate depends on follows from that |
| **A missing value stays missing and is flagged** | ⚠️ Unknown is a valid output; a plausible invention is not. Guessing puts a real teacher in a class they are not timetabled to teach |
| **Confidence is advisory to a reviewer, never a publication criterion** | Publishing on confidence grounds makes the model's opinion school policy |
| **Validation can only reject, never repair** | A rule that finds a defect quarantines the record; it does not correct it |

---

## 2. Candidate input sources

⚠️ **❓ `UNKNOWN` which of these will actually arrive** — `CL-AIML-006`,
`CL-SUB-005`. Each is a separate path with a different failure mode and a
different confidence behaviour.

| Source | How it would be ingested | Model needed? | Typical failure mode | Permission required |
|---|---|---|---|---|
| **Excel `.xlsx`** | **Tier 1 — deterministic.** Read as what it is: a ZIP package of XML parts, ISO/IEC 29500. Cells addressed by row and column; styles carry meaning | ⛔ **No** | Merged cells spanning a header block; a second header row partway down; meaning carried by fill colour or bold rather than text; hidden rows contradicting visible ones | 📄 Hold the file |
| **CSV** | **Tier 1 — deterministic.** Flat delimited text | ⛔ **No** | Title and date lines *above* the real header row; a byte-order mark prepended to the first field name; a comma inside a name inside an unquoted field; blank rows used as period separators | 📄 Hold the file |
| **Google Sheets export** | **Tier 1 — deterministic.** Produces `.xlsx` or `.csv` | ⛔ **No** | ⚠️ **A point-in-time snapshot, not a live link.** It silently diverges from the live sheet; formulas export as their last computed value, so the *derivation* is lost; two people export minutes apart and get two different files | 📄 Access to the sheet |
| **PDF, digital (text layer)** | **Tier 1 — deterministic** (+ Tier 2 only to recover a borderless grid). Text tokens carry page coordinates; no rendering required to read them | ⛔ **No** | Column order reconstructed from coordinates is **genuinely ambiguous** in a table with no ruling lines; the header repeats on each page and is read twice; a font encoding quirk turns a character into a substitute | 📄 Hold the file |
| **PDF, scanned (image only)** | **Tiers 2 → 3 → 4.** No text tokens exist; the page is an image | ⚠️ **Yes — OCR + layout + extraction** | Skew, glare, shadow, faded toner, a crease through a cell; a handwritten margin note read as part of the grid | ⚖️ **Written permission** to derive text from the document — `CL-NAS-015`, `CL-SEARCH-005` |
| **Photographed / handwritten board** | **Tiers 3 → 4.** Image only, no fixed grid, free placement | ⚠️ **Yes — OCR + model** | Everything above, plus: reflection, marker strokes crossing a cell, an entry overwritten rather than erased, the board partly out of frame — and worst, **the board changed after the photograph was taken** | ⛔ **⛔ Photographing school material is a permission, not a technical step** — `CL-SUB-006`, `CL-SEC-011` |
| **Typed web form inside CloudLight** | **Tier 1 — deterministic.** Controlled fields and pick-lists | ⛔ **No** | Autocomplete silently substituting a near-miss name; a transposition; a person keying what they *assume* rather than what they were told | ✅ None — but only if a named approver exists at all |
| **Paper register** | **Manual keying into the web form** | ⛔ **No** | The person believes the register is current and it is not; two registers exist and nobody knows which is authoritative | ✅ None |

---

## 3. What is verified about the deterministic formats

> **This section exists because "use a real parser" is a claim, and claims need
> evidence.**

| Claim | Class | Source |
|---|---|---|
| An `.xlsx` is a ZIP package of XML parts; cell contents, addresses and formatting are **addressable without inference** | ✅ `VERIFIED` | ISO/IEC 29500; Microsoft's own documentation describes renaming a `.docx` to `.zip` to explore the XML |
| A text-layer PDF yields **real text tokens with coordinates**; an image-only PDF yields none, which is why OCR is *optional* in document pipelines | ✅ `VERIFIED` | Docling pipeline description |
| A model reading a spreadsheet cell cannot exceed the cell addresses it was given | ✅ `VERIFIED` | Logical consequence of the format; the cells are the input |
| Tier 1 output is **byte-identical on every run, on every machine** | ✅ `VERIFIED` by construction | A deterministic parser over a deterministic format |

### 3.1 What is verified about the AI stages, and why it matters here

| Stage | Published cost | Class |
|---|---|---|
| **Layout detection** | *"< 1 s/page, single CPU"* at 72 dpi | ✅ `VERIFIED` |
| **Table structure** | *"2–6 s/table on a standard CPU"* | ✅ `VERIFIED` |
| **OCR** | *"runs fairly slow on CPU (upwards of 30 seconds per page)"* | ✅ `VERIFIED` |

⚠️ **OCR is the dominant expense by one to two orders of magnitude.** The design
consequence, and the reason the pipeline is shaped this way: **OCR runs only on
pages that have no text layer.** Running it *"just in case"* on a clean export
would multiply the cost of the common case by an order of magnitude to obtain, at
best, the same characters the parser already had — and at worst, a different set
of them.

⚠️ **Three honest limits of those numbers**, because they are frequently
over-read:

- 🟡 **They are Docling's figures on Docling's corpus** — papers and manuals, not
  school substitution sheets. They justify a **ratio** between stages. They do
  **not** predict accuracy or wall-clock on a timetable.
- 🟡 **No GPU benchmark exists in that report.** The authors state GPU acceleration
  is *"currently work-in-progress and largely untested"*. ❓ We will not invent a
  CPU-vs-GPU figure. 🧪 **Any claim that CloudLight "runs OCR on GPU" is
  `PLAUSIBLE_REQUIRES_TESTING` until measured on this school's own hardware.**
- ⚠️ **The training distribution is not school timetables.** The referenced layout
  model is derived from a corpus of finance, science, patents, tenders, law and
  manuals with 11 layout classes. ⚠️ **A school timetable is not in that
  distribution.** Transfer is 🧪 `PLAUSIBLE_REQUIRES_TESTING`, not established.

---

## 4. The permission gates — separate from the technical ones

⚠️ **The most consequential gate here is not technical, and it is not ours.**

| Gate | Question | If refused |
|---|---|---|
| ⛔ **May sheets be photographed at all?** | `CL-SUB-006` | ⛔ **No photograph ingestion of any kind.** Not a degraded mode — absent |
| ⛔ **Is a photograph ever authoritative?** | `CL-SUB-007` | ⛔ **Assume a person must always confirm what was photographed.** A photograph is an input to a proposal, never a record |
| ⚖️ **May file contents be derived** — including OCR — and stored as a derived copy? | `CL-NAS-015`, `CL-SEARCH-005` | No full-text search. Metadata search only |
| 📄 **May timetable data be published to classroom screens?** | `CL-TIME-024`, `CL-DASH-001` | ⚠️ **The core feature does not exist.** Publishing to screens may not be established practice at all |

⚠️ **Photographing school material is a permission question wearing a technical
costume.** Timetables, substitution sheets and staff material contain personal
data. ⛔ **Asking the question by photographing the board would be answering it
badly and irreversibly** — that act is precisely what is being asked about.

⚠️ **Visibility is not authority.** A board that everyone can see is not a board
that anyone approved.

---

## 5. What happens for each source, end to end

⚠️ **The pipeline is in [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md).
What follows is only the per-source consequence.**

| Source | Ingestion path | Review burden on the timetable department | Publication |
|---|---|---|---|
| `.xlsx` / CSV | Tier 1 | ⚠️ **Low but not zero.** Correct extraction errors, not transcription errors | ✅ After named approval |
| Google Sheets export | Tier 1 | ⚠️ Low, **plus a staleness check** — the snapshot has already diverged by the time it is read | ✅ After named approval |
| Digital PDF | Tier 1 (+2) | 🟡 Medium — column-order ambiguity in borderless tables | ✅ After named approval |
| Scanned PDF | Tiers 2→3→4 | 🔴 **High.** Every page needs review against the original image | ✅ After named approval |
| Photographed board | Tiers 3→4 | 🔴 **Highest.** Reviewer sees the photograph and the board, side by side | ⛔ **Never authoritative.** Proposal only |
| Typed form | Tier 1 | 🟢 **Highest accuracy, highest friction.** The person is doing the transcription | ✅ After named approval |
| Paper register | Manual keying | 🟢 Same as typed form | ✅ After named approval |

⚠️ **The accuracy and the friction trade against each other.** A typed form is the
most accurate ingestion path available because there is no extraction step to get
wrong — and the most expensive for the person doing it. ⚠️ **If the school's
master is paper, providing a fast manual entry form from day one is not a failure.
It is the honest first version**, and `CL-TIME-002` returning "paper only" should
close the extraction track rather than degrade it.

⚠️ **"No teacher should have to re-key what is already written down" is not
achievable for paper.** It is worth stating plainly rather than designing an OCR
tier around a source the school might not have.

---

## 6. What has not been done

⚠️ **Recorded explicitly, because this document could otherwise be misread as
describing a working ingestion system.**

| | Status |
|---|---|
| A school timetable seen | ⛔ **Never.** Not one file, not one page |
| A format confirmed | ⛔ **No.** All seven categories above are hypothetical for this school |
| The authoritative source identified | ⛔ **No.** `CL-TIME-001` `UNANSWERED` |
| The approver identified | ⛔ **No.** `CL-SUB-001` `UNANSWERED` |
| Any document ingested | ⛔ **No** |
| Any parser written or run | ⛔ **No.** Tier 1 is *described*, not implemented |
| Any OCR or model run | ⛔ **No.** No AI service has been contacted; `C2` prohibits it without written authorisation |
| Any document photographed | ⛔ **No.** Prohibited by `C24` and gated by `CL-SUB-006` |
| Any test fixture built from a real school document | ⛔ **No** |

⚠️ **Every format assessment in §2 rests on the format's documented structure**,
✅ `VERIFIED` — and on ❓ **`UNKNOWN`** as to whether this school uses that format.

---

## 7. The cheapest test, and why it is not technical

⚠️ **The cheapest thing that would resolve most of this document is not a parser.**

> **Ask for the current master timetable, in whatever form it is actually kept**,
> and ask who last edited it. Ten minutes. 📄

That single conversation would answer `CL-TIME-001`, `CL-TIME-002`, `CL-TIME-003`
and most of `CL-TIME-005`–`-008` — and would let a parser be chosen against a real
format instead of against a plausible one.

⚠️ **Until then, any Tier 1 parser we write is written against a guess.** A parser
built for a format the school does not use is not reusable; it is a rewrite
waiting to happen, and it will look like progress right up until it is discarded.

---

## 8. Related documents

| Document | Covers |
|---|---|
| [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) | ⭐ **The pipeline** — parsing tiers, the record, validation `V-01`…`V-15`, states, quarantine, corrections, audit, who approves |
| [`../integrations/identity_provider.md`](../integrations/identity_provider.md) | Who the approver is — the other half of the publication path |
| [`../architecture/data_model.md`](../architecture/data_model.md) | The entities these inputs become — design only |
| [`../requirements.md`](../requirements.md) | `FR-TIME-11`, `FR-TIME-12`, `FR-SUB-07`, `FR-SUB-08` |
| [`../open_questions_register.md`](../open_questions_register.md) | `CL-TIME-001` as blocking question `OQ-12` |
| [`../decision_log.md`](../decision_log.md) | `D-006` deterministic precedes AI · `D-104` OCR scope, **OPEN** |
| [`../sources.md`](../sources.md) | Bibliography, including the Docling and Microsoft documentation cited above |
| [`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md) | 576 questions. **0 answered** |