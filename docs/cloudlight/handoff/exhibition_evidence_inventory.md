# CloudLight — Exhibition Evidence Inventory

**Date:** 2026-10-11 · **For:** the third agent, who will produce a school
exhibition proposal

> **🔴 READ THIS BEFORE WRITING A SINGLE LINE OF THE PROPOSAL.**
>
> **There is no working software in this repository. Nothing runs. Nothing has
> been deployed. Nothing has been demonstrated. No school system has been
> contacted.**
>
> Every CloudLight capability is a **proposal**. This document exists to make it
> impossible to accidentally present a proposal as a product.

---

## 1. Evidence classes — use these words precisely

| Class | Meaning | Safe to present as a capability? |
|---|---|---|
| `VERIFIED` | Confirmed in official vendor/standards documentation, quoted with a source | Only as a **platform fact**, never as "we built it" |
| `DOCUMENTED_NOT_HARDWARE_VERIFIED` | A vendor documents it; untested on this school's equipment | **No** — present as "based on vendor documentation" |
| `PLAUSIBLE_REQUIRES_TESTING` | Engineering judgement; needs a test | **No** — present as "proposed, needs testing" |
| `UNKNOWN` | Nobody knows | **No** — present as an open question |
| `REQUIRES_SCHOOL_PERMISSION` | Blocked on written approval | **No** — present as "requires school approval" |
| `REQUIRES_VENDOR_OR_SERVICE_PROVIDER` | Blocked on a vendor | **No** |
| `NOT_RECOMMENDED_WITHOUT_APPROVAL` | Rejected absent explicit approval | Present as a **safety decision** |

Full matrix with per-claim exhibition verdicts:
[`../research/claim_evidence_matrix.md`](../research/claim_evidence_matrix.md).

---

## 2. 🔴 Claims that must NEVER appear in the proposal

Each of these would be false. This list exists to be checked against.

| # | Forbidden claim | Why it is false | Safe alternative |
|---|---|---|---|
| 1 | "The system is built" / "the platform works" | There is **no code** | "An architecture and feasibility study has been completed" |
| 2 | "We tested it in the school" / "it was trialled" | No pilot has occurred; no school contact exists | "A pilot is proposed, pending school approval" |
| 3 | "The school has approved this" | **No approval of any kind exists** | "Approval would be required from…" |
| 4 | "The system connects to the school NAS" | No NAS has been accessed | "Design assumes an authorised read-only service account, if one is granted" |
| 5 | "It works offline on the smartboards" | Unbuilt, and offline caching is an **unresolved high-risk unknown** | "Offline behaviour is designed for; a browser-kiosk cache remains unverified" |
| 6 | "The AI assistant answers questions" | Unbuilt; external AI is prohibited by default | "An assistant is proposed, contingent on school data policy" |
| 7 | "Teachers authenticate securely with a six-digit code" | **A shared six-digit code is not proof of identity** | "Individual teacher accounts; a code may only authorise one action" |
| 8 | "It handles the timetable and substitutions" | Unbuilt; no school timetable has ever been seen | "A timetable ingestion pipeline is specified; no real timetable has been received" |
| 9 | "SHANA delivers items on request" | Interface not agreed; **zero verified destinations exist** | "An interface is proposed; no agreement exists between the two subsystems" |
| 10 | Any screenshot of a working interface | **No screenshots exist** | Draw a labelled architecture diagram instead |
| 11 | "Tests passed" / any coverage figure | **Zero tests have ever been executed** | "No code exists, therefore no tests" |
| 12 | "This is ready for exhibition / deployment" | Nothing is built | "This is a feasibility and architecture study" |

⚠️ **The repository root contains 9 JPEG images (`Image.jpeg`, `2.jpeg` … `8.jpeg`).**
SHANA has already determined these are photographs of **LCD/display panels** and is
unrelated to this project. **They must never be presented as robot, classroom,
smartboard or system evidence.** See SHANA's
`docs/shana/handoff/exhibition_evidence_inventory.md` for the full determination.

---

## 3. What actually exists as evidence

### 3.1 Documentation — this is the entire evidence base

| Artefact | Count | Lines | Evidence value |
|---|---|---|---|
| CloudLight documents (flat) | 22 | ~13,000 | Design reasoning |
| CloudLight documents (subfolders) | 25 | ~7,000 | Design, research, test plans, handoff |
| CloudLight questions | **576** | 980 | Requirements discovery — **0 answered** |
| SHANA questions | 434 | — | Robotics discovery — **0 answered** |
| CloudLight ADRs | 2 | — | ADR-013 modular monolith, ADR-014 no robot motion |
| External sources registered | 36 research threads / 62 sources | — | Vendor and standards citations |
| Claims tracked | 62 | — | Each with an exhibition verdict |
| Tests executed | **0** | — | — |
| Screenshots of working software | **0** | — | — |
| Commits | **0** | — | Nothing has been version-controlled yet |

⚠️ **The evidence base is analytical, not empirical.** This is a legitimate and
valuable body of work — but it is evidence *about feasibility*, not evidence of
a working system, and the proposal must say so.

### 3.2 Architecture diagrams

⚠️ **There are no diagram image files.** Diagrams exist only as ASCII/text inside
Markdown, and must be rendered or redrawn rather than screenshotted:

| Diagram | Location | Useful for an exhibition proposal? |
|---|---|---|
| CloudLight ↔ SHANA boundary | `../overview.md` §2 | ✅ Yes — clearly conveys the safety model |
| System topology | `../architecture.md` §3 | ✅ Yes |
| Data flow | `../architecture.md` §4 | ✅ Yes |
| Timetable pipeline | `../timetable_and_substitutions.md` | ✅ Yes — shows the human approval gate |
| Failure behaviour | `../architecture.md` §5 | ✅ Yes — shows honesty under failure |
| System context (C4) | `architecture/system_context.md` | ✅ Yes |
| Trust boundaries | `../security_and_privacy.md` | ✅ Yes |

**Recommendation:** redraw 3–4 of these as clean labelled graphics for the
proposal. Label every one *"proposed architecture — not implemented"*.

### 3.3 Mock / illustrative data

| Artefact | Type | Safe to use? |
|---|---|---|
| SHANA's `data/schemas/*.yaml` | Illustrative schema definitions, headers say `PROPOSED — not implemented` | ✅ Yes, **labelled as illustrative** |
| Example teacher queries in `../resource_search.md` | Written by the project team | ✅ Yes, **labelled as illustrative** |
| Example entity field sketches in `architecture/data_model.md` | Illustrative | ✅ Yes, **labelled as design** |
| Any timetable or resource content | **None exists** | N/A |

⚠️ **There is no mock *application*.** Because there is no application, there is
nothing running on fabricated data. This is worth stating positively: the
exhibition cannot accidentally show fake school data, because it cannot show
anything at all.

---

## 4. Existing integration evidence

| Integration | Real? | Mock? | None? |
|---|---|---|---|
| School LAN | ❌ | ❌ | ✅ Not probed |
| NAS | ❌ | ❌ | ✅ Never accessed |
| Identity provider | ❌ | ❌ | ✅ None chosen or contacted |
| Smartboards | ❌ | ❌ | ✅ None touched |
| External AI | ❌ | ❌ | ✅ None contacted |
| Update source | ❌ | ❌ | ✅ None exists |
| **SHANA** | ❌ | ⚠️ **Two divergent design proposals** | ✅ Nothing implemented |

⚠️ **The only integration-adjacent artefact is the pair of incompatible interface
proposals.** That is itself a legitimate and honest exhibit: it demonstrates that
the safety boundary was designed from both sides, and shows exactly what still
needs agreeing.

---

## 5. Known limitations that MUST be disclosed

Any proposal omitting these would be misleading.

| # | Limitation | Evidence |
|---|---|---|
| 1 | **No code exists.** Zero source files | `implementation_inventory.md` §1 |
| 2 | **No test has ever been run** | `../testing/test_inventory.md` — all `NOT RUN` |
| 3 | **No school system has been contacted or probed** | Repository-wide verification |
| 4 | **Nothing about this school's infrastructure is known** — not one fact verified | `../research/claim_evidence_matrix.md` — all 13 school-infrastructure claims `UNKNOWN` |
| 5 | **No approval of any kind has been obtained** | 1,010 questions, 0 answered |
| 6 | **Nine of sixteen proposed capabilities are out of the first release** | `../first_release_scope.md` |
| 7 | **Four of those nine are deferred because the prerequisite may not exist** | `../feasibility_report.md` §22 |
| 8 | **A six-digit code is not proof of teacher identity** | `../authentication_and_authorisation.md` |
| 9 | **Offline behaviour on a real smartboard is an unresolved high-risk unknown** | `../windows_startup.md` |
| 10 | **No delivery destination exists** — SHANA's schema defaults all to `NEEDS_SURVEY` | `implementation_inventory.md` §4.3 |
| 11 | **The two subsystem interfaces have never been reconciled** | `implementation_inventory.md` §4.2 |
| 12 | **A credential sits in a gitignored local script and has not been rotated** | `../security_and_privacy.md`; owner action required |

---

## 6. Suitable demonstration sequence — **for the proposal only**

⚠️ **Nothing below can currently be demonstrated, because nothing exists.** This is
a sequence of *proposals and reasoning* that CAN be presented honestly, using only
the material in this repository.

| # | Exhibit | What it actually shows | Material | Claim class |
|---|---|---|---|---|
| 1 | **The problem** | Teachers cannot see who is teaching which class, from which room, right now | `../product_vision.md` | `PLAUSIBLE_REQUIRES_TESTING` — confirm with teachers |
| 2 | **The safety boundary** | CloudLight may request a delivery; it may never command the robot | `ADR-014`, `../shana_integration.md` | ✅ **VERIFIED design decision** |
| 3 | **Why an access point is not a server** | Network infrastructure and compute are different things — a common, costly misconception | `../network_and_hosting.md` | ✅ **VERIFIED** |
| 4 | **Why a six-digit code is not an identity** | A security finding that improves the design before it is built | `../authentication_and_authorisation.md` | ✅ **VERIFIED analysis** |
| 5 | **The human approval gate** | No automated path — including AI — can publish a timetable change | `../timetable_and_substitutions.md` | ✅ **VERIFIED design invariant** |
| 6 | **Honesty under failure** | Stale data is marked; a blank screen never masquerades as data | `../classroom_dashboard.md` | ✅ **VERIFIED design invariant** |
| 7 | **The 576 questions** | Rigour in discovery rather than assumption | The questionnaire itself | ✅ **VERIFIED — it exists** |
| 8 | **What we do not know** | 13 school-infrastructure claims, all `UNKNOWN`, listed openly | `../research/claim_evidence_matrix.md` | ✅ **VERIFIED — the honesty is the exhibit** |
| 9 | **The proposed first release** | A small, dependency-free slice that needs none of the unknowns | `../first_release_scope.md` | `PLAUSIBLE_REQUIRES_TESTING` |
| 10 | **The next ten actions** | Cheapest questions first, because they can invalidate the design | `../next_actions.md` | ✅ **VERIFIED as a plan** |

⚠️ **Exhibits 2–6 and 8 are the strongest available.** They are genuine
*reasoning artefacts*: a designed safety boundary, a corrected misconception, a
security finding, and an honest inventory of unknowns. An exhibition that
presented these accurately would be more credible than one claiming a working
system.

---

## 7. Suitable measurable outcomes

⚠️ **No outcome has been measured. These are proposed measurements with the
evidence that would be needed.** Each must be phrased as a *target*, never as a
*result*.

| Proposed measure | How it would be measured | Evidence needed | Currently |
|---|---|---|---|
| Correct lesson shown within N seconds of period start | Timer against a reference clock, N confirmed by the school | Built system + school-approved N | ❌ Unbuilt, N unknown |
| Zero unapproved substitutions displayed as teachers | Audit log review across a pilot term | Built system | ❌ Unbuilt |
| Zero invented teacher names | Adversarial test with deliberately incomplete data | Built system + tests | ❌ Unbuilt |
| Displays recover from server loss without intervention | Controlled network-cut test on one board | Built system + one board | ❌ Unbuilt |
| Rollback across all displays within N minutes | Induced bad publication | Built system, school-approved N | ❌ Unbuilt |
| Teachers find a resource in under N seconds | Timed usability task with real teachers | Built system + search index | ❌ Unbuilt + blocked on storage |
| Search never returns an unauthorised document | Adversarial permission test both directions | Built system + directory | ❌ Unbuilt + blocked on identity |
| Reduction in substitution-publishing time | Before/after study at the timetable desk | Built system + school data | ❌ Unbuilt |

⚠️ **Never state a number without the phrase "proposed target" or "requires
measurement".**

---

## 8. Sources that support project claims

Registered in [`../research/source_register.md`](../research/source_register.md) —
36 research threads. The categories that most support an exhibition proposal:

| Domain | What it supports | Class |
|---|---|---|
| Windows kiosk documentation (Microsoft Learn) | That a managed kiosk browser is a documented, supported approach | `VERIFIED` — platform capability only |
| OWASP authentication guidance | That the six-digit-code finding follows standard practice | `VERIFIED` |
| NIST SP 800-63B | That SMS is a *restricted* authenticator — supports the MFA recommendation | `VERIFIED` |
| TUF specification | That update systems face rollback/freeze/fast-forward attacks and how to resist them | `VERIFIED` |
| Permission-aware search implementations | That ACL-carried roles filtered at query time is an established technique | `VERIFIED` as a technique, untested here |
| Local inference runtime documentation | Rough sizing for local models | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| OWASP LLM guidance | Prompt injection is a known, structural failure mode of RAG | `VERIFIED` as a class of risk |

⚠️ **None of these is evidence that CloudLight works.** They evidence that the
*approach* is sound. Say so precisely.

---

## 9. Evidence that still needs collecting

In priority order, for whoever picks this up next.

| # | Evidence needed | Who can produce it | Blocks |
|---|---|---|---|
| 1 | Written answer to the 5 P0 questions | School administration, IT, timetable staff | Everything |
| 2 | Confirmation that a persistent service is permitted | Administration + IT | Any deployment |
| 3 | Confirmation that displays can reach the host, and whether isolation is on | IT | Any deployment |
| 4 | Windows edition and admin rights on one board | IT | Display design |
| 5 | A real sample timetable, with permission | Timetable staff | Timetable design validation |
| 6 | Whether shared storage exists and its permission model | IT | Resource features |
| 7 | A joint interface session between CloudLight and SHANA | Both agents | Robot feature |
| 8 | One verified delivery destination | SHANA custodian | Any delivery demo |
| 9 | Teacher confirmation that the problem statement is real | Teachers | Exhibition framing |
| 10 | Any working software at all | Future work | Everything demonstrable |

---

## 10. The one-paragraph honest summary

> This repository contains a **completed architecture and feasibility study** for
> a school platform, alongside the existing study for its robotics subsystem. It
> contains **no software, no tests, no deployment, and no contact with the school**.
> Its 1,010 questions are **unanswered**. What it does contain is a rigorous
> account of what is unknown, why certain designs were rejected on safety grounds,
> and what the first ten questions should be. **Presenting it as a working system
> would be false; presenting it as thorough, honest groundwork would be accurate
> and is genuinely valuable.**

---

## Related

- [`agent_handoff.md`](agent_handoff.md) — start here
- [`implementation_inventory.md`](implementation_inventory.md) — component-by-component status
- [`unresolved_dependencies.md`](unresolved_dependencies.md) — what is blocked and on whom
- [`../research/claim_evidence_matrix.md`](../research/claim_evidence_matrix.md) — per-claim exhibition verdicts
- [`../overview/current_project_state.md`](../overview/current_project_state.md) — factual state
- [`../testing/known_failures.md`](../testing/known_failures.md) — failures of this process itself