# CloudLight — Glossary

**Phase 0 · 2026-10-11**

Terms used across the CloudLight documentation, in plain English. Where a term has
a common misunderstanding, that is stated.

⚠️ **Where a term appears in both CloudLight and SHANA, the meaning is
deliberately identical.** The SHANA glossary at
[`../../shana/overview/glossary.md`](../../shana/overview/glossary.md) governs
shared terms — in particular `UNKNOWN`, go/no-go gate, assumption, reversal
condition and the delivery vocabulary. This document does not restate SHANA's
robot and navigation terms; it extends the shared vocabulary to CloudLight's
concerns.

---

## A

**ACL** — *Access Control List.* The list of principals a file or folder grants
or denies access to, held by the storage system itself. ⚠️ **CloudLight holds no
parallel permission model** — it defers to the share's own ACLs, because two
permission models in one product means the weaker one wins at the worst moment.

**AI** — Artificial intelligence. In CloudLight's vocabulary, ⚠️ **any model output
is a proposal and never a decision.** A model may draft; deterministic code
validates; a named human approves.

**Approval gate** — The single point in the pipeline at which a named person with
authority converts a *proposal* into something that may be shown. ⚠️ Exactly one
gate exists and it is human. No automated path reaches a display, regardless of
input quality or model confidence.

**Assigned Access** — The Windows single-app kiosk mechanism. ⚠️ **Edition-dependent
and unverified on this school's hardware** — `CL-WIN-001`. The other kiosk
mechanism is Shell Launcher; the two cannot be combined on one machine.

**Audit log** — An append-only record of who did what, when, to which object, and
with what outcome. ⚠️ **An audit log that omits the actor is not an audit log.**
Shared accounts make it unfillable, which is why individual accounts are required
for anything consequential.

**Authorship vs authority** — ⚠️ A record is authoritative because *a person with
authority approved it*, not because it was photographed, parsed, or extracted at
high confidence. Visibility is not authority.

---

## C

**CloudLight** — The school-facing digital platform proposed for this project: a
local-first, school-hosted system that shows classroom information on smartboards
and manages timetable and substitution data. ⚠️ **CloudLight is the platform. SHANA
is one robotics subsystem inside it.** Neither gets a separate repository. In this
repository CloudLight **exists only as documentation** — there is no code.

**Cloud VM** — A virtual machine hosted off school premises. ⚠️ **A scope change,
not a fallback.** It breaks the local-first premise, reintroduces internet
dependency, and moves school data off school infrastructure.

---

## D

**DENY over ALLOW** — The rule that an explicit denial always beats an explicit
grant when both apply to the same file. Windows ACL evaluation works this way, and
CloudLight must honour it exactly rather than approximating it — because
approximating it means a file you may not open appears in your results.

**Deterministic parsing** — Reading a document with a program that yields the
same answer every time, on every machine, offline and free. ⚠️ **It precedes AI,
without exception.** If a source has a stable cell structure, a model is never
involved in reading it — a model cannot exceed the cell addresses it was handed,
is slower, non-deterministic, costs money, and cannot be reasoned about.

---

## E

**Evidence class** — The label attached to a claim showing how well it is
established. The seven classes used across CloudLight are listed below under
*Verification classification*. ⚠️ **A document labelled `UNKNOWN` is not a weak
`VERIFIED`.** It is an open question and must never be cited as a finding.

**Exhibition** — A demonstration to an audience. ⚠️ **Not a pilot, and not
acceptance.** A demonstration with mock data is a separate, labelled activity
(`CL-EXPO-002`). Conflating the two is how a mock-up gets accepted as a working
system.

---

## F

**Fail-closed** — When something fails, the system becomes *less* capable, never
more. ⚠️ The critical case is an outage: a system that degrades to "open" when the
server is unreachable has silently converted an authenticated system into an
unauthenticated one. A display must show cached PUBLIC-class data with a visible
staleness marker — never blank, never frozen, never the last privileged screen.

**First release** — The smallest version that is useful on its own, depends on no
unanswered blocking question, and establishes the correct-data path everything
else attaches to. See [`../first_release_scope.md`](../first_release_scope.md).
⚠️ **Release 0 is a spine, not a subset of the product.**

---

## I

**Idempotency** — Sending the same request twice has the same effect as sending it
once. ⚠️ This matters physically, not just technically: a duplicated delivery is a
real event with a real object. Every delivery request carries a unique
idempotency key, and a retry does not create a second delivery.

---

## L

**Local inference** — Running a model on school hardware, so nothing leaves the
network. ⚠️ The only AI path that survives a school refusing external services.
Hard prerequisite that is easy to miss: ⚠️ **a permanently offline school cannot
download model weights.** Every binary, model and tokeniser must be fetched
elsewhere and transferred across the air gap in advance.

---

## M

**MFA** — *Multi-factor authentication.* Requiring more than one independent thing
to prove identity. ⚠️ SMS and PSTN are **restricted** authenticators under NIST
SP 800-63B — known interception and SIM-swap exposure — and are not used.

**Modular monolith** — One deployable process with enforced internal module
boundaries. ⚠️ **Not the same as a monolith.** The boundaries exist so a later
split stays possible and so modules cannot quietly reach into each other's data;
they are enforced in code and by test. Chosen because microservices solve an
organisational scaling problem this project — one school, a few dozen displays,
one host, one IT generalist — does not have. Recorded in ADR-013.

---

## N

**NAS** — *Network Attached Storage.* A file server on the network holding shared
files. ⚠️ **Whether one exists in this school is `UNKNOWN` and is a P0 question**
(`CL-NAS-001`). A school may keep every teaching file on laptops, pen drives and
personal cloud drives, and *"no"* is a completely valid answer that closes the
resource track permanently.

**No-go** — A gate state meaning the activity is not started and must not start.
⚠️ Distinguished from *blocked*, which means a specific named blocker exists.

---

## O

**One-action authorisation** — A short-lived, single-use credential that
authorises **one specific action** and nothing else. ⚠️ It is **not**
authentication. A shared six-digit classroom code may only ever be an
one-action authorisation or a second factor behind a real first factor; it may
never stand in for a named teacher. 10⁶ possibilities with no attempt limit is
brute-forceable in a free period.

**OIDC / SAML** — The two standard protocols for single sign-on. Relevant because
they are the mechanism by which a school directory would authenticate CloudLight
users — ❓ if a directory exists, which is unknown.

---

## P

**Permission-filtered retrieval** — Returning only those results the
authenticated user is entitled to see, with the permission check applied **before**
ranking, not after. ⚠️ Ranking-then-filtering leaks existence and metadata through
ordering and counts. ⚠️ The check happens **again at open time**, because ACLs
change between indexing and opening and the metadata itself is the leak.

**Pilot** — A limited deployment with a real school, real teachers and real data,
conducted under written acceptance criteria agreed in advance. ⚠️ **A pilot is an
acceptance event, not a demonstration.** It is the only thing that can retire a
risk.

**Prompt injection** — An attempt to make a model follow instructions embedded in
content it retrieves, rather than the instructions it was given. ⚠️ **This is not
fully mitigable by filtering.** The structural defence is *segregation*:
retrieved content is treated as data and never as instruction, and the assistant
has no pathway to effect — it may only propose.

**Publication** — The act of making a record visible on a classroom display. ⚠️
**Always requires human approval.** The constraint is on *publication*, not on
*proposal* — machine drafts remain permitted, visibly marked as unconfirmed.

---

## Q

**Quarantine** — The destination for a record that failed validation or review.
⚠️ **Quarantine is not a bin.** A quarantined record stays retained, inspectable
and attributable — a person must be able to see which document it came from, which
rule rejected it, and what it said. Silently dropping it is indistinguishable from
losing a substitution.

---

## R

**RBAC** — *Role-Based Access Control.* Permissions attached to roles rather than
to people, so that granting and reviewing access is tractable for a school.
⚠️ Must be layered with **deny by default**: an explicit grant is required for
every permission, because that is the only model that fails safely when the role
model is wrong.

**Read-only service account** — A dedicated credential CloudLight uses to read
school storage. ⛔ **It must have its own credentials and never be a staff or
administrator credential** — otherwise every action CloudLight takes is attributed
to a person who did not perform it.

**Reversal condition** — The recorded circumstance under which a decision would be
revisited. ⚠️ **A decision without one becomes a decision nobody dares revisit**,
and should be treated as provisional whatever state is written beside it.

**Rollback** — Returning to a previous known-good version. ⚠️ **Rollback must be
*rehearsed*, not intended.** An untested rollback is an intention.

---

## S

**SMB** — The Windows file-sharing protocol used to reach a NAS. ⚠️ SMBv2/v3 are
assumed available; ⛔ SMBv1 is deprecated and unsupported, so a share reachable
only over SMBv1 is a **blocked dependency, not a workaround**.

**SHANA** — *School Autonomous Notification & Assist.* The robotics subsystem
inside CloudLight: navigation, localisation, route safety, battery, hazards, safe
stopping and delivery execution. ⚠️ **SHANA validates every safety decision
independently, and CloudLight never sends a motor command.** The boundary is
expressed as an **absence of capability**, not a policy — see ADR-014.

**Signed update** — A release artefact carrying a cryptographic signature that the
consuming device verifies **before accepting it**. ⚠️ Signature verification
belongs on the consuming device, not at the distribution point, because a school
LAN is not trusted. ⚠️ A naive "check the signature" design resists none of
rollback, freeze, fast-forward or mix-and-match attacks.

**Smartboard** — A classroom display with a touchscreen, typically Windows. ⚠️ **A
client, never a server.** Boards sleep, restart mid-lesson, lock, and are switched
off; re-imaging one would take down every classroom it serves.

**Staleness** — The age of the data on a display. ⚠️ **Every displayed value
carries its age.** A screen frozen since 08:00 looks exactly like a correct one,
so a visible last-synced timestamp is the difference between information and
decoration.

---

## T

**TUF** — *The Update Framework.* A published specification defining root,
snapshot, targets and timestamp roles with **separate keys** and required
expiry, so that a frozen or stale repository state becomes invalid on its own.
✅ VERIFIED as a specification. ⚠️ **It is presented in
[`../ota_updates.md`](../ota_updates.md) §5.1 as the reference design, not as a
decision to adopt it.** A simplified TUF-shaped design may be more maintainable
for one operator.

---

## V

**Verification classification** — ⚠️ **Seven classes**, used consistently across
CloudLight documentation. Getting the class right is how a reader knows whether a
sentence is a finding or a question.

| Class | Means |
|---|---|
| ✅ `VERIFIED` | Confirmed in official vendor, standards or regulatory documentation, quoted with a source |
| 🟡 `DOCUMENTED_NOT_HARDWARE_VERIFIED` | A vendor documents it; **not confirmed on this school's equipment** |
| 🧪 `PLAUSIBLE_REQUIRES_TESTING` | Engineering judgement. No vendor claim. Needs a test |
| ❓ `UNKNOWN` | No evidence either way. ⚠️ **Not assumed. To be answered by a named party** |
| ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` | We will not implement this without an explicit recorded decision |
| ⚠️ `REQUIRES_SCHOOL_PERMISSION` | Blocked until a named school authority approves in writing |
| ⚠️ `REQUIRES_VENDOR_OR_SERVICE_PROVIDER` | Blocked until a vendor or provider confirms |

⚠️ **Two rules that are not negotiable.** A `PLAUSIBLE` or `UNKNOWN` document is
an **open question, not a weak verified** — it must not be cited as a finding. And
`ANSWERED` ≠ `VERIFIED`: the first means a person replied, the second means
evidence exists.

---

## W

**WebAuthn / passkeys** — A phishing-resistant second factor. ✅ Preferred for MFA in
this project; TOTP is acceptable; ⛔ SMS is not.

---

## Words this project refuses to use loosely

| Word | What it must not be allowed to imply |
|---|---|
| **"Implemented"** | That code exists. ⚠️ It does not. Zero lines |
| **"Verified"** | That it was tested here. ⚠️ Only vendor documentation has been read |
| **"Pilot"** | That a school was involved. ⚠️ None was |
| **"Integration"** | That two systems talk. ⚠️ No integration exists with anything |
| **"Works"** | That it was run. ⚠️ It has never been run |
| **"Default"** | That it was chosen. ⚠️ Most defaults here are *conservative fallbacks* applying because nobody answered |

---

## Related documents

| Document | Covers |
|---|---|
| [`../overview.md`](../overview.md) | The narrative version of the same ground rules |
| [`current_project_state.md`](current_project_state.md) | What exists and what does not |
| [`../../shana/overview/glossary.md`](../../shana/overview/glossary.md) | The robot and navigation vocabulary — governs shared terms |
| [`../requirements.md`](../requirements.md) | The `C1`–`C12` constraints, in full |
| [`../decision_log.md`](../decision_log.md) | Reversal conditions and rejected options |