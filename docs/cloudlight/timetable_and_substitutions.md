# CloudLight — Timetable and Substitution Ingestion Pipeline

**Phase 0 · Documentation only · 2026-10-11**

**Status: PROPOSED. Nothing has been built.** No parser has been written, no
document ingested, no timetable displayed, no screen mounted. The school has a
regular timetable and *may* provide a substitution sheet; the format, frequency,
authorship and approval route of both are unknown to us. Everything below is a
proposed design and a list of questions, not a description of a running system.

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in a primary source — an official standard, vendor documentation, or a cited paper. Quoted or directly derived, with a URL in §Sources |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | Published by a project or vendor; we have not measured it on the school's own devices, network, or documents |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | Our engineering judgement. No source. Must be measured before it is relied on |
| ❓ **UNKNOWN** | No evidence found. Not assumed in either direction. Open questions are cross-referenced to the `CL-*` gates |
| ⛔ **NOT RECOMMENDED WITHOUT APPROVAL** | A design we will not implement without an explicit, recorded school decision |

**Companion documents:** [`../architecture/data_flow.md`](../architecture/data_flow.md)
· `docs/architecture/delivery_state_machine.md` (SHANA delivery state machine)
· [`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)
— the last of which holds every open gate referenced here.

---

## 1. The governing principle

> **The authoritative source of timetable and substitution data is defined by
> school policy, not by this document, and not by whatever CloudLight can most
> easily read.**

CloudLight is a **consumer and renderer** of an authoritative record. It does not
become authoritative by being installed, by being on the school network, or by
being the only system displaying the information. If the school's written
procedure says the timetable of record lives in a register kept by a named member
of staff, then that register is the source — not a scan of it, not a spreadsheet
someone exported from it last Tuesday, and not the screen on the wall.

| Situation | Authoritative? | Why |
|---|---|---|
| A photograph of a classroom board | ⛔ **No** | It is visible, not approved. Visibility is not authority |
| An AI extraction at 99% confidence | ⛔ **No** | Confidence is a property of the model, not of the school |
| An AI extraction approved by a named person | 🟡 Candidate | The authority is the person; the model only did the typing |
| A hand-keyed record approved by a named person | 🟡 Candidate | Same. Provenance differs, authority does not |
| A record on a display | — | **Only after human approval, and only with the approval attributed** |

⚠️ **Only a human-approved record publishes to a display.** This is the rule the
rest of this document elaborates, and it is not weakened by a confidence
threshold, a time pressure, or a convenience argument. ⚠️ **What this document
must not do is quietly become the policy.** If CloudLight cannot determine which
source the school considers authoritative, that is a question to ask the school
(`CL-SUB-001`, `CL-SUB-005`), not a gap to fill by defaulting to whatever arrived
by email most recently. We have not invented a timetable department, a timetable
format, or a signing procedure, because we have not been told any of them exist.
❓ **UNKNOWN.**

---

## 2. Pipeline overview

```
  [Source document]  →  Ingest (record hash, provenance, timestamp)
                          ↓
                    Normalise / structural parse
                          ↓
                    Schema validation  ──fail──►  QUARANTINE (never published)
                          ↓ pass
                    Cross-check against existing timetable
                          ↓
                    (OCR + layout + LLM extraction)  only for unstructured input
                          ↓
                    Confidence scoring + conflict detection
                          ↓
                    HUMAN REVIEW  ──reject──►  QUARANTINE
                          ↓ approve (recorded, attributed)
                    Versioned record  →  Publish to specific displays
                          ↓
                    Displays  (with last-synced timestamp)
```

**What the diagram asserts, in order:**

1. **Ingest is provenance-first.** A document is hashed and attributed *before* it
   is parsed. An un-attributable source can never be approved — there is nobody to
   attribute the approval to.
2. **Structural parsing precedes model invocation.** The model is reached only for
   input that has already failed to yield a structure deterministically.
3. **Validation can only reject, never repair.** A rule that finds a defect sends
   the record to quarantine; it does not correct the record.
4. **Cross-check precedes extraction.** Contradictions are caught *before* an
   expensive, uncertain step runs.
5. **There is exactly one approval gate and it is human.** Everything upstream is
   a proposal; the approval is an attributed, timestamped, reversible act.
6. **Quarantine is a real destination, not an error message.** Rejected records
   stay retained, inspectable, and attributable — just never shown as fact.

⛔ **Quarantine is not a bin.** A quarantined record must still be resolvable by a
person: which document it came from, which rule rejected it, and what it said.
Silently dropping it is indistinguishable from losing a substitution.

⚠️ **Publication is targeted, never broadcast.** `CL-SUB-022` asks whether
substitutions should reach all displays, only affected rooms, or staff phones. The
blast radius of one mistaken approval is a school design decision. Until it is
answered, the safe default is the smallest possible scope — the rooms the change
actually affects (`CL-SUB-022`).

---

## 3. Input formats and their handling

The formats below are the **categories** of input, not a claim about what this
school uses. ❓ **UNKNOWN** which of them will actually arrive (`CL-AIML-006`,
`CL-SUB-005`). Each category is a separate path with a different failure mode and
a different confidence behaviour.

| Format | Structure | Parser tier | AI required? | Typical failure mode |
|---|---|---|---|---|
| **Excel `.xlsx`** | A ZIP package of XML parts; cells addressed by row/column, styles carry meaning | **1 — deterministic** | **No** | Merged cells spanning a header block; a second header row partway down the sheet; meaning carried by fill colour or bold rather than by text; hidden rows that contradict visible ones |
| **CSV** | Flat delimited text | **1 — deterministic** | **No** | Title and date lines *above* the real header row; a byte-order mark prepended to the first field name; a comma inside a name inside an unquoted field; blank rows used as period separators |
| **Google Sheets export** | Produces `.xlsx` or `.csv`; a **point-in-time snapshot**, not a live link | **1 — deterministic** | **No** | The snapshot silently diverges from the live sheet; formulas export as their last computed value, so the *derivation* is lost; two people export minutes apart and get two different files |
| **PDF, digital (text layer)** | Text tokens with page coordinates; no rendering required to read it | **1 — deterministic** (+ tier 2 only to recover a borderless grid) | **No** | Column order reconstructed from coordinates is genuinely ambiguous in a table with no ruling lines; the header repeats on each page and is read twice; a font encoding quirk turns a character into a substitute |
| **PDF, scanned (image only)** | No text tokens exist. The page is an image | **2 → 3 → 4** | **Yes (OCR)** | Skew, glare, shadow across the sheet, faded toner, a crease through a cell; the handwritten note in the margin is read as part of the grid |
| **Photographed / handwritten board** | Image only, no fixed grid, handwriting, free placement | **3 → 4** | **Yes (OCR + model)** | Everything above, plus: reflection, marker strokes crossing a cell, an entry overwritten rather than erased, the board partly out of frame, and — worst — **the board changed after the photograph was taken** |
| **Plain typed form in CloudLight** | A form with controlled fields and pick-lists | **1 — deterministic** | **No** | Autocomplete silently substituting a near-miss name; a transposition; a person keying what they assume rather than what they were told. Mitigated by closed vocabularies and by rejecting anything not on the list |

⚠️ **Deterministic parsing first, without exception.** If the source has a stable
cell structure, read it with a real spreadsheet or CSV parser and **do not
involve a model**. A model reading an `.xlsx` cannot do better than the cell
addresses it was handed, is slower, is non-deterministic, costs money, works
online, and cannot be reasoned about. AI is for genuinely unstructured input and
for nothing else.

### 3.1 The verifiable facts behind that rule

**✅ VERIFIED — an `.xlsx` is a ZIP archive of XML parts.** Office Open XML
documents are Open Packaging Conventions packages: you can rename a `.docx` to
`.zip` and open it. Microsoft's own documentation states that *"To view the XML
code in a document, explore the contents of the .zip package… rename the extension
on the minimum document from .docx to .zip"*; the standard is ISO/IEC 29500.
Consequence: the cell contents, their row/column addresses, and their formatting
are all **addressable without inference**. Reading them is a lookup, not a
guess.

**✅ VERIFIED — a text-layer PDF can be read directly; an image-only PDF cannot.**
Docling's pipeline description states that each document *"is first parsed by a
PDF backend, which retrieves the programmatic text tokens, consisting of string
content and its coordinates on the page"*. Text-layer pages therefore yield real
text tokens; a scanned page yields none, which is why Docling lists OCR as
**optional**: *"Optionally applies OCR, e.g. for scanned PDFs."*

**✅ VERIFIED — the relative cost of the three model stages.** The Docling report
gives per-stage CPU costs directly:

| Stage | What the report says | Cost |
|---|---|---|
| **Layout detection** | Page images *"at 72 dpi resolution, which can be processed on a single CPU with sub-second latency"* | **< 1 s/page, single CPU** |
| **Table structure (TableFormer)** | *"Typical tables require between 2 and 6 seconds to be processed on a standard CPU, strongly dependent on the amount of included table cells"* | **2–6 s/table, CPU** |
| **OCR (EasyOCR, 216 dpi)** | *"we observe that it runs fairly slow on CPU (upwards of 30 seconds per page)"* | **> 30 s/page, CPU** |

**OCR is therefore the dominant expense by a wide margin** — one to two orders of
magnitude above layout detection. The report's 225-page benchmark (3 arXiv papers
+ 2 IBM Redbooks) makes the point structurally: **the headline Table 1 is
explicitly run with OCR disabled.** At the default 4-thread budget an Apple M3 Max
processed 225 pages in 177 s (1.27 pages/s, peak RSS 6.20 GB) and an Intel Xeon
E5-2690 in 375 s (0.60 pages/s, 6.16 GB); the lighter `pypdfium` backend reached 2.18
and 0.94 pages/s.

> **Design consequence, and the reason the pipeline is shaped this way: OCR runs
> only on pages that have no text layer.** Running it "just in case" on a clean
> export would multiply the cost of the common case by an order of magnitude to
> obtain, at best, the same characters the parser already had — and, at worst, a
> different set of them.

⚠️ **The honest limits of those numbers, stated plainly:**

- 🟡 They are **Docling's figures on Docling's corpus** — papers and manuals, not
  school substitution sheets. They justify a *ratio* between stages. They do
  **not** predict accuracy or wall-clock on a timetable.
- 🟡 **No GPU benchmark exists in the report.** The authors state that
  *"establishing GPU acceleration support for the AI models is currently
  work-in-progress and largely untested"*. We therefore have **no published
  CPU-vs-GPU figure to cite**, and ❓ **UNKNOWN** whether GPU acceleration helps
  here at all. We are not going to invent one; any claim that CloudLight "runs OCR
  on GPU" is 🧪 **PLAUSIBLE, REQUIRES TESTING** until measured on the school's own
  hardware.
- 🟡 `pypdfium` was *"faster and more memory efficient"* but at *"the expense of
  worse quality results, especially in table structure recovery"* — a documented
  speed/accuracy trade, and a reminder that the cheapest parser is not always the
  right one.

⚠️ **The training distribution is not school timetables.** Docling's layout model
is *"derived from RT-DETR and re-trained on DocLayNet"* — **80,863 human-annotated
pages** from finance, science, patents, tenders, law and manuals, with 11 layout
classes (Caption, Footnote, Formula, List-item, Page-footer, Page-header, Picture,
Section-header, Table, Text, Title). A school timetable is not in that
distribution. 🧪 Transfer to school layout is
**PLAUSIBLE, REQUIRES TESTING**, not established.

---

## 4. Parsing tiers, in order

Tiers are attempted **in sequence**. A later tier runs only when an earlier one
has failed to produce a complete, structurally sound result. A tier may not be
skipped, and no tier may add a value the source did not contain.

### Tier 1 — Deterministic structural parsing

Spreadsheet cells, CSV rows, and PDF text tokens with their coordinates. **Zero
model involvement.** Always attempted first, on every input, without exception. It
is fast, free, offline, exact, and — critically — **repeatable**: the same document
yields byte-identical output on every run, on every machine. Every property the
review gate depends on follows from this. 🟡 **DOCUMENTED, NOT VERIFIED ON OUR
HARDWARE** for the specific libraries we have not yet chosen or measured; the
*principle* is ✅ **VERIFIED** by the format definitions above.

⛔ Tier 1 may read cells, delimiters, and text-token coordinates, strip a
byte-order mark, and normalise whitespace. It may not infer meaning from colour,
bold, or position alone; it may not decide that a blank cell means "no change";
and it may not guess a header row that is not present. When it finds no
structure, it says so and escalates.

### Tier 2 — Layout analysis

Determines page structure: where the table is, which row is the header, where
columns begin and end, which cells are merged, and how emphasis (bold, colour,
underline) is used. This is where a borderless grid stops being ambiguous.

Docling's layout model is an **object detector that predicts the bounding boxes
and classes of page elements**, *"derived from RT-DETR and re-trained on
DocLayNet"* — a **labelled document-layout corpus** of human-annotated pages.
TableFormer, the table-structure stage, is a vision-transformer trained to
*"predict the logical row and column structure of a given table… and determine
which table cells belong to column headers, row headers or the table body"*, and
is documented as handling *"partial or no borderlines, empty cells, rows or
columns, cell spans and hierarchy."*

🧪 **We are not attaching a benchmark number to school layout analysis**, because
none of the cited work reports accuracy on school substitution sheets. We describe
the *approach* — detection trained on a labelled layout corpus, then a structure
model that tolerates borderless grids — and mark the performance **PLAUSIBLE,
REQUIRES TESTING**.

⚠️ **Layout is where "everyone just knows the layout" becomes a blocker.**
`CL-AIML-007` asks whether the school's timetables use a grid at all or rely on
colour coding, bold, abbreviations and arrows; `CL-AIML-008` asks whether the
headers are written out. A sheet with **unlabelled headers cannot be extracted
reliably by anyone** — human or model — and must be manually configured. Colour is
invisible to plain OCR and needs a layout-aware model.

### Tier 3 — OCR

**Only for image-only or handwritten input.** Never runs on a text-layer document.

| Property | Status |
|---|---|
| Printed text recognition | ✅ **VERIFIED** — a mature, well-characterised capability |
| **Handwritten** text recognition | ⚠️ **Materially lower accuracy than print.** `CL-AIML-009` is a P0 gate precisely because this gap exists and because nobody has measured it on this school's documents |
| Language / script coverage | ⛔ **A hard feasibility limit, not a quality setting.** The OCR engine referenced in the Docling report is documented as *"Ready-to-use ocr with 80+ supported languages"*. A script outside that set is **not read at lower quality — it is not read.** `CL-AIML-010` asks which scripts appear |
| Cost | 🟡 > 30 s per page on CPU, per the report. The dominant expense in the whole chain |

> ⚠️ **Lower handwriting accuracy is a reason to require human review, not a
> reason to automate harder.** A whiteboard photograph is the *least* reliable
> input in the system and the one most likely to be mistaken for trustworthy,
> because it looks like a real, authoritative artefact. That appearance is exactly
> the trap. See §8.

⛔ **Abbreviation and short-code expansion is not a model problem.** If the school
uses codes, they need a lookup table the school itself validates
(`CL-AIML-011`). An unresolved code a model "helpfully" expands into a
similar-looking staff name is a silent misassignment — a teacher sent to the wrong
class. **Unresolved codes become review items, never guesses.**

### Tier 4 — LLM-assisted field extraction

A model maps ambiguous extracted content onto a **strict JSON schema**. Never
free text. Never prose, never a sentence, never a best-effort narrative.

```jsonc
// The only shape a model is ever asked to produce.
{
  "academic_year": "string | null",
  "class_section":  "string | null",
  "day_or_date":    "string | null",
  "period_index":   "integer | null",
  "subject":        "string | null",
  "room":           "string | null",
  "state":          "scheduled | substituted | cancelled | room_changed | null",
  "original_teacher":  "string | null",
  "replacement_teacher": "string | null"
}
```

⛔ **Four rules apply to every value in that object:**

- **Never free-text.** No field accepts a sentence.
- **Never guess.** A field the source does not contain is `null` — not the most
  plausible value, not the nearest name in the staff list, not the teacher's usual
  partner. `CL-AIML-013` asks what the school wants when the system is not
  confident; we design for **flag and hold**.
- **Never resolve.** No staff-list lookup, no fuzzy spellings, no choosing between
  two similar names. Resolution is deterministic and belongs to tiers 1 and 5.
- **Never write to the timetable.** The model produces a proposal. Tier 5 and a
  human decide.

### Tier 5 — Deterministic validation

The last gate, and **the only one permitted to reject**. Same input, same output,
every run. The rules are in §5. A rule that cannot be written as a deterministic
check does not belong in this tier and must not be smuggled in as an "AI judgement
call" — it is either a human decision or it is not made. ⚠️ **Tier 5 has no repair
action.** It rejects. It does not fill in, round, default, or "reasonably
interpret". Auto-correction of a timetable is how a wrong teacher becomes a real
one.

---

## 5. Schema, validation and conflict detection

### 5.1 The record

One versioned record per lesson-slot event. Fields marked *required* have no
default and no `null` in a published record.

| Field | Type | Required | Notes |
|---|---|---|---|
| `academic_year` | string | ● | Which year this record belongs to. Records outside it are rejected |
| `class_section` | string | ● | Must exist in the school's class list |
| `day_or_date` | string | ● | Weekday for a recurring slot, calendar date for a dated substitution |
| `period_index` | integer | ● | Must fall inside the school's day model |
| `subject` | string | ● | |
| `scheduled_teacher` | string | ● | From the published master timetable. **Never overridden silently** (§6) |
| `room` | string | ● | Must exist in the room registry |
| `state` | enum | ● | `scheduled` · `substituted` · `cancelled` · `room_changed` |
| `original_teacher` | string | ● when `state = substituted` | Carried for audit and dispute |
| `replacement_teacher` | string | ● when `state = substituted` or `room_changed` | ⛔ Never auto-selected (`CL-SUB-012`) |
| `source_document_ref` | object | ● | Hash, filename, origin, received-at |
| `approved_by` | identity | ● | The named human. Never a shared account |
| `approved_at` | timestamp | ● | |
| `version` | integer | ● | Monotonic. Increments on every supersession (§7) |
| `confidence` | object | ○ | Per-field extraction confidence. **Never a publication criterion** |

⚠️ **`confidence` is an advisory to the reviewer, not a gate.** A record at 0.99
and a record at 0.30 are both *proposals*. Publishing one and not the other on
confidence grounds would make the model's opinion the school policy — which §1
forbids.

### 5.2 Validation rules

These reject. Nothing here repairs a record.

| Rule | Rejects | Severity | Auto-fixable? |
|---|---|---|---|
| **V-01** Unknown class | A record naming a class not in the school's list | **Blocking** | ❌ No — never invent a class |
| **V-02** Unknown teacher | A replacement teacher not on the staff list | **Blocking** | ❌ No — never fuzzy-match a name |
| **V-03** Unknown room | A room not in the room registry | **Blocking** | ❌ No |
| **V-04** Period out of range | `period_index` outside the school's day model | **Blocking** | ❌ No — the day model is the school's |
| **V-05** Teacher double-booked | Overlapping assignment for one teacher in the same time range | **Blocking** | ❌ No — reassignment is a staffing decision |
| **V-06** Room double-booked | Overlapping assignment for one room in the same time range | **Blocking** | ❌ No |
| **V-07** Inverted times | End time before or equal to start time | **Blocking** | ❌ No |
| **V-08** Missing required field | Any field marked required above is absent or null | **Blocking** | ❌ No — an absent value stays absent |
| **V-09** Date outside academic year | A date before the year starts or after it ends | **Blocking** | ❌ No |
| **V-10** Substitution with no lesson | A substitution for a slot with no scheduled lesson | **Conflict** | ❌ No |
| **V-11** Replacement already allocated | A replacement teacher already booked elsewhere in the same period | **Conflict** | ❌ No — `CL-SUB-011` |
| **V-12** Duplicate substitution | Two substitutions for the same slot in the same period | **Conflict** | ⛔ **Never last-write-wins** |
| **V-13** Unresolved code | An abbreviation with no entry in the validated lookup table | **Conflict** | ❌ No (`CL-AIML-011`) |
| **V-14** Superseded version | A record whose `version` is lower than the current published version | **Rejected as stale** | ❌ No — §7 governs |
| **V-15** State/field mismatch | `state = substituted` with no replacement, or `cancelled` with a replacement | **Blocking** | ❌ No |

⚠️ **Every blocking rule is a `HARD REJECT`, not a warning** — the system has no
"publish anyway" path for one. `CL-AIML-013` asks whether a reviewer may accept a
low-confidence record; the answer here is that **reviewers may accept a proposal,
never a rule violation.** If the school later waives that, the waiver is recorded
as a school decision, not smuggled in as a default.

⚠️ **V-12 is the rule where the obvious implementation is catastrophic.** If two
sheets substitute the same slot, last-write-wins means the file ingested last
silently wins — and no ordering makes that correct. Two conflicting instructions
are **an event for a human to resolve** (`CL-SUB-011`); the slot shows an explicit
unknown state until they do.

### 5.3 Cross-document reconciliation

Before a proposed record can reach review, it is checked against the timetable
already published:

| Check | Question | Outcome |
|---|---|---|
| **R-1** Does a scheduled lesson exist for this class/period? | The substitution must match an existing scheduled lesson | No match → **conflict** (V-10). A substitution for a never-timetabled slot is a data error, not a substitution |
| **R-2** Is the replacement free in this period? | Has this teacher been allocated elsewhere at this time | Allocated → **conflict** (V-11). The most common real substitution error |
| **R-3** Is anyone else already substituting this slot? | Does a second document claim the same slot | Yes → **conflict** (V-12). Human resolves. Never automatic |
| **R-4** Is the room free and does it suit the subject? | Can the class be in that room for this period | Room occupied → **conflict**. Subject/room mismatch → review flag (`CL-SUB-016`) |
| **R-5** Is this after the substitution was already applied? | A late sheet re-stating something already published | Deduplicated, both versions retained, no conflict |
| **R-6** Is the timing still in the future? | A change arriving for a period already in progress | `CL-SUB-019` — mid-lesson effect is **UNKNOWN**, never assumed retroactive |

⚠️ **Reconciliation compares against a known-good baseline; it never guesses.** A
conflict does not resolve itself by picking the more recent file. It escalates.

---

## 6. The three substitution states — and why they must never be conflated

Three different facts, with three different sources, three different confidence
levels, and three different display treatments. Collapsing any two of them is how
a wrong teacher ends up on a wall.

| State | Where it comes from | Authority | May be displayed as a teacher? |
|---|---|---|---|
| **Scheduled teacher** | The published master timetable for the academic year | The timetable of record | **Yes** — the baseline the school already published |
| **Proposed substitution** | Extracted from a document, or typed and not yet approved | **None.** A suggestion from a parser or a person at a keyboard | ⛔ **No. Never.** Must be visibly marked as unconfirmed |
| **Confirmed substitute** | Approved by the named school authority for that change | The named approver | **Yes** — the only substitution that may appear as an actual teacher |

> ⛔ **The UI must never display a proposed substitution as a confirmed teacher.**
> This is not a warning about wording. A display has no room for nuance: a name on
> a screen in a classroom **is** the decision, to everyone in the room. The only
> meaningful distinction is between a name that is displayed and a name that is not.

> ⛔ **The UI must never invent a teacher when data is missing or ambiguous — it
> must show an explicit unknown state.** "Mr. —", "Sub: TBC", "To be confirmed", or
> a distinct visual marker. An unknown state is honest, actionable, and disappears
> the moment someone approves the substitution. A plausible wrong name is
> neither, and it puts a real teacher in a class they are not timetabled to teach.

⚠️ **The scheduled teacher is never overridden silently.** The replacement field
is *additional to* the scheduled teacher, not a replacement for it in the record.
The scheduled teacher is retained, needed for dispute resolution, and kept in
`original_teacher` precisely so that "who was supposed to be there" remains
answerable after the fact.

⚠️ **`cancelled` is a distinct state, not a substitution.** A dropped period —
training, sports, assembly — suppresses the lesson and assigns no one
(`CL-SUB-017`); its display wording and its validation both differ. Conflating
"cancelled" with "substituted by nobody" produces a screen that reads like an
error. **`room_changed` is a third shape** (`CL-SUB-016`): it changes no teacher
and validates against the room registry and the teacher's other allocations. ⚠️
**Mid-lesson application is unresolved.** `CL-SUB-019` asks whether a substitution
takes effect from the moment it is issued or only from the next period. We do
**not** assume, and we do not implement a mid-lesson display change by default.
❓ **UNKNOWN** until the school answers.

---

## 7. Corrections, supersession and rollback

### 7.1 Append-only versioning

> **A correction creates a NEW version that supersedes an old one. It never edits
> a published record in place.**

An in-place edit destroys the evidence that a change happened. If a period
displays the wrong teacher and is corrected, the only questions that matter later
are *"who published the wrong one, and when?"* and *"what did the screen say at
14:20?"* An in-place update answers neither.

```
  v1  scheduled                    approved 07:40  ← retained, never rewritten
  v2  substituted  (repl: A)        approved 08:05  ← supersedes v1
  v3  cancelled                    approved 09:12  ← supersedes v2
        │
        └─ every version keeps: its own hash, its own approver, its own timestamp,
           and the version it superseded
```

⚠️ **The current published record is a pointer, not a copy.** Displays resolve the
pointer; rolling back moves it; no row is ever updated. **Correct** creates a new
version and supersedes the previous. **Withdraw** marks a new version withdrawn
and leaves the previous reachable. **Roll back** moves the pointer *and* records a
new version saying why — "reverted because the wrong substitute was named" is
itself a record, and a rollback that erases the rollback is an unrepeatable favour.
⛔ **Delete is not available.** Retention removes by age, not by hand.

### 7.2 Withdrawal of a bad publication

Every display carries a last-synced timestamp (`CL-OTA-008` names timetable feed
freshness as a health check). Withdrawal must be complete and bounded:

| Property | Requirement |
|---|---|
| **Propagation window** | ❓ **UNKNOWN** — `CL-SUB-023` and `CL-OTA-007` ask the school for an acceptable rollback time. A number must come from the school, not from us |
| **Target** | The **same display scope** the bad publication reached, plus any display that received it since. A partial withdrawal is worse than none |
| **Visibility** | Recorded against the version, so a later dispute can see the data was wrong and when it was withdrawn |
| **Custody** | Whoever can publish can withdraw. If separation of duties is in force, withdrawal follows the same rule as publication (`CL-ADMIN-003`) |

⚠️ **Rollback must be a tested procedure, not an intention.** "We can roll back"
is a claim about a design; "the rollback was executed at 09:14 and every display
reconciled within the stated window" is a claim about a system. Until it has been
rehearsed, the honest label is 🧪 **PLAUSIBLE, REQUIRES TESTING**, and the school
should be told so. ⚠️ **A display that has lost contact must not be assumed to have
withdrawn** — it holds the bad version until it reconnects. This is why the
withdrawal record carries the same version identity as the publication: a display
reconciling against version N cannot silently keep showing version N-1.

### 7.3 Data and code need separate promotion paths

> **An urgent timetable correction must not be blocked behind the software
> release approval process.**

A room swapped for tomorrow is an operational fact about tomorrow. Routing it
through engineering release discipline — code review, staging, a deploy window, a
signed artefact — would delay it by hours or days for no safety benefit. Using one
pipeline for both "change a room number" and "ship a new parser" makes one far too
slow and the other far too casual.

| Artefact | Promotion path | Approver | Open gate |
|---|---|---|---|
| **Timetable / substitution data** | Its own path, independent of code | Whoever holds timetable authority (`CL-SUB-001`) | `CL-OTA-019`, `CL-OTA-020` |
| **Configuration** | Signed configuration data, not a software install | IT | `CL-OTA-011`, `CL-OTA-020` |
| **Software / parser changes** | Full deployment approval | Deployment authority (`CL-OTA-003`) | `CL-OTA-004`, `CL-OTA-006` |

⚠️ **The cost of this split is real and must be stated to the school.** Separate
promotion paths mean the timetable path must be at least as well controlled as the
code path — signed, attributed, logged, reversible — or it becomes the weak door.
Separating them is not a licence to do data updates carelessly; it is a
requirement to control them just as carefully, by a different and faster route.
⛔ **Data promotion must never become a backdoor to code promotion.** A signed
configuration bundle must not be able to carry executable code. That is the whole
of `CL-OTA-001` and the reason it is P0.

---

## 8. Photographing classroom boards — the permission gate

> **If the school authorises photographing a whiteboard or substitution board,
> the image is an input to a *proposal*, never an authoritative record, and a
> human must confirm it.**

This section is written to be quotable to a parent or a governor. ⚠️ **A
photographed board looks authoritative because it is a real artefact someone wrote
by hand.** That is precisely the failure mode. The chain of custody of a whiteboard
is: a person writes it, a camera sees it, a model reads it, somebody approves it, a
screen shows it — at least four of those five steps are *after* the moment the
information was true. The board is a snapshot of a conversation, not a record of a
decision.

| Question | Status |
|---|---|
| Is photographing the substitution board permitted at all? | ⛔ **`CL-SUB-006` — UNANSWERED. Without this permission the entire camera path is blocked regardless of technical feasibility** |
| Is a photograph treated as authoritative, or must a person confirm it? | ⛔ **`CL-SUB-007` — UNANSWERED.** Our design assumes **confirmation is mandatory**, because treating OCR output as authoritative removes every safety margin this section assumes |
| Whose permission, and covering what exactly — board only, or the room? | ⛔ **`CL-SUB-006`** |
| What device, and who holds it? | ❓ `CL-SUB-008` — UNANSWERED |

**If the photograph path is authorised, these are non-negotiable:**

- **Consent is required and it is specific.** Permission to photograph a timetable
  board is not permission to photograph a corridor, a staff room, a classroom
  during a lesson, or a person. `CL-SUB-006` must be answered in a scope narrow
  enough to photograph one board.
- **A photograph must never contain a child.** If the frame can contain a student,
  a name, a photograph, or any identifiable pupil information, the frame is out of
  scope. ⛔ `CL-AIML-004` asks whether these documents contain children's names,
  photographs, or identifiable data — **an unanswered P0**, and a "yes" changes the
  consent, retention, and access rules for every captured image and every derived
  record.
- **Retention of captured images is a school decision.** ⛔ `CL-AIML-005` asks
  whether images must be deleted after confirmation or archived, and for how long.
  We propose **no default**. An image is not consumed by being read; it is a
  personal-data artefact on a disk, carrying whatever obligation its content
  carries.
- **Images are not a training set.** ⛔ Captured images are **not** used to train,
  fine-tune, evaluate, or improve any model, in this project or any other. A
  design commitment, not an open question.
- **Third-party processing is a separate permission.** `CL-AIML-003` asks whether
  document images may be processed by a third-party AI service or must stay on
  school infrastructure. "Yes to photographing, no to cloud processing" answers
  two different questions and both must be honoured. 🟡 Until answered, the
  assumption is **on-premises processing only**.

⛔ **This path is optional and may be refused entirely — with no loss of core
function.** CloudLight must work completely with a typed form and a file import
(`CL-AIML-020`, `CL-SUB-005`). If the school refuses photographic capture the
product is undiminished; the board simply becomes something a person reads and
types. **That is a design requirement, not a degradation.** ⚠️ **The permission
gate precedes the technology.** There is no version of this system in which a
camera exists in a school before `CL-SUB-006` and `CL-SUB-007` have written
answers from a named authority. A camera that exists without that answer has been
installed without permission, regardless of what the software promises.

---

## 9. Audit trail and source-document retention

### 9.1 What must be retained, and for how long

| Artefact | Retained | Why |
|---|---|---|
| **Original document hash** | ● Permanently for the life of the record | The only way to prove which version of which file produced a record |
| **Original filename, origin, received-at** | ● | "Where did this come from" is the first question in every dispute |
| **Extracted values** | ● As-extracted, unchanged, per field | §9.2 |
| **Human-corrected values** | ● | §9.2 |
| **Who extracted** | ● | System identity if machine; a named person if typed |
| **Who approved, when** | ● | The attribution that makes the record publishable |
| **Prior version** | ● Superseded, never overwritten | §7 |
| **Confidence per field** | ○ Advisory | Trending extraction quality |
| **Captured board images** | ⛔ **Unresolved — `CL-AIML-005`** | See §8. Not our decision to make |
| **Extracted staff and class names** | ● As part of the record | Personal data — see below |

⚠️ **Retention periods are a school policy decision, not a technical one**, and are
deliberately left as `TBD` here rather than guessed. ❓ **UNKNOWN** pending the
school's answer.

### 9.2 Extracted versus human-corrected

Both are retained, side by side, forever:

```
  as_extracted     { "replacement_teacher": "R. Sharma",   confidence: 0.91 }
  as_approved      { "replacement_teacher": "R. Sharman"  }
  corrected_by     <named person>   at <timestamp>
```

> ⚠️ **Without this, it is impossible to distinguish "the AI misread it" from
> "the office decided wrongly."** Those are different failures with different
> owners and different fixes. If only the final value is kept, *"how often does
> our extractor get it wrong?"* is permanently unanswerable and the system cannot
> be improved — only trusted or abandoned. If the extractor systematically
> confuses two similar names, that is visible **only** if every human correction
> is logged against its extraction. `CL-AIML-018` and `CL-SUB-020` ask whether the
> school requires this; disputes over "the screen said otherwise" cannot be
> resolved after the fact without it.

### 9.3 Source documents contain personal data

⚠️ **A substitution sheet names real teachers, real classes, and sometimes real
students' names, absences, and medical-adjacent arrangements.** The source document
is personal data regardless of its format, and every captured image and retained
record inherits that. Access to source documents must be permissioned separately
from access to the timetable itself (`CL-ADMIN-002`): a role that can view today's
periods should not, by default, browse two years of who was absent and why.

⛔ **No source document, extracted record, or captured image may leave the school
boundary** without an explicit written answer to `CL-AIML-003`. CloudLight is
local-first for this reason among others. ⚠️ **Audit-log access is itself a
privilege**: whoever can *edit* the log can rewrite history afterwards, which
defeats the purpose. The log should be append-only, and administrators should not
be the same people who can silently alter the record of what they did.

---

## 10. Who approves

### 10.1 The authority is a school decision

| Gate | Question | Status |
|---|---|---|
| **`CL-SUB-001`** | **Who currently decides a substitution** — the class teacher, the timetable clerk, a Vice Principal, a subject head, someone else? | ❓ **UNANSWERED.** CloudLight must route approval to the exact role that **already** holds the authority |
| **`CL-SUB-002`** | Does a substitution require counter-signature or a second confirmation before it is final? | ❓ **UNANSWERED.** A two-person rule changes the gate from one tap to a two-step |
| **`CL-SUB-003`** | Is it acceptable that CloudLight **never** publishes an unapproved substitution, even if that means a display is briefly out of date? | ❓ **UNANSWERED.** We design for "yes" (§11) |
| **`CL-ADMIN-003`** | Must **one person request and another publish**, or may a single authorised member of staff do both? | ❓ **UNANSWERED** |

⚠️ **We have deliberately not written the word "timetable department" into any
role definition here, because we have not been told one exists.** `CL-SUB-001`
asks who decides a substitution, offering class teacher, timetable clerk, Vice
Principal and subject head — because we do not know the answer. Inventing an org
chart to tidy a permission model would mean designing for a school that may not
be the one we get. ⚠️ **Approval authority is configured from the school's
answer, never defaulted.** An unconfigured approval role must mean *nothing can be
published*, not *anything can be published*. `CL-ADMIN-001` asks whether authority
belongs to a named person, a role, or a shared account; ⛔ **shared accounts are
excluded** — they destroy attribution in the audit log, which makes §9 useless
and makes `approved_by` a fiction.

### 10.2 Separation of duties

> **Separation of duties — requester ≠ publisher — is the cheapest misuse
> control available in this product.**

| Without it | With it |
|---|---|
| One account can propose, approve, and publish | Proposal and publication are distinct, attributed acts |
| A careless keystroke publishes to every classroom | A second person sees it first |
| A compromised account affects the whole school | A compromised account can propose; it still cannot publish |
| The audit log shows one name for everything | The audit log shows two distinct humans |

⚠️ **This is the control most likely to be waived without saying so** — a thinly
staffed school will find one-person operation faster, and nobody will write it
down. `CL-ADMIN-008` already records that this school has very few technical staff;
`CL-ADMIN-009` asks who acts when the principal or timetabler is absent and whether
that delegation is time-limited. Permanent informal delegation is exactly how two
conflicting timetable edits arrive at once from two people with equal authority.

> ⚠️ **CloudLight must support separation of duties even where the school later
> waives it.** It is a configurable policy, not a hard-coded assumption. If waived,
> the waiver is recorded, the configuration says single-approver mode, and the
> audit log notes that mode was in effect — so a later question ("was this checked
> by two people?") has an honest answer. ⚠️ **Break-glass access needs the same
> discipline** (`CL-ADMIN-010`): an emergency elevated account without review
> conditions becomes the ordinary way privileged work gets done.

---

## 11. Fail-closed design

> **Any failure in the chain results in the previous known-good version continuing
> to display, with an explicit staleness indicator and an alert to whoever holds
> timetable authority. CloudLight must never fail open into a guessed timetable.**

```

Any failure at any stage
  │
  ├── parser error          ─┐
  ├── validation reject     ─┤
  ├── conflict detected     ─┼──►  QUARANTINE   (retained, inspectable, never shown)
  ├── reviewer rejects      ─┤
  ├── low confidence        ─┤
  ├── source unavailable    ─┤
  ├── model unavailable     ─┤
  └── feed unreachable      ─┘
                                     │
                                     ▼
              Last known-good version KEEPS DISPLAYING
              + staleness indicator shown on the display
              + alert raised to the timetable authority
              + nothing is guessed, filled, or carried forward
```

| Failure | What the display shows | What the operator sees |
|---|---|---|
| No new document arrived today | Yesterday's approved timetable, **marked stale** | Alert: feed has not refreshed |
| A document arrived and failed validation | Yesterday's approved timetable, **marked stale** | The rejected record in quarantine, with the rule that rejected it |
| A substitution is proposed but not approved | Yesterday's approved timetable for that slot | Pending proposal visible **to the reviewer only**, never to a display |
| Conflict between two substitutions | The prior confirmed state for that slot | Conflict raised to a named human. Slot shows *unknown*, not a coin toss |
| Display loses contact with the server | Its cached version, **with its own last-synced timestamp visible** | Device flagged offline in the health view |
| The server is down | Cached version on every display | `CL-OTA-014` asks how long the school needs full offline operation. ❓ **UNKNOWN** |

⚠️ **A display showing something old must SAY SO.** An undated screen is worse
than no screen: it asserts a currency it does not have. Every display shows when
it last synchronised, and one past its staleness threshold shows that state
visibly rather than hiding it (`CL-OTA-008`, timetable feed freshness). ⚠️ **"Stale
but labelled" is acceptable; "fresh but wrong" is not.** `CL-SUB-003` asks whether
the school accepts that CloudLight never publishes an unapproved substitution even
if it means a display is briefly out of date; until answered, we design for the
delay. A teacher arriving at a screen reading *"last updated 08:05"* asks a
question; a teacher arriving at a screen naming an unapproved substitute trusts a
lie. ⛔ **CloudLight must never fail open into a guessed timetable.** There is no
code path that constructs a plausible value, fills a missing field, carries a
stale record forward as current, or picks between two conflicting instructions.
*Not knowing* is a first-class, displayable state — not a defect to be papered over
with a default.

---

## The invariant, restated

> **Only a named human can cause a timetable to change on a screen. Everything
> else — every parser, every model, every confidence score — is an assistant to
> that human's judgement, or it is nothing at all.**

---

## Sources

All sources accessed **2026-10-11**. Types follow the repository convention:
`STD` · `OFFICIAL-DOCS` · `PEER-REVIEWED` · `PREPRINT` (arXiv, not refereed) ·
`OFFICIAL-CODE` (project repository, code or data).

| # | Title | URL | Type | What it supports here |
|---|---|---|---|---|
| 1 | **Docling Technical Report** — Auer, Lysak, Nassar, Dolfi et al., IBM Research, arXiv:2408.09869v5 (9 Dec 2024) | `https://arxiv.org/abs/2408.09869` | PREPRINT | Pipeline order (§2, §4); PDF backend *"retrieves the programmatic text tokens, consisting of string content and its coordinates on the page"* (§3.1); OCR is **optional**, *"e.g. for scanned PDFs"* (§3.1); **layout detection at 72 dpi on a single CPU with sub-second latency**; **TableFormer 2–6 s per table on a standard CPU**; **EasyOCR *"upwards of 30 seconds per page"* on CPU (§3.1 cost table); the 225-page benchmark run **with OCR disabled**, with per-system throughput and peak RSS (§3.1); **GPU acceleration *"work-in-progress and largely untested"*** — hence ❓ **UNKNOWN** for the CPU-vs-GPU comparison (§3.1) |
| 2 | **TableFormer: Table Structure Understanding with Transformers** — Nassar, Livathinos, Lysak, Staar, CVPR 2022 (arXiv:2203.01017) | `https://arxiv.org/abs/2203.01017` | PEER-REVIEWED | The table-structure approach described in Tier 2: predicting logical row/column structure and header/body roles; documented tolerance of *"partial or no borderlines, empty cells, rows or columns, cell spans"* (§4, Tier 2) |
| 3 | **DocLayNet: A Large Human-Annotated Dataset for Document-Layout Analysis** — Pfitzmann, Auer, Dolfi, Nassar, Staar, KDD 2022 (arXiv:2206.01062) | `https://arxiv.org/abs/2206.01062` | PEER-REVIEWED | That the layout stage is an approach trained on a **labelled document-layout corpus** — 80,863 human-annotated pages, 11 layout classes (§4, Tier 2). Also the basis for noting the corpus is *not* school timetables |
| 4 | **DS4SD/DocLayNet dataset repository** | `https://github.com/DS4SD/DocLayNet` | OFFICIAL-CODE | Human annotation by trained experts; fixed train/test/validation splits; per-page bounding-box annotations and digital text cells with coordinates. Supports the claim in §3.1 that the layout training distribution is finance/science/patents/tenders/law/manuals, not school documents |
| 5 | **EasyOCR** — JaidedAI/EasyOCR (cited as ref. [1] in the Docling report) | `https://github.com/JaidedAI/EasyOCR` | OFFICIAL-CODE | *"Ready-to-use ocr with 80+ supported languages."* Supports the **script-coverage limit** in Tier 3 (§4): an unsupported script is not read at lower quality, it is not read. Backs `CL-AIML-010` |
| 6 | **Structure of a WordprocessingML document** — Microsoft Learn, Open XML SDK documentation | `https://learn.microsoft.com/en-us/office/open-xml/word/structure-of-a-wordprocessingml-document` | OFFICIAL-DOCS | ✅ That Office Open XML documents are **ZIP packages containing XML parts** — *"rename the extension on the minimum document from .docx to .zip"* (§3, `.xlsx` row; §3.1). Cell contents and addresses are therefore readable without inference |
| 7 | **ISO/IEC 29500** — Office Open XML File Formats | `https://www.iso.org/standard/71691.html` | STD | The underlying standard for the OOXML package format cited in source 6 (§3.1) |
| 8 | **ISO 32000-1 (PDF)** — published copy, Adobe Acrobat SDK documentation portal | `https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf` | STD | PDF document structure: page content streams carry text-showing operators and graphics; a scanned page is an **image**, with no character data to retrieve. Supports the text-layer vs image-only distinction in §3.1 |
| 9 | **RFC 4180 — Common Format and MIME Type for Comma-Separated Values (CSV) Files** | `https://www.rfc-editor.org/rfc/rfc4180` | STD | CSV quoting and escaping rules — a field containing a comma, a quote, or a line break must be quoted. Supports the CSV failure-mode row in §3 |
| 10 | **Create, view, or download a file** — Google Docs Editors Help | `https://support.google.com/docs/answer/49114` | OFFICIAL-DOCS | That a Google Sheets file is exported as a chosen **file type** — a point-in-time download, not a live link. Supports the Google Sheets export row in §3 and the snapshot-drift failure mode |

**Repository sources cited by gate ID:** every `CL-*` identifier referenced in
this document is defined in
[`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md).
**All of them are currently `UNANSWERED`.** This document is a design that
presumes those answers; it is not a record of answers received.

⚠️ **No school document was seen.** No timetable, substitution sheet, board
photograph, staff list, or class list was examined, because none has been provided.
No parser was run against a school document. No parser exists.