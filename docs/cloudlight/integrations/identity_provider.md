# CloudLight — Identity Provider Integration

**Phase 0 · DESIGN ONLY · BLOCKED · 2026-10-11**

> ## ⛔ **No identity provider has been chosen. None has been contacted. No SSO
> configuration exists. No identity system of any kind has been configured,
> tested, or provisioned.**
>
> **Three models are compared below. All three are proposals. The decision is
> blocked on `CL-AUTH-001`, `CL-AUTH-002` and `CL-AUTH-003` — all `UNANSWERED`.**

> ### ⚠️ **This document must not be read as a description of an integration.**
> It describes **three options and what each would cost**, so that a school
> answering `CL-AUTH-001` has something concrete to answer *about*.

Deeper treatment of identity mechanics — account lifecycle, MFA, sessions,
recovery, secrets, the delivery-request flow — is in
[`../authentication_and_authorisation.md`](../authentication_and_authorisation.md).
This document covers **the choice of provider** only, and does not repeat it.

---

## 1. The finding, stated up front

> **Authentication answers *who are you?* and produces a verified identity — a
> claim that "this is Ms Rao". Authorisation answers *what may you do?* and
> produces a permission decision against a specific action and resource.**
>
> ⚠️ **A shared six-digit classroom code answers neither.** It proves that
> *somewhere in this room, someone knows six digits* — that is the entire content
> of the assertion.

⚠️ **The asymmetry that makes this obvious once stated:** everything else
CloudLight does is reversible. A wrong timetable is corrected. A bad screen is
reset. **An unattributable action is not reversible at all**, because there is
nothing to reverse *against* — no person to ask, no session to revoke, no account
to disable.

⚠️ **What is lost together when accounts are shared:** **attribution** (the audit
log records an event but not a person, and `CL-SEC-020` is P0, so a shared account
makes it unfillable); **revocation** (you cannot revoke a room — revoking a shared
code locks out the next class as well as the leaver, `CL-AUTH-022`); and
**accountability** (when a delivery request is questioned, *"a teacher did it"* is
not an answer, `CL-SEC-015`).

---

## 2. The three models

| | **A. School directory via OIDC/SAML** | **B. CloudLight-managed accounts** | **C. Shared classroom PIN** |
|---|---|---|---|
| **What it proves** | That a person holds an identity the school already issues, revokes and audits | That a named human was provisioned by a named administrator | ⛔ **Nothing about identity.** Only that someone in the room knows a value |
| **Guessing space** | N/A — the identity provider's second factor applies | N/A — the identity provider's second factor applies | ⚠️ **10⁶.** Expected 500,000 guesses; a free period is 40–45 minutes |
| **Our effort** | 🟢 Low — a standard protocol, well documented | 🟡 Medium — annual provisioning plus mid-term joins | 🟢 Trivial — ⚠️ **that is the problem** |
| **School effort** | 🔴 **High.** IT must configure a relying party, grant claims, and own a new integration | 🟢 None | 🟢 None |
| **Permission required** | ⚖️ **Written school authorisation** to authenticate against the directory | Written agreement that CloudLight may hold the staff list | ⚖️ Written acknowledgement of what it does and does not prove |
| **Offboarding risk** | 🟢 **Follows the school's HR process** — with the caveat below | 🟠 **Entirely ours.** We own it, therefore we must do it | ⛔ **Cannot be revoked from a person, only from a room** |
| **Audit trail** | ✅ Attributable by construction | ✅ Attributable, *if* offboarding actually happens | ⛔ **Fictional.** No person can be named |
| **Revocation granularity** | One person | One person | One room — and every class using it |
| **Verdict** | ✅ **Preferred if it exists and is permitted** | ✅ **The realistic fallback, with costs** | ⛔ **Rejected as identity** |

---

### 2.1 Model A — School directory via OIDC/SAML

> **Preferred, conditional on `CL-AUTH-001` returning "yes, and permission has
> been given".**

| | |
|---|---|
| **How it would work** | CloudLight becomes a **relying party**. Staff authenticate against the school's own directory — Active Directory, LDAP, Google Workspace, Microsoft 365 — over OIDC or SAML. CloudLight receives a signed assertion and derives identity and group membership from it |
| **What it proves** | That a person holds an identity the school itself issues and revokes |
| **Why preferred** | ⛔ **It makes the offboarding problem the school's, where it already works.** ⚠️ And it gives CloudLight group membership, which is what permission-filtered search needs at query time |
| **What it requires from the school** | ⚠️ **Real work.** A relying party registration, claim mapping, group export, and a new integration in an estate that has one IT generalist. ⚠️ **The school's cost is higher than ours, and that asymmetry is why it may be refused** |
| **Blockers** | ❓ Does a directory exist — `CL-AUTH-001`. ❓ What is it — `CL-AUTH-002`. ⚖️ Is SSO permitted — `CL-AUTH-003`. ⚠️ **All three `UNANSWERED`** |
| **Failure mode if it is attempted and abandoned halfway** | ⚠️ **A half-configured relying party is an authentication surface with no owner.** Configuration must be removed as carefully as it was added |

⚠️ **The directory's existence is `UNKNOWN`, not assumed.** Whether this school runs
Active Directory, Google Workspace, Microsoft 365, nothing at all, or a
spreadsheet has never been established.

⚠️ **A caveat that must not be skipped: a directory does not remove the
offboarding problem, it relocates it.** If a teacher leaves and HR updates the
directory, the directory is authoritative and we inherit that. But if the
directory is stale — the normal condition in a small school where one person
maintains it — CloudLight inherits that staleness too. ⚠️ **The directory's
accuracy becomes CloudLight's permission accuracy.**

### 2.2 Model B — CloudLight-managed accounts

> **The most likely branch, and it has consequences nobody enjoys.**
> `CL-AUTH-001` returning "no directory" is a completely normal answer for a
> school of this size.

⚠️ **If the school has no directory, CloudLight becomes the system of record for
teacher identity.** That is a significant, quiet transfer of responsibility that
nobody in a feasibility conversation tends to notice happening.

| Consequence | Detail |
|---|---|
| **Offboarding becomes entirely ours** | ⚠️ An account survives until someone removes it — `CL-AUTH-007`, `CL-ADMIN-006` |
| **HR and CloudLight diverge silently** | ⚠️ **This is the normal failure mode, not the edge case.** A leaver is removed from the staff list and CloudLight is never told |
| **Annual provisioning becomes a security event** | ⚠️ Identity accuracy in September determines permission accuracy for the rest of the year — `CL-ADMIN-005` |
| **The Principal becomes a dependency** | Offboarding confirmation is a human process with no automation, and human processes at the end of a term fail |
| **A live credential outlives employment** | ⚠️ **The sentence to quote to the Principal.** A teacher who leaves in March still holds a working credential unless explicitly revoked |
| **Revocation must be immediate and server-side** | "At next login" is not revocation — `CL-AUTH-022`, `CL-SEC-004`. Existing sessions are **killed**, not merely prevented from renewing |

⚠️ **The recommendation in [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md)
§2.2 is to treat the annual staff list as a *signed artefact*** — produced by the
school, dated, accepted by a named person. Not an export emailed at some point and
never refreshed.

⚠️ **Recovery must be at least as strong as the factor it replaces.** It is
routinely the weakest link and is called out explicitly in
`../authentication_and_authorisation.md` §10. A teacher locked out at 07:50 either
has a same-day recovery or does not use the system — `CL-AUTH-014`, `FR-AUTH-10`.

### 2.3 Model C — Shared classroom PIN ⛔ REJECTED AS IDENTITY

> **This is the model the original brief proposed. It cannot be used as identity,
> and the reasoning is not a preference.**

| Evidence | Detail |
|---|---|
| **Guessing space** | ⚠️ **10⁶ possibilities.** Expected 500,000 guesses at ~10/s. A free period is 40–45 minutes. **Two are enough** |
| **Visibility** | A classroom display is **shared and frequently unattended**. Any code shown can be photographed by every child in the room |
| **Attribution** | It identifies nobody. An unauthorised delivery request would be **unattributable** |
| **Audit** | ⛔ **Cannot answer "which teacher did this"** — which makes `CL-SEC-020` unfillable |
| **Revocation** | Cannot revoke one person without revoking everyone |
| **The failure mode nobody designs for** | ⚠️ **It looks like authentication.** A code prompt, a green tick, a success message — teachers, administrators *and auditors* all believe an identity was established when none was. ⛔ **A successful code entry must never be presented as "signed in as…"** |

⚠️ **Where a six-digit code IS permitted:** authorising **one** specific action, or
acting as a **second factor** behind a real first factor. Nothing else.

⚠️ **Where it is NOT permitted:** authenticating a named teacher; standing in for
an individual account; a shared per-classroom credential. ⛔ **Never.**

⚠️ **Required controls wherever a code appears:** short lifetime; single use;
attempt limits and lockout with administrator notification (`CL-SEC-003`); rate
limiting on **resend** as well as on entry — because resend is the standard bypass
of an attempt limit; absent from URLs, logs, screenshots, configuration and Git.

⚠️ **The specific action the brief contemplated — a teacher asking a robot to move
something through a corridor — is exactly the action that must not rest on a shared
code.** Individual authentication, plus a code only if the school decides a second
factor is acceptable.

---

## 3. The open questions, mapped

⚠️ **These three questions are the whole of the block. None has been asked, and
none has been answered.**

| ID | Question | Priority | What the answer decides | If unanswered |
|---|---|---|---|---|
| **`CL-AUTH-001`** | Does the school maintain a **staff identity directory** — one list of teachers, each with a stable identifier? | **P0** | ⚖️ **Every authorisation decision and the entire audit trail** | ⚠️ Assume **no directory.** Consequence: Model B, and CloudLight owns identity and offboarding |
| **`CL-AUTH-002`** | What **kind** of directory does it use — Active Directory, LDAP, Google Workspace, Microsoft 365, or something else? | **P0** | The protocol, the claims available, the integration effort | ❓ Unknown. Model A cannot be scoped at all |
| **`CL-AUTH-003`** | Is CloudLight **permitted to authenticate against it**, in writing? | **P0** | Whether Model A is available at all | ⛔ Assume **not permitted.** Falls back to Model B |

| Related | Question | Priority | Notes |
|---|---|---|---|
| **`CL-AUTH-004`** | Must every action be **attributable to one named teacher**, or is a shared classroom account acceptable? | **P0** | Whether per-user accounts are required *at all*. ⚠️ Assume **individual accounts for anything consequential** |
| **`CL-AUTH-005`** | If a shared account is used, what is it permitted to do? | P1 | The never-list, if the school insists |
| **`CL-AUTH-006`** | May CloudLight **create and manage accounts** itself? | P1 | Model B's authority |
| **`CL-AUTH-007`** | Within what window must a leaver's access **end**? | P1 | A school HR process property, not a technical one |
| **`CL-AUTH-008`** | When does **provisioning** happen — at year start, or per join? | P1 | Whether day one works |
| **`CL-AUTH-010`** | Is TLS required on the internal LAN, and is a self-signed certificate acceptable? | P1 | Whether SSO assertion transport works at all |
| **`CL-AUTH-012`** | Would teachers accept an **extra sign-in step** on every classroom login? | P1 | ⚠️ The adoption risk is **larger** than the security risk |
| **`CL-AUTH-014`** / **`CL-AUTH-015`** | What is the **recovery** path, and does it need the Principal for a routine reset? | P1 | A teacher locked out at 07:50 |
| **`CL-AUTH-020`** / **`CL-AUTH-021`** | May an **unattended display** hold a signed-in teacher session? | **P0** | ⚠️ ⛔ Recommend **no**. A persistent session on a shared panel means anyone in the room inherits teacher authority |
| **`CL-CODE-002`** / **`CL-CODE-015`** | Written acknowledgement that a code may **not** authenticate a person | **P0** | Model C's permanent rejection |
| **`CL-ADMIN-005`** / **`CL-ADMIN-006`** | If CloudLight owns accounts: who provisions, and who offboards? | **P0** | Model B's operating cost |

⚠️ **Every ID above was verified by grep against**
[`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md).
**None was invented.** All are `UNANSWERED`.

---

## 4. What a choice would commit us to

⚠️ **Whichever model is chosen, three things follow and none is cheap.**

| | Model A (directory) | Model B (CloudLight-owned) | Model C (shared PIN) |
|---|---|---|---|
| **Offboarding** | Inherited from HR | ⚠️ **Ours. Entirely** | ⛔ Impossible |
| **Audit trail** | Attributable | Attributable *if* offboarding happens | ⛔ Fictional |
| **Permission-filtered search** | ⚠️ Needs group membership, which we get from the directory | ⚠️ Requires us to model group membership ourselves | ⛔ Not possible |
| **Failure if wrong** | Integration refused after we build against it | ⚠️ Staff list diverges; permissions quietly over-grant | ⛔ A shared code becomes identity in practice — `R-22` |

⚠️ **The realistic sequencing** — and it is not the one the brief implies:

```
CL-AUTH-001 (does a directory exist?)
    │
    ├── YES ──► CL-AUTH-003 (permitted?) ──► Model A
    │                                             │
    │                                     NO ────┴──► Model B
    │
    └── NO ─────────────────────────────────► Model B  ← most likely branch
```

⚠️ **Model B is the likely default, and it should be scoped honestly rather than
treated as a fallback.** If the school has no directory, CloudLight takes on
provisioning, offboarding, a signed annual staff list, a named offboarding
authority, and a documented divergence risk — and **that is a legitimate, workable
arrangement**, provided somebody at the school agrees to own the staff list.

---

## 5. What has not been done

⚠️ **Recorded explicitly, because this document could otherwise be misread as
describing an integration.**

| | Status |
|---|---|
| Identity provider selected | ⛔ **No.** Blocked on `CL-AUTH-001`–`003` |
| Identity provider contacted | ⛔ **No.** No school IT team has been approached about any integration |
| SSO configuration exists | ⛔ **No.** No relying party, no claim mapping, no directory access |
| Accounts provisioned | ⛔ **No.** Zero accounts exist |
| Group membership sourced | ⛔ **No.** Permission-filtered search has nothing to filter by |
| Second factor implemented | ⛔ **No.** Passkeys, TOTP and code mechanics are *designed*, not built |
| Session management implemented | ⛔ **No.** Idle and absolute timeouts are *specified* in `NFR-SEC-03`, not built |
| Audit log exists | ⛔ **No.** Requirements are `FR-ADMIN-03`, `FR-ADMIN-04` |
| **Identity decision made by anyone** | ⛔ **No.** Every relevant question is `UNANSWERED` |

---

## 6. A note on the temptation this document creates

⚠️ **This document makes three integration paths look tractable. None of them is.**

Each is a well-understood piece of engineering. What none of them includes is the
part that actually determines whether it succeeds: **a school IT team with one
person and other duties, being asked to configure a new relying party in an estate
nobody has documented.**

⚠️ **A project team that reads this document and concludes "identity is solved,
let's start" has misread it.** Every path here is blocked on three `P0` questions
that require a conversation with a person. The engineering is the easy part, and it
has not started.

---

## 7. Related documents

| Document | Covers |
|---|---|
| [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) | The full identity treatment — lifecycle, MFA, codes, sessions, recovery, the delivery flow |
| [`../security_and_privacy.md`](../security_and_privacy.md) | Threat model; `T01`, `T18`; data classification; secrets handling |
| [`../requirements.md`](../requirements.md) | `FR-AUTH-01`…`14`, `NFR-SEC-03`…`09` |
| [`../open_questions_register.md`](../open_questions_register.md) | `CL-AUTH-001` as blocking question `OQ-8` |
| [`../risk_register.md`](../risk_register.md) | `R-21` no directory · `R-22` shared code as identity · `R-23` unattended board session |
| [`../decision_log.md`](../decision_log.md) | `D-005` the code rules · `D-102` identity source, **OPEN** |
| [`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md) | 576 questions. **0 answered** |