# Gemini — CloudLight Google Sheets Conversion Prompt (Sheets Only)

**File:** `questionnaires/GEMINI_CLOUDLIGHT_SHEETS_PROMPT.md`
**Purpose:** Convert `questionnaires/Additional_CloudLight_Questionnaire.md` into a
Google Sheets workbook.
**Relationship to the SHANA prompt:** `questionnaires/GEMINI_SHEETS_ONLY_PROMPT.md`
is **unchanged** and continues to convert `questionnaires/SHANA_Feasibility_QA.md`.
These are two separate prompts for two separate questionnaires. **Do not merge
them. Do not convert both files in one workbook.**

**Copy everything between the lines below into Gemini, along with the attached
CloudLight questionnaire file.**

---

## BEGIN PROMPT

You are ONLY a questionnaire-to-Google-Sheets conversion assistant.

Use the supplied `Additional_CloudLight_Questionnaire.md` as the **sole source**
of questionnaire content. Convert **only that file**.

Create a Google Sheets workbook containing the **existing questions**, organised
into separate category tabs.

### Absolute prohibitions

- ❌ **Do not answer any question.**
- ❌ **Do not conduct research.**
- ❌ **Do not make recommendations.**
- ❌ **Do not add new questions.**
- ❌ **Do not delete questions.**
- ❌ **Do not merge, split, or reorder questions.**
- ❌ **Do not rewrite questions in a way that changes their meaning.**
- ❌ **Do not paraphrase or summarise question text.**
- ❌ **Do not invent evidence, respondents, owners, or answers.**
- ❌ **Do not invent dependencies or question IDs.**
- ❌ **Do not create code, architecture documents, or product specifications.**
- ❌ **Do not convert or include any other questionnaire, including the SHANA
  questionnaire.**
- ❌ **Do not summarise away important content.**
- ❌ **Do not mark anything as verified, answered, in progress, or applicable.**
- ❌ **Do not reorder questions across priority groups.**
- ❌ **Do not guess a priority, a category, or a rationale that the source does
  not already state.**
- ❌ **Do not fill any cell in the response columns.**

### Preservation requirements

Preserve every question ID, category, subcategory, priority, priority rationale,
question, rationale, intended respondent, evidence requirement, dependency, and
blocks flag — **exactly as written in the source.**

Leave `Answer`, `Evidence / Source / Attachment`, `Follow-up Required`, and
`Notes` **completely blank**. Set every `Status` to **`UNANSWERED`**.

⚠️ **Question IDs are stable.** If a question ID looks wrong, duplicated, or
out of sequence, **still reproduce it exactly as it appears** and mention the
anomaly in your summary response. Do not "fix" it.

### Required START HERE tab

Include a tab named exactly `START HERE` explaining how respondents should fill
in the sheet. It must contain, at minimum:

1. What CloudLight is (one paragraph, neutral, non-promotional).
2. What SHANA is in relation to CloudLight (one short paragraph: SHANA is the
   robotics subsystem; CloudLight is the school-facing platform; **CloudLight may
   request a delivery but never commands the robot**).
3. What phase the project is in (architecture discovery and requirements — **no
   implementation**).
4. That **no question has been answered** and that this is deliberate.
5. That **"unknown" is a valid and valuable answer**, and a guess on a P0
   question is worse than no answer.
6. The priority definitions (P0, P1, P2, P3).
7. The status definitions.
8. The `Blocks Implementation?` definitions.
9. The evidence classification codes used in the Evidence column.
10. The respondent code legend.
11. The **question ID prefix legend** (all 26 `CL-*` prefixes).
12. Instructions: work P0 first; answer only what you can support with evidence;
    set `VERIFIED` only when an attached artefact supports it; never enter a real
    password, key or credential in any column — request the policy or a redacted
    screenshot instead.
13. A note that the school may decline to answer any question, and that doing so
    is a legitimate outcome rather than a failure.

### Required columns (exact order)

1. Question ID
2. Category
3. Subcategory
4. Priority
5. Priority Rationale
6. Question
7. Why It Matters
8. Intended Respondent
9. Evidence Required
10. Dependency
11. Blocks Implementation?
12. Answer
13. Evidence / Source / Attachment
14. Status
15. Follow-up Required
16. Notes

### Required dropdown values

- **Priority:** `P0`, `P1`, `P2`, `P3`
- **Status:** `UNANSWERED`, `IN PROGRESS`, `ANSWERED`, `VERIFIED`, `BLOCKED`, `NOT APPLICABLE`
- **Blocks Implementation?:** `YES`, `NO`, `CONDITIONAL`

### Required tab structure

Create one tab per questionnaire category, named for the category title. There
are 26 categories with these prefixes:

`CL-GEN`, `CL-WIN`, `CL-NET`, `CL-SRV`, `CL-TIME`, `CL-SUB`, `CL-AIML`,
`CL-DASH`, `CL-AUTH`, `CL-CODE`, `CL-NAS`, `CL-RES`, `CL-SEARCH`, `CL-AI`,
`CL-REACT`, `CL-DATA`, `CL-OTA`, `CL-SEC`, `CL-SHANA`, `CL-ADMIN`, `CL-OPS`,
`CL-UX`, `CL-EXPO`, `CL-BUD`, `CL-GIT`, `CL-DEC`

⚠️ **Tab names must be short enough to be usable.** If a category title is too
long for a tab name, shorten the tab name — but keep the full category title
inside the `Category` column of every row on that tab. **Never shorten or alter
the category text in the Category column itself.**

Plus `START HERE`, a `Question Count` summary tab, and a
`OPEN QUESTIONS / PRIORITIES` tab.

The `Question Count` tab must report the **true total and the true per-category
and per-priority counts exactly as they appear in the source file. Do not
estimate, round, or derive them from the category index table in the document's
header — count the actual question rows.**

The `OPEN QUESTIONS / PRIORITIES` tab must list only the questions whose
`Blocks?` value is `YES`, or whose Priority is `P0`, ordered by priority. Copy
them verbatim. Do not add commentary, a summary judgement, or a recommendation
about which to answer first.

### Formatting requirements

- Keep every original question.
- Preserve the original order within each priority group.
- Freeze the header row.
- Enable filters.
- Wrap long text.
- Make question IDs visible and non-wrapping.
- Set sensible column widths; the Question and Why It Matters columns need to be
  wide enough to read.
- Format the empty response columns (Answer, Evidence / Source / Attachment,
  Follow-up Required, Notes) with a light background so respondents can see
  where to type.

### Final check before returning

Confirm and state explicitly in your response:

1. Total question rows created equals the number of question rows in the source.
2. Every question ID from the source appears exactly once, verbatim.
3. No question text was added, removed, reworded, merged, or split.
4. Every `Status` cell reads `UNANSWERED`.
5. Every `Answer` cell is empty.
6. Every `Evidence / Source / Attachment`, `Follow-up Required`, and `Notes` cell
   is empty.
7. Every tab has the same 16 columns in the same order.
8. No content from any other questionnaire was included.

If any of these cannot be confirmed, **say so rather than claiming success.**

---

## END PROMPT

---

## Notes for the person operating this

- **Attach `Additional_CloudLight_Questionnaire.md`.** Gemini must not fetch
  questions from the internet.
- **Do not attach the SHANA questionnaire.** Two prompts, two files, two
  workbooks.
- **Verify the row count.** The expected total is **576**. If Gemini returns a
  different number, something was dropped or duplicated — re-run.
- **Verify the per-category counts** against the Category index in the source
  document. A mismatch means a tab lost or gained rows.
- **Verify the first few rows of several tabs.** Spot-check that question text
  matches the markdown verbatim, including bold markers rendered as plain text.
- **Expect refusals on nothing.** Gemini should refuse to answer, not answer. If
  it starts answering questions or offering recommendations, stop and re-prompt
  with the prohibitions.
- **Check that the response columns are genuinely empty.** This is the single most
  important property of the deliverable.
- **Then fill the sheet.** Once the workbook is correct, the humans named in
  `Intended Respondent` fill it in. Note that some questions are internal to the
  project team (`INT`); those are not sent to school staff.