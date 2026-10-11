# CloudLight — Research Log

**Chronological record of research work performed on CloudLight.**

**All entries: 2026-10-11.** Recorded in the order the work was done, most
recent last.

> ### ⚠️ **Read this before reading any entry.**
>
> Every entry below records **what was actually done**. Where nothing was done,
> it says nothing was done and names what that costs. This log contains **no
> entries describing work on school hardware**, because **no such work occurred**.

> ### ⚠️ **The counters, for the whole project, as of this log:**
>
> Network probes: **zero**. Smartboards powered on or touched: **zero**. NAS
> shares mounted: **zero**. AI services contacted: **zero**. Models downloaded
> or run: **zero**. Robot operated or connected: **zero**. School documents
> read: **zero**. Hosts deployed: **zero**. Tests executed: **zero**.
> School questions answered: **zero**.

---

## Phase A — Establishing what this repository actually is

### 2026-10-11 — Repository discovery: is there any code here?

- **Question:** Before any research, what does this repository actually contain? Is CloudLight an existing system or a design?
- **Method:** Enumerated the repository tree and inventoried every file type present. Checked specifically for `package.json`, build files, manifests, lockfiles, test directories, and any JavaScript, TypeScript, Python, Go or Rust source.
- **What was established:** ⛔ **There is ZERO application source code.** No language runtime sources, no build configuration, no dependency manifests, no tests, no CI configuration for an application. The repository contains documentation and data schemas only. **CloudLight is a proposed system that has never been built.** Every document describing its behaviour describes *intent*, never *observation*.
- **What remained unknown:** Nothing about the repository. This was fully determinable and is now settled.
- **Affected documents:** every document in this folder; the framing of [`../feasibility_report.md`](../feasibility_report.md); the implementation-status column in [`claim_evidence_matrix.md`](claim_evidence_matrix.md)
- **Confidence change:** ⛔ **Every CloudLight claim is now firmly `DESIGN_ONLY`.** There was never a possibility of `RESEARCH_ONLY` findings from execution, because there is nothing to execute.

### 2026-10-11 — Ownership and boundary discovery

- **Question:** Whose files are these, and what may be touched?
- **Method:** Read the ownership declarations at the head of SHANA's integration document and compared them against the repository tree.
- **What was established:** `docs/shana/**`, `data/**` and `questionnaires/**` are **SHANA-owned**. `docs/cloudlight/**` is CloudLight's. SHANA's own document states its integration proposal is **`DESIGN_ONLY` — never implemented, never validated against a real partner**, and that the schemas in `data/schemas/` *"have never been parsed by code."* The schema README confirms: *"Documentation only. No schema has been implemented. No database exists."*
- **What remained unknown:** The actual robot interface, which only the robot custodian and vendor can supply.
- **Affected documents:** [`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md) · [`../../../data/schemas/README.md`](../../../data/schemas/README.md) — **both read only, never modified**
- **Confidence change:** `CE` claims about SHANA integration are `RESEARCH_ONLY` / `UNKNOWN`, never `DESIGN_ONLY`-complete.

---

## Phase B — The finding that reframed the brief

### 2026-10-11 — The access-point premise

- **Question:** The brief assumed that because every smartboard connects to the school's access point, the platform could run there. Is that right?
- **Method:** Read network vendor documentation on what an access point *is* — specifically Cisco's WLAN overview for the bridging/routing description — then read Aruba and WatchGuard documentation on client isolation to test the second half of the assumption.
- **What was established:** ⛔ **An access point is a Layer-2 bridge that forwards frames. It is not an application host.** Router "port forwarding" does not create a backend — it forwards to a device that must already exist. ✅ **And the second half of the premise fails too:** client isolation *"isolates clients from one another and disables all peer-to-peer communication within the network"*, allowing *"only client to gateway traffic"*. On Aruba it is a **per-SSID** setting; on WatchGuard firmware 8.6.x+ it also blocks traffic *"with wired-side hosts on the same network"*. **A board can show full Wi-Fi signal, be correctly associated, and still be unable to reach any peer device.**
- **What remained unknown:** ⛔ Everything about this school's actual network. Is isolation on? Is the SSID the boards use one where it is enabled? What firewall rules apply? **None of this may be determined by probing** — [`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md) records the prohibition, and it is correct.
- **Affected documents:** [`../network_and_hosting.md`](../network_and_hosting.md) §2.1, §2.5 · [`../feasibility_report.md`](../feasibility_report.md) §5, §6 · [`source_register.md`](source_register.md) `CR-008`, `CR-009`
- **Confidence change:** ✅ The *general* claim moved to `VERIFIED`. ❓ The *school-specific* claim is `UNKNOWN` and is P0. **This is the project's most likely silent failure — the design fails with a screen that never loads, not with an error.**

---

## Phase C — Windows, kiosk and startup

### 2026-10-11 — Windows kiosk mechanisms: Assigned Access, Shell Launcher, Edge kiosk

- **Question:** Is there a supported way to lock a classroom Windows display to one application, and what does that actually do?
- **Method:** Read Microsoft Learn documentation on the kiosk configuration overview, single-app Assigned Access, Shell Launcher configuration and deployment, the Shell Launcher quickstart, Edge kiosk mode, and Intune-delivered kiosk settings. Recorded edition support tables verbatim rather than paraphrasing them.
- **What was established:** ✅ Assigned Access is the documented single-app kiosk mechanism and applies lockdown automatically; the default `Ctrl+Alt+Del` breakout is configurable; the `LogonUI` `IdleTimeOut` default is 30 s and is **explicitly documented not to apply to Edge kiosk mode**. ✅ Shell Launcher is Enterprise/Education only, is enabled through the Assigned Access CSP, and **cannot be combined with Assigned Access on one machine**. ✅ Edge kiosk mode offers two lockdown experiences. ⛔⛔ **The idle timeout does not restart Edge — Assigned Access or Shell Launcher is required**, per Microsoft's own warning. ✅ **A Windows service cannot display UI**: services have been unable to interact with a user since Windows Vista, running in session 0. This is a **platform limit, not a policy choice**, and it closes off the service route definitively.
- **What remained unknown:** ❓ **Which Windows edition is on these boards** (`CL-WIN-001`). ❓ Whether kiosk accounts have administrator rights (`CL-WIN-003`). ❓ Whether any MDM is deployed (`CL-WIN-005`). ⛔ **No smartboard was powered on, configured or touched.**
- **Affected documents:** [`../windows_startup.md`](../windows_startup.md) §1–§3 · [`source_register.md`](source_register.md) `CR-001`, `CR-002`, `CR-004`, `CR-006`
- **Confidence change:** All platform claims → `DOCUMENTED_NOT_HARDWARE_VERIFIED`. No claim → `VERIFIED` about the school's estate.

### 2026-10-11 — Unified Write Filter, and a lockdown layer that may not exist

- **Question:** Can write-filtering make boards resilient to accidental change?
- **Method:** Read the UWF overlay documentation and the `UWF_Filter` WMI provider page. Extracted the edition support table verbatim.
- **What was established:** ⛔ **UWF is not available on Windows Home or Pro.** Enterprise, Education and IoT Enterprise only. ✅ The overlay defaults to 1024 MB and is cleared on reboot. ⛔ **Overlay exhaustion causes failed writes and an unresponsive device**, and ⛔ **shutdown can take "up to an hour or longer" when the overlay is full.** Enabling it casually is worse than not enabling it.
- **What remained unknown:** ❓ The boards' edition — so whether this layer exists at all is unknown. **If any board runs Pro, this option is permanently unavailable and cannot be enabled by configuration, only by re-imaging.**
- **Affected documents:** [`../windows_startup.md`](../windows_startup.md) §3 · [`source_register.md`](source_register.md) `CR-005`
- **Confidence change:** `PLAUSIBLE_REQUIRES_TESTING`. **If `CL-WIN-001` returns Pro, this should be retired as unavailable rather than left open.**

### 2026-10-11 — ⛔ The kiosk-offline contradiction (the highest-risk unknown)

- **Question:** The display must keep working when the host is unreachable (`NFR-OFF-02`). The display must also be locked down (`FR-AUTH-02`, constraint C7). **Can both hold?**
- **Method:** Cross-read the Edge kiosk-mode documentation against the offline requirements. Specifically checked what the documented lockdown experience does to session persistence.
- **What was established:** ⛔⛔ **Both Edge kiosk lockdown experiences run an InPrivate session with autofill disabled.** ⚠️ Whether persistent offline caching survives that configuration **is not answered by any source**, and no board has been touched. **If a locked-down kiosk session forbids persistent caching, one of the two non-negotiable requirements must give.** The resolution is not a design decision — it is a test on one non-critical machine, and it is cheap.
- **What remained unknown:** ⛔ Everything that matters. This is the single highest-risk unknown in the project.
- **Affected documents:** [`../windows_startup.md`](../windows_startup.md) · [`../sources.md`](../sources.md) §5.1 · [`source_register.md`](source_register.md) `CR-003`
- **Confidence change:** ⛔ **This is why `VERIFIED` on its own is not sufficient as a confidence label.** The vendor documentation is authoritative *and* the requirement may be unachievable.

### 2026-10-11 — Boot race

- **Question:** Will the display have a network when it has a browser?
- **Method:** Reasoned from the architecture rather than from a source. No vendor documentation was found that addresses classroom boot ordering, and **none was invented**.
- **What was established:** 🧪 A PC boots faster than the network comes up, so a display will start before it can reach the host. **The fix is an application-level offline state showing cached data with a visible staleness marker — not a browser flag.**
- **What remained unknown:** Everything about this school's actual boot behaviour, power stability and network readiness timing.
- **Affected documents:** [`../windows_startup.md`](../windows_startup.md) §4 · [`../classroom_dashboard.md`](../classroom_dashboard.md) §6
- **Confidence change:** 🧪 Remains `PLAUSIBLE_REQUIRES_TESTING`. ⚠️ **Recorded in [`../sources.md`](../sources.md) §4 #4 as community knowledge, not a vendor claim.** It is very likely correct and entirely untested. An honest entry in the register, not a weakness in it.

---

## Phase D — Authentication

### 2026-10-11 — ⛔ The six-digit-code finding

- **Question:** The brief proposed a six-digit code for classroom actions. Is that sufficient?
- **Method:** Read NIST SP 800-63B Rev. 3 and Rev. 4 for OTP length, lifetime and attempt limits; read the OWASP Authentication and MFA cheat sheets; then reasoned about the *physical* setting — a code shown on a shared, unattended classroom display.
- **What was established:** ⛔ **A six-digit code is not sufficient as identity.** The guessing space is 10⁶, which is brute-forceable in a free period without attempt limits — and NIST caps consecutive failed attempts at 100. ⚠️ A classroom display is **shared and unattended; any code shown can be photographed.** ⛔ A shared code **identifies nobody**, so an unauthorised request would be unattributable, the audit cannot answer "which teacher did this", and one person cannot be revoked without revoking everyone. ⚠️ NIST also notes single-factor authenticators should be **at least 8 digits**. ✅ **Permitted uses: authorising one specific action; or a second factor behind a real first factor.** ⛔ **Never permitted: authenticating a named teacher, or standing in for individual accounts.**
- **What remained unknown:** ❓ `CL-CODE-003` (P0) — which actions the school considers "consequential" — has not been answered, so the consequential/non-consequential split is **this project's proposal, not school policy.** ❓ Whether teachers have suitable second-factor devices at all (`CL-AUTH-013`).
- **Affected documents:** [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) §4–§5 · [`../decision_log.md`](../decision_log.md) `D-005` · [`../feasibility_report.md`](../feasibility_report.md) §12 · [`source_register.md`](source_register.md) `CR-028`
- **Confidence change:** ⛔ → `NOT_RECOMMENDED_WITHOUT_APPROVAL` as identity. 🟡 → permitted only as a one-action authorisation or second factor, **with server-side rate limiting as a P0 control, not a usability convenience.** ⛔ **A request for a robot to move something through a corridor is exactly the action that must not rest on a shared code.**

### 2026-10-11 — Sessions on an unattended shared device

- **Question:** What session controls apply to a classroom display nobody is logged into?
- **Method:** Read the OWASP Session Management cheat sheet and the MDN `Set-Cookie` reference, checking each documented limitation rather than just the headline guidance.
- **What was established:** ✅ `httpOnly`/`Secure`/`SameSite`, idle **and** absolute timeouts as **distinct** controls, server-side invalidation on logout and privilege change, no session identifier in a URL or `localStorage`. ✅ **NIST SP 800-63B Rev. 4 requires both an overall reauthentication timeout and an inactivity timeout** — not one or the other. ⚠️⛔ **`HttpOnly` has a documented limitation: the cookie is still *sent* on JavaScript-initiated requests.** It prevents *reading* the value, not the browser *sending* it.
- **What remained unknown:** ❓ Whether an unattended display may hold a teacher session **at all** — that is the school's decision (`CL-AUTH-020`), and the project's recommendation is **no**: anyone in the room would inherit teacher authority.
- **Affected documents:** [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) §9 · [`../security_and_privacy.md`](../security_and_privacy.md) §6 · [`source_register.md`](source_register.md) `CR-027`, `CR-030`
- **Confidence change:** ✅ → `VERIFIED` as best practice. ❓ School policy → `UNKNOWN`.

---

## Phase E — Storage, permissions and search

### 2026-10-11 — Permission-aware retrieval, and the two failure directions

- **Question:** Can search respect the storage's own permissions, or must CloudLight maintain a parallel permission model?
- **Method:** Read the NTFS, ACL and SID documentation; read the Elastic document/field-level security documentation **specifically for its failure modes rather than its features**; read RFC 1813 for the NFS permission model; read SQLite FTS5 for the no-second-copy constraint.
- **What was established:** ✅ **The mechanism is established and implemented by production permission-aware search products**: capture each file's ACL principals during the crawl, match them against the signed-in user's group memberships at **query time**, honour **DENY over ALLOW**, and **re-verify at open time**. ⛔ **Omitting the filter query entirely disables document-level security — it fails OPEN.** Multiple roles combine with **OR**, with the vendor's own warning that you may *"inadvertently grant wider access than intended"*. ⛔ Document-level security *"doesn't affect global index statistics that relevancy scoring uses"*, so a restricted user *"could still learn about field names and terms that only exist in inaccessible documents"*. ⚠️ **Broad SIDs are why an index built with the wrong identity is over-broad.** ⛔⛔ **Both failure directions are wrong**: an over-broad index returns files the user could not browse to; an over-narrow one hides files they can open. **Both must be tested.** ✅ **A dedicated read-only service account is mandatory** — indexing under a teacher's credentials makes every result masquerade as that teacher's access.
- **What remained unknown:** ⛔ **Whether shared storage exists at all** (`CL-NAS-001`). ❓ What it is, what filesystem, what protocol (`CL-NAS-002`, `CL-NAS-003`). ❓ Which directory authenticates against it (`CL-AUTH-001`, `CL-AUTH-002`). ❓ **Corpus size** (`CL-SEARCH-014`) — so **no search engine can be chosen**, and choosing one would be guessing at a number nobody has measured. ⛔ **No NAS was contacted. No share was mounted. No file was indexed.**
- **Affected documents:** [`../resource_search.md`](../resource_search.md) §3 · [`../resource_browser.md`](../resource_browser.md) §2–§3 · [`../decision_log.md`](../decision_log.md) `D-026` · [`source_register.md`](source_register.md) `CR-012`, `CR-013`, `CR-015`
- **Confidence change:** ✅ Mechanism → `VERIFIED` as a technique. 🧪 Its feasibility **here** → `PLAUSIBLE_REQUIRES_TESTING`. ⛔ The invariant → `NOT_RECOMMENDED_WITHOUT_APPROVAL`: **search never widens access; it can only narrow it.**

### 2026-10-11 — SMBv1 as a blocked dependency

- **Question:** What happens if the school's storage speaks only an obsolete protocol?
- **Method:** Read the SMBv1 deprecation, detection and hardening documentation and the MS-SMB2 specification.
- **What was established:** ⛔ SMBv1 is *"deprecated and should not be installed nor enabled"*; it has not been installed by default since Windows 10 1709; SMB 2.02 through 3.1.1 remain fully supported. ⚠️ SMB1-only devices are *"likely running older versions of Linux, Samba, or other types of third-party software"* that are *"no longer supported"*.
- **What remained unknown:** ❓ Which situation applies here (`CL-NAS-002`).
- **Affected documents:** [`../resource_browser.md`](../resource_browser.md) §2.2 · [`source_register.md`](source_register.md) `CR-011`
- **Confidence change:** ✅ → `VERIFIED` as a general fact. ⚠️ **The consequence is a decision, not a finding:** if the answer is SMB1-only, the resource features are not deferred — they are **cancelled**, and the recommendation is to replace the storage rather than weaken the client.

---

## Phase F — Timetable parsing, deterministic-first

### 2026-10-11 — ⛔ The strongest single finding in document parsing

- **Question:** If the school's timetable is a Word or Excel file, is parsing it a machine-learning problem?
- **Method:** Read the Microsoft Open XML structure documentation and the ISO/IEC 29500 reference, and verified the ZIP-container claim directly against the documentation's own test procedure.
- **What was established:** ✅✅ **Office Open XML documents are ZIP packages containing XML parts** — verifiable by renaming the extension from `.docx` to `.zip`. **Cell contents and cell addresses are therefore readable without any inference whatsoever.** For a school timetable in `.xlsx` or `.docx`, **there is no OCR problem at all.**
- **What remained unknown:** ⛔⛔ **What format the school's timetable is actually in.** No timetable, substitution sheet, staff list or class list has been seen. **No parser was run, because no parser exists.**
- **Affected documents:** [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §3, §3.1 · [`../decision_log.md`](../decision_log.md) `D-006` · [`source_register.md`](source_register.md) `CR-022`
- **Confidence change:** ✅ → `VERIFIED`, **but conditional on an unanswered question.** Obtaining one sample file in its real format is the highest-value single action in the parsing domain.

### 2026-10-11 — Deterministic-first, and the five-tier parsing ladder

- **Question:** Which parts of timetable processing should be deterministic, and which may involve a model?
- **Method:** Read the Docling technical report, TableFormer, the DocLayNet paper and dataset repository, and both OCR engines. Read the reports for **what their benchmarks actually measured**, not for their headline figures.
- **What was established:** ✅ **Five tiers, with deterministic structural parsing always tried first**: (1) deterministic structural parse of xlsx/CSV/PDF text layer — *no model*; (2) layout analysis, only when the table has no reliable cell structure; (3) OCR, only for image-only or handwritten input; (4) LLM field extraction into a **strict schema**, only for ambiguous fields; (5) **deterministic validation — the only tier permitted to reject.** ⛔⛔ **The Docling 225-page benchmark ran with OCR disabled**, so its attractive sub-second layout figures and its scanned-document case come from **different experiments** — and its own numbers put OCR at *"upwards of 30 seconds per page"* on CPU. ⛔ **TableFormer and DocLayNet were trained on finance, science, patents, tenders, law and manuals — not school timetables.** A merged-cell grid timetable is close to the documented tolerance limit, not the happy path. ⛔⛔ **OCR language coverage is not handwriting coverage: an unsupported script is not read poorly — it is not read.** ⛔ GPU acceleration is *"work-in-progress and largely untested"* in the source itself, so the CPU-vs-GPU comparison is `UNKNOWN`, not merely unmeasured.
- **What remained unknown:** ❓ Whether photographing boards is even permitted (`CL-SUB-006`). ❓ The timetable format. ❓ Who owns the master timetable (`CL-TIME-001`).
- **Affected documents:** [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §2–§4 · [`../feasibility_report.md`](../feasibility_report.md) §9 · [`source_register.md`](source_register.md) `CR-023`, `CR-024`, `CR-025`
- **Confidence change:** Tier 1 → `VERIFIED` as a design consequence. 🧪 Tiers 2–4 → `PLAUSIBLE_REQUIRES_TESTING`. ✅ **Three non-negotiable rules established:** deterministic parsing is always tried first; **AI never fills a missing value with a plausible guess — unknown is a valid output, a plausible invention is not**; **a model never publishes.** ⛔ **AI-assisted extraction should not be in the first release** — highest risk, lowest value, gated on an approval workflow that does not yet exist.

### 2026-10-11 — The approval gate and the staleness contract

- **Question:** Who decides that a substitution is real, and what does the display do when it does not know?
- **Method:** Reasoned from the failure modes of a wall-mounted screen in front of children, then checked against the decision log and the requirements. **No source was consulted, and none was invented.**
- **What was established:** ✅ **A record is authoritative because a person with authority approved it — not because it was photographed, parsed or extracted.** Publication requires a named human approver, recorded with the source document and the extracted values. ✅ **The staleness contract: stale content is still shown, with a visible unambiguous marker. Stale-and-shown is acceptable *only* with the marker. Stale-and-unmarked is never acceptable.** ⛔ Never a blank screen, never a frozen clock, never a confidently-wrong countdown.
- **What remained unknown:** ❓ Who owns the master timetable (`CL-TIME-001`). ❓ Who approves a substitution (`CL-SUB-001`). ❓ Whether a second person must counter-sign (`CL-SUB-002`). ❓ Whether photographing boards is permitted at all (`CL-SUB-006`).
- **Affected documents:** [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §7–§8 · [`../classroom_dashboard.md`](../classroom_dashboard.md) §6 · [`../decision_log.md`](../decision_log.md) `D-008`, `D-009`
- **Confidence change:** ⚠️ The staleness tiers and the display-state catalogue are **entirely engineering judgement** — [`../classroom_dashboard.md`](../classroom_dashboard.md) carries **no external citations at all**, by its own statement. Recorded in [`source_register.md`](source_register.md) as such rather than dressed up as a finding.

---

## Phase G — Local AI, and why it is last

### 2026-10-11 — Local inference sizing

- **Question:** Roughly what hardware would a local model need?
- **Method:** Read the Hugging Face optimisation and quantisation documentation and the QLoRA paper. Then derived the 4-bit figure rather than copying it.
- **What was established:** ✅ The load rule of thumb: `~4×X` GB at float32, `~2×X` GB at bfloat16/float16 for an `X`-billion-parameter model; 8-bit and 4-bit quantisation are established techniques. ✅ Open-weight models are routinely run locally on CPU with no GPU. ⚠️ **The `0.6–0.7 GB per billion parameters at 4-bit` figure is this project's own derivation, not a vendor constant** — and is recorded as unsourced in [`../sources.md`](../sources.md) §4 #6. ⚠️ QLoRA is a **fine-tuning** paper; inference-time residency is a different claim.
- **What remained unknown:** ❓ **Whether a GPU exists at all** (`CL-AI-009`). ❓ Whether any AI service is even permitted (`CL-AI-001`). ⛔ **No model was downloaded. No model was run.**
- **Affected documents:** [`../ai_assistant.md`](../ai_assistant.md) §6.1 · [`../network_and_hosting.md`](../network_and_hosting.md) §4.2 · [`../decision_log.md`](../decision_log.md) `D-024`
- **Confidence change:** 🧪 → `PLAUSIBLE_REQUIRES_TESTING`. ✅ **Practical conclusion: do not size a host around inference.** Size it for the core product.

### 2026-10-11 — The KV-cache trap

- **Question:** Does a model that fits when empty still fit at a realistic context length?
- **Method:** Reasoned from the architecture against the load rule of thumb. **No source states this**, and none was invented to support it.
- **What was established:** 🧪 **The KV cache remains in full precision and grows with context length.** A model that fits at zero context can fail at a realistic one. Any sizing must budget for context, not for zero.
- **What remained unknown:** Everything quantitative. **No figure is given because none can be verified without running a model.**
- **Affected documents:** [`../network_and_hosting.md`](../network_and_hosting.md) §4.2 · [`../sources.md`](../sources.md) §4 #7, §5.5
- **Confidence change:** 🧪 `PLAUSIBLE_REQUIRES_TESTING`. Recorded as a **contradiction between a widely repeated sizing rule and the architecture that rule ignores** — the kind of entry that belongs in a log rather than a footnote.

### 2026-10-11 — ⛔ Prompt injection is not fully mitigable

- **Question:** Can an AI assistant over school documents be made safe by filtering?
- **Method:** Read the OWASP GenAI LLM01 entry and the OWASP prompt-injection prevention cheat sheet, then the independent "lethal trifecta" framing.
- **What was established:** ⛔⛔ **Indirect** injection arrives *"from external sources, such as websites or files"* — and a school shared drive is precisely that. ⛔⛔ **RAG and fine-tuning do not fully mitigate it, and fool-proof prevention is unclear.** OWASP's own position is that **no mitigation is complete.** The documented scenario of an attacker modifying a document inside a RAG corpus is the realistic threat here. ✅ **The structural defence is removal, not filtering**: CloudLight's assistant has no external communication channel, so removing that leg of the trifecta removes the exfiltration path.
- **What remained unknown:** ❓ Whether the school would accept an assistant at all (`CL-AI-001`).
- **Affected documents:** [`../ai_assistant.md`](../ai_assistant.md) §8 · [`../security_and_privacy.md`](../security_and_privacy.md) §13 · [`../requirements.md`](../requirements.md) `FR-AI-05` · [`source_register.md`](source_register.md) `CR-020`, `CR-021`
- **Confidence change:** ⛔ The finding → `VERIFIED`. ⚠️ **The leap from "not fully mitigated" to "not deployable in a classroom" is this project's own judgement**, recorded in [`../sources.md`](../sources.md) §4 #10 and not attributed to OWASP.

### 2026-10-11 — The air-gap trap nobody wrote down until this pass

- **Question:** If the school is permanently offline, how does anything get installed?
- **Method:** Reasoned from the offline-first requirement against the update and inference requirements. Not found in any source; derived here.
- **What was established:** ⚠️⛔ **A permanently offline school cannot download model weights, binaries or tokenisers.** Everything must be fetched on an internet-connected machine and transferred across the air gap in advance. **A missed file breaks setup silently.** The same applies to certificates, dependencies and release artefacts.
- **What remained unknown:** ❓ Whether the school is in fact permanently offline, or whether a controlled update path exists (`CL-OTA-012`).
- **Affected documents:** [`../network_and_hosting.md`](../network_and_hosting.md) §6.3 · [`../ai_assistant.md`](../ai_assistant.md) §6
- **Confidence change:** 🧪 `PLAUSIBLE_REQUIRES_TESTING`. Recorded because it is easy to miss and expensive to discover after deployment.

---

## Phase H — Updates

### 2026-10-11 — TUF, and why "check the signature" is not a design

- **Question:** Is signature verification sufficient for updates over a school LAN?
- **Method:** Read the TUF specification and its threat-model page.
- **What was established:** ✅ TUF is a **published specification, not a product**. It defines **root, snapshot, targets and timestamp** roles with **separate keys** and requires **expiry on metadata**, so a frozen or stale repository becomes invalid on its own. ✅ Its threat model **names** rollback, freeze, fast-forward and mix-and-match. ⛔ **A signature with no expiry and no version pinning defeats none of these.** ✅ From the OWASP code-signing sheet: **the signature must be verified by the software consuming the artefact, not by the distribution point** — verification belongs **on the board**, not on the server.
- **What remained unknown:** ❓ Whether kiosk accounts can install an updating agent (`CL-OTA-010`). ❓ Whether automatic unattended deployment is permitted (`CL-OTA-012`). ❓ Where the signing key would live (`CL-OTA-005`, requires school permission).
- **Affected documents:** [`../ota_updates.md`](../ota_updates.md) §5 · [`../decision_log.md`](../decision_log.md) `D-025` · [`source_register.md`](source_register.md) `CR-032`, `CR-034`
- **Confidence change:** ✅ The general finding → `VERIFIED`. ⚠️ **TUF is presented as the reference design, not as a decision to adopt** — adopting it is significant work and is not proposed. ⛔ [`../ota_updates.md`](../ota_updates.md) §10's kiosk-elevation and Group-Policy claims **explicitly cite no vendor source**, because none was verified in that pass. **That gap was left open rather than filled with a plausible citation.**

---

## Phase I — SHANA schema review (read-only)

### 2026-10-11 — Reading SHANA's schemas without touching them

- **Question:** Does anything SHANA has published contradict CloudLight's stated boundary?
- **Method:** Read `data/schemas/README.md` and the four schema YAML files, plus SHANA's integration document. **Read-only — SHANA owns these files and another agent is working on them.**
- **What was established:** ✅ **No contradiction.** SHANA's own ownership table states the same principle CloudLight adopts: *any component whose failure could injure a person must not depend on an external system's availability.* SHANA owns robot motion, localisation, obstacle avoidance, battery management, route validation and physical emergency response — **none of which it delegates**. The schema README encodes five invariants, of which two bear directly on CloudLight: **a delivery is never `COMPLETED` without recipient confirmation**, and **no schema contains a field capable of identifying a person**. ✅ The schemas carry no motor, path, speed or actuator field — so a motor command is not merely forbidden by policy, it is **not expressible**.
- **What remained unknown:** ⛔⛔ **The proposed interface is `DESIGN_ONLY` — never implemented, never validated against a real partner, and the schemas "have never been parsed by code."** ❓ The actual interface depends entirely on what the robot vendor permits (`CL-SHANA-001`).
- **Affected documents:** [`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md) · [`../../../data/schemas/README.md`](../../../data/schemas/README.md) — **read only; neither was modified**
- **Confidence change:** ✅ The boundary → `NOT_RECOMMENDED_WITHOUT_APPROVAL`, expressed as an **absence of capability** rather than a policy. ⛔ The interface → `REQUIRES_VENDOR_OR_SERVICE_PROVIDER`.

### 2026-10-11 — SHANA offline independence

- **Question:** What happens to the classrooms if the robot subsystem fails?
- **Method:** Reasoned from the brief's hard requirement that CloudLight must operate if SHANA is offline, then checked against SHANA's stated ownership.
- **What was established:** ✅ **CloudLight operates fully when SHANA is offline.** ✅ **What the robot does when CloudLight is down is SHANA's decision, not CloudLight's specification** — and CloudLight's obligation is to design so the question never becomes load-bearing. ✅ **CloudLight must not create pressure on SHANA to degrade safety to keep a request moving.** ⛔ **A stop can never be delayed or overridden by a peer.** ⛔ **Delivery completion requires explicit recipient confirmation, never inferred from arrival.**
- **What remained unknown:** ❓ Nothing CloudLight-side. This one is a constraint, not a question.
- **Affected documents:** [`../shana_integration.md`](../shana_integration.md) · [`../architecture.md`](../architecture.md) §5 · [`../decision_log.md`](../decision_log.md) `D-003`, `D-004`
- **Confidence change:** ⛔ → `NOT_RECOMMENDED_WITHOUT_APPROVAL` for all five invariants. **None is revisable by either team.**

---

## Phase J — Register assembly

### 2026-10-11 — Consolidating the bibliography into a register

- **Question:** What has actually been researched, on what basis, and how well established is each part?
- **Method:** Extracted every URL actually cited across all CloudLight documents with `grep -oh 'https\?://…' docs/cloudlight/*.md | sort | uniq -c`, then consolidated each document's `## Sources` section into the 36 research threads in [`source_register.md`](source_register.md). **Only URLs actually cited were registered; none was invented.**
- **What was established:** ✅ 36 research threads across nine domains. ⚠️ **Three internal citations in existing CloudLight documents resolve to nothing** — `docs/infrastructure/school_network.md`, `docs/safety/risk_register.md`, `docs/safety/student_privacy.md` are cited but **not present** in the tree; and `docs/architecture/delivery_state_machine.md` does not exist. Recorded rather than repaired, because another agent owns those documents.
- **What remained unknown:** Everything that was unknown before. **Consolidation added no new knowledge** — by design.
- **Affected documents:** [`source_register.md`](source_register.md) · [`../sources.md`](../sources.md) §5.8
- **Confidence change:** None. This entry records bookkeeping, and says so.

### 2026-10-11 — Recording the accessibility gap

- **Question:** What accessibility standard governs the classroom display?
- **Method:** Searched all CloudLight documents for any accessibility citation. **Found none.**
- **What was established:** ⛔ **No WCAG, no EN 301 549, no school policy is cited anywhere in CloudLight.** The contrast ratios (4.5:1, 3:1), target sizes (44×44 px), no-hover-dependence and colour-not-sole-meaning requirements **rest on no source at all.** They were chosen by this project as arguable proposals.
- **What remained unknown:** ❓ Whether any accessibility requirement is binding here (`CL-UX-004`).
- **Affected documents:** [`../requirements.md`](../requirements.md) §3.5 · [`source_register.md`](source_register.md) `CR-035` · [`../sources.md`](../sources.md) §2.8, §4 #2
- **Confidence change:** 🧪 `PLAUSIBLE_REQUIRES_TESTING`. ✅ **This is the highest-value gap in the register and the cheapest to close.**

---

## What was NOT done, in one place

> ### ⚠️ **So that no future reader mistakes silence for a finding.**

| Not done | Consequence |
|---|---|
| ⛔ **No school network accessed, probed, scanned or pinged** | Every network property is a question, not a finding |
| ⛔ **No smartboard powered on, configured or touched** | Every Windows and kiosk behaviour claim is vendor-documented, not observed |
| ⛔ **No NAS contacted, no share mounted, no file indexed** | No storage or permission fact about this school exists |
| ⛔ **No AI service contacted, no account created, no model downloaded or run** | Every inference figure is a published rule of thumb, not a measurement |
| ⛔ **No timetable, substitution sheet, staff list or class list seen** | **No parser was run, because no parser exists** |
| ⛔ **No robot operated, connected to or commanded** | The SHANA interface claims are CloudLight-side design only |
| ⛔ **No host bought, installed or powered on; no container run; no cloud instance created** | No cost is quoted and no deployment has been attempted |
| ⛔ **No credential created, attempted or rotated; no log written; no incident occurred** | There is no system to have an incident with |
| ⛔ **No test of any kind was executed** | Every classification rests on documentation and reasoning |

⚠️ **And the standing note.** Where these documents and the school's written
policy disagree, **the school is right and these documents are wrong.**

⚠️ **Filling an `UNKNOWN` with a plausible default is the failure mode this log
exists to prevent.** Every ❓ in it is a genuine gap, and the next entry in this
log should be an answer from a person — not another inference.

**Related documents:** [`research_index.md`](research_index.md) ·
[`source_register.md`](source_register.md) ·
[`claim_evidence_matrix.md`](claim_evidence_matrix.md) ·
[`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md)