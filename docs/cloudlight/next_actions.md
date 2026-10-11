# CloudLight — Next Actions

**Phase 0 · Documentation only · 2026-10-11**

> **⚠️ NOTHING ON THIS LIST IS IMPLEMENTATION.**
> No code is to be written. No device is to be touched. No network is to be
> probed. This list contains **questions, measurements and approvals** — the only
> three kinds of work that can move CloudLight forward while the gates are shut.
>
> Every `CL-*` ID below is `UNANSWERED` in
> [`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md).

---

## 1. Framing

> **These are questions, measurements and approvals — in that order of
> preference. Implementation is not on this list, and must not be added to it
> until the `P0` gates have returned.**

### 1.1 Why this order

| Rank | Kind of action | Why it is preferred |
|---|---|---|
| **1** | **Question** — ask a named person something only they can answer | Costs minutes. Invalidates work that would otherwise take weeks |
| **2** | **Measurement** — observe something that already exists | Costs an hour. Converts an assumption into a fact with evidence attached |
| **3** | **Approval** — obtain a decision recorded in writing | Slowest, and occasionally impossible. But nothing is stored, deployed or published without one |
| **—** | **Implementation** | ⛔ **Not on this list.** Building against an unanswered `P0` produces a guess with normative grammar, which is considerably harder to notice than a guess in a comment ([`requirements.md`](requirements.md) §1.2) |

### 1.2 The ranking principle

> **A five-minute question to the Principal outranks a week of engineering,
> because it can invalidate the entire design.**

The reasoning is asymmetric and worth stating plainly:

| | A question | A week of engineering |
|---|---|---|
| Cost | Minutes | A week |
| What it can tell you | Whether the project is possible | Whether *your* implementation worked |
| What happens if you skip it | You find out in week six, having built on a guess | — |
| Reversibility if wrong | Total — nothing was built | Partial — the work is already spent |

⚠️ **Four of the ten actions below can individually end the project** rather
than change its shape. Three more can each close off an entire feature track.
None of them costs more than a conversation.

⚠️ **Research is finished.** No amount of further reading will produce these
answers, because they are properties of a specific school — its firewall rules,
its Windows edition, its approval chain, its policy. *"Unknown" is a valid and
valuable answer"*; a guess is not
([`open_questions_register.md`](open_questions_register.md) §1).

### 1.3 What the whole list costs

| Measure | Estimate |
|---|---|
| Principal or Administration time | ⬜ **~25 minutes**, across three short conversations |
| School IT time | ⬜ **~40 minutes**, including one short site visit |
| Total elapsed | One week, running in parallel |
| Total engineering | **Zero** |

⚠️ **That is the entire remaining cost of learning whether this project is
possible.** Everything after it — design, build, deployment — is only worth
spending once these return.

---

## 2. The ten actions, in priority order

Ranked so the **cheapest, highest-unblocking** actions come first.

| # | Action | Who | Effort | What it unblocks | Gate | Depends on |
|---|---|---|---|---|---|---|
| **1** | **Ask the Principal whether a persistent local service is permitted at all.** Is one school machine allowed to run a service, for the whole working day, during lessons? | Principal / Administration | ⬜ **10 min** — one conversation | ⛔ **Everything.** Without a permitted host there is no local-first product and this becomes a different project | `P0` | ✅ `CL-SRV-001` ✅ `CL-GEN-002` |
| **2** | **Ask IT whether client/AP isolation and firewall rules permit classroom devices to reach a host.** Is isolation on? Which ports are permitted between the classroom segment and a server segment? | School IT | ⬜ **15 min** — one conversation | ⛔ **Every display.** ⚠️ Isolation fails *silently*: boards associate, show full signal, and never load | `P0` | ✅ `CL-NET-003` ✅ `CL-NET-008` ✅ `CL-NET-014` |
| **3** | **Ask administration who approves a substitution, and who owns the master timetable.** Class teacher, clerk, Vice Principal, subject head? | Timetable / Administration | ⬜ **15 min** — one conversation | ⛔ **The core feature.** A perfect pipeline with the wrong approver has no publication path at all | `P0` | ✅ `CL-SUB-001` ✅ `CL-TIME-001` |
| **4** | **Ask IT what Windows edition and build is on the classroom boards, and whether the kiosk account has local administrator rights.** | School IT | ⬜ **10 min** — one conversation | The **display design**. ⛔ Without Assigned Access or Shell Launcher the kiosk recommendation collapses and the fallback is a browser opened by hand daily | `P0` | ✅ `CL-WIN-001` ✅ `CL-WIN-003` |
| **5** | **Ask IT whether a NAS or shared file server actually exists — and what it is.** Or do teachers keep materials on laptops, pen drives and personal cloud drives? | School IT | ⬜ **10 min** — one conversation | The **entire resource track**, and through it the assistant. ⚠️ *"No"* closes both permanently — which is a valid, useful answer | `P0` | ✅ `CL-NAS-001` ✅ `CL-NAS-002` |
| **6** | **Ask whether a staff identity directory exists, and whether SSO against it is permitted.** A single list of teachers with stable identifiers? AD, LDAP, Google Workspace, M365? | Administration / IT | ⬜ **10 min** — one conversation | **Authentication**, and every attributable audit decision. ⚠️ *"No"* means CloudLight owns provisioning *and* offboarding | `P0` | ✅ `CL-AUTH-001` ✅ `CL-AUTH-002` ✅ `CL-AUTH-003` |
| **7** | **Ask the Principal whether any external AI service is permitted to receive school content.** One yes/no, in writing. | Principal | ⬜ **5 min** — one conversation | The **assistant**, and nothing else. ⛔ May permanently close it. ⚠️ Not inferable from any technical fact | `P0` | ✅ `CL-AI-001` ✅ `CL-DEC-010` |
| **8** | **Ask whether shared storage may be indexed at all, and which folders are permitted.** A dedicated read-only service account? An explicit approved folder list? | Administration + IT | ⬜ **10 min** — one conversation | **Search.** ⛔ *Moot if action 5 returned "no"* — ask only if storage exists | `P0` | ✅ `CL-NAS-007` ✅ `CL-NAS-008` ✅ `CL-NAS-012` |
| **9** | **Ask about an always-on host: is there a spare machine, a spare port, a suitable location, and a UPS?** And who would physically restart it during school hours? | Facilities / IT | ⬜ **30 min** — one site walk plus a conversation | **Deployment.** Turns "a server is needed" into a specific machine in a specific room with a specific person | `P0` / `P1` | ✅ `CL-SRV-002` ✅ `CL-SRV-003` ✅ `CL-SRV-007` ✅ `CL-SRV-011` |
| **10** | **Run ONE cheap authorised technical test:** `winver` on a single **non-critical** board to confirm the edition, plus look for a spare Ethernet port nearby. | School IT, with explicit permission | ⬜ **15 min** on site | **Converts actions 4 and 9 from claims into measurements.** ⚠️ Only on a machine IT has named as non-critical — never on a live classroom board unasked | `P0` | ✅ `CL-WIN-001` ✅ `CL-WIN-007` |

**Legend.** ✅ = ID verified by grep against
[`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)
on 2026-10-11 — every ID appears as a row key in that file. ⬜ = effort estimate,
proposal requiring confirmation. **No ID on this list was invented.**

⚠️ **Actions 1–4 are the four that end the project rather than change its shape.**
Actions 5–8 each close or open an entire feature track. Action 9 is the first
that requires anyone to leave a desk. Action 10 is the only one that touches
hardware at all — and only with explicit permission.

⚠️ **Action 8 is conditional on action 5.** If there is no shared storage, there
is nothing to index, and asking whether indexing is permitted is a question about
a hypothetical.

---

## 3. What must NOT be done yet

> **A short, firm list.** Every item below is a way of converting an unknown into
> a commitment that nobody agreed to.

| ⛔ | Do not | Why |
|---|---|---|
| 1 | **Install anything on any smartboard** | No software is authorised, no edition is confirmed, and a misconfigured kiosk takes a classroom out of use. ⛔ `CL-WIN-007` names which machine may be tested, and it has not been answered |
| 2 | **Build application code** | Every module is gated by a `P0` answer. Code written now encodes a guess as a structure, and restructuring is far more expensive than starting again |
| 3 | **Probe the school network** | Scanning, port testing or connectivity testing on infrastructure nobody administers is a serious act regardless of intent. ⚠️ Read the AP configuration from someone who administers it instead |
| 4 | **Photograph any document** | Timetables, substitution sheets and staff material contain personal data. ⛔ `CL-SUB-006` asks whether photographing the board is permitted at all — and it is a permission question, not a technical one |
| 5 | **Contact any external AI service** | ⛔ Sending school content to a third party without written authorisation is the exact act `CL-AI-001` asks about. Asking the question by doing it would answer it badly and irreversibly |
| 6 | **Promise any feature to the school** | ⚠️ An unstated scope becomes an unstated commitment. Twelve capabilities are deferred; each was promised to somebody at some point (`CL-DEC-017`) |
| 7 | **Mark anything `VERIFIED` without evidence** | `ANSWERED` means a person replied. `VERIFIED` means evidence exists. ⚠️ Eleven gates have no plausible default that is safe to assume |

⚠️ **Actions 6 and 7 are the two that damage the relationship rather than the
system.** A feature promised and then withdrawn costs more credibility than one
never offered. And a gate marked verified on the strength of a conversation is
how a project discovers, in front of a class, that it was built on an assumption.

---

## 4. Sequencing

```
   INDEPENDENT — ASK ALL FOUR IN THE SAME WEEK
   ═══════════════════════════════════════════

   (1) Principal    · persistent service permitted?  · CL-SRV-001, CL-GEN-002
        │
        ├── NO ──► ⛔ STOP. The local-first premise fails.
        │           Document the platform; do not build. Escalate.
        ▼
   (2) School IT    · isolation? firewall?          · CL-NET-003, CL-NET-008, CL-NET-014
        │                   (independent of (1) — ask both the same day)
        ▼
   (3) Timetable    · who approves a substitution?   · CL-SUB-001, CL-TIME-001
        │                   (independent of (1) and (2))
        ▼
   (4) School IT    · Windows edition? admin rights?· CL-WIN-001, CL-WIN-003
        │
        ▼
   ═══ NOTHING ELSE CAN BE FINALISED UNTIL THESE FOUR RETURN ═══


   THEN — only what the answers make meaningful
   ────────────────────────────────────────────────

   (5) IT            · does shared storage exist?   · CL-NAS-001, CL-NAS-002
        │
        ├── NO ──► ⛔ Drop resource features AND the assistant. Permanently.
        │           ✅ The core product is unaffected. Proceed to (9).
        ▼
   (6) Admin / IT    · staff directory? SSO?        · CL-AUTH-001..003
        ▼
   (7) Principal     · external AI permitted?       · CL-AI-001, CL-DEC-010
        │                   (independent of (5) and (6) — any week)
        ▼
   (8) Admin + IT    · may storage be indexed?      · CL-NAS-007/008/012
        │                   ⛔ MOOT if (5) returned "no"
        ▼
   (9) Facilities    · machine? room? power? UPS?   · CL-SRV-002/003/007/011
        │
        ▼
   (10) School IT    · winver on ONE non-critical board  · CL-WIN-001, CL-WIN-007
                      ⚠️ With explicit permission, named machine, nothing installed
```

### 4.1 Why (1)–(4) can be asked together

Each is asked of a **different person or role**, answers **no question asked in
any other**, and none requires the answer to any other. They are four independent
conversations, not a chain.

⚠️ **The practical wrinkle, stated honestly.** Actions 2 and 4 are both School IT,
and actions 1 and 7 are both the Principal. So this is realistically **two or
three conversations, not four** — which is also why it fits comfortably in one
week. Bundle 2 and 4 into a single IT conversation. Bundle 1 and 7 into a single
Principal conversation, and ask 7 while you have them: *"may any school content
go to an external AI service?"* takes five seconds and closes off an entire
feature permanently, in either direction.

### 4.2 What happens when they return

| Outcome | Response |
|---|---|
| **All four favourable** | ✅ Proceed to actions 5–9, then to Release 0 planning ([`first_release_scope.md`](first_release_scope.md) §7) |
| **Any one unfavourable** | ⛔ **Do not proceed on the assumed answer.** Record it, take the smaller scope, and re-plan |
| **Any one unanswered** | Apply the conservative default recorded in [`open_questions_register.md`](open_questions_register.md) §2 — ⚠️ **never an optimistic one** |
| **`CL-OPS-022` returned "nobody"** | ⛔ **Stop.** This is the one answer that ends the project rather than shrinking it |

⚠️ **Nine of the worst-case answers produce a smaller project.** ✅ A timetable
and substitution display that works, on a machine the school already owns, is
worth more than a full platform that nothing maintains. **One does not.**

---

## 5. Where each action is recorded

| Action | Questionnaire IDs — ✅ all verified by grep 2026-10-11 | Tracked in |
|---|---|---|
| **1** — persistent service | ✅ `CL-SRV-001` ✅ `CL-GEN-002` | OQ-2 → [`open_questions_register.md`](open_questions_register.md) §2; R-05, R-45 → [`risk_register.md`](risk_register.md) §1; D-101 → [`decision_log.md`](decision_log.md) §3 |
| **2** — isolation & firewall | ✅ `CL-NET-003` ✅ `CL-NET-008` ✅ `CL-NET-014` | OQ-3, OQ-4 → [`open_questions_register.md`](open_questions_register.md) §2; R-06, R-07 → [`risk_register.md`](risk_register.md) §1; baseline assumption 3, 4 → [`deployment_options.md`](deployment_options.md) §4.3 |
| **3** — approval authority | ✅ `CL-SUB-001` ✅ `CL-TIME-001` | OQ-12, OQ-14 → [`open_questions_register.md`](open_questions_register.md) §2; R-15, R-16 → [`risk_register.md`](risk_register.md) §1; D-008 already decided → [`decision_log.md`](decision_log.md) §1 |
| **4** — Windows edition | ✅ `CL-WIN-001` ✅ `CL-WIN-003` | OQ-10, OQ-11 → [`open_questions_register.md`](open_questions_register.md) §2; R-11, R-12 → [`risk_register.md`](risk_register.md) §1; D-101, D-102 → [`decision_log.md`](decision_log.md) §3 |
| **5** — shared storage | ✅ `CL-NAS-001` ✅ `CL-NAS-002` | OQ-5 → [`open_questions_register.md`](open_questions_register.md) §2; R-19 → [`risk_register.md`](risk_register.md) §1; D-103 → [`decision_log.md`](decision_log.md) §3 |
| **6** — identity directory | ✅ `CL-AUTH-001` ✅ `CL-AUTH-002` ✅ `CL-AUTH-003` | OQ-8 → [`open_questions_register.md`](open_questions_register.md) §2; R-21 → [`risk_register.md`](risk_register.md) §1; D-102 → [`decision_log.md`](decision_log.md) §3 |
| **7** — external AI policy | ✅ `CL-AI-001` ✅ `CL-DEC-010` | OQ-16 → [`open_questions_register.md`](open_questions_register.md) §2; R-30 → [`risk_register.md`](risk_register.md) §1; D-105 → [`decision_log.md`](decision_log.md) §3 |
| **8** — indexing permission | ✅ `CL-NAS-007` ✅ `CL-NAS-008` ✅ `CL-NAS-012` | OQ-7 → [`open_questions_register.md`](open_questions_register.md) §2; R-28, R-29 → [`risk_register.md`](risk_register.md) §1 |
| **9** — host, power, restart | ✅ `CL-SRV-002` ✅ `CL-SRV-003` ✅ `CL-SRV-007` ✅ `CL-SRV-011` | [`open_questions_register.md`](open_questions_register.md) §4 (Facilities); R-01, R-05 → [`risk_register.md`](risk_register.md) §1; D-101 → [`decision_log.md`](decision_log.md) §3 |
| **10** — the one technical test | ✅ `CL-WIN-001` ✅ `CL-WIN-007` | [`open_questions_register.md`](open_questions_register.md) §5; R-11 → [`risk_register.md`](risk_register.md) §1 |

### 5.1 Where an *answer* gets written

| When an answer arrives | Where it goes | Never |
|---|---|---|
| The questionnaire row, with `Status`, and evidence in `Evidence / Source / Attachment` | `../../questionnaires/Additional_CloudLight_Questionnaire.md` | ⛔ Never in this repository as school data — see D-010 |
| A blocked question becomes answered | [`open_questions_register.md`](open_questions_register.md) §2 — the register shrinks | ⛔ Not deleted from the questionnaire |
| A decision is taken | [`decision_log.md`](decision_log.md) — state, reasoning, evidence, what would change it | ⛔ Never silently |
| A risk changes | [`risk_register.md`](risk_register.md) — re-rated, with residual recorded | ⛔ No risk retires through research alone |

⚠️ **Answered ≠ verified.** `ANSWERED` means a person replied. `VERIFIED` means a
piece of evidence exists and is attached. Only one of them changes what we are
allowed to build.

---

## 6. The status of this list

> **This is a list of things to ask, not things to do.** Ten questions. One week.
> Roughly two hours of somebody's time.
>
> ⛔ **Nothing may be implemented until actions 1–4 return.**

| If all ten return favourably | Then |
|---|---|
| Release 0 becomes plannable | [`first_release_scope.md`](first_release_scope.md) §7 gates cleared; §9 exit criteria become the target |
| The resource track opens | Only if action 5 returned *"yes"* — browse first, search later |
| The assistant may open | Only if action 7 returned *"yes"* in writing — and it is still built last, or never |
| SHANA stays closed | `CL-SHANA-001` and `CL-SHANA-015` remain unanswered and are **not** on this list |

⚠️ **Action 7 has no follow-up on this list.** If the Principal permits external
AI, that permission is not a green light to contact one — it opens a design
question about which inference runs where, which requires hardware nobody has
identified (`CL-AI-009`). ✅ Permitted is not the same as built.

---

**Related documents:** [`overview.md`](overview.md) ·
[`open_questions_register.md`](open_questions_register.md) ·
[`first_release_scope.md`](first_release_scope.md) ·
[`feasibility_report.md`](feasibility_report.md) ·
[`network_and_hosting.md`](network_and_hosting.md) ·
[`windows_startup.md`](windows_startup.md) ·
[`requirements.md`](requirements.md) · [`risk_register.md`](risk_register.md) ·
[`decision_log.md`](decision_log.md) ·
[`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)