# CloudLight — Assumptions

**Phase 0 · Documentation only · 2026-10-11**

**Every assumption the project is currently running on. None is verified against
this school.**

> **⚠️ There is no CloudLight code in this repository.** No smartboard has been
> touched, no school network has been probed, no NAS has been reached, no timetable
> has been read, no robot has been operated and no AI service has been contacted.
> Every row below is a statement the design *rests on*, not a statement about
> something observed.

An assumption is a statement treated as true so that work can proceed, when in
fact it is not established. Each carries **why we assume it**, **what breaks if
it is false**, a **cheap test**, and the `CL-*` question that would settle it.

⚠️ **An assumption that is never written down becomes an assumption that is never
tested.** This register exists so that cannot happen here. Companion documents:
[`constraints.md`](constraints.md) for the walls these assumptions sit inside, and
[`go_no_go_gates.md`](go_no_go_gates.md) for the checkpoints that close them.

**Evidence labels used throughout** (consistent with
[`../feasibility_report.md`](../feasibility_report.md)):

| Label | Meaning |
|---|---|
| ✅ `VERIFIED` | Confirmed in official vendor, standards or regulatory documentation, quoted with a source |
| 🟡 `DOCUMENTED_NOT_HARDWARE_VERIFIED` | A vendor documents it; not confirmed on this school's equipment |
| 🧪 `PLAUSIBLE_REQUIRES_TESTING` | Engineering judgement. No vendor claim |
| ❓ `UNKNOWN` | No evidence either way. Not assumed true or false |
| ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` | Rejected by our judgement, vendor guidance, or an accepted constraint |

---

## 1. Class A — Assumptions about the school's permission to host

The most expensive assumption class. Three of these can end the project outright
rather than change its shape.

| ID | Assumption | Why we assume it | If false, what breaks | Cheap test | Linked question | Status |
|---|---|---|---|---|---|---|
| **CA-001** | An **always-on host** is permitted — one school machine may run a persistent service through the working day | Every CloudLight design document is local-first; there is no non-local alternative that keeps school data on school premises | ⛔ **The entire platform has no home.** This becomes a different project, not a smaller one. A cloud VM is a scope change, not a fallback | Ask the Principal. Ten minutes, one conversation | `CL-SRV-001`, `CL-GEN-002` | ❓ UNKNOWN |
| **CA-002** | A **suitable physical machine exists** — spare, always-on, with a spare network port and a place to put it | The baseline deployment names option (a), an authorised existing always-on PC, as cheapest and most supportable | ⛔ No host → no backend → blank boards. Procurement becomes a budget conversation with an unknown ceiling | One site walk with IT or facilities | `CL-SRV-002`, `CL-SRV-007` | ❓ UNKNOWN |
| **CA-003** | Someone will **physically restart the host** during school hours after an unplanned shutdown | Assume we are not the on-call rota. Somebody must be, and naming them is a people decision not a technical one | ⚠️ An outage lasts until the next working day. In practice this alone ends teacher trust in a first pilot | Ask who. Five minutes | `CL-SRV-011`, `CL-OPS-006` | ❓ UNKNOWN |
| **CA-004** | The school accepts a **year-two owner** for this system — a person or a department | Most feasibility failures are abandonment, not technical failure | ⛔ **The project should not be built.** `CL-OPS-022` is described in the register as the single most important answer in the set | Ask the Principal. One decision | `CL-OPS-022` | ❓ UNKNOWN |

⚠️ **CA-001 and CA-004 are the two that stop the project rather than reshape it.**
Everything else in this register is a scope conversation.

---

## 2. Class B — Assumptions about the network

The failure mode here is **silent**. Nothing errors; the board simply never
loads. That is why these are cheap to test and expensive to discover late.

| ID | Assumption | Why we assume it | If false, what breaks | Cheap test | Linked question | Status |
|---|---|---|---|---|---|---|
| **CA-005** | **Classroom displays can reach the host** across the firewall and between subnets | The brief states all smartboards and computers can reach the same LAN or access point. We have taken that as intent, not as a finding | ⚠️ **Every display is blank.** Not one board, every board, at once, with no useful error | Ask IT for the firewall rule and the permitted source segment | `CL-NET-003`, `CL-NET-014`, `CL-NET-002` | ❓ UNKNOWN |
| **CA-006** | **Client / AP isolation is not enabled** on the wireless network the boards use | Isolation is common on managed and guest wireless; we assume the *permissive* case only because it is what the brief implies | ⚠️ **Silent total failure.** Boards associate, show full signal, and cannot reach any local host. Discovered on demonstration day | Read the AP configuration — from whoever administers it | `CL-NET-008` | ❓ UNKNOWN |
| **CA-007** | A **local DNS name and a stable address** can be created for the host | A memorable name is the cheapest maintenance tool available and hard-coded addresses turn every address change into a site visit | Hard-coded IPs everywhere; the host moves and every display needs a physical visit | Ask IT whether a DNS record and a DHCP reservation can be made | `CL-NET-004`, `CL-NET-006`, `CL-NET-012` | ❓ UNKNOWN |
| **CA-008** | A **TLS certificate the displays trust** can exist offline | HTTPS on a LAN is assumed in every security requirement. A self-signed certificate the board does not trust is worse than no TLS | Every board shows a security warning simultaneously; the dashboard dies everywhere at once | Ask IT whether an internal CA exists | `CL-SEC-024`, `CL-AUTH-010` | ❓ UNKNOWN |

⚠️ **CA-008 is the highest-leverage unknown in the security set** because it fails
*uniformly and instantly* across every device rather than gradually.

---

## 3. Class C — Assumptions about classroom devices

| ID | Assumption | Why we assume it | If false, what breaks | Cheap test | Linked question | Status |
|---|---|---|---|---|---|---|
| **CA-009** | **Boards support a kiosk lockdown** — Assigned Access or Shell Launcher | Microsoft documents both as the supported path to a locked-down fullscreen display; we have assumed a Pro/Enterprise/Education edition | ⛔ **The display must be opened by hand every morning.** Not an acceptable daily experience; the product is unusable as specified | `winver` on one non-critical board | `CL-WIN-001` | 🟡 DOCUMENTED_NOT_HARDWARE_VERIFIED |
| **CA-010** | **Kiosk accounts have local administrator rights** | Self-start, self-update and local repair all depend on it | ⚠️ No self-install, no self-update. Every configuration change becomes a site visit by one person. This is a *supportability* failure, not a security one | Ask IT | `CL-WIN-003` | ❓ UNKNOWN |
| **CA-011** | **Offline caching works inside a kiosk session** | Without it the fail-closed offline design has no substrate on the board | The degraded state becomes "a browser opened by hand, showing an error page". The boot race wins | Test on one board, one reboot, network unplugged | `CL-WIN-004` (kiosk session), `CL-DASH-012` | ❓ UNKNOWN |
| **CA-012** | **Displays are wired, or can be** | Wireless boards face isolation, roaming and signal questions; wired boards face cable questions. We have not assumed which | If wireless-only and isolation is on: ⛔ no local hosting is possible for those rooms | Look at one room | `CL-NET-005`, `CL-WIN-020` | ❓ UNKNOWN |
| **CA-013** | **One non-critical machine may be tested on**, on explicit permission from IT | Testing must happen somewhere. It cannot be a live classroom board unasked | ⛔ **The kiosk design stays hypothetical forever.** The single cheapest unblocking test in the project never runs | Ask IT to name one machine | `CL-WIN-007` | ❓ UNKNOWN |

---

## 4. Class D — Assumptions about school data and identity

| ID | Assumption | Why we assume it | If false, what breaks | Cheap test | Linked question | Status |
|---|---|---|---|---|---|---|
| **CA-014** | A **staff identity directory exists** — one list of teachers with stable identifiers | Permission-aware search and attributable audit both depend on knowing who a person is at query time | ⚠️ CloudLight becomes the system of record for teacher identity. Provisioning *and offboarding* become ours, and HR and CloudLight diverge silently | Ask IT or administration | `CL-AUTH-001`, `CL-AUTH-002` | ❓ UNKNOWN |
| **CA-015** | **A timetable authority exists** who can approve a substitution — a clerk, an office, a named person | Publication requires a named approver; a perfect pipeline with the wrong approver has no publication path | ⛔ **The core feature has no publication path.** The system can compute and cannot say anything | Ask administration | `CL-SUB-001`, `CL-TIME-001` | ❓ UNKNOWN |
| **CA-016** | **A master timetable exists in some form we can read** — a spreadsheet, a PDF, a portal | Every ingestion design begins from an input | Core feature has no input. If the timetable is paper only, manual keyed entry becomes the honest first version | Ask for the current master, in whatever form it is kept | `CL-TIME-001`, `CL-TIME-002`, `CL-TIME-003` | ❓ UNKNOWN |
| **CA-017** | **Publishing timetable and substitution data to classroom screens is already permitted practice** | The school already runs physical boards with this information | A feature that looks essential turns into a new permission that must be requested and defended | Ask whether the current board is approved practice | `CL-TIME-024`, `CL-DASH-001` | ❓ UNKNOWN |

---

## 5. Class E — Assumptions about storage

| ID | Assumption | Why we assume it | If false, what breaks | Cheap test | Linked question | Status |
|---|---|---|---|---|---|---|
| **CA-018** | **A shared storage system exists** — a NAS or file server with teaching material | Resource browsing, resource search and (through them) the assistant all assume somewhere to read from | ⚠️ **Resource browsing and search do not exist.** ✅ The rest of the product is unaffected. This is a *shape* change, not a project-ending one | Ask IT. Ten minutes | `CL-NAS-001`, `CL-NAS-002` | ❓ UNKNOWN |
| **CA-019** | **The resource corpus is small enough** that database-native full-text search is adequate | One relational database is the proposed default (ADR-013). A separate search engine is justified only by a *measured* size | ⚠️ The search engine decision must be revisited — but the *product* still works. Keyword search returns more slowly | A directory listing and a `du` on the share. Minutes | `CL-SEARCH-014`, `CL-NAS-003` | ❓ UNKNOWN |
| **CA-020** | **Storage permissions are per-user or per-group**, not folder-inherited only | Query-time permission filtering requires knowing which principals can reach a file | ⛔ **Per-teacher filtering is not safe.** Search over material a user cannot selectively access is not shippable | Export one share's ACL and read it | `CL-NAS-009`, `CL-NAS-011` | ❓ UNKNOWN |

⚠️ **CA-018 is the assumption most often mistaken for a fact.** The brief implies a
school with a NAS. A school may keep every teaching file on laptops, pen drives and
personal cloud drives, and *"no"* is a completely valid answer.

---

## 6. Class F — Assumptions about our own design decisions

These are assumptions *we* make about our own design. They are testable by
reading our own documents, which makes them the cheapest in this register to close.

| ID | Assumption | Why we assume it | If false, what breaks | Cheap test | Linked question | Status |
|---|---|---|---|---|---|---|
| **CA-021** | **A local model could run on school hardware, if local inference is ever needed** | It is the only AI path that survives a school refusing external services | The assistant is ⛔ **dropped, not weakened.** Core product unaffected — which is precisely why it is built last | None yet. Only meaningful after `CL-AI-001` returns | `CL-AI-001`, `CL-AI-009` | 🧪 PLAUSIBLE_REQUIRES_TESTING |
| **CA-022** | **External AI is not permitted** by default, and will not be assumed | Permission has not been sought. Assuming it would send school data to a third party on the strength of an assumption | If permission is *granted*, the assistant gains an option. ⚠️ It does **not** unblock the assistant — retrieval quality still gates it | Ask the Principal. One yes/no, in writing | `CL-AI-001`, `CL-SEC-011`, `CL-GEN-004` | ⛔ NOT_RECOMMENDED_WITHOUT_APPROVAL |
| **CA-023** | **Deterministic parsing handles most timetable sources**, and AI is only needed for genuinely unstructured input | `.xlsx` is a ZIP of XML; CSV is delimited text; a text-layer PDF yields tokens with coordinates. All readable exactly, repeatably, offline, free | If the school's master is scanned or handwritten, the pipeline shifts toward tiers 2–4 — 🧪 with materially lower accuracy and a higher human review cost | Ask what the master actually is | `CL-TIME-002`, `CL-SUB-005` | ✅ VERIFIED (format structure) · ❓ UNKNOWN (this school's format) |
| **CA-024** | **A modular monolith is operable by one IT generalist** with other duties | One school, a few dozen displays, one host, no operations team. Microservices solve an organisational scaling problem this project does not have | A system requiring a competent distributed-systems engineer to keep running, in a school with none | Count people, not processes | `CL-ADMIN-008`, `CL-WIN-020` | 🧪 PLAUSIBLE_REQUIRES_TESTING · ADR-013 |

---

## 7. Assumptions we have explicitly NOT made

Recorded so they are not silently reintroduced by a later reader who finds the
project unfamiliar. Each was considered and is **not** being relied upon.

| Assumption NOT made | Why it is not assumed | Where it is recorded |
|---|---|---|
| ❌ *"All smartboards can reach the same access point, therefore a backend can exist and devices can reach each other"* | An access point is a **Layer-2 bridge**. It forwards frames; it does not run a server, a database or an application. Shared connectivity establishes neither a compute host nor device-to-device reachability | ✅ VERIFIED. `network_and_hosting.md` §2; decision `X-02` |
| ❌ *"A six-digit classroom code identifies a teacher"* | 10⁶ possibilities, expected 500,000 guesses, a free period is 40–45 minutes. A shared code proves someone in the room knows six digits — it names nobody | ⛔ `X-07`; `CL-CODE-002`; [`../overview.md`](../overview.md) §4.2 |
| ❌ *"A photograph of a substitution board is a record"* | Visibility is not authority. A misread is structurally plausible and looks correct — a `7` for a `1`, a struck-out cell read as valid | ⛔ `X-06`; `CL-SUB-007` |
| ❌ *"A smartboard can host the platform"* | Boards sleep, restart mid-lesson, lock and are switched off. Re-imaging one would take down every classroom it serves | ⛔ `X-03`; `deployment_options.md` §2.5 |
| ❌ *"Access on the school LAN implies trust"* | It is an internal network shared with student devices. Treating it as trusted inverts the actual risk model | `constraints.md` C6; `security_and_privacy.md` |
| ❌ *"A model reading a spreadsheet cell is better than a library"* | A model cannot exceed the cell addresses it was handed, is slower, non-deterministic, costs money, works online, and cannot be reasoned about | ✅ VERIFIED (`timetable_and_substitutions.md` §3.1) |
| ❌ *"High confidence extraction can publish"* | Confidence is a property of the model, not of the school. Publishing on confidence grounds makes the model's opinion school policy | ⛔ `timetable_and_substitutions.md` §5.1 |
| ❌ *"More capable hardware justifies a more complex architecture"* | The scaling argument for microservices does not apply at this size. Adding operational concepts to a one-person estate is a cost with no matching benefit | ADR-013; `X-01` |
| ❌ *"An unreviewed statement anywhere in these documents is probably right"* | Every school-specific fact is a question with a named owner. A sentence that reads as a finding is a defect and should be corrected | [`open_questions_register.md`](../open_questions_register.md) §5 |

---

## 8. Review triggers

Re-open this register when any of the following occurs:

| Trigger | Why |
|---|---|
| A `P0` question is answered | At least one row above is now a fact rather than an assumption |
| A gate changes status | Gates are the mechanism that closes assumptions — see [`go_no_go_gates.md`](go_no_go_gates.md) |
| A measurement differs from an assumption | Especially `CA-006`, `CA-009`, `CA-019` |
| Scope changes | A new capability re-opens the deferred-feature assumptions (`CA-018`, `CA-021`) |
| A vendor, authority or supplier changes position | Notably the SHANA interface (`CL-SHANA-001`) |
| **Any assumption is more than 30 days old and still unverified** | ⚠️ See below |

> ⚠️ **An assumption older than 30 days and still unverified should be treated as
> `UNKNOWN`, not as a working assumption.** If the project cannot function without
> an answer, the answer must be obtained.

---

## 9. Summary

| Class | Count | Verified | Unverified |
|---|---|---|---|
| A — Permission to host | 4 | 0 | 4 |
| B — Network | 4 | 0 | 4 |
| C — Classroom devices | 5 | 0 | 5 (1 documented, untested) |
| D — School data and identity | 4 | 0 | 4 |
| E — Storage | 3 | 0 | 3 |
| F — Our own design | 4 | 1 (format structure) | 3 |
| **Total** | **24** | **1** | **23** |

| Category | Count |
|---|---|
| Assumptions that **end the project** if false | 3 — `CA-001`, `CA-004`, `CA-006` |
| Assumptions that **close a feature track** if false | 3 — `CA-018`, `CA-020`, `CA-021` |
| Assumptions answerable by **one conversation** | 11 |
| Assumptions requiring **hardware access** | 3 — `CA-009`, `CA-011`, `CA-013` |
| Assumptions closable by **reading our own documents** | 4 (Class F) |

⚠️ **23 of 24 assumptions are unverified, and not one of the school-side
assumptions has been tested.** That is the accurate state of CloudLight at the end
of the feasibility phase. It is not a criticism of the work — it is the reason the
work was done, and the cheapest of these answers cost ten minutes of somebody's
time.

---

## 10. Related documents

| Document | Covers |
|---|---|
| [`constraints.md`](constraints.md) | The binding walls these assumptions sit inside |
| [`go_no_go_gates.md`](go_no_go_gates.md) | The checkpoints at which assumptions become evidence |
| [`../feasibility_report.md`](../feasibility_report.md) | Conclusions classified by evidence strength |
| [`../open_questions_register.md`](../open_questions_register.md) | The 18 questions that block architecture decisions |
| [`../risk_register.md`](../risk_register.md) | Risks, ratings, mitigations and residual risk |
| [`../decision_log.md`](../decision_log.md) | Decisions made, provisional defaults, rejected options |
| [`../deployment_options.md`](../deployment_options.md) | The baseline deployment and its eleven embedded assumptions |
| [`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md) | 576 questions, every one `UNANSWERED` |