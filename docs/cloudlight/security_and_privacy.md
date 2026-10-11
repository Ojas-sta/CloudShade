# CloudLight — Threat Model, Data Classification and Incident Response

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ NOTHING IN THIS DOCUMENT HAS BEEN TESTED
>
> **No control described here has been implemented, configured, deployed or
> verified.** No school network has been accessed or probed. No device was
> compromised — or protected. No credential was created, attempted or rotated.
> No log was written. No incident has occurred, because no system exists.
>
> This is a **design-time threat model**. Its purpose is to make the first line of
> code written respect these constraints, not to certify a system that runs.

---

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in a standard or authoritative documentation, quoted with a URL in §Sources |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | A vendor or standards body documents it; we have not confirmed it on any device this school actually owns |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No external claim. Our engineering judgement |
| ❓ **UNKNOWN** | No evidence either way. Not assumed. To be answered by a named person at the school |
| ⛔ **NOT RECOMMENDED / NOT PERMITTED** | Rejected by our own judgement, by external guidance, or by school policy once known |

---

## 0. The core principle

> ### ⚠️ **A school LAN is not inherently trusted.**
>
> It is internal. It is **not** private.

Every control below is written on the assumption that the network is a potential
adversary. That is not paranoia and not a claim about this school — it is a
statement about the category:

| Property of a school LAN | Consequence for CloudLight |
|---|---|
| **Devices of unknown provenance** | Students bring laptops, phones and tablets. Guest and vendor devices attach. Camera systems, attendance terminals, and whatever the estate has accumulated |
| **Shared with students and guests** | The people nearest a classroom display are, for most of the day, students |
| **Administered by very few people** | Often one person, sometimes a vendor, rarely with dedicated time. A security posture that depends on one person's availability is fragile by construction |
| **Credential sharing is normal** | Staff share logins to make lessons work. A design assuming non-sharing is designing for a school that does not exist |
| **Unknown management capability** | ❓ Whether IT can enforce segmentation, filtering or even a firewall policy is `CL-NET-001`, `CL-NET-002`, `CL-NET-007` — all unanswered |
| **Physical access is shared** | Server rooms, offices and stores are unlocked more often than not |

⛔ **The consequence:** *reachability must not be treated as authorisation.*
Anything reachable from a classroom display must assume that a student, a guest
device, or a curious member of staff can reach it too.

⚠️ **The honest inverse:** a threat model written against an adversary we cannot
characterise becomes either theatre or paralysis. The mitigations below are
chosen to be **implementable by one competent person on school hardware** —
because a control requiring a security team to operate does not get operated in
a school.

---

## 1. Scope and assets

### 1.1 What is being protected

⚠️ **This list defines the scope. Anything not on it is out of scope here, and
anything added to it needs a threat row.**

| # | Asset | Class (§4) | Why it matters | Worst credible loss |
|---|---|---|---|---|
| **A1** | **Teacher credentials and authenticated sessions** | 🔴 RESTRICTED | Every consequential action is attributed to a signed-in teacher. Credentials root all authorisation | A teacher is impersonated; actions in the audit log are attributable to a person who did not perform them |
| **A2** | **Timetable and substitution data** | 🟠 INTERNAL → CONFIDENTIAL | ⚠️ **Students act directly on this.** It decides where a child goes and who supervises them | A student attends the wrong room; a teacher is not where a class expects them |
| **A3** | **Staff and student personal data** | 🔴 CONFIDENTIAL | Names, classes, attendance, marks, behaviour notes, and possibly medical and disciplinary records | Reputational and legal harm to a child or an employee; in India this engages the Digital Personal Data Protection Act, 2023 |
| **A4** | **NAS teaching resources, including confidential staff material** | 🟠 CONFIDENTIAL | Staff routinely place material on shared storage never intended for students: answer keys, moderation notes, appraisal files, a teacher's own notes | Confidential material displayed on a classroom wall to a class of children |
| **A5** | **The search index** | 🟠 Derived CONFIDENTIAL | ⚠️ **An index is a copy, and a copy outlives the permission.** Extracted text from every indexed document, in one place, queryable, with no NAS path in front of it | A single query returns material from across the school that no individual was ever entitled to see together |
| **A6** | **AI conversation history and query logs** | 🔴 RESTRICTED | Conversations can contain student names, marks, behaviour and staff grievances, typed freely into a chat box by anyone who can reach the display | A searchable archive of school conversations containing identifying personal data, retained indefinitely by default |
| **A7** | **The software update signing key** | 🔴 RESTRICTED | Whoever holds it can execute code on every classroom display | Persistent, school-wide, quiet code execution, indistinguishable from a legitimate deployment |
| **A8** | **Audit logs** | 🟠 INTERNAL → CONFIDENTIAL | The only record that answers "who did this". Also a behavioural record of staff, carrying its own privacy weight | An administrator erases their own actions; a misuse becomes uninvestigable |
| **A9** | **The SHANA delivery-request channel** | 🟠 INTERNAL | A request endpoint that can create a physical task in a corridor | Any host on the network can move the robot |

⚠️ **A4 and A5 are the pair most often missed.** A NAS with correct folder
permissions is a *controlled* thing. The moment a search index copies and extracts
its contents, it becomes an *uncontrolled* copy of the same material — and
`CL-SEC-007` (P0) asks precisely whether CloudLight must enforce the NAS's
permissions rather than hold its own.

### 1.2 From whom

| # | Adversary | Capability assumed | Motivation | What they want |
|---|---|---|---|---|
| **X1** | **Curious student** | Physical access to a classroom display, a phone, a free period, no technical skill | Curiosity, boredom, social leverage | To see what they should not see; to change what is on the screen |
| **X2** | **Student who photographs a screen** | A phone camera — a feature, not an exploit | Social currency, sometimes malice | A durable copy of a timetable, a name, a staff note |
| **X3** | **Opportunistic student** | X1 plus willingness to try a few six-digit codes | Boredom escalating to access | A session, or any document they can reach |
| **X4** | **Teacher with legitimate but broad access** | A valid account; may see more than they should | Convenience, curiosity, good intentions | A resource outside their class; a faster way to do their job |
| **X5** | **Departed staff member with a still-valid session** | A token or remembered code outliving their employment | Accidentally, or deliberately after a dispute | Continued access they no longer have a right to |
| **X6** | **Malicious or careless document author** | ⛔ **Not an external attacker.** A staff member or student who uploads a file | Carelessness, curiosity, or intent | Code execution on the server; a poisoned index; an injected instruction retrieved later |
| **X7** | **Insider with administrator rights** | Full local control of the host | Malice, or the cover-up of a mistake | Erasing evidence; persistent access; change that looks legitimate |
| **X8** | **Attacker on the school LAN** | Some foothold on any host or device, any skill level | Opportunistic | Credentials, database, update path, index, robot |

⚠️ **X6 inverts the usual assumption, and deserves emphasis.** The most likely
source of a malicious document, a poisoned index entry or an injection payload is
**not** an external attacker. It is a member of staff uploading a file containing
a macro, an embedded object, or a paragraph that reads like an instruction.
Treating staff uploads as trusted because "staff are not attackers" is the exact
reasoning that lets T08 and T09 through unmitigated.

⚠️ **X7 cannot be fully mitigated by software on a box one person administers.**
See §13.

---

## 2. Trust boundaries

```
 ┌─ DISPLAY SEGMENT ─────────────────────────────────────────────┐
 │  [Smartboard PC ×N]   ← students, guests and staff all here   │
 │  (kiosk browser on one shared device)                         │
 │  ⚠️ ASSUMED HOSTILE. Any of X1–X3 stands here.                │
 └──────────────┬───────────────────────────────────────────────┘
                │ B1  HTTPS — TLS required even inside the LAN
                │     (CL-SEC-024). The display is a CLIENT only.
                ▼
 ┌─ SERVER SEGMENT ──────────────────────────────────────────────┐
 │  [CloudLight host]  backend · database · search index ·       │
 │                      audit log · AI inference · update key    │
 │  ┌───────────────┐   ┌───────────────────┐                     │
 │  │ ADMIN CONSOLE │   │ PUBLIC APP PORT   │                     │
 │  │ ⛔ only from   │   │ ✅ from the       │                     │
 │  │ ADMIN segment │   │ display segment   │                     │
 │  └───────────────┘   └───────────────────┘                     │
 └──┬──────────┬───────────────┬─────────────┬──────────────────┘
    │          │               │             │
 B2 │ SMB      │ B3 mutual     │ B4 egress   │ B5 signed updates
    │          │   authn       │  ⛔ default │  ✅ verified ON
    ▼          ▼        DENIED │    DENIED   │  THE CLIENT
 ┌────────┐ ┌─────────┐   ┌────────┐  └──────────────────┐
 │  NAS   │ │ SHANA   │   │EXTERNAL│     ┌─────────────┐
 │resources│ │delivery │   │AI /    │     │UPDATE SOURCE│
 │⚠️ own  │ │requests │   │internet│     │signed builds│
 │ACLs    │ │⛔ NO    │   │⛔ PROHIB│     └─────────────┘
 │must be │ │MOTOR    │   │ITED    │
 │enforced│ │PATH     │   └────────┘
 └────────┘ └─────────┘

 ┌─ ADMINISTRATION SEGMENT ─────────────────────────────────────┐
 │  IT workstations · switch / AP / firewall administration    │
 │  ✅ The only segment permitted to reach the admin console    │
 └─────────────────────────────────────────────────────────────┘
```

| # | Boundary | Crosses | Direction policy | What must be true |
|---|---|---|---|---|
| **B1** | Display ↔ Server | School network | Display may reach **the application port only**. No inbound to the display, no admin surface | TLS even on an internal LAN (`CL-SEC-024`); a stable memorable local name; a permitted firewall path (`CL-NET-003`, `CL-NET-014`) |
| **B2** | Server ↔ NAS | SMB / file share | Server reads **only** the shares it must. No student or display device may hold a NAS credential | ⚠️ **The index must enforce the NAS's ACLs, not its own** (`CL-SEC-007`, P0) |
| **B3** | Server ↔ SHANA | Mutually authenticated service channel | CloudLight sends **high-level requests only**; SHANA sends **status only** | ⛔ **P0.** An unauthenticated task endpoint lets any host on the network create robot tasks (§11) |
| **B4** | Server ↔ External AI / internet | Egress | ⛔ **PROHIBITED by default.** Any egress is a named exception | If ever enabled, school records reach a third party and need explicit approval (`CL-SEC-011`) |
| **B5** | Update source ↔ Displays | Signed artefact distribution | ⛔ **The client verifies the signature.** A deployment credential alone is not sufficient | Client-side verification, or both controls (`CL-SEC-013`, `CL-OTA-004`) |
| **B6** | Admin console ↔ Server | Management segment | ⛔ **Unreachable from the display segment.** No exceptions | `CL-SEC-023` (P0). ❓ Whether the school can enforce this is unknown |

⚠️ **B4 is the boundary most likely to be crossed by accident.** "Just testing a
model", "just checking the API once", "the upload feature needs the internet" —
each is one misconfiguration from sending a student's name to a third party. It is
therefore expressed as a **default-deny egress posture**, not a policy someone
must remember.

⚠️ **B6 decides whether B1's flat network is survivable.** On one flat network, a
single compromised display is **one hop** from the control plane, and every other
control in this document is compensating for that. See §7.

---

## 3. Threat register

**Rating:** `L` likelihood × `S` severity, both 1–5.
🔴 High (15–25) · 🟠 Medium (8–14) · 🟡 Low (1–7)

**Status vocabulary:** `MITIGATION PLANNED` · `OPEN — APPROVAL REQUIRED` ·
`OPEN — ANSWER REQUIRED` · `MONITOR`

⚠️ **Every status below is `MITIGATION PLANNED` or an `OPEN` variant. None is
`IMPLEMENTED`, because nothing has been implemented.** Scores are deliberately
not inflated: a threat is High because a control designed for it is *absent by
default*, not because the scenario is exotic.

### 3.1 The register

| # | Threat | Actor | L | S | Score | Severity | Impact | Mitigation | Residual risk | Questionnaire |
|---|---|---|---|---|---|---|---|---|---|---|
| **T01** | **Unauthorised classroom access to a display** | X1, X3 | 5 | 3 | **15** | 🔴 High | Whatever the signed-out state shows becomes public. If it is a resource browser, students reach material at will; if an admin session survives, exposure is total | An unauthenticated **read-only home state** as the default, PUBLIC class only. No staff surface, no search, no assistant, no settings without authentication. Auto-return home after a defined idle period | 🟡 **Cannot be eliminated.** A screen in a corridor is visible to whoever is in front of it. The control decides *what* is visible, not whether | `CL-SEC-001` (P0), `CL-SEC-002` |
| **T02** | **Student operating an unattended smartboard** | X1 | 5 | 2 | **10** | 🟠 Medium | Navigation outside the intended surface; a resource opened; a display left unusable. Held at 2 because changing *consequential* data is T13/T14 | Read-only default; no destructive action without an authenticated confirmation; ⛔ **no signed-in state persists** across a reboot or a period change; kiosk mode with no address bar, no downloads; every state change logged | 🟡 **Accepted.** A student can always press a button. Pressing must not be consequential | `CL-SEC-001`, `CL-UX-020`, `CL-WIN-022` |
| **T03** | **Stolen or hijacked teacher session** | X3, X5 | 3 | 5 | **15** | 🔴 High | ⚠️ **Full account compromise and the worst audit damage.** Every action attributed to the teacher. A token replayed across classrooms is undetectable until the teacher notices something they did not do | Short **idle and absolute** timeouts; re-auth at every period change; the token bound to the device that created it; ✅ **immediate server-side revocation** from another classroom; invalidation on sign-out, role change and staff departure | 🟠 **Medium.** A token captured by someone with thirty seconds of physical access is captured. Revocation bounds the damage; it does not prevent capture | `CL-SEC-002`, `CL-SEC-004`, `CL-SEC-005` |
| **T04** | **Guessable six-digit codes and brute force** | X3 | 4 | 3 | **12** | 🟠 Medium | ⚠️ **A six-digit code is 10⁶ possibilities.** With no attempt limit it falls to a student with a phone and a free period. ✅ NIST SP 800-63B requires rate-limiting against failed attempts at every assurance level | Defined attempt limit, then lockout with administrator notification; short validity; single use; codes issued only in response to a server-initiated request; **and above all the scope restriction below** | 🟠 **Medium while the code is scoped as designed. 🔴 High if it ever authenticates a person.** See the constraint below | `CL-SEC-003` (P0) |
| **T05** | **Credential leakage — URLs, logs, local storage, git, screenshots** | X1, X7 | 4 | 5 | **20** | 🔴 High | ⚠️ **The highest-likelihood, highest-consequence row, and not hypothetical.** ✅ **VERIFIED in this repository:** `start_opencode_server.sh` holds a plaintext administrative password, binds to `0.0.0.0:4096`, and is not in `.gitignore`. Recorded as a standing finding against `start_opencode_server.sh`; ⚠️ **the script IS gitignored** (`.gitignore` line 119), so it is not committed — the exposure is local, not in version control | The absolute rules in §5. Runtime secrets only. `httpOnly`/`Secure`/`SameSite` cookies. ⚠️ **Rotation, not deletion, is the only remedy** | 🔴 **Live defect.** The existing credential must be rotated and treated as compromised. That requires the repository owner; it is not ours to close | `CL-SEC-006`, `CL-GIT-014`, `CL-GIT-015`, `CL-SEC-005` |
| **T06** | **Unauthorised resource access** | X4 | 3 | 4 | **12** | 🟠 Medium | A teacher reads material outside their class: another department's confidential notes, a colleague's appraisal, an answer key. Not a student intrusion — an over-broad permission doing the same job | ✅ **Enforce the existing NAS permissions rather than maintaining a parallel model** (`CL-SEC-007`). A second ACL system diverges, and it diverges **openly**. Deny by default; entitlement checked **at query time**, not at page load | 🟡 **Low if B2 holds. 🟠 High if CloudLight holds its own ACLs** — because then two systems disagree and only one is right | `CL-SEC-007` (P0), `CL-NAS-006` |
| **T07** | **NAS permission bypass via search** | X1, X4 | 4 | 4 | **16** | 🔴 High | ⚠️ **The default failure mode of any search feature.** The indexer reads everything it can reach, extracts text, and serves it to anyone who can type — with **no NAS path in front of the result and no folder structure to hide behind** | Index at the **permission** level, not the file level: every entry carries the source document's entitlement set, filtered at query time. Index only intended shares. ⛔ Exclude CONFIDENTIAL/RESTRICTED without an explicit recorded decision. Delete entries when the source is deleted | 🟠 **Medium, permanently.** ⚠️ The index is a derived copy of restricted content and a new thing to protect: its own backup, its own retention, its own ACLs | `CL-SEC-007` (P0), `CL-DATA-020`, `CL-NAS-004` |
| **T08** | **Malicious documents — viewers and parsers as an attack surface** | X6 | 3 | 4 | **12** | 🟠 Medium | Code execution on the **server** during indexing or extraction, or on a display during preview. Because indexing is automatic and search is student-triggerable, **one careless upload reaches every classroom** | Never execute or render untrusted documents server-side. Extraction in a **sandboxed, resource-limited, non-networked** process, discarded after use. Isolated preview viewer. Deny by extension and content-type. ✅ OWASP File Upload Cheat Sheet is the pattern. A scanner is a second layer, not the first | 🟠 **Medium.** Parsers are a large, historically weak attack surface, and ⛔ "trusted because staff uploaded it" is not a control | `CL-SEC-008` (P0), `CL-NAS-004` |
| **T09** | **Prompt injection via retrieved documents** | X6, any student who can place text where it will be indexed | 4 | 3 | **12** | 🟠 Medium | A paragraph in a staff document that reads as an instruction changes what the assistant answers — a fabricated announcement, a wrong substitution, **a confident invention on a classroom wall** | ⚠️ **Every retrieved document is untrusted data, never instruction.** No system prompt built from retrieved content; retrieval output fenced and labelled; ⛔ **a document can never cause an action, only inform an answer a human sees**; nothing reaches a display without deterministic validation | 🟠 **Medium and irreducible.** There is no complete defence against injection in a retrieval-augmented system. The control reduces blast radius; it does not remove the class of attack | `CL-SEC-009`, `CL-SEC-010` |
| **T10** | **AI data leakage to an external service** | X7, or an accidental configuration | 3 | 5 | **15** | 🔴 High | ⚠️ **One misconfiguration moves student names, staff records or a confidential document to a third-party processor.** Once sent it is subject to that party's retention, jurisdiction and breach history. No school decision undoes it | ⛔ **External inference PROHIBITED by default**; egress default-deny (B4). All inference stays on school infrastructure. Enabling it requires written approval, data minimisation and a documented retention position | 🟡 **Low while egress is denied. 🔴 High the moment it is enabled.** ⚠️ This row is a **switch, not a gradient** | `CL-SEC-011`, `CL-SEC-010`, `CL-NET-015` |
| **T11** | **Server compromise** | X7, X8 | 2 | 5 | **10** | 🟠 Medium | ⚠️ **Everything is on one box** — database, index, audit log, session store, signing key, NAS credential, and the model if local inference is enabled. Nothing else holds a copy of the secrets | The host runs **nothing else**; patched on a schedule (`CL-SRV-019`); no inbound internet; admin access only from the management segment (B6); secrets at runtime not configuration; backups encrypted and off-host (§9) | 🔴 **Concentrated by design.** Local-first means one host is the whole trust anchor. Accepted deliberately (§13 A3); mitigated by backup, audit and segmentation rather than reduced | `CL-SRV-005`, `CL-SRV-019`, `CL-GIT-015` |
| **T12** | **Unauthorised software update / build substitution** | X7, or anyone with write access to the update source | 2 | 5 | **10** | 🟠 Medium | ⛔ **Code execution in every classroom simultaneously, signed as legitimate.** The update path is a privileged code-execution path, and a correctly signed bad build is indistinguishable from a real one to every user in the school | **Client-side signature verification**, so the display refuses what it cannot verify — ⛔ not merely an internal deployment credential. Record the running version on each display so a bad release is identifiable and reversible | 🟠 **Medium.** ⛔ The signing key **is** asset A7. Anyone holding it can ship a valid build. It needs hardware custody, a written procedure and a rotation story | `CL-SEC-013`, `CL-OTA-004`, `CL-GIT-018` |
| **T13** | **Incorrect timetable publication** | X4, or a data-entry error — **not necessarily an adversary** | 3 | 4 | **12** | 🟠 Medium | ⚠️ **Students act on it directly.** A wrong room, a wrong teacher, a wrong period. ⚠️ CloudLight does not create the error — it **displays it on every screen at once**, turning a small clerical mistake into a school-wide one | Versioned timetable with immutable history and rollback; two-stage publication for consequential changes; constraints preventing conflicting entries; **the dashboard shows the published version and its age**, so a stale timetable is visibly stale | 🟠 **Medium, irreducible.** Timetabling is a hard, pressure-heavy clerical task done by few people, often late. CloudLight can make errors **visible and reversible**, not impossible | `CL-SEC-014`, `CL-DATA-017`, `CL-DATA-018` |
| **T14** | **Incorrect substitution displayed to students** | X4, or a mistake under time pressure | 4 | 4 | **16** | 🔴 High | ⚠️ **Higher than T13, and the gap is deliberate.** Substitutions are last-minute by nature, made under pressure, rarely double-checked, often published from a phone. A student who trusts a stale board walks to an empty room; a teacher who is not where the board says is unavailable during a period they should cover | Approval before publication, mirroring how the school corrects a wrong substitution **today**; a defined escalation path; who may publish a last-minute change at all; ⛔ a *proposed* substitution is never shown as confirmed; visible publication timestamp; a **withdraw** action as easy as publish | 🟠 **Medium.** ⚠️ The mitigation is **procedural and the school must own it.** Engineering cannot make a rushed human decision correct | `CL-SEC-015`, `CL-SUB-001`, `CL-SUB-002`, `CL-SUB-003`, `CL-SUB-004` |
| **T15** | **Replayed delivery requests** | X8, or a duplicated retry | 2 | 4 | **8** | 🟠 Medium | ⚠️ **A second robot moving through a corridor carrying an item nobody asked for.** A physical event, not a data bug, and indistinguishable from a legitimate second task | ✅ **Idempotency key on every request**, generated before the first transmission attempt; SHANA treats a repeated key as the same request and creates no second task; reconciliation on restart rather than re-send. **P0** | 🟡 **Low given idempotency. 🔴 High without it.** ⚠️ It protects against accidental **and** deliberate replay of a captured valid request. It does not protect against a *fresh* valid request — that is T16 | `CL-SEC-016` (P0), `CL-SHANA-018` |
| **T16** | **SHANA integration failure or a rogue task channel** | X8 — any host on the network | 3 | 4 | **12** | 🟠 Medium | ⛔ **An unauthenticated task endpoint lets any device that can reach SHANA create robot tasks.** A student with a laptop, or a guest device, becomes a delivery client. With the motor-command path absent the worst case is bounded, but that is still a moving machine in a corridor with no identified requester | ✅ **P0. Mutual authentication** — mutual TLS or a signed request token with short expiry. ⛔ Not a shared static key alone: single-key systems are single points of compromise. ⛔ **No motor, path, speed or actuator command path anywhere in CloudLight**, in any mode including debug, admin and exhibition | 🟠 **Medium, permanently.** ⚠️ Mutual authentication proves *which system* is calling. It does not prove a human authorised the delivery. Accepted — §13 A6 | `CL-SHANA-015` (P0), `CL-SEC-016`, `CL-SEC-017`, `CL-NET-021` |
| **T17** | **Stale dashboard data presented as current** | Nobody — a **system** failure acting as an adversary | 4 | 3 | **12** | 🟠 Medium | Displays run for hours and school networks drop. A board shows "Period 4 · Room 12 · Ms Rao" from 09:40 while it is 14:20. ⚠️ **A person then acts on a false picture** | ⚠️ **Every value carries its age.** Staleness tiers with a defined threshold; beyond it an honest **"data unavailable"** rather than a confident stale value; ⛔ **`UNKNOWN` never renders as `IN_PROGRESS`** | 🟠 **Medium.** ⚠️ Fail-closed means an honest blank screen — a worse user experience, a better security property. Deliberate trade (§12) | `CL-SEC-018`, `CL-SEC-017`, `CL-SEC-021` |
| **T18** | **Audit-log tampering** | X7 — an administrator covering their tracks | 3 | 4 | **12** | 🟠 Medium | ⚠️ **An administrator-editable log provides accountability to nobody, including the school.** The one actor whose actions most need recording is the one able to erase them | **Append-only with tamper evidence**: a chained hash or signed batch, so modifying entry *n* invalidates *n+1…end*. Forward an off-host copy the administrator does not control. Log every sign-in, publication, substitution change, resource access, delivery request and deployment | 🔴 **A full administrator can defeat software-only tamper evidence.** ⚠️ This is the honest limit: it raises the cost and leaves a trace. Only an off-host copy the admin cannot reach closes it — an infrastructure question, not a code question | `CL-SEC-019` (P0), `CL-SEC-020`, `CL-DATA-009` |
| **T19** | **Network loss and server outage** | Power cuts, switch failure, a fibre cut, an unplanned reboot | 5 | 2 | **10** | 🟠 Medium | ⚠️ **Availability first — but with a consequence that is easy to miss: an outage removes authentication enforcement.** If any surface degrades to "open" when the server is unreachable, an outage has silently converted an authenticated system into an unauthenticated one. A board stuck on a cached privileged view is a disclosure with a long tail | ⛔ **Fail closed.** Nothing becomes *more* accessible. Cached content limited to PUBLIC class; return to the read-only home state, never the last privileged screen; UPS and automatic restart; manual fallback agreed (`CL-SEC-022`) | 🟠 **Medium, and routine.** ⚠️ Likelihood is genuinely 5 — this will happen many times a term. It is a security event **only if the failure mode opens something** | `CL-SEC-021`, `CL-SEC-022`, `CL-DATA-014`, `CL-NET-019` |
| **T20** | **Exposed administration endpoints** | X1, X8 | 3 | 5 | **15** | 🔴 High | ⚠️ **On a flat school LAN a compromised display is one hop from the control plane.** User management, timetable publication, permission changes, deployment and the console become reachable from a classroom. Whoever gets there holds the timetable office | Administration interfaces reachable **only** from the management segment (B6); ⛔ no admin surface on any port the display segment can reach; a separate service credential; `CL-SEC-023` names exactly which functions must be unreachable, so IT has a concrete request rather than a vague one | 🔴 **High if the school runs a flat network.** ⚠️ This row is conditional on `CL-NET-001` and is **the single most important unanswered question for CloudLight's security posture** | `CL-SEC-023` (P0), `CL-SEC-024`, `CL-NET-001`, `CL-NET-007`, `CL-NET-023` |

### 3.2 Distribution

| Severity | Count | IDs |
|---|---|---|
| 🔴 **High (15–25)** | **6** | T01, T03, T05, T07, T10, T20 |
| 🟠 **Medium (8–14)** | **14** | T02, T04, T06, T08, T09, T11, T12, T13, T14, T15, T16, T17, T18, T19 |
| 🟡 **Low (1–7)** | **0** | — |

⚠️ **No threat here scores Low.** That is not alarmism; it is the arithmetic of a
system holding staff and child personal data on a network nobody can currently
describe. **The register's value is in the ranking, not the total.** T05 and T20
decide whether the rest of this document is comfortable or merely honest.

### 3.3 The four rows that carry the argument

| Row | The one sentence that decides it |
|---|---|
| **T04** | ⛔ **A six-digit code authorises exactly one action, or it is a second factor. It never authenticates a person.** This holds T04 at 🟠 12 instead of 🔴 20. If a code can produce a session the score becomes 4 × 5 = **🔴 20** and the design has changed character. The constraint is not negotiable and does not depend on the school's answer to `CL-SEC-003` |
| **T05** | ⚠️ The highest-likelihood row is the one **already realised**. A credential in git history **stays there permanently** — rewriting history does not reach a clone, a fork, a mirror or last month's backup. Only **rotation** works, which is why §5 says "never" rather than "be careful" |
| **T07** | ⚠️ **The index is the threat, not the feature.** A filesystem with correct permissions is a well-understood control. A search index over it is a **new access-control surface nobody has reviewed** — and it is the feature teachers will ask for first |
| **T14** | Timetables are settled days ahead and reviewed; substitutions are made at 08:40, under pressure, by someone rushing to cover a gap. ⚠️ **The difference is process, not software**, and process is the school's to own |

---

## 4. Data classification

⚠️ **Classification is a proposal.** The school may have its own scheme; if it
does, **the school's scheme governs** and this table merges into it rather than
replacing it.

| Class | Examples | Where it lives | Who may read | Retention | Classification rationale |
|---|---|---|---|---|---|
| 🟢 **PUBLIC** | School-approved announcements; today's timetable and substitutions once published; room names; periods and bells; a published events calendar | CloudLight DB (published subset); every classroom display | **Anyone who can see the screen.** No authentication | While current; superseded versions pruned per `CL-DATA-016` | ⛔ **Approved for school publication.** The only class permitted on a signed-out display — and PUBLIC *because the school published it*, never because it is harmless in isolation |
| 🟡 **INTERNAL** | Draft timetable; unpublished substitutions; staff directory; resource folder structure and metadata; delivery request records; configuration | CloudLight DB; audit log; NAS metadata | **Any authenticated staff member**, subject to role | Drafts until superseded or a term. Delivery records per `CL-SEC-025`. Directory until staff leave | Useful across the school; disclosure would be embarrassing or operationally harmful but not, alone, damaging to a child. ⚠️ **A draft substitution is INTERNAL, not PUBLIC — this is the entire reason T14 exists** |
| 🟠 **CONFIDENTIAL** | Staff records; salary; medical and leave information; disciplinary records; **examination papers and answer keys**; moderation notes; appraisal material; student results and behaviour notes | CloudLight DB; **the NAS**; ⛔ **the search index** | **Named individuals and roles only**, enforced by the NAS's own permissions where one exists (`CL-SEC-007`) | School policy; deletion on request where applicable (`CL-SEC-025`) | Disclosure causes real harm to a person — financial, medical, professional — or defeats the purpose of an assessment |
| 🔴 **RESTRICTED** | Passwords and hashes; session tokens; **the update signing key**; NAS and server service credentials; AI conversation logs containing student-identifying content; full audit logs; backups holding any of the above | ⛔ **Nowhere a user can reach.** Environment or secret store at runtime; encrypted backup media | **Named systems and named people only.** Not a role — a specific credential | Secrets: **rotated, never retained.** Tokens until expiry. AI conversation logs: **shortest workable period**, then deleted or irreversibly anonymised | ⛔ **Compromise of any one is a security incident by definition.** Not "a concern" — an incident |
| ⛔ **NEVER STORED** | Raw passwords (only a salted, costed hash); motor or actuator parameters; raw robot video; biometric data; any persistent student identifier collected for surveillance | — | — | — | ⛔ **Existence is the defect.** Per `../safety/student_privacy.md`: no student is ever identified, no raw video is retained |

### 4.1 ⚠️ Examination material is the class that breaks the others

| Property | Examination papers and answer keys |
|---|---|
| **Sensitivity while sealed** | 🔴 **RESTRICTED** — disclosure defeats the assessment entirely |
| **Time-sensitivity** | ⚠️ **Extreme.** Value expires in hours and then becomes permanently worthless to the student — and permanently damaging to the school |
| **Sensitivity once leaked** | ⛔ **Permanent.** ⚠️ A leak cannot be recalled. No retraction reaches a photograph, a forward, or a re-uploaded question paper |
| **Search index** | 🔴 **The highest-consequence item in this document.** ⚠️ **Indexing an answer key creates a permanent, queryable, low-latency copy of material whose entire value is its short life.** The file can be deleted from the NAS and **the extracted text remains** (`CL-DATA-020`) |
| **Therefore** | ⛔ **CONFIDENTIAL and RESTRICTED material is excluded from the index by default, by share and by path** — not by filename pattern. A per-file allow-list of what *may* be indexed is safer than a block-list of what may not |

⚠️ **This is the clearest example of why derived data is a security concern
rather than a performance concern.** Deleting the source file is a *file*
operation. Deleting it from every index, cache and extracted-text store is a
*systems* operation. Only the second removes the copy.

### 4.2 Classification obligations

| # | Obligation | Question |
|---|---|---|
| 1 | ⚠️ **Logs about students and staff carry their own privacy obligations.** Classifying a record CONFIDENTIAL does not stop it being written into a less-protected log | `CL-OPS-011`, `CL-OPS-010` |
| 2 | ⚠️ **Every class needs a stated retention period**, and deletion must reach backups, the index and extracted text — not only the live database | `CL-SEC-025`, `CL-DATA-015`, `CL-DATA-016`, `CL-DATA-020` |
| 3 | ⚠️ **A backup inherits the highest class of anything in it.** An unencrypted backup is ungoverned CONFIDENTIAL material in a drawer | `CL-DATA-010` (P0) |
| 4 | ⚠️ **A class with no enforcement mechanism is a label, not a control** | `CL-SEC-007` |

---

## 5. Secrets handling rules

⛔ **These rules are absolute. Not guidance, not defaults, not subject to "just
this once".**

| # | Rule | Why it is absolute |
|---|---|---|
| **S1** | ⛔ **Never in source control.** No passwords, tokens, keys, certificates or NAS credentials in any tracked file | ⚠️ **A credential in git history stays there permanently.** History rewriting does not reach a clone, a fork, a mirror, or last month's backup. Only **rotation** is effective |
| **S2** | ⛔ **Never in a URL** — path, query string, fragment or redirect target | URLs land in browser history, proxy logs, server access logs, referrer headers, bookmarks, and screenshots of the address bar, all of which outlive the request |
| **S3** | ⛔ **Never in browser `localStorage`, `sessionStorage` or IndexedDB** | ⚠️ Any client-side store persists on a **shared kiosk** and is readable by the next class in that room. Session cookies must be `httpOnly`, `Secure`, `SameSite` (`CL-SEC-005`) |
| **S4** | ⛔ **Never in a public configuration file** — no committed `.env`, no `config.json`, no settings screen that displays a secret, no sample file with a real-looking value | Samples get copied and configuration files get committed. The boundary is never where you put it |
| **S5** | ⛔ **Never in logs, crash reports, stack traces, support transcripts or screenshots** | ⚠️ Log aggregation and error-reporting services are exactly where secrets accumulate without anyone deciding to put them there. ✅ OWASP's Logging Cheat Sheet: data should not normally be logged |
| **S6** | ⛔ **Never displayed on a classroom screen after the teacher looks away** | ⚠️ The screen faces a corridor. An unmasking password field, a copied token on the clipboard, or a "your code is 481920" panel left for thirty seconds is a **physical disclosure** |
| **S7** | ⛔ **Never in the questionnaire Evidence column** of `../../questionnaires/Additional_CloudLight_Questionnaire.md` | ⚠️ Evidence is the one field explicitly invited to hold real artefacts. It must never hold a real secret — only a redacted screenshot, or a described test whose result is a yes/no |
| **S8** | ⛔ **No real school credential ever enters this repository** — in any file, branch, commit message, issue or draft | ⚠️ There is no stage at which a credential is "only temporarily" here. If it arrives, it is compromised and must be rotated on arrival |
| **S9** | ✅ **Secrets are supplied at runtime** — environment or secret store — never read from a file inside the repository | Makes S1 true by construction rather than by discipline |
| **S10** | ✅ **Rotation is a documented procedure, exercised before it is needed** | An untested rotation during an incident is a failed rotation |

⚠️ **S1 is not hypothetical in this repository.** ✅ **VERIFIED** —
`start_opencode_server.sh` holds a plaintext administrative password, binds to
`0.0.0.0:4096`, and is not in `.gitignore`. Recorded 🔴 Critical in
the repository root files `start_opencode_server.sh` and `.gitignore`. **The
credential value is deliberately not reproduced in any document in this
repository, and this one is no exception.**

⚠️ **The rule that matters most:** ⛔ **If a real school credential is ever
pasted into this repository, the correct response is to rotate the credential —
not to delete the line, remove the file, or rewrite the commit.**

**Mapped to:** `CL-SEC-006` (secrets handling, P1) · `CL-GIT-014` (secrets policy
in source control, P0) · `CL-GIT-015` (where school credentials live at runtime,
P1) · `CL-GIT-019` (school configuration stays outside the repository) ·
`CL-SEC-005` (acceptable credential stores).

---

## 6. Authentication and authorisation controls

⚠️ **Summary only.** The full design belongs in `authentication_and_authorisation.md`.
⚠️ **Not present in this repository at the date above** — a planned document.
❓ Nothing below has been implemented.

| # | Control | Statement | Reference |
|---|---|---|---|
| **1** | ✅ **Individual accounts for anything consequential** | ⛔ No shared, generic, class-level or room-level logins for timetable publication, substitution, permission change, deployment or user management. ✅ `CL-DATA-004` asks directly and its own rationale is the answer: shared logins make the audit trail meaningless, and the audit trail is what makes permission filtering *verifiable* | `CL-DATA-004`, `CL-DATA-003` |
| **2** | ✅ **`httpOnly`, `Secure`, `SameSite` cookies** | `httpOnly` keeps the session out of `document.cookie` and therefore out of S3. `Secure` keeps it off plaintext. `SameSite` cuts cross-site request abuse. ✅ OWASP Session Management Cheat Sheet treats all three as baseline | §5 S3 |
| **3** | ✅ **Short idle timeout *and* short absolute timeout** | ⚠️ **Both, and they are not interchangeable.** Idle bounds "signed in and abandoned"; absolute bounds "signed in and continuously used by someone else". ✅ NIST SP 800-63B requires **both** at every assurance level — AAL1: an inactivity timeout *"MAY be applied but is not required"*; AAL2: overall ≤ 24 h, inactivity ≤ 1 h | `CL-SEC-002`, `CL-SEC-018` |
| **4** | ✅ **Re-authentication at the period change** | The room changes hands. The session should not survive the handover | `CL-SEC-002` |
| **5** | ✅ **Immediate server-side revocation** | ⛔ Token deletion takes effect **at once**, from another classroom, not at next login. Revocation local to the captured device is not revocation | `CL-SEC-004` |
| **6** | ⛔ **A six-digit code authorises exactly one action, or it is a second factor. It never authenticates a person.** | ⚠️ The load-bearing constraint of T04. A code that can produce a session converts 🟠 12 into 🔴 20 | `CL-SEC-003` |
| **7** | ✅ **Rate limiting, attempt limits, lockout with administrator notification** | ✅ NIST SP 800-63B: verifiers *"SHALL implement a rate-limiting mechanism that effectively limits the number of failed authentication attempts"* — at every assurance level, not as an enhancement | `CL-SEC-003` |
| **8** | ✅ **Deny by default, enforced server-side at query time** | ⛔ A permission checked in the frontend is a UI hint. ✅ OWASP: *"Never rely on client-side access control checks"*; validate on every request | `CL-SEC-007` |
| **9** | ✅ **Enforce the NAS's permissions; do not maintain a parallel model** | A second ACL system diverges from the first, and it diverges **openly** | `CL-SEC-007` |
| **10** | ✅ **Least privilege per role / class / subject / department** | `CL-DATA-005` asks which the school uses; the answer decides whether filtering is a filter or a join | `CL-DATA-005` |
| **11** | ✅ **Step-up authentication for privileged actions** | Publishing a timetable, changing a permission and deploying a build are all higher-consequence than viewing a timetable | `CL-SEC-014`, `CL-SEC-015`, `CL-OTA-004` |
| **12** | ⚠️ **NIST assurance levels are a reference, not a compliance target** | ✅ SP 800-63B is written for US federal agencies. A school timetable system is not one. Its **structural** requirements — rate limiting, dual timeouts, revocation, single-use secrets — transfer. Its specific numeric thresholds do not | — |

⚠️ **Control 1 is where schools push back, and the pushback is usually
reasonable.** A shared staff login *is* how many schools operate today. The answer
is not to ignore the requirement — it is to state plainly what a shared login
costs: every audit-log action becomes unattributable, `CL-SEC-019`'s tamper
evidence protects nothing worth protecting, and `CL-SEC-004`'s revocation cannot
revoke a session that belongs to five people.

---

## 7. Network boundaries and segmentation

> ### ⚠️ **A compromise of any single classroom display must not be one hop from the control plane.**

This is not hygiene. On a flat network, an attacker who reaches one classroom
board has by default reached everything, because nothing is between them. Every
control in §3 is then compensating for a missing boundary rather than standing on
its own.

### 7.1 Requested segmentation

| Segment | Contains | May reach | ⛔ Must not reach |
|---|---|---|---|
| **Display** | Classroom smartboards; staff devices in classrooms | The application port on the server segment; DNS | Management segment; admin console; NAS admin interfaces; other school server roles |
| **Server** | The CloudLight host | Display segment (app port); DNS; NAS file-share ports | Management segment except from the management address; guest networks; CCTV |
| **Management** | IT workstations; switch, AP and firewall administration | Everything | — |
| **SHANA** | The robot link — a **separate wired Ethernet segment** | The server's dedicated interface only | ⚠️ **Must not share a broadcast domain with anything else** |

⚠️ **The SHANA segment is a different network path entirely** — wired Ethernet,
an isolated subnet, its own interface on the host. The failure mode to avoid is
sharing a broadcast domain with the classroom LAN, where discovery succeeds and
unicast fails — a fault that looks like a network problem and is very hard to
diagnose. See `../infrastructure/school_network.md` and
`shana_integration.md` §3.

### 7.2 Specific path restrictions

| Path | Policy | Why |
|---|---|---|
| **Display → admin console** | ⛔ **Deny** | `CL-SEC-023` (P0). User management, timetable publication, permission changes and deployment must be unreachable from a student-facing segment |
| **Display → NAS** | ⛔ **Deny directly.** All NAS access is server-mediated | A board holding a NAS credential is a board holding a share credential |
| **Server → NAS** | ✅ **Narrow.** Read-only, specific shares, service account only | Least privilege: a compromised server gets the teaching resources, not the whole file server |
| **Server → SHANA** | ✅ **Narrow.** One interface, one destination, one port | `CL-SEC-016`, `CL-SHANA-015` |
| **Server → internet** | ⛔ **Deny by default.** Named exceptions only | T10. Reduces exposure and makes exfiltration conspicuous |
| **Update source → displays** | ✅ **Signed artefacts, client-verified** | T12. `CL-SEC-013`, `CL-OTA-004` |
| **Board → board** | ⛔ **Isolate where possible** | Client isolation (`CL-NET-008`). Without it one compromised board reaches every other |

**Mapped to:** `CL-SEC-023` (P0) · `CL-NET-001` · `CL-NET-007` · `CL-NET-008` ·
`CL-NET-014` · `CL-NET-023` · `CL-SEC-024`.

### 7.3 ⚠️ If the school runs a flat network — the honest position

❓ **Whether this school segments anything is unknown.** `CL-NET-001` (P0) asks
directly and is unanswered. Flat networks are common in Indian schools and it would
be dishonest to assume otherwise.

| If the answer to `CL-NET-001` is… | Then |
|---|---|
| **"Yes, classroom devices are segmented"** | The §7.1 plan is a configuration request, not a construction project. Proceed |
| **"No, it is one flat network"** | ⚠️ **Segmentation cannot be assumed.** The compensating control becomes: **strong application-layer authentication on every surface; a separated administrative surface reachable only by a route classroom devices do not use; default-deny egress from the server; off-host log copies.** ⛔ **And this must be recorded as an explicitly accepted risk, by a named person, in writing — not absorbed silently** (§13 A4) |

⚠️ **"We will rely on the password" is not a compensating control.** Under T05
(🔴 20) and T20 (🔴 15), a single credential path guarding a control plane one
compromised board can reach is not a mitigation. It is a single point of failure
with extra steps.

---

## 8. Logging, monitoring and retention

### 8.1 What must be logged

| Event | Why | Class |
|---|---|---|
| Sign-in success, sign-in failure, **lockout**, sign-out | The only attribution anchor for everything else | INTERNAL |
| **Session revocation, and who requested it** | `CL-SEC-004`. An unexplained revocation is a misconfiguration or an incident | INTERNAL |
| Timetable publication **and the version published** | `CL-SEC-014`. Answers "what did the board say at 09:40?" | INTERNAL |
| Substitution proposal, approval, publication, **withdrawal** | `CL-SEC-015`, `CL-SUB-002` | INTERNAL |
| Resource access — **who read what, when** | T06, T07. The permission model is verifiable only if reads are recorded | CONFIDENTIAL in aggregate |
| Permission change; role assignment | T06, T20 | CONFIDENTIAL |
| **Administrative login and every administrative action** | T18. The actor most in need of recording | CONFIDENTIAL |
| Delivery request created, sent, accepted, rejected, completed | T15, T16 | INTERNAL |
| **Deployment: version, signature result, deploying identity** | T12, and the only way to know which display runs which build (`CL-GIT-018`) | RESTRICTED-adjacent |
| Lockout and repeated-failure events | T04 | INTERNAL |
| Service start, stop, crash, failed update | T11, T19 | INTERNAL |

⛔ **What must never be logged:** any secret, token, password or signing key
(§5 S5) · raw document contents · the full text of assistant conversations ·
anything that would turn the log into a surveillance record.

### 8.2 ⚠️ Logs must not contain secrets

| Risk | Rule |
|---|---|
| Credentials logged on an authentication error path | ✅ OWASP's Logging Cheat Sheet is explicit: *"Data should not normally be logged"* — and lists credentials among the things not to log |
| Session tokens in access logs | ⛔ Redact at the writer. A log that can be unredacted later is not redacted |
| **AI conversation content in query logs** | ⚠️ **Store the query identifier, the actor, the timestamp and the outcome — not the text.** The text may contain a child's name, a mark or a staff grievance, and it is exactly what ends up in a support ticket |
| Filenames and usernames in diagnostics | `CL-OPS-011` asks what may appear in logs about students and staff and who may read them. ⚠️ **Both unanswered** |

### 8.3 Append-only with tamper evidence

| Property | Requirement | Reference |
|---|---|---|
| **Append-only** | No update path to an existing entry. Corrections are **new** entries | `CL-SEC-019` (P0) |
| **Tamper-evident** | Chained hash or signed batch. ⚠️ Modifying entry *n* invalidates *n+1…end*, so an edit or deletion is detectable | `CL-SEC-019` |
| **Copy off-host** | ⚠️ **A copy the administrator cannot reach is the only real defence**, because a host-equivalent actor can defeat software-only evidence | `CL-DATA-009` |
| **Clock integrity** | A log whose timestamps can be rewound is not an audit log. Monotonic time where possible; record clock drift explicitly | `CL-SRV-020` |
| ⚠️ **Honest limit** | 🔴 **A full administrator can defeat software-only tamper evidence.** A residual risk (§13 A5), not a solved problem | `CL-SEC-019` |

### 8.4 Retention, size, and the privacy of logs

| Aspect | Position | Reference |
|---|---|---|
| **Retention per category** | ⚠️ **Undefined.** Timetable changes, substitutions, sign-ins, resource access, delivery records, AI conversation history and audit logs each need a stated period | `CL-SEC-025`, `CL-DATA-015` |
| **Log rotation** | ✅ **Bounded by size, not unbounded growth.** School disks fill silently and take the whole platform down — an unattended display box with a full disk is a common and entirely avoidable outage | `CL-OPS-016`, `CL-SRV-021` |
| **Retention must reach backups and the index** | ⚠️ A policy that prunes the live database but leaves the data in a backup and an index is **not a retention policy** | `CL-DATA-016`, `CL-DATA-020` |
| ⚠️ **Logs about students and staff carry their own privacy obligations** | A record of who signed in, when, from which display and which resources they read is a behavioural record of identifiable people. ⚠️ **It needs its own access control and its own retention justification — logging more is not free** | `CL-OPS-011`, `CL-OPS-010` |
| **Who may read them** | ⛔ **Not "all teaching staff."** Access to audit logs is itself a privilege, and over-broad visibility is a privacy problem in its own right | `CL-SEC-020` |
| **Monitoring vs. surveillance** | ⛔ **No per-student activity tracking of any kind.** A classroom system recording what each child searched is a different product with different legal consequences | `../safety/student_privacy.md` |

⚠️ **`CL-SEC-020` asks whether audit information is visible to teaching staff, and
that is as much a privacy question as a security one.** Over-broad visibility
creates a problem; under-broad coverage leaves the highest-consequence actions
unattributable. Neither default is safe.

**Mapped to:** `CL-SEC-019` · `CL-SEC-020` · `CL-SEC-025` · `CL-OPS-010` ·
`CL-OPS-011` · `CL-OPS-016` · `CL-DATA-015` · `CL-DATA-016` · `CL-DATA-020`.

---

## 9. Backup and recovery as a security control

> ### ⚠️ **A backup on the same disk is not a backup. A backup never restored is not a backup. And a backup inherits the highest classification of anything inside it.**

### 9.1 Three rules

| # | Rule | Why |
|---|---|---|
| **B1** | ⛔ **Never on the same disk, never in the same directory tree** | A disk failure destroys both. A mistaken recursive delete takes both. `CL-DATA-009` asks about off-site; `CL-OPS-012` asks what is backed up at all |
| **B2** | ✅ **Encrypt every backup** | ⚠️ **A backup inherits the sensitivity of its contents.** An unencrypted backup is an ungoverned second copy of CONFIDENTIAL and RESTRICTED material in a drawer or an inbox. `CL-DATA-010` (P0) asks this and the answer must be yes |
| **B3** | ⛔ **An untested backup provides no assurance** | `CL-OPS-014` (P0) asks whether any school system backup has ever been restored from, and by whom. ⚠️ **A restore test that happens "when there is time" does not happen.** Cheapest drill in this document: one spare machine and a few hours |

### 9.2 What must be backed up, and what need not be

| Item | Rebuildable? | Priority | Why |
|---|---|---|---|
| **Database** — timetable, substitutions, users, permissions, audit | ⛔ **No** | 🔴 **Critical** | ⚠️ **The only system of record.** Everything else can be reconstructed; this cannot |
| **Configuration and role mappings** | 🧪 Partly | 🔴 High | Small, but reconstructible only with the original knowledge — usually one person's memory |
| **Audit logs** | 🧪 Partly | 🔴 **High — part of incident response** | ⚠️ **See §9.3.** Losing the log during an incident destroys the incident |
| **Search index** | ✅ Yes | 🟡 Low | Derived. Rebuildable from source. ⚠️ Backing it up saves time, not correctness — **and it is also a copy of restricted content that must be encrypted and that expires with its source** (`CL-DATA-020`) |
| **Cached model weights** | ✅ Yes | 🟢 Low | Re-downloadable |

### 9.3 ⚠️ Recovery of audit logs is part of incident response

This is the security-specific point, and it is filed under operations, so it is
easy to miss:

| Situation | What happens if the log was not protected |
|---|---|
| An administrator tampered with the log to cover a mistake | ⚠️ **The off-host copy is the only evidence that survives.** Without it the question is permanently unanswerable |
| An attacker deletes the log after a compromise | ⚠️ **Deleting the log is the most diagnostic action an attacker can take.** If deletion is possible, its absence of record is itself inconclusive |
| A restore after catastrophic loss | ⚠️ **Restoring the database without restoring the logs creates a history with a silent gap** — the most dangerous possible artefact, because it looks complete |
| A disputed timetable change | ⚠️ "What did the board show on Tuesday?" is unanswerable without a retained log, and the dispute becomes unresolvable (`CL-DATA-017`) |

⚠️ **Therefore: audit logs are backed up, encrypted, copied off-host, and
retained on a schedule independent of the database's.** A procedure that covers
the database and skips the log has produced a recovery story that fails precisely
when it is needed.

### 9.4 Recovery is a security control only after it is rehearsed

| State | What it proves | What it does not prove |
|---|---|---|
| **A backup exists** | Something was copied | Nothing about completeness, currency or restorability |
| **A backup has been restored** | One restore succeeded | Nothing about the next one |
| ✅ **A backup was restored onto a clean machine and the data verified** | Recovery is real | — |

⚠️ **And the institutional risk:** if exactly one person can restore CloudLight,
the system is **personal knowledge, not institutional knowledge**, and it fails
the moment that person is absent (`CL-OPS-006`, `CL-OPS-015`).

**Mapped to:** `CL-DATA-008` · `CL-DATA-009` · `CL-DATA-010` · `CL-DATA-011` ·
`CL-DATA-012` · `CL-OPS-012` · `CL-OPS-013` · `CL-OPS-014` · `CL-OPS-015`.

---

## 10. Incident response

⚠️ **Nothing below has been rehearsed. No runbook exists, no contact has been
named, no scenario has been practised.** ❓ `CL-SEC-026` asks who the named
contact is — unanswered. **A runbook never walked through is a document, not a
capability.**

### 10.1 Severity, in school terms

| Level | Definition | Examples | First hour |
|---|---|---|---|
| 🔴 **Critical** | Credentials, the signing key, or restricted personal data compromised; a physical-safety path affected; a rogue robot task channel exists | Update key exposed; admin console reached from a classroom; CONFIDENTIAL records disclosed; any possible motor-command path found | **Revoke, isolate, notify** |
| 🟠 **Serious** | Authorisation bypassed but no credential known lost; an action taken in the system's name without authority | NAS permissions bypassed via search; a substitution published by someone who should not have been able to; an unauthorised deployment | **Preserve, then contain** |
| 🟡 **Moderate** | Disclosure without clear privilege escalation; availability degradation with no safety or privacy effect | Topology in a log; a stale board showing wrong data; repeated lockouts; assistant unavailable | **Record, investigate, schedule** |
| ⚪ **Minor** | Policy violation with no material exposure | Two teachers sharing a password where the loss is negligible; a stale shared folder | **Log and review** |

⚠️ **Severity is assessed on impact to the school, not on the sophistication of
the technique.** A student guessing a code with a phone during a free period is a
🔴 or 🟠 event. A sophisticated intrusion that read nothing and changed nothing
is 🟡.

### 10.2 The three FIRST actions

> ### ⚠️ **One tension, stated first.** Revoking access and cutting paths usually
> destroys the evidence needed to determine what was accessed. **Preserve first
> where it takes seconds** — a screenshot of the screen, a note of the time,
> exporting the log before touching anything. **But a compromise that is still
> live outranks preservation.** An attacker still signed in at 11:00 is a bigger
> problem than a missing forensic detail at 11:30. When in doubt, contain — and
> document the loss of evidence.
>
> ❓ **`CL-SEC-012` asks the school to decide this priority order — revoke first,
> cut first, or preserve first. It is unanswered, and this document does not get
> to decide it for the school.**

| Situation | **FIRST action** | Then | Never |
|---|---|---|---|
| **Suspected credential compromise** — a password, token or code may have been seen, copied, photographed or guessed | 🔴 **Revoke server-side, immediately.** Invalidate the session and force re-authentication — **including on every device the credential could have reached**. Rotate the credential itself, not just the session | Establish scope from the audit log: which actions, from where, when. Notify the affected person. Determine whether it was photographed off a screen | ⛔ **Do not wait to "gather more information" first.** ❌ Do not let the affected person keep using the account "while we investigate" — that account **is** the evidence |
| **Suspected unauthorised resource access** — material read, indexed, extracted or displayed without entitlement | 🔴 **Remove the content from every derived store immediately** — the index, any extracted text, any cache. ⛔ **Not only the source file** (`CL-DATA-020`). Narrow the entitlement set that granted it | Identify and delete every index entry derived from the material. Review the permission model against `CL-SEC-007` — this almost always indicates a divergence from the NAS's model. Determine what was displayed, and on which screens | ⛔ **Do not delete only the source file and call it done.** ⚠️ The index is the durable copy and the hard one to reach |
| **Suspected unauthorised update or deployment** | 🔴 **Disconnect displays from the update source and stop distribution.** Verify each display's signature state and running version independently | Determine what was deployed, when, by whom and what it did. Assess whether the signing key was used or copied. ⛔ **Assume the key is compromised and rotate it** — a key that shipped an unauthorised build has demonstrated it can | ⛔ **Do not roll forward to a "cleaner" build before establishing what the unauthorised one did.** ⛔ A good signature does not mean a good build: if the key was stolen, the signature is **valid** |

### 10.3 Evidence preservation

| Rule | Detail |
|---|---|
| **Photograph first** | If a screen shows something it should not, photograph it before touching anything. Cost: ten seconds. Value: the whole incident |
| **Note the time** | School clocks differ. Record what the screen shows *and* the real time |
| **Export the log before remediation** | ⚠️ Especially relevant after T18, because remediation is exactly when logs get truncated |
| **Do not "fix it and move on"** | ⚠️ A quiet fix destroys the evidence and leaves the route open |
| **Keep logs out of the narrative** | ❓ `CL-OPS-011` governs what may appear about staff. An incident report containing personal data needs the same care as the data |
| **Keep the artefact** | A malicious document (T08) or a rogue update (T12) must be preserved intact, not deleted on the way to "cleaning" |

### 10.4 The named contact question

| # | Question | Status | Why it is first |
|---|---|---|---|
| ❓ | **Who is the school's named contact for a security incident involving CloudLight, and outside school hours what is the expected response time before escalation?** | **UNANSWERED** — `CL-SEC-026` (P2) | ⚠️ **Without a named person and a clock, incidents are discovered by users rather than reported.** The first and most valuable hours are lost to *not knowing who to call*, not to not knowing what to do. This document cannot fill that gap and ❓ must not be filled with a plausible default |

### 10.5 ⚠️ The school's decision outranks this document

> ### ⚠️ **This is research documentation from a project team, not the school's security policy.**
>
> **When the school's own incident-response procedure conflicts with anything in this section, the school's procedure governs — without exception and without discussion.** CloudLight's designers are not the school's data-protection owners, its incident-response owners, or its accountable officers. Where §10 conflicts with the school's written procedure, **the school is right and this document is wrong**, and this document should be corrected to match.

⚠️ **The same applies to §7 and to every accepted risk in §13.** A risk accepted
here has not been accepted by the school.

**Mapped to:** `CL-SEC-012` (P2) · `CL-SEC-026` (P2) · `CL-SEC-019` ·
`CL-DATA-020` · `CL-OTA-004`.

---

## 11. SHANA integration security

⚠️ **Summary only.** The full boundary is in `shana_integration.md`.
**No interface has been agreed with SHANA's authors, no schema validated, no
joint test session held.**

| # | Property | Security consequence |
|---|---|---|
| **1** | ✅ **P0 — mutual authentication.** CloudLight authenticates to SHANA and vice versa: mutual TLS, or a signed request token with short expiry | ⛔ **An unauthenticated task endpoint lets any host on the network create robot tasks** — any device with a browser is a client. ⛔ Not a shared static key alone: single-key systems are single points of compromise. `CL-SHANA-015`, `CL-SEC-016` |
| **2** | ✅ **P0 — idempotency on every request** | ⚠️ **Prevents a replayed or retried request creating a duplicate physical task.** Two tasks from one tap is a *physical safety event*: a second machine moving through a corridor with a load nobody asked for. `CL-SEC-016`, `CL-SHANA-018` |
| **3** | ⛔ **No motor-command path anywhere in CloudLight** — any mode, including debug, admin, exhibition and "just for testing" | This is what makes everything above **bounded**. Without it, a software defect or an intrusion becomes physical motion in an occupied corridor. An **absence of capability**, not a policy |
| **4** | ✅ **Requests persisted before transmission** | If SHANA never receives the request the record still exists. Reconciliation on restart by `request_id`, not re-send |
| **5** | ✅ **`UNKNOWN` never renders as `IN_PROGRESS`** | A stale "On the way" causes a teacher to wait for a robot that already stopped (`CL-SEC-017`) |
| **6** | ✅ **CloudLight operates fully when SHANA is offline** | ⛔ A robot failure must never blank a classroom screen. ⚠️ **The dependency arrow points from CloudLight to SHANA, and it is optional** |
| **7** | ✅ **No raw robot video ever leaves the robot; no person is ever identified** | `../safety/student_privacy.md`. Inherited from SHANA's own decisions — not CloudLight's to relax |

### 11.1 ⚠️ What a P0 control does and does not buy

| The controls buy | The controls do **not** buy |
|---|---|
| A request provably came from *this school's* CloudLight | ⛔ **That a human authorised it.** Mutual authentication proves *which system*, not *whose intent* |
| No duplicate physical task from a replayed request | ⛔ Protection against a *fresh, valid* request made by an authenticated but misused account |
| A bounded worst case — an unauthorised delivery request, not motion control | ⛔ Protection of the robot link itself, which is a separate segment (`../infrastructure/school_network.md`) |

⚠️ **Honest statement:** the second and third gaps are accepted, not solved. They
are recorded in §13 with the compensating controls that make them tolerable at
this scale.

---

## 12. Availability as a security property

> ### ⚠️ **A display that fails closed with an honest "data unavailable" is safer than one that shows stale data confidently.**

| Failure | The security consequence | The rule |
|---|---|---|
| **Server unreachable** | ⚠️ **Authentication enforcement may be removed.** If any surface becomes *more* accessible when the server is down, an outage has silently converted an authenticated system into an unauthenticated one | ⛔ **Fail closed.** Nothing becomes more accessible. The board returns to the read-only home state |
| **Board stuck on a cached privileged view** | ⚠️ **A disclosure with a long tail.** A cached staff surface facing a corridor for the next three periods | Cached content limited to PUBLIC class. ⛔ Cache the *home state*, never the last privileged screen |
| **Stale data shown as current** | A person acts on a false picture | ⚠️ **Every value carries its age.** Beyond the staleness threshold (`CL-SEC-018`), an honest unavailable state |
| **Stale robot status** | A teacher sends someone to wait for a delivery that will never arrive | ✅ `UNKNOWN` with an age. ⛔ Never rendered as `IN_PROGRESS` (`CL-SEC-017`) |
| **Assistant unavailable** | A teacher routes around it — or a stale answer is presented as live | ⛔ **The assistant never publishes to a display without deterministic validation.** An unavailable assistant says so; it does not guess |

### 12.1 ⚠️ A network outage is a security event

⚠️ **This framing is deliberate and easy to disagree with.** A network outage is
normally an availability problem. Here it is also an **authentication-control
event**, for three reasons:

| # | Why |
|---|---|
| 1 | **Every authorisation decision is made server-side.** If the server is unreachable, **no authorisation decision is being made** — and anything still rendering is rendering from a cache of an earlier decision |
| 2 | **The outage is the natural moment to try something** — a curious student with a free period and a board that looks frozen |
| 3 | ⚠️ **The honest failure mode is a board that says "unavailable."** A worse *user experience* than a confident stale number, and a better *security property*. T19 is rated 🟠 10 rather than higher precisely because the failure mode is designed, not accidental |

⚠️ **Availability and security pull against each other here, and the design
resolves the tension in favour of the honest blank.** ⚠️ **That trade is a
product decision, and the school must agree with it** (`CL-SEC-021`, `CL-SEC-022`).

**Mapped to:** `CL-SEC-021` · `CL-SEC-022` · `CL-SEC-018` · `CL-SEC-017` ·
`CL-WIN-011` · `CL-DATA-013`.

---

## 13. Accepted risks

⚠️ **These are risks the design knowingly accepts. ⚠️ None has been accepted by
the school** — this document has no authority to accept anything on the school's
behalf. Each row requires a named person to sign it.

| # | Accepted risk | Compensating control | Eliminated? | Who must accept |
|---|---|---|---|---|
| **A1** | ⚠️ **A physical classroom screen is visible to whoever stands in front of it.** A student will see whatever the signed-out state shows | PUBLIC-class signed-out home state only; no staff surface without authentication; every value carries its age | ❌ **No.** The requirement is to decide *what* is visible | School administration — `CL-SEC-001` |
| **A2** | ⚠️ **A camera in the room can photograph any screen**, including one showing a staff name or a confidential note | No confidential content on student-facing surfaces; no identifying personal data on PUBLIC-class surfaces | ❌ **No.** ⛔ Not technically eliminable in a room with a door | School administration |
| **A3** | ⚠️ **Everything is on one host** — database, index, audit log, secrets, signing key. No second system holds a copy | Backups encrypted and off-host; default-deny egress; separated admin surface; tamper-evident logs with an off-host copy | ❌ **No.** ⚠️ Concentrated **by design** — that is what local-first means | School administration + IT |
| **A4** | ⚠️ **If the school runs a flat network, a compromised display is one hop from the control plane** | Strong application-layer authentication everywhere; separated administrative surface; default-deny egress; server-mediated NAS access | ❌ **Not eliminable without segmentation.** ⛔ **Must be accepted explicitly and in writing** | ⛔ **School IT + Principal** — `CL-SEC-023`, `CL-NET-001` |
| **A5** | ⚠️ **An administrator with full control of the host can defeat software-only log tampering** | Tamper-evident chaining; ⛔ off-host copy the administrator cannot reach; audit of administrative actions | ❌ **No.** ⚠️ A real limit, not a solved problem | School administration — `CL-SEC-019` |
| **A6** | ⚠️ **Mutual authentication proves which system called, not that a person authorised the delivery** | Request persisted before transmission; bounded request vocabulary; ⛔ no motor-command path anywhere; SHANA's independent validation may reject any request | ❌ **No.** Accepted because the alternative is worse | School administration + robot custodian — `CL-SHANA-015` |
| **A7** | ⚠️ **The search index is a copy of restricted content that outlives the permission check on the file** | Index at the entitlement level, filtered at query time; CONFIDENTIAL/RESTRICTED excluded by default; entries deleted with the source | ❌ **No.** ⚠️ A copy is a new thing to protect | School administration + IT — `CL-SEC-007`, `CL-DATA-020` |
| **A8** | ⚠️ **Staff share passwords**, and a design assuming they do not fails on the first real day | Individual accounts for consequential actions only; shared access for viewing, where the loss is negligible; audit distinguishes the two | ❌ **No.** Accepted deliberately, with scope | School administration — `CL-DATA-004` |
| **A9** | ⚠️ **The signing key is a single high-value secret** whose compromise is indistinguishable from a legitimate deployment | Client-side verification; documented key custody; rehearsed rotation; version recorded on every display | ❌ **No.** ⚠️ Hardware custody is an operational discipline, not a code property | Repository owner + school administration — `CL-SEC-013` |
| **A10** | ⚠️ **An offline-first system has no patch path unless one is deliberately built and exercised** | Documented, scheduled, tested update path; maintenance window (`CL-SRV-019`); internal CA if available (`CL-SEC-024`) | ⚌ **No.** ⚠️ Offline-first is a resilience property, **not a security posture** | School IT — `CL-SRV-019`, `CL-OPS-020` |
| **A11** | ⚠️ **Only one person may know how to run or restore the system** | Written runbooks; rehearsed restore; an off-site rotation dependency | ❌ **No.** ⚠️ Personal knowledge, not institutional knowledge | School administration — `CL-OPS-006`, `CL-OPS-015` |
| **A12** | ⚠️ **Prompt injection has no complete defence** in a retrieval-augmented system | Retrieved content treated strictly as data; deterministic validation before any display; the assistant never causes an action | ❌ **No.** ⚠️ Blast radius reduced; class of attack not eliminated | School administration — `CL-SEC-009` |

⚠️ **Twelve accepted risks, none eliminable.** That is the honest state of a
school-grade local-first platform in 2026, and a threat model claiming otherwise
would be worth less than none.

⚠️ **A4 and A5 most need a signature.** The first accepts that the control plane
may be one hop from a classroom. The second accepts that the person most in need
of watching cannot be fully watched by software. Both should be signed by someone
who can be asked about them in a year's time.

---

## 14. What is not decided

⚠️ **Every row is `UNANSWERED` in
`../../questionnaires/Additional_CloudLight_Questionnaire.md` as of 2026-10-11.**
✅ VERIFIED by reading the questionnaire: no answer has been supplied, and none
has been inferred here. **The right-hand column is empty, which is the point.**

| # | `CL-*` | Question | Pri | What it blocks | Who can answer | If unanswered |
|---|---|---|---|---|---|---|
| 1 | `CL-SEC-001` | Is a smartboard usable by any student, or does it fall back to an unauthenticated read-only home state? | **P0** | ⚠️ **The entire default attack surface** — T01 | School administration | Assume the restrictive reading: signed-out home state, PUBLIC class only |
| 2 | `CL-SEC-023` / `CL-NET-001` | **Does the school segment classroom devices from office, laboratory, CCTV and guest traffic today?** | **P0** | ⚠️ **T20.** Whether §7 is a configuration request or a construction project; A4 | School IT | ⛔ **No segmentation may be assumed.** §7.3 applies; A4 accepted |
| 3 | `CL-SEC-007` | Must CloudLight enforce the NAS's permissions, or hold its own copy of the access rules? | **P0** | ⚠️ **T06, T07, A7.** The whole permission model | IT + vendor | Enforce the NAS's. ⛔ A parallel model is not a safe default |
| 4 | `CL-SEC-008` | Must files be scanned and opened in an isolated viewer, or are staff uploads trusted? | **P0** | T08 | IT | ⛔ **Not trusted.** Isolated viewer, sandboxed extraction |
| 5 | `CL-SEC-019` | Must the activity log be append-only with tamper evidence, and how long preserved? | **P0** | T18, A5, §10 | IT | Chained integrity **and** an off-host copy |
| 6 | `CL-DATA-008` / `CL-OPS-012` | How often is the database backed up, where, who holds it — and what is backed up today? | **P0** | §9 | IT + vendor | Daily, off-host, encrypted, with a named holder. Assume nothing is currently backed up |
| 7 | `CL-DATA-010` | Will backups contain the same sensitive material, and must they be encrypted? | **P0** | §9 B2 | School administration | ✅ **Yes, and yes.** Non-negotiable |
| 8 | `CL-OPS-014` | Has any school system backup **ever been restored** from, and by whom? | **P0** | ⚠️ **§9.4. Whether recovery is a capability or a document** | IT | Assume it has **not**. Schedule the first test |
| 9 | `CL-SEC-003` | How many failed six-digit attempts before lockout, and for how long? | **P0** | T04 | IT | Rate-limit, short expiry, single use — **and §6 control 6 regardless** |
| 10 | `CL-SHANA-015` / `CL-SEC-016` | What identity must accompany a delivery request, and what makes it non-replayable? | **P0** | T15, T16, A6 | Robot custodian + vendor | ⛔ **No integration.** Build without SHANA; add it last |
| 11 | `CL-DATA-004` | Individual accounts for every teacher, or is shared sign-in acceptable anywhere? | **P0** | ⚠️ **§6 control 1.** The audit trail | School administration | Individual accounts for everything consequential |
| 12 | `CL-SEC-010` | Which information may the assistant include on a classroom display, and which is prohibited? | **P0** | T09, T10 | School administration | ⛔ **No student names, marks, attendance, behaviour or photographs** on any display |
| 13 | `CL-SEC-002` | How long may a session remain valid on a shared smartboard, and must it end at the period change? | **P0** | T01, T03 | School administration | Short idle **and** absolute timeouts; re-auth at period change |
| 14 | `CL-GIT-014` / `CL-GIT-015` | What is our secrets policy, and where do school credentials live at runtime? | **P0** / P1 | ⚠️ **T05, S1, S9** | IT | ⛔ **The §5 rules apply immediately, without waiting for an answer** |
| 15 | `CL-SEC-011` | May any question, document excerpt or school record leave school infrastructure for an external model? | P1 | T10, B4 | School administration | ⛔ **Prohibited.** Egress default-deny |
| 16 | `CL-SEC-013` | Which control stops someone without signing authority shipping a change — client-side verification, a deployment credential, or both? | P1 | T12, A9 | IT | ✅ **Client-side verification.** A credential alone is not sufficient |
| 17 | `CL-SEC-018` | How old may data be on a classroom screen before this school considers it misleading rather than late? | P1 | T17, §12 | School administration | A visible age on everything; an honest unavailable state beyond the threshold |
| 18 | `CL-SEC-009` | Is any document expected to be **obeyed as instruction** by the assistant? | P1 | T09, A12 | School administration | ⛔ **No.** Every document is data. If yes, name it explicitly |
| 19 | `CL-SEC-025` / `CL-DATA-015` | Retention per category, and which must be deletable on request? | P1 | §4.2, §8.4 | School administration | Shortest workable period; **AI conversation history deleted first** |
| 20 | `CL-SEC-024` | Is HTTPS required inside the LAN, and can the school issue an internal certificate? | P1 | B1, §2 | IT | ✅ **TLS regardless.** An internal CA removes the offline-expiry failure mode |
| 21 | `CL-DATA-020` | When a document is removed from the NAS, must its index entries and extracted text also be deleted, and how quickly? | P2 | ⚠️ **T07, §10.2.** The derivation leak | IT | ⛔ **Immediately.** Deleting only the source is not deletion |
| 22 | `CL-SEC-020` / `CL-OPS-011` | Which actions must appear in the log, is any of it visible to teaching staff, and what may logs contain about students and staff? | P2 | §8.1–§8.2, T18 | School administration | Log the §8.1 set; anonymised identifiers; ⛔ **not visible to all staff** |
| 23 | `CL-OPS-010` | Are there existing log-retention rules, and how long must access records be kept? | P2 | §8.4 | School administration | Bounded rotation by size; ⛔ not unbounded growth |
| 24 | `CL-SEC-012` | If the host is suspected compromised, what is the school's priority order — revoke, cut, or preserve? | P2 | §10.2 | IT | Contain, and document the loss of evidence |
| 25 | `CL-SEC-026` | **Who is the named contact for a security incident**, and outside hours, what response time before escalation? | P2 | ⚠️ **§10.4. The first and most valuable hours** | School administration | ❌ **Cannot be defaulted.** Escalate to the Principal by any available route |
| 26 | `CL-SEC-021` / `CL-SEC-022` | During a full outage, what must still work? Is manual fallback acceptable, and after how many hours is it critical? | P2 | T19, §12 | School administration | Fail closed to the read-only home state |

⚠️ **Rows 1, 2, 3 and 11 decide what this system is.** All four are P0, all four
are policy questions rather than engineering questions, and all four are
unanswered.

⚠️ **Note what is conspicuously absent from the "who can answer" column.** Twenty
of the twenty-six answers must come from the school or its IT department, not
from the project team. ❓ **This document cannot proceed past Phase 0 without
them, and no engineering judgement substitutes for them.**

---

## Sources

**Types:** `OFFICIAL-STANDARD` (standards body) · `OFFICIAL-OPEN` (OWASP community project) · `INTERNAL` (this repository). **Access date for all external sources: 2026-10-11.**

| # | Title | URL | Type | What it supports |
|---|---|---|---|---|
| 1 | **Digital Identity Guidelines — Authentication and Authenticator Management (NIST SP 800-63B Rev. 4)** | `https://pages.nist.gov/800-63-4/sp800-63b.html` | OFFICIAL-STANDARD | T03, T04; §6 controls 3, 7. ✅ **Both an overall reauthentication timeout and an inactivity timeout are required, not one or the other** — AAL1: an inactivity timeout *"MAY be applied but is not required"*; AAL2: overall ≤ 24 h, inactivity ≤ 1 h. Rate-limiting is a **SHALL** at every AAL: verifiers *"SHALL implement a rate-limiting mechanism that effectively limits the number of failed authentication attempts"*. Passwords used as single-factor *"SHALL be a minimum of 15 characters"*, and verifiers *"SHALL NOT require subscribers to change passwords periodically"* while *"SHALL force a change if there is evidence that the authenticator has been compromised"* — the rotation rule in S10. ✅ The abstract states the guidelines *"are not intended to constrain the development or use of standards outside of this purpose"*, supporting §6 control 12 |
| 2 | **Session Management Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html` | OFFICIAL-OPEN | §6 controls 2, 3, 5 — `httpOnly`, `Secure` and `SameSite` as baseline session-cookie attributes; idle **and** absolute timeout; revocation effective immediately rather than at next login (T03, `CL-SEC-004`) |
| 3 | **Authentication Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html` | OFFICIAL-OPEN | §6 controls 3, 7, 11 — MFA as defence in depth, login throttling, reauthentication after a privilege-elevation event (T01, T13, T14, T18) |
| 4 | **Authorization Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html` | OFFICIAL-OPEN | §6 controls 8, 9 — ⚠️ *"Never rely on client-side access control checks"*; validate permissions on every request; deny by default with a single server-side default-deny policy. Supports T06, T07 |
| 5 | **Secrets Management Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html` | OFFICIAL-OPEN | §5 S1, S9, S10 — secrets supplied at runtime, never in source control; the *"rotate, never reuse"* lifecycle principle; and the central point that **a secret in version control must be assumed compromised** (T05, `CL-GIT-014`) |
| 6 | **Logging Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html` | OFFICIAL-OPEN | §8.2 — ✅ *"Data should not normally be logged"*, with credentials among the excluded items; log-injection prevention; log access is itself access-controlled data (T18, `CL-SEC-019`, `CL-OPS-011`) |
| 7 | **File Upload Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html` | OFFICIAL-OPEN | T08 — defence in depth: extension and content-type allow-listing, filename randomisation, size limits, and ✅ **store uploads outside the webroot and never execute or interpret them server-side**. The pattern for sandboxed, non-executing extraction (`CL-SEC-008`) |
| 8 | **Code Signing Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Code_Signing_Cheat_Sheet.html` | OFFICIAL-OPEN | T12, A9 — ✅ **the signature must be verified by the software consuming the artefact**, not by the distribution point; the private key is the crown jewel; key compromise means every artefact is suspect (`CL-SEC-013`) |
| 9 | **LLM Prompt Injection Prevention Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html` | OFFICIAL-OPEN | T09, A12 — retrieved content treated as **data, not instruction**; segregating instructions from retrieved content; and OWASP's own framing that **no mitigation is complete**, which is why §13 records A12 as accepted rather than solved |
| 10 | **Incident Response Recommendations and Considerations for Cybersecurity Risk Management: A CSF 2.0 Community Profile (NIST SP 800-61 Rev. 3)** | `https://csrc.nist.gov/pubs/sp/800/61/r3/final` | OFFICIAL-STANDARD | §10 — incident-response preparation integrated with organisational risk management, and the importance of a defined, named response capability. ✅ Published April 2025; supersedes Rev. 2 (2012) |
| 11 | **`docs/safety/risk_register.md`** | *(this repository — `../safety/risk_register.md`)* | INTERNAL | ⚠️ **The rating convention this document follows**: `L` × `S` both 1–5; 🔴 High 15–25, 🟠 Medium 8–14, 🟡 Low 1–7; and the status vocabulary `MITIGATION PLANNED` / `OPEN — APPROVAL REQUIRED` / `OPEN — ANSWER REQUIRED` / `MONITOR` |
| 12 | **`start_opencode_server.sh`** and **`.gitignore`** | *(repository root)* | INTERNAL | T05, §5 S1 — ✅ **VERIFIED:** the repository-root script contains a hardcoded plaintext password and binds `0.0.0.0`. ⚠️ **Correction:** it *is* listed in `.gitignore`, so it is not committed; the exposure is local. No document here quotes the credential value |
| 13 | **`docs/safety/student_privacy.md`** | *(this repository — `../safety/student_privacy.md`)* | INTERNAL | §4 ⛔ NEVER STORED row; §8.4 ⛔ no per-student activity tracking; §11 control 7 — no student is ever identified; no raw video retention |
| 14 | **`docs/infrastructure/school_network.md`** | *(this repository — `../infrastructure/school_network.md`)* | INTERNAL | §7.1, §11 — the robot link is a **separate wired Ethernet segment** that must not share a broadcast domain with the classroom LAN; and the scan/probe prohibition that makes every network fact a question rather than a finding |
| 15 | **`docs/architecture/architecture.md`** and `architecture_decisions/` | *(this repository — `../architecture/`)* | INTERNAL | §2 B3, §11 control 3 — the no-AI-to-the-robot invariant, the deterministic-validator rule, the local-server principle |
| 16 | **`docs/cloudlight/network_and_hosting.md`** | *(this repository — `network_and_hosting.md`)* | INTERNAL | §7 segmentation request; §9 backup and UPS; §11 the concentrated host; §12 default-deny egress. ⚠️ **Its §6.1 states the same core principle: "A school LAN must NOT be assumed trusted"** |
| 17 | **`docs/cloudlight/shana_integration.md`** | *(this repository — `shana_integration.md`)* | INTERNAL | §11 — mutual authentication and idempotency as P0; `UNKNOWN` never rendering as `IN_PROGRESS`; the ten invariants; and the recommendation to build without SHANA and add it last |
| 18 | **`questionnaires/Additional_CloudLight_Questionnaire.md`** | *(this repository — `../../questionnaires/Additional_CloudLight_Questionnaire.md`)* | INTERNAL | ⚠️ **Every `CL-*` identifier cited here**, with priority, respondent, dependency and evidence requirement. ✅ VERIFIED as of 2026-10-11: **all twenty-six questions in §14 are recorded as `UNANSWERED`** |

---

⚠️ **Standing note.** **Tests actually executed to date: zero.** No control here has been implemented, configured, deployed or verified. No school network has been accessed or probed. No credential was created, attempted or rotated. No log was written and no incident has occurred, because no system exists.

**Every ❓ UNKNOWN here is a genuine gap, not an omission.** Filling one with a plausible default is the failure mode this document exists to prevent.

**And the last word is not ours.** §10.5, §13 and §14 are where the school decides, not the project team. Where this document and the school's written policy disagree, **the school is right and this document is wrong.**