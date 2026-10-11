# CloudLight — Known Failures and Unproven Areas

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ THIS DOCUMENT RECORDS FAILURES
>
> **Part A records failures in how this documentation was produced** — fabricated
> cross-references, a false security claim, a stray file, a concurrent restructure
> that broke links. They are recorded here rather than quietly corrected, because
> a document set that hides its own errors cannot be trusted to report the
> school's.
>
> **Part B records technical areas that are unproven** — some because we could
> not test them, most because the school has not answered.
>
> ⛔ **Neither part is a list of bugs in a working system. There is no working
> system.** Both parts describe weaknesses in a Phase 0 documentation effort and
> in an unverified technical position.

**Read with:** [`test_strategy.md`](test_strategy.md) ·
[`test_inventory.md`](test_inventory.md) ·
[`acceptance_criteria.md`](acceptance_criteria.md)

---

# Part A — Known failures of the research and documentation process

> ### ⚠️ **The work in this repository was partly produced by subagents.**
>
> Subagents draft quickly and confidently, and they will invent a plausible file
> path, a plausible citation or a plausible fact just as readily as they will
> retrieve a real one. **Nothing an agent asserts should be believed without
> checking it against the filesystem or the source.** Every failure below was
> found by checking, not by reading carefully.

## A.1 Fabricated cross-reference paths in drafted documents

| | |
|---|---|
| **What happened** | Subagents drafting CloudLight documents produced **link targets that did not exist** — paths to files that were never created, or that existed under a different name or in a different directory |
| **Why it matters** | A fabricated cross-reference is worse than a missing one. A missing link is visible. A fabricated one *reads as verified*, and a reader following it either finds nothing or — worse — finds a *different* file and believes the link confirmed it |
| **Example still present at time of writing** | `../architecture/architecture.md`, `../safety/risk_register.md`, `../safety/student_privacy.md`, `../infrastructure/school_network.md` and `../architecture/data_flow.md` are linked from `requirements.md`, `ai_assistant.md`, `resource_browser.md`, `resource_search.md`, `risk_register.md` and `timetable_and_substitutions.md`. ⛔ **None of those paths exists.** `docs/safety/`, `docs/infrastructure/` and `docs/cloudlight/safety/` are all absent |
| **Corrected?** | 🟡 **Partially.** These four documents correctly wrote them and they are live-correct. The pre-existing broken links **remain broken** and were deliberately **not** repaired here — those files belong to other agents working concurrently, and this agent's brief forbids modifying them |
| **Residual risk** | ⛔ **High and live.** A broken link is invisible to every reader and to version control. It will survive indefinitely unless a link check runs in CI, and **there is no CI in this repository** |

### A.1.1 A fabricated path caught in this document's own work

> ⚠️ **Recorded because it happened here, not only elsewhere.**

While writing these four testing documents, a questionnaire link was authored as
`../../questionnaires/Additional_CloudLight_Questionnaire.md` — the form correct
from `docs/cloudlight/`, but **one level short from `docs/cloudlight/testing/`**.
Resolved, that path points at `docs/questionnaires/`, which does not exist.

⛔ **It would have shipped broken**, into a document whose entire subject is
the danger of unverified assertions.

**Correction applied:** all four testing documents now use
`../../../questionnaires/Additional_CloudLight_Questionnaire.md`.

**What this demonstrates:** the failure mode is not a property of careless
authors. **It is a property of the method.** A path is as easy to get wrong as a
citation, and it is exactly as wrong. The only defence is mechanical validation
against the filesystem — which is how this one was caught.

## A.2 A false claim that a credential-bearing script was not gitignored

| | |
|---|---|
| **What happened** | A subagent asserted that a credential-bearing script in the repository **was not covered by `.gitignore`** — which, if true, would mean a plaintext password was exposed and needed urgent rotation |
| **The truth** | ⛔ **The claim was false. The script *is* gitignored.** `.gitignore` line 119 lists `start_opencode_server.sh` explicitly, under a section headed *"🔴 CREDENTIAL-BEARING SCRIPT — never commit"*, with the reason spelled out in the comment. Verified with `git check-ignore`, which confirms the ignore rule matches |
| **Why it matters** | ⚠️ **This is the most dangerous class of error in this repository.** A fabricated security finding produces a false alarm that consumes attention, and — in the inverse case — a missed real finding produces silence. A document set that reports both identically is a document set whose security findings mean nothing |
| **Corrected?** | ✅ Yes. The claim was withdrawn once checked against the file |
| **Residual risk** | ⛔ **The credential in that script is real and is still there.** It is correctly gitignored, and that is a repository hygiene control, **not** a rotation. ⚠️ **The owner should still rotate it.** Ignored by git is not the same as harmless |

### A.3 A stray pointer file created and removed

| | |
|---|---|
| **What happened** | A subagent created a pointer or index file that was not requested and did not belong — a navigation file, of the kind this agent's own brief explicitly forbids creating |
| **Why it matters** | Unrequested files in a documentation set become de-facto navigation. Once a reader finds a pointer file, it is authoritative whether or not anyone validated it, and it silently entrenches whatever paths it contains — including fabricated ones |
| **Corrected?** | ✅ Yes. The file was removed |
| **Residual risk** | 🟡 **Low, and partly unverifiable.** The file is gone, and the repository has no committed history — every file is currently untracked — so **there is no revision record proving what existed, when, or what it contained.** The removal is asserted here on the basis of the working session, not on evidence a third party could audit |

## A.4 Concurrent restructure breaking links mid-task

| | |
|---|---|
| **What happened** | **The repository tree was restructured while this work was in progress, by another agent working concurrently.** New directories appeared under `docs/cloudlight/` mid-session — `docs/cloudlight/research/` was created between 22:30 and 22:31 during the writing of these documents. Documents written against the earlier tree lost their targets |
| **Why it matters** | ⚠️ **Cross-references are only valid relative to a tree that still exists.** Any document set assembled by more than one agent has a window in which its links are wrong, and nothing detects that automatically |
| **Corrected?** | 🟡 **Continuously, by luck rather than by process.** Broken links were repaired as they were noticed. New documents written during the restructure were left linking to files their sibling documents had not yet created — `research_index.md` linked to `research_log.md`, `source_register.md` and `claim_evidence_matrix.md`, two of which did not exist when the link was first observed and the third of which still does not |
| **Residual risk** | ⛔ **Live right now.** At the time of writing, a link check across `docs/` found broken targets in `requirements.md`, `ai_assistant.md`, `resource_browser.md`, `resource_search.md`, `risk_register.md`, `timetable_and_substitutions.md` and `docs/cloudlight/research/`. Some are expected to resolve as the other agent finishes writing. **Some are not** — see A.1 |

### A.4.1 A structural cause, not just bad timing

⚠️ **The deeper problem is that two different document conventions coexist in
this repository.**

`docs/shana/` is organised into subdirectories — `safety/`, `architecture/`,
`infrastructure/`, `testing/` and others. `docs/cloudlight/` is **flat**, with
every document in one directory.

Several CloudLight documents were drafted with SHANA-style paths —
`../safety/risk_register.md` — implying a `docs/cloudlight/safety/` directory
that does not exist. This is not bad luck. **It is two incompatible layouts, one
adopted per project, with no decision recorded about which applies to CloudLight.**

⛔ **This will recur on every future document** until the layout is settled. It is
a structural defect, not an incident.

### A.5 What all four failures have in common

> ⚠️ **Every one of them was caught by mechanical checking, not by reading.**
>
> None was found by an agent reviewing its own work, and none would have been
> found by a careful reader either — a fabricated path looks exactly like a real
> one until it is resolved, and a false security claim is indistinguishable from
> a true one on the page.
>
> ✅ **The working conclusion is a simple one: nothing produced by an agent in
> this repository is evidence until it has been checked against a file that
> exists.** Including the claims in this document.

## A.6 Residual risk summary

| # | Risk | Severity | Status |
|---|---|---|---|
| **A6.1** | Broken cross-document links persist undetected | 🔴 High | 🟡 Partially corrected; **no CI check exists to prevent recurrence** |
| **A6.2** | Incompatible document layouts between `shana/` and `cloudlight/` | 🔴 High | 🔴 **Unresolved — structural, not an incident** |
| **A6.3** | Fabricated facts are indistinguishable from real ones without checking | 🔴 High | 🟡 Mitigated by checking, **not by process** |
| **A6.4** | Plaintext credential still present in a gitignored script | 🟠 Medium | ✅ Correctly ignored; ⚠️ **not rotated — owner's action** |
| **A6.5** | No revision history — nothing committed | 🟠 Medium | 🔴 **Unresolved.** Every claim here is asserted, not diffable |
| **A6.6** | Concurrent agents writing to one tree without coordination | 🟠 Medium | 🔴 **Unresolved by design.** Links will keep breaking |

---

# Part B — Unproven technical areas

> ⚠️ **None of these has been tested, because none of them can be.** There is no
> code, no hardware access, and — for most — no school answer. This table records
> what would have to be true for a claim to be credible, and whether that can be
> established today.

## B.1 Unproven technical areas

| Area | Why unproven | What would prove it | Can it be proven now? |
|---|---|---|---|
| **Durable offline caching inside a browser kiosk InPrivate session** | ⛔ **The highest-risk unknown in the project.** Both Microsoft Edge kiosk lockdown experiences run an InPrivate session, and an InPrivate session is *by design* one in which user data is not retained. Whether a service worker can install a durable offline cache, and whether `localStorage` survives a kiosk restart, is **version-dependent behaviour on the specific Edge build this school uses** ([`../windows_startup.md`](../windows_startup.md) §7) | Install the kiosk profile on **one real board**, take the machine offline, restart the browser, and show that today's lesson still renders with a correct `Last synced` time. Repeat for the silent offline→live transition | 🔴 **No.** Requires the school's hardware, their Edge build, and `CL-WIN-001` — a `P0` question, **UNANSWERED**. ⛔ **This cannot be simulated with a different browser and any result obtained elsewhere would not transfer** |
| **School LAN reachability and client isolation** | `CL-NET-003`, `CL-NET-008`, `CL-NET-014` unanswered. ⛔ **Probing school infrastructure is prohibited** by a constraint already accepted in this repository, so the obvious test is forbidden | Ask school IT directly about subnet, client isolation and firewall rules — a **fifteen-minute question**, not a test ([`../feasibility_report.md`](../feasibility_report.md) §20 assumption 2) | 🔴 **No.** But ⚠️ **this is the cheapest item on the list and the one most worth doing first.** It needs a conversation, not equipment |
| **NAS permission-aware retrieval correctness, in both directions** | `FR-SEARCH-02`, `FR-SEARCH-03` and `FR-RES-04` are `CONFIRMED` invariants, and ⛔ **none can be tested against a mock.** A permission test against a fake directory proves the fake is correct. `CL-NAS-001` — *does a shared store exist at all?* — is `P0` and **UNANSWERED** | Against a real share with real ACLs: (a) confirm a permitted document is returned and openable; (b) confirm a forbidden document is absent from results, counts and aggregates; (c) **change an ACL after indexing and confirm the change takes effect at the next open, not only at the next index** | 🔴 **No.** Blocked on `CL-NAS-001`, `CL-NAS-002`, `CL-NAS-007`, `CL-NAS-011`. ⛔ **If the school has no shared storage, these questions are unanswerable and the feature does not exist** — not "is delayed" |
| **Local model capability on unknown school hardware** | No school hardware has been seen, requested or measured. GPU presence, RAM and CPU are all unknown, and `CL-AIML-009` is `UNANSWERED` | Measure the actual candidate host; run the intended model on it; record tokens/second and whether it meets any latency target the school states | 🔴 **No.** ⚠️ And the honest note is that **Release 0 does not need this** — inference is out of scope ([`../first_release_scope.md`](../first_release_scope.md) §2.2). ⛔ **Do not size the host around inference** |
| **Whether the school permits any of it at all** | ⛔ **This is not a technical question, and it is the one that matters most.** 576 of 576 questionnaire questions are `UNANSWERED`. `CL-SRV-001` (may a persistent service run?), `CL-GEN-001` (who authorises storing staff data?), `CL-SUB-001` (who owns the master timetable?) are all `P0` | **Four conversations.** Principal, IT, timetable department, administration. Each takes minutes to ask and cannot be answered by any amount of engineering | 🔴 **No — and this is the binding constraint on the entire project** |

## B.2 Areas with no record of doubt

> ⚠️ **This table is deliberately empty, and its emptiness should not be read as confidence.**
>
> The list above is not exhaustive. It records the unknowns that were identified
> while reading the source documents. ⚠️ **A document set can only report the
> risks its authors thought of.** The highest-risk unknown is very often the one
> nobody wrote down.
>
> ✅ The one structural defence available is already stated in
> [`../first_release_scope.md`](../first_release_scope.md) §7: **gates G1–G4 are
> checked before implementation begins**, not after. The purpose of a gate list is
> precisely to catch the unknown that was not in the author's list.

## B.3 What would reduce the largest risks fastest

> ⚠️ **None of the following requires code, hardware or engineering effort.**
> They are conversations, and they are currently the fastest way to turn this
> project from hypothetical into buildable.

| Priority | Question | `CL-*` | Owner | Cost |
|---|---|---|---|---|
| **1** | ⛔ May a persistent local service run on school premises? | `CL-SRV-001` | Principal / IT | 15 minutes |
| **2** | ⛔ Can classroom devices actually reach a host? | `CL-NET-003`, `CL-NET-008` | School IT | 15 minutes |
| **3** | ⛔ Who approves a substitution, and who owns the master timetable? | `CL-SUB-001`, `CL-TIME-001` | Timetable dept | 15 minutes |
| **4** | ⛔ Is there a shared file store at all? | `CL-NAS-001` | School IT | 15 minutes |
| **5** | ⚠️ What Windows edition is on the boards? | `CL-WIN-001` | School IT | One `winver` |
| **6** | ⚠️ Is there a staff identity directory? | `CL-AUTH-001` | Administration | 15 minutes |
| **7** | ⚠️ **Who owns this in year two?** | `CL-OPS-022` | Principal | ⛔ **The question that decides whether to build at all** |

⚠️ **Question 7 is the one most easily deferred and the one with the worst
outcome.** [`../first_release_scope.md`](../first_release_scope.md) §7 puts it
plainly: ⛔ *a project with no year-two owner should not be built.* It is the one
worst case that ends the project rather than shrinking it.

---

## B.4 Honest position

> ⚠️ **This document records two kinds of failure and neither is small.**
>
> **Part A** says the documentation process produced fabricated paths, a false
> security claim, a stray file and concurrent link breakage — and that the
> structural cause (two incompatible document layouts) is still unfixed.
>
> **Part B** says the central technical assumptions — durable offline caching on
> the school's browser, school network reachability, permission correctness,
> model capability, and above all **permission itself** — are all unproven.
>
> ⛔ **Nothing here is a bug report against a system. There is no system.** These
> are the weaknesses of a Phase 0 feasibility effort, recorded so that whoever
> reads this next knows exactly what they are inheriting.
>
> ✅ **And the honest summary of both parts is the same: this repository contains
> a large amount of careful analysis and no software.** The analysis is worth
> having. The software has not been started.

---

**Related documents:** [`test_strategy.md`](test_strategy.md) ·
[`test_inventory.md`](test_inventory.md) ·
[`acceptance_criteria.md`](acceptance_criteria.md) ·
[`../feasibility_report.md`](../feasibility_report.md) ·
[`../first_release_scope.md`](../first_release_scope.md) ·
[`../requirements.md`](../requirements.md) ·
[`../windows_startup.md`](../windows_startup.md) ·
[`../resource_search.md`](../resource_search.md) ·
[`../network_and_hosting.md`](../network_and_hosting.md) ·
[`../open_questions_register.md`](../open_questions_register.md) ·
[`../risk_register.md`](../risk_register.md) ·
[`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md)