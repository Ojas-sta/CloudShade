# CloudLight — Requirements

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ NOTHING HERE IS APPROVED
>
> **No requirement in this document has been agreed by the school, by school IT,
> by the timetable department, or by any member of staff.** Every `CL-*` question
> in [`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)
> (576 of them) is `UNANSWERED` as of the date above.
>
> **Nothing has been built.** No host exists, no network has been accessed or
> probed, no smartboard has been touched, no timetable has been seen, no robot has
> been operated, and no school document has been read.
>
> This document converts a set of **proposed capabilities** into **testable
> statements** so that they can be disagreed with precisely. It does not convert
> them into commitments.

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in authoritative vendor documentation or a published standard |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | A vendor or standard documents it; not confirmed on any device this school owns |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No vendor claim. Engineering judgement |
| ❓ **UNKNOWN** | No evidence either way. Not assumed. To be answered by a named party |
| ⛔ **NOT RECOMMENDED / NOT PERMITTED** | Rejected by our judgement, by vendor guidance, or by a constraint already accepted in this repository |

---

## 1. How to read this document

### 1.1 The four requirement classes

Every requirement below carries exactly one class. The class tells you **what
would have to happen for it to become a commitment.**

| Class | Meaning | What it takes to promote it |
|---|---|---|
| **`CONFIRMED`** | Follows from a binding constraint **already accepted in this repository**. It is true whether or not anyone answers a questionnaire | Nothing. It is already binding. It can still be *disproved* by a fact we do not have |
| **`PROPOSED`** | The intended behaviour. Sensible, argued for, and **agreed by nobody** | Written agreement from the named party who would be affected |
| **`PENDING ANSWER`** | Cannot be specified until a named party answers a questionnaire question. The requirement is written as a **question-shaped hole**, not as a specification | The answer to the named `CL-*` question |
| **`OUT OF SCOPE`** | Deliberately excluded from the first release. See §6 | The named question that would unblock it, plus a decision to un-scope it |

### 1.2 The rule about `PENDING ANSWER`

> ### ⚠️ **A `PENDING ANSWER` requirement cannot be written as a specification, and this document does not write one.**
>
> Writing *"the system shall display the current period for the bound class"* is
> **impossible** until we know whether a period is defined by wall-clock time, by
> bell, by teacher, by room, or by a weekly matrix that rotates. `CL-TIME-005` and
> `CL-TIME-006` are both P0 and both `UNANSWERED`. A specification written before
> those answers arrive is not a specification — it is a guess with normative
> grammar, and it is considerably harder to notice than a guess in a comment.
>
> Every such requirement below therefore records **what must be true**, **why it
> matters**, and **which question unblocks it** — and stops there.

### 1.3 The rule about numbers

> ### ⚠️ **Every numeric target in this document is a proposal requiring confirmation.**
>
> No school has stated a required refresh rate, latency budget, session timeout,
> device count, restore time, data-loss window or font size. The figures in §3 are
> **engineering proposals chosen to be arguable**, and several are deliberately
> rounded or expressed as ranges precisely because presenting a precise number
> would misrepresent the confidence we do not have. Each carries the question that
> would confirm or replace it.

### 1.4 Cross-references

| Document | Scope |
|---|---|
| [`network_and_hosting.md`](network_and_hosting.md) | The network/host/display distinction; hosting options; segmentation; backup; power |
| [`classroom_dashboard.md`](classroom_dashboard.md) | Display content, countdown, degraded states |
| [`timetable_and_substitutions.md`](timetable_and_substitutions.md) | Timetable data model, period model, substitution workflow |
| [`resource_browser.md`](resource_browser.md) | Storage reality check, extraction, ACL capture |
| [`resource_search.md`](resource_search.md) | Index tiers, permission-aware retrieval, staleness |
| [`ai_assistant.md`](ai_assistant.md) | Local inference, RAG, prompt injection, propose-only boundary |
| [`authentication_and_authorisation.md`](authentication_and_authorisation.md) | Identity, sessions, six-digit codes, unattended boards |
| [`security_and_privacy.md`](security_and_privacy.md) | Threat model, controls, privacy boundary |
| [`ota_updates.md`](ota_updates.md) | Release signing, staged rollout, rollback |
| [`shana_integration.md`](shana_integration.md) | The CloudLight/SHANA ownership boundary |
| [`../shana/architecture/architecture.md`](../shana/architecture/architecture.md) | The AI/deterministic boundary; the local application server |
| [`../shana/safety/risk_register.md`](../shana/safety/risk_register.md) | The `L × S` rating convention used across this repository |
| [`../shana/safety/student_privacy.md`](../shana/safety/student_privacy.md) | The no-identification invariant |
| [`../shana/infrastructure/school_network.md`](../shana/infrastructure/school_network.md) | The prohibition on probing school infrastructure |
| [`deployment_options.md`](deployment_options.md) | Host and client-form options; upgrade triggers; handover |

---

## 2. Functional requirements by module

⚠️ **Columns are fixed:** `ID | Requirement | Class | Rationale | Depends on questionnaire`
Every "Depends on" value is a `CL-*` ID from the CloudLight questionnaire. A `-`
means the requirement follows from a constraint already accepted here and needs no
answer to be stated.

### 2.1 Classroom Display

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-DASH-01 | The system **MUST NOT** permit any change to timetable, substitution or resource state from a classroom display | `CONFIRMED` | The display is a read-only surface. A writable student-facing screen is an unauthenticated write path to the school's timetable. Fixed by [`security_and_privacy.md`](security_and_privacy.md) and [`classroom_dashboard.md`](classroom_dashboard.md) | — |
| FR-DASH-02 | The system **MUST NOT** require a teacher login for the display to show a lesson | `CONFIRMED` | A login prompt in front of a class, at lesson start, is a barrier that gets bypassed or abandoned. The display is read-only *and* unauthenticated by design | `CL-SEC-001`, `CL-DASH-021` |
| FR-DASH-03 | The system **MUST** show the lesson for the room the display is bound to, not the room its class is nominally in | `PROPOSED` | A lab, sports or art lesson legitimately happens somewhere else. A display showing the wrong room is confidently wrong | `CL-TIME-010`, `CL-DASH-001` |
| FR-DASH-04 | The system **MUST** show a countdown to the end of the current lesson | `PROPOSED` | The single most-requested classroom-display behaviour; also the behaviour most sensitive to period-model error | `CL-DASH-008`, `CL-TIME-006` |
| FR-DASH-05 | The countdown **MUST NOT** be shown at all when the display's clock is unsynced or its time source is unverified; it **MUST** instead show an explicit unreliable marker | `PENDING ANSWER` | A countdown computed from a wrong clock is a confident falsehood in front of students. Whether the school prefers hidden or marked is their call | `CL-DASH-009`, `CL-DASH-010` |
| FR-DASH-06 | The system **MUST** show a visible last-synced timestamp when serving cached data | `PROPOSED` | Silent staleness is indistinguishable from "nothing is scheduled". A timestamp converts a guess into a fact a teacher can act on | `CL-DASH-013`, `CL-SEC-018` |
| FR-DASH-07 | The system **MUST** display a distinct, non-error state when it cannot reach the host, rather than a blank screen or a browser error page | `PROPOSED` | A blank screen is indistinguishable from a broken panel; a browser error page implies the school has broken something | `CL-DASH-012`, `CL-UX-012`, `CL-SEC-021` |
| FR-DASH-08 | The system **MUST** support operating today's lesson from cache when the host is unreachable | `PENDING ANSWER` | What must survive a full outage is a school policy decision, not a technical one — the two candidate lists differ | `CL-SEC-021`, `CL-SEC-022` |
| FR-DASH-09 | The system **MUST** render an empty or holiday day as an explicit "no lessons today" state, not as a blank or stale timetable | `PENDING ANSWER` | Showing a normal lesson on a holiday day undermines every other correctness guarantee on the display | `CL-DASH-014`, `CL-TIME-013`, `CL-TIME-015` |
| FR-DASH-10 | The system **MUST NOT** display student names, marks, grades or attendance | `CONFIRMED` | Fixed by the no-identification invariant in [`../shana/safety/student_privacy.md`](../shana/safety/student_privacy.md) | `CL-DASH-003`, `CL-DASH-002`, `CL-SEC-010` |
| FR-DASH-11 | The system **MUST** distinguish free periods, cancelled periods and substituted periods without relying on colour | `PROPOSED` | Colour is never the sole carrier of meaning. A substitution shown only as a different colour is invisible to a colour-blind teacher and to a photocopier | `CL-UX-005`, `CL-UX-006` |
| FR-DASH-12 | Announcements, if supported at all, **MUST** require named approval before appearing on any display | `PENDING ANSWER` | Whether announcements exist is not yet established; if they do, "who may post" is an administration decision | `CL-DASH-006`, `CL-DASH-007` |
| FR-DASH-13 | The system **MUST** operate fully when SHANA is unreachable, showing a clear non-alarming "robot unavailable" state | `CONFIRMED` | A teacher must be able to distinguish "no robot" from "CloudLight is broken" | `CL-DASH-022`, `CL-SEC-017` |
| FR-DASH-14 | The system **MUST NOT** show a robot status whose value is `UNKNOWN` as though it were `IN_PROGRESS` | `CONFIRMED` | An invented status on a wall in front of children is the worst available failure | `CL-SHANA-*` (see [`shana_integration.md`](shana_integration.md)) |
| FR-DASH-15 | The system **MUST** bound the number of distinct display states to a documented, testable catalogue | `PROPOSED` | Unbounded state growth is how a failure mode becomes undiscoverable | `CL-DASH-004`, `CL-DASH-005` |

### 2.2 Timetable

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-TIME-01 | The system **MUST** treat the timetable as **read-only to CloudLight** unless the school names CloudLight as an authoritative system of record | `CONFIRMED` | Two live copies create reconciliation ambiguity; the fastest way to lose staff trust is a display disagreeing with the staff-room board | `CL-TIME-001`, `CL-DATA-019` |
| FR-TIME-02 | The system **MUST** model the period/day profile (period count, duration, morning/afternoon blocks) as configuration, not as code constants | `CONFIRMED` | It is the core schema every downstream feature depends on, and no two schools share one | `CL-TIME-005`, `CL-TIME-006`, `CL-TIME-008` |
| FR-TIME-03 | The system **MUST NOT** assume uniform period duration | `CONFIRMED` | Double periods, half periods and practicals are common; a uniform assumption shows the wrong remaining time to students | `CL-TIME-006` |
| FR-TIME-04 | The system **MUST** model breaks, intervals and non-teaching blocks as first-class entries that terminate the preceding lesson | `CONFIRMED` | A display that never shows a break keeps counting down into a period that does not exist | `CL-TIME-007`, `CL-TIME-016` |
| FR-TIME-05 | The system **MUST** model the academic calendar (holidays, term breaks, special days) as data with an explicit rollover or expiry rule | `CONFIRMED` | Without a rollover rule the system keeps serving last year's schedule in April — a failure that is silent and total | `CL-TIME-013`, `CL-TIME-014`, `CL-TIME-015` |
| FR-TIME-06 | The system **MUST** represent special days as an explicit alternate timetable, never as an ad-hoc correction to the master | `PROPOSED` | Exam days deliberately break the normal period structure | `CL-TIME-015` |
| FR-TIME-07 | The system **MUST** store timetable changes as an immutable, versioned history that can be rolled back | `PENDING ANSWER` | Whether the school wants version history or only current state is an administration decision with a storage cost | `CL-DATA-017`, `CL-SUB-020` |
| FR-TIME-08 | The system **MUST** attribute every timetable change to one named actor | `PROPOSED` | Unattributable changes cannot be audited, reviewed or rolled back confidently | `CL-AUTH-004`, `CL-SEC-020` |
| FR-TIME-09 | The system **MUST** detect and refuse a duplicate or conflicting assignment (same room, same period, two teachers) | `PROPOSED` | A double-booked room produces two wrong displays simultaneously | `CL-DATA-018` |
| FR-TIME-10 | The system **MUST NOT** automatically publish a timetable derived by machine extraction without human approval | `PENDING ANSWER` | The acceptable review burden is the timetable department's call | `CL-SUB-003`, `CL-SEC-014`, `CL-AIML-*` |
| FR-TIME-11 | The system **MUST** be able to ingest the timetable from whatever format the school actually uses | `PENDING ANSWER` | Spreadsheet, printed PDF, portal and software package imply four different ingestion paths. We do not know which applies | `CL-TIME-002`, `CL-TIME-003` |
| FR-TIME-12 | The system **MUST** preserve the source document alongside any derived record | `PROPOSED` | Without the source, a disputed timetable entry cannot be adjudicated — only believed | `CL-DATA-001`, `CL-TIME-001` |
| FR-TIME-13 | The system **MUST** tolerate a class or teacher absent from the source data, without dropping the whole import | `PROPOSED` | Partial failure must be visible and reported, not silently fatal | `CL-TIME-009`, `CL-TIME-012` |
| FR-TIME-14 | The system **MUST NOT** be the sole copy of the timetable under any deployment | `CONFIRMED` | The school retains its authoritative record regardless of what CloudLight stores | `CL-TIME-001`, `CL-DATA-019` |

### 2.3 Substitution

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-SUB-01 | The system **MUST NOT** publish an unapproved substitution | `PENDING ANSWER` | `CL-SUB-003` asks directly and is P0: never publish, at the cost of brief staleness. We recommend *yes*; the school decides | `CL-SUB-003`, `CL-SUB-002` |
| FR-SUB-02 | The system **MUST** support the full substitution state model — `scheduled`, `proposed`, `cancelled`, `room-changed` — as distinct, non-collapsing states | `PROPOSED` | Collapsing "proposed" into "scheduled" destroys the audit trail; collapsing "cancelled" into "absent" produces phantom frees | `CL-SUB-019`, `CL-SUB-012` |
| FR-SUB-03 | The system **MUST NOT** automatically select a substitute teacher when none is named | `PENDING ANSWER` | `CL-SUB-012` is P0 and asks whether this is ever acceptable. Our position is that it should not be | `CL-SUB-012`, `CL-SUB-014` |
| FR-SUB-04 | The system **MUST** record a substitution record even when no teacher is available, representing the "no teacher" outcome explicitly | `PROPOSED` | A class sent to the library and a class with a free study period are different outcomes and must not look identical | `CL-SUB-015` |
| FR-SUB-05 | The system **MUST** support withdrawal and correction of a published substitution with a full audit record | `PROPOSED` | The school withdraws errors today by phone or strike-through; CloudLight needs an equivalent, recorded path | `CL-SUB-004`, `CL-SUB-023`, `CL-SEC-015` |
| FR-SUB-06 | The system **MUST** propagate a correction to already-displaying boards within the school's stated tolerance | `PENDING ANSWER` | A wrong lesson stays wrong on the wall until it is corrected; how fast that must happen is the school's call | `CL-SUB-023`, `CL-SEC-014`, `CL-DASH-013` |
| FR-SUB-07 | The system **MUST NOT** treat a photograph of a substitution board as authoritative | `PENDING ANSWER` | Whether a photograph is sufficient input or always needs human confirmation is an administration decision | `CL-SUB-006`, `CL-SUB-007` |
| FR-SUB-08 | The system **MUST** refuse to ingest a substitution photograph or document without explicit permission to do so | `CONFIRMED` | Photographing school material is a permission, not a technical step | `CL-SUB-006`, `CL-SEC-011` |
| FR-SUB-09 | The system **MUST** model a substitution's effective time — mid-lesson versus next period — as data | `PROPOSED` | Both are legitimate school practices and they produce different displays | `CL-SUB-019` |
| FR-SUB-10 | The system **MUST** support room-changing substitutions | `PROPOSED` | A room change moves the lesson to a different display; the existing display must change too | `CL-SUB-016`, `CL-TIME-010` |
| FR-SUB-11 | The system **MUST** record who approved a substitution and when | `PROPOSED` | `CL-SUB-020` asks for an audit trail; without an actor there is none | `CL-SUB-020`, `CL-SUB-002` |
| FR-SUB-12 | The system **MUST NOT** contain student-identifying information in substitution records | `CONFIRMED` | Fixed by the no-identification invariant | `CL-SUB-021`, `CL-DASH-003` |
| FR-SUB-13 | The system **MUST** surface a conflict (one teacher marked free by two departments) rather than choosing one | `PENDING ANSWER` | Resolution requires a human with authority we do not have | `CL-SUB-011` |
| FR-SUB-14 | The system **MUST** bound substitution visibility to the rooms the school specifies | `PENDING ANSWER` | All rooms, only affected rooms, or also staff phones — three different privacy postures | `CL-SUB-022`, `CL-SUB-024` |

### 2.4 Resource Browser

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-RES-01 | The system **MUST NOT** write to, move, rename or delete any resource on the school's storage | `CONFIRMED` | CloudLight is a reader of the school's NAS, not its custodian | `CL-NAS-*`, `CL-RES-*` |
| FR-RES-02 | The system **MUST** access storage using a **read-only service account** with its own credentials | `PROPOSED` | Reusing a staff or administrator credential makes every CloudLight action attributable to a person who did not perform it | `CL-SEC-005`, `CL-NAS-006` |
| FR-RES-03 | The system **MUST NOT** honour a permission decision that differs from the storage's own ACLs | `CONFIRMED` | Two permission models in one product means the weaker one wins at the worst moment | `CL-SEC-007` |
| FR-RES-04 | The system **MUST** re-check permission at **query time** and again at **open time**, not only at index time | `CONFIRMED` | ACLs change between indexing and opening; an index-time-only check leaks on every permission change | `CL-SEARCH-002`, `CL-SEARCH-003` |
| FR-RES-05 | The system **MUST NOT** be enabled at all if the school has no reachable file share | `PENDING ANSWER` | There is no fallback. If there is no share, this module has nothing to browse | `CL-NAS-001`, `CL-NAS-002`, `CL-SRV-014` |
| FR-RES-06 | The system **MUST** support directories, files and basic hierarchical navigation without extracting content | `PROPOSED` | Browsing must work when parsing is disabled or broken — the fallback tier | `CL-RES-001` |
| FR-RES-07 | The system **MUST** treat extracted text as a **derived copy**, not as the document | `PROPOSED` | A derived copy that outlives its source is a privacy and correctness problem | `CL-DATA-020` |
| FR-RES-08 | The system **MUST** delete derived index entries and extracted text when the source is deleted | `PENDING ANSWER` | Whether deletion must propagate to the index is explicitly a school decision | `CL-DATA-020` |
| FR-RES-09 | The system **MUST** extract any file in a crash-isolated, sandboxed process | `PROPOSED` | A parser is an execution surface; a malformed document must not take down the service | `CL-SEC-008` |
| FR-RES-10 | The system **MUST NOT** execute or interpret uploaded or fetched files as part of indexing | `CONFIRMED` | Standard defence-in-depth; adopted from [`security_and_privacy.md`](security_and_privacy.md) | `CL-SEC-008` |
| FR-RES-11 | The system **MUST NOT** store the extracted content where the web server could serve it directly | `PROPOSED` | Derived text served as a static file bypasses every access check | `CL-SEC-007` |
| FR-RES-12 | The system **MUST** enforce a size and decompression limit computed **after** decompression | `PROPOSED` | A limit checked before decompression is not a limit | `CL-SEC-008` |
| FR-RES-13 | The system **MUST NOT** assume SMBv2/SMBv3 is available, nor assume SMBv1 is acceptable | `CONFIRMED` | SMBv1 is deprecated by Microsoft and unsupported; a share reachable only over SMBv1 is a blocked dependency, not a workaround | `CL-NAS-002`, `CL-NAS-004` |
| FR-RES-14 | The system **MUST** degrade to directory listing only when extraction fails for a given file | `PROPOSED` | One unreadable file must not remove its whole directory from the product | `CL-RES-*` |
| FR-RES-15 | The system **MUST** show whether a resource is missing from the index rather than implying it does not exist | `PROPOSED` | Absence of an index entry is ambiguous between "not indexed" and "not permitted" | `CL-SEARCH-*`, `CL-RES-*` |

### 2.5 Search

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-SEARCH-01 | The system **MUST** provide keyword search over resource metadata and, where extraction succeeded, extracted text | `PROPOSED` | Keyword search is the tier that works on modest hardware and requires no model | `CL-SEARCH-001`, `CL-SEARCH-002` |
| FR-SEARCH-02 | The system **MUST** filter results by the authenticated user's entitlements **before** ranking or returning them | `CONFIRMED` | Ranking-then-filtering leaks existence and metadata through ordering and counts | `CL-SEC-007`, `CL-SEARCH-003` |
| FR-SEARCH-03 | The system **MUST NOT** return a result the user may not open | `CONFIRMED` | A clickable result that fails on open is a worse leak than no result, because it confirms the document exists | `CL-SEC-007` |
| FR-SEARCH-04 | The system **MUST** display the age of the index to staff, distinguishing "not indexed" from "indexed and you may see it" | `PROPOSED` | A stale index silently produces confidently wrong answers | `CL-SEARCH-*`, `CL-DATA-020` |
| FR-SEARCH-05 | The system **MUST NOT** reveal field names or term counts that exist only in inaccessible documents | `CONFIRMED` | Aggregated statistics leak across a permission boundary; this is a documented real-world failure | `CL-SEC-007` |
| FR-SEARCH-06 | The system **MUST** degrade to metadata-only search when the content index is unavailable | `PROPOSED` | An index rebuild must not take search offline | `CL-SEARCH-*` |
| FR-SEARCH-07 | The system **MUST NOT** return an unverified machine-ranked result as though it were authoritative | `PROPOSED` | Distinguishing "exact title match" from "semantically similar" is the difference between a useful answer and a plausible one | `CL-SEARCH-*`, `CL-UX-018` |
| FR-SEARCH-08 | Search **MUST** work in the languages and scripts the school actually uses | `PENDING ANSWER` | Script coverage is a hard limit of any OCR or stemmer; we do not know the language set | `CL-UX-016`, `CL-UX-017` |
| FR-SEARCH-09 | The system **MUST NOT** depend on file-change notification from the storage to remain correct | `PROPOSED` | Notification is an optimisation; its absence must cause delay, not wrongness | `CL-NAS-002`, `CL-NET-*` |
| FR-SEARCH-10 | The system **MUST** re-verify a file against storage at the moment it is opened | `PROPOSED` | The file may have been deleted or its permissions changed since indexing | `CL-SEARCH-003` |
| FR-SEARCH-11 | Semantic or vector search **MUST NOT** be required for the search module to satisfy its function | `PROPOSED` | It is an enhancement tier; making it a prerequisite converts an optional capability into a hardware dependency | `CL-SEARCH-*`, `CL-AI-009` |
| FR-SEARCH-12 | The system **MUST** bound index rebuild time so that it does not run during a period change | `PROPOSED` | A Monday-morning reindex that stalls the dashboard is visible to every teacher simultaneously | `CL-SRV-021`, `CL-OPS-001` |

### 2.6 AI Assistant

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-AI-01 | The system **MUST NOT** call an external AI service unless the school has given written authorisation for that specific service | `CONFIRMED` | Sending school material to a third party is a data-transfer decision the school has not made | `CL-SEC-011`, `CL-BUD-006` |
| FR-AI-02 | The system **MUST** default to fully local inference, with no outbound call in any code path, including error handlers | `CONFIRMED` | Default-off is a design property. A default-on with a kill switch is a different and weaker thing | `CL-SEC-011`, `CL-GEN-004` |
| FR-AI-03 | The assistant **MUST** be **propose-only**: it may draft, and a human must approve before anything is published or executed | `CONFIRMED` | Fixed by the AI/deterministic boundary in [`../shana/architecture/architecture.md`](../shana/architecture/ architecture.md) and SHANA `D-005` *(indexed as ADR-001)* | `CL-SEC-009`, `CL-SUB-003` |
| FR-AI-04 | The assistant **MUST NOT** be able to reach the robot under any code path, including a tool-use or function-calling path | `CONFIRMED` | The boundary is expressed as an **absence of capability**, not as a policy. See [`shana_integration.md`](shana_integration.md) | — |
| FR-AI-05 | Retrieved document content **MUST** be treated as **data, never as instruction** | `CONFIRMED` | Indirect prompt injection is documented as not fully mitigable; the structural defence is segregation, not filtering | `CL-SEC-009` |
| FR-AI-06 | The system **MUST NOT** allow a school document to act as a standing instruction to the assistant | `CONFIRMED` | A "handbook" the assistant obeys is an unversioned, unreviewed instruction channel into a classroom | `CL-SEC-009` |
| FR-AI-07 | The system **MUST** display the source of every claim it makes, or say that it cannot | `PROPOSED` | An unsourced answer in a classroom is indistinguishable from an invented one | `CL-UX-018` |
| FR-AI-08 | The system **MUST NOT** display a student name, photograph or attendance-derived statement in an assistant answer | `CONFIRMED` | Fixed by the no-identification invariant | `CL-SEC-010`, `CL-DASH-003` |
| FR-AI-09 | The system **MUST** validate assistant output against a deterministic schema **in code**, not by asking the model to be careful | `CONFIRMED` | Documented mitigation for prompt injection; the model is not the validator | `CL-AI-*`, `CL-AIML-*` |
| FR-AI-10 | Assistant conversation history **MUST** be retained only as long as the school's stated policy permits | `PENDING ANSWER` | Retention periods for conversation transcripts are a school/legal determination | `CL-SEC-025`, `CL-DATA-015` |
| FR-AI-11 | The assistant **MUST NOT** be available on a student-facing display | `PROPOSED` | A generative surface in a room of children is a safety and attention problem independent of its accuracy | `CL-SEC-010`, `CL-DASH-002` |
| FR-AI-12 | The system **MUST** degrade to a non-AI answer path — deterministic lookup, or a clear refusal — when the model is unavailable | `PROPOSED` | An assistant that fails silently is worse than one that is absent | `CL-AI-*`, `CL-SEC-021` |
| FR-AI-13 | The assistant **MUST NOT** be included in the first release | `OUT OF SCOPE` | §6 #2. It is the largest unverified dependency in the project and has a human cost before it has a benefit | `CL-AI-009`, `CL-AI-023`, `CL-BUD-006` |

### 2.7 Authentication

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-AUTH-01 | The system **MUST** support individual teacher accounts | `PROPOSED` | Every action must be attributable to one named teacher; a shared account makes the audit trail fictional | `CL-AUTH-004`, `CL-DATA-004` |
| FR-AUTH-02 | The system **MUST NOT** keep a teacher session valid on an unattended display | `CONFIRMED` | A board holding a teacher's session that any child can approach is an account compromise waiting to happen | `CL-AUTH-020`, `CL-AUTH-021`, `CL-SEC-001` |
| FR-AUTH-03 | The system **MUST** implement both an idle timeout and an absolute session timeout, as distinct controls | `PROPOSED` | A single timeout either lets an abandoned session live forever or logs out an active teacher mid-lesson | `CL-AUTH-016`, `CL-SEC-002` |
| FR-AUTH-04 | The system **MUST** invalidate a session server-side immediately on logout or privilege change | `PROPOSED` | Client-side deletion of a token does not invalidate anything | `CL-SEC-004`, `CL-AUTH-022` |
| FR-AUTH-05 | The system **MUST NOT** store a session token in a URL | `CONFIRMED` | URLs leak through history, referrer headers, logs and screenshots | `CL-AUTH-*` |
| FR-AUTH-06 | The system **MUST NOT** store a session token in browser local storage | `PROPOSED` | Local storage is readable by any script on the origin, including injected script | `CL-SEC-005` |
| FR-AUTH-07 | The system **MUST** re-evaluate authorisation on **every request**, server-side | `CONFIRMED` | A permission checked once at sign-in is revoked the moment it is granted | `CL-SEC-007`, `CL-AUTH-024` |
| FR-AUTH-08 | The system **MUST** rate-limit six-digit code attempts and lock out with administrator notification | `PENDING ANSWER` | The threshold and the notification target are school policy | `CL-SEC-003`, `CL-CODE-*` |
| FR-AUTH-09 | The system **MUST NOT** use SMS or PSTN as a second factor | `PROPOSED` | NIST designates it a restricted authenticator with known interception and SIM-swap exposure. See [`security_and_privacy.md`](security_and_privacy.md) | `CL-AUTH-011`, `CL-AUTH-012` |
| FR-AUTH-10 | The system **MUST** provide a recovery path that does not require the Principal for a routine reset | `PENDING ANSWER` | A teacher locked out at 07:50 either gets a same-day recovery or does not use the system | `CL-AUTH-014`, `CL-AUTH-015` |
| FR-AUTH-11 | The system **MUST** deactivate a leaver's account within the school's stated window | `PENDING ANSWER` | The window is a school HR process property, not a technical one | `CL-AUTH-007`, `CL-AUTH-008` |
| FR-AUTH-12 | The system **MUST** NOT store, log or transmit a password or secret in plaintext | `CONFIRMED` | Fixed by the no-secret-in-Git/logs/URLs constraint in §4 | `CL-SEC-006`, `CL-AUTH-009`, `CL-GIT-014` |
| FR-AUTH-13 | The system **SHOULD** support a second factor for administrative roles even if teachers do not use one | `PROPOSED` | The privileged account is the one worth attacking; the ordinary teacher account is not | `CL-AUTH-011`, `CL-AUTH-024` |
| FR-AUTH-14 | The system **MUST NOT** present a login prompt on a display as the normal way to view a lesson | `CONFIRMED` | Same reason as FR-DASH-02. Login-at-the-board is for *override*, not for viewing | `CL-DASH-021`, `CL-SEC-001` |

### 2.8 Administration

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-ADMIN-01 | The system **MUST** deny by default and require an explicit grant for every permission | `CONFIRMED` | Deny-by-default is the only model that fails safely when the model is wrong | `CL-SEC-007`, `CL-DATA-005` |
| FR-ADMIN-02 | The system **MUST** keep administration interfaces unreachable from a student-facing segment | `PROPOSED` | On a flat network, any compromised display is one hop from the control plane | `CL-SEC-023`, `CL-NET-001` |
| FR-ADMIN-03 | The system **MUST** record every administrative action with actor, timestamp, action and affected object | `PROPOSED` | An audit trail that omits the actor is not an audit trail | `CL-SEC-020`, `CL-SEC-019` |
| FR-ADMIN-04 | The system **MUST** make the audit log append-only with tamper evidence | `PENDING ANSWER` | Whether this is required is `CL-SEC-019`; we recommend it and record it as a requirement either way | `CL-SEC-019`, `CL-OTA-016` |
| FR-ADMIN-05 | The system **MUST** support least privilege and separation between publishing and approving | `PENDING ANSWER` | Whether one person may both propose and approve a substitution is an administration decision | `CL-SUB-002`, `CL-OTA-003` |
| FR-ADMIN-06 | The system **MUST NOT** provide a path that disables auditing, even for an administrator | `CONFIRMED` | An admin who can erase their own actions is an unaccountable admin | `CL-SEC-019` |
| FR-ADMIN-07 | The system **MUST** allow an operator to promote corrected configuration data from staging to production through a recorded step | `PROPOSED` | Direct editing of live timetable data is how a wrong lesson reaches a wall | `CL-OTA-020`, `CL-DATA-006` |
| FR-ADMIN-08 | The system **MUST** support role assignment by the school, not by a hard-coded list | `PENDING ANSWER` | Which roles exist beyond teacher is not established | `CL-AUTH-024`, `CL-DATA-005` |
| FR-ADMIN-09 | The system **MUST NOT** require a school administrator account for routine classroom operation | `PROPOSED` | If a lesson depends on an admin account, it fails on the first admin absence | `CL-OPS-006` |
| FR-ADMIN-10 | The system **MUST** provide a documented manual fallback for each administrative workflow | `PROPOSED` | When the backend is down, the school still runs its timetable — on paper or a whiteboard | `CL-SEC-022` |

### 2.9 Update Management

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-OTA-01 | A display **MUST** accept a frontend bundle only if it is validly signed, and **MUST** refuse an unsigned or incorrectly signed one | `PROPOSED` | Signature verification belongs on the consuming device, not at the distribution point | `CL-OTA-004`, `CL-SEC-013` |
| FR-OTA-02 | The system **MUST** support rolling back to the previous known-good release | `PROPOSED` | Every update is a bet; rollback is what makes the bet acceptable | `CL-OTA-007` |
| FR-OTA-03 | The system **MUST NOT** deploy automatically without per-deployment approval until the school says otherwise | `CONFIRMED` | Fixed by the separate-approval-path constraint in §4 | `CL-OTA-012`, `CL-OTA-003` |
| FR-OTA-04 | Software updates and timetable/data updates **MUST** use **separate approval paths** | `CONFIRMED` | A data correction is urgent and routine; a software change is neither. One path forces one of them to be wrong | `CL-OTA-019`, `CL-OTA-001` |
| FR-OTA-05 | The system **MUST** tolerate a display running a cached frontend one version behind the backend | `PROPOSED` | Boards are slow to update and may be mid-update when the backend moves | `CL-OTA-006` |
| FR-OTA-06 | The system **MUST** abort an interrupted update cleanly to the previous version, never to a partial one | `PROPOSED` | Half-applied frontend is indistinguishable from a broken board to a teacher | `CL-OTA-013` |
| FR-OTA-07 | The system **MUST** support piloting an update on one nominated display before general rollout | `PROPOSED` | The cheapest real test of a release is one board for one day | `CL-OTA-009` |
| FR-OTA-08 | The system **MUST** record which release reached which display, when, by whom, and with what outcome | `PROPOSED` | Without it, "which boards are broken" is a walk-round | `CL-OTA-015`, `CL-OTA-021` |
| FR-OTA-09 | The system **MUST** re-enrol a reimaged or replaced display onto the last approved release | `PROPOSED` | Boards are reimaged and returned from repair without warning | `CL-OTA-022` |
| FR-OTA-10 | The system **MUST NOT** require the update infrastructure to be reachable for the platform to remain usable | `CONFIRMED` | A school may be isolated for weeks. Offline capability is not optional | `CL-OTA-014`, `CL-SEC-021` |
| FR-OTA-11 | The system **MUST** document the full update path — how updates reach the host, who applies them, how long they take, and what happens on failure | `PROPOSED` | An offline-capable system with no patch path is less secure, not more | `CL-SRV-019`, `CL-OTA-018`, `CL-OPS-021` |
| FR-OTA-12 | The system **MUST** define completion checks that include the storage mount, not only the database | `PROPOSED` | A healthy database on an unmounted NAS renders search and browsing silently dead | `CL-OTA-008`, `CL-NAS-002` |
| FR-OTA-13 | OTA self-update of the **backend host** is **OUT OF SCOPE** for the first release | `OUT OF SCOPE` | §6 #4. It requires release signing infrastructure that does not exist | `CL-OTA-004`, `CL-OTA-005` |
| FR-OTA-14 | The system **MUST NOT** require elevated Windows privilege on a display to continue functioning | `PENDING ANSWER` | Whether an updating kiosk agent can be installed is a school IT policy question | `CL-OTA-010`, `CL-OTA-011` |

### 2.10 SHANA Integration

| ID | Requirement | Class | Rationale | Depends on questionnaire |
|---|---|---|---|---|
| FR-SHANA-01 | CloudLight **MUST NOT** send motor, path, speed, actuator or localisation commands | `CONFIRMED` | SHANA's [ADR-001](../shana/architecture/architecture_decisions/ADR-001-no-ai-reaches-the-robot.md) (`D-005`) and the SHANA ownership boundary. Non-negotiable and not revisable by either team | — |
| FR-SHANA-02 | CloudLight **MUST** send high-level delivery intent only — *what*, *to where*, *by when* | `CONFIRMED` | The interface boundary is defined by what may cross it | `CL-SHANA-002` |
| FR-SHANA-03 | SHANA **MUST** independently validate safety, and CloudLight **MUST NOT** assert or influence that validation | `CONFIRMED` | Safety ownership never transfers. CloudLight is not qualified to make this judgement | `CL-SHANA-003` |
| FR-SHANA-04 | CloudLight **MUST** operate fully when SHANA is unreachable | `CONFIRMED` | The robot is a subsystem, not a dependency. A stalled integration must never stall a lesson | `CL-DASH-022`, `CL-SEC-017` |
| FR-SHANA-05 | CloudLight **MUST NOT** treat an `UNKNOWN` robot status as a successful or in-progress one | `CONFIRMED` | An invented status on a classroom wall is the worst available failure | `CL-SEC-017` |
| FR-SHANA-06 | Every delivery request **MUST** carry a unique idempotency key, and a retry **MUST NOT** create a second delivery | `PROPOSED` | A duplicated physical delivery is a real failure with a real object | `CL-SEC-016` |
| FR-SHANA-07 | SHANA **MUST** verify the request originated from this school's CloudLight instance | `PENDING ANSWER` | Mutual authentication on a robot link is a P0 and is unresolved | `CL-SHANA-015`, `CL-SEC-016` |
| FR-SHANA-08 | The robot link **MUST** be a network path distinct from the classroom LAN, sharing no broadcast domain | `CONFIRMED` | Sharing a broadcast domain produces a fault that looks like a network fault and is very hard to diagnose | [`../shana/infrastructure/school_network.md`](../shana/infrastructure/school_network.md) |
| FR-SHANA-09 | The display **MUST** show delivery status as a non-blocking element that never obscures or replaces the lesson | `PROPOSED` | A delivery widget that competes with the lesson is a design failure regardless of its correctness | `CL-SHANA-005`, `CL-DASH-004` |
| FR-SHANA-10 | CloudLight **MUST NOT** present a delivery as complete without a status from SHANA | `CONFIRMED` | "Complete" inferred locally is a safety-adjacent falsehood | `CL-SHANA-008` |
| FR-SHANA-11 | SHANA integration **MUST** be built last, after the core platform is demonstrated working without it | `PROPOSED` | The robot link adds a network path, an authentication problem and a failure mode to a product that has none of those yet | `CL-SHANA-*` |
| FR-SHANA-12 | CloudLight **MUST** send no student-identifying information in any delivery request | `CONFIRMED` | Fixed by the no-identification invariant | `CL-SEC-010`, `CL-SHANA-*` |

---

## 3. Non-functional requirements

> ### ⚠️ **Every target in the "Target" column is a proposal. None is a requirement yet.**
> Each is our engineering estimate for what a school could plausibly live with. Each
> is attached to the question that would confirm, raise or replace it. A target that
> nobody agreed to is not a requirement; it is a guess that would be expensive to
> discover late.

### 3.1 Availability and continuity

| ID | Requirement | Target *(proposal)* | Class | Notes |
|---|---|---|---|---|
| NFR-AVL-01 | The dashboard is available for the whole teaching day, every teaching day | 100% of powered-network hours | `PROPOSED` | Bounded by [`network_and_hosting.md`](network_and_hosting.md) §7.2 — a week with the uplink unplugged must still work |
| NFR-AVL-02 | Disks do not fill and stop the service | < 80% used on the data volume | `PROPOSED` | Confirms `CL-SRV-013`, `CL-OPS-016` |
| NFR-AVL-03 | An unattended host recovers from power return without a human | Automatic, or within one working day | `PENDING ANSWER` | Depends on UPS and on who restarts it — `CL-SRV-003`, `CL-SRV-011` |
| NFR-AVL-04 | No update or maintenance may interrupt a scheduled lesson | Zero, without exception | `PENDING ANSWER` | `CL-OTA-018`, `CL-OPS-020` |
| NFR-AVL-05 | Downtime is visible to a person, not discovered by a teacher | An alert reaches a named person | `PENDING ANSWER` | `CL-NET-020`, `CL-OPS-007`, `CL-OPS-008` |

### 3.2 Performance

| ID | Requirement | Target *(proposal)* | Class | Notes |
|---|---|---|---|---|
| NFR-PERF-01 | Current-lesson view renders and is legible | ≤ 2 s from page load | `PROPOSED` | `CL-UX-011` asks when a teacher stops waiting. **2 s is our guess, not their answer** |
| NFR-PERF-02 | Countdown updates without a visible jump | ~1 Hz, from a synced clock | `PROPOSED` | Must not be derived from network round-trips |
| NFR-PERF-03 | A substitution change is visible on affected displays | ≤ 5 s from approval | `PROPOSED` | `CL-SUB-023`, `CL-SEC-014` set the real tolerance |
| NFR-PERF-04 | A keyword search returns usable results | ≤ 1 s at the 95th percentile | `PROPOSED` | Depends entirely on corpus size, which is unknown — `CL-SRV-015`, `CL-OPS-017` |
| NFR-PERF-05 | The service does not stall during a period change or an index rebuild | No user-visible stall | `PROPOSED` | `CL-SRV-021`, `CL-OPS-001` |
| NFR-PERF-06 | Concurrent users at Monday-morning peak | **Unknown — deliberately not estimated** | `PENDING ANSWER` | ⚠️ We refuse to invent this. `CL-SRV-021`, `CL-DASH-001`, `CL-AUTH-008` |

### 3.3 Offline behaviour

| ID | Requirement | Target *(proposal)* | Class | Notes |
|---|---|---|---|---|
| NFR-OFF-01 | Core operation does not require the public internet | **Zero** — hard constraint | `CONFIRMED` | Timetable, substitution, countdown, browsing, keyword search |
| NFR-OFF-02 | The display shows today's lesson from cache when the host is unreachable | Last successfully synced data, visibly timestamped | `PENDING ANSWER` | How much must survive is `CL-SEC-021` |
| NFR-OFF-03 | Staleness on screen becomes visually unambiguous beyond a stated age | Configurable threshold, on screen | `PROPOSED` | `CL-SEC-018`, `CL-DASH-013` |
| NFR-OFF-04 | The service remains fully usable with the update infrastructure unreachable | Unlimited duration in principle | `CONFIRMED` | `CL-OTA-014` |
| NFR-OFF-05 | Offline capability **MUST NOT** be used as a reason to skip patching | A documented, tested update path exists | `CONFIRMED` | See [`network_and_hosting.md`](network_and_hosting.md) §7.3 — offline-first is a resilience property, not a security posture |

### 3.4 Recovery

| ID | Requirement | Target *(proposal)* | Class | Notes |
|---|---|---|---|---|
| NFR-REC-01 | Recovery time objective | Hours, not days | `PENDING ANSWER` | `CL-DATA-011` — could legitimately be "by the next working day" |
| NFR-REC-02 | Recovery point objective | Last complete backup | `PENDING ANSWER` | `CL-SRV-018`, `CL-DATA-008` |
| NFR-REC-03 | A backup is proven only by a successful restore onto a clean machine | At least once, on a schedule | `PROPOSED` | A backup that has never been restored is a file, not a plan |
| NFR-REC-04 | A non-technical member of staff can restore service from written instructions | Tested, with a named person | `PENDING ANSWER` | `CL-OPS-015`, `CL-SRV-017`, `CL-DATA-012` |
| NFR-REC-05 | The database survives a power cut mid-write without a partial record | Transaction atomic; corruption possible at sector level | `CONFIRMED` | Vendor-documented behaviour with a documented limit; see [`network_and_hosting.md`](network_and_hosting.md) §9.4 |

### 3.5 Accessibility

| ID | Requirement | Target *(proposal)* | Class | Notes |
|---|---|---|---|---|
| NFR-ACC-01 | Legible at the back of the room | **Unknown — deliberately not estimated** | `PENDING ANSWER` | ⚠️ We do not know the room size, the screen size or the viewing distance. `CL-UX-001`, `CL-DASH-016`, `CL-DASH-018` |
| NFR-ACC-02 | Text/background contrast meets a published ratio | 4.5:1 body text, 3:1 large text | `PROPOSED` | A widely used general threshold. ⚠️ **Not** a substitute for `CL-UX-003` (sunlight) or `CL-UX-004` (school policy) |
| NFR-ACC-03 | Interactive targets are large enough for the youngest student who touches the board | ≥ 44 × 44 px logical | `PENDING ANSWER` | A common general threshold. The age that matters is `CL-UX-007`, `CL-UX-008` |
| NFR-ACC-04 | Every action is available by tap alone | Zero hover dependence | `PROPOSED` | `CL-UX-013`. Hover is unusable on a touchscreen and undiscoverable for a new user |
| NFR-ACC-05 | Colour is never the sole carrier of meaning | Every state distinguishable in greyscale | `PROPOSED` | `CL-UX-005`, `CL-UX-006`. Also survives printing and photocopying |
| NFR-ACC-06 | The display is fully operable by keyboard alone | All controls reachable and visible | `PROPOSED` | `CL-UX-015` — some teachers use boards without touching them |
| NFR-ACC-07 | Error and empty states use plain language | No technical terms | `PROPOSED` | `CL-UX-018` |
| NFR-ACC-08 | Required languages and scripts render correctly | **Unknown — not estimated** | `PENDING ANSWER` | `CL-UX-016`, `CL-UX-017`. Script coverage is a hard limit of many components |

### 3.6 Security

| ID | Requirement | Target *(proposal)* | Class | Notes |
|---|---|---|---|---|
| NFR-SEC-01 | TLS between display, backend and storage, even on the LAN | Required, with a certificate the display trusts | `PENDING ANSWER` | `CL-SEC-024`, `CL-AUTH-010`. A self-signed certificate the board does not trust is worse than none |
| NFR-SEC-02 | An internal CA removes the offline certificate-expiry failure mode | Internal CA, if the school has one | `PENDING ANSWER` | `CL-SEC-024`, `CL-OPS-018`. This is the single highest-leverage unknown in the list |
| NFR-SEC-03 | Session idle timeout and absolute timeout | Idle ≤ 15 min; absolute ≤ 8 h | `PROPOSED` | ⚠️ **Proposals only.** `CL-AUTH-016`, `CL-SEC-002`. Teachers in a staff room will object to 15 minutes; we propose it anyway and record why |
| NFR-SEC-04 | Access is decided server-side on every request | No client-side-only checks | `CONFIRMED` | Documented anti-pattern; adopted in [`security_and_privacy.md`](security_and_privacy.md) |
| NFR-SEC-05 | The audit log is append-only with tamper evidence | Required | `PENDING ANSWER` | `CL-SEC-019` |
| NFR-SEC-06 | No secret in Git, in a URL, in a log, or in local storage | Zero tolerance | `CONFIRMED` | §4. `CL-SEC-006`, `CL-GIT-014` |
| NFR-SEC-07 | The school LAN is treated as untrusted | Application-layer controls regardless of network location | `CONFIRMED` | §4. `CL-NET-001`, `CL-SEC-023` |
| NFR-SEC-08 | Administration interfaces are unreachable from a student-facing segment | Denied by default | `PROPOSED` | `CL-SEC-023`. Compensate in the application if the network cannot be changed |
| NFR-SEC-09 | Failed authentication attempts are rate-limited | Threshold per school policy | `PENDING ANSWER` | `CL-SEC-003`, `CL-AUTH-016` |

### 3.7 Privacy

| ID | Requirement | Target *(proposal)* | Class | Notes |
|---|---|---|---|---|
| NFR-PRV-01 | No external AI service receives school content | Default off, code-level | `CONFIRMED` | §4. `CL-SEC-011` |
| NFR-PRV-02 | No student-identifying data in any early pilot | Zero | `PROPOSED` | A pilot that cannot succeed without student data should not run yet |
| NFR-PRV-03 | No student data on a student-facing display | Zero | `CONFIRMED` | `CL-DASH-003`, `CL-SEC-010` |
| NFR-PRV-04 | Every data category has a stated retention period | All of them | `PENDING ANSWER` | `CL-SEC-025`, `CL-DATA-015`, `CL-DATA-016`, `CL-SRV-006` |
| NFR-PRV-05 | Data is deletable on schedule, **including in backups** | All categories | `PENDING ANSWER` | `CL-DATA-016`, `CL-DATA-020` |
| NFR-PRV-06 | Logs record access without recording content or credentials | No content, ever | `CONFIRMED` | `CL-OPS-010`, `CL-OPS-011`, `CL-SEC-020` |

### 3.8 Scalability

| ID | Requirement | Target *(proposal)* | Class | Notes |
|---|---|---|---|---|
| NFR-SCA-01 | Number of classroom displays supported | **Unknown — deliberately not estimated** | `PENDING ANSWER` | ⚠️ We do not know how many rooms exist. `CL-DASH-001`, `CL-DASH-018` |
| NFR-SCA-02 | Adding a display is a configuration change, not a deployment | Minutes, by an administrator | `PROPOSED` | `CL-OTA-022`, `CL-DATA-006` |
| NFR-SCA-03 | Storage growth is monitored before it becomes an outage | Alert below the free-space threshold | `PROPOSED` | `CL-OPS-016`, `CL-SRV-015` |
| NFR-SCA-04 | One host serves all displays | Single host until a trigger in [`deployment_options.md`](deployment_options.md) §6 fires | `CONFIRMED` | ⚠️ A single host is a **single point of failure** and is accepted deliberately, not overlooked |

### 3.9 Maintainability and locality

| ID | Requirement | Target *(proposal)* | Class | Notes |
|---|---|---|---|---|
| NFR-MNT-01 | The school can continue without the project team | Documented handover, named owner | `PENDING ANSWER` | ⚠️ **This is the requirement most likely to be quietly dropped.** `CL-OPS-022`, `CL-BUD-020` |
| NFR-MNT-02 | Common failures are diagnosable from written instructions by non-technical staff | Tested, not assumed | `PROPOSED` | `CL-OPS-006`, `CL-OPS-015` |
| NFR-MNT-03 | Administrative workflows are documented in plain language | All of them | `PROPOSED` | `CL-UX-018`, `CL-BUD-020` |
| NFR-MNT-04 | Time zone is configurable and recorded with all data | School local time, explicitly | `PROPOSED` | `CL-DASH-011`. An Indian school spans `Asia/Kolkata` and may have staff on other clocks |
| NFR-MNT-05 | Licence obligations are known before deployment | Documented | `PENDING ANSWER` | `CL-BUD-005`, `CL-BUD-004` |

---

## 4. Constraints

> ### ⚠️ **These are binding on CloudLight regardless of any answer.**
> A constraint follows from a decision already accepted in this repository, from a
> documented platform limit, or from a safety boundary. Constraints are not
> requirements to be negotiated — they are the walls inside which requirements are
> written.

| # | Constraint | Source of the constraint | What it forecloses |
|---|---|---|---|
| **C1** | **Local-only operation by default.** Core features do not require the public internet | [`network_and_hosting.md`](network_and_hosting.md) §7.2 | Any design where a lesson depends on an external service |
| **C2** | **No external AI service without written authorisation**, for that specific service, from a named person | `CL-SEC-011`, `CL-GEN-004` | "We will try it and see". Not permitted |
| **C3** | **CloudLight never controls robot motors.** No motor, path, speed, actuator or localisation command may cross the boundary | [`shana_integration.md`](shana_integration.md); CloudLight ADR-014; SHANA `D-005` *(indexed as ADR-001)* | Any CloudLight-side safety reasoning about the robot. CloudLight is not qualified to make it |
| **C4** | **SHANA independently validates safety.** Safety ownership does not transfer | `CL-SHANA-003` | CloudLight asserting, pre-empting or inferring a safety judgement |
| **C5** | **No secret in Git, in a URL, in a log, or in browser local storage** | `CL-SEC-006`, `CL-GIT-014` | Convenience. A secret in version control must be assumed compromised |
| **C6** | **A school LAN is not trusted.** It is an internal network shared with student devices | [`network_and_hosting.md`](network_and_hosting.md) §6.1 | Relying on "it's internal" as a security control |
| **C7** | **The classroom display is read-only and unauthenticated** | [`classroom_dashboard.md`](classroom_dashboard.md); `CL-SEC-001` | Any write path from a display; any session held open on a board |
| **C8** | **Publication of timetable and substitution data requires human approval** | `CL-SUB-003`, `CL-OTA-019` | Fully automatic publication. Note this is a *constraint on publication*, not on proposal — machine drafts remain permitted |
| **C9** | **Software and data updates use separate approval paths** | `CL-OTA-001`, `CL-OTA-019` | A single "deploy everything" pipeline |
| **C10** | **No student-identifying data on a student-facing display or in a delivery request** | [`../shana/safety/student_privacy.md`](../shana/safety/student_privacy.md) | Individualised dashboards. Period |
| **C11** | **Scanning, probing or port-scanning the school network is prohibited** | [`../shana/infrastructure/school_network.md`](../shana/infrastructure/school_network.md) | Resolving any network unknown by ourselves. IT checks what needs checking |
| **C12** | **The classroom smartboard is a client, never the server** | [`deployment_options.md`](deployment_options.md) §2 option (d) | Peer-hosted and board-hosted designs, which also fail on client isolation |

---

## 5. Data requirements

### 5.1 Entities the system must model

| # | Entity | Purpose | Status |
|---|---|---|---|
| 1 | **Academic year** | Bounds every timetable and substitution record; drives expiry | ❓ Rollover rule unknown — `CL-TIME-014` |
| 2 | **Class / section** | The unit a timetable is written against | ❓ Streams unknown — `CL-TIME-011` |
| 3 | **Room** | The unit a display is bound to | ❓ Room inventory unknown — `CL-DASH-001`, `CL-TIME-010` |
| 4 | **Teacher** | The unit a substitution is written against | ❓ Identity source unknown — `CL-AUTH-001`, `CL-AUTH-002` |
| 5 | **Period / day profile** | Period count, durations, morning/afternoon blocks | ⛔ **Cannot be modelled without an answer** — `CL-TIME-005`, `CL-TIME-006`, `CL-TIME-007`, `CL-TIME-008` |
| 6 | **Timetable slot** | (class, period, weekday or date) → subject, teacher, room | ⛔ Depends on the day model — `CL-TIME-008`, `CL-TIME-009` |
| 7 | **Substitution record** | With explicit states `scheduled` / `proposed` / `cancelled` / `room-changed` | 🧪 States proposed; the workflow is `CL-SUB-*` |
| 8 | **Source document** | The artefact a derived record came from, retained | 🧪 Proposed. Retention `CL-SRV-006` |
| 9 | **Approval record** | Who approved what, when, and on what basis | 🧪 Proposed. Separation `CL-SUB-002`, `CL-OTA-003` |
| 10 | **Version** | Immutable timetable history with rollback | ❓ Whether history is wanted at all — `CL-DATA-017` |
| 11 | **Resource metadata** | Title, path, type, size, dates, source ACL identity | 🧪 Depends on the storage model — `CL-RES-*`, `CL-NAS-*` |
| 12 | **Audit event** | Actor, timestamp, action, object, outcome | 🧪 Proposed. Append-only is `CL-SEC-019` |
| 13 | **Academic calendar** | Holidays, term breaks, special days | ❓ Unknown and P0 — `CL-TIME-013`, `CL-TIME-015` |
| 14 | **Device registration** | Which display is bound to which room | 🧪 Proposed — `CL-OTA-022` |

### 5.2 The entity that blocks the most

> ### ⚠️ **Entity 5 — the period and day profile — blocks entities 6, 7, 9 and 13, and therefore most of §2.**
>
> `CL-TIME-005` (how many periods, of what length, are there morning and afternoon
> blocks) is **P0** and `UNANSWERED`. Until it is answered, the countdown logic,
> the substitution matcher and the display scheduler cannot be specified at all —
> only argued about. Filling this in with a conventional six- or eight-period day
> would be inventing a school.

### 5.3 Entities that must NOT exist

| Must not be modelled | Why |
|---|---|
| A per-student record | Fixed by the no-identification invariant, C10 |
| A student's activity or access history | Fixed by [`../shana/safety/student_privacy.md`](../shana/safety/student_privacy.md) |
| A second authoritative timetable copy | Two live copies diverge — `CL-TIME-001`, `CL-DATA-019` |
| An unversioned "current timetable" with no history | Unattributable corrections — `CL-DATA-017` |

---

## 6. Explicitly out of scope for the first release

Each exclusion is deliberate. Each names the question that would unblock it, so
that none of them becomes a silent omission.

| # | Excluded from release 1 | Reason for exclusion | What would unblock it |
|---|---|---|---|
| 1 | **Semantic / vector search** | Requires a model, hardware and a corpus whose size is unknown. Keyword search covers the same need at a fraction of the cost | `CL-AI-009`, `CL-SEARCH-*`, `CL-SRV-015`, plus a corpus size |
| 2 | **AI assistant** | The largest unverified dependency in the project. Human review cost arrives before the benefit does, and the accuracy of an assistant over school documents is untested here | `CL-AI-009` (GPU), `CL-AI-023` (latency tolerance), `CL-BUD-006`, `CL-SEC-011` |
| 3 | **OCR of handwritten boards** | Vendor-documented accuracy figures do not transfer to this school's handwriting, lighting and board types. Script coverage is a hard limit | `CL-AIML-010`, `CL-SUB-006`, `CL-SUB-007`, `CL-UX-016` |
| 4 | **OTA self-update of the backend host** | Requires release-signing infrastructure, a key custody policy and staged rollout — none exists | `CL-OTA-004`, `CL-OTA-005`, `CL-OTA-009`, `CL-OTA-011` |
| 5 | **SHANA task submission** | Robot-side authentication is an unresolved P0; the interface is a proposal checked against no vendor SDK | `CL-SHANA-015`, `CL-SEC-016`, `CL-SHANA-008` |
| 6 | **Mobile apps** | Multiplies client maintenance by an unknown device count for no release-1 classroom benefit | `CL-DATA-004`, `CL-AUTH-018`, `CL-BUD-004` |
| 7 | **Multi-school support** | One school, one academic year, one set of answers. Multi-tenancy before a single tenant is proven is a classic inversion | `CL-BUD-018`, and a second school that wants it |
| 8 | **Writing back to the school's authoritative timetable system** | Two-way sync creates an ownership problem we have no answer to | `CL-TIME-001`, `CL-DATA-019` |

---

## 7. Traceability

> ### ⚠️ **This section exists so that nothing is quietly dropped.**
> Every capability in the original proposal appears below, mapped to the
> requirements that implement it and to the questionnaire categories that gate it.
> A capability with no requirement ID would be a capability that was forgotten.

| # | Capability from the brief | Requirement IDs | Gating questionnaire categories | Out of scope? |
|---|---|---|---|---|
| 1 | **Classroom displays** | FR-DASH-01…15, NFR-ACC-01…08, NFR-OFF-02…03 | `CL-DASH-*`, `CL-UX-*`, `CL-WIN-*` | No |
| 2 | **Lesson countdown** | FR-DASH-04, FR-DASH-05, NFR-PERF-02 | `CL-DASH-008`, `CL-DASH-009`, `CL-DASH-010`, `CL-DASH-011`, `CL-TIME-006` | No |
| 3 | **Degraded and offline states** | FR-DASH-06…09, NFR-OFF-01…05, FR-AI-12 | `CL-DASH-012`, `CL-SEC-018`, `CL-SEC-021`, `CL-SEC-022`, `CL-OTA-014` | No |
| 4 | **Timetable ingestion** | FR-TIME-01…06, FR-TIME-11…14 | `CL-TIME-001`…`CL-TIME-016`, `CL-DATA-019` | No |
| 5 | **Timetable versioning and rollback** | FR-TIME-07…09, FR-ADMIN-07, FR-ADMIN-10 | `CL-DATA-017`, `CL-DATA-018`, `CL-OTA-020` | No |
| 6 | **Substitution proposal** | FR-SUB-02…04, FR-SUB-09…10, FR-AI-03 | `CL-SUB-005`…`CL-SUB-019`, `CL-AIML-*` | No |
| 7 | **Substitution approval and publication** | FR-SUB-01, FR-SUB-05, FR-SUB-11, FR-ADMIN-03…05, C8, C9 | `CL-SUB-001`…`CL-SUB-003`, `CL-SUB-020`, `CL-OTA-003` | No |
| 8 | **Substitution rollback** | FR-SUB-05, FR-SUB-06, NFR-PERF-03 | `CL-SUB-004`, `CL-SUB-023`, `CL-SEC-014`, `CL-SEC-015` | No |
| 9 | **NAS resource browsing** | FR-RES-01…15, NFR-SCA-02 | `CL-NAS-*`, `CL-RES-*`, `CL-SRV-014` | No |
| 10 | **Resource search** | FR-SEARCH-01…12 | `CL-SEARCH-*`, `CL-RES-*`, `CL-SEC-007` | No |
| 11 | **AI assistant** | FR-AI-01…13 | `CL-AI-*`, `CL-SEC-009`…`CL-SEC-011` | ⛔ **Yes — release 1** (§6 #2) |
| 12 | **Teacher authentication** | FR-AUTH-01…14, FR-DASH-02, FR-DASH-14, NFR-SEC-03…09 | `CL-AUTH-*`, `CL-CODE-*`, `CL-SEC-002`…`CL-SEC-006` | No |
| 13 | **Administration and roles** | FR-ADMIN-01…10, NFR-SEC-08 | `CL-ADMIN-*`, `CL-AUTH-024`, `CL-SEC-023` | No |
| 14 | **OTA / update management** | FR-OTA-01…14, NFR-OFF-04, NFR-MNT-05 | `CL-OTA-*`, `CL-SRV-019`, `CL-OPS-021` | ⚠️ **Partly** — backend self-update excluded (§6 #4) |
| 15 | **SHANA delivery-request interface** | FR-SHANA-01…12, FR-DASH-13, FR-DASH-14 | `CL-SHANA-*`, `CL-SEC-016`, `CL-SEC-017` | ⛔ **Yes — release 1** (§6 #5) |
| 16 | **Backup, recovery and handover** | NFR-REC-01…05, NFR-MNT-01…03, NFR-AVL-03 | `CL-DATA-008`…`CL-DATA-016`, `CL-SRV-016`…`CL-SRV-018`, `CL-OPS-012`…`CL-OPS-022`, `CL-BUD-013`, `CL-BUD-020` | No |

### 7.1 Reading the table honestly

| Observation | What it means |
|---|---|
| 2 of 16 capabilities are excluded from release 1 | Both are the **highest-visibility** ones. A release without an AI assistant and without robot delivery is a smaller product than the brief describes |
| Capability 16 is not a feature | Backup, recovery and handover have no user interface and no demo. They are also the requirements most likely to be skipped when a project is under time pressure |
| Fourteen capabilities have `PENDING ANSWER` requirements inside them | A capability can be in scope and still be unspecifiable. That is the normal state of this project |
| Every capability is gated by at least one `P0` question | None of these sixteen can start without an answer from a named person at the school |

⚠️ **No capability in the table above has been validated by anyone at the school.**

---

**Related documents:** [`network_and_hosting.md`](network_and_hosting.md) ·
[`classroom_dashboard.md`](classroom_dashboard.md) ·
[`timetable_and_substitutions.md`](timetable_and_substitutions.md) ·
[`resource_browser.md`](resource_browser.md) · [`resource_search.md`](resource_search.md) ·
[`ai_assistant.md`](ai_assistant.md) · [`authentication_and_authorisation.md`](authentication_and_authorisation.md) ·
[`security_and_privacy.md`](security_and_privacy.md) · [`ota_updates.md`](ota_updates.md) ·
[`shana_integration.md`](shana_integration.md) · [`deployment_options.md`](deployment_options.md) ·
[`sources.md`](sources.md)

**Requirements written: 186 (133 functional, 53 non-functional).**
**By class — `CONFIRMED`: 58 · `PROPOSED`: 81 · `PENDING ANSWER`: 45 · `OUT OF SCOPE`: 2.**
**Capabilities covered by §7 traceability: 16 of 16.**
**Requirements agreed by the school: zero. Systems built: zero. Tests executed: zero.**