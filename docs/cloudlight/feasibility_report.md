# CloudLight — Feasibility Report

**Phase 0 · Documentation only · 2026-10-11**

This report answers the questions posed for CloudLight, and **classifies every
conclusion by how well it is actually established.**

⚠️ **Nothing here has been verified on school hardware.** No smartboard was
touched, no network probed, no NAS accessed, no robot operated, no AI service
contacted. Where a conclusion rests on reasoning rather than evidence, it says so.

---

## How to read the classifications

| Label | Meaning |
|---|---|
| **VERIFIED** | Confirmed in official vendor, standards or regulatory documentation, quoted with a source |
| **DOCUMENTED_NOT_HARDWARE_VERIFIED** | A vendor documents it; not tested on this school's equipment |
| **PLAUSIBLE_REQUIRES_TESTING** | Engineering judgement; no vendor claim; needs a test |
| **UNKNOWN** | No evidence found. Not assumed either way |
| **REQUIRES_SCHOOL_PERMISSION** | Blocked until a named school authority approves in writing |
| **REQUIRES_VENDOR_OR_SERVICE_PROVIDER** | Blocked until a vendor or provider confirms |
| **NOT_RECOMMENDED_WITHOUT_APPROVAL** | We will not implement this without an explicit recorded decision |

⚠️ **A document labelled PLAUSIBLE or UNKNOWN is not a weak VERIFIED.** It is an
open question, and it must not be cited as a finding.

---

## 1. What should CloudLight be?

> **A local-first, school-hosted classroom information platform, delivered as a
> modular monolith, browser-first, human-gated at every point where information
> reaches a screen.**

| Component | Conclusion | Classification |
|---|---|---|
| Form: local-first platform on the school's own network | Correct target. It keeps school data inside the school and removes internet dependency from core operation | **VERIFIED** (feasibility of local-only operation is a design property, not a claim) |
| Form: modular monolith, not microservices | Correct at this scale. One school, a few dozen displays, no operations staff. Microservices solve an organisational scaling problem this project does not have | **PLAUSIBLE_REQUIRES_TESTING** — sizing argument is sound; confirm device count (`CL-WIN-020`) |
| Form: browser-first rather than a packaged desktop client | Correct. A browser kiosk takes updates from the server; an installed client needs an installer and an update path on every classroom machine | **DOCUMENTED_NOT_HARDWARE_VERIFIED** — Windows kiosk behaviour is documented; untested here |
| Form: **not** microservices, **not** an access-point-hosted service, **not** a smartboard-as-server | All three rejected with reasons | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| Every capability deferred that has no confirmed precondition | Correct sequencing | **PLAUSIBLE_REQUIRES_TESTING** |

⚠️ **The scope recommendation: nine of sixteen proposed capabilities are not in
the first release.** Four of those are deferred not because they are hard but
because **the thing they depend on has not been confirmed to exist** — shared
storage, an identity directory, an always-on host, and AI policy.

---

## 2. How should CloudLight and SHANA be separated?

> **CloudLight may request a delivery. It may never command the robot.**

| Concern | Owner | Classification |
|---|---|---|
| CloudLight never sends motor, path, speed or actuator commands | **SHANA boundary is absolute** | **NOT_RECOMMENDED_WITHOUT_APPROVAL** — a non-negotiable constraint |
| Navigation, localisation, route safety, battery reserve, hazards, safe stopping, emergency behaviour | **SHANA, entirely** | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| Teacher identity, request authorisation, request audit, status display | **CloudLight** | **PLAUSIBLE_REQUIRES_TESTING** |
| Shared repository, single Git history, shared ADRs | Correct. CloudLight and SHANA stay in **this** repository | **VERIFIED** |

⚠️ **The boundary is expressed as an absence of capability, not as a policy.** A
low-level command path that exists "only for debugging" converts a software defect
into physical motion in an occupied corridor. The correct design is for CloudLight
to have **no way to express** a motor command.

**The separation is cleanest when SHANA has no visibility into CloudLight's
internals and vice versa** — which is why the SHANA-facing document
([`shana_integration.md`](shana_integration.md)) is written from CloudLight's side
as a *proposal*, and explicitly does not claim SHANA has agreed to it.

---

## 3. How should they communicate?

> **CloudLight sends an authorised, idempotent, high-level request and receives
> status only. SHANA may always reject.**

| Property | Requirement | Classification |
|---|---|---|
| Direction | CloudLight → SHANA: request. SHANA → CloudLight: status. **No command channel in reverse** | **PLAUSIBLE_REQUIRES_TESTING** |
| Authentication | Mutual authentication. **Without it, any host on the network can move the robot** | **NOT_RECOMMENDED_WITHOUT_APPROVAL** — P0 control |
| Idempotency | Client-generated request ID; a retry is the same request; no duplicate physical task | **NOT_RECOMMENDED_WITHOUT_APPROVAL** — duplicate dispatch is a physical event |
| SHANA's authority | Acceptance is not a right. Rejection is a normal outcome | **PLAUSIBLE_REQUIRES_TESTING** |
| Versioning | Versioned messages; graceful degradation on change | **PLAUSIBLE_REQUIRES_TESTING** |
| Reconciliation | On restart, recover in-flight tasks by request ID — never re-send as new | **PLAUSIBLE_REQUIRES_TESTING** |
| Actual interface | **Unknown.** Depends entirely on what the robot vendor permits | **REQUIRES_VENDOR_OR_SERVICE_PROVIDER** |

⚠️ **The interface itself is UNKNOWN.** `CL-SHANA-001` must be answered by the
robot custodian and the vendor before any integration work begins. Everything
above is CloudLight's *requirement*, not a description of what SHANA offers.

---

## 4. Where should the backend run?

> **On an always-on, school-owned machine the school has explicitly permitted to
> run a persistent service.**

| Option | Verdict | Classification |
|---|---|---|
| Authorised always-on school computer | **Preferred** — cheapest, most supportable, most likely permitted | **REQUIRES_SCHOOL_PERMISSION** |
| Dedicated small server | Acceptable if no suitable computer exists; adds procurement | **REQUIRES_SCHOOL_PERMISSION** |
| NAS-hosted service | **Possible but platform-dependent.** Some NAS platforms can run containers and provide a reverse proxy with TLS termination. ⚠️ Must be confirmed against the actual NAS model and OS version, and raises a durability/supportability question | **UNKNOWN** — `CL-NAS-002` |
| A Windows smartboard as the server | ⛔ **Rejected.** Re-imaging a classroom board would take down every classroom | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| An access point or router as the server | ⛔ **Rejected** — see §6 | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| Cloud VM | Fallback only. Moves school data off school infrastructure and reintroduces internet dependency | **REQUIRES_SCHOOL_PERMISSION** |

⚠️ **Whether a persistent local service is permitted at all is
`REQUIRES_SCHOOL_PERMISSION` and it blocks everything.** `CL-SRV-001`,
`CL-GEN-002`. If the answer is no, the entire local-first premise fails and this
becomes a different project.

---

## 5. Can the smartboards access that server?

> **Unknown. And it is the single most important unknown in this project.**

The chain that must hold is:

```
smartboard  →  resolves a name  →  reaches the host address  →  allowed by firewall
             →  TLS presented and trusted  →  backend responds
```

Each link fails independently. **A working ping proves none of the rest.**

| Factor | Question | Classification |
|---|---|---|
| Physical/Wi-Fi connectivity | Are boards wired, or on Wi-Fi? | **UNKNOWN** |
| Name resolution | Can boards resolve a local DNS name? | **UNKNOWN** |
| Addressing | Is DHCP managed? Can a stable address be reserved? | **UNKNOWN** |
| Client isolation | Is AP/client isolation enabled? ⚠️ Common on managed and guest networks — it silently blocks device-to-device traffic on the same SSID | **UNKNOWN** |
| Firewall | Do rules permit the display segment to reach the server? | **UNKNOWN** |
| Segmentation | Are classroom devices already separated from servers? | **UNKNOWN** |
| Hosting permission | May a persistent service run at all? | **REQUIRES_SCHOOL_PERMISSION** |
| Hardware | Is there a spare always-on machine with a free port and a UPS? | **UNKNOWN** |

⚠️ **If isolation is on, the design fails silently.** Not with an error — with a
screen that simply never loads. This is the most likely quiet failure in the whole
project and it is why `CL-NET-008` is a P0.

---

## 6. What does the access point actually provide?

> **Layer-2 connectivity only. It cannot host the platform.**

| Question | Answer | Classification |
|---|---|---|
| Can an access point run a web server or backend? | **No.** An AP is a bridge that forwards frames | **VERIFIED** |
| Can a router's "port forwarding" host a backend? | **No.** It forwards to a device that must already exist | **VERIFIED** |
| Do some enterprise APs offer application-like services? | Some offer captive-portal or authentication services. ⛔ **None of them is an application host** | **VERIFIED** |
| Does "all smartboards can reach the same AP" imply a backend can exist? | **No.** It establishes at most a shared Layer-2 segment | **VERIFIED** |
| Does it imply devices can reach each other? | **No — and isolation may prevent exactly that** | **VERIFIED** |

⚠️ **This corrects a premise in the brief.** The premise is not wrong, it is
incomplete: connectivity infrastructure and a compute host are different things,
and the first has been assumed while the second has not been confirmed.

---

## 7. What must be confirmed by school IT?

| # | Fact | Why it changes the design | Question |
|---|---|---|---|
| 1 | Are classroom devices reachable from an always-on host, and vice versa? | Without it there is no system | `CL-NET-003` |
| 2 | Is client/AP isolation enabled? | Silently breaks local hosting | `CL-NET-008` |
| 3 | Can a persistent service run on a school machine? | Without it the whole premise fails | `CL-SRV-001` |
| 4 | Is there a spare always-on machine with a spare network port and a UPS? | Determines where the backend runs | `CL-SRV-007`, `CL-SRV-003` |
| 5 | Can a local DNS name and stable address be created? | Maintainability | `CL-NET-006`, `CL-NET-012` |
| 6 | What firewall rules permit or block the path? | Determines the exact request to make | `CL-NET-014` |
| 7 | Is a NAS present, and what is it? | Determines whether resource features exist | `CL-NAS-001`, `CL-NAS-002` |
| 8 | Does that NAS support a supported integration API? | Determines whether indexing is viable | `CL-NAS-004` |
| 9 | What Windows edition and build is on each board? | Determines whether kiosk lockdown exists at all | `CL-WIN-001` |
| 10 | Do kiosk accounts have local administrator rights? | Determines whether self-start and self-update are possible | `CL-WIN-003` |
| 11 | Is Intune or any MDM in use? | Determines the whole configuration method | `CL-WIN-005` |
| 12 | Can one non-critical board be used for a kiosk test? | Cheapest way to discover feasibility | `CL-WIN-007` |
| 13 | Is TLS on the internal LAN required, and is a self-signed certificate acceptable? | Determines whether plain HTTP is permissible | `CL-SEC-024`, `CL-AUTH-010` |
| 14 | Is there a staff identity directory? | Determines the whole identity model | `CL-AUTH-001`, `CL-AUTH-002` |
| 15 | Is any external AI service permitted? | Determines whether an assistant can exist | `CL-AI-001` |
| 16 | Is automatic unattended deployment permitted? | Determines the update model | `CL-OTA-012` |

⚠️ **Items 1, 2 and 3 are the blockers.** Without all three, no pilot is possible.

---

## 8. How should regular timetable and substitution data be reconciled?

> **The authoritative source is defined by school policy. CloudLight is a consumer
> and renderer until the school says otherwise.**

| Rule | Classification |
|---|---|
| A record is authoritative because a person with authority approved it — not because it was photographed, parsed or extracted | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| CloudLight holds versions; a correction **supersedes** rather than edits a published record | **PLAUSIBLE_REQUIRES_TESTING** |
| Every publication is attributable to a named approver, with the source document retained | **REQUIRES_SCHOOL_PERMISSION** — retention has obligations |
| On disagreement between the school's file and CloudLight's copy, **the school's file wins** and a human reconciles | **PLAUSIBLE_REQUIRES_TESTING** |
| Who owns the master timetable | **UNKNOWN** — `CL-TIME-001` |
| Who approves a substitution | **UNKNOWN** — `CL-SUB-001` |
| Whether a second person must counter-sign | **UNKNOWN** — `CL-SUB-002` |
| Whether photographing boards is permitted at all | **UNKNOWN** — `CL-SUB-006` |

---

## 9. Which parts of timetable processing should be deterministic?

> **Everything that can be parsed deterministically. AI only where the source is
> genuinely unstructured.**

| Tier | When used | Model involved | Classification |
|---|---|---|---|
| 1. Deterministic structural parse (xlsx, CSV, PDF text layer) | **Default. Always tried first** | No | **VERIFIED** — these formats have stable, machine-readable structure |
| 2. Layout analysis | Only when the table has no reliable cell structure | Layout model | **PLAUSIBLE_REQUIRES_TESTING** |
| 3. OCR | Only for image-only or handwritten input | OCR engine | **PLAUSIBLE_REQUIRES_TESTING** — accuracy on handwriting is materially lower |
| 4. LLM field extraction | Only for ambiguous fields, into a **strict schema** | LLM | **PLAUSIBLE_REQUIRES_TESTING** |
| 5. Deterministic validation | **Always. The only tier permitted to reject** | No | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |

⚠️ **Three non-negotiable rules:**

1. **Deterministic parsing is tried first, always.** A model is never used to read
   a spreadsheet cell that a library reads exactly.
2. **AI never fills a missing value with a plausible guess.** Unknown is a valid
   output; a plausible invention is not.
3. **A model never publishes.** Publication requires a named human approver,
   recorded with the source document and the extracted values.

⛔ **AI-assisted extraction should not be in the first release.** It is the
highest-risk, lowest-value component in the pipeline, and it is gated on the
approval workflow existing at all.

---

## 10. Where could AI/ML help?

| Use | Verdict | Classification |
|---|---|---|
| Extracting fields from an ambiguous or scanned timetable | Plausible, **assistive only**, human-confirmed | **PLAUSIBLE_REQUIRES_TESTING** |
| Layout and table-structure recognition on structured PDFs | Plausible; cheaper than OCR | **PLAUSIBLE_REQUIRES_TESTING** |
| Confidence scoring to route documents to review | Valuable and low-risk | **PLAUSIBLE_REQUIRES_TESTING** |
| **Choosing a substitute teacher** | ⛔ **Never.** That is a human staffing decision | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| **Publishing any timetable change** | ⛔ **Never** | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| **Approving a substitution** | ⛔ **Never** | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| **Granting permissions** | ⛔ **Never** | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| **Issuing any robot command** | ⛔ **Never.** Inherited from SHANA's [ADR-001](../shana/architecture/architecture_decisions/ADR-001-no-ai-reaches-the-robot.md) (`D-005`) | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| Reading a photograph of a classroom board as authoritative | ⛔ **Never** | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |

⚠️ **The single highest-value AI use in the timetable pipeline is confidence
scoring — deciding which documents need human attention — not extraction.**
It carries far less risk for most of the benefit.

---

## 11. How should teachers authenticate?

> **Individual teacher accounts for anything consequential. A six-digit code may
> only ever authorise one action or act as a second factor.**

| Question | Answer | Classification |
|---|---|---|
| Is an individual account sufficient? | **No, not by itself** for the highest-risk actions | — |
| Preferred identity source | The school's own directory via SSO, **if one exists and permission is granted** | **REQUIRES_SCHOOL_PERMISSION**; existence is **UNKNOWN** (`CL-AUTH-002`) |
| If no directory exists | CloudLight owns accounts — and therefore owns provisioning and offboarding | **PLAUSIBLE_REQUIRES_TESTING** |
| Second factor | Passkeys/WebAuthn preferred; TOTP acceptable; ⛔ SMS is a *restricted* authenticator under NIST SP 800-63B (SS7 interception, SIM-swap, number-porting) | **VERIFIED** |
| Sessions | `httpOnly`/`Secure`/`SameSite=Strict`; idle **and** absolute timeouts; server-side revocation; ⛔ never in local storage | **VERIFIED** (session-management best practice) |
| May an unattended display hold a teacher session? | ⛔ **Recommend no.** A persistent session on a shared panel means anyone in the room inherits teacher authority | **REQUIRES_SCHOOL_PERMISSION** — the school decides (`CL-AUTH-020`) |
| Is a shared six-digit code sufficient to prove a teacher? | ⛔ **No.** See §12 | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |

⚠️ **Recovery is a universal bypass if it is weak.** It must be at least as
strong as the factor it replaces. This is routinely the weakest link and is called
out explicitly in [`authentication_and_authorisation.md`](authentication_and_authorisation.md).

---

## 12. Is a six-digit code sufficient for the proposed action?

> **No. It is not sufficient as identity. It may be sufficient as a
> one-action authorisation or as a second factor.**

| Consider | Finding |
|---|---|
| Guessing space | 10⁶ — brute-forceable in a free period without attempt limits |
| Visibility | A classroom display is shared and unattended; any code shown can be photographed |
| Attribution | A shared code identifies nobody, so an unauthorised delivery request would be unattributable |
| Audit | Cannot answer "which teacher did this" |
| Revocation | Cannot revoke one person without revoking everyone |
| **Permitted uses** | Authorising **one** specific action; second factor behind a real first factor |
| **Never permitted** | Authenticating a named teacher; standing in for individual accounts |
| Required controls | Short lifetime; single use; attempt limits and lockout; rate limiting on resend; never displayed after the teacher looks away; never in a URL, log, browser storage, config or Git |
| **Server-side** | ⛔ **Must be a P0 control.** Not a usability convenience |

⚠️ **The specific request — a teacher asking a robot to move something through a
corridor — is exactly the action that must not rest on a shared code.** Individual
authentication, plus a code only if the school decides a second factor is
acceptable.

---

## 13. How can authorised NAS resources be searched?

> **Permission-filtered at query time, and re-verified at open time.**

| Layer | Requirement | Classification |
|---|---|---|
| Index | Path, metadata, and — only if authorised — extracted text, by **reference**, without copying originals | **PLAUSIBLE_REQUIRES_TESTING** |
| Crawl account | ⛔ **Must be a dedicated read-only service account.** Indexing under a teacher's credentials makes every result masquerade as that teacher's access | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| Filtering | Evaluate the user's permissions at **query time**, not index time | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| Re-verification | ⛔ **Re-check permission at open time against the storage layer.** Index entries outlive permission changes; a revocation must take effect immediately | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| Enforcement | CloudLight enforces the storage's own rules; it holds no parallel permission model that can drift | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| Confidential material | Exclusion decided **before** indexing, not discovered afterwards | **REQUIRES_SCHOOL_PERMISSION** |
| Existence of storage | **Unknown.** A school may have no shared storage at all | **UNKNOWN** (`CL-NAS-001`) |
| Corpus size | **Unknown.** Determines the search engine | **UNKNOWN** (`CL-SEARCH-014`) |
| Identity integration | Permission-aware search generally needs the same directory the storage authenticates against | **REQUIRES_SCHOOL_PERMISSION** |

⚠️ **Both failure directions are wrong.** An index built on an over-broad
principal returns files the user could not browse to. An index that ignores a
principal the file depends on hides files the user can open. Both must be tested.

---

## 14. Can resource search preserve storage permissions?

> **Yes, in principle — but only with directory integration, and it must be tested
> in both directions.**

| Finding | Classification |
|---|---|
| The mechanism is established: record each file's ACL principals during crawl; match them against the signed-in user's group memberships at query time; honour DENY over ALLOW; re-verify at open | **VERIFIED** as a technique (implemented by production permission-aware search products) |
| It requires the searcher to know the user's group memberships, which in turn requires directory integration | **VERIFIED** |
| Path-dependent access is harder than ACL-based access: a user may reach a file through one share but not another | **PLAUSIBLE_REQUIRES_TESTING** |
| Stale ACLs are a real exposure window: a file's ACL is captured at index time, so a mid-session ACL change may not be reflected until re-index | **PLAUSIBLE_REQUIRES_TESTING** |
| **Can the frontend ever exceed storage permissions?** | ⛔ **No — and this is the design's central invariant.** Search never widens access; it can only narrow it |

⚠️ **The answer is "yes, with conditions, and unverified here."** It cannot be
confirmed as feasible in this school until the storage system, its permission
model and the identity directory are known.

---

## 15. Which AI assistant deployment options are feasible?

> **Local inference if at all; no external service without written authorisation.**

| Option | Data leaves school? | Feasibility | Classification |
|---|---|---|---|
| Local model on a school host | No | Feasible for grounded short answers over a modest corpus, if the hardware allows. ⚠️ Hardware is unknown | **PLAUSIBLE_REQUIRES_TESTING** — `CL-AI-009` |
| Local embeddings + external generation | Document text leaves | Usually **not acceptable** — the document content is the sensitive part | **REQUIRES_SCHOOL_PERMISSION** |
| Fully external API | Yes | ⛔ **Not a default.** Permitted only in writing, with verified no-retention configuration | **REQUIRES_SCHOOL_PERMISSION** — `CL-AI-001` |
| **No assistant at all** | No | ⛔ **A legitimate outcome.** Value depends entirely on retrieval quality, which is deferred | **REQUIRES_SCHOOL_PERMISSION** |

⚠️ **Local inference has a hard prerequisite that is easy to miss: a permanently
offline school cannot download model weights.** Every binary, model and tokeniser
must be fetched on an internet-connected machine and transferred across the air
gap in advance. A missed file breaks setup silently.

⚠️ **Recommendation: build the assistant last, or never.** Its entire value
depends on permission-aware retrieval working correctly, which is itself deferred.
It is the largest data-protection surface in the product and the least certain to
be approved.

---

## 16. How should Windows startup work?

> **A managed kiosk browser — but this is gated on the Windows edition, which is
> unknown.**

| Finding | Classification |
|---|---|
| Microsoft Edge kiosk mode with Assigned Access (single-app kiosk) or Shell Launcher is the documented path to a locked-down fullscreen display | **VERIFIED** (Microsoft documentation) |
| Assigned Access support depends on Windows edition; Shell Launcher is Enterprise/Education; the two cannot be combined on one machine | **VERIFIED** |
| A Windows Service **cannot display UI** and therefore cannot be the kiosk launcher | **VERIFIED** |
| `--kiosk-idle-timeout-minutes` resets the session but does **not** relaunch Edge — a separate mechanism is required | **VERIFIED** |
| The `LogonUI` `IdleTimeOut` registry value is explicitly documented as **not** applying to Edge kiosk mode | **VERIFIED** |
| The assigned-access resume timeout defaults to 30 seconds and is separately configurable | **VERIFIED** |
| **What edition is actually installed on these boards?** | **UNKNOWN** — `CL-WIN-001` |
| **Does a durable offline cache work inside an Edge kiosk InPrivate session?** | **UNKNOWN** — flagged as the highest-risk unknown in [`windows_startup.md`](windows_startup.md); version-dependent behaviour |
| Is admin rights available on kiosk accounts? | **UNKNOWN** — `CL-WIN-003` |

⚠️ **If the boards run an edition without Assigned Access or Shell Launcher, the
entire kiosk recommendation collapses** and the fallback is a browser opened by
hand — which is not an acceptable daily experience.

⚠️ **The boot race is real and is not fixed by a browser flag.** The machine
frequently boots faster than the network comes up. The fix is an
**application-level offline state** showing cached data with a visible staleness
marker.

---

## 17. How should OTA updates and rollback work?

> **Eight separate channels, each signed, each staged, each rollback-able. Robot
> firmware is not one of them.**

| Finding | Classification |
|---|---|
| "OTA" is not one thing: frontend bundle, backend, kiosk config, timetable data, search index, AI model, SHANA integration, and **separately** robot firmware | **VERIFIED** as a design consequence |
| A school LAN is not trusted, so a smartboard must accept an update **only if signed** | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| A naive "check the signature" design resists none of rollback, freeze, fast-forward or mix-and-match attacks; The Update Framework (TUF) is the published reference design for resisting them | **VERIFIED** (TUF specification) |
| Data corrections must not be blocked behind a software-release approval — separate promotion paths | **PLAUSIBLE_REQUIRES_TESTING** |
| Rollback must be **rehearsed**, not intended | **PLAUSIBLE_REQUIRES_TESTING** |
| Rollout must start on one nominated board for a full school day | **PLAUSIBLE_REQUIRES_TESTING** |
| A permanently offline system cannot receive security patches | **VERIFIED** — a genuine tension with "local-first" |
| **Can kiosk accounts install an updating agent?** | **UNKNOWN** — `CL-OTA-010` |
| **Is automatic unattended deployment permitted?** | **UNKNOWN** — `CL-OTA-012` |
| **Where does the signing key live?** | **REQUIRES_SCHOOL_PERMISSION** — `CL-OTA-005` |

⚠️ **Robot firmware flashing is a separate, higher-risk physical activity.
CloudLight must never be its delivery path.** Conflating a web bundle refresh
with firmware flashing is a category error with safety consequences.

---

## 18. How should SHANA remain safe when CloudLight is unavailable?

> **CloudLight must not be in SHANA's safety path. If CloudLight is down, no
> safety function may be affected.**

| Requirement | Classification |
|---|---|
| SHANA's behaviour when CloudLight is unreachable is **SHANA's decision**, not CloudLight's specification | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| CloudLight must not create pressure on SHANA to degrade safety to keep a request moving | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| No AI model output may reach the robot — inherited from SHANA's [ADR-001](../shana/architecture/architecture_decisions/ADR-001-no-ai-reaches-the-robot.md) (`D-005`) | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| A stop can never be delayed or overridden by a peer | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| Delivery completion requires explicit recipient confirmation, never inferred from arrival | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |
| CloudLight operates **fully** when SHANA is offline | **NOT_RECOMMENDED_WITHOUT_APPROVAL** |

⚠️ **The answer to "what does the robot do when CloudLight is down" belongs to
SHANA's authors and their safety case.** CloudLight's obligation is to design so
that the question never becomes load-bearing.

---

## 19. What permissions are required?

| # | Permission | Blocks | Question |
|---|---|---|---|
| 1 | Run a persistent service on school premises | **Everything** | `CL-SRV-001`, `CL-GEN-002` |
| 2 | Allow classroom devices to reach that host | **Everything** | `CL-NET-003` |
| 3 | Create a local DNS name and stable address | Deployment | `CL-NET-006` |
| 4 | Publish timetable/substitution data to classroom screens | The core feature | `CL-DASH-001`, `CL-TIME-024` |
| 5 | Define who owns the master timetable and who approves a substitution | The core feature | `CL-TIME-001`, `CL-SUB-001` |
| 6 | Store identifiable staff availability/substitution records | The core feature | `CL-GEN-004` |
| 7 | Issue individual teacher accounts | Authentication | `CL-AUTH-001`, `CL-AUTH-003` |
| 8 | Create a dedicated read-only service account on shared storage | Resource features | `CL-NAS-007` |
| 9 | Approve an explicit list of folders for indexing | Resource features | `CL-NAS-008` |
| 10 | Index file contents (a derived copy), including OCR | Full-text search | `CL-NAS-015`, `CL-SEARCH-005` |
| 11 | Permit any external AI service | The assistant | `CL-AI-001` |
| 12 | Permit kiosk configuration on classroom devices | Display deployment | `CL-WIN-003`, `CL-WIN-007` |
| 13 | Permit automated unattended deployment | Updates | `CL-OTA-012` |
| 14 | Hold release signing keys | Updates | `CL-OTA-005` |
| 15 | Store audit logs, and for how long | Audit | `CL-SEC-019`, `CL-SEC-025` |
| 16 | Authorise a SHANA integration | Robot feature | `CL-SHANA-001`, `CL-SHANA-015` |

---

## 20. What are the highest-risk assumptions?

| # | Assumption | If false | Cheap test |
|---|---|---|---|
| 1 | An always-on host is permitted | **The project cannot proceed as designed** | Ask IT and the Principal. Fifteen minutes |
| 2 | Displays can reach that host | No system; silently, if isolation is on | Ask IT about isolation and firewall rules |
| 3 | A shared storage system exists | Resource features do not exist; the rest still does | Ask IT |
| 4 | A staff identity directory exists | CloudLight owns identity, provisioning and offboarding | Ask IT |
| 5 | Boards support a kiosk lockdown | The display must be opened by hand daily | `winver` on one board |
| 6 | Kiosk accounts have admin rights | No self-start, no self-update; every change is a site visit | Ask IT |
| 7 | A timetable authority exists to approve substitutions | **The core feature has no publication path** | Ask administration |
| 8 | External AI is not permitted | ⛔ **The assistant is simply dropped** | Ask the Principal |
| 9 | Offline caching works inside a kiosk session | The offline display degrades to a manual-start browser | Test on one board |
| 10 | Power and network are stable during the day | Displays show stale data or nothing | Ask facilities |

⚠️ **Assumptions 1, 2 and 7 are the three that end the project** rather than
change its shape. Everything else changes the shape.

---

## 21. What is the smallest useful first release?

> **A correct, honest timetable and substitution display on one or two classroom
> smartboards, backed by a single always-on school host, with human approval
> before anything publishes.**

| In scope | Out of scope |
|---|---|
| Classroom dashboard: current lesson, subject, times, countdown | AI assistant |
| Scheduled teacher, and confirmed substitute when approved | Semantic / vector search |
| Visible last-synchronised timestamp | OCR of photographed boards |
| Deterministic timetable ingestion from a spreadsheet or CSV | AI-assisted extraction |
| Substitution pipeline with human approval | Autonomous substitution choice |
| Versioned, rollback-able publication | Resource browsing and search *(no storage confirmed)* |
| Individual teacher accounts | SHANA task submission |
| Separate administration surface | OTA self-update |
| Fail-closed offline behaviour with a staleness marker | Mobile apps |
| Backups and health checks | Multi-school support |

⚠️ **This release is useful on its own**, does not depend on any of the five
blocked unknowns, and establishes the correct-data path that everything else
would later depend on. See [`first_release_scope.md`](first_release_scope.md).

---

## 22. What should be postponed?

| Deferred | Because |
|---|---|
| Resource browsing and search | **Shared storage is not confirmed to exist** |
| AI assistant | Retrieval quality is unproven; largest data-protection surface; gated on permission |
| Semantic / vector search | Filename and metadata search satisfy most real teacher queries at a fraction of the cost |
| OCR and board photographs | Permission unconfirmed; accuracy on handwriting is materially lower |
| SHANA integration | Eight open questions; adding it first would put an unconfirmed dependency under the core product |
| OTA self-update | Ship a release first; there is nothing to update yet |
| Packaged desktop clients | A browser kiosk updates itself |
| External AI inference | Should not be assumed permitted |
| Multi-school support | One school |
| Real-time WebSocket push | Periodic refresh is adequate until latency requirements are measured |

⚠️ **Almost nothing here is deferred because it is technically hard.** Most is
deferred because a precondition has not been confirmed to exist.

---

## 23. Unresolved questions

The full set is in
[`open_questions_register.md`](open_questions_register.md) and the 576-question
instrument is `../../questionnaires/Additional_CloudLight_Questionnaire.md`.

**The five that must be answered first:**

| # | Question | Who | Consequence if unanswered |
|---|---|---|---|
| 1 | May a persistent service run on school premises? | Administration + IT | **The project cannot proceed** |
| 2 | Can classroom devices reach that service? | IT | **The project cannot proceed** |
| 3 | Is client isolation enabled on the school network? | IT | Fails silently — the worst failure mode |
| 4 | Who approves a substitution? | Timetable/administration | **The core feature has no publication path** |
| 5 | What Windows edition is on the classroom boards? | IT | The kiosk design may be impossible |

---

## 24. Sources

Full bibliography with URLs, types and limitations:
[`sources.md`](sources.md).

Every conclusion above is traceable to either a cited source, an internal
document, or an explicitly labelled engineering judgement. Claims resting on
judgement rather than evidence are listed in `sources.md` §4 so that nobody
mistakes them for verified findings.

⚠️ **No school-specific fact in this report has been verified**, because none has
been checked. Every school-specific statement is a question with an owner.