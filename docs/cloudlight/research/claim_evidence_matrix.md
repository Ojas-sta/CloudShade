# CloudLight — Claim-to-Evidence Matrix

**Every claim someone might say out loud about CloudLight, with what it rests on.**

> ### ⚠️ **How to read this.**
>
> - **Implementation status** — what exists: `DESIGN_ONLY` (written down, nothing
>   built) · `RESEARCH_ONLY` (researched, never implemented) · `BLOCKED` (cannot
>   proceed until someone answers) · `UNKNOWN`.
> - **Verification status** — how well it is established: `VERIFIED` ·
>   `DOCUMENTED_NOT_HARDWARE_VERIFIED` · `PLAUSIBLE_REQUIRES_TESTING` ·
>   `UNKNOWN` · `REQUIRES_SCHOOL_PERMISSION` ·
>   `REQUIRES_VENDOR_OR_SERVICE_PROVIDER` · `NOT_RECOMMENDED_WITHOUT_APPROVAL`.
> - **May it appear in an exhibition proposal?** — `YES` · `YES_WITH_CAVEAT` ·
>   `NO`. ⛔ **A `NO` is absolute.** Nothing may be presented as an operational
>   capability, a deployed feature, or a demonstrated result.
> - **Recommended accurate wording** — a sentence that may be said *as written*.
>   ⚠️ **Say the sentence in that column. Do not improvise a stronger version.**

> ### ⠔⛔ **No claim about this school's infrastructure is verified. None.**
> ### ⛔ **No claim about this school's infrastructure appears as `VERIFIED`.**
>
> **Network probes: zero. Smartboards touched: zero. NAS shares mounted: zero.
> AI services contacted: zero. Models run: zero. Robot operated: zero. School
> documents read: zero. Hosts deployed: zero. Tests executed: zero.**
>
> Every school-specific row below is `UNKNOWN` or `REQUIRES_SCHOOL_PERMISSION`.
> There are **576 `CL-*` questions** in
> [`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md)
> and **all 576 are `UNANSWERED`.**

---

## 1. Platform form and hosting

| Claim ID | Exact claim | Source | Impl. status | Verification status | Limitations | Exhibition? | Recommended accurate wording |
|---|---|---|---|---|---|---|---|
| `CE-001` | An access point can host the CloudLight backend | `CR-009` | DESIGN_ONLY | `VERIFIED` *(that this is false)* | An AP is a Layer-2 bridge. Port forwarding forwards to a device that must already exist. Some enterprise APs offer captive-portal services — none is an application host | ⛔ **NO** — as stated, this is false. May only be quoted **in its refuted form** | "An access point connects devices; it cannot host a server. That is a networking fact, not a design opinion." |
| `CE-002` | An access point is a Layer-2 bridge, not an application host | `CR-009` Cisco WLAN overview | DESIGN_ONLY | `VERIFIED` | Single vendor's documentation for one product line. Establishes the general topology only | ✅ **YES** | "An access point is a bridge — it moves frames, it doesn't run software. So the platform needs a real computer." |
| `CE-003` | CloudLight will run on an always-on school machine | `CR-010`, [`../decision_log.md`](../decision_log.md) `D-101` | **BLOCKED** | `REQUIRES_SCHOOL_PERMISSION` · `CL-SRV-001`, `CL-GEN-002` | **Whether a persistent service may run on school premises at all is unanswered.** If the answer is no, the entire local-first premise fails | 🟡 **YES_WITH_CAVEAT** | "We would propose an always-on school computer to run this on — **but that needs the school's permission, and we don't have it yet.**" |
| `CE-004` | Classroom displays can reach that host | `CR-008`, `CR-009` | **BLOCKED** | `UNKNOWN` · `CL-NET-003` | Six links in the chain — name resolution, addressing, isolation, firewall, segmentation, TLS — each fails independently. **A working ping proves none of the rest.** No network was probed | ⛔ **NO** | *(No wording. This must not be asserted. If asked: "That's one of the first things we'd need the school to confirm.")* |
| `CE-005` | Client isolation may block the system while Wi-Fi looks fine | `CR-008` Aruba + WatchGuard | RESEARCH_ONLY | `VERIFIED` *(as a general risk)* | Vendor-specific. ❓ Whether it is enabled on **this** school's SSID is `CL-NET-008`, P0, **unanswered** | 🟡 **YES_WITH_CAVEAT** — as a risk, not as a finding about this school | "Some school networks block device-to-device traffic even when the Wi-Fi looks perfect. **That's a general risk we need to check with IT**, not something we've found here." |
| `CE-006` | The NAS could host the backend | `CR-010` Synology | RESEARCH_ONLY | `UNKNOWN` · `CL-NAS-001`, `CL-NAS-002` | A product feature page, not a specification. Availability varies by model and DSM version. **A NAS may not exist at all** | ⛔ **NO** | "If the school has a NAS, it *might* be able to host the service — **we haven't confirmed there is one, or what it is.**" |
| `CE-007` | CloudLight is a modular monolith, not microservices | [`../architecture.md`](../architecture.md) §2, `D-021` | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | 🧪 **Engineering judgement.** The sizing argument is sound: a few dozen displays polling on a schedule is not a scaling problem. ❓ Confirm device count (`CL-WIN-020`) | 🟡 **YES_WITH_CAVEAT** | "We're proposing one application with clear internal boundaries, rather than lots of separate services — **because for one school, microservices would add operational burden without solving a real problem.**" |
| `CE-008` | CloudLight is browser-first, not a packaged desktop client | `D-022`, `CR-001`…`CR-003` | DESIGN_ONLY | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | Windows kiosk behaviour is documented; untested here. **Depends entirely on the board's Windows edition, which is unknown** | 🟡 **YES_WITH_CAVEAT** | "The design uses a locked-down web browser rather than an installed app — **mainly so updates reach the screen from the server** rather than needing an installer on every classroom machine." |

---

## 2. Windows, kiosk and the offline contradiction

| Claim ID | Exact claim | Source | Impl. status | Verification status | Limitations | Exhibition? | Recommended accurate wording |
|---|---|---|---|---|---|---|---|
| `CE-009` | Microsoft Edge kiosk mode with Assigned Access or Shell Launcher is the documented path to a locked-down display | `CR-001`, `CR-002`, `CR-003` | DESIGN_ONLY | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | Microsoft documents this. **No board has been touched.** ❓ Which edition is installed is `CL-WIN-001`, P0 | 🟡 **YES_WITH_CAVEAT** | "Microsoft's own documentation describes a locked-down browser mode for exactly this. **We've followed their documentation; we haven't tested it on your boards.**" |
| `CE-010` | The kiosk design works on this school's boards | `CR-001`, `CR-005` | **BLOCKED** | `UNKNOWN` · `CL-WIN-001`, `CL-WIN-003` | Assigned Access support depends on edition; Shell Launcher is Enterprise/Education only; ⛔ **UWF is unavailable on Home and Pro**. If the boards run Pro, the entire recommendation collapses | ⛔ **NO** | "Whether that's possible depends on which Windows edition your boards run — **that's the first question we'd ask.**" |
| `CE-011` | A Windows service cannot be the kiosk launcher | `CR-006` Interactive Services | DESIGN_ONLY | `VERIFIED` | A **platform limit since Windows Vista**, not a policy choice. Services run in session 0 and cannot display UI | ✅ **YES** | "A Windows background service can't put a window on screen — that's a Windows limitation, not something we chose." |
| `CE-012` | The kiosk idle timeout restarts the browser | `CR-003` Edge kiosk mode | DESIGN_ONLY | `VERIFIED` *(that this is **false**)* | ⛔ Microsoft documents verbatim that the idle timeout **does not** restart Edge — a separate mechanism is required. **Common misconception; do not repeat it** | ⛔ **NO** as stated | "One thing worth knowing: the browser's own idle timeout doesn't bring the browser back. **You need a separate restart mechanism configured around it.**" |
| `CE-013` | The display will keep working when the host is unreachable | `CR-003`, `CR-005` | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` · **highest-risk unknown** | ⛔⛔ **Both documented kiosk lockdown experiences run an InPrivate session with autofill disabled.** Whether persistent offline caching survives that **is not answered by any source, and no board has been touched.** If caching is forbidden, one of two non-negotiable requirements must give | ⛔ **NO** | "The design intends the screen to keep showing the last known timetable, clearly marked with when it was last updated. **Whether that's possible inside a locked-down browser session is something we still need to test on one machine.**" |
| `CE-014` | Displays boot faster than the network, so an offline state is needed | [`research_log.md`](research_log.md) Phase C | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | 🧪 **Community knowledge, not a vendor claim.** Very likely correct; entirely untested. Recorded as unsourced judgement in [`../sources.md`](../sources.md) §4 #4 | 🟡 **YES_WITH_CAVEAT** | "A classroom computer usually starts up before the network does, **so the design assumes the screen may be briefly offline** and handles it explicitly rather than showing a blank screen." |

---

## 3. Timetable and substitutions

| Claim ID | Exact claim | Source | Impl. status | Verification status | Limitations | Exhibition? | Recommended accurate wording |
|---|---|---|---|---|---|---|---|
| `CE-015` | If the timetable is a Word or Excel file, it can be read exactly, with no AI involved | `CR-022` OOXML | DESIGN_ONLY | `VERIFIED` | ✅ Office Open XML files are ZIP containers of XML — **but this is conditional on a fact nobody has confirmed**: what format the school's timetable is actually in | 🟡 **YES_WITH_CAVEAT** | "If your timetable is a Word or Excel file — **which we don't know yet** — it can be read exactly and reliably, without any AI guessing." |
| `CE-016` | Deterministic parsing is always tried before any AI | `CR-022`, `D-006`, [`../feasibility_report.md`](../feasibility_report.md) §9 | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | 🧪 Design principle, sound and standard, but nothing has been parsed. ⛔ **No parser exists** | ✅ **YES** | "The first thing any timetable system should do is read the file properly. **AI is only ever a fallback for genuinely messy input — never the first choice.**" |
| `CE-017` | OCR can read a photograph of a classroom board | `CR-025`, `CR-023` | RESEARCH_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | ⛔ **An unsupported script is not read poorly — it is not read.** Language counts say nothing about handwriting. ⚠️ **Whether photographing boards is permitted is `CL-SUB-006`, unanswered** | ⛔ **NO** | "Reading a handwritten board photograph is possible in principle, **but it's unreliable, and we'd want the school to confirm photographing boards is acceptable before we even tried it.**" |
| `CE-018` | AI can extract timetable fields from scanned documents | `CR-023`, `CR-024` | RESEARCH_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | ⛔⛔ **The Docling benchmark ran with OCR disabled**, so its fast figures and its scanned-document case come from different experiments. ⛔ **Table models were trained on finance, patents and law — not school timetables.** ❓ No school document has been seen | ⛔ **NO** | *(Not presentable. If pressed: "Automated reading of scanned timetables is the riskiest part of this project, **which is exactly why it's not in the first version.**")* |
| `CE-019` | A substitution appears on screen only after a named person approves it | `D-008`, `FR-SUB-*` | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` · `CL-SUB-001` | ✅ The gate is the correctness property, not an accessory. ❓ **Who approves is unknown**; whether a second person must counter-sign is `CL-SUB-002` | 🟡 **YES_WITH_CAVEAT** | "Nothing appears on a classroom screen until a named person approves it. **The school decides who that person is — we don't know yet.**" |
| `CE-020` | Stale content is shown with a visible marker, never unmarked and never blank | [`../classroom_dashboard.md`](../classroom_dashboard.md) §6, `D-009` | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | 🧪 **Engineering judgement.** ⚠️ [`../classroom_dashboard.md`](../classroom_dashboard.md) states it carries **no external citations at all** — the display-state catalogue, countdown tiers and staleness tiers are entirely design | ✅ **YES** — it is a principle, and principles may be stated | "If the screen can't reach the server, it shows what it last knew — **and says so, with the time.** Showing stale data unmarked, or a blank screen, is the thing we've designed hardest to avoid." |

---

## 4. Authentication and access

| Claim ID | Exact claim | Source | Impl. status | Verification status | Limitations | Exhibition? | Recommended accurate wording |
|---|---|---|---|---|---|---|---|
| `CE-021` | A shared six-digit code is sufficient to identify a teacher | `CR-028`, `D-005`, [`../feasibility_report.md`](../feasibility_report.md) §12 | DESIGN_ONLY | ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` | ⛔ **It is not, and the project says so.** 10⁶ guessing space; a shared unattended display can be photographed; **the code identifies nobody**, so nothing is attributable and no one can be revoked individually | ⛔ **NO** | *(Not presentable as a capability. If asked directly: "**No — a shared code doesn't prove who anyone is.** That's why the design uses individual teacher accounts.")* |
| `CE-022` | A six-digit code may authorise **one** action, or act as a second factor | `D-005`, `CR-028` | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | 🟡 Permitted only with short lifetime, single use, attempt limits and lockout, rate-limited resend, never displayed after the teacher looks away, never in a URL, log, browser storage, config or Git. ⚠️ ⛔ **Server-side rate limiting is a P0 control, not a usability convenience.** ❓ `CL-CODE-003` — which actions are "consequential" — is unanswered, so the split is **our proposal, not school policy** | 🟡 **YES_WITH_CAVEAT** | "A short code can be allowed to approve **one specific action**, or to act as a second factor — **but never to stand in for knowing who someone is.**" |
| `CE-023` | Requesting a robot delivery requires individual authentication | `CE-022`, `D-003`, `FR-AUTH-01` | DESIGN_ONLY | ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` | ⛔ **A physical action with a real load and a real route is the most consequential thing this system can do.** It must not rest on a shared code | ✅ **YES** — as a safety property | "Asking a robot to move something through a corridor **always requires a named, logged-in teacher.** That one is never allowed to be shared." |
| `CE-024` | SMS is a suitable second factor | `CR-028` NIST + OWASP | DESIGN_ONLY | ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` | ⛔ NIST designates SMS/PSTN a **"restricted" authenticator** — SS7 interception, SIM-swap, number-porting | ⛔ **NO** | "**We wouldn't use text-message codes** — the standard treats them as the weakest factor. Passkeys or an authenticator app are the realistic options." |
| `CE-025` | Sessions have both an idle and an absolute timeout | `CR-027` NIST SP 800-63B Rev. 4, `CR-030` | DESIGN_ONLY | `VERIFIED` | ✅ Rev. 4 requires **both**, not one or the other. ⚠️ US federal guidance — adopted as **good practice, not law**; whether it binds an Indian school is `CL-AUTH-016` | 🟡 **YES_WITH_CAVEAT** | "Sessions log out on their own after a set idle time **and** after a hard maximum, whichever comes first — **that's standard practice, which we follow as a matter of good practice rather than legal requirement.**" |
| `CE-026` | A display may hold a logged-in teacher session | [`../requirements.md`](../requirements.md) `FR-AUTH-02`, `CL-AUTH-020` | DESIGN_ONLY | `UNKNOWN` · `REQUIRES_SCHOOL_PERMISSION` | ⛔ **The project's recommendation is no** — anyone in the room would inherit teacher authority. ❓ **It is the school's decision** | ⛔ **NO** | "We'd recommend **not** leaving a teacher logged in on a shared classroom screen. **But that's the school's call, and we haven't asked yet.**" |

---

## 5. Resources, search and AI

| Claim ID | Exact claim | Source | Impl. status | Verification status | Limitations | Exhibition? | Recommended accurate wording |
|---|---|---|---|---|---|---|---|
| `CE-027` | Search results are filtered by the user's permissions before ranking | `CR-015`, `D-026`, `FR-SEARCH-02` | RESEARCH_ONLY | `VERIFIED` *(as a technique)* | ✅ Implemented by production permission-aware search. ⛔ **Not implemented here.** ❓ Requires the identity directory the storage authenticates against (`CL-AUTH-002`) | 🟡 **YES_WITH_CAVEAT** | "Search is filtered by what each person is allowed to see **before results are shown** — the system never widens what the underlying storage permits." |
| `CE-028` | Search will work at this school | `CR-015`, `CR-012` | **BLOCKED** | `UNKNOWN` · `CL-NAS-001`, `CL-SEARCH-014` | ⛔ **A school may have no shared storage at all.** ❓ Corpus size unknown → **no search engine can be chosen.** ❓ Which directory, which filesystem | ⛔ **NO** | *(Not presentable. "**We don't know yet whether the school has shared storage that this would work against.** That's a question for IT.")* |
| `CE-029` | ⛔ Search can never show a document the user cannot open | `CR-015`, `D-026` | DESIGN_ONLY | ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` | The design's central invariant. ⚠️ **Both failure directions are wrong**: an over-broad index leaks; an over-narrow one hides files the user can open. **Both must be tested** | ✅ **YES** — as an invariant | "The search layer can only ever make access **narrower**, never wider. **It can't grant access to anything the underlying storage already refuses.**" |
| `CE-030` | AI answers are grounded in school documents, not generated freely | `CR-020`, `CR-021`, `FR-AI-05` | RESEARCH_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | 🧪 Design intent. ⚠️ ⛔ **The leap from "injection is not fully mitigable" to "not deployable in a classroom" is this project's judgement** | ⛔ **NO** | *(No assistant exists. "**We're not proposing an AI assistant in the first version at all.** Its value depends entirely on search working first, which is still unconfirmed.")* |
| `CE-031` | CloudLight will use local AI inference, not an external service | `CR-018`, `CR-019`, `D-024`, `D-105` | RESEARCH_ONLY | `PLAUSIBLE_REQUIRES_TESTING` · `CL-AI-001`, `CL-AI-009` | ⛔ **Whether any AI service is permitted at all is a school decision nobody has made.** ❓ Whether a GPU exists is unknown. ⛔ **No model was downloaded or run.** ⚠️ A permanently offline school cannot download model weights — everything must be transferred in advance, and a missed file breaks setup silently | 🟡 **YES_WITH_CAVEAT** | "If AI is ever used, **we'd want it running entirely on school hardware**, so no school data leaves the building. **But whether AI is approved at all is the school's decision, and it's not been asked.**" |
| `CE-032` | AI can be made safe by filtering | `CR-020` OWASP LLM01 | RESEARCH_ONLY | ⛔ **Explicitly contradicted** | ⛔⛔ **OWASP states RAG and fine-tuning do NOT fully mitigate prompt injection, that fool-proof prevention is unclear, and that no mitigation is complete.** ⛔ An attacker modifying a document in a RAG corpus *is* the school shared-drive scenario | ⛔ **NO** | ⛔ *(Not presentable under any wording. "Filtering alone is not considered sufficient — **that's why the design removes the assistant's ability to send anything anywhere at all**, rather than trusting filters.")* |
| `CE-033` | No external AI service by default | [`../architecture.md`](../architecture.md) §8, `D-024`, `X-05` | DESIGN_ONLY | `NOT_RECOMMENDED_WITHOUT_APPROVAL` | ⛔ **Prohibited by default.** Any external service would send school data to a third party without authorisation — permitted only in writing, with verified no-retention configuration | ✅ **YES** — as a constraint | "**Nothing goes to an outside AI service by default.** If the school ever wanted that, it would need explicit written permission — and we haven't asked for one." |

---

## 6. Updates and operations

| Claim ID | Exact claim | Source | Impl. status | Verification status | Limitations | Exhibition? | Recommended accurate wording |
|---|---|---|---|---|---|---|---|
| `CE-034` | Updates are signed and verified on the device | `CR-032`, `CR-034`, `D-025` | DESIGN_ONLY | `VERIFIED` *(as a principle)* | ✅ The signature must be verified by the **consuming software**, not the distribution point. ⛔ **No key exists, no artefact has been signed, no board enrolled** | 🟡 **YES_WITH_CAVEAT** | "Any software update would have to be **signed, and the signature checked on the screen itself** rather than trusted because it came from our server." |
| `CE-035` | Signature checking alone makes updates secure | `CR-034` TUF | DESIGN_ONLY | `VERIFIED` *(that this is **false**)* | ⛔ **TUF names rollback, freeze, fast-forward and mix-and-match attacks — all of which a signature alone defeats none of.** Metadata expiry and separate role keys are what resist them | ⛔ **NO** as stated | "Checking a signature isn't enough on its own — **you also need version pinning and expiry**, or an old signed-but-vulnerable version could be replayed. That's why the design names those attacks explicitly." |
| `CE-036` | CloudLight can update itself in the field | `CR-034`, `D-025` | RESEARCH_ONLY | `UNKNOWN` · `CL-OTA-010`, `CL-OTA-012` | ❓ Whether kiosk accounts can install an updating agent. ❓ Whether unattended deployment is permitted. ⛔ **If accounts lack admin rights, every change becomes a site visit** — a cost nobody has priced | ⛔ **NO** | "Automated updates are **proposed but not decided** — it depends on what the school's IT allows, and we haven't asked. **There is also nothing installed to update yet.**" |
| `CE-037` | A school LAN is trusted because it is internal | [`../sources.md`](../sources.md) §4 #5 | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | 🧪 **Sound reasoning with no single citation.** ⚠️ It happens to be right, which is **not** the same as being evidenced | ⛔ **NO** as stated | "We treat the school network as **untrusted**, the same as the internet — **being inside the building doesn't make it safe.**" |
| `CE-038` | Offline-first means the system is secure | [`../sources.md`](../sources.md) §5.6 | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | ⚠️ **Offline-first is a resilience property, not a security posture.** A permanently offline host never receives OS, database, browser or library security fixes; credentials that never rotate are eventually compromised | ⛔ **NO** as stated | "**Working without internet is about reliability, not safety.** A system that never updates is a system that never gets security fixes — so there's a real trade-off there that we're not pretending doesn't exist." |
| `CE-039` | A named person would respond to a failure | `CR-033`, `CL-SRV-020`, `CL-NET-020` | **BLOCKED** | `UNKNOWN` · `REQUIRES_SCHOOL_PERMISSION` | ⛔⚠️ **Two questions presuppose a monitoring capability nobody has confirmed exists.** If there is none, **there is no path by which a host failure reaches a person**, which makes `NFR-AVL-05` unachievable as written | ⛔ **NO** | "**We'd need to know who at the school would be told if something broke** — and that turns out to be a bigger question than it looks, because it assumes some monitoring exists." |

---

## 7. The CloudLight ↔ SHANA boundary

| Claim ID | Exact claim | Source | Impl. status | Verification status | Limitations | Exhibition? | Recommended accurate wording |
|---|---|---|---|---|---|---|---|
| `CE-040` | CloudLight may command the robot | `D-003`, ADR-014, `FR-SHANA-01` | DESIGN_ONLY | ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` | ⛔ **Never.** ✅ **Correctly expressed as an absence of capability**: the SHANA schemas contain no motor, path, speed or actuator field, so a motor command is **not expressible**, not merely forbidden by policy | ⛔ **NO** | *(Not presentable. If asked: "**CloudLight can ask for a delivery. It cannot tell the robot how to move — there is no channel for that, by design.**")* |
| `CE-041` | CloudLight never controls robot motion | `D-003`, [`../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md`](../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md) | DESIGN_ONLY | ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` — **non-negotiable** | ✅ Expressed as an absence of capability rather than a policy. ⚠️ **A low-level command path that exists "only for debugging" converts a software defect into physical motion in an occupied corridor** | ✅ **YES** — it is a safety property, and stating it strengthens the proposal | "**CloudLight can request a delivery; it can never command the robot.** That boundary isn't a rule we wrote down — it's a limit on what the two systems can even say to each other." |
| `CE-042` | The CloudLight↔SHANA interface exists | [`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md) §3, `CL-SHANA-001` | **BLOCKED** | `REQUIRES_VENDOR_OR_SERVICE_PROVIDER` | ⛔ **SHANA's own document marks it `DESIGN_ONLY` — never implemented, never validated against a real partner — and says the schemas "have never been parsed by code."** ⛔ **SHANA has not agreed to CloudLight's proposal.** ❓ The real interface depends entirely on what the robot vendor permits | ⛔ **NO** | *(Not presentable. "**The interface between the two is proposed and documented, not built. What it actually ends up looking like depends on the robot's own vendor.**")* |
| `CE-043` | The classroom system keeps working if the robot subsystem is down | `D-004`, [`../shana_integration.md`](../shana_integration.md) | DESIGN_ONLY | ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` — a constraint | ✅ **CloudLight operates fully when SHANA is offline.** ✅ **What the robot does when CloudLight is down is SHANA's decision, not CloudLight's specification.** ✅ Any component whose failure could injure a person must not depend on an external system's availability | ✅ **YES** — as a constraint | "**If the robot part is completely down, the classroom screens are completely unaffected.** That's a hard rule — a robot problem must never be able to blank a classroom screen." |
| `CE-044` | A delivery can be marked complete automatically | [`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md) §2, `data/schemas/README.md` | DESIGN_ONLY | ⛔ `NOT_RECOMMENDED_WITHOUT_APPROVAL` | ⛔ **Never inferred from arrival.** A delivery is never `COMPLETED` without explicit recipient confirmation | ✅ **YES** — as a safety property | "**A delivery is only recorded as finished when the person receiving it confirms it** — never just because the robot stopped moving." |
| `CE-045` | CloudLight and SHANA are already integrated | `CL-SHANA-001`, `CL-SHANA-015` | **BLOCKED** | `UNKNOWN` · `REQUIRES_VENDOR_OR_SERVICE_PROVIDER` | ⛔⛔ **There is no integration.** ❓ Eight open questions. ❓ The robot vendor has not been contacted. ✅ **The design recommendation is to build the classroom product first and add the robot integration last**, so an unconfirmed dependency never sits under the core product | ⛔ **NO** | ⛔ *(Not presentable. "**We'd build the classroom system first and add the robot side later** — the robot interface has open questions that only the robot's own vendor and custodian can answer.")* |

---

## 8. First-release scope

| Claim ID | Exact claim | Source | Impl. status | Verification status | Limitations | Exhibition? | Recommended accurate wording |
|---|---|---|---|---|---|---|---|
| `CE-046` | CloudLight is a working system | [`research_log.md`](research_log.md) Phase A | **NONE EXISTS** | `UNKNOWN` | ⛔⛔ **There is ZERO application source code in this repository.** No build files, no dependencies, no tests. CloudLight is a **proposal**, currently at design stage | ⛔ **NO** | ⛔ **The single most important thing not to say: do not present CloudLight as working, running, built or deployed.** "This is a design and research project at this stage. **Nothing has been built yet.**" |
| `CE-047` | Nine of sixteen proposed capabilities are not in the first release | [`../feasibility_report.md`](../feasibility_report.md) §1, [`../first_release_scope.md`](../first_release_scope.md) | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | ⚠️ **Four are deferred not because they are hard but because the thing they depend on has not been confirmed to exist** — shared storage, an identity directory, an always-on host, and AI policy | ✅ **YES** — stating limits strengthens a proposal | "**The first version is deliberately small** — timetable and substitution display on one or two screens, with human approval before anything publishes. Most of the rest waits on questions only the school can answer." |
| `CE-048` | The first release can start now | [`../feasibility_report.md`](../feasibility_report.md) §20 | **BLOCKED** | `REQUIRES_SCHOOL_PERMISSION` | ⛔ **Assumptions 1, 2 and 7 end the project rather than change its shape:** an always-on host is permitted, displays can reach it, and a timetable authority exists to approve substitutions. **All three are unanswered** | ⛔ **NO** | "**Before we could pilot anything, the school would need to confirm three things** — that a computer is allowed to run a service, that the screens can reach it, and who approves a substitution." |
| `CE-049` | ✅ The proposed first release needs **no** unconfirmed prerequisite | [`../first_release_scope.md`](../first_release_scope.md) §1, [`../feasibility_report.md`](../feasibility_report.md) §21 | DESIGN_ONLY | `PLAUSIBLE_REQUIRES_TESTING` | ✅ The core timetable display does not depend on AI, shared storage, an identity directory or the robot — which is exactly why it is the proposed starting point | ✅ **YES** | "**The timetable display is the part we can build first**, because it doesn't depend on the robot, on AI, or on anything else we're not sure exists." |

---

## 9. ⛔ Claims about this school's actual infrastructure — **all UNKNOWN**

> ### ⚠️ **Every row in this section is `UNKNOWN`. There is no exception.**
> ### ⛔ **None of these may be asserted, implied, or presented as a finding.**

| Claim ID | Exact claim | Source | Impl. status | Verification status | Limitations | Exhibition? | Recommended accurate wording |
|---|---|---|---|---|---|---|---|
| `CE-050` | Displays are wired or on Wi-Fi | `CL-DASH-*`, `CL-NET-*` | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | Not asked, not probed | ⛔ **NO** | "We don't know yet whether the classroom screens are wired or on Wi-Fi — **that's one of the first things we'd need to check with IT.**" |
| `CE-051` | Client isolation is (or is not) enabled on the school network | `CL-NET-008` P0 | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | ⛔ **No network was probed.** This is the single most important unknown in the project | ⛔ **NO** | "**Whether the school network blocks device-to-device traffic is unknown**, and it's the question that would most affect whether any of this works." |
| `CE-052` | A NAS exists at this school | `CL-NAS-001` | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | **A school may have no shared storage at all.** No NAS was contacted | ⛔ **NO** | "**We don't know whether the school has shared storage at all.** If it does, some of these features become possible; if it doesn't, they simply don't exist." |
| `CE-053` | There is a staff identity directory | `CL-AUTH-001`, `CL-AUTH-002` | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | ❓ No directory was contacted or enumerated. If none exists, CloudLight must own accounts — **and therefore own provisioning and offboarding** | ⛔ **NO** | "**Whether the school has a staff login system is unknown.** If it does, we'd integrate with it; if not, we'd have to manage teacher accounts ourselves." |
| `CE-054` | A board may be used for testing | `CL-WIN-007` | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | ⛔ **No smartboard was touched.** ⛔ This permission is what unlocks the cheapest, most decisive test in the project | ⛔ **NO** | "The cheapest thing that would move this forward is **permission to test on one non-critical machine** — that's a fifteen-minute test that answers the biggest open question." |
| `CE-055` | An external AI service is permitted | `CL-AI-001` | **BLOCKED** | `UNKNOWN` · `REQUIRES_SCHOOL_PERMISSION` | Not asked. ⛔ **A `NO` simply drops the assistant** | ⛔ **NO** | "**Whether AI is permitted at all is the school's decision, and it hasn't been made.** A 'no' is a completely acceptable answer — it would just mean no assistant." |
| `CE-056` | An unattended board may hold a teacher session | `CL-AUTH-020` | **BLOCKED** | `UNKNOWN` · `REQUIRES_SCHOOL_PERMISSION` | ⛔ **The project recommends no** | ⛔ **NO** | "**We'd recommend not leaving a teacher logged in on a shared screen**, but that's the school's decision." |
| `CE-057` | Photographs of classroom boards may be taken | `CL-SUB-006` | **BLOCKED** | `UNKNOWN` · `REQUIRES_SCHOOL_PERMISSION` | Not asked. ⛔ A `NO` closes the entire OCR domain permanently | ⛔ **NO** | "**We haven't asked whether photographing classroom boards is acceptable**, and we wouldn't do it either way without that answer." |
| `CE-058` | Shared storage uses SMB, NFS or something else | `CL-NAS-002`, `CL-NAS-003` | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | ⛔ If it is SMB1-only, the resource features are **cancelled**, not deferred | ⛔ **NO** | "**What kind of storage the school runs, if any, is unknown to us.** That matters, because it determines whether the resource features are possible at all." |
| `CE-059` | The classroom boards' Windows edition and build | `CL-WIN-001` P0 | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | ⛔ **A five-minute `winver` command settles it.** ⚠️ Not run, because no board was accessed | ⛔ **NO** | "**We don't know which Windows version the classroom screens run** — and that single fact determines whether the locked-down display design is even possible." |
| `CE-060` | A named person would respond to a failure | `CL-SEC-026`, `CL-SRV-020` | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | ⚠️ **Presupposes monitoring that may not exist.** ❓ Who at this school would be contacted is unknown | ⛔ **NO** | "**We'd need to know who at the school would be told if something went wrong.** That turns out to assume some monitoring exists, which is itself unconfirmed." |
| `CE-061` | The school's timetable is in a particular file format | `CL-TIME-009`, `CL-DATA-019` | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | ⛔ **No timetable has been seen.** ⚠️ This single fact decides whether deterministic parsing works or the project faces OCR | ⛔ **NO** | "**We haven't seen the timetable file.** If it's a spreadsheet, it can be read exactly; if it's a scan or a photograph, that's a very different problem." |
| `CE-062` | Any accessibility requirement applies to the display | `CL-UX-004`, [`source_register.md`](source_register.md) `CR-035` | **BLOCKED** | `UNKNOWN` · **REQUIRES_SCHOOL_PERMISSION** | ⛔⛔ **No accessibility standard — no WCAG, no EN 301 549, no school policy — is cited anywhere in CloudLight.** The contrast ratios and target sizes are **this project's proposals with no source behind them** | 🟡 **YES_WITH_CAVEAT** | "The display design is **built to be readable from the back of a room**, but **we haven't yet grounded those choices in an accessibility standard — and we don't know whether the school has one.**" |

---

## 10. What must never be said

> ### ⛔ **Non-negotiable. A single `NO` violation misrepresents the entire project.**

| ⛔ Never say | ⛔ Because |
|---|---|
| *"CloudLight is working" / "the system runs" / "it's deployed"* | ⛔ There is **zero application source code**. Nothing is built |
| *"We've tested it on the school's screens"* | ⛔ **Zero smartboards touched** |
| *"It's fast" / "it handles X displays" / "search is under a second"* | ⛔ **Zero tests executed.** Every performance number is a proposal, not a measurement |
| *"It works on their network"* | ⛔ **Zero network probes.** Client isolation is unknown and may break it silently |
| *"It connects to the robot"* | ⛔ The interface is `DESIGN_ONLY`, unvalidated, and **SHANA has not agreed to it** |
| *"The AI assistant answers questions about school documents"* | ⛔ No assistant exists, none is approved, and injection is **not fully mitigable** |
| *"It reads the timetable automatically"* | ⛔ **No timetable has been seen. No parser exists.** Nobody knows the file format |
| *"It's secure because it's on the school network"* | ⛔ The LAN is treated as **untrusted** precisely because internal does not mean safe |
| *"Six-digit codes keep it simple"* | ⛔ A shared code **identifies nobody.** Presenting it as sufficient is the exact failure `D-005` rejects |
| *"It updates itself automatically"* | ⛔ No key exists, no agent is installed, and ❓ unattended deployment is **unconfirmed** |
| *"Any of this needs no school approval"* | ⛔ **Sixteen permissions are required.** Five are blockers. All questions are `UNANSWERED` |

⚠️ **And the rule that covers all of the above:** if a claim would require a
person at the school to nod before it were true, **it is not yet true.**

---

## 11. The five sentences that are safe

> ✅ **These may be said, more or less as written, because each is either a
> general fact or an explicitly labelled design intent.**

1. **"This is a design and research project. Nothing has been built yet, and nothing has been tested on school equipment."**
2. **"An access point connects devices; it can't host a server — so the platform needs a real computer, which is a question for the school."**
3. **"CloudLight can ask for a robot delivery. It can never command the robot. That boundary is a limit on what the two systems can say to each other, not a rule we agreed on."**
4. **"Nothing appears on a classroom screen until a named person approves it, and if the screen can't reach the server it shows what it last knew — with the time it was last updated."**
5. **"Everything we don't know about your setup is written down as a question with an owner, rather than filled in with a guess."**

⚠️ **Note that four of the five are about limits.** That is not modesty. It is
the strongest available position: a proposal whose gaps are named is one a
school can evaluate, and a proposal whose gaps are hidden is one it cannot.

---

## 12. Standing note

⚠️ **No claim about this school's infrastructure is verified, because none has
been checked.** Every school-specific row in §9 is `UNKNOWN` or
`REQUIRES_SCHOOL_PERMISSION`, and all 576 `CL-*` questions are `UNANSWERED`.

⚠️ **Where these documents and the school's written policy disagree, the school
is right and this matrix is wrong.**

⚠️ **Filling an `UNKNOWN` with a plausible default is the failure mode this
matrix exists to prevent.** When a question is answered, update the row — and
move its verification status, never delete it.

**Related documents:** [`research_index.md`](research_index.md) ·
[`source_register.md`](source_register.md) ·
[`research_log.md`](research_log.md) ·
[`../feasibility_report.md`](../feasibility_report.md) ·
[`../sources.md`](../sources.md) ·
[`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md)