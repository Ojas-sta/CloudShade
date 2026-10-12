# CloudLight — Sources

**All accessed 2026-10-11**

> ## ⚠️ This is the CloudLight bibliography. It is NOT the SHANA bibliography.
>
> The SHANA sources live in
> `docs/references/sources.md` (the SHANA bibliography)
> and `docs/platform_selection/sources.md`.
> **That file has not been read into this one, has not been merged, and has not
> been modified.** Where a SHANA source is relevant to CloudLight it is cited from
> its own register, not copied here.

---

## 1. Method and its limits

### 1.1 What was done

| Step | Done |
|---|---|
| Vendor and standards documentation read directly | ✅ |
| Published security guidance and peer-reviewed literature read | ✅ |
| Internal cross-references between the CloudLight documents consolidated | ✅ |
| Existing CloudLight document `## Sources` sections collected and merged | ✅ |

### 1.2 What was NOT done — and this is the important part

> ### ⚠️ **Nothing in this project has been hardware-verified on school equipment.**

| Not done | Consequence |
|---|---|
| **No school network accessed, probed, scanned or pinged** | Every network property is a question, not a finding |
| **No smartboard powered on, configured or touched** | Every Windows and kiosk behaviour claim is vendor-documented, not observed |
| **No robot operated, connected to or commanded** | The SHANA interface claims here are CloudLight-side design only |
| **No external AI service contacted, no model downloaded, no model run** | Every inference figure is a published rule of thumb, not a measurement |
| **No school data sent anywhere; no school document read** | No claim about this school's timetable, NAS or files exists |
| **No timetable, substitution sheet, staff list or class list seen** | No parser was run, because no parser exists |
| **No host bought, installed or powered on; no container run; no cloud instance created** | No cost is quoted and no deployment has been attempted |

### 1.3 Rules applied to every entry

1. **Only URLs cited in a sibling CloudLight document appear in §2.** This file consolidates; it does not expand. A source nobody consulted is not a source.
2. **Vendor marketing claims are labelled as such** and are not relied on where a specification exists.
3. **`NOT FOUND` means searched and not located. It is not evidence of absence.** See §3.
4. **Contradictions are recorded in §5, not silently resolved.**
5. **Statements that rest on no source at all are listed in §4**, so nobody mistakes engineering judgement for a verified finding.

### 1.4 Type vocabulary

| Type | Meaning |
|---|---|
| `OFFICIAL-MFR` | Manufacturer or vendor documentation |
| `OFFICIAL-STD` | Published standard: IETF RFC, ISO, W3C |
| `SECURITY-STD` | Government or standards-body security guidance |
| `SECURITY-REF` | Independent security community guidance |
| `PEER-REVIEWED` | Refereed publication |
| `COMMUNITY` | Maintained open-source project documentation, or an unreviewed preprint |
| `INDEPENDENT` | Named independent practitioner or researcher |
| `INTERNAL` | This repository |

⚠️ **This vocabulary is not identical to the SHANA register's**, which uses
`OFFICIAL-MFR · OFFICIAL-GOV · OFFICIAL-CODE · STD · LEGAL · PEER-REVIEWED ·
INDEPENDENT · COMMUNITY · AGGREGATOR`. Merging the two would require a decision
about the whole repository, and that is not this file's job.

---

## 2. Sources by domain

⚠️ **Access date for every external source: 2026-10-11.** Consolidation of the
`## Sources` sections of [`classroom_dashboard.md`](classroom_dashboard.md),
[`windows_startup.md`](windows_startup.md), [`timetable_and_substitutions.md`](timetable_and_substitutions.md),
[`resource_browser.md`](resource_browser.md), [`resource_search.md`](resource_search.md),
[`ai_assistant.md`](ai_assistant.md),
[`authentication_and_authorisation.md`](authentication_and_authorisation.md),
[`security_and_privacy.md`](security_and_privacy.md),
[`network_and_hosting.md`](network_and_hosting.md), [`ota_updates.md`](ota_updates.md)
and [`shana_integration.md`](shana_integration.md).

⚠️ **[`classroom_dashboard.md`](classroom_dashboard.md) carries no external
citations at all** — by its own statement, *"no claim here rests on one"*. Its
display states, countdown tiers and failure catalogue are therefore **entirely
engineering judgement** and appear in §4.

### 2.1 Windows smartboards, kiosk mode and device management

| # | Title | URL | Type | What it supports | Limitations |
|---|---|---|---|---|---|
| 1 | **Windows Single-App and Multi-App Kiosk Configuration Options Overview** | `https://learn.microsoft.com/en-us/windows/configuration/kiosk/` | OFFICIAL-MFR | Edition support for Assigned Access and Shell Launcher; that Shell Launcher and Assigned Access cannot both be configured; Shell Launcher replaces `explorer.exe` | ⛔ Not this school's hardware. Edge is given no kiosk experience by default — a support cost, not a free one |
| 2 | **Configure a Single-App Kiosk with Assigned Access** | `https://learn.microsoft.com/en-us/windows/configuration/assigned-access/configure-single-app-kiosk` | OFFICIAL-MFR | `Set-AssignedAccess`; the default `Ctrl+Alt+Del` breakout sequence and its configurability; `LogonUI` `IdleTimeOut` default 30 s; the `IdleTimeOut` policy does **not** apply to Edge kiosk mode | ⛔ Breakout is a documented escape hatch — a lockdown decision, not a technical limit. `CL-WIN-*` unanswered |
| 3 | **Create a Shell Launcher configuration file** | `https://learn.microsoft.com/en-us/windows/configuration/assigned-access/shell-launcher/configuration-file` | OFFICIAL-MFR | `ReturnCodeActions` with `RestartShell` / `RestartDevice` / `ShutdownDevice` / `DoNothing`; the four-custom-actions-plus-one-default limit; `AutoLogonAccount` creating a local standard `Kiosk` user | Vendor warns that exceeding the action limit results in **nothing happening** |
| 4 | **Configure Shell Launcher** | `https://learn.microsoft.com/en-us/windows/configuration/shell-launcher/configure` | OFFICIAL-MFR | Shell Launcher as an optional component enabled by the Assigned Access CSP; OMA-URI `./Vendor/MSFT/AssignedAccess/ShellLauncher` | Entail that someone with the right privilege must apply it |
| 5 | **Quickstart: Configure a kiosk experience with Shell Launcher** | `https://learn.microsoft.com/en-us/windows/configuration/shell-launcher/quickstart-kiosk` | OFFICIAL-MFR | The canonical Edge-in-Shell-Launcher command line including `--kiosk-idle-timeout-minutes=2`; the MDM Bridge application requiring SYSTEM | A **quickstart**, i.e. a worked example, not a normative specification |
| 6 | **Configure Microsoft Edge kiosk mode** | `https://learn.microsoft.com/en-us/deployedge/microsoft-edge-configure-kiosk-mode` | OFFICIAL-MFR | The two lockdown experiences (Digital/Interactive Signage, Public-Browsing); ⛔ **both run an InPrivate session with autofill disabled**; `--kiosk`, `--edge-kiosk-type=fullscreen`, `--edge-kiosk-type=public-browsing`, `--no-first-run`; **`--kiosk-idle-timeout-minutes` defaults (full-screen 0, public browsing 5), range 0 / 1–1440**; ⛔ **the idle timeout does not restart Edge — Assigned Access or Shell Launcher is required**; the feature support matrix (F11/F12/CTRL+N/CTRL+T blocked, multi-tab, `edge://` blocking); the kiosk-mode Group Policy table | ⚠️ **This is the source of the contradiction in §5.1.** "InPrivate session" is the documented behaviour; whether persistent offline caching survives it on this hardware is untested. `CL-WIN-*`, `CL-OTA-011` |
| 7 | **Kiosk settings for Windows in Microsoft Intune** | `https://learn.microsoft.com/en-us/intune/intune-service/configuration/kiosk-settings-windows` | OFFICIAL-MFR | An Intune-delivered kiosk profile: Edge Kiosk URL, kiosk mode type, "Refresh browser after idle time" 0–1440 minutes | ⚠️ Requires Intune. ❓ Whether this school uses any MDM at all — `CL-AUTH-018`, `CL-WIN-*` |
| 8 | **Windows IoT Enterprise LTSC in Volume License** | `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/deployment/volume-license` | OFFICIAL-MFR | The verbatim fixed-function restriction: *"intended for use for fixed-function specialized devices only and cannot be used as a replacement of Windows desktop for general purpose computing"*; upgrade-licence-only availability | ⚠️ **A licensing constraint, not a technical one.** ❓ Volume Licensing availability at this school — `CL-BUD-005` |
| 9 | **Windows IoT Enterprise LTSC 2021 release notes** | `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/whats-new/windows-10-iot-enterprise-ltsc-2021` | OFFICIAL-MFR | Build 19044; servicing dates 2021-11-16 to 2032-01-13; Fixed Lifecycle Policy; equivalence to Windows 10 21H2 | Lifecycle end date is a **hard horizon**, not a preference |
| 10 | **Windows IoT Enterprise release history** | `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/whats-new/release-history` | OFFICIAL-MFR | Confirmation of lifecycle dates and the fixed-purpose-device framing | Historical record; version applicability varies |
| 11 | **Unified Write Filter — overlay location and size** | `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/customize/uwfoverlay` | OFFICIAL-MFR | Overlay behaviour; RAM overlay cleared on reboot; 1024 MB default size; ⛔ **overlay exhaustion causing failed writes and an unresponsive device**; ⛔ **shutdown taking "up to an hour or longer" when the overlay is full** | ⚠️ The case against enabling UWF casually. A powerful lockdown tool that can make a classroom device unbootable-slow |
| 12 | **`UWF_Filter` WMI provider** | `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/customize/uwf-filter` | OFFICIAL-MFR | Edition support table (Home No, Pro No, Enterprise Yes, Education Yes, IoT Enterprise Yes); administrator account required to change UWF configuration | ⛔ **Not available on Windows Home or Pro.** If any board runs Pro, UWF is simply not an option |
| 13 | **Interactive Services (Win32)** | `https://learn.microsoft.com/en-us/windows/win32/services/interactive-services` | OFFICIAL-MFR | ⛔ **Services cannot directly interact with a user since Windows Vista**; all services run in Terminal Services session 0; `NoInteractiveServices` defaults to 1; the `CreateProcessAsUser` workaround and its risk | ⚠️ **This is why a Windows service cannot be the kiosk launcher.** It is a platform limit, not a policy choice |
| 14 | **Task Scheduler for developers** | `https://learn.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-start-page` | OFFICIAL-MFR | Documented triggers — at logon, at boot, on idle, on event, on registration, on session state change | ⚠️ Documents the mechanism; does not guarantee behaviour on this estate |
| 15 | **`ITaskSettings::put_RestartCount`** | `https://learn.microsoft.com/en-us/windows/win32/api/taskschd/nf-taskschd-itasksettings-put_restartcount` | OFFICIAL-MFR | Restart-on-failure as a first-class Task Scheduler setting | API reference; no guarantee the school permits it to be configured |
| 16 | **`schtasks` commands** | `https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks` | OFFICIAL-MFR | The Administrators group is required to schedule, view or change tasks | ⛔ Requires local admin. `CL-WIN-003` is unanswered, so this is unknown in practice |
| 17 | **Mixed content (MDN)** | `https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content` | OFFICIAL-STD | Upgradable vs blockable content; ⛔ **CSS `url()` values and web fonts can be blockable**; ⛔ **mixed content to an IP-address host is blocked rather than upgraded**; localhost treated as a secure origin | ⛔ **Directly relevant to `network_and_hosting.md` §5.5**: an IP-address-based deployment can fail to load assets, while a hostname-based one does not |
| 18 | **`Set-Cookie` header (MDN)** | `https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie` | OFFICIAL-STD | `HttpOnly` **and its documented limitation — the cookie is still sent on JavaScript-initiated requests**; `SameSite=Strict/Lax/None`; `Secure`; the `__Host-` prefix requirements | ⚠️ `HttpOnly` is not "invisible to scripts". It prevents *reading* the value, not the browser *sending* it |

### 2.2 Local networking, SMB/CIFS and storage

| # | Title | URL | Type | What it supports | Limitations |
|---|---|---|---|---|---|
| 19 | **Client Isolation — Aruba Central** | `https://arubanetworking.hpe.com/techdocs/central/2.5.7/content/aos10x/cfg/aps/client-isolation.htm` | OFFICIAL-MFR | Isolation *"isolates clients from one another and disables all peer-to-peer communication within the network"*, allowing *"only client to gateway traffic"*; unlisted destinations are dropped | ⚠️ **Vendor-specific.** ❓ Whether this school runs Aruba — `CL-NET-008` is P1 and `UNANSWERED` |
| 20 | **Configuring Client Isolation / Wireless Network Profiles — Aruba Central** | `https://arubanetworking.hpe.com/techdocs/central/latest/content/nms/access-points/cfg/networks/conf_wlan_ssid.htm` | OFFICIAL-MFR | "Deny inter-user bridging" and "Deny Intra VLAN Traffic" are **per-SSID settings**; *"the clients can connect to the Internet, but cannot communicate with each other"* | ⚠️ **Per-SSID**, so it may be enabled on one SSID and not another. Only IT can tell us which |
| 21 | **About AP Client Isolation — WatchGuard** | `https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Firmware/wireless/ap_station_isolation_c.html` | OFFICIAL-MFR | Isolation blocks clients on the same AP, across radios and across APs; on firmware 8.6.x+ also blocks communication *"with wired-side hosts on the same network"* | ⚠️ ⛔ **This is the finding that breaks a local-first design while the board still shows full Wi-Fi signal.** Vendor-specific |
| 22 | **Wireless LAN Configuration Guide, Cisco IOS XE — WLAN Overview** | `https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/wlan/configuration/xe-16-7/wl-16-7-book/wl-overview.html` | OFFICIAL-MFR | An access point serves *"as the connection point between wireless and wired networks"* and operates *"in bridging and routing mode"* | ⚠️ Correct for the cited IOS XE release. ✅ This is the source of the correction that an AP is a Layer-2 bridge and **not** an application host |
| 23 | **Container Manager — Synology** | `https://www.synology.com/en-us/dsm/feature/docker` | OFFICIAL-MFR | Containers run *"directly on your NAS"*; Compose-based multi-container projects; CPU/RAM/network monitoring | ⚠️ A **product feature page** — marketing framing. ❓ Availability and limits vary by model and DSM version. `CL-NAS-001`, `CL-NAS-002` |
| 24 | **Advanced — Login Portal, DSM 7 (Synology Knowledge Center)** | `https://kb.synology.com/en-global/DSM/help/DSM/AdminCenter/system_login_portal_advanced?version=7` | OFFICIAL-MFR | *"Your Synology NAS can act as a reverse proxy server"*; source and destination protocol/hostname/port; HSTS; selectable **access control profiles** restricting by source IP | ⚠️ **Versioned to DSM 7.** The feature moved between generations (Application Portal in DSM 6 → Login Portal in DSM 7). ❓ Unknown which DSM this school runs |
| 25 | **Login Portal — DSM 7 (Synology Knowledge Center)** | `https://kb.synology.com/en-global/DSM/help/DSM/AdminCenter/system_login_portal` | OFFICIAL-MFR | Custom alias/domain; reverse-proxy and access-control-profile management | As above |
| 26 | **SMBv1 is not installed by default in Windows 10/Server** | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/smbv1-not-installed-by-default-in-windows` | OFFICIAL-MFR | SMBv1 *"no longer installed by default"* since 1709; deprecated 2014; ⛔ *"SMB1 is deprecated and should not be installed nor enabled"*; SMB 2.02/2.1/3.0/3.02/3.1.1 still *"fully supported"*; SMB1-only devices *"likely running older versions of Linux, Samba, or other types of third-party software"* that are *"no longer supported"* | ⛔ **This is a blocked dependency, not a workaround.** If the school NAS speaks only SMB1, we cannot fix it and must not try |
| 27 | **Detect, enable, and disable SMBv1, SMBv2, and SMBv3 in Windows** | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/detect-enable-and-disable-smbv1-v2-v3` | OFFICIAL-MFR | *"SMBv1 isn't installed by default in any edition of Windows 11 or Windows Server 2019 and later versions"*; Microsoft recommends updating the server rather than reinstalling SMBv1 | Troubleshooting guidance, not an architecture decision |
| 28 | **SMB security hardening in Windows Server and Windows Client** | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-security-hardening` | OFFICIAL-MFR | SMB signing required by default on Windows 11 24H2 / Server 2025; negotiation *"from SMB 2.0.2 to 3.1.1"*; encryption available from SMB 3.0; ⛔ SMB Remote Mailslots *"doesn't provide any authentication or authorization mechanisms"* | Version-specific defaults. ⚠️ An encrypted share may not work against an older NAS — untested |
| 29 | **[MS-SMB2]: Server Message Block Protocol Versions 2 and 3** | `https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/5606ad47-5ee0-437a-817e-70c366052962` | OFFICIAL-MFR | The authoritative SMB 2 / SMB 3 protocol specification | Specification, not user documentation |
| 30 | **NTFS overview** | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview` | OFFICIAL-MFR | *"assign detailed permissions to files and folders using Access Control Lists (ACLs) … specify which users and groups have access"*; NTFS carries security descriptors, encryption and quotas | Windows-specific. ❓ This school's storage is NTFS — `CL-NAS-002` |
| 31 | **Access Control Lists** | `https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists` | OFFICIAL-MFR | A DACL *"identifies the trustees that are allowed or denied access"*; *"The system checks the ACEs in sequence until it finds one or more ACEs that allow all the requested access rights, or until any of the requested rights is denied"* | ⛔ **The basis of DENY overriding ALLOW.** Evaluation order matters more than the intuitive reading |
| 32 | **Security Identifiers** | `https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers` | OFFICIAL-MFR | A SID *"is used to uniquely identify a security principal or security group"*; never reused; `S-1-1-0` is *"a group that includes all users"*; `S-1-5-11` is all authenticated users and computers | ⚠️ **These broad principals are why an index built with the wrong identity is over-broad.** Identity source itself is `❓ UNKNOWN` — `CL-AUTH-001`, `CL-AUTH-002` |
| 33 | **RFC 1813 — NFS Version 3 Protocol Specification** | `https://www.rfc-editor.org/rfc/rfc1813` | OFFICIAL-STD | The NFSv3 `ACCESS` procedure as an explicit over-the-wire permission check; `AUTH_UNIX` carries *"the user's effective user ID, effective group ID and groups with each call"* and the server checks permissions from them | ⚠️ `AUTH_UNIX` is **weak identity** — the client asserts its own IDs. Named by the IANA as obsolete. Listed as a second model to understand, **not** a recommendation |
| 34 | **SQLite: Is SQLite Transactional** | `https://sqlite.org/transactional.html` | COMMUNITY | Transactions are *"atomic, consistent, isolated, and durable"* even across a power failure; *"all changes within a single transaction either occur completely or not at all"* | A specific single-file engine. Choosing it is a decision that has not been formally made |
| 35 | **SQLite: Atomic Commit** | `https://sqlite.org/atomiccommit.html` | COMMUNITY | ⛔ **The honest limit:** a power loss mid-sector-write *"opens SQLite up to the possibility of database corruption"*, which SQLite cannot test for or remedy | ⚠️ **Read together with source 34, this is the whole story.** Do not cite 34 without 35 |
| 36 | **SQLite: Write-Ahead Logging** | `https://www3.sqlite.org/wal.html` | COMMUNITY | Without syncing content to disk on commit, *"transactions are no longer durable and might rollback following a power failure"* | The durability setting must be chosen deliberately, not defaulted |

### 2.3 Search, indexing and permission-aware retrieval

| # | Title | URL | Type | What it supports | Limitations |
|---|---|---|---|---|---|
| 37 | **Controlling access at the document and field level — Elastic** | `https://www.elastic.co/docs/deploy-manage/users-roles/cluster-or-deployment-auth/controlling-access-at-document-field-level` | OFFICIAL-MFR | A real implementation of the pattern: per-role query templating against the authenticated user; ⛔ **"Omitting the `query` parameter entirely disables document level security"** (fails **open**); multiple roles combine with **OR**, with Elastic's own warning of *"inadvertently grant wider access than intended"*; document-level security *"doesn't affect global index statistics that relevancy scoring uses"*; a restricted user *"could still learn about field names and terms that only exist in inaccessible documents"* | ⚠️ **Product-specific, and deliberately used as a cautionary example rather than a selection.** Elastic is not a decided technology. The three failure modes are general |
| 38 | **SQLite FTS5 Extension** | `https://sqlite.org/fts5.html` | COMMUNITY | `bm25()` ranking; `snippet()` / `highlight()`; prefix and Boolean/NEAR queries; `content=''` and `content='table'` modes letting an index exist **without a second copy of the source document**; `unicode61` default diacritic removal; ⛔ the porter stemmer is documented as limited to *"English language terms only"* | ⚠️ ⛔ **The English-only stemmer is a hard limit for a multilingual school.** ❓ `CL-UX-016`, `CL-UX-017` are unanswered |
| 39 | **Apache Tika** | `https://tika.apache.org/` | COMMUNITY | *"detects and extracts metadata and text from over a thousand different file types … useful for search engine indexing"*; 4.x moves parsing into *"crash-isolated forked processes"* | Named as an **example**, not a selection. ⚠️ No extraction has been run against any school document |
| 40 | **Tesseract OCR** | `https://github.com/tesseract-ocr/tesseract` | COMMUNITY | *"can recognize more than 100 languages"*; PNG/JPEG/TIFF input; the project's own advice that *"in many cases, in order to get better OCR results, you'll need to improve the quality of the image"*; training data must be supplied per language | Named as an **example**. ⛔ **An unsupported script is not read poorly — it is not read.** Vendor-reported language counts say nothing about handwriting or board conditions |
| 41 | **Inotify — a powerful yet simple file change notification system** | `https://www.kernel.org/doc/html/latest/filesystems/inotify.html` | COMMUNITY | Kernel file-change notification as a mechanism; watches attached to an inotify instance; ordering guaranteed by a single queue | ⚠️ ⛔ **Not** evidence that this school's storage, OS or host supports it. ❌ Over NFS the mechanism does not work as people expect — a limitation not addressed by this source |

### 2.4 Local AI inference and retrieval-augmented generation

| # | Title | URL | Type | What it supports | Limitations |
|---|---|---|---|---|---|
| 42 | **Optimizing LLMs for Speed and Memory — Hugging Face** | `https://huggingface.co/docs/transformers/llm_tutorial_optimization` | COMMUNITY | The load rule of thumb (`~4×X` GB float32, `~2×X` GB bfloat16/float16 for an `X`-billion-parameter model); 8-bit and 4-bit quantisation as the path to smaller-hardware inference | ⚠️ **A general rule of thumb for loading, not a performance guarantee.** The `0.6–0.7 GB per billion parameters at 4-bit` figure used elsewhere is **our own derivation**, not a vendor constant |
| 43 | **Quantization overview — Hugging Face Transformers** | `https://huggingface.co/docs/transformers/quantization/overview` | COMMUNITY | 4-bit and 8-bit quantisation as established techniques for large models on limited hardware | Establishes the technique; says nothing about quality loss on a specific model |
| 44 | **QLoRA: Efficient Finetuning of Quantized LLMs — Dettmers et al.** | `https://arxiv.org/abs/2305.14314` | PEER-REVIEWED | 4-bit quantisation as an established research technique for running capable models on limited hardware | ⚠️ A **fine-tuning** paper. Inference-time memory behaviour is a different claim |
| 45 | **Feature extraction task — Hugging Face Transformers** | `https://huggingface.co/docs/transformers/en/tasks/feature_extraction` | COMMUNITY | Text embedding models as a standard, small, widely available model class | ⚠️ Embeddings are **multilingual only if the model is**. ❓ `CL-UX-016`, `CL-UX-017` unanswered |
| 46 | **Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks — Lewis et al.** | `https://arxiv.org/abs/2005.11401` | PEER-REVIEWED | The retrieval-plus-generation architecture; the basis for treating retrieval quality as the limiting factor on answer quality | A general result. ⚠️ Says nothing about a school corpus's retrievability |
| 47 | **Recursive character text splitting — LangChain** | `https://python.langchain.com/docs/how_to/recursive_text_splitter/` | COMMUNITY | Chunking as a parameterised, tuned choice with explicit chunk size and overlap, not a default | ⚠️ **Vendor-documented framework guidance.** Chunk parameters have no defensible default for this corpus because the corpus does not exist |
| 48 | **llama.cpp** | `https://github.com/ggml-org/llama.cpp` | COMMUNITY | Open-weight models are routinely run locally on CPU, with no GPU | ⚠️ A project README, not a benchmark on this hardware. ❓ `CL-AI-009` is unanswered, so we do not know if there is a GPU at all |
| 49 | **Ollama** | `https://github.com/ollama/ollama` | COMMUNITY | Local model serving on consumer hardware is packaged and routine | As above. Named as evidence that the approach is **ordinary**, not that it is **sufficient** |
| 50 | **LLM01:2025 Prompt Injection — OWASP Gen AI Security Project** | `https://genai.owasp.org/llmrisk/llm01-prompt-injection/` | SECURITY-REF | ⛔ **Indirect** injection as input *"from external sources, such as websites or files"*; ⛔ **RAG and fine-tuning do not fully mitigate it** and fool-proof prevention is unclear; scenario #4, an attacker modifying a document in a RAG corpus; mitigations include *"use deterministic code to validate adherence"* to output formats, handling functions *"in code rather than providing them to the model"*, requiring human approval for high-risk actions, and *"segregate and identify external content"* | ⚠️ ⛔ **The "does not fully mitigate" finding is the important one.** It means the AI assistant cannot be made safe by filtering alone — see §4 |
| 51 | **The lethal trifecta for AI agents — Simon Willison** | `https://simonwillison.net/2025/Jun/16/the-lethal-trifecta` | INDEPENDENT | The three-property framing — private data, untrusted content, external communication — and the conclusion that **removing one leg is the reliable structural defence** | ⚠️ **A named practitioner's framing, not a standard.** Adopted here precisely because it argues for a *structural* boundary (no external communication ⇒ no leak) rather than filtering |

### 2.5 Document parsing, layout analysis, OCR and table structure

| # | Title | URL | Type | What it supports | Limitations |
|---|---|---|---|---|---|
| 52 | **Docling Technical Report** — Auer, Lysak, Nassar, Dolfi et al., IBM Research | `https://arxiv.org/abs/2408.09869` | COMMUNITY | Pipeline order; PDF backend *"retrieves the programmatic text tokens, consisting of string content and its coordinates on the page"*; ⛔ OCR is **optional**, *"e.g. for scanned PDFs"*; **layout detection at 72 dpi on a single CPU with sub-second latency**; **TableFormer 2–6 s per table on a standard CPU**; **EasyOCR *"upwards of 30 seconds per page"* on CPU; the 225-page benchmark run **with OCR disabled**; ⛔ GPU acceleration *"work-in-progress and largely untested"* | ⚠️ **An unreviewed preprint.** ⚠️ **Its own benchmark ran with OCR disabled**, so its layout and table figures say little about a scanned or handwritten school document — see §5.4 |
| 53 | **TableFormer: Table Structure Understanding with Transformers** — Nassar et al., CVPR 2022 | `https://arxiv.org/abs/2203.01017` | PEER-REVIEWED | Predicting logical row/column structure and header/body roles; documented tolerance of *"partial or no borderlines, empty cells, rows or columns, cell spans"* | ⛔ **Trained on document corpora that are not school timetables** (see source 54). A merged-cell grid timetable is close to its worst case |
| 54 | **DocLayNet: A Large Human-Annotated Dataset for Document-Layout Analysis** — Pfitzmann et al., KDD 2022 | `https://arxiv.org/abs/2206.01062` | PEER-REVIEWED | That layout analysis is trained on a **labelled document-layout corpus** — 80,863 human-annotated pages, 11 layout classes | ⛔ The corpus is finance, science, patents, tenders, law and manuals. **Not school timetables.** This is the strongest available argument against trusting general layout benchmarks |
| 55 | **DS4SD/DocLayNet dataset repository** | `https://github.com/DS4SD/DocLayNet` | COMMUNITY | Human annotation by trained experts; fixed splits; per-page bounding boxes and digital text cells with coordinates | As above — a dataset card, and its *purpose* is to show the training distribution |
| 56 | **EasyOCR** | `https://github.com/JaidedAI/EasyOCR` | COMMUNITY | *"Ready-to-use ocr with 80+ supported languages."* | ⛔ **Language count says nothing about handwriting.** An unsupported script is not read poorly — it is not read |
| 57 | **Structure of a WordprocessingML document** — Microsoft Learn | `https://learn.microsoft.com/en-us/office/open-xml/word/structure-of-a-wordprocessingml-document` | OFFICIAL-MFR | ✅ That Office Open XML documents are **ZIP packages containing XML parts** — *"rename the extension on the minimum document from .docx to .zip"*. Cell contents and addresses are therefore **readable without inference** | ✅ This is the strongest single finding in document parsing for this project: **if the timetable is a `.docx` or `.xlsx`, there is no OCR problem at all** |
| 58 | **ISO/IEC 29500 — Office Open XML File Formats** | `https://www.iso.org/standard/71691.html` | OFFICIAL-STD | The underlying standard for the OOXML package format | Paywalled. Cited as the normative reference behind source 57 |
| 59 | **ISO 32000-1 (PDF)** — published copy, Adobe Acrobat SDK documentation portal | `https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf` | OFFICIAL-STD | PDF page content streams carry text-showing operators and graphics; ⛔ **a scanned page is an image with no character data to retrieve** | The published copy may lag the current revision. The *principle* — text layer vs image-only — is stable |
| 60 | **RFC 4180 — Common Format and MIME Type for CSV** | `https://www.rfc-editor.org/rfc/rfc4180` | OFFICIAL-STD | Quoting and escaping rules; a field containing a comma, a quote or a line break **must** be quoted | ⚠️ **A school spreadsheet exported to CSV will contain commas and newlines inside cells.** This is a common, silent data-corruption failure |
| 61 | **Create, view, or download a file** — Google Docs Editors Help | `https://support.google.com/docs/answer/49114` | OFFICIAL-MFR | That a Google Sheets file is exported as a chosen **file type** — a point-in-time download, not a live link | Support documentation, not an API contract. Supports the snapshot-drift failure mode: the export is stale the moment it is taken |

### 2.6 Authentication, authorisation, sessions, MFA and secrets

| # | Title | URL | Type | What it supports | Limitations |
|---|---|---|---|---|---|
| 62 | **Digital Identity Guidelines: Authentication and Lifecycle Management (NIST SP 800-63B)** | `https://pages.nist.gov/800-63-3/sp800-63b.html` | SECURITY-STD | Failed attempts limited to **no more than 100 consecutive attempts**; ⛔ **SMS/PSTN designated a "restricted" authenticator** (SS7, SIM-swap, number-porting); OTP validity **ten minutes or less**, single-use, single-factor authenticators of **at least 8 digits**; AAL2 requires two distinct factors | ⚠️ **Rev. 3.** Cited by [`authentication_and_authorisation.md`](authentication_and_authorisation.md); [`security_and_privacy.md`](security_and_privacy.md) cites Rev. 4. ⚠️ See §5.3 — **the two revisions disagree on timeouts** |
| 63 | **Digital Identity Guidelines (NIST SP 800-63B Rev. 4)** | `https://pages.nist.gov/800-63-4/sp800-63b.html` | SECURITY-STD | ✅ **Both an overall reauthentication timeout and an inactivity timeout are required, not one or the other** — AAL1 inactivity *"MAY be applied but is not required"*; AAL2 overall ≤ 24 h, inactivity ≤ 1 h. Rate-limiting is a **SHALL** at every AAL. Passwords as single-factor *"SHALL be a minimum of 15 characters"*; verifiers *"SHALL NOT require subscribers to change passwords periodically"* but *"SHALL force a change if there is evidence that the authenticator has been compromised"*. The guidelines *"are not intended to constrain the development or use of standards outside of this purpose"* | ⚠️ **Rev. 4.** ⚠️ **US federal guidance.** Whether it is binding at an Indian school is `❓ UNKNOWN` — `CL-AUTH-016`. Adopted as good practice, not as law |
| 64 | **Authentication Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html` | SECURITY-REF | MFA as the primary defence against credential compromise; single-use codes; short OTP lifetime; rate limiting; OTP secrets not stored in plaintext; **cryptographically secure** random-number generation for tokens | Community guidance. ⚠️ A cheat sheet is a summary, not a specification |
| 65 | **Multifactor Authentication Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html` | SECURITY-REF | Factor strength ranking; TOTP, push and SMS characteristics; ⛔ **number matching against push-bombing / MFA-fatigue attacks** | As above |
| 66 | **Session Management Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html` | SECURITY-REF | `httpOnly`, `Secure`, `SameSite`; ⛔ no session identifier in the URL; ⛔ none in `localStorage`; session ID rotation on authentication; **server-side invalidation on logout and privilege change**; idle **and** absolute timeout as **distinct** controls | ⛔ **Directly relevant to the unattended-display problem**: an open kiosk session is a session with no human present to end it |
| 67 | **Access Control Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html` | SECURITY-REF | Least privilege, separation of duties, deny by default, and **access control decisions made server-side on every request**, not once at sign-in | As above |
| 68 | **Authorization Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html` | SECURITY-REF | ⛔ *"Never rely on client-side access control checks"*; validate permissions on every request; deny by default with a single server-side default-deny policy | As above. ⚠️ **Not the same document as source 67** — they are cited separately because they are |
| 69 | **Secrets Management Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html` | SECURITY-REF | Secrets supplied at runtime, never in source control; the *"rotate, never reuse"* lifecycle; and the central point that ⛔ **a secret in version control must be assumed compromised** | ⚠️ Directly reinforces the finding already recorded in `start_opencode_server.sh` |
| 70 | **Logging Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html` | SECURITY-REF | ✅ *"Data should not normally be logged"*, with credentials among the excluded items; log-injection prevention; ⛔ **log access is itself access-controlled data** | Community guidance |
| 71 | **File Upload Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html` | SECURITY-REF | ⛔ *"ZIP bombs, XML bombs … or simply huge files in a way to fill the server storage"*; *"don't trust the Content-Type header as it can be spoofed"*; size limits computed **after** decompression; ✅ **store uploads outside the webroot and never execute or interpret them server-side**; antivirus or sandbox | ⚠️ Written for uploads; applied here by analogy to extraction from storage. The analogy is ours |
| 72 | **Code Signing Cheat Sheet — OWASP** | `https://cheatsheetseries.owasp.org/cheatsheets/Code_Signing_Cheat_Sheet.html` | SECURITY-REF | ✅ **the signature must be verified by the software consuming the artefact**, not by the distribution point; the private key is the crown jewel; key compromise means every artefact is suspect | ⚠️ This is the source of the distinction that matters for OTA: verification on the *board*, not on the server |
| 73 | **Web Authentication API: An Introduction — W3C TAG** | `https://www.w3.org/2021/03/bringing-passwordless-login-with-webauthn.html` | OFFICIAL-STD | WebAuthn passkeys are **origin-bound and phishing-resistant**, and the private key never leaves the authenticator | ⚠️ **A W3C TAG article, not the specification.** ❓ Whether school devices support passkeys at all — `CL-AUTH-013` is unanswered |
| 74 | **Incident Response Recommendations … (NIST SP 800-61 Rev. 3)** | `https://csrc.nist.gov/pubs/sp/800/61/r3/final` | SECURITY-STD | Incident-response preparation integrated with organisational risk management; the importance of a **defined, named response capability**. Published April 2025, superseding Rev. 2 (2012) | ⚠️ No incident has occurred, because no system exists. `CL-SEC-026` asks who this school's named contact is |

### 2.7 Software update security

| # | Title | URL | Type | What it supports | Limitations |
|---|---|---|---|---|---|
| 75 | **The Update Framework (TUF) — Specification** | `https://theupdateframework.io/specification/latest/` | OFFICIAL-STD | TUF is a **published specification, not a product**; it defines **root, snapshot, targets and timestamp** roles with **separate keys**, and requires **expiry** on metadata, so a frozen or stale repository state becomes **invalid on its own** | ⚠️ **Presented as the reference design, not as a decision to adopt.** Adopting TUF is a significant piece of work and is not proposed |
| 76 | **The Update Framework — About** | `https://theupdateframework.io/about/` | OFFICIAL-STD | TUF's stated threat model **names** the rollback, freeze, fast-forward and mix-and-match attacks it exists to resist | ⚠️ ✅ **This is the point**: "check the signature" alone is *not* the design. A signature with no expiry and no version pinning does not defeat rollback |

⚠️ **On Windows platform claims for updates.** [`ota_updates.md`](ota_updates.md)
§10 discusses kiosk elevation, scheduled tasks and Group Policy constraints and
**explicitly cites no vendor source for them**, because none was verified during
that pass. That is a deliberate gap, recorded rather than filled.

### 2.8 Accessibility and web UI

> ### ⚠️ **This domain is thin, and that is an honest finding rather than an omission.**

**No accessibility standard — no WCAG, no EN 301 549, no school policy — is cited
anywhere in the CloudLight documents.** The accessibility requirements in
[`requirements.md`](requirements.md) §3.5 (contrast ratios, target sizes, no
hover dependence, colour not the sole carrier of meaning) therefore rest on **no
source at all** and appear in §4.

| # | Title | URL | Type | What it supports | Limitations |
|---|---|---|---|---|---|
| 77 | **Mixed content (MDN)** | `https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content` | OFFICIAL-STD | A web-UI constraint that bites on the day TLS is switched on: ⛔ CSS `url()` values and web fonts **can be blockable**, not merely upgradable; ⛔ mixed content to an **IP-address host is blocked rather than upgraded** | ⚠️ Repeated from source 17 because it is load-bearing in *both* the hosting and the UI contexts |
| 78 | **`Set-Cookie` header (MDN)** | `https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie` | OFFICIAL-STD | Session-cookie mechanics that constrain an unattended shared device; the `__Host-` prefix requirements | As source 18 |

### 2.9 Internal

| # | Document | Location | Class | What it supports |
|---|---|---|---|---|
| 79 | **Additional CloudLight Questionnaire** | `../../questionnaires/Additional_CloudLight_Questionnaire.md` | INTERNAL | **Every `CL-*` identifier cited in every CloudLight document**, with priority, respondent, dependency, evidence requirement and status |
| 80 | **School network** | `../shana/infrastructure/school_network.md` | INTERNAL | ⛔ **The prohibition on probing school infrastructure** — the reason every network fact in this project is a question rather than a finding; also the robot link on `192.168.123.0/24` |
| 81 | **Risk register** | `../shana/safety/risk_register.md` | INTERNAL | The `L × S` rating convention (🔴 High 15–25, 🟠 Medium 8–14, 🟡 Low 1–7) and the status vocabulary used across CloudLight documents |
| 82 | **Student privacy** | `../shana/safety/student_privacy.md` | INTERNAL | The no-identification invariant; no per-student activity tracking; no raw video retention — the basis of constraint C10 |
| 83 | **Repository root script** | `start_opencode_server.sh` + `.gitignore` | INTERNAL | ✅ **VERIFIED:** the script holds a hardcoded plaintext password and binds `0.0.0.0`; it **is** listed in `.gitignore` and so is not committed. Basis for constraint C5 |
| 84 | **Architecture** | `../shana/architecture/architecture.md` | INTERNAL | Evidence-label conventions; the local application server; the AI/deterministic boundary |
| 85 | **Architecture decisions** | `docs/architecture/architecture_decisions/` (ADR-013–016) and `docs/shana/architecture/architecture_decisions/` (ADR-001–012) | INTERNAL | **SHANA [ADR-001](../shana/architecture/architecture_decisions/ADR-001-no-ai-reaches-the-robot.md) (`D-005`) — no AI model may reach the robot** — the basis of FR-AI-04 and constraint C3 |
| 86 | **SHANA sources** | `../references/sources.md` | INTERNAL | ⚠️ **Not merged into this file.** Referenced where a CloudLight claim depends on robot-side documentation |

---

## 3. Sources deliberately NOT used

⚠️ **Recorded so that nobody repeats the search.** "Not used" here means considered
and rejected, or searched and not located — **not** that the claim is false.

| Source considered | Why not used | What remains uncertain |
|---|---|---|
| **WCAG 2.2 (W3C)** | Not consulted during the sibling-document pass. Adding it now would retro-fit a citation onto numbers we chose rather than claims we sourced — see §4 | ⛔ **The accessibility targets in [`requirements.md`](requirements.md) §3.5 are unsourced proposals.** This is the highest-value gap to close |
| **EN 301 549 / RPvF (European accessibility standard)** | Out of scope for an Indian school. ❓ Whether any equivalent applies here is `CL-UX-004` | Whether any accessibility requirement is binding rather than advisory |
| **Synology-specific NAS hardware specifications** | Product pages describe models in the range, not the model this school owns | ❓ Whether the actual NAS supports containers at all — `CL-NAS-001`, `CL-NAS-002`, `CL-NAS-003` |
| **Microsoft Intune / MEM documentation beyond kiosk settings** | Would require assuming the school runs Intune. ❓ Unanswered | Whether MDM is available at all — `CL-AUTH-018` |
| **Specific smartboard or interactive-panel vendor documentation** | ❓ **No panel model is known.** Citing a vendor would require inventing the hardware | ❓ **Everything about the actual displays** — resolution, touch digitisation, brightness, viewing angle — `CL-DASH-018`, `CL-WIN-018` |
| **Published benchmarks of local LLM inference on CPU** | Highly hardware-specific; a benchmark on other hardware supports nothing about `CL-AI-009`, which is unanswered | ❓ Whether inference is even on the table. `CL-AI-009` |
| **A cloud-VM hosting provider's documentation** | Option (e) is a contingency only, never a primary. Writing provider documentation would imply a decision not made | ❓ Whether the school would accept cloud hosting at all — `CL-SRV-008`, `CL-SRV-005` |
| **School-specific network documentation, prior incident reports, or the school's existing systems documentation** | ⛔ **None has been provided, and none may be requested by probing** | Every network fact in this project. `CL-NET-*` |
| **Any Indian-specific education-technology regulation** | ❓ **NOT FOUND** — searched, not located, and not established whether such a requirement applies to a single school's internal display system | Whether any external obligation applies to storing a staff timetable. `CL-SRV-005`, `CL-DATA-015` |
| **The SHANA source register, merged in** | ⛔ It is a **separate bibliography**. CloudLight and SHANA have different scope, different access dates and different audiences | Robot-side claims are cited from their own register, not duplicated here |

---

## 4. Claims that rest on no source

> ### ⚠️ **These are engineering judgement. They are not verified findings, and no citation anywhere makes them so.**
>
> They are listed precisely so that nobody inherits them as established fact. Several
> are reasonable; reasonableness is not evidence.

| # | Claim | Where | Why it is unsourced |
|---|---|---|---|
| 1 | **Every number in the non-functional targets table** — the 2 s render, 1 Hz countdown, 5 s propagation, 1 s search, 80% disk, 15 min idle, 8 h absolute | [`requirements.md`](requirements.md) §3 | ⚠️ **We invented these as arguable proposals.** No school has stated a requirement. Each is attached to the question that would confirm it |
| 2 | **The contrast ratios (4.5:1, 3:1) and target sizes (44×44 px)** | [`requirements.md`](requirements.md) §3.5 | ⛔ **No accessibility standard is cited anywhere in this project.** Widely used general thresholds, but uncited here |
| 3 | **The entire display-state catalogue** — countdown tiers, staleness tiers, the four teacher states, the failure-state list | [`classroom_dashboard.md`](classroom_dashboard.md) | ⛔ **That document states it carries no external citations at all.** Entirely design on paper, unproven against a real room and screen |
| 4 | **The observation that a PC boots faster than the network**, and the retry/recovery matrix that follows from it | [`windows_startup.md`](windows_startup.md) §4 | 🧪 Community knowledge about real deployments, not a vendor claim and not observed here. Very likely correct; entirely untested |
| 5 | **The proposition that a school LAN must not be assumed trusted** | [`network_and_hosting.md`](network_and_hosting.md) §6.1 | 🧪 Sound reasoning, no single citation. ⚠️ **It happens to be right, which is not the same as being evidenced** |
| 6 | **The `0.6–0.7 GB of RAM per billion parameters at 4-bit` figure** | [`network_and_hosting.md`](network_and_hosting.md) §4.2 | 🧪 **Our own derivation** from the vendor rule of thumb (source 42), which states loading costs, not quantised inference residency |
| 7 | **That host sizing must budget for a realistic KV-cache context length** | [`network_and_hosting.md`](network_and_hosting.md) §4.2 | 🧪 **A trap that sizing estimates usually miss** — an insight, not a measurement. No figure is given because none can be verified without running a model |
| 8 | **The requirement that the timetable version history be immutable and rollback-able** | [`requirements.md`](requirements.md) FR-TIME-07 | 🧪 Standard practice, adopted by judgement. Whether the school wants the storage cost is `CL-DATA-017` |
| 9 | **The recommendation to build without SHANA and add it last** | [`shana_integration.md`](shana_integration.md) | 🧪 A sequencing judgement. Reasonable, but it is a preference, not a finding |
| 10 | **The claim that the AI assistant cannot be made safe by filtering alone** | [`ai_assistant.md`](ai_assistant.md); `requirements.md` FR-AI-05 | ⚠️ **Partly sourced** — sources 50 and 51 say RAG "does not fully mitigate" injection and argue for structural boundaries. ⚠️ **The leap from "not fully mitigated" to "not deployable in a classroom" is ours** |
| 11 | **That a backup which has never been restored is not a backup** | [`network_and_hosting.md`](network_and_hosting.md) §9.6 | 🧪 A truism, and it is true, and it is still judgement |
| 12 | **Every `PENDING ANSWER` and every ❓ UNKNOWN in the CloudLight documents** | all | ⛔ **These are gaps, not claims.** They are recorded so that nobody fills one with a plausible default |

---

## 5. Contradictions and unresolved points

⚠️ **Recorded, not resolved.** Each of these is somewhere a reasonable reader
would expect the sources to agree and they do not, or where a requirement and a
platform limit pull in opposite directions.

### 5.1 🔴 Kiosk lockdown versus offline capability — **the most important one**

| Side | Source |
|---|---|
| **The display must keep working when the host is unreachable** | `requirements.md` NFR-OFF-02; `CL-SEC-021` |
| **The display must be locked down and unauthenticated** | `requirements.md` FR-AUTH-02, constraint C7; `CL-SEC-001` |
| **The documented kiosk lockdown experience runs an InPrivate session with autofill disabled** | Source 6 — Edge kiosk mode |
| **Whether persistent offline caching survives that configuration on this hardware** | ❓ **UNKNOWN. No source answers this, and no board has been touched** |

⚠️ **If a locked-down kiosk session forbids persistent caching, one of the two
non-negotiable requirements must give.** The resolution is not a design decision —
it is **a test on one non-critical machine**, and it is cheap. Until it is run,
**the offline tier that is actually achievable is unproven.**

### 5.2 🔴 Kiosk idle timeout does not restart the browser by itself

| Claim | Source |
|---|---|
| `--kiosk-idle-timeout-minutes` exists and accepts 0 or 1–1440 | Source 6 |
| ⛔ **The idle timeout does not restart Edge — Assigned Access or Shell Launcher is required** | Source 6, verbatim vendor warning |

⚠️ **Contradicts the common assumption that a browser flag makes a kiosk.** The
display can silently stop responding mid-lesson unless a shell is configured.
⛔ **Also compounded by source 12**: Unified Write Filter is not available on
Windows Home or Pro, so an entire lockdown layer may be unavailable depending on
the board's edition — `CL-WIN-001` is unanswered.

### 5.3 🟡 NIST SP 800-63B Rev. 3 versus Rev. 4

| Point | Rev. 3 (source 62) | Rev. 4 (source 63) |
|---|---|---|
| Timeouts | Cited for failed-attempt limits and OTP validity; **no timeout figures quoted in the CloudLight documents** | ✅ **Explicitly requires both an overall and an inactivity timeout.** AAL2: overall ≤ 24 h, inactivity ≤ 1 h |
| Scope | — | States the guidelines *"are not intended to constrain the development or use of standards outside of this purpose"* |

⚠️ **Two revisions of the same document are cited in two sibling CloudLight
documents, and the newer one is the one that actually answers the timeout
question.** This file cites both and flags the overlap rather than silently
picking one. ⚠️ Neither is binding at an Indian school — `CL-AUTH-016`.

### 5.4 🟡 Document-parsing benchmarks ran without OCR

| Claim | Source |
|---|---|
| Layout detection in sub-second latency on a single CPU; TableFormer 2–6 s per table | Source 52 |
| ⛔ **The 225-page benchmark was run with OCR disabled** | Source 52 |

⚠️ **The attractive latency figures and the scanned-document case come from
different experiments.** If a school's timetable is a scan, a photograph or a
handwritten board, none of those figures apply — and source 52's own numbers put
OCR at *"upwards of 30 seconds per page"* on CPU. ⚠️ And sources 53–55 establish
that the underlying models were trained on finance, science, patents, tenders,
law and manuals — **not school timetables**.

### 5.5 🟡 A weak model fits — until the context gets long

| Side | Source |
|---|---|
| 4-bit quantisation makes capable models run on limited hardware | Sources 43, 44 |
| ⛔ **The KV cache remains in full precision and grows with context length** | Our derivation — source 42 does not say this; see §4 #7 |

⚠️ **A model that fits when empty can fail at a realistic context length.** Any
sizing must budget for context, not for zero. ⚠️ This is recorded as a
contradiction because it is one between a widely repeated sizing rule and the
architecture that rule ignores.

### 5.6 🟡 Offline-first resilience versus offline-first security

| Side | Evidence |
|---|---|
| Core operation must not require the internet — a school uplink may be down for a term | `network_and_hosting.md` §7.2 — a design principle |
| ⛔ A permanently offline host never receives OS, database, browser or library security fixes; credentials that never rotate are credentials that are eventually compromised | Same document, §7.3 |
| ⚠️ Certificate expiry presents an untrusted warning to **every board simultaneously** if renewal needs the internet | `NFR-SEC-02` |

⚠️ **Offline-first is a resilience property, not a security posture.** The
resolution is an internal CA (`CL-SEC-024`) or a documented, tested update path —
not a relaxation of either side.

### 5.7 🟡 SMBv1 is deprecated — and the school NAS may only speak it

| Side | Evidence |
|---|---|
| ⛔ *"SMB1 is deprecated and should not be installed nor enabled"*; SMB1-only devices are *"likely running older versions of Linux, Samba, or other types of third-party software"* that are *"no longer supported"* | Sources 26, 27 |
| A share may be reachable **only** over SMB1 | The failure mode anticipated in [`resource_browser.md`](resource_browser.md) |

⚠️ **The vendor guidance and the reality can only be reconciled by replacing the
storage, not by enabling SMB1.** ❓ Which situation applies is `CL-NAS-002`, and
it cannot be determined without asking IT.

### 5.8 ⚠️ Broken internal cross-references — **affects verifiability**

> ### ⚠️ **Several CloudLight documents cite repository paths that do not exist, so the support they claim cannot be located.**

Checked against the repository tree on 2026-10-11. **This file does not modify
those documents** — another agent owns them — but a reader following a citation
will hit a dead end:

| Cited path | Actually located at | Cited in |
|---|---|---|
| `docs/architecture/architecture.md` | `docs/architecture/architecture.md` | `network_and_hosting.md`, `shana_integration.md`, `ai_assistant.md`, `windows_startup.md` |
| `docs/architecture/data_flow.md` | `docs/architecture/data_flow.md` | `timetable_and_substitutions.md`, `ai_assistant.md` |
| `docs/architecture/delivery_state_machine.md` | ⚠️ **No such file.** The nearest is `docs/architecture/task_state_machine.md` | `timetable_and_substitutions.md`, `ai_assistant.md`, `shana_integration.md`, `ota_updates.md` |
| `docs/safety/student_privacy.md` | `docs/safety/student_privacy.md` | `ai_assistant.md`, `shana_integration.md` |
| `docs/safety/risk_register.md` | `docs/safety/risk_register.md` | `security_and_privacy.md` |
| `docs/infrastructure/school_network.md` | `docs/infrastructure/school_network.md` | `network_and_hosting.md`, `ota_updates.md`, `shana_integration.md`, `windows_startup.md` |
| `docs/references/sources.md` | ⚠️ **No such file.** See the note at the head of this document | — |
| `docs/cloudlight/architecture.md`, `docs/cloudlight/feasibility_report.md` | ⚠️ **Not present** at the date above | — |

⚠️ **The affected documents are internally consistent with each other**, and
several of them — [`security_and_privacy.md`](security_and_privacy.md) in
particular — use the correct `../...` relative form. The inconsistency is
between documents, not within them. ⚠️ **This file uses only paths verified to
resolve.**

### 5.9 🟡 A question asked of the wrong people

⚠️ **`CL-SRV-020` asks whether the host should "report its own health and failures
somewhere the school already monitors", and `CL-NET-020` asks whether the school
network is monitored.** Both presuppose a monitoring capability nobody has
confirmed exists. ❓ If the answer is no, both questions are moot and **there is no
path by which a host failure reaches a person** — which makes `NFR-AVL-05`
unachievable as written rather than merely unmet.

---

⚠️ **Standing note.** Every external source above is a **general published
standard, vendor document, security reference or research paper describing
systems in general**. **None of it describes this school, its hardware, its data,
its network or its policy.**

**Network probes: zero. Smartboards touched: zero. Robots operated: zero. AI
services contacted: zero. Models run: zero. School documents read: zero. Hosts
deployed: zero. Tests executed: zero. Questions answered: zero.**

⚠️ **And the last word is not ours.** Every `❓ UNKNOWN` in this project is a
genuine gap, not an omission. Filling one with a plausible default is the failure
mode this bibliography exists to prevent.

**Related documents:** [`requirements.md`](requirements.md) ·
[`deployment_options.md`](deployment_options.md) ·
[`network_and_hosting.md`](network_and_hosting.md) ·
[`security_and_privacy.md`](security_and_privacy.md) · [`ota_updates.md`](ota_updates.md) ·
[`ai_assistant.md`](ai_assistant.md) · `docs/references/sources.md` (the SHANA bibliography) ·
`docs/platform_selection/sources.md`