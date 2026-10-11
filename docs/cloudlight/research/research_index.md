# CloudLight — Research Index

**Phase 0 · Documentation only · Register compiled 2026-10-11**

This folder holds the research register for CloudLight. It is a **record of what
was researched, on what basis, and how well it is established** — not a place for
new findings.

> ### ⚠️ **There is no application source code in this repository.**
>
> No JavaScript, TypeScript, Python, Go or Rust. No `package.json`, no build
> files, no tests. Every CloudLight document is design work. Every statement
> about behaviour is a statement about **documentation and reasoning**, never
> about a running system, because there is no running system.

> ### ⚠️ **No claim in this register has been verified on school equipment.**
>
> Network probes: zero. Smartboards touched: zero. NAS shares mounted: zero.
> AI services contacted: zero. Models downloaded or run: zero. Robot operated:
> zero. School documents read: zero. Hosts deployed: zero.

---

## 1. What is in this folder

| Document | What it is for | Read it when |
|---|---|---|
| **`research_index.md`** *(this file)* | The map. What each document here is for, which research ID covers which domain, and the rules for extending the register | First. Before adding anything |
| [`source_register.md`](source_register.md) | The detailed register. One block per research thread with all 17 required fields — what was read, what it establishes, what it does **not** establish, and the action that would upgrade or retire it | When you need to know whether a claim is sourced, and how solidly |
| [`research_log.md`](research_log.md) | The chronological record. What research was actually performed, in order, with method, result, and what remained unknown. Including the things that were *not* done | When you need to know what has already been attempted, so you do not repeat it |
| [`claim_evidence_matrix.md`](claim_evidence_matrix.md) | Claim by claim, the implementation status, the verification status, and — unambiguously — whether the claim may appear in an exhibition proposal | Before anything is said out loud about CloudLight to anyone outside the team |

⚠️ **The four are deliberately separate.** `source_register.md` answers *where
did this come from*. `research_log.md` answers *when and how was this found*.
`claim_evidence_matrix.md` answers *what may we say*. Conflating them is how a
citation starts being read as a capability.

---

## 2. Domain index

| Domain | Research IDs | Primary documents |
|---|---|---|
| **Windows & device management** | `CR-001` … `CR-005` | [`../windows_startup.md`](../windows_startup.md) · [`../requirements.md`](../requirements.md) · [`../feasibility_report.md`](../feasibility_report.md) §16 |
| **Networking & storage** | `CR-006` … `CR-012` | [`../network_and_hosting.md`](../network_and_hosting.md) · [`../resource_browser.md`](../resource_browser.md) §2–§3 |
| **Search & indexing** | `CR-013` … `CR-016` | [`../resource_search.md`](../resource_search.md) · [`../resource_browser.md`](../resource_browser.md) §9 |
| **AI inference & RAG** | `CR-017` … `CR-021` | [`../ai_assistant.md`](../ai_assistant.md) · [`../network_and_hosting.md`](../network_and_hosting.md) §4.2 |
| **Document parsing / OCR** | `CR-022` … `CR-026` | [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §2–§4 |
| **Authentication & sessions** | `CR-027` … `CR-033` | [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) · [`../security_and_privacy.md`](../security_and_privacy.md) §6 |
| **Update security** | `CR-034` | [`../ota_updates.md`](../ota_updates.md) §5 |
| **Accessibility** | `CR-035` | [`../requirements.md`](../requirements.md) §3.5 — ⚠️ **the gap, not a source** |
| **Internal repository documents** | `CR-036` … `CR-041` | [`../sources.md`](../sources.md) §2.9 · [`../decision_log.md`](../decision_log.md) · [`../shana_integration.md`](../shana_integration.md) |

---

## 3. The research ID scheme

| Rule | Why |
|---|---|
| Every research thread gets a **`CR-nnn`** identifier | So a claim can name the research behind it without repeating a URL |
| IDs are **sequential** and start at `CR-001` | Gaps in the sequence mean a deleted entry; there are none and there should not be |
| **An ID is never reused.** If a source is retired, the ID stays with a new `Confidence` and a new `Next verification step` | A reused ID would make an old citation silently mean something else |
| **Every new research task gets a new ID in the same working session** that produced it | A register updated weeks later is a register nobody trusts |
| An ID covers a **research thread** — one question, possibly several sources | One thread against four vendor pages is one row, not four |

⚠️ **An ID is not a confidence level and not a priority.** `CR-001` being first
does not make it better. What a claim is worth comes from its `Confidence` field
in [`source_register.md`](source_register.md), never from its number.

### 3.1 How to add a research thread

1. Open [`source_register.md`](source_register.md), find the highest `CR-nnn`.
2. Add `CR-` + one higher, in the correct domain group.
3. Fill **all 17 fields.** A field you cannot fill is filled with an honest
   statement of what is missing — never with a plausible value.
4. Add the domain to the table in §2 above if it is new.
5. Add an entry to [`research_log.md`](research_log.md) covering what was done.
6. If the thread settles a claim that will be said out loud, add it to
   [`claim_evidence_matrix.md`](claim_evidence_matrix.md) with its status.

---

## 4. Two categories that must never be confused

> ### ⚠️ **Every school-specific item in CloudLight is currently an assumption, not a finding.**

| | **External documentation research** | **Assumption about this school** |
|---|---|---|
| **What it is** | What a vendor, a standards body or a paper publishes about systems **in general** | A belief about *this* school's hardware, network, data, people or policy |
| **Status** | `VERIFIED` where the publisher is authoritative | **Unknown.** Recorded as `❓ UNKNOWN` with a `CL-*` question |
| **Basis** | A citable document, read directly | Reasoning, analogy, or a question nobody has answered yet |
| **Example** | "Microsoft documents that `--kiosk-idle-timeout-minutes` does not relaunch Edge" | "The classroom boards run Windows 11 Pro" |
| **Example** | "Microsoft documents that `UWF_Filter` is unavailable on Home and Pro" | "The boards can be re-imaged to IoT Enterprise" |
| **Example** | "WatchGuard documents that client isolation blocks peer traffic" | "Client isolation is enabled on this school's SSID" |

⚠️ **The distinction is not academic.** `CR-003` establishes that Edge kiosk mode
runs an InPrivate session. It does **not** establish that CloudLight can cache
timetable data offline on these boards. The first is vendor documentation; the
second is a question (`CL-WIN-011`, `CL-WIN-007`). Quoting the first in support
of the second is the specific failure this folder exists to prevent.

⚠️ **And it is worse than that.** A general document about SMB deprecation does
not tell us what this school's NAS speaks. A paper about layout analysis trained
on financial documents does not tell us it can read a school timetable. A
framework's benchmark says nothing about hardware nobody owns.

**The governing rule:** *a source establishes general truth about a class of
systems. It never establishes a fact about this school. A school fact exists only
as an answer from a person at the school, recorded in writing.*

---

## 5. Related documents

| Document | Covers |
|---|---|
| [`../sources.md`](../sources.md) | The bibliography this register is built from — 86 numbered entries |
| [`../requirements.md`](../requirements.md) | `FR-*` and `NFR-*` identifiers referenced throughout |
| [`../decision_log.md`](../decision_log.md) | `D-0nn` decisions, provisional defaults and open decisions |
| [`../open_questions_register.md`](../open_questions_register.md) | Every `❓ UNKNOWN` and who owns it |
| [`../risk_register.md`](../risk_register.md) | Risks by domain |
| [`../feasibility_report.md`](../feasibility_report.md) | Every conclusion classified by evidence strength |
| [`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md) | All 576 `CL-*` questions — **every one currently `UNANSWERED`** |

⚠️ **CloudLight is the platform. SHANA is a robotics subsystem within it.** The
SHANA bibliography is a separate document and is not merged into this register.
Where a CloudLight claim depends on robot-side documentation, it is cited from
SHANA's own register.

⚠️ **Related internal artifacts reviewed read-only for this register:**
[`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md) ·
[`../../../data/schemas/README.md`](../../../data/schemas/README.md) ·
[`../../architecture/architecture_decisions/ADR-013-modular-monolith-not-microservices.md`](../../architecture/architecture_decisions/ADR-013-modular-monolith-not-microservices.md) ·
[`../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md`](../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md)

**Standing note.** Filling an `UNKNOWN` with a plausible default is the failure
mode this folder exists to prevent. When this folder is extended, the extension
must not make any unknown smaller.