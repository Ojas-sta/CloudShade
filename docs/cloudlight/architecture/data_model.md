# CloudLight — Data Model

**Phase 0 · DESIGN ONLY · 2026-10-11**

> ## ⚠️ **No schema has been implemented. No database exists. No migration exists.**
> This document proposes entities and field sketches in prose. It is **not** a
> DDL specification, not a type contract, and not a schema file.
>
> **It must be revised — not merely filled in — once the school's actual
> timetable has been seen.**

> ### ⚠️ **The single unresolved dependency is `CL-TIME-005`: how a teaching day
> is structured.**
>
> **How many periods, of what length, are there morning and afternoon blocks?**
> It is P0 and `UNANSWERED`. Until it is answered the countdown logic, the
> substitution matcher and the display scheduler **cannot be specified at all —
> only argued about.** Filling this in with a conventional six- or eight-period
> day would be **inventing a school**, and a wrong day model does not degrade: it
> **rejects correct data**, which is indistinguishable from being wrong.
>
> ⚠️ **No school timetable has ever been seen by anyone working on this project.**

This document **extends** rather than replaces §5 of
[`../requirements.md`](../requirements.md), which remains the authoritative
statement of what the system must model and what it must **not**. Where this
document adds detail, it is a sketch of *how*, not a statement of *whether*.

---

## 1. The unresolved dependency, stated first

| Aspect | What is unknown | Question | Consequence |
|---|---|---|---|
| **Period count and length** | ❓ Whether a day is 5, 6, 7 or 8 periods, whether durations are uniform | `CL-TIME-005`, `CL-TIME-006` | The countdown, the whole schedule resolution, every period-indexed query |
| **Day structure** | ❓ Morning/afternoon blocks, separate timings, separate timetables | `CL-TIME-005`, `CL-TIME-007` | Whether a "period 3" is a single concept or two |
| **Non-teaching blocks** | ❓ Breaks, intervals, assemblies, tutorial periods | `CL-TIME-007`, `CL-TIME-016` | Whether a break is a row or the absence of one |
| **Irregularity** | ❓ Half periods, double periods, practicals, part-time teachers, shared rooms | `CL-TIME-006`, `CL-TIME-012` | Whether the grid model survives contact |
| **Academic calendar** | ❓ Holidays, term breaks, exam weeks, special days | `CL-TIME-013`, `CL-TIME-015` | Whether an empty day is an explicit state or a missing row |
| **Weekly rotation** | ❓ Does the week rotate, or is the timetable fixed by weekday | `CL-TIME-008`, `CL-TIME-009` | Whether a slot is (class, period, weekday) or (class, period, date) |

⚠️ **The design stance is to model irregularity before regularity.** An
unscheduled period must be a **first-class state**, never a missing row — because a
missing row and a deliberate free period are different facts, and only one of them
is an error.

⚠️ **What the model below assumes for illustration** — a regular grid of equal
periods with morning and afternoon blocks, a fixed weekday structure, and no
weekly rotation. **Every one of those is an assumption, not a finding.** They are
used so the sketches have something concrete to attach to, and they must be
replaced the moment `CL-TIME-005` returns.

---

## 2. Entity map

```
                        ┌───────────────────┐
                        │   ACADEMIC YEAR   │  ← expiry, rollover
                        └─────────┬─────────┘
                                  │
        ┌───────────────┬─────────┴─────────┬────────────────┐
        │               │                   │                │
        ▼               ▼                   ▼                ▼
  ┌───────────┐   ┌──────────────┐   ┌──────────────┐  ┌──────────────┐
  │   CLASS / │   │    ROOM      │   │   TEACHER    │  │  ACADEMIC    │
  │  SECTION  │   │              │   │              │  │  CALENDAR    │
  └─────┬─────┘   └──────┬───────┘   └──────┬───────┘  └──────────────┘
        │                │                  │
        └────────┬───────┴──────────┬───────┘
                 │                  │
                 ▼                  ▼
        ┌─────────────────┐   ┌──────────────────┐
        │  DAY PROFILE    │   │  DEVICE          │
        │  (periods,      │   │  REGISTRATION    │
        │   blocks)       │   │  (display→room)  │
        └────────┬────────┘   └──────────────────┘
                 │
                 ▼
        ┌──────────────────────────────────────────┐
        │              TIMETABLE SLOT              │
        │  (class × period × day-or-date)          │
        │      → subject, teacher, room            │
        └───────────────────┬──────────────────────┘
                            │
        ┌───────────────────┴──────────────────────┐
        ▼                                          ▼
┌──────────────────┐                    ┌──────────────────┐
│  SUBSTITUTION    │                    │  SOURCE DOCUMENT │
│  RECORD          │◄───────────────────│  (retained)      │
│  + states        │                    └────────┬─────────┘
└────────┬─────────┘                             │
         │              ┌──────────────┐         │
         └─────────────►│ APPROVAL     │◄────────┘
                        │ RECORD       │
                        └──────────────┘

  VERSION ── every change supersedes rather than edits (append-only)

  RESOURCE METADATA ── gated on CL-NAS-001; may never exist
  AUDIT EVENT ──────── actor, timestamp, action, object, outcome
```

---

## 3. Entity sketches

⚠️ **These are field sketches in prose.** Types are indicative. Nothing here is a
contract, and every field sketch is provisional until §1 is answered.

### 3.1 Academic year

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `label` | string | ● | e.g. `2026-27` |
| `starts_on` / `ends_on` | date | ● | Bounds every timetable and substitution record |
| `published_from` | date | ● | ⛔ **The rollover rule is unknown** — `CL-TIME-014` |
| `expires_on` | date | ● | Without this the system serves last year's schedule in April — a failure that is **silent and total** |
| `status` | enum | ● | `current` · `archived` |

⚠️ **The rollover rule is the dangerous field.** Without an explicit expiry, a
display in April confidently shows a schedule from the previous year. `FR-TIME-05`
requires this as data with an explicit rule.

### 3.2 Class / section

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `name` | string | ● | Must exist in the school's own class list |
| `academic_year_id` | foreign key | ● | |
| `streams` | relation | ○ | ❓ **Unknown whether streams exist** — `CL-TIME-011` |
| `rollover_rule` | enum | ● | What happens when a class is promoted |

⚠️ `V-01` rejects a record naming a class not on this list — **blocking, and never
auto-fixable.** CloudLight never invents a class.

### 3.3 Room

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `name` | string | ● | |
| `building` | string | ○ | ❓ Unknown whether the school distinguishes buildings |
| `capacity` | integer | ○ | ❓ Unknown |
| `subjects_supported` | relation | ○ | ⚠️ A subject/room mismatch is a **review flag**, not a rejection — `CL-SUB-016` |

⚠️ **The room, not the class, is what a display is bound to.** A lab, sports or
art lesson legitimately happens somewhere else (`FR-DASH-03`). A display showing
the wrong room is confidently wrong.

### 3.4 Teacher

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `display_name` | string | ● | |
| `directory_ref` | string | ○ | ⚠️ **Only if a directory exists** — `CL-AUTH-001`. ❓ Unknown |
| `employment_status` | enum | ● | ⛔ Offboarding depends on it — `CL-AUTH-007` |
| `account_id` | foreign key | ○ | Only if CloudLight owns identity |

⚠️ **`V-02` rejects a replacement teacher not on the staff list** — blocking, never
fuzzy-matched. ⛔ **And CloudLight never auto-selects a substitute** (`CL-SUB-012`):
a class sent to the library and a class with free study are different outcomes and
must not look identical.

### 3.5 Period / day profile ⚠️ THE PROVISIONAL ENTITY

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `academic_year_id` | foreign key | ● | |
| `name` | string | ● | ⚠️ **The school's naming, not ours** |
| `sequence` | integer | ● | Order within the day |
| `block` | string | ● | Morning / afternoon / other — ❓ structure unknown |
| `starts_at` / `ends_at` | local time | ● | ⚠️ **Times are the school's; the timezone is school local** — `CL-DASH-011` |
| `kind` | enum | ● | `lesson` · `break` · `interval` · `assembly` · `non-teaching` |
| `duration_minutes` | integer | ● | ⛔ **Never assumed uniform** — `FR-TIME-03` |

⚠️ **`kind` exists so a break is a real row that terminates the preceding lesson,
rather than the absence of a row** (`FR-TIME-04`). A display that never shows a
break keeps counting down into a period that does not exist.

⚠️ **`FR-TIME-02` requires this to be configuration, not code constants.** It is the
core schema every downstream feature depends on, and no two schools share one.

### 3.6 Timetable slot

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `class_section_id` | foreign key | ● | |
| `day_profile_id` | foreign key | ● | |
| `period_id` | foreign key | ● | `V-04` rejects one outside the day model |
| `weekday` | integer **or** `date` | ● | ⚠️ **Which one is unknown** — see §1 |
| `subject` | string | ● | |
| `teacher_id` | foreign key | ● | |
| `room_id` | foreign key | ● | `V-06` rejects a double-booked room |
| `academic_year_id` | foreign key | ● | `V-09` rejects a date outside the year |

⚠️ **`V-05` and `V-06` reject double-booked teachers and rooms, blocking.** A
double-booked room produces two wrong displays simultaneously.

### 3.7 Substitution record ⭐ and its states

The field set is derived from
[`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §5.1.
This sketch adds only the state machine, because **the states are the whole
point** and they must never conflate.

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `timetable_slot_id` | foreign key | ● | `V-10`: a substitution for a slot with no lesson is a **conflict** |
| `state` | enum | ● | `proposed` · `scheduled` · `confirmed` · `cancelled` · `room_changed` |
| `original_teacher_id` | foreign key | ● when substituted | Retained for dispute. **Never silently overridden** |
| `replacement_teacher_id` | foreign key | ● when substituted / room_changed | ⛔ Never auto-selected |
| `effective_from` | timestamp | ● | ⚠️ Mid-lesson vs next period is `CL-SUB-019` — **modelled as data** |
| `source_document_id` | foreign key | ● | Provenance |
| `approved_by` | identity | ● | ⛔ **Never a shared account** |
| `approved_at` | timestamp | ● | |
| `version` | integer | ● | Monotonic; increments on every supersession |
| `confidence` | per-field object | ○ | ⚠️ **Advisory to the reviewer. Never a publication criterion** |

#### The states, and what each may be displayed as

| State | Authority | May appear on a display **as a teacher**? |
|---|---|---|
| **Scheduled** (from the published master) | The timetable of record | ✅ Yes — the baseline the school already publishes |
| **Proposed** (extracted or typed, not yet approved) | ⛔ None | ⛔ **Never.** Must be visibly marked unconfirmed |
| **Confirmed** (approved by the named authority) | The named approver | ✅ Yes — the only substitution that may appear as an actual teacher |
| **Cancelled** | The approver | N/A — assigns no one. ⛔ **Not a substitution** |
| **Room changed** | The approver | Teacher unchanged; the room moves, and so does the display |

⚠️ **A display has no room for nuance: a name on a screen in a classroom *is* the
decision, to everyone in the room.** The only meaningful distinction is between a
name that is displayed and a name that is not.

⚠️ **The UI must never invent a teacher when data is missing.** *"Sub: TBC"* is
honest, actionable and disappears the moment someone approves. A plausible wrong
name is neither, and puts a real teacher in a class they are not timetabled to
teach.

⚠️ **`V-12` — two documents substituting the same slot — is never last-write-wins.**
The file ingested last silently wins, and no ordering makes that correct. Two
conflicting instructions are **an event for a human to resolve**; the slot shows
an explicit unknown state until they do.

### 3.8 Source document

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `content_hash` | hash | ● | Hashed and attributed **before parsing** — provenance first |
| `filename` / `origin` | string | ● | |
| `received_at` | timestamp | ● | |
| `retention_until` | date | ● | ❓ Retention is `CL-SRV-006` and carries obligations |
| `contains_personal_data` | boolean | ● | ⚠️ ⚠️ **Source documents contain personal data** by definition |

⚠️ **Without the source, a disputed timetable entry cannot be adjudicated — only
believed** (`FR-TIME-12`). An un-attributable source can never be approved, because
there is nobody to attribute the approval to.

### 3.9 Approval record

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `object_type` / `object_id` | enum + identifier | ● | What was approved |
| `approver_id` | identity | ● | ⛔ **A named person, never a shared account** |
| `approved_at` | timestamp | ● | |
| `basis` | enum | ● | `reviewed_document` · `keyed_from_person` · `machine_extraction_confirmed` |
| `source_document_id` | foreign key | ● | |
| `countersigned_by` | identity | ○ | ❓ Whether separation of duties is required — `CL-SUB-002` |

⚠️ **The three bases are not equivalent, and the distinction is deliberate.** A
hand-keyed record has no derivation risk. A machine extraction *reviewed* by a
person inherits the review as its authority — the model did the typing, the person
did the approving. ⚠️ **Recording the basis is what stops a machine extraction being
presented later as if it were hand-keyed.**

### 3.10 Version

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `entity_type` / `entity_id` | enum + identifier | ● | |
| `version` | integer | ● | Monotonic |
| `supersedes` | foreign key | ● | ⚠️ **Append-only. A correction supersedes; it never edits** |
| `created_by` / `created_at` | identity + timestamp | ● | `FR-TIME-08` |
| `reason` | text | ○ | |

⚠️ **Whether the school wants version history at all is an administration
decision** with a storage cost — `CL-DATA-017`. ❓ Unanswered. ⚠️ **An unversioned
"current timetable" with no history makes corrections unattributable**, which is
why it appears in §5.3 of `../requirements.md` as an entity that must not exist.

### 3.11 Resource metadata ⚠️ MAY NEVER EXIST

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `storage_ref` | path or URI | ● | **By reference only — never a copy** |
| `source_acl_identity` | set of principals | ● | ⚠️ **The permission set captured at index time** |
| `title` / `type` / `size` / `dates` | — | ○ | Metadata, not content |
| `extracted_text_ref` | reference | ○ | ⚠️ **A derived copy, not the document** — deleted when the source is |
| `indexed_at` | timestamp | ● | Drives the index-age display |

⚠️ **An index is a copy, and a copy outlives the permission.** `A5` classifies the
index itself as derived CONFIDENTIAL. ⛔ **CONFIDENTIAL and RESTRICTED material is
excluded by share and by path — not by filename pattern.** An allow-list of what
*may* be indexed is safer than a block-list of what may not.

⚠️ **This entire entity is gated on `CL-NAS-001`.** If no shared storage exists,
these tables are never created. That is a legitimate outcome, not a failure.

### 3.12 Audit event

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `actor` | identity | ● | ⚠️ **Without the actor it is not an audit trail** |
| `timestamp` | timestamp | ● | Server-side, authoritative |
| `action` | enum | ● | |
| `object_type` / `object_id` | enum + identifier | ● | |
| `outcome` | enum | ● | `success` · `denied` · `failed` |
| `detail` | structured | ○ | ⚠️ **No credential, no session token, never content** |

⚠️ **Append-only with tamper evidence** (`FR-ADMIN-04`), and ⛔ **no path that
disables auditing, even for an administrator** (`FR-ADMIN-06`). An admin who can
erase their own actions is an unaccountable admin.

### 3.13 Academic calendar

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `academic_year_id` | foreign key | ● | |
| `date` | date | ● | |
| `kind` | enum | ● | `holiday` · `term_break` · `special_day` · `normal` |
| `alternate_profile_id` | foreign key | ○ | ⚠️ Special days are an **explicit alternate timetable**, never an ad-hoc correction |

⚠️ **Unknown and P0** — `CL-TIME-013`, `CL-TIME-015`. Showing a normal lesson on a
holiday undermines every other correctness guarantee on the display.

### 3.14 Device registration

| Field | Indicative type | Required | Notes |
|---|---|---|---|
| `id` | identifier | ● | |
| `room_id` | foreign key | ● | ⚠️ **The display is bound to the room, not the class** |
| `enrollment_ref` | identifier | ● | For re-enrolment after re-imaging — `FR-OTA-09` |
| `last_seen_at` / `version` | timestamp + string | ● | Every display reports its version — `FR-OTA-08` |
| `last_approved_release` | reference | ● | Boards fall back to this on update failure |

⚠️ **The display count is unknown** — `CL-DASH-001`, `CL-WIN-020` — and
`NFR-SCA-01` **deliberately declines to estimate it**. ⚠️ We refuse to invent this
number.

---

## 4. Key relationships

| From | To | Cardinality | Constraint |
|---|---|---|---|
| Academic year | Class / Room / Day profile | 1 : many | A slot outside the year is **rejected** — `V-09` |
| Day profile | Period | 1 : many | ⚠️ Blocks and non-teaching rows are periods too |
| Class × Period × Day | Timetable slot | 1 : 0..1 | ⚠️ **An unscheduled slot is a first-class state, not a missing row** |
| Timetable slot | Substitution | 1 : 0..many | `V-10` rejects a substitution for a slot with no lesson |
| Substitution | Approval record | 1 : 1+ | ⚠️ **Exactly one named approver** |
| Any derived record | Source document | n : 1 | ⛔ **Without provenance, it cannot be approved** |
| Any record | Version | 1 : many | ⚠️ **Append-only.** A correction supersedes |
| Room | Device registration | 1 : many | Adding a display is configuration, not a deployment — `NFR-SCA-02` |
| Teacher | Account | 0..1 : 1 | ⛔ **Only if CloudLight owns identity** |

⚠️ **The two relationships that carry the safety properties** are *approval record*
and *version*. Remove the approver and publication becomes unreviewable. Remove
the version and a wrong publication becomes unwithdrawable. Both are structural,
not procedural — which is why they are modelled as required fields rather than
workflow steps.

---

## 5. Entities that must NOT exist

⚠️ **Restating §5.3 of [`../requirements.md`](../requirements.md)**, because the
most dangerous modelling error is the one nobody notices.

| Must not be modelled | Why |
|---|---|
| **A per-student record** | Fixed by the no-identification invariant — `C10`, inherited from SHANA |
| **A student's activity or access history** | Same invariant. ⚠️ Students are viewers, never users |
| **A second authoritative timetable copy** | Two live copies diverge. CloudLight is a **consumer and renderer** — `C19` |
| **An unversioned "current timetable" with no history** | Unattributable corrections — §3.10 |
| **A student-facing personalisation entity** | ⚠️ Nothing individualised on a wall |
| **A stored credential or session token** | ⛔ `C5`. Secrets live in a runtime secret store, nowhere else |
| **A confidence-as-permission field** | ⚠️ Confidence is a property of a model, not of the school. It is advisory to a reviewer, never a publication gate |

---

## 6. What this document deliberately does not specify

⚠️ **These omissions are structural, not oversights.**

| Not specified | Why |
|---|---|
| Types, lengths, nullability | ⚠️ **§1 is unanswered.** Precise types on an unknown domain model would be a guess with normative grammar |
| Primary and foreign key constraints | Follow from the relationships once §1 is answered |
| Indexes | Depend on the corpus size and query patterns — ❓ `CL-SEARCH-014` unknown |
| Storage engine | ❓ `CL-DATA-002`. `D-023` proposes one relational database as a **provisional** default |
| Migration strategy | ⚠️ **There is nothing to migrate from.** No schema exists |
| Multi-tenancy | ❓ `CL-BUD-018`. Multi-tenancy before a single tenant is proven is a classic inversion |

---

## 7. Related documents

| Document | Covers |
|---|---|
| [`../requirements.md`](../requirements.md) | §5 — the authoritative entity list and the must-not-exist list |
| [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) | The pipeline, the record fields, validation rules `V-01`…`V-15` |
| [`system_context.md`](system_context.md) | The C4 context view this model sits inside |
| [`../integrations/timetable_sources.md`](../integrations/timetable_sources.md) | Candidate input formats — ❓ **no school timetable has been seen** |
| [`../open_questions_register.md`](../open_questions_register.md) | §1's questions, and their conservative defaults |
| [`../decision_log.md`](../decision_log.md) | `D-023` — the single relational database, provisional |
| [`../../shana/safety/student_privacy.md`](../../shana/safety/student_privacy.md) | SHANA's no-identification invariant, inherited by `C10` |