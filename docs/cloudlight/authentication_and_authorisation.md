# CloudLight — Authentication, Authorisation and Action Codes

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ NOTHING IN THIS DOCUMENT HAS BEEN TESTED
>
> **No identity provider has been contacted. No directory has been enumerated. No
> credential was created, attempted, guessed or stored. No smartboard has been
> signed in to. No session was opened. No code was issued.**
>
> Every school-specific statement below is an **open question** routed to a named
> human. If a sentence here reads like a fact about this school, it is wrong.

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in a published standard or authoritative vendor document, cited in §Sources |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | A standard or vendor documents it; not confirmed against any device this school owns |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No source claims it. Our engineering judgement |
| ❓ **UNKNOWN** | No evidence either way. Deliberately not assumed |
| ⛔ **NOT RECOMMENDED / NOT PERMITTED** | Rejected by our judgement, by published standards guidance, or by a school policy boundary once known |

---

## 1. The identity question, stated precisely

### 1.1 The finding, stated up front

> ### ⚠️ **A shared six-digit classroom code is NOT sufficient proof of teacher identity.**
>
> **A six-digit code may only ever be either:**
> **(a) an authorisation for one specific action, or**
> **(b) a second factor behind a real first factor.**
>
> **It may never stand in for a named teacher.**

⚠️ This is not a preference or a hardening suggestion. It is the load-bearing
security finding of this document, and it is `CL-CODE-002` (P0) — recorded in
`../../questionnaires/Additional_CloudLight_Questionnaire.md` as requiring
**written acknowledgement from school administration**.

⚠️ **The asymmetry that makes this obvious once stated:** everything else
CloudLight does is reversible. A wrong timetable is corrected. A bad screen is
reset. **An unattributable action is not reversible at all**, because there is
nothing to reverse *against* — no person to ask, no session to revoke, no account
to disable.

**Authentication** answers *who are you?* and produces a verified identity — a
claim that "this is Ms Rao". **Authorisation** answers *what may you do?* and
produces a permission decision against a specific action and resource. Getting
authentication wrong means a person acts as someone they are not; getting
authorisation wrong means a verified person does something they should not have
been able to do. ⚠️ **A shared classroom code answers neither question.** It proves
that **somewhere in this room, someone knows six digits** — the entire content of
the assertion. It names nobody, cannot be revoked against a person, and survives
the departure of everyone who ever knew it.

⚠️ **The most common design error** is to build a flow that *looks* like it
authenticates — a code prompt, a success message, a green tick — so that teachers,
administrators and auditors all believe an identity was established when none was.
**A successful code entry must never be presented as "signed in as…"**

### 1.2 ⚠️ The guessing space, quantified

🧪 **Keyspace 10⁶ (1,000,000). Expected guesses to hit: 500,000 — half the
keyspace, on average.** At a hand-typed rate of roughly 10 guesses per second, a
live unthrottled system falls in about 14 hours. ⛔ **A free period is 40–45
minutes. Two is enough.**

⚠️ **The conclusion is not "six digits is too short".** Six digits is a reasonable
human-typable length and this document does not propose replacing it. The
conclusion is:

> **Six digits is only safe because it is short-lived, single-use, attempt-limited
> and rate-limited. Remove any one of those four and the keyspace becomes the
> vulnerability.**

| Without… | The attack becomes… |
|---|---|
| **Attempt limiting** | A one-million-entry keyspace searched at human speed. The primary threat in `CL-SEC-003` |
| **Short lifetime** | A code displayed on a wall for a lesson, photographed by every child in the room |
| **Single use** | One observed entry is enough; the code is a reusable classroom credential |
| **Rate limiting per classroom** | Attempts are spread across rooms so no single counter ever trips |

🟡 **NIST SP 800-63B** states that verifiers should limit **failed authentication
attempts on a single account** to no more than **100 consecutive attempts**.
✅ VERIFIED — §Sources 2. ⚠️ **That is a ceiling, not a target.** ⚠️ **The
classroom code is far weaker than that ceiling assumes**, because it is not tied to
an account — there is no "consecutive attempts on a single account" to count
against. The binding is per-classroom, and it **resets whenever a fresh code is
issued.** This is why §6 requires limits on **code issuance** as well as on **code
entry**.

---

## 2. Identity sources

⚠️ **No option below has been evaluated against this school.** Whether the school
runs Active Directory, Google Workspace, Microsoft 365, nothing at all, or a
spreadsheet, is ❓ **UNKNOWN** — `CL-AUTH-002`.

| Option | What it proves | Provisioning burden | Offboarding risk | What the school must decide | IDs |
|---|---|---|---|---|---|
| **School directory / SSO** (AD, LDAP, Google Workspace, M365 — via OIDC or SAML) | That a person holds an identity the school already issues, revokes and audits | 🟢 Low for us, **high for school IT** | 🟢 Follows the school's HR-driven process | Whether CloudLight may authenticate against it, **in writing** | `CL-AUTH-001` – `-003` |
| **CloudLight-managed accounts** | That a named human was provisioned by a named administrator | 🟡 Medium — annual provisioning plus mid-term joins | 🟡 **Entirely ours.** We own it, therefore we must do it | Who holds provisioning authority, and from which authoritative staff list | `CL-AUTH-006`, `CL-AUTH-008`, `CL-ADMIN-005`, `CL-ADMIN-006` |
| **Shared classroom PIN or code** | ⛔ **Nothing about identity.** Only that someone in the room knows a value | 🟢 Trivial — that is the problem | ⛔ **Cannot be revoked from a person, only from a room** | ⛔ Whether to accept this at all, and if so exactly where | `CL-AUTH-004`, `CL-AUTH-005`, `CL-CODE-001` |

### 2.1 ⚠️ The default assumption

> **Individual teacher accounts are required for anything consequential.**
> Shared accounts destroy attribution, revocation and accountability — all three
> at once, and none of them recoverable after the fact.

⚠️ **Why this is a default and not a debate.** The three properties shared
accounts destroy are not style preferences; they are the mechanisms by which a
security control functions. **Attribution** — the audit log records an event but
not a person, and `CL-SEC-020` (P0) makes the log a requirement, so a shared
account makes it unfillable. **Revocation** — you cannot revoke a room; revoking
a shared code locks out the next class as well as the leaver (`CL-AUTH-022`,
`CL-ADMIN-006`). **Accountability** — when a delivery request is questioned,
*"a teacher did it"* is not an answer (`CL-ADMIN-003`, `CL-SEC-015`).

⚠️ **"Consequential" is defined by this document pending `CL-CODE-003`:** any
action that reaches a robot, changes published data, changes a permission, or
discloses material not otherwise visible in the room.

### 2.2 ⚠️ If the school has no directory, CloudLight owns identity

⚠️ **This is the most likely branch and it has consequences nobody enjoys.**
`CL-AUTH-001` (P0) asks whether a staff identity directory exists at all.

⚠️ **CloudLight becomes the system of record for teacher identity** — the
authoritative list of who teaches here becomes a security artefact, and
**offboarding becomes entirely ours**, because an account survives until someone
removes it (`CL-AUTH-007`, `CL-ADMIN-006`). ⚠️ **HR and CloudLight will diverge
silently**: a leaver is removed from the staff list and CloudLight is never told.
**This is the normal failure mode, not the edge case.** ⚠️ **Annual provisioning
becomes a security event** — identity accuracy in September determines permission
accuracy for the rest of the year (`CL-ADMIN-005`) — and **the Principal becomes a
dependency**, because offboarding confirmation is a human process with no
automation, and human processes at the end of a term fail.

⚠️ **Recommendation: treat the annual staff list as a signed artefact** — produced
by the school, dated, accepted by a named person, not an export emailed at some
point and never refreshed.

---

## 3. Account lifecycle

⚠️ **Most schools treat offboarding as administrative tidiness. It is a security
control.** `CL-AUTH-007` (P1) and `CL-ADMIN-006` (P0) exist because the same
failure is described from two directions.

| Phase | What must happen | Failure if it does not | IDs |
|---|---|---|---|
| **Start of academic year** | Every staff member needing access is provisioned from the authoritative staff list, before first use | Staff cannot work on day one, or the fix is to disable authentication entirely | `CL-AUTH-008`, `CL-ADMIN-005` |
| **Mid-term join** | A new or substitute teacher is provisioned quickly enough to teach on their first day | The pilot's most common complaint: "the new teacher could not sign in" | `CL-ADMIN-016`, `CL-AUTH-006` |
| **Transfer** | Rights change; the account survives | A transferee keeps access to the old department's resources | `CL-ADMIN-006` |
| **Extended absence** | A defined, logged decision: suspend, or leave active | Undefined, the account stays active through a sabbatical | `CL-AUTH-006` |
| **Departure** | Access ends **immediately** — not at next login, not next term | ⛔ **A live credential in the hands of a former member of staff** | `CL-AUTH-007`, `CL-AUTH-022`, `CL-ADMIN-006` |
| **Guest / visiting** | Time-limited by construction, with an expiry nobody has to remember | ⛔ Guest accounts accumulate into a permanent, unowned access path | `CL-ADMIN-016` |

⚠️ **An account which outlives employment is a live credential** — the sentence in
this section to quote to the Principal. A teacher who leaves in March still holds
a **working credential** unless explicitly revoked; **knowledge of the school's
staffing** (where every room is, who teaches what, when rooms are empty);
**knowledge of how the system is used**, including what an approval prompt means;
and **any credential the school never asked to be rotated**. ⚠️ **And it will not
be used maliciously in most cases** — by a former colleague helping out, a
contractor "just checking", or nobody at all. The control is not about assuming
bad faith; it is about the fact that the access cannot be ended if we never noticed
it existed.

**Offboarding requirements:** revocation takes effect **immediately and
server-side** ("at next login" is not revocation — `CL-AUTH-022`, `CL-SEC-004`);
existing sessions are **killed, not merely prevented from renewing**; a **named
human confirms the departure**, because there is no HR feed and CloudLight cannot
learn of a resignation by itself (`CL-AUTH-007`); the offboarding is **logged**,
or there is no evidence it happened; a **periodic review of who holds access**
exists, because the people who *forget* to offboard are not the ones who forget
deliberately (`CL-AUTH-008`); and **content ownership transfers**, because
resources created by a departing teacher become orphaned and invisible within a
year — a content problem with a security cause (`CL-RES-011`). ⚠️ **A periodic
access review is the cheapest control in this document:** it needs no integration,
catches most real failures, and is the standard answer to "how do you know the
accounts are right?".

---

## 4. Multi-factor authentication

⚠️ **MFA is the single most effective defence against credential compromise.** A
password alone protects an account only as long as the password stays secret; a
second factor changes the question from *"do you know the password"* to *"do you
also have the device"*. 🟡 NIST SP 800-63B requires authentication at **AAL2** —
two distinct factors — where compromise would be significant. ✅ VERIFIED —
§Sources 2.

| Rank | Factor | Strength | Notes for this school |
|---|---|---|---|
| **1** | **Passkeys / WebAuthn** | 🟢 **Phishing-resistant. The private key never leaves the authenticator** and is never transmitted | 🧪 Best fit for the classroom problem: nothing is typed on a student-reachable screen, so **no code is ever exposed on the board**. ❓ Device capability — `CL-AUTH-013` |
| **2** | **TOTP from an authenticator app** (RFC 6238) | 🟢 Strong. Time-based, single-use, never transmitted off-device. ⚠️ Phishable if a teacher types it into a convincing fake page | 🧪 Most realistic second factor where passkeys are unavailable. ❓ `CL-AUTH-011` |
| **3** | **Push notification** | 🟡 Moderate, **with two named weaknesses**: *push bombing* / MFA fatigue (repeated prompts until one is approved), and unsolicited entry. ✅ OWASP and NIST both require **number matching** — the user types the number shown in the prompt, defeating both | 🧡 Acceptable **only** with number matching and rate limiting |
| **4** | **SMS / PSTN one-time code** | 🟡 **Designated a *restricted* authenticator by NIST SP 800-63B** — subject to SS7 interception, SIM-swap fraud and number porting outside the user's control | ⛔ **NOT RECOMMENDED for anything handling school data** |

⚠️ **The SS7 point deserves one sentence**, because it is counter-intuitive: an
SMS code is delivered over a network whose routing and subscriber records are not
under the school's control. A correct code can be intercepted or redirected
without any visible symptom and without any credential being stolen from the
device. NIST's "restricted" classification reflects this, not a general distrust
of phones.

### 4.1 ⚠️ The adoption risk is larger than the security risk

⚠️ `CL-AUTH-012` (P1) asks whether teachers would accept an extra sign-in step
during a busy period. **This is a usability question with a security consequence,
and it cuts both ways.** A control that is too strict produces staff who share a
login, write credentials on a board, or ask a colleague to "just do it once" — ⛔
which trains staff that the control can be bypassed, and a control staff route
around is not a control. A control that is too loose produces a student with a
phone acting as a teacher — ⛔ the obvious failure, but a faster one to detect.

🧪 **Design conclusion:** the right number of factors is the number teachers will
actually use under time pressure. A second factor required **once per day** is a
session-level property; one required **at every kiosk touch** is an
interaction-level one, and the second will not survive contact with a timetable
change at 08:40. ⚠️ **And the recovery path (§10) is what makes strictness
survivable** — a control whose failure mode is "the lesson stops" is a control
that gets disabled.

### 4.2 One-time code requirements — wherever codes appear

🟡 These apply to **every** one-time code in this system, including the classroom
code of §6. ✅ VERIFIED — §Sources 1, 2. Codes must be **short-lived** (NIST
recommends OTP validity of **ten minutes or less**; 🧪 a classroom action code
should be far shorter — §6.1); **single-use and invalidated immediately on
success**, because ⛔ the check-without-the-check bug is the most common OTP defect
in real systems; **attempt-limited**, because without a limit the keyspace *is* the
vulnerability (§1.3); generated with a **cryptographically secure RNG**, because
⛔ `Math.random()`, `rand()`, a timestamp or a counter is not a random number
generator for this purpose and OWASP calls out standard RNGs as a weakness in token
generation; **8 digits or longer where usability allows**, since NIST states
single-factor authenticators *should* be at least 8 digits and six is below that —
which is why the classroom code's safety depends entirely on §6, not on its length;
and **never stored in plaintext at rest**, because a store of usable codes is a
store of credentials.

---

## 5. The six-digit code — permitted and forbidden uses

⚠️ **This table is 🧪 our proposal, not school policy.** `CL-CODE-003` (P0) and
`CL-CODE-004` (P1) ask the school to state which actions fall on each side.

| Action | Code alone? | As second factor? | Named teacher? | Reason |
|---|---|---|---|---|
| **Request a SHANA delivery** | ⛔ **No** | 🟡 Possible | 🟢 **Yes** | A physical action with a real load and a real route. The most consequential thing this system can do. `CL-CODE-003`, `CL-SEC-016` |
| **Publish a timetable change** | ⛔ **No** | 🟡 Possible | 🟢 **Yes** | Published data is acted upon by children on every wall. `CL-SEC-014` |
| **Approve a substitution** | ⛔ **No** | 🟡 Possible | 🟢 **Yes** | Puts a named human's name on a screen as the responsible adult for a room. `CL-ADMIN-003`, `CL-SUB-003` |
| **Change permissions** | ⛔ **No — never** | ⛔ **No** | 🟢 **Yes** | Permission changes are how every other control is bypassed. `CL-ADMIN-002`, `CL-ADMIN-010` |
| **View a confidential staff folder** | ⛔ **No — never** | ⛔ **No** | 🟢 **Yes** | ⛔ Discloses personnel material. No version of this is authorisable by a room-shared code. `CL-DASH-003`, `CL-SEC-010` |
| **Browse / open a resource** | 🟡 Possibly | 🟢 Yes | 🟢 Yes | ⚠️ **Still requires the underlying entitlement**, checked against the signed-in identity, not against the code |
| **Clear a notice** | 🟡 Possibly | 🟢 Yes | 🟢 Yes | ⛔ **Only** the notice the authorising teacher is entitled to clear. The most defensible code-alone candidate |
| **Show the timetable** | ✅ **Already permitted unauthenticated** | — | — | ⚠️ The dashboard is read-only and unauthenticated by design (`./classroom_dashboard.md` §1). A code here would be security theatre |

### 5.1 The never-list

⛔ **Never authorised by a classroom code, in any configuration, under any school
policy, for any duration:** granting or changing a permission (a code that can
change permissions can grant itself the authority to do anything else — the
control protects itself); creating, deleting or disabling an account (same
argument — a code that manages accounts manages its own survival); changing MFA
settings; viewing personnel or confidential folders; publishing exam- or
results-related items (⛔ `CL-DASH-003` prohibits results on student-facing
surfaces at all); approving a break-glass request (break-glass exists for when
normal authorisation is unavailable — a code that approves it makes it ordinary);
signing a deployment (see `./ota_updates.md`); and anything performed while
unauthenticated (⛔ there is no first factor to pair it with, so by definition
this is §1.1's forbidden case).

### 5.2 ⚠️ A code authorises an action; it never authenticates a person

⚠️ **The wording matters as much as the mechanism.** If the interface says
*"Signed in as teacher"* after a code entry, an administrator reading the screen,
a screenshot in a ticket and a parent's question all receive the same false
answer. 🧪 **Proposed wording:** *"Action authorised by code · Ms Rao signed in at
09:12"* — the code authorises, the person remains the identity of record.

---

## 6. Code mechanics

⚠️ **Every property below is load-bearing.** §1.3 established that the six-digit
keyspace is survivable *only* because of these controls. A code feature
implementing three of the seven is worse than none, because it creates the
appearance of a controlled path where none exists.

### 6.1 Lifetime, binding, single use and rate limiting

⚠️ `CL-CODE-005` (P0), `CL-CODE-006` (P1), `CL-CODE-008` (P0) and `CL-SEC-003`
(P0) set most of the numbers below. 🧪 **Every value here is provisional — the
school must set them.** All are enforced **server-side**; ⛔ a client-side timer
is advisory.

| Property | 🧪 Proposal | Rationale |
|---|---|---|
| **Validity** | **Single-action** (preferred), or **30–60 seconds** for a displayed code | A long display window turns a one-action code into a reusable credential |
| **Binding** | One **classroom**, one **action type**, one **requesting principal** | A code valid in any room is a school-wide credential |
| **Used at most once**, and **invalidated on success** | Consumed inside the same transaction as the authorised action | So a race cannot consume it twice, and ⛔ the check-without-the-check bug does not leave a live code |
| **After success, the display returns to signed-out** | — | ⛔ Leaving it authorised gives a captured code value *after* its window (`CL-CODE-012`, P1) |
| **An unused code expires** | Silence, not reuse (§6.2) | — |
| **Attempts per code** | 🧪 Low single digits | Five attempts against a six-digit code is a 1-in-200,000 per-code hit probability |
| **Attempts per classroom per window** | 🟡 Bounded across *codes*, not per code | ⛔ **Counting per code alone is defeated by requesting a new code after every attempt** (§6.2) |
| **Also bounded per session and school-wide per source** | — | ⛔ Attacker distributes attempts across rooms so no single counter ever trips; one host spraying every classroom must trip something |
| **Lockout behaviour** | Lock the **authorisation path**, not the display | ⛔ Locking the display locks the teacher out of the lesson. Locking a specific pending action contains the attack and preserves the lesson |
| **Lockout notification, and logged admin override** | Notify an administrator | Otherwise an attack is invisible, and support becomes the bypass — see §10 |

⚠️ NIST recommends OTP validity of ten minutes or less; ✅ VERIFIED — §Sources 2.
⚠️ **The classroom code is an order of magnitude shorter than NIST's ceiling, and
deliberately so** — the conditions under which it is displayed (on a wall, in a
room of children) demand it. ⚠️ **`CL-CODE-010`** asks how many classrooms could
hold an active code at once; 🧪 that estimate sizes the rate limiter — and is a
useful sanity check, because if the answer is "all of them, all day", the limiter
needs a per-day dimension too.

### 6.2 ⚠️ Resend — the standard bypass of an attempt limit

> **If issuing a new code is not itself rate-limited, a 30-second code lifetime is an unlimited guessing window.**

The failure is exact: the attempt counter is attached to the **code**. Attacker
makes 5 attempts → requests a fresh code → counter resets → repeat. Over an hour,
every digit-position combination is tested at speed.

⚠️ **The correct control is a counter on the *authorisation request*, not on the
code.** "This classroom has requested N codes in the last T minutes" bounds the
attack regardless of code management. `CL-CODE-009` (P1) asks whether issuance is
itself rate-limited — **the answer must be yes.** 🧪 **On resend semantics:**
generating a **fresh** code beats resending the same one, because it guarantees
any code photographed from the previous screen is dead. Resending the same value
into a longer window actively increases exposure.

### 6.3 ⚠️ Replay — the photographed code

⚠️ `CL-CODE-011` (P0) asks what stops a student photographing a displayed code,
**and whether the school treats that as a realistic threat.** ⚠️ The second half
matters more: a threat model the school has not accepted is one whose control
will be quietly dropped to save time.

| Threat | Realistic because | Control |
|---|---|---|
| **Photograph of a displayed code** | On a wall at reading distance, in a room of children, with phones | Short lifetime; single use; code never lingers (`CL-CODE-016`) |
| **Entry later, another room** | A code bound to one classroom is worthless elsewhere | Binding (§6.1) |
| **Entry later, same room, same action** | ⛔ Single use does not defeat this if the code is unused | Absolute expiry — the code dies whether used or not |
| **Entry after the legitimate action** | ⛔ A captured code used a second time | Invalidation on success (§6.1) |
| **Replaying the *request* rather than the code** | A captured authenticated request resubmitted | ⛔ **Idempotency key** — `CL-SEC-016` (P0). A replayed request must not create a second physical delivery |

⚠️ **Idempotency is not an authentication control, and this document is careful
not to conflate them.** A replay of a *captured request* is prevented by
idempotency; a replay of a *captured code* is prevented by lifetime, binding and
single use. Different problems, different controls — and conflating them is how
one goes unaddressed.

### 6.4 Cancellation of an unused code

⚠️ `CL-CODE-019` (P2) — and the questionnaire is right that this is the **most
likely day-to-day complaint in a pilot**, which is why it is settled in advance.
🧪 On every one of these events — the teacher leaving the room before using the
code, the lesson ending, a substitution occurring, the teacher changing their mind
— the code **expires server-side**. A "cancel" that only clears the screen leaves
a live code behind. ⛔ **A code issued to one teacher must never become usable by
another**, including a substitute arriving mid-lesson: that is a shared-account
failure wearing a code's clothing. Cancellation should be one tap, because friction
here produces workaround behaviour.

---

## 7. Where a code may never appear

⛔ **`CL-CODE-013` (P0) asks where a code may appear at all.** The principle:
**a code is a secret with a thirty-second lifetime, and every location that
persists it converts a short-lived secret into a long-lived one.**

| ⛔ Never | Why |
|---|---|
| **On the classroom screen after the teacher looks away** | Readable by every child in the room within seconds. `CL-CODE-016` (P0) — physical exposure, not logical exposure |
| **In a URL** | ⛔ URLs leak into browser history, `Referer` headers, proxy logs, server access logs, error reports and screenshots — often to parties with no app access. A one-time code in a URL is not a one-time code |
| **In browser `localStorage`** | ⛔ Readable by any script on that origin, persists indefinitely, is not cleared when the tab closes, and **survives on a shared kiosk until someone removes it**. `CL-SEC-005`, `CL-REACT-016` (P0) |
| **In source control** | ⛔ Permanent, readable by anyone with a repository copy, effectively impossible to purge. `CL-GIT-014` (P0) |
| **In a config file** | ⛔ Config files get pasted into support tickets. `CL-GIT-015` |
| **In logs, crash reports or support transcripts** | ⛔ See below |

⚠️ **Logs are the leak people forget, because debug bundles outlive the code that
produced them.** The lifetime mismatch is the whole argument: the **code** lives
30–60 seconds, single use. The **application log** lives days to months and ships
in a bundle. The **crash report** is uploaded and retained indefinitely. The
**support transcript** is forwarded between staff and vendor over months. The
**screenshot in a ticket** lives forever. A code logged at 09:14 with a 60-second
lifetime is **still a readable credential in a support bundle in March**,
because the reader in March has no way to know when it was issued or how long it
lived. `CL-CODE-014` (P1) and `CL-SEC-006` both exist for this. 🧪 **Practical
consequence:** logging must be structured so a code *cannot* be logged even by
accident — not "we remember not to". **No code, at any nesting depth, in any
field, in any log sink.**

---

## 8. Unattended smartboards — the hardest identity problem

⚠️ **Everything so far has assumed a human at a keyboard.** A classroom
smartboard is not that, and this is where the model meets reality: it is
**shared** (six periods a day, six sets of hands), **frequently unattended**
(between lessons, assembly, free periods), **reachable by children**
(⛔ `CL-UX-008` — the youngest student touching displays is ❓ **UNKNOWN**), and
**left switched on** (nobody closes it or looks at it between periods).
⚠️ `CL-CODE-017` asks separately whether any display has presence detection.

### 8.1 ⚠️ May a display hold a signed-in session while the room is empty?

> **🧪 Recommendation: NO. With a short idle timeout.**
>
> A persistent teacher session on an open panel means **anyone in the room
> inherits that teacher's authority** — including delegated permissions and the
> ability to act as the responsible adult for that room.

⚠️ **The reasoning is worth stating plainly, because "always signed in" is the most
commercially attractive property this product could have.** It removes friction
entirely. It is also the single property that converts a wall display into an
unattended point of staff authority in a room full of children.

⚠️ **`CL-AUTH-020` (P0)** asks exactly this and requires a **written school
decision**. ⚠️ It is the question most likely to be answered "yes, for
convenience" — and the answer must be recorded with the consequence stated,
because the person who inherits a session did not choose to and cannot see that
they did.

| Option | Friction | Security | Assessment |
|---|---|---|---|
| **Always signed in** | None | ⛔ **Any student, any time** | ⛔ Not recommended |
| **Idle timeout of a few minutes** | Re-authenticate per lesson | 🟡 Bounded exposure | 🟡 **Recommended default** |
| **Timeout at period boundary regardless** | Predictable rhythm | 🟢 Bounded by the school day | 🟡 Good; matches `CL-SEC-002` |
| **Re-authenticate per sensitive action** | Per action | 🟢 Highest assurance | 🟡 Best for high-consequence actions; possibly too slow for routine ones |

⚠️ **The right answer is probably a combination**: a session ending at the period
boundary, plus a one-action code for the occasional mid-lesson action. That is
exactly the §5 shape, and it is why §5 is designed that way.

### 8.2 Preventing student operation of an unattended board

⚠️ `CL-AUTH-021` (P0) and `CL-SEC-001` (P0) both ask how a student is prevented
from operating an unattended board. The **unauthenticated read-only home state**
whenever nobody is signed in 🟢 removes the attack surface when the room is idle at
🟢 low cost, and is 🟢 **the baseline this project assumes**
(`./classroom_dashboard.md` §1). An **idle timeout** is 🟢 **required**, returning
the display to that read-only state unattended, at 🟡 the cost that teachers
re-authenticate. **Kiosk lockdown** would 🟢 remove the browser, OS and desktop
from reach entirely and is 🟡 the strongest single control — but it needs 🟡 school
IT permission that may not be available. A **physical control** (a key, a
staff-only button) 🟢 ensures nothing a child can touch reaches the app, at 🟡 the
cost of hardware plus someone remembering to use it — 🧪 often the honest answer in
a real school. **Presence detection** ending authorisation when the room empties is
🟢 effective and ⛔ unlikely to exist (❓ `CL-CODE-017`).

⚠️ **The non-negotiable part:** whatever the school decides, an **unauthenticated
state must exist and must be the default.** A system with no signed-out state is a
system that is always signed in as someone.

### 8.3 Device trust and session binding

⚠️ **Are classroom devices enrolled and managed by school IT?** ❓ **UNKNOWN.** If
unmanaged, remote lock and device identity are unavailable and §8's controls
collapse to the timeout alone (`CL-AUTH-018`, P1). ⚠️ **May a session be bound to
one enrolled device?** 🧪 **Recommended where devices are enrolled** — a session
that cannot be carried to another display is worth much less to anyone who copies
it from one (`CL-AUTH-019`, P2). ⛔ **Should a session follow the teacher to any
display?** This is a **split decision**: convenience says yes, but `CL-SEC-004`
says a portable stolen session is usable school-wide. 🧪 **Device-bound for
displays, portable for personal devices** (`CL-AUTH-017`, P2).

⚠️ **The genuine tension, stated rather than resolved:** a display that always
requires re-authentication is one teachers will ask to bypass, and a bypass
request from staff is the most dangerous input this project will receive. The
correct response is to make the *consequential* path protected and the
*incidental* path frictionless — not to weaken the former.

---

## 9. Sessions

⚠️ **A session identifier is, for its lifetime, equivalent to the strongest
authentication method used to create it.** It is a bearer token: whoever holds it
is the teacher. **Its lifetime is a security control, not a configuration
preference.**

| Requirement | Reason | Status |
|---|---|---|
| **`httpOnly` cookie** | ⛔ Not readable by JavaScript. A token in `localStorage` is readable by **any script on the origin** — including a compromised dependency and any injected script a malicious indexed document carries (`CL-SEC-008`, `CL-SEC-009`) | ⛔ Required |
| **`Secure` flag** | Not sent over plaintext HTTP. ⛔ Plain HTTP login on a school LAN transmits the password to any device that can see it | ⛔ Required |
| **`SameSite=Strict`** | Not attached to cross-site requests. ⛔ `Lax` still permits a top-level cross-site GET, sufficient to leak a token in some flows | 🟡 Required; `Lax` only with a stated reason |
| ⛔ **NEVER `localStorage` / `sessionStorage`** | Persistent, script-readable, survives on shared hardware until manually removed. `CL-SEC-005`, `CL-REACT-016` | ⛔ Prohibited |

🟡 **OWASP's Session Management Cheat Sheet** sets out each of these and the
"no session identifier in the URL" rule; ✅ VERIFIED — §Sources 4.

⚠️ **`CL-AUTH-016` and `CL-SEC-002` ask about timeouts and ❓ neither is answered.**
Three distinct controls are needed, and conflating them is a common error: an
**idle timeout** bounds a session with no interaction (short — a display is used in
bursts, so a long idle window buys nothing and exposes a lot); an **absolute
timeout** bounds total lifetime regardless of activity; and a **renewal timeout**
bounds a continuously used session. ⚠️ **The renewal timeout is the one people
omit,** because an absolute timeout measured from sign-in is defeated by a display
switched off and on each morning — each power cycle is a new session. A renewal
timeout measured from *last re-authentication of the person* is what actually
bounds exposure.

⚠️ **Server-side invalidation requirements.** Logout must invalidate **server-side,
immediately** — ⛔ a client-side logout that only clears a cookie leaves a valid
session on the server. A **privilege change must invalidate existing sessions** —
⛔ a teacher whose permissions were reduced must not continue under the old grant
until their session expires. The **session identifier must be rotated on
authentication**, preventing session fixation: an identifier an attacker planted
before sign-in is replaced by one they do not know. Revocation must be
**immediate**, not "at next login" (`CL-AUTH-022`, `CL-SEC-004`), and must take
effect **from another room** — ⚠️ a session captured from one smartboard is
otherwise usable across the school for its entire natural lifetime.

⚠️ **This is the concrete content of §3's warning:** offboarding that marks an
account "disabled" but leaves its sessions alive has offboarded the login, not the
access.

---

## 10. Recovery

> ### ⚠️ **The recovery path is a universal bypass if it is weak.**
>
> Every control in this document can be set aside by an attacker who reaches the
> recovery flow. **A weak recovery path is not a weaker system; it is an absent
> one.**

⚠️ `CL-AUTH-014` (P1) asks what recovery is and who verifies the requester. The
honest answer in most deployments is "an administrator resets it", and the honest
problem is that a single human's judgement is then the entire remaining boundary.

| Requirement | Rationale |
|---|---|
| **Recovery is at least as strong as the factor it replaces** | ⛔ Recovery weaker than the lost factor means the lost factor was optional |
| **Recovery codes are single-use secrets, stored as secrets** | Not emailed, not shown once and forgotten, not stored beside the password they replace |
| **Rate limits on recovery attempts, per account and per requester** | ⛔ An unthrottled recovery flow is a password-reset oracle |
| **Session rotation after recovery** | ⛔ If an attacker drove the recovery, any session they hold must die |
| **A deliberate, logged, policy-documented path** | ⛔ **Not a silent fallback to something weaker.** "We couldn't verify, so we emailed a link" is the exact failure |
| **Administrator resets are audited acts by a named human** | `CL-AUTH-023`, `CL-SEC-019` |
| **Who may reset during school hours is a policy decision** | `CL-AUTH-015` (P1). ⚠️ If only the Principal may reset, a forgotten password halts a lesson — and the predictable response is to stop asking |

⚠️ **The adoption trap, stated once more because it recurs at every control here:**
a recovery path slow enough to be annoying gets bypassed, and a bypassed recovery
path is not a recovery path. The goal is a flow that is **fast for a legitimate
teacher in a corridor and slow for everyone else** — different requirements, which
is why this is the school's decision rather than ours.

---

## 11. Authorisation model

⚠️ **Role-based access control** attaches permissions to roles, not individuals —
🧪 individual exceptions are where permission models go wrong. ⚠️ **Least
privilege** means each role holds what its job needs and nothing else;
`CL-ADMIN-002` (P0) asks which of user accounts, timetable publication,
substitution changes, resource permissions, software deployment and audit access
must be **separately** permissioned. ⛔ **Separation of duties** matters because a
bundled administrator compromises everything at once, making it the most
attractive single target in the system. ⚠️ **Department-scoped administration** —
timetable, library and facilities holding their own limited rights — reduces a
single point of failure (`CL-ADMIN-020`, P3). ⛔ **Break-glass with review**,
because break-glass without review conditions becomes the ordinary way privileged
work is done (`CL-ADMIN-010`, P0). ⚠️ **Read-only oversight**, because someone
must be able to *see* administration without changing it (`CL-ADMIN-017`, P3).

### 11.1 ⚠️ Separation of duties, sized to what the school can staff

⚠️ `CL-ADMIN-003` (P0) asks whether one person must request a substitution and
another publish it. ⚠️ **The textbook answer is four-way separation — request,
approve, publish, audit — and this school cannot staff it.** `CL-ADMIN-008` (P1)
records that this school has very few technical staff. An ideal separation
requiring four people exists on paper and not in practice, which is worse than one
designed honestly, because it produces false assurance.

| Model | Requires | Assessment |
|---|---|---|
| **Four-way separation** | Four distinct people | ⛔ **Not staffable.** Recorded here only to be rejected |
| **Two-person separation** (request ≠ publish) | Two people in the timetable office | 🟡 The clearest cheap control. `CL-ADMIN-003` |
| **Single person + audit** | One person, plus a log somebody else reads | 🧪 **Realistic minimum.** Weaker, and should be stated as weaker |
| **Single person, no review** | One person | ⛔ **Not acceptable.** `CL-ADMIN-011` asks whether administrator activity is reviewable by someone else; if not, there is no accountability |

⚠️ **Whatever the school chooses must be written down, with the weaker option
named as weaker.** A separation-of-duties claim of "we logged it" is a trade-off.
That is acceptable in a school with two technical staff; it is not acceptable while
also being described as four-way separation.

### 11.2 Break-glass

⚠️ `CL-ADMIN-010` (P0) asks whether a break-glass account is acceptable here.

⚠️ **It exists, because the alternative is worse** — ⛔ "no break-glass" in practice
means improvisation at 08:30 when the timetable is wrong. **Its access is as narrow
as possible**, because emergency does not require standing administrator rights.
**Every use requires a stated reason before, not after**; **every use is alerted
immediately to somebody who can stop it**, because alerting afterwards is not
control; and ⛔ **every use is reviewed afterwards by a named person** — without
review, emergency access becomes normal access with extra paperwork, which is the
documented failure mode.

---

## 12. Transport and storage

### 12.1 ⚠️ TLS on an internal LAN

> **A school LAN is not trusted merely because it is internal.**
> `./network_and_hosting.md` §6.1 already establishes this: student devices of
> unknown provenance, guest and vendor devices, shared credential practice.

| Property | Requirement |
|---|---|
| **TLS, display → backend** | ⛔ Required. A session cookie sent in plaintext is readable by every device on the segment |
| **TLS, backend → NAS** | ⛔ Required for anything carrying credentials or resource content |
| **TLS, internal DNS** | ⚠️ Consider; depends on the resolver |
| **Plain HTTP anywhere** | ⛔ Not for a login surface. `CL-AUTH-010`, `CL-SEC-024` |

⚠️ **What a school LAN actually contains:** student laptops and phones, a guest
network, contractor laptops, camera systems, attendance terminals, and devices
nobody has inventoried. The correct posture is that **any device on the segment
could be observing traffic**, and that "but it's only internal" is not a control.

### 12.2 Self-signed certificates

❓ **Is a self-signed certificate acceptable?** `CL-AUTH-010`, `CL-SEC-024` — school
policy, not our decision. ⚠️ **Trust distribution** is the real problem: the
certificate must be trusted on **every display**, or every board shows a warning a
teacher will click through — and clicking through a warning makes it worthless.
⚠️ **Expiry** (`CL-OPS-018`, P0) is the sharpest failure mode: **an expired
certificate makes every board in the school fail simultaneously**, with a cause not
obvious to a teacher. 🟢 **An internal CA**, if the school has one, makes renewal a
local operation and removes the sharpest offline failure mode in
`./network_and_hosting.md` §7.3 entirely.

### 12.3 Secrets at rest and at runtime

| Requirement | Rationale | ID |
|---|---|---|
| ⛔ **Never in Git** | A credential in git history is permanent and effectively unrecallable | `CL-GIT-014` (P0) |
| ⛔ **Never in a committed, shared or ticket-pasted config file** | Config files travel | `CL-GIT-015` (P1) |
| **Secrets live in an environment-provided store at runtime** | Separates secret from code; makes one build portable between schools | `CL-GIT-019` (P2) |
| **Encrypted at rest** | Backups inherit the sensitivity of their contents | `CL-SEC-025` |
| **Service-account credentials distinct from human credentials** | ⛔ A shared service credential under a human's name is unattributable in exactly the way §2.1 rejects | — |
| **Rotation possible without a code change** | ⛔ A secret embedded in code cannot be rotated without a deploy, so it is never rotated | — |
| **Someone audits for committed secrets** | `CL-GIT-014` asks who audits. ⛔ "Nobody" is the common answer | `CL-SEC-006` |

---

## 13. The delivery-request flow, end to end

⚠️ **This is the only flow in CloudLight that ends in a physical action**, so it
is specified in full. It is the clearest possible demonstration of §1.

```
 (1) TEACHER AUTHENTICATES INDIVIDUALLY
     Named identity established.        ← the identity of record
              │
 (2) TEACHER INITIATES DELIVERY REQUEST
     "Deliver X to Room Y".  Deterministic. No AI in this path.
              │
 (3) [IF REQUIRED] ONE-ACTION CODE
     Authorises THIS request only. Not a second teacher. Not a session.
              │
 (4) DETERMINISTIC BACKEND CHECKS
     permission · destination validity · time window · SHANA reachable
              │
 (5) PERSIST WITH IDEMPOTENCY KEY
     A replayed request returns the ORIGINAL result. Never a 2nd delivery.
              │
 (6) SUBMIT TO SHANA
     High-level request only. Never motor, path, speed or navigation.
              │
 (7) STATUS DISPLAYED
     Explicit unavailable state with an age when SHANA is offline.
```

⚠️ **Step 1 is the only step that produces an identity.** ⛔ If it is skipped,
nothing downstream has a person to attribute the action to.

⚠️ **Step 2 is never interpreted by a model** — `../architecture/architecture_decisions/`,
**ADR-001**: no AI model may reach the robot. **Step 4 is where that becomes
real:** a check that depends on a model is a check that can be wrong. A model may
help a teacher *write* a request; a model may never decide whether that request is
authorised. ⛔ **Step 5 is not optional:** without an idempotency key
(`CL-SEC-016`, P0), a retried or replayed request produces **a second physical
delivery** — a real load, a real route, indistinguishable from the original.

⚠️ **Step 7 must distinguish "robot offline" from "CloudLight broken."** ⛔ A stale
robot status presented as current sends staff to a room expecting a delivery that
will never arrive (`CL-SEC-017`).

> **The authenticated teacher — not the code — is the identity of record.**
> If the code is present, it authorised one action. If it is absent, the teacher
> was still named. **There is no version of this flow in which the only thing we
> know is "someone in Room 4 knew six digits".**

---

## 14. Audit

⚠️ `CL-SEC-019` (P0): *"an administrator-editable log provides accountability to
nobody, including the school."* `CL-AUTH-023` (P1) and `CL-SEC-020` (P2) exist for
the same requirement.

| Event | Why | ID |
|---|---|---|
| **Sign-in — success and failure** | The failure rate is the early warning of a guessing attack | `CL-AUTH-023` |
| **Sign-out** | Paired with sign-in, it bounds session lifetime retrospectively | `CL-AUTH-023` |
| **Lockout after repeated failures** | ⛔ Otherwise an attack is invisible | `CL-SEC-003` |
| **Code issuance** — classroom, time, requesting identity | Establishes the window in which any code could have been used | `CL-CODE-020` |
| **Code use** — classroom, time, outcome, **which teacher was signed in** | ⛔ A code-authorised action with no record of the person present leaves no accountable trace at all | `CL-CODE-020` (P1) |
| **Permission change** | The highest-consequence administrative act | `CL-ADMIN-002` |
| **Account lifecycle** — created, disabled, re-enabled | Offboarding evidence | `CL-ADMIN-006` |
| **Delivery request** — who, what, where, when, outcome | `CL-SEC-020` names delivery request explicitly | `CL-SEC-016` |
| **Break-glass use and its review** | ⛔ Emergency access without a trail is indistinguishable from ordinary access | `CL-ADMIN-010` |
| **Recovery and password reset** | ⛔ Recovery is the universal bypass; it must be the most visible thing in the log | `CL-AUTH-014` |
| **Append-only with tamper evidence** | ⛔ Otherwise whoever administers the system can erase their own actions, and the log cannot be used against them | `CL-SEC-019` (P0) |
| **A defined, bounded retention period** | ❓ `CL-AUTH-023`, `CL-SEC-025`, `CL-OPS-010` — not ours to set. ⛔ Indefinite retention of records containing staff names is itself a policy problem | — |
| **Log content bounded by the same principle** | ⚠️ `CL-OPS-011` — request paths and usernames can end up in logs | — |
| **Readable without deep technical skill** | ⚠️ `CL-ADMIN-018` — a log nobody has time to open is not audit. A periodic summary is often more usable than the raw log | — |

⚠️ **A note on who audits the auditor.** `CL-ADMIN-011` asks whether an
administrator's activity is reviewable by *someone else*. Append-only storage
protects the record; it does not ensure anyone reads it. **Both are required** —
and the second is a staffing question, which returns to §11.1.

---

## 15. What is not decided

⚠️ **Nothing here is ours to decide.** ⚠️ **All are recorded as `UNANSWERED` in
`../../questionnaires/Additional_CloudLight_Questionnaire.md` as of 2026-10-11.**
⚠️ Rows sharing an answerer and a blocking theme are grouped.

| ID | Question | Who can answer | What it blocks |
|---|---|---|---|
| `CL-AUTH-001` | Does a staff identity directory exist | Administration | Everything. Without it CloudLight owns identity and offboarding |
| `CL-AUTH-002`, `CL-AUTH-003` | Is AD, LDAP, Google Workspace or M365 in use; may CloudLight authenticate against it | IT / administration | SSO feasibility; ⛔ written permission cannot be assumed |
| `CL-AUTH-004` | Must every action be attributable to one named teacher | Administration | ⛔ **The foundational decision of this document** |
| `CL-AUTH-005` – `CL-AUTH-008`, `CL-AUTH-012` | Login sharing today; who provisions accounts; staff numbers and turnover; will teachers accept an extra step | Administration / teachers | Adoption risk — a control staff bypass is not a control — and the provisioning workload |
| `CL-AUTH-009` | What password policy already applies | IT | CloudLight must not be weaker than the school's own systems |
| `CL-AUTH-010`, `CL-SEC-024` | Is TLS required internally; is a self-signed certificate acceptable; can the school issue an internal one | IT | ⛔ Whether the display may talk plain HTTP at all |
| `CL-AUTH-011`, `CL-CODE-018` | Is any second factor in use; if a code is a second factor, what is the first | IT | ⛔ **Whether a code can ever be a second factor** |
| `CL-AUTH-013` | Do school devices support passkeys or biometric unlock | IT | Whether anything is ever typed on a student-reachable screen |
| `CL-AUTH-014`, `CL-AUTH-015` | What is the recovery path and who verifies the requester; may anyone reset during school hours | Administration / IT | ⛔ A weak recovery path is a universal bypass |
| `CL-AUTH-016`, `CL-SEC-002` | What idle and maximum session length does policy require | IT / teachers | §9. ⛔ P0 for `CL-SEC-002` |
| `CL-AUTH-017` – `CL-AUTH-019` | Should a session follow a teacher to any display; are devices enrolled; may a session be bound to one | Administration / IT | ⚠️ Portability vs device binding, and whether device trust exists at all |
| `CL-AUTH-020` | May an unattended display stay signed in | Administration | ⛔ **P0. The largest identity risk in the display design** |
| `CL-AUTH-021`, `CL-SEC-001` | How is a student prevented from operating an unattended board; may any student use a smartboard at all | Administration / teachers | ⛔ P0. The read-only baseline |
| `CL-AUTH-022`, `CL-SEC-004`, `CL-SEC-005` | Can access and a copied session be revoked immediately; which credential stores are acceptable | IT | ⛔ Delayed revocation and surviving credential stores undermine the account model |
| `CL-AUTH-023`, `CL-SEC-019`, `CL-SEC-020` | Which events are recorded, for how long, and must the log be tamper-evident | Administration / IT | ⛔ P0. The log schema is expensive to change after launch |
| `CL-AUTH-024` | Which staff roles need access beyond ordinary teaching | Administration | Least privilege cannot be defined by the vendor |
| `CL-CODE-001`, `CL-CODE-002` | Is a named teacher required for every action; confirm a code is never authentication | Administration | ⛔ **P0. The finding this document exists to state** |
| `CL-CODE-003`, `CL-CODE-004` | Which actions must always require a named teacher; which, if any, suit a code alone | Administration | The never-list in §5.1; keeps code use from creeping outward |
| `CL-CODE-005` – `CL-CODE-007`, `CL-CODE-019` | Code lifetime, reuse, length and character set, and early cancellation | Administration / IT / timetable office | ⛔ Long display windows turn a one-action code into a credential; the most likely pilot complaint |
| `CL-CODE-008`, `CL-SEC-003` | Failed attempts before lockout, lockout duration, and notification | Administration / IT | ⛔ P0. Without it the code is guessable in a free period |
| `CL-CODE-009`, `CL-CODE-010` | Is resend rate-limited; how many active codes at once on a teaching day | IT / timetable office | ⛔ Unthrottled resend = an unbounded guessing window |
| `CL-CODE-011`, `CL-CODE-012` | What stops a photographed code; does the display return to signed-out after use | Administration | ⚠️ The threat model must come from the school; a captured code must have no later value |
| `CL-CODE-013`, `CL-CODE-014`, `CL-SEC-006` | Where may a code appear; must codes be excluded from logs, crash reports and transcripts | IT / implementation | ⛔ Debug bundles outlive the code |
| `CL-CODE-016` | Must the code disappear when the teacher looks away | Administration | ⛔ P0. Physical exposure, not logical exposure |
| `CL-CODE-020` | Must code use be logged with classroom, time and outcome | Administration | A code-authorised action with no record has no trace |
| `CL-ADMIN-002` | Which administrative rights must be separately permissioned | Administration | ⛔ P0. Bundled rights make every compromise total |
| `CL-ADMIN-003` | Must one person request a substitution and another publish | Timetable office | The clearest cheap control, and the one most likely to be waived silently |
| `CL-ADMIN-005`, `CL-ADMIN-006`, `CL-ADMIN-016` | How accounts are created, how quickly access ends, and what expires a guest account | Administration | ⛔ P0. Manual provisioning drifts; offboarding is usually the failed control |
| `CL-ADMIN-008` | Who is realistically available at short notice | Administration | ⚠️ **The constraint that sizes the entire authorisation model** |
| `CL-ADMIN-010` | Is a break-glass account acceptable, and who reviews it | Administration | ⛔ P0. Uncontrolled emergency access is the route around least privilege |
| `CL-GIT-014`, `CL-GIT-015` | What is our secrets policy; where do NAS or server credentials live at runtime | IT | ⛔ P0. Even a private repository leaks eventually |
| `CL-OPS-018` | Which certificates or tokens does the school operate, and who tracks expiry | IT | ⛔ P0. An expired certificate takes every board down at once |

---

## The invariants

Five rules survive every open question above. If the answers arrive and they are
inconvenient, these five do not move.

> **1. A six-digit code is never identity.** It authorises one action, or it is a
> second factor. It never stands in for a named teacher, and no interface may
> present it as though it does.

> **2. Individual identity for anything consequential.** Shared accounts destroy
> attribution, revocation and accountability simultaneously, and none of the three
> is recoverable afterwards.

> **3. The teacher is the identity of record.** The delivery flow exists to make
> this true end to end: authenticate individually, then authorise one action.

> **4. A code is short, single-use, attempt-limited and never persisted.** Each of
> those four properties is load-bearing. Remove one and the million-entry keyspace
> becomes the vulnerability.

> **5. Recovery is as strong as what it replaces.** A recovery path weaker, faster
> or more available than the factor it replaces is not a recovery path. It is an
> off switch.

---

## Sources

**Types:** `GOVERNMENT-STANDARD` · `OFFICIAL-OPEN` (maintained open-source
project) · `INTERNAL` (this repository). **Access date for all external sources:
2026-10-11.**

| # | Title | URL | Type | What it supports |
|---|---|---|---|---|
| 1 | **Authentication Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html` | OFFICIAL-OPEN | §4.2, §7 — MFA as the primary defence against credential compromise; single-use codes; short OTP lifetime; rate limiting; OTP secrets not stored in plaintext; **cryptographically secure** random-number generation for tokens |
| 2 | **Digital Identity Guidelines: Authentication and Lifecycle Management (NIST SP 800-63B)** | `https://pages.nist.gov/800-63-3/sp800-63b.html` | GOVERNMENT-STANDARD | §1.3 — failed attempts limited to **no more than 100 consecutive attempts** on a single account; §4 — **SMS/PSTN designated a "restricted" authenticator** (SS7, SIM-swap, number-porting risk); §4.2 — **OTP validity of ten minutes or less**, single-use, invalidated on use, **single-factor authenticators of at least 8 digits**; AAL2 requiring two distinct factors |
| 3 | **Multifactor Authentication Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html` | OFFICIAL-OPEN | §4 — factor strength ranking; TOTP, push and SMS characteristics; **number matching against push-bombing / MFA-fatigue attacks** |
| 4 | **Session Management Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html` | OFFICIAL-OPEN | §9 — `httpOnly`, `Secure`, `SameSite`; ⛔ no session identifier in the URL; ⛔ none in `localStorage`; session ID rotation on authentication; server-side invalidation on logout and privilege change; idle and absolute timeout as distinct controls |
| 5 | **Web Authentication API: An Introduction — W3C TAG** | `https://www.w3.org/2021/03/bringing-passwordless-login-with-webauthn.html` | GOVERNMENT-STANDARD | §4 — WebAuthn passkeys are **origin-bound, phishing-resistant, and the private key never leaves the authenticator** |
| 6 | **Access Control Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html` | OFFICIAL-OPEN | §11 — least privilege, separation of duties, deny by default, and **access control decisions made server-side on every request**, not once at sign-in |
| 7 | **`../architecture/architecture.md`** and **`../architecture/architecture_decisions/`** | *(this repository)* | INTERNAL | §13 — the local application server; the AI/deterministic boundary; **ADR-001: no AI model may reach the robot** |
| 8 | **`./network_and_hosting.md`** | *(this repository)* | INTERNAL | §12 — a school LAN is not trusted merely because it is internal; TLS even on the LAN; certificate expiry as a simultaneous school-wide failure; offline-capability vs patchability |
| 9 | **`./classroom_dashboard.md`** | *(this repository)* | INTERNAL | §5, §8 — the display is **read-only and unauthenticated by design**; `CL-DASH-001` log in at the display; accidental child touch during a lesson; `CL-UX-008` age of youngest student touching displays |
| 10 | **`./shana_integration.md`** | *(this repository)* | INTERNAL | §13 — SHANA-side ownership boundary; high-level requests only; **safety never transfers to CloudLight** |
| 11 | **`../../questionnaires/Additional_CloudLight_Questionnaire.md`** | *(this repository)* | INTERNAL | Every `CL-AUTH-*`, `CL-CODE-*`, `CL-SEC-*`, `CL-ADMIN-*`, `CL-GIT-*` and `CL-OPS-*` identifier in §15, with priority, respondent, evidence requirement and `UNANSWERED` status |

---

⚠️ **Standing note.** Every external source above is a **general published
standard or project document**. **None of it describes this school.** No identity
provider was contacted, no directory enumerated, no credential created or
attempted, no smartboard signed in to. The questions in §15 remain questions until
school IT and school administration answer them in writing.

**Tests actually executed to date: zero.**