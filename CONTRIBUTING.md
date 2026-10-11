# Contributing to SHANA

## Current phase

This project is in **feasibility and requirements discovery**. It is
documentation-first. Contributions of **research, analysis, documentation, and
question refinement** are welcome. Contributions of **implementation code are
out of scope until the go/no-go gates are passed.**

---

## What this phase needs most

1. **Answers.** If you are school staff, a robot custodian, an administrator, or
   a lift service provider, the most valuable thing you can contribute is an
   evidence-backed answer to a `P0` question.
2. **Correction.** If you find something inaccurate, wrong, or overclaimed in the
   documentation, that is the single most useful contribution.
3. **Missing sources.** If a claim rests on a weak source, point to a better one.
4. **Missing questions.** If something is not being asked that should be, add it
   to the questionnaire with a new ID.

---

## What is not accepted in this phase

- ❌ Production code, robot-control code, or server implementations.
- ❌ Any configuration of a robot, lift, network, radio, or camera.
- ❌ Any claim that a capability has been demonstrated, unless you can attach the
  evidence.
- ❌ Any inferred or assumed school-specific fact presented as established.
- ❌ Committed credentials, keys, tokens, CCTV details, or private network information.

---

## Evidence rules

Every significant claim carries an evidence label:

| Label | Use when |
|---|---|
| `VERIFIED` | Authoritative documentation or direct authorised evidence |
| `DOCUMENTED_NOT_HARDWARE_VERIFIED` | Manufacturer/vendor documents it; we have not tested it |
| `PLAUSIBLE_REQUIRES_TESTING` | Technically plausible, feasibility not established |
| `UNKNOWN` | Insufficient information |
| `REQUIRES_SCHOOL_PERMISSION` | Requires explicit school approval |
| `REQUIRES_MANUFACTURER_OR_SERVICE_PROVIDER` | Requires vendor confirmation |
| `NOT_RECOMMENDED_OR_NOT_PERMITTED_WITHOUT_APPROVAL` | Unsafe or unsupported |

### Citation requirements

Every claim must carry:

- Source title
- Publisher
- URL
- Publication or update date, if available
- Date accessed
- The specific claim supported
- Any relevant limitation
- Whether the source applies to **our exact hardware revision**

⚠️ **If you could not access a source, say so. Do not write as though you read
it.** `NOT FOUND` is a valid and valuable finding. Do not use marketing claims
as proof of safety or capability.

---

## Writing conventions

- **Never overclaim.** "Documented, not verified on our hardware" is a correct
  and valuable sentence. "Works" is not acceptable without evidence.
- **Record uncertainty explicitly.** Where a number conflicts across sources, show
  the conflict rather than picking one silently.
- **Distinguish absence of permission from absence of capability.** They need
  different fixes.
- **Use UK-influenced English consistently**, matching the existing documents.
- **Keep tables readable.** Wide tables break in diffs; prefer one topic per table.

---

## Changing a question

Question IDs in `questionnaires/SHANA_Feasibility_QA.md` are **stable**.

- Never reuse an ID for a different question.
- If a question is removed, keep the row and mark it `NOT APPLICABLE` with a note.
- If a question is reworded, keep the ID and note the change.
- If a question is split into two, keep the original ID for one half and allocate
  a new ID for the other.

---

## Changing a decision

Decisions live in `docs/shana/feasibility/decision_log.md`.

Every decision records its **reversal condition** — what would cause it to be
revisited. ⚠️ A decision without a recorded reversal condition becomes a decision
nobody dares revisit. If you reverse a decision, record why, with the evidence
that forced it.

---

## Commit and branch conventions

- Branch names: `research/<topic>`, `docs/<topic>`, `questions/<topic>`.
- One logical change per commit.
- Reference question IDs in commit messages where relevant, e.g.
  `docs(feasibility): record r1 answers to SH-STAIR-001..003`.
- **Do not commit, push, publish, or change remotes without explicit
  authorisation from the repository owner.**

---

## Secrets

Never commit, and never paste into an issue or document:

- Wi-Fi keys or passwords
- CCTV credentials or stream URLs
- API tokens
- Robot or server credentials
- Private network topology details, IP assignments, or VLAN configurations
- Student or staff personal data
- Floor plans or building layouts not approved for circulation

If you discover any of these already committed, tell the repository owner
immediately. See [`SECURITY.md`](SECURITY.md).

---

## Before you open a pull request

- [ ] No credentials, keys, tokens, or private network details included
- [ ] Every new claim carries an evidence label and a citation
- [ ] No capability is described as operational without evidence
- [ ] Changes to the questionnaire follow the ID stability rules
- [ ] Changes to decisions include a reversal condition
- [ ] Documentation builds and reads correctly