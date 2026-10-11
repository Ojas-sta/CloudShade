# CloudLight — Unresolved Dependencies

**Date:** 2026-10-11**

Every dependency CloudLight cannot satisfy alone, who owns it, and what happens if
it is never satisfied. **Nothing here has been resolved.**

Status: `BLOCKED` · `UNKNOWN` · `AGREEMENT_REQUIRED` · `PROCESS`

---

## 1. Dependencies on the school

These require a person, a measurement, or a signature. **No contact has occurred.**

| # | ID | Dependency | Owner | Status | If never satisfied |
|---|---|---|---|---|---|
| 1 | `DEP-SCH-01` | Written permission to run a persistent service on school premises | Administration + IT | `BLOCKED` | 🔴 **The local-first premise fails. The project cannot proceed** |
| 2 | `DEP-SCH-02` | Confirmation that classroom devices can reach a school-hosted service | IT | `BLOCKED` | 🔴 **The project cannot proceed** |
| 3 | `DEP-SCH-03` | Whether client/AP isolation is enabled | IT | `UNKNOWN` | 🔴 **Silent failure** — the display simply never loads, with no error |
| 4 | `DEP-SCH-04` | Firewall rules permitting the display segment to reach the server | IT | `BLOCKED` | No deployment |
| 5 | `DEP-SCH-05` | A spare always-on machine with a network port and a UPS | IT | `UNKNOWN` | No deployment |
| 6 | `DEP-SCH-06` | Windows edition and build on classroom boards | IT | `UNKNOWN` | Kiosk lockdown may be impossible; the display design changes |
| 7 | `DEP-SCH-07` | Local administrator rights on kiosk accounts | IT | `UNKNOWN` | No self-start, no self-update; every change is a site visit |
| 8 | `DEP-SCH-08` | Whether one non-critical board may be used for a kiosk test | IT | `BLOCKED` | Cannot validate the display design at all |
| 9 | `DEP-SCH-09` | Who owns the master timetable | Timetable staff | `UNKNOWN` | 🔴 **The core feature has no authoritative source** |
| 10 | `DEP-SCH-10` | Who approves a substitution, and whether a second person must countersign | Timetable staff | `UNKNOWN` | 🔴 **The core feature has no publication path** |
| 11 | `DEP-SCH-11` | Whether publication to classroom screens is approved practice | Administration | `BLOCKED` | No display feature |
| 12 | `DEP-SCH-12` | Whether a staff identity directory exists | IT | `UNKNOWN` | CloudLight owns identity, provisioning and offboarding |
| 13 | `DEP-SCH-13` | Whether SSO may be used | Administration | `BLOCKED` | No SSO; standalone accounts only |
| 14 | `DEP-SCH-14` | Whether shared storage exists, and what it is | IT | `UNKNOWN` | ⛔ **All resource features are dropped** |
| 15 | `DEP-SCH-15` | Permission for a dedicated read-only service account | IT | `BLOCKED` | No indexing |
| 16 | `DEP-SCH-16` | Which folders may be indexed | Administration | `BLOCKED` | No search |
| 17 | `DEP-SCH-17` | Whether file contents may be indexed (a derived copy), including OCR | Administration | `BLOCKED` | Metadata-only search |
| 18 | `DEP-SCH-18` | Whether any external AI service is permitted | Administration | `BLOCKED` | ⛔ **No assistant feature, ever** |
| 19 | `DEP-SCH-19` | Whether unattended automated software deployment is permitted | IT | `BLOCKED` | Manual, logged deploys only |
| 20 | `DEP-SCH-20` | Where release signing keys would be held | Administration + IT | `BLOCKED` | Updates cannot be securely distributed |
| 21 | `DEP-SCH-21` | A named approver for data processing and retention | Administration | `BLOCKED` | No personal data may be stored at all |
| 22 | `DEP-SCH-22` | Whether a pilot is permitted, and on what terms | Administration | `BLOCKED` | No validation is ever possible |
| 23 | `DEP-SCH-23` | Budget, and over what period | Administration | `UNKNOWN` | Nothing can be procured |
| 24 | `DEP-SCH-24` | Whether photographing a whiteboard is permitted, and whether the result may ever be authoritative | Timetable staff | `UNKNOWN` | That capture path is closed |

---

## 2. Dependencies on SHANA

⚠️ **CloudLight cannot satisfy any of these alone.** Each requires the SHANA owner.

| # | ID | Dependency | Owner | Status | If never satisfied |
|---|---|---|---|---|---|
| 25 | `DEP-SHANA-01` | A documented, permitted delivery-request interface | SHANA custodian + vendor | `UNKNOWN` | CloudLight ships without the robot feature |
| 26 | `DEP-SHANA-02` | **Agreement on message schemas between both owners** | **Both** | `AGREEMENT_REQUIRED` — **never discussed** | 🔴 No integration |
| 27 | `DEP-SHANA-03` | **Mutual authentication between CloudLight and SHANA** | **Both** | `AGREEMENT_REQUIRED` | 🔴 **Unsafe.** An unauthenticated endpoint lets any host on the network request a task |
| 28 | `DEP-SHANA-04` | **An idempotency mechanism expressible in SHANA's schema** | **Both** | 🔴 **GAP** — SHANA's `delivery.schema.yaml` has no idempotency field | 🔴 **Duplicate physical task possible.** A safety event, not a data bug |
| 29 | `DEP-SHANA-05` | A distinct "rejected on policy" outcome, separate from "failed" | Both | `AGREEMENT_REQUIRED` | Teachers cannot be told *why* a request was declined |
| 30 | `DEP-SHANA-06` | Agreement on which inbound commands CloudLight may send | Both | `AGREEMENT_REQUIRED` | SHANA anticipates `task.pause`/`task.resume`/`system.stop`; CloudLight's design says it must never send them |
| 31 | `DEP-SHANA-07` | At least one destination with `status: ACTIVE` | SHANA custodian | 🔴 **ZERO EXIST** — `destination.schema.yaml` defaults to `NEEDS_SURVEY` | 🔴 **No delivery can occur at all, regardless of any agreement** |
| 32 | `DEP-SHANA-08` | A documented policy for robot behaviour when CloudLight is unreachable | SHANA custodian | `UNKNOWN` | SHANA decides unilaterally; CloudLight adapts |
| 33 | `DEP-SHANA-09` | Reconciliation semantics after a restart | Both | `AGREEMENT_REQUIRED` | In-flight tasks may be orphaned or duplicated |
| 34 | `DEP-SHANA-10` | Cancellation semantics, including after pickup | SHANA custodian | `UNKNOWN` | No cancellation offered in CloudLight |
| 35 | `DEP-SHANA-11` | Whether the robot network is reachable from CloudLight's segment | IT + SHANA | `UNKNOWN` | Integration deferred indefinitely |

⚠️ **`DEP-SHANA-04` and `DEP-SHANA-07` are the two most serious.**
The first is a genuine gap between two independently drafted schemas — CloudLight
requires deduplication, SHANA's schema cannot express it. The second means the
robot feature is blocked by a physical site survey, not by any software work.

---

## 3. Dependencies on external parties

| # | ID | Dependency | Owner | Status | If never satisfied |
|---|---|---|---|---|---|
| 36 | `DEP-EXT-01` | Robot vendor confirming the interface and any secondary-development authorisation | SHANA custodian + Unitree | `REQUIRES_VENDOR_OR_SERVICE_PROVIDER` | No robot feature |
| 37 | `DEP-EXT-02` | NAS vendor documentation, if a NAS is used | IT | `UNKNOWN` | Protocol-level integration only |
| 38 | `DEP-EXT-03` | A source for model weights, if local AI were ever approved | — | Not applicable | Local inference impossible without an air-gap transfer procedure |

---

## 4. Process dependencies

| # | ID | Dependency | Owner | Status | Impact |
|---|---|---|---|---|---|
| 39 | `DEP-PROC-01` | SHANA's revert of `docs/shana/` → `docs/` completes | SHANA | `PROCESS` | Until then, 9 CloudLight links are temporarily broken. **Recorded, not our to fix** |
| 40 | `DEP-PROC-02` | The shared ADR index lists ADR-013 and ADR-014 | Shared | `PROCESS` | Two accepted decisions are not discoverable from the index |
| 41 | `DEP-PROC-03` | A first Git commit | Repository owner | `PROCESS` | ⚠️ **0 commits.** Nothing is version-controlled; there is no history and no rollback |
| 42 | `DEP-PROC-04` | The plaintext credential in `start_opencode_server.sh` rotated by its owner | File owner | `OPEN` | Gitignored, so not committed — but not rotated |
| 43 | `DEP-PROC-05` | A decision on whether the CloudLight flat layout is restructured into the subfolder convention | CloudLight | `OPEN` | Cosmetic, but causes recurring link-check friction |

---

## 5. What CloudLight can do without any dependency

⚠️ **Not nothing.** These require no school answer, no vendor, and no robot:

| # | Action | Why it is safe now |
|---|---|---|
| 1 | Model the timetable and substitution data structure against **synthetic** fixtures | The model shape can be explored before any real timetable is seen |
| 2 | Prototype **deterministic** spreadsheet and CSV parsing against synthetic files | No model involved; no school data; no permission needed |
| 3 | Implement the schedule **conflict-detection rules** as pure functions with synthetic inputs | Deterministic, testable, no external dependency |
| 4 | Design and render the **degraded-state** contract (no network, no snapshot, quarantined record) | Pure presentation logic |
| 5 | Draft the timetable approval workflow **for staff to review on paper** | A document, not a system |
| 6 | Finalise the **risk register, decision log and claim-evidence matrix** | Already largely done |
| 7 | Draft the **interface contract proposal** for a joint session with SHANA | Requires no robot and no school |
| 8 | Write **acceptance criteria and a test plan** that can be executed the moment code exists | Already drafted |

⚠️ **This is the recommended immediate path.** It advances the project without
depending on any answer, and it front-loads exactly the deterministic, safety-
critical work that should be done first.

Deliberately **not** on that list: installing anything on a smartboard, probing
the network, contacting any external AI service, or writing anything that would
need rewriting once the real timetable structure is known.

---

## 6. Sequencing

```
  ASK (parallel, week 1)          BUILD-FREE (parallel, now)      JOINT (needs both)
  ───────────────────────         ─────────────────────────      ──────────────────
  DEP-SCH-01  permission  ┐                                ┌──── DEP-SHANA-02  schemas
  DEP-SCH-02  reachability│                                │──── DEP-SHANA-03  authn
  DEP-SCH-03  isolation   ├─→ gates pass ─→ first release  │──── DEP-SHANA-04  idempotency
  DEP-SCH-09  timetable   │                    built       └──── DEP-SHANA-07  a destination
  DEP-SCH-10  approval    │                                  ↑
  DEP-SCH-06  Windows     ┘                            PHYSICAL SITE SURVEY
  DEP-SCH-14  storage?          deterministic parsing         (SHANA-owned,
                                conflict rules               long lead time —
                                data model                  start the email now)
```

⚠️ **The SHANA column cannot start without a joint session between both owners,
and the destination survey cannot be shortened.** Both have long lead times and
should be started in parallel with the school questions, not after them.

---

## Related

- [`../open_questions_register.md`](../open_questions_register.md) — the questions behind these
- [`../next_actions.md`](../next_actions.md) — the ordered action list
- [`implementation_inventory.md`](implementation_inventory.md) — component status
- [`../shana_integration.md`](../shana_integration.md) — CloudLight's interface proposal
- `docs/shana/handoff/unresolved_dependencies.md` — SHANA's own dependency view (read-only)