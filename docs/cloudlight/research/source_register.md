# CloudLight — Source Register

**Phase 0 · Documentation only · Compiled 2026-10-11**

Every research thread behind CloudLight, with what it establishes, what it does
not establish, and what would retire it.

> ### ⚠️ **Read the `Limitations` field. It is the important column.**
>
> Every entry below is external documentation describing **systems in general**.
> **None of it describes this school, its hardware, its network, its data or its
> policy.** A `VERIFIED` confidence below means *the publisher is authoritative
> about the general claim*. It never means *this is true of the school*.

> ### ⚠️ **No hardware was verified.**
>
> Network probes: zero. Smartboards touched: zero. NAS shares mounted: zero.
> AI services contacted: zero. Models run: zero. Robot operated: zero. School
> documents read: zero. Hosts deployed: zero. Tests executed: zero.
> Questions answered: zero.

**Fields per entry:** Research ID · Topic · Question investigated · Source title ·
Publisher · URL or repository path · Publication or update date · Date accessed ·
Relevant section · Claim supported · Limitations · Version applicability ·
Confidence · Related requirements · Related decisions · Related open questions ·
Next verification step.

**Confidence vocabulary** (matches [`../feasibility_report.md`](../feasibility_report.md)):

| Value | Meaning |
|---|---|
| `VERIFIED` | Confirmed in authoritative vendor, standards or regulatory documentation, read directly |
| `DOCUMENTED_NOT_HARDWARE_VERIFIED` | A vendor documents it; not tested on this school's equipment |
| `PLAUSIBLE_REQUIRES_TESTING` | Engineering judgement; no vendor claim; needs a test |

⚠️ **There is no entry in this register with confidence `VERIFIED` about this
school.** Every school-specific fact is an open question, listed by `CL-*` ID in
[`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md)
and **all 576 are currently `UNANSWERED`**.

---

## 1. Windows smartboards, kiosk mode and device management

### CR-001 — Assigned Access and the single-app kiosk

| Field | Value |
|---|---|
| Research ID | `CR-001` |
| Topic | Windows kiosk lockdown |
| Question investigated | Is there a supported way to lock a classroom Windows display to a single application without someone being able to escape? |
| Source title | Windows Single-App and Multi-App Kiosk Configuration Options Overview; Configure a Single-App Kiosk with Assigned Access |
| Publisher | Microsoft (vendor documentation) |
| URL or repository path | `https://learn.microsoft.com/en-us/windows/configuration/kiosk/` · `https://learn.microsoft.com/en-us/windows/configuration/assigned-access/configure-single-app-kiosk` |
| Publication or update date | UNKNOWN — continuously revised Microsoft Learn content |
| Date accessed | `2026-10-11` |
| Relevant section | [`../windows_startup.md`](../windows_startup.md) §1–§2 |
| Claim supported | Assigned Access is the documented single-app kiosk mechanism; it applies lockdown policies automatically; the default `Ctrl+Alt+Del` breakout sequence exists and is configurable; the `LogonUI` `IdleTimeOut` default is 30 seconds and is **explicitly documented not to apply to Edge kiosk mode**; a single-app kiosk requires UAC and console sign-in, not RDP |
| Limitations | ⛔ Establishes **nothing** about this school's boards, their edition, or their build. Breakout being configurable is a lockdown *decision*, not a solved problem. Whether the school permits any of this at all is unanswered |
| Version applicability | Applies to the Windows editions listed in the edition table on the kiosk overview page. Expected to remain applicable across current Windows 10/11 releases, but **edition support has changed between releases** and must be re-checked against whatever is actually installed |
| Confidence | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| Related requirements | `FR-DASH-02`, `FR-DASH-14` |
| Related decisions | `D-009` |
| Related open questions | `CL-WIN-001` (edition — P0), `CL-WIN-003` (admin rights — P0), `CL-WIN-007` (a board to test on — P0) |
| Next verification step | Run `winver` on one non-critical classroom board and record the exact edition and build. Costs five minutes and settles whether the entire kiosk design is even available |

### CR-002 — Shell Launcher and its configuration surface

| Field | Value |
|---|---|
| Research ID | `CR-002` |
| Topic | Windows kiosk lockdown |
| Question investigated | What does Shell Launcher actually let us configure, and what happens if we exceed its limits? |
| Source title | Create a Shell Launcher configuration file; Configure Shell Launcher; Quickstart: Configure a kiosk experience with Shell Launcher |
| Publisher | Microsoft (vendor documentation) |
| URL or repository path | `https://learn.microsoft.com/en-us/windows/configuration/assigned-access/shell-launcher/configuration-file` · `https://learn.microsoft.com/en-us/windows/configuration/shell-launcher/configure` · `https://learn.microsoft.com/en-us/windows/configuration/shell-launcher/quickstart-kiosk` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../windows_startup.md`](../windows_startup.md) §2–§3 |
| Claim supported | Shell Launcher is an optional component enabled through the Assigned Access CSP, OMA-URI `./Vendor/MSFT/AssignedAccess/ShellLauncher`; `ReturnCodeActions` supports `RestartShell`, `RestartDevice`, `ShutdownDevice`, `DoNothing`; `AutoLogonAccount` creates a local standard `Kiosk` user; the canonical Edge command line includes `--kiosk-idle-timeout-minutes=2`; applying configuration requires SYSTEM-level privilege via MDM Bridge |
| Limitations | ⛔ The quickstart is a **worked example, not a normative specification** — it shows one valid command line, not the complete set. Shell Launcher and Assigned Access **cannot both be configured** on one machine. Applying any of this needs administrative rights nobody has confirmed exist. The vendor's own warning that exceeding the four-custom-actions-plus-one-default limit means **nothing happens** is a silent failure mode, not a warning dialog |
| Version applicability | Enterprise and Education editions only. Expected to persist, but the quickstart's exact command line may drift as Edge flags change |
| Confidence | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| Related requirements | `FR-DASH-02`, `FR-OTA-04` |
| Related decisions | `D-022` |
| Related open questions | `CL-WIN-001`, `CL-WIN-003`, `CL-WIN-005` (is any MDM in use — unanswered, which would change the entire delivery method) |
| Next verification step | Confirm with IT whether any configuration-management tool is deployed. If not, the whole MDM Bridge delivery route in the quickstart is unavailable and a manual per-board configuration is implied |

### CR-003 — Edge kiosk mode and the InPrivate problem

| Field | Value |
|---|---|
| Research ID | `CR-003` |
| Topic | Browser kiosk behaviour |
| Question investigated | Does the documented Edge kiosk lockdown experience conflict with the requirement that a display keep working when the server is unreachable? |
| Source title | Configure Microsoft Edge kiosk mode |
| Publisher | Microsoft (vendor documentation) |
| URL or repository path | `https://learn.microsoft.com/en-us/deployedge/microsoft-edge-configure-kiosk-mode` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../windows_startup.md`](../windows_startup.md) §2; [`../sources.md`](../sources.md) §5.1 |
| Claim supported | Two lockdown experiences exist — Digital/Interactive Signage and Public-Browsing; **both run an InPrivate session with autofill disabled**; `--kiosk-idle-timeout-minutes` accepts 0 or 1–1440, defaulting to 0 (full-screen) and 5 (public browsing); ⛔ **the idle timeout does not restart Edge — Assigned Access or Shell Launcher is required**; F11/F12/CTRL+N/CTRL+T are blocked; `edge://` URLs are blocked |
| Limitations | ⛔⛔ **This is the single most load-bearing contradiction in the project.** A display that must keep working while the host is unreachable (`NFR-OFF-02`) may be running an InPrivate session that forbids persistent caching. **No source answers whether persistent offline caching survives this configuration, and no board has been touched.** Also compounded by `CR-005`: an entire additional lockdown layer may be unavailable depending on the board's edition |
| Version applicability | The two experiences and their defaults are current as of the accessed page. ⛔ **The caching behaviour under InPrivate is version-dependent and untested here.** The page itself records feature availability by Edge version (89/90) |
| Confidence | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| Related requirements | `FR-DASH-02`, `FR-DASH-14`, `NFR-OFF-02` |
| Related decisions | `D-009`, `D-022` |
| Related open questions | `CL-WIN-001`, `CL-WIN-007`, `CL-WIN-011`, `CL-OTA-011` |
| Next verification step | ⛔ **A fifteen-minute test on one non-critical machine: launch Edge in kiosk mode, disconnect the network, reload, and observe whether previously-loaded content survives.** This is the cheapest test in the entire project and it currently decides whether the offline tier is achievable at all |

### CR-004 — Intune-delivered kiosk configuration

| Field | Value |
|---|---|
| Research ID | `CR-004` |
| Topic | Device management |
| Question investigated | If the school manages these devices, is there a supported way to deliver a kiosk profile centrally? |
| Source title | Kiosk settings for Windows in Microsoft Intune |
| Publisher | Microsoft (vendor documentation) |
| URL or repository path | `https://learn.microsoft.com/en-us/intune/intune-service/configuration/kiosk-settings-windows` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../windows_startup.md`](../windows_startup.md) §2 |
| Claim supported | An Intune-delivered kiosk profile can carry an Edge Kiosk URL, a kiosk mode type, and a "Refresh browser after idle time" setting from 0 to 1440 minutes |
| Limitations | ⛔ **Requires Microsoft Intune.** Citing this without checking would assume the school runs a commercial MDM product it may not run at all. ❓ Nobody has asked. **This entry exists to record a capability that may be irrelevant to this school** |
| Version applicability | Tied to the Intune service's current feature set, which changes independently of Windows |
| Confidence | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| Related requirements | `FR-OTA-04`, `FR-ADMIN-*` |
| Related decisions | `D-022` |
| Related open questions | `CL-AUTH-018`, `CL-WIN-005` |
| Next verification step | Ask IT one question: "Is Intune or any other device management tool in use?" The answer determines the entire configuration-delivery method |

### CR-005 — Unified Write Filter

| Field | Value |
|---|---|
| Research ID | `CR-005` |
| Topic | Device lockdown |
| Question investigated | Can write-filtering be used to make classroom boards resilient to accidental or malicious change? |
| Source title | Unified Write Filter — overlay location and size; `UWF_Filter` WMI provider |
| Publisher | Microsoft (vendor documentation) |
| URL or repository path | `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/customize/uwfoverlay` · `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/customize/uwf-filter` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../windows_startup.md`](../windows_startup.md) §3 |
| Claim supported | The edition support table is explicit: **Home No, Pro No, Enterprise Yes, Education Yes, IoT Enterprise Yes**. The RAM overlay is cleared on reboot; default overlay size is 1024 MB; ⛔ **overlay exhaustion causes failed writes and an unresponsive device**; ⛔ **shutdown can take "up to an hour or longer" when the overlay is full**; changing UWF configuration requires an administrator account |
| Limitations | ⛔ **Not available on Windows Home or Pro.** If any board runs Pro, this entire lockdown layer is simply not an option — and it cannot be enabled by configuration, only by re-imaging. The failure modes are severe enough that enabling it casually is worse than not enabling it: a classroom device can become unusable-slow. **The RAM-overlay-cleared-on-reboot behaviour also means downloaded installers do not survive a restart**, which interacts directly with the offline-update problem in `CR-012` |
| Version applicability | The edition table is stable across recent Windows releases but **must be re-checked against the installed edition**, which is unknown (`CL-WIN-001`) |
| Confidence | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| Related requirements | `FR-WIN-*`, `FR-SEC-05` |
| Related decisions | `D-009` |
| Related open questions | `CL-WIN-001`, `CL-WIN-003` |
| Next verification step | Read the edition from `winver` on one board. If it is Pro, mark UWF **permanently unavailable** in [`../decision_log.md`](../decision_log.md) rather than leaving it as an open option |

### CR-006 — Why a Windows service cannot be the kiosk launcher

| Field | Value |
|---|---|
| Research ID | `CR-006` |
| Topic | Startup mechanism |
| Question investigated | Could a background Windows service start the kiosk application, avoiding per-user configuration? |
| Source title | Interactive Services (Win32); Task Scheduler for developers; `ITaskSettings::put_RestartCount`; `schtasks` commands |
| Publisher | Microsoft (vendor documentation and API reference) |
| URL or repository path | `https://learn.microsoft.com/en-us/windows/win32/services/interactive-services` · `https://learn.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-start-page` · `https://learn.microsoft.com/en-us/windows/win32/api/taskschd/nf-taskschd-itasksettings-put_restartcount` · `https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../windows_startup.md`](../windows_startup.md) §3 |
| Claim supported | ⛔ **A platform limit, not a policy choice:** services cannot directly interact with a user since Windows Vista; all services run in Terminal Services session 0; `NoInteractiveServices` defaults to 1; the `CreateProcessAsUser` workaround exists and carries documented risk. Task Scheduler documents triggers at logon, boot, idle, event, registration and session-state change, plus restart-on-failure as a first-class setting; `schtasks` requires membership of the Administrators group |
| Limitations | ⛔ **This closes off the service route definitively** — no amount of configuration makes a session-0 service display UI. Task Scheduler documents the *mechanism* only; it does not guarantee the school permits tasks to be created, and every route requires local administrator rights nobody has confirmed. ⚠️ Note [`../ota_updates.md`](../ota_updates.md) §10 discusses kiosk elevation and scheduled-task constraints and **explicitly cites no vendor source for them** — a deliberate gap recorded rather than filled |
| Version applicability | Session-0 isolation has been in place since Windows Vista and is expected to persist. The Task Scheduler API surface is stable |
| Confidence | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| Related requirements | `FR-DASH-02`, `FR-OTA-04` |
| Related decisions | `D-022` |
| Related open questions | `CL-WIN-003` |
| Next verification step | Ask whether the kiosk account may be a local administrator. If it may not, self-start and self-update both become site visits, and that must be priced into the deployment plan rather than discovered later |

### CR-007 — Windows IoT Enterprise licensing and lifecycle

| Field | Value |
|---|---|
| Research ID | `CR-007` |
| Topic | Device lockdown and licensing |
| Question investigated | Is IoT Enterprise a legitimate option for classroom boards, and what is its maintenance horizon? |
| Source title | Windows IoT Enterprise LTSC in Volume License; Windows IoT Enterprise LTSC 2021 release notes; Windows IoT Enterprise release history |
| Publisher | Microsoft (vendor documentation) |
| URL or repository path | `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/deployment/volume-license` · `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/whats-new/windows-10-iot-enterprise-ltsc-2021` · `https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/whats-new/release-history` |
| Publication or update date | LTSC 2021 release notes: servicing window stated as 2021-11-16 to 2032-01-13, Fixed Lifecycle Policy |
| Date accessed | `2026-10-11` |
| Relevant section | [`../windows_startup.md`](../windows_startup.md) §1 |
| Claim supported | The verbatim licensing restriction: IoT Enterprise LTSC is *"intended for use for fixed-function specialized devices only and cannot be used as a replacement of Windows desktop for general purpose computing"*; it is upgrade-licence-only through Volume Licensing; LTSC 2021 is build 19044, equivalent to Windows 10 21H2 |
| Limitations | ⚠️ **A licensing constraint, not a technical one.** The technical capability is real; whether the school may lawfully acquire and use it is a commercial and legal question this document cannot answer. ❓ Whether Volume Licensing is available to this school at all is `CL-BUD-005`, unanswered. The 2032-01-13 date is a **hard horizon**, not a preference — it is a date on which the option disappears |
| Version applicability | LTSC 2021 runs to 2032-01-13 under Fixed Lifecycle. Other LTSC releases have different dates; the release-history page must be re-read for the current line |
| Confidence | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| Related requirements | `FR-DASH-02` |
| Related decisions | `D-022` |
| Related open questions | `CL-BUD-005`, `CL-WIN-001` |
| Next verification step | Ask the school's administration whether Volume Licensing is reachable. Do not assume re-imaging is even an option until this is answered |

---

## 2. Local networking, SMB/CIFS and storage

### CR-008 — Client isolation breaks local-first design silently

| Field | Value |
|---|---|
| Research ID | `CR-008` |
| Topic | Network isolation |
| Question investigated | Can a classroom device on the school's Wi-Fi reach a server on the same network? |
| Source title | Client Isolation — Aruba Central; Configuring Client Isolation / Wireless Network Profiles — Aruba Central; About AP Client Isolation — WatchGuard |
| Publisher | Aruba (HPE) and WatchGuard (network vendor documentation) |
| URL or repository path | `https://arubanetworking.hpe.com/techdocs/central/2.5.7/content/aos10x/cfg/aps/client-isolation.htm` · `https://arubanetworking.hpe.com/techdocs/central/latest/content/nms/access-points/cfg/networks/conf_wlan_ssid.htm` · `https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Firmware/wireless/ap_station_isolation_c.html` |
| Publication or update date | Aruba page versioned to Central 2.5.7; the second Aruba page is on `latest` |
| Date accessed | `2026-10-11` |
| Relevant section | [`../network_and_hosting.md`](../network_and_hosting.md) §2.5; [`../feasibility_report.md`](../feasibility_report.md) §5 |
| Claim supported | Client isolation *"isolates clients from one another and disables all peer-to-peer communication within the network"*, allowing *"only client to gateway traffic"*; unlisted destinations are dropped. On Aruba it is a **per-SSID** setting (*"the clients can connect to the Internet, but cannot communicate with each other"*). On WatchGuard it blocks clients on the same AP, across radios and across APs, and on firmware 8.6.x+ also blocks communication *"with wired-side hosts on the same network"* |
| Limitations | ⛔⛔ **This is the finding that breaks a local-first design while the board still shows full Wi-Fi signal bars.** The device is connected, the SSID is correct, the signal is strong — and device-to-device traffic is silently refused. There is no error the user sees. ⚠️ **Vendor-specific.** Whether this school runs Aruba, WatchGuard, or something else entirely is `CL-NET-008`, P0 and **UNANSWERED**. ❓ No network was probed, so nothing here is known about this school |
| Version applicability | Aruba behaviours are tied to AOS 10.x and Central; WatchGuard behaviours are tied to firmware 8.6.x+. Both change. **If the school's actual equipment differs, this entry's specifics may not apply at all** |
| Confidence | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| Related requirements | `NFR-AVL-01`, `FR-SRV-*` |
| Related decisions | `D-101` |
| Related open questions | `CL-NET-008` (P0), `CL-NET-003` (P0), `CL-NET-007` |
| Next verification step | ⛔ **Ask IT one question: "Is client isolation or AP isolation enabled on the classroom SSID?"** This is P0 because it is the most likely quiet failure in the entire project — the design fails with a screen that simply never loads, not with an error |

### CR-009 — An access point is a bridge, not a host

| Field | Value |
|---|---|
| Research ID | `CR-009` |
| Topic | Network topology |
| Question investigated | Can the access point the boards connect to host the CloudLight backend? |
| Source title | Wireless LAN Configuration Guide, Cisco IOS XE — WLAN Overview |
| Publisher | Cisco (network vendor documentation) |
| URL or repository path | `https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/wlan/configuration/xe-16-7/wl-16-7-book/wl-overview.html` |
| Publication or update date | UNKNOWN; the path is versioned to IOS XE 16.7 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../network_and_hosting.md`](../network_and_hosting.md) §2.1; [`../feasibility_report.md`](../feasibility_report.md) §6 |
| Claim supported | An access point serves *"as the connection point between wireless and wired networks"* and operates *"in bridging and routing mode"* — that is, a **Layer-2 function forwarding frames**, not an application host |
| Limitations | Establishes the general topology of an AP. ⚠️ **A single vendor's documentation for one product line.** Some enterprise APs do offer captive-portal or authentication services, but none of those is an application host. Router "port forwarding" does not create a backend — it forwards to a device that must already exist. ❓ **What this school's actual AP is and what it supports is unknown** (`CL-NET-008`) |
| Version applicability | Cisco IOS XE 16.7. Other vendors implement APs consistently but a reader should not treat one vendor's doc as universal without checking their own |
| Confidence | `DOCUMENTED_NOT_HARDWARE_VERIFIED` |
| Related requirements | `NFR-SCA-03` |
| Related decisions | `D-101`, `X-02` |
| Related open questions | `CL-NET-008`, `CL-SRV-001` |
| Next verification step | Identify the AP make and model from a photograph of the label, or ask IT. Then confirm the real answer to a different question: **is there any always-on machine at all?** |

### CR-010 — NAS as a host, and as a reverse proxy

| Field | Value |
|---|---|
| Research ID | `CR-010` |
| Topic | Hosting |
| Question investigated | Could the school's NAS run the backend, and could it terminate TLS? |
| Source title | Container Manager — Synology; Advanced — Login Portal, DSM 7; Login Portal — DSM 7 |
| Publisher | Synology (vendor documentation) |
| URL or repository path | `https://www.synology.com/en-us/dsm/feature/docker` · `https://kb.synology.com/en-global/DSM/help/DSM/AdminCenter/system_login_portal_advanced?version=7` · `https://kb.synology.com/en-global/DSM/help/DSM/AdminCenter/system_login_portal` |
| Publication or update date | Login Portal pages versioned to DSM 7 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../network_and_hosting.md`](../network_and_hosting.md) §3.4 |
| Claim supported | Some NAS platforms run containers *"directly on your NAS"* with Compose-based multi-container projects; DSM 7 provides a reverse-proxy feature (*"Your Synology NAS can act as a reverse proxy server"*) with source/destination protocol, hostname, port, HSTS and **selectable access control profiles restricting access by source IP**; a custom alias or domain can be configured |
| Limitations | ⚠️ **A product feature page is marketing framing, not a specification.** Availability and limits vary by model and DSM version. ⛔ **The feature moved between generations** — Application Portal in DSM 6 became Login Portal in DSM 7 — so a DSM version assumption invalidates the citation. ❓ **Whether a NAS exists at all, what model it is, and what it runs are all unanswered**: `CL-NAS-001`, `CL-NAS-002`. A NAS also raises a durability question nobody has assessed: consumer NAS hardware is not a server |
| Version applicability | DSM 7 specifically. DSM 6, DSM 7.1 and later may differ. **Must be re-checked against the actual DSM version once known** |
| Confidence | `PLAUSIBLE_REQUIRES_TESTING` |
| Related requirements | `NFR-SCA-03`, `NFR-MNT-05` |
| Related decisions | `D-101`, `D-023` |
| Related open questions | `CL-NAS-001`, `CL-NAS-002`, `CL-NAS-003`, `CL-NAS-004` |
| Next verification step | Ask IT what storage exists, if any. **Then check the model against the container-support list.** Do not design around a NAS that may not exist |

### CR-011 — SMBv1 deprecation as a hard blocker

| Field | Value |
|---|---|
| Research ID | `CR-011` |
| Topic | Storage protocols |
| Question investigated | What happens if the school's storage speaks only an obsolete protocol version? |
| Source title | SMBv1 is not installed by default in Windows 10/Server; Detect, enable, and disable SMBv1, SMBv2, and SMBv3 in Windows; SMB security hardening in Windows Server and Windows Client; [MS-SMB2] protocol specification |
| Publisher | Microsoft (vendor documentation and Open Specification) |
| URL or repository path | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/smbv1-not-installed-by-default-in-windows` · `https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/detect-enable-and-disable-smbv1-v2-v3` · `https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-security-hardening` · `https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/5606ad47-5ee0-437a-817e-70c366052962` |
| Publication or update date | UNKNOWN; hardening defaults stated for Windows 11 24H2 / Server 2025 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../resource_browser.md`](../resource_browser.md) §2.2; [`../resource_search.md`](../resource_search.md) §3.6 |
| Claim supported | SMBv1 is *"no longer installed by default"* since Windows 10 1709 and deprecated since 2014; ⛔ *"SMB1 is deprecated and should not be installed nor enabled"*; SMB 2.02/2.1/3.0/3.02/3.1.1 remain *"fully supported"*; SMB1-only devices are *"likely running older versions of Linux, Samba, or other types of third-party software"* that are *"no longer supported"*; Microsoft's recommendation is to update the server, not to reinstall SMBv1; SMB signing is required by default on Windows 11 24H2 / Server 2025; SMB Remote Mailslots *"doesn't provide any authentication or authorization mechanisms"* |
| Limitations | ⛔ **This is a blocked dependency, not a workaround.** If the school NAS speaks only SMB1, we cannot fix it, and we must not try. An encrypted or signing-required share may also fail against an older NAS — untested. ❓ **Which situation applies here is unknown and unknowable without asking IT** (`CL-NAS-002`) |
| Version applicability | Defaults are version-specific: signing requirements changed at Windows 11 24H2 / Server 2025. The deprecation itself has been stable since 2014 |
| Confidence | `VERIFIED` |
| Related requirements | `FR-RES-01`, `FR-RES-02`, `NFR-SEC-05` |
| Related decisions | `D-026` |
| Related open questions | `CL-NAS-002`, `CL-NAS-003` |
| Next verification step | Ask IT what SMB dialect the NAS offers. **If the answer is SMB1-only, the resource features are not deferred — they are cancelled, and the recommendation is to replace the storage rather than weaken the client** |

### CR-012 — NTFS ACLs, evaluation order and SIDs

| Field | Value |
|---|---|
| Research ID | `CR-012` |
| Topic | Storage permissions |
| Question investigated | Can CloudLight evaluate a user's access to a file using the storage's own rules, rather than inventing a parallel permission model? |
| Source title | NTFS overview; Access Control Lists; Security Identifiers |
| Publisher | Microsoft (vendor documentation) |
| URL or repository path | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview` · `https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists` · `https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../resource_browser.md`](../resource_browser.md) §3.1; [`../resource_search.md`](../resource_search.md) §3.3–§3.4 |
| Claim supported | NTFS allows an administrator to *"assign detailed permissions to files and folders using Access Control Lists (ACLs) … specify which users and groups have access"*; a DACL *"identifies the trustees that are allowed or denied access"*; ⛔ **evaluation order matters** — *"The system checks the ACEs in sequence until it finds one or more ACEs that allow all the requested access rights, or until any of the requested rights are denied"*; a SID *"is used to uniquely identify a security principal or security group"* and is never reused; `S-1-1-0` is *"a group that includes all users"* and `S-1-5-11` is all authenticated users and computers |
| Limitations | Windows-specific. ⛔ **This is the basis of the DENY-overrides-ALLOW rule, and the evaluation order is not the intuitive reading** — a naive implementation that treats the list as an unordered allow-list will be wrong. ⚠️ **The broad principals are exactly why an index built with the wrong identity is over-broad**: an ACL naming Everyone means everyone, and an index keyed on the wrong principal returns files the user could not browse to. ❓ **Whether this school's storage is NTFS at all is unknown** (`CL-NAS-002`), and the identity source itself is `CL-AUTH-001` / `CL-AUTH-002`, both unanswered |
| Version applicability | Stable Windows behaviour across current releases |
| Confidence | `VERIFIED` |
| Related requirements | `FR-SEARCH-02`, `FR-SEARCH-03`, `FR-RES-01` |
| Related decisions | `D-026` |
| Related open questions | `CL-NAS-002`, `CL-AUTH-001`, `CL-AUTH-002`, `CL-NAS-007` |
| Next verification step | Ask two questions together: what filesystem is the storage, and what directory authenticates against it. Without both, permission-filtered search cannot be designed, only imagined |

### CR-013 — NFS as a second permission model

| Field | Value |
|---|---|
| Research ID | `CR-013` |
| Topic | Storage permissions |
| Question investigated | If the storage is NFS rather than SMB, can permissions still be evaluated correctly? |
| Source title | RFC 1813 — NFS Version 3 Protocol Specification |
| Publisher | IETF (standard) |
| URL or repository path | `https://www.rfc-editor.org/rfc/rfc1813` |
| Publication or update date | 1995 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../resource_browser.md`](../resource_browser.md) §2.1 |
| Claim supported | NFSv3 has an explicit `ACCESS` procedure that performs a permission check over the wire; `AUTH_UNIX` carries *"the user's effective user ID, effective group ID and groups with each call"* and the server checks permissions from those values |
| Limitations | ⚠️ **`AUTH_UNIX` is weak identity — the client asserts its own IDs.** The IANA has named it obsolete. **This entry is listed as a second model to understand, not as a recommendation.** If the school storage uses NFS with `AUTH_UNIX`, permission-aware retrieval rests on assertions the client controls. ❓ Which protocol the school uses is unknown (`CL-NAS-003`) |
| Version applicability | RFC 1813 is NFSv3. NFSv4 replaced `AUTH_UNIX` with a much stronger identity model and is **not** covered by this entry |
| Confidence | `VERIFIED` |
| Related requirements | `FR-SEARCH-02`, `FR-SEARCH-03` |
| Related decisions | `D-026` |
| Related open questions | `CL-NAS-003` |
| Next verification step | Ask IT which protocol the shares use. If NFSv4, re-read this thread — the conclusion changes materially |

### CR-014 — SQLite durability and its honest limit

| Field | Value |
|---|---|
| Research ID | `CR-014` |
| Topic | Data integrity |
| Question investigated | Can a single-file relational database survive a power cut in a school with unreliable power? |
| Source title | SQLite: Is SQLite Transactional; SQLite: Atomic Commit; SQLite: Write-Ahead Logging |
| Publisher | SQLite (maintained open-source project documentation) |
| URL or repository path | `https://sqlite.org/transactional.html` · `https://sqlite.org/atomiccommit.html` · `https://www3.sqlite.org/wal.html` |
| Publication or update date | UNKNOWN — continuously revised |
| Date accessed | `2026-10-11` |
| Relevant section | [`../network_and_hosting.md`](../network_and_hosting.md) §9.4 |
| Claim supported | Transactions are *"atomic, consistent, isolated, and durable"* even across a power failure; *"all changes within a single transaction either occur completely or not at all"*; ⛔ **and the honest limit** — a power loss mid-sector-write *"opens SQLite up to the possibility of database corruption"*, which SQLite **cannot test for or remedy**; without syncing content to disk on commit, *"transactions are no longer durable and might rollback following a power failure"* |
| Limitations | ⚠️ **Read the three entries together, not selectively.** Citing only the transactional guarantee is a misrepresentation of the project's own source. A specific single-file engine is being described; choosing it is a decision that **has not been formally made** (`D-023` records a single relational database, not this engine specifically). The durability setting must be chosen deliberately rather than defaulted |
| Version applicability | Applies to SQLite generally; WAL behaviour is version-dependent in its performance characteristics |
| Confidence | `VERIFIED` |
| Related requirements | `NFR-REC-01`, `NFR-REC-02` |
| Related decisions | `D-023` |
| Related open questions | `CL-DATA-*`, `CL-OPS-*` |
| Next verification step | When a host is chosen, **kill the power on a test machine mid-write and restore from backup**. Until that is done, the durability claim is a documented property nobody has observed |

---

## 3. Search, indexing and permission-aware retrieval

### CR-015 — Document-level security as a cautionary example

| Field | Value |
|---|---|
| Research ID | `CR-015` |
| Topic | Permission-aware search |
| Question investigated | How does a production system actually implement document-level access control, and how does it fail? |
| Source title | Controlling access at the document and field level — Elastic |
| Publisher | Elastic (vendor documentation) |
| URL or repository path | `https://www.elastic.co/docs/deploy-manage/users-roles/cluster-or-deployment-auth/controlling-access-at-document-field-level` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../resource_search.md`](../resource_search.md) §3.5 |
| Claim supported | Per-role query templating against the authenticated user is a real, shipped implementation pattern. ⛔ **Three documented failure modes:** omitting the `query` parameter entirely **disables** document-level security, i.e. it fails **open**; multiple roles combine with **OR** (*"only one of the role queries must match for a document to be returned"*) with Elastic's own warning that you may *"inadvertently grant wider access than intended"*; and document-level security *"doesn't affect global index statistics that relevancy scoring uses"*, so a restricted user *"could still learn about field names and terms that only exist in inaccessible documents"* |
| Limitations | ⚠️ **Product-specific, and used here deliberately as a cautionary example rather than a selection.** Elastic is **not** a decided technology — `D-103` is open and the corpus size that would settle it is unknown (`CL-SEARCH-014`). ⛔ The three failure modes are general and are the actual lesson: filtering must fail **closed**, roles must not OR open access, and metadata leaks through statistics whether or not documents do |
| Version applicability | Elastic's behaviour as documented at access date. The *pattern* and the *failure modes* are the durable content; the specific query syntax is not |
| Confidence | `VERIFIED` |
| Related requirements | `FR-SEARCH-02`, `FR-SEARCH-03` |
| Related decisions | `D-026`, `D-103` |
| Related open questions | `CL-SEARCH-002`, `CL-SEARCH-003`, `CL-SEARCH-014` |
| Next verification step | Ask for a file count and total size. **Until then no engine can be chosen, and choosing one would be guessing at a number nobody has measured** |

### CR-016 — SQLite FTS5 as the conservative default

| Field | Value |
|---|---|
| Research ID | `CR-016` |
| Topic | Search indexing |
| Question investigated | Is there a search capability that needs no additional service, on modest hardware? |
| Source title | SQLite FTS5 Extension |
| Publisher | SQLite (maintained open-source project documentation) |
| URL or repository path | `https://sqlite.org/fts5.html` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../resource_search.md`](../resource_search.md) §1 tiers 1–4 |
| Claim supported | `bm25()` ranking; `snippet()` and `highlight()`; prefix and Boolean/NEAR queries; `content=''` and `content='table'` modes let an index exist **without a second copy of the source document**, which matters because CloudLight must not duplicate school files; `unicode61` provides default diacritic removal; ⛔ **the porter stemmer is documented as limited to "English language terms only"** |
| Limitations | ⛔ **The English-only stemmer is a hard limit for a multilingual school.** ❓ Whether this school needs Hindi, Marathi or any other script in search is `CL-UX-016` / `CL-UX-017`, both unanswered. `snippet()` and `highlight()` exist **because** snippets require stored text, so the no-second-copy benefit and snippet capability trade against each other. This is a candidate, not a selection — `D-103` is open |
| Version applicability | FTS5 is the current full-text module; FTS3/4 are legacy. Module behaviour is stable but options are added over time |
| Confidence | `VERIFIED` |
| Related requirements | `FR-SEARCH-01`, `NFR-PERF-05` |
| Related decisions | `D-023`, `D-103` |
| Related open questions | `CL-SEARCH-014`, `CL-UX-016`, `CL-UX-017` |
| Next verification step | Ask which languages staff would search in. A one-line answer that determines whether the default configuration is usable |

### CR-017 — Extraction tooling and file-change notification

| Field | Value |
|---|---|
| Research ID | `CR-017` |
| Topic | Indexing pipeline |
| Question investigated | What could extract text from school files and notice when they change? |
| Source title | Apache Tika; Inotify — a powerful yet simple file change notification system |
| Publisher | Apache Software Foundation and the Linux kernel project |
| URL or repository path | `https://tika.apache.org/` · `https://www.kernel.org/doc/html/latest/filesystems/inotify.html` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../resource_search.md`](../resource_search.md) §5.4, §9.2 |
| Claim supported | Tika *"detects and extracts metadata and text from over a thousand different file types … useful for search engine indexing"*, with 4.x moving parsing into *"crash-isolated forked processes"* — the isolation [`../resource_browser.md`](../resource_browser.md) §8.2 requires. Inotify provides kernel file-change notification, with watches attached to an inotify instance and ordering guaranteed by a single queue |
| Limitations | ⚠️ **Named as examples, not selections.** ⛔⛔ **No extraction has been run against any school document.** ⛔ Inotify is **not** evidence that this school's storage, OS or host supports it — ❌ **over NFS the mechanism does not work as people expect**, a limitation this source does not address and which is not recorded elsewhere in the project. If the school storage is NFS, the change-notification design needs rethinking entirely |
| Version applicability | Tika 4.x is the current line with the forked-process behaviour; 1.x/2.x differ materially. Inotify behaviour is Linux-kernel-specific and absent on Windows, SMB and NFS |
| Confidence | `PLAUSIBLE_REQUIRES_TESTING` |
| Related requirements | `FR-SEARCH-01`, `FR-RES-02` |
| Related decisions | `D-023`, `D-103` |
| Related open questions | `CL-NAS-003`, `CL-NAS-015`, `CL-SEARCH-014` |
| Next verification step | Determine the storage protocol first. **If it is NFS, inotify-based change detection is not viable and this entry must be revisited** |

---

## 4. Local AI inference and retrieval-augmented generation

### CR-018 — The memory rule of thumb for local models

| Field | Value |
|---|---|
| Research ID | `CR-018` |
| Topic | Local inference sizing |
| Question investigated | Roughly how much RAM does a local model of a given size need? |
| Source title | Optimizing LLMs for Speed and Memory — Hugging Face; Quantization overview; QLoRA: Efficient Finetuning of Quantized LLMs |
| Publisher | Hugging Face (project documentation) and Dettmers et al. (peer-reviewed preprint) |
| URL or repository path | `https://huggingface.co/docs/transformers/llm_tutorial_optimization` · `https://huggingface.co/docs/transformers/quantization/overview` · `https://arxiv.org/abs/2305.14314` |
| Publication or update date | QLoRA arXiv:2305.14314 — 2023 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../ai_assistant.md`](../ai_assistant.md) §6.1; [`../network_and_hosting.md`](../network_and_hosting.md) §4.2 |
| Claim supported | The load rule of thumb: `~4×X` GB at float32 and `~2×X` GB at bfloat16/float16 for an `X`-billion-parameter model; 8-bit and 4-bit quantisation are established techniques for running capable models on limited hardware |
| Limitations | ⚠️⛔ **A general rule of thumb for *loading*, not a performance guarantee and not an inference-residency figure.** ⛔ **The `0.6–0.7 GB per billion parameters at 4-bit` figure used elsewhere in CloudLight is this project's own derivation, not a vendor constant**, and [`../sources.md`](../sources.md) §4 #6 records it as unsourced. ⚠️ QLoRA is a **fine-tuning** paper — inference-time memory behaviour is a different claim. ❓ **We do not know whether a GPU exists at all** (`CL-AI-009`, unanswered), and **no model has been downloaded or run** |
| Version applicability | Figures are general to quantised transformer inference and move with quantisation schemes and kernels. Must be re-derived for any specific model |
| Confidence | `PLAUSIBLE_REQUIRES_TESTING` |
| Related requirements | `FR-AI-01`, `NFR-SCA-03` |
| Related decisions | `D-024`, `D-105` |
| Related open questions | `CL-AI-009`, `CL-DEC-010` |
| Next verification step | ⛔ **Do not size a host around inference.** Size it for the core product. If AI is later approved, benchmark an actual model on actual hardware — a rule of thumb is not a measurement |

### CR-019 — Running open-weight models locally is ordinary, not sufficient

| Field | Value |
|---|---|
| Research ID | `CR-019` |
| Topic | Local inference feasibility |
| Question investigated | Is running a capable model locally on ordinary hardware an established practice, or an exotic one? |
| Source title | llama.cpp; Ollama |
| Publisher | ggml-org and Ollama (open-source project documentation) |
| URL or repository path | `https://github.com/ggml-org/llama.cpp` · `https://github.com/ollama/ollama` |
| Publication or update date | UNKNOWN — both projects move continuously |
| Date accessed | `2026-10-11` |
| Relevant section | [`../ai_assistant.md`](../ai_assistant.md) §6 |
| Claim supported | Open-weight models are routinely run locally on CPU with no GPU, and local model serving is packaged rather than exotic |
| Limitations | ⚠️ **A project README is not a benchmark on this hardware.** These are cited as evidence that the *approach is ordinary*, never that it is *sufficient for this school's corpus on a host nobody has identified*. ❓ Both `CL-AI-009` (hardware) and `CL-AI-001` (is external AI even permitted) are unanswered. ⚠️⛔ **A permanently offline school cannot download model weights** — every binary, model and tokeniser must be fetched on an internet-connected machine and transferred in advance, and a missed file breaks setup silently |
| Version applicability | Both projects change frequently; version-specific behaviour should not be assumed stable across months |
| Confidence | `PLAUSIBLE_REQUIRES_TESTING` |
| Related requirements | `FR-AI-01`, `FR-AI-12` |
| Related decisions | `D-024`, `D-105` |
| Related open questions | `CL-AI-001`, `CL-AI-009`, `CL-DEC-010` |
| Next verification step | Ask the Principal whether any AI service is permitted at all (`CL-AI-001`). **If the answer is no, the entire domain is closed and no hardware question matters** |

### CR-020 — Prompt injection is not fully mitigable

| Field | Value |
|---|---|
| Research ID | `CR-020` |
| Topic | AI safety |
| Question investigated | Can an AI assistant over school documents be made safe by filtering? |
| Source title | LLM01:2025 Prompt Injection — OWASP Gen AI Security Project; LLM Prompt Injection Prevention Cheat Sheet — OWASP |
| Publisher | OWASP (independent security community foundation) |
| URL or repository path | `https://genai.owasp.org/llmrisk/llm01-prompt-injection/` · `https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html` |
| Publication or update date | The LLM Top 10 entry is labelled `LLM01:2025` |
| Date accessed | `2026-10-11` |
| Relevant section | [`../ai_assistant.md`](../ai_assistant.md) §8.1, §8.3; [`../security_and_privacy.md`](../security_and_privacy.md) §13 |
| Claim supported | **Indirect** injection arrives as input *"from external sources, such as websites or files"*; ⛔⛔ **RAG and fine-tuning do not fully mitigate it and fool-proof prevention is unclear**; scenario #4 is an attacker modifying a document inside a RAG corpus — which is *precisely* a school shared drive. Mitigations include *"use deterministic code to validate adherence"* to output formats, handling functions *"in code rather than providing them to the model"*, requiring human approval for high-risk actions, and *"segregate and identify external content"*. OWASP's own framing is that **no mitigation is complete** |
| Limitations | ⛔ **The "does not fully mitigate" finding is the important one.** It means the assistant cannot be made safe by filtering alone. ⚠️ **Partly sourced:** the leap from "not fully mitigated" to "not deployable in a classroom" is this project's own judgement and is recorded as such in [`../sources.md`](../sources.md) §4 #10. **This is a general risk register entry and says nothing about any specific model, corpus or deployment** |
| Version applicability | LLM01:2025 reflects the state of the risk list at that edition; the risk landscape moves fast and the entry should be re-read |
| Confidence | `VERIFIED` |
| Related requirements | `FR-AI-05`, `FR-AI-06` |
| Related decisions | `D-007`, `D-105` |
| Related open questions | `CL-AI-001`, `CL-SEC-009` |
| Next verification step | ⛔ **This one cannot be retired by testing.** The finding is that no mitigation is complete. What can be retired is the *proposal to deploy one* — which is already deferred by `D-105` |

### CR-021 — The structural defence is removal, not filtering

| Field | Value |
|---|---|
| Research ID | `CR-021` |
| Topic | AI safety |
| Question investigated | If filtering cannot make an AI assistant safe, what structural property can? |
| Source title | The lethal trifecta for AI agents: private data, untrusted content, and external communication |
| Publisher | Simon Willison (named independent practitioner) |
| URL or repository path | `https://simonwillison.net/2025/Jun/16/the-lethal-trifecta` |
| Publication or update date | 2025-06-16 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../ai_assistant.md`](../ai_assistant.md) §8.4 |
| Claim supported | The three-property framing — private data, untrusted content, external communication — and the conclusion that **removing one leg is the reliable structural defence**. CloudLight's position removes the third: the assistant has no external communication channel, so no injected instruction can exfiltrate anything |
| Limitations | ⚠️ **A named practitioner's framing, not a standard, and not peer-reviewed.** It is adopted here precisely because it argues for a *structural* boundary rather than filtering — but it remains one person's argument, and CloudLight inherits its force rather than verifying it. ❓ Whether the school would accept an assistant with no external capability is unanswered (`CL-AI-001`) |
| Version applicability | A 2025 argument about agent architectures. It generalises to any tool-using agent; its specific discussion of current models will age |
| Confidence | `PLAUSIBLE_REQUIRES_TESTING` |
| Related requirements | `FR-AI-05`, `FR-AI-07` |
| Related decisions | `D-007`, `D-105` |
| Related open questions | `CL-AI-001`, `CL-DEC-010` |
| Next verification step | Treat as design rationale rather than evidence. If the assistant is ever built, this is the assumption most worth revisiting with an actual deployment review |

---

## 5. Document parsing, layout analysis, OCR and table structure

### CR-022 — OOXML files need no inference at all

| Field | Value |
|---|---|
| Research ID | `CR-022` |
| Topic | Document parsing |
| Question investigated | If the school's timetable is a Word or Excel file, is parsing it a machine-learning problem? |
| Source title | Structure of a WordprocessingML document; ISO/IEC 29500 — Office Open XML File Formats |
| Publisher | Microsoft (vendor documentation) and ISO/IEC (standard) |
| URL or repository path | `https://learn.microsoft.com/en-us/office/open-xml/word/structure-of-a-wordprocessingml-document` · `https://www.iso.org/standard/71691.html` |
| Publication or update date | ISO/IEC 29500 — 2016 editions and later parts; paywalled |
| Date accessed | `2026-10-11` |
| Relevant section | [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §3, §3.1 |
| Claim supported | ✅ Office Open XML documents are **ZIP packages containing XML parts** — verifiable by *"renam[e] the extension on the minimum document from .docx to .zip"*. Cell contents and cell addresses are therefore **readable without any inference whatsoever** |
| Limitations | ✅ **The strongest single finding in document parsing for this project.** ⛔ **And it is conditional on a fact nobody knows**: what format the school's timetable is actually in. ❓ No timetable, substitution sheet, staff list or class list has been seen, so no parser was run — **because no parser exists**. A merged-cell grid timetable is readable but the *semantics* of merged cells still require deliberate handling |
| Version applicability | OOXML has been stable since Office 2007. Legacy binary `.xls` and `.doc` formats are a **different** case and are **not** covered by this entry |
| Confidence | `VERIFIED` |
| Related requirements | `FR-TIME-04`, `FR-TIME-05` |
| Related decisions | `D-006` |
| Related open questions | `CL-TIME-009`, `CL-TIME-010`, `CL-DATA-019` |
| Next verification step | ⛔ **Ask the timetable owner for one sample file in its real format.** This single question determines whether the whole deterministic-first path works or whether the project faces OCR. It is the highest-value question in the parsing domain |

### CR-023 — Benchmarks that ran without OCR

| Field | Value |
|---|---|
| Research ID | `CR-023` |
| Topic | Document parsing |
| Question investigated | How fast and how accurate is modern document-layout analysis on CPU? |
| Source title | Docling Technical Report — Auer, Lysak, Nassar, Dolfi et al., IBM Research |
| Publisher | arXiv (unrefereed preprint, IBM Research) |
| URL or repository path | `https://arxiv.org/abs/2408.09869` |
| Publication or update date | arXiv:2408.09869v5 — 2024-12-09 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §3.1, §4 Tier 2 |
| Claim supported | Pipeline ordering; the PDF backend *"retrieves the programmatic text tokens, consisting of string content and its coordinates on the page"*; ⛔ **OCR is optional**, *"e.g. for scanned PDFs"*; layout detection at 72 dpi on a single CPU with sub-second latency; TableFormer 2–6 s per table on a standard CPU; ⛔ EasyOCR *"upwards of 30 seconds per page"* on CPU; ⛔ **the 225-page benchmark was run with OCR disabled**; ⛔ GPU acceleration is *"work-in-progress and largely untested"* |
| Limitations | ⚠️⛔⛔ **An unreviewed preprint, and its own benchmark ran with OCR disabled.** The attractive latency figures and the scanned-document case come from **different experiments**. If the school timetable is a scan, a photograph or a handwritten board, **none of those figures apply** — and this source's own numbers put OCR at upwards of 30 seconds per page on CPU. ⚠️ Because GPU acceleration is untested in the source itself, the CPU-vs-GPU comparison is **`UNKNOWN` here**, not merely unmeasured. ❓ No school document has been processed |
| Version applicability | A specific system at a specific preprint version. Later releases change both figures and pipeline order |
| Confidence | `PLAUSIBLE_REQUIRES_TESTING` |
| Related requirements | `FR-TIME-04`, `FR-TIME-06` |
| Related decisions | `D-006`, `D-104` |
| Related open questions | `CL-AIML-009`, `CL-SUB-006`, `CL-SEC-005` |
| Next verification step | ⛔ **This thread does not need a test; it needs a precondition.** Obtain one real timetable document and determine its format. Only then is any of this relevant |

### CR-024 — Table-structure models were trained on the wrong documents

| Field | Value |
|---|---|
| Research ID | `CR-024` |
| Topic | Document parsing |
| Question investigated | Do table-recognition benchmarks apply to a school timetable? |
| Source title | TableFormer: Table Structure Understanding with Transformers (Nassar et al., CVPR 2022); DocLayNet: A Large Human-Annotated Dataset for Document-Layout Analysis (Pfitzmann et al., KDD 2022); DS4SD/DocLayNet dataset repository |
| Publisher | CVF/KDD (peer-reviewed) and DS4SD (dataset repository) |
| URL or repository path | `https://arxiv.org/abs/2203.01017` · `https://arxiv.org/abs/2206.01062` · `https://github.com/DS4SD/DocLayNet` |
| Publication or update date | CVPR 2022; KDD 2022 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §4 Tier 2 |
| Claim supported | TableFormer predicts logical row/column structure and header/body roles, with documented tolerance of *"partial or no borderlines, empty cells, rows or columns, cell spans"*; layout analysis is trained on a **labelled corpus of 80,863 human-annotated pages across 11 layout classes**, human-annotated by trained experts with fixed splits and per-page bounding boxes |
| Limitations | ⛔⛔ **The corpus is finance, science, patents, tenders, law and manuals — not school timetables.** This is the strongest available argument *against* trusting general layout benchmarks, and it is cited here for that purpose. ⛔ **A merged-cell grid timetable is close to TableFormer's worst case**: cell spans are exactly the documented tolerance limit, not the happy path. No school timetable has been evaluated |
| Version applicability | Models and datasets at their published versions. The dataset card's value here is its description of the **training distribution**, which does not change |
| Confidence | `VERIFIED` |
| Related requirements | `FR-TIME-04`, `FR-TIME-06` |
| Related decisions | `D-006`, `D-104` |
| Related open questions | `CL-TIME-009`, `CL-AIML-009` |
| Next verification step | ⛔ **This claim is a reason to prefer deterministic parsing, not a problem to test.** If a real timetable sample ever becomes available, evaluate against it and accept that a general benchmark says nothing |

### CR-025 — OCR language coverage is not handwriting coverage

| Field | Value |
|---|---|
| Research ID | `CR-025` |
| Topic | OCR |
| Question investigated | What can OCR actually read on a classroom board? |
| Source title | Tesseract OCR; EasyOCR |
| Publisher | tesseract-ocr and JaidedAI (open-source project documentation) |
| URL or repository path | `https://github.com/tesseract-ocr/tesseract` · `https://github.com/JaidedAI/EasyOCR` |
| Publication or update date | UNKNOWN — both move continuously |
| Date accessed | `2026-10-11` |
| Relevant section | [`../resource_search.md`](../resource_search.md) §9.3; [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §4 Tier 3 |
| Claim supported | Tesseract *"can recognize more than 100 languages"* with PNG/JPEG/TIFF input; the project's own advice that *"in many cases, in order to get better OCR results, you'll need to improve the quality of the image"*; training data must be supplied per language. EasyOCR is *"Ready-to-use ocr with 80+ supported languages"* |
| Limitations | ⛔⛔ **Language count says nothing about handwriting.** ⛔ **An unsupported script is not read poorly — it is not read.** Vendor-reported language counts are a marketing figure about a different capability than the one a classroom board needs. ⚠️ Named as examples, **not selections**. ❓ Whether photographing boards is even permitted is `CL-SUB-006`, unanswered, and if it is not, this entire domain is moot |
| Version applicability | Both projects change frequently; accuracy figures are model- and version-specific and none has been measured here |
| Confidence | `PLAUSIBLE_REQUIRES_TESTING` |
| Related requirements | `FR-TIME-06`, `NFR-PRV-04` |
| Related decisions | `D-006`, `D-104`, `X-06` |
| Related open questions | `CL-AIML-010`, `CL-SUB-006`, `CL-SEC-005` |
| Next verification step | ⛔ **Ask whether photographing classroom boards is permitted.** A `NO` closes this thread permanently and is cheaper than any benchmark |

### CR-026 — PDF, CSV and export drift

| Field | Value |
|---|---|
| Research ID | `CR-026` |
| Topic | Document formats |
| Question investigated | What do PDF, CSV and cloud-export formats actually guarantee about extractable content? |
| Source title | ISO 32000-1 (PDF), published copy — Adobe Acrobat SDK documentation portal; RFC 4180 — Common Format and MIME Type for CSV; Create, view, or download a file — Google Docs Editors Help |
| Publisher | ISO (standard, via Adobe's documentation portal), IETF (standard), Google (support documentation) |
| URL or repository path | `https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf` · `https://www.rfc-editor.org/rfc/rfc4180` · `https://support.google.com/docs/answer/49114` |
| Publication or update date | PDF published copy is the 2008 revision; RFC 4180 is 2005 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../timetable_and_substitutions.md`](../timetable_and_substitutions.md) §3 |
| Claim supported | PDF page content streams carry text-showing operators and graphics; ⛔ **a scanned page is an image with no character data to retrieve** — the text-layer versus image-only distinction. RFC 4180: a field containing a comma, a quote or a line break **must** be quoted. A Google Sheets file is exported as a chosen **file type** — a point-in-time download, **not a live link** |
| Limitations | ⚠️ The Adobe-hosted PDF copy may lag the current ISO revision; the *principle* — text layer versus image-only — is stable regardless. ⚠️ **CSV is a common, silent data-corruption failure**: a school spreadsheet exported to CSV will contain commas and newlines inside cells. ⚠️ Google support documentation is not an API contract — it is cited only to establish the **snapshot-drift** failure mode, that an export is stale the moment it is taken. ❓ No export has been taken and no CSV parsed |
| Version applicability | RFC 4180 is stable. PDF has revisions. Google Sheets export behaviour changes without notice |
| Confidence | `VERIFIED` |
| Related requirements | `FR-TIME-04`, `FR-DATA-07` |
| Related decisions | `D-006` |
| Related open questions | `CL-TIME-009`, `CL-DATA-019` |
| Next verification step | ⛔ **Ask for a sample in whatever format the school already uses**, including whether it is a Google Sheet. If it is, the snapshot-drift problem is real on day one and must be designed for, not discovered later |

---

## 6. Authentication, authorisation, sessions, MFA and secrets

### CR-027 — NIST SP 800-63B Rev. 4 — timeouts are both required

| Field | Value |
|---|---|
| Research ID | `CR-027` |
| Topic | Identity and session policy |
| Question investigated | What does current US federal guidance actually require of session timeouts and rate limiting? |
| Source title | Digital Identity Guidelines: Authentication and Authenticator Management (NIST SP 800-63B Rev. 4) |
| Publisher | NIST (US government standards body) |
| URL or repository path | `https://pages.nist.gov/800-63-4/sp800-63b.html` |
| Publication or update date | Rev. 4 — current edition at access date |
| Date accessed | `2026-10-11` |
| Relevant section | [`../security_and_privacy.md`](../security_and_privacy.md) §6 controls 3, 7 |
| Claim supported | ✅ **Both an overall reauthentication timeout and an inactivity timeout are required, not one or the other** — AAL1 inactivity *"MAY be applied but is not required"*; AAL2 overall ≤ 24 h and inactivity ≤ 1 h. Rate-limiting is a **SHALL** at every AAL. Passwords as single-factor *"SHALL be a minimum of 15 characters"*; verifiers *"SHALL NOT require subscribers to change passwords periodically"* but *"SHALL force a change if there is evidence that the authenticator has been compromised"*. The guidelines *"are not intended to constrain the development or use of standards outside of this purpose"* |
| Limitations | ⚠️ **US federal guidance. Whether it is binding at an Indian school is unknown** (`CL-AUTH-016`). It is adopted as **good practice, not as law**. ⚠️ **Rev. 3 is cited separately by [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) and disagrees on timeouts** — see [`../sources.md`](../sources.md) §5.3. Both are cited here; the conflict is recorded, not silently resolved |
| Version applicability | Rev. 4 is current at access date. **Rev. 5 will change it.** Re-read before relying on any specific figure |
| Confidence | `VERIFIED` |
| Related requirements | `FR-AUTH-03`, `FR-AUTH-04`, `NFR-SEC-03` |
| Related decisions | `D-005` |
| Related open questions | `CL-AUTH-016`, `CL-SEC-002` |
| Next verification step | ⛔ **The timeout figures need no test — they need adoption.** Write them into the requirements and stop arguing about them. The *open* question is whether the school accepts the values, which is `CL-AUTH-016` |

### CR-028 — SMS is a restricted authenticator

| Field | Value |
|---|---|
| Research ID | `CR-028` |
| Topic | Second factors |
| Question investigated | Can SMS be the second factor for teacher authentication? |
| Source title | Digital Identity Guidelines: Authentication and Lifecycle Management (NIST SP 800-63B Rev. 3); Multifactor Authentication Cheat Sheet — OWASP; Authentication Cheat Sheet — OWASP |
| Publisher | NIST (US government standards body) and OWASP (security community foundation) |
| URL or repository path | `https://pages.nist.gov/800-63-3/sp800-63b.html` · `https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html` · `https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html` |
| Publication or update date | Rev. 3 superseded by Rev. 4 (`CR-027`); OWASP cheat sheets continuously revised |
| Date accessed | `2026-10-11` |
| Relevant section | [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) §4 |
| Claim supported | ⛔ SMS/PSTN is designated a **"restricted" authenticator** — SS7 interception, SIM-swap and number-porting; failed attempts limited to **no more than 100 consecutive attempts**; OTP validity **ten minutes or less**, single-use and invalidated on use; single-factor authenticators of **at least 8 digits**; AAL2 requires two distinct factors. OWASP ranks factors by strength, documents push-bombing / MFA-fatigue risks, and requires **cryptographically secure** random-number generation for tokens |
| Limitations | ⚠️ **Rev. 3 is superseded.** It is cited here because [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) cites it and because the 8-digit single-factor rule is relevant to the six-digit-code finding in [`../decision_log.md`](../decision_log.md) `D-005`. ⚠️ **A cheat sheet is a summary, not a specification.** ❓ Nothing here establishes what authentication this school uses or would permit — `CL-AUTH-001`, `CL-AUTH-013` |
| Version applicability | Rev. 3 is historical. The restricted-authenticator designation is stable across Rev. 3 and Rev. 4 |
| Confidence | `VERIFIED` |
| Related requirements | `FR-AUTH-05`, `FR-AUTH-06`, `NFR-SEC-04` |
| Related decisions | `D-005` |
| Related open questions | `CL-AUTH-013`, `CL-AUTH-016` |
| Next verification step | ⛔ **None needed for the finding.** SMS is excluded by standard and by judgement. The open question is whether teachers have smartphones capable of passkeys or TOTP at all — `CL-AUTH-013` |

### CR-029 — WebAuthn passkeys and where they fit

| Field | Value |
|---|---|
| Research ID | `CR-029` |
| Topic | Second factors |
| Question investigated | Is a phishing-resistant second factor achievable on school devices? |
| Source title | Web Authentication API: An Introduction — W3C TAG |
| Publisher | W3C Technical Architecture Group |
| URL or repository path | `https://www.w3.org/2021/03/bringing-passwordless-login-with-webauthn.html` |
| Publication or update date | 2021-03 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) §4 |
| Claim supported | WebAuthn passkeys are **origin-bound and phishing-resistant**, and the private key never leaves the authenticator |
| Limitations | ⚠️ **A W3C TAG article, not the specification** — it is explanatory material written in 2021, and the passkey ecosystem has moved considerably since. ❓ **Whether school devices support passkeys at all is unknown** (`CL-AUTH-013`), and passkeys require teacher-owned authenticators, which raises a provisioning question nobody has asked: `CL-AUTH-014` |
| Version applicability | A 2021 TAG article describing a since-maturing platform. **The security properties are stable; the deployment guidance is dated** |
| Confidence | `PLAUSIBLE_REQUIRES_TESTING` |
| Related requirements | `FR-AUTH-06`, `FR-AUTH-07` |
| Related decisions | `D-005` |
| Related open questions | `CL-AUTH-013`, `CL-AUTH-014` |
| Next verification step | Ask whether teachers have personal smartphones the school would permit for authentication. **If not, passkeys are unavailable and TOTP becomes the realistic ceiling** |

### CR-030 — Session management for an unattended shared device

| Field | Value |
|---|---|
| Research ID | `CR-030` |
| Topic | Sessions |
| Question investigated | What session controls apply to a classroom display nobody is logged into? |
| Source title | Session Management Cheat Sheet — OWASP; `Set-Cookie` header (MDN) |
| Publisher | OWASP (security community foundation) and MDN Web Docs |
| URL or repository path | `https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html` · `https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) §9 |
| Claim supported | `httpOnly`, `Secure`, `SameSite`; ⛔ no session identifier in the URL; ⛔ none in `localStorage`; session ID rotation on authentication; **server-side invalidation on logout and privilege change**; idle **and** absolute timeout as **distinct** controls. ⛔ **`HttpOnly` has a documented limitation — the cookie is still *sent* on JavaScript-initiated requests**; `SameSite=Strict/Lax/None`; the `__Host-` prefix requirements |
| Limitations | ⛔⛔ **Directly relevant to the unattended-display problem**: an open kiosk session is a session with **no human present to end it**, so every idle-timeout figure matters more than usual. ⚠️ **`HttpOnly` is not "invisible to scripts"** — it prevents *reading* the value, not the browser *sending* it, and this project has stated the limitation accurately. ❓ Nothing here has been configured anywhere; there is no server to configure |
| Version applicability | MDN documents current web-platform behaviour. `SameSite` defaults have changed across browser versions and **must be set explicitly** rather than relied on |
| Confidence | `VERIFIED` |
| Related requirements | `FR-AUTH-03`, `FR-AUTH-14`, `NFR-SEC-02` |
| Related decisions | `D-009` |
| Related open questions | `CL-AUTH-020`, `CL-SEC-002` |
| Next verification step | ⛔ **Nothing to test — nothing is built.** The step is to write these controls into the requirements, which `FR-AUTH-03` and `FR-AUTH-14` already do |

### CR-031 — Authorisation decided server-side, every request

| Field | Value |
|---|---|
| Research ID | `CR-031` |
| Topic | Authorisation |
| Question investigated | Where must an access-control decision be made for it to be trustworthy? |
| Source title | Access Control Cheat Sheet — OWASP; Authorization Cheat Sheet — OWASP; Secrets Management Cheat Sheet — OWASP; Logging Cheat Sheet — OWASP |
| Publisher | OWASP (security community foundation) |
| URL or repository path | `https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html` · `https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html` · `https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html` · `https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../authentication_and_authorisation.md`](../authentication_and_authorisation.md) §11; [`../security_and_privacy.md`](../security_and_privacy.md) §5, §6, §8 |
| Claim supported | Least privilege, separation of duties, deny by default, and **access control decisions made server-side on every request**, not once at sign-in; ⛔ *"Never rely on client-side access control checks"*; a single server-side default-deny policy. Secrets supplied at runtime, never in source control; *"rotate, never reuse"*; ⛔ **a secret in version control must be assumed compromised**. ✅ *"Data should not normally be logged"*, with credentials among the excluded items; log injection prevention; ⛔ **log access is itself access-controlled data** |
| Limitations | ⚠️ **Community guidance, and cheat sheets are summaries rather than specifications.** ⚠️ The Access Control and Authorization cheat sheets are **not the same document** and [`../sources.md`](../sources.md) cites them separately because they are. ✅ The secrets point **independently confirms a finding already made in this repository**: [`../../start_opencode_server.sh`](../../../start_opencode_server.sh) contains a hardcoded plaintext password, *is* listed in `.gitignore`, and is therefore not committed. No document quotes the credential value |
| Version applicability | Continuously revised; principles are stable, specifics age |
| Confidence | `VERIFIED` |
| Related requirements | `FR-SEARCH-02`, `FR-ADMIN-09`, `NFR-SEC-07`, `NFR-MNT-04` |
| Related decisions | `D-026`, `D-010` |
| Related open questions | `CL-SEC-019`, `CL-GIT-014`, `CL-OPS-011` |
| Next verification step | ⛔ **Not a research question.** These are requirements, already written. The verification step is implementation-time, and implementation has not begun |

### CR-032 — Code signing verified on the consumer

| Field | Value |
|---|---|
| Research ID | `CR-032` |
| Topic | Update security |
| Question investigated | Where must a software signature be checked for the check to mean anything? |
| Source title | Code Signing Cheat Sheet — OWASP; File Upload Cheat Sheet — OWASP |
| Publisher | OWASP (security community foundation) |
| URL or repository path | `https://cheatsheetseries.owasp.org/cheatsheets/Code_Signing_Cheat_Sheet.html` · `https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html` |
| Publication or update date | UNKNOWN |
| Date accessed | `2026-10-11` |
| Relevant section | [`../ota_updates.md`](../ota_updates.md) §5; [`../security_and_privacy.md`](../security_and_privacy.md) §6 control 11 |
| Claim supported | ✅ **The signature must be verified by the software consuming the artefact, not by the distribution point** — this is the source of the distinction that matters for OTA: verification happens **on the board**, not on the server. The private key is the crown jewel; key compromise means every artefact is suspect. Separately: ⛔ *"ZIP bombs, XML bombs … or simply huge files in a way to fill the server storage"*; *"don't trust the Content-Type header as it can be spoofed"*; size limits computed **after decompression**; ✅ **store uploads outside the webroot and never execute or interpret them server-side** |
| Limitations | ⚠️ **The File Upload sheet is written for uploads; applying it here to extraction from storage is an analogy, and the analogy is this project's own.** ⚠️ No key exists, no artefact has been signed, and no board has been enrolled. ❓ **Where the signing key would live is `CL-OTA-005`, which requires school permission**, and holding a signing key is listed among the sixteen permissions in [`../feasibility_report.md`](../feasibility_report.md) §19 |
| Version applicability | Platform-agnostic principles |
| Confidence | `VERIFIED` |
| Related requirements | `FR-OTA-01`, `NFR-SEC-06`, `NFR-SEC-08` |
| Related decisions | `D-025` |
| Related open questions | `CL-SEC-013`, `CL-OTA-005`, `CL-OTA-004` |
| Next verification step | ⛔ **Nothing to test until a key exists.** Ask the school whether it will hold release signing keys — `CL-OTA-005`. Until then `D-025` is a design intent, not a capability |

### CR-033 — Incident response needs a named human

| Field | Value |
|---|---|
| Research ID | `CR-033` |
| Topic | Operations |
| Question investigated | What does preparing for an incident require when there is no system and no IT team? |
| Source title | Incident Response Recommendations and Considerations for Cybersecurity Risk Management: A CSF 2.0 Community Profile (NIST SP 800-61 Rev. 3) |
| Publisher | NIST (US government standards body) |
| URL or repository path | `https://csrc.nist.gov/pubs/sp/800/61/r3/final` |
| Publication or update date | Published 2025-04, superseding Rev. 2 (2012) |
| Date accessed | `2026-10-11` |
| Relevant section | [`../security_and_privacy.md`](../security_and_privacy.md) §10 |
| Claim supported | Incident-response preparation should be integrated with organisational risk management, and the existence of a **defined, named response capability** is central to it |
| Limitations | ⚠️ **US federal guidance.** No incident has occurred, because no system exists. ⛔❓ **This exposes a structural gap**: `CL-SRV-020` asks whether the host should report failures somewhere the school already monitors, and `CL-NET-020` asks whether the school network is monitored — **both presuppose a monitoring capability nobody has confirmed exists.** If the answer is no, both questions are moot and **there is no path by which a host failure reaches a person**, which makes `NFR-AVL-05` unachievable as written rather than merely unmet |
| Version applicability | Current revision as of access date |
| Confidence | `VERIFIED` |
| Related requirements | `NFR-AVL-05`, `NFR-MNT-05` |
| Related decisions | `D-101` |
| Related open questions | `CL-SEC-026`, `CL-SRV-020`, `CL-NET-020` |
| Next verification step | ⛔ **Ask who at this school would be the named contact.** If nobody exists, that is a governance finding, and it should be raised before any deployment rather than discovered after |

---

## 7. Software update security

### CR-034 — The Update Framework

| Field | Value |
|---|---|
| Research ID | `CR-034` |
| Topic | Update security |
| Question investigated | Does "check the signature" constitute a secure update design? |
| Source title | The Update Framework (TUF) — Specification; The Update Framework — About |
| Publisher | The Update Framework project (open specification) |
| URL or repository path | `https://theupdateframework.io/specification/latest/` · `https://theupdateframework.io/about/` |
| Publication or update date | UNKNOWN — `latest` specification, continuously revised |
| Date accessed | `2026-10-11` |
| Relevant section | [`../ota_updates.md`](../ota_updates.md) §5 |
| Claim supported | ✅ TUF is a **published specification, not a product**. It defines **root, snapshot, targets and timestamp** roles with **separate keys**, and requires **expiry on metadata**, so a frozen or stale repository state becomes **invalid on its own**. ✅ Its threat model **names** the rollback, freeze, fast-forward and mix-and-match attacks it exists to resist — confirming that a signature alone is *not* the design |
| Limitations | ⚠️ **Presented as the reference design, not as a decision to adopt.** Adopting TUF is a significant piece of work and **is not proposed**. ⛔ It does not tell us what Windows mechanisms are available on these boards. ⚠️ [`../ota_updates.md`](../ota_updates.md) §10 discusses kiosk elevation, scheduled tasks and Group Policy constraints and **explicitly cites no vendor source for them** — a deliberate gap recorded rather than filled with an unverified citation |
| Version applicability | The specification evolves; the four-role structure and expiry requirement have been stable for years and are expected to remain |
| Confidence | `VERIFIED` |
| Related requirements | `FR-OTA-01`, `FR-OTA-02`, `FR-OTA-07` |
| Related decisions | `D-025` |
| Related open questions | `CL-OTA-004`, `CL-OTA-007`, `CL-OTA-010`, `CL-OTA-012` |
| Next verification step | ⛔ **Do not implement TUF.** Implement `D-025` — signed artefacts verified on the consuming device — and record the rollback and freeze resistance as **not implemented**. If a threat model is ever needed in full, revisit this |

---

## 8. Accessibility

### CR-035 — The accessibility gap, recorded as a gap

| Field | Value |
|---|---|
| Research ID | `CR-035` |
| Topic | Accessibility |
| Question investigated | What accessibility standard governs the classroom display? |
| Source title | **None.** No WCAG, no EN 301 549, no school policy is cited anywhere in the CloudLight documents |
| Publisher | — |
| URL or repository path | — |
| Publication or update date | — |
| Date accessed | — |
| Relevant section | [`../requirements.md`](../requirements.md) §3.5 |
| Claim supported | **Nothing.** This entry exists to record that the accessibility requirements — contrast ratios (4.5:1, 3:1), target sizes (44×44 px), no hover dependence, colour not the sole carrier of meaning — rest on **no source at all** |
| Limitations | ⛔⛔ **The numbers were chosen by this project as arguable proposals, not derived from any standard.** They are widely used general thresholds, but they are **uncited here**, and [`../classroom_dashboard.md`](../classroom_dashboard.md) states it carries **no external citations at all** — its display states, countdown tiers and failure catalogue are entirely engineering judgement |
| Version applicability | Not applicable |
| Confidence | `PLAUSIBLE_REQUIRES_TESTING` |
| Related requirements | `NFR-ACC-01` … `NFR-ACC-08` |
| Related decisions | — |
| Related open questions | `CL-UX-004` (whether any accessibility requirement is binding rather than advisory) |
| Next verification step | ⛔ **The highest-value gap in this register.** Consult WCAG 2.2 and, separately, ask the school whether any accessibility policy applies. Until then every accessibility figure in [`../requirements.md`](../requirements.md) §3.5 is a proposal and must be described as one |

---

## 9. Internal repository documents

### CR-036 — The CloudLight questionnaire

| Field | Value |
|---|---|
| Research ID | `CR-036` |
| Topic | Open questions |
| Question investigated | Where is every unresolved CloudLight question recorded, and who owns each? |
| Source title | Additional CloudLight Questionnaire |
| Publisher | This repository (internal) |
| URL or repository path | [`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md) |
| Publication or update date | 2026-10-11 |
| Date accessed | `2026-10-11` |
| Relevant section | Throughout every CloudLight document |
| Claim supported | **576 `CL-*` questions** across 26 categories, each with priority, respondent, dependency, evidence requirement and status. ⚠️ **Every one of the 576 is currently `UNANSWERED`** |
| Limitations | ⛔ **A question bank is not an answer set.** ⚠️ It records what nobody knows; citing it as a source for a *finding* inverts its purpose. ⚠️ The questionnaire is **SHANA-owned** and was **read only** in compiling this register |
| Version applicability | Current as of access date; grows as documents cite new IDs |
| Confidence | `VERIFIED` |
| Related requirements | All |
| Related decisions | All |
| Related open questions | All |
| Next verification step | ⛔ **This is the work item.** Not a research question. Obtain answers in writing, beginning with `CL-SRV-001`, `CL-NET-003` and `CL-NET-008` |

### CR-037 — CloudLight's own architecture and decisions

| Field | Value |
|---|---|
| Research ID | `CR-037` |
| Topic | Architecture records |
| Question investigated | What has this project already decided, and what remains provisional? |
| Source title | CloudLight architecture; decision log; requirements |
| Publisher | This repository (internal) |
| URL or repository path | [`../architecture.md`](../architecture.md) · [`../decision_log.md`](../decision_log.md) · [`../requirements.md`](../requirements.md) |
| Publication or update date | 2026-10-11 |
| Date accessed | `2026-10-11` |
| Relevant section | `D-001` … `D-010` (decided) · `D-021` … `D-026` (provisional) · `D-101` … `D-106` (open) · `X-01` … `X-12` (rejected) |
| Claim supported | The evidence-label vocabulary used throughout CloudLight; the module list and boundaries; `D-003` no motor commands; `D-004` full operation when SHANA is offline; `D-005` six-digit code limits; `D-006` deterministic before AI; `D-008` human approval gates publication; `D-021` modular monolith; `D-022` browser-first; `D-024` local inference preferred; `D-025` signed artefacts; `D-026` open-time re-verification |
| Limitations | ⚠️ **These are this project's own decisions and its own reasoning.** Citing them is citing ourselves. ✅ They are valuable precisely because they are *explicitly labelled* by strength — `D-021` through `D-026` are **provisional defaults**, not settled choices. ⚠️ [`../sources.md`](../sources.md) §5.8 records that several CloudLight documents cite repository paths that do not resolve; this register uses only paths verified to resolve |
| Version applicability | Current as of access date |
| Confidence | `VERIFIED` |
| Related requirements | All |
| Related decisions | All |
| Related open questions | — |
| Next verification step | ⛔ **Not research.** Decisions change through the process in [`../decision_log.md`](../decision_log.md) §5, including their reversal conditions |

### CR-038 — SHANA's boundary, read-only

| Field | Value |
|---|---|
| Research ID | `CR-038` |
| Topic | Cross-system boundary |
| Question investigated | What has SHANA stated about the CloudLight boundary, and does it agree with CloudLight's proposal? |
| Source title | SHANA ↔ CloudLight Integration; `data/schemas/README.md`; ADR-014 — CloudLight never controls robot motion |
| Publisher | This repository (internal, **SHANA-owned**) |
| URL or repository path | [`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md) · [`../../../data/schemas/README.md`](../../../data/schemas/README.md) · [`../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md`](../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md) |
| Publication or update date | 2026-10-11 |
| Date accessed | `2026-10-11` |
| Relevant section | [`../../shana/architecture/cloudlight_integration.md`](../../shana/architecture/cloudlight_integration.md) §1–§3 |
| Claim supported | The ownership rule: SHANA owns `docs/shana/**` and may read `docs/cloudlight/**` **read-only, never modify**. SHANA owns robot motion, localisation, obstacle avoidance, battery management, route validation and physical emergency response. The principle: **any component whose failure could injure a person must not depend on an external system's availability.** The proposed interface is explicitly marked **`DESIGN_ONLY` — never implemented, never validated against a real partner**, and its schemas *"have never been parsed by code"*. `data/schemas/README.md`: *"Documentation only. No schema has been implemented. No database exists."* |
| Limitations | ⛔⛔ **These files were read, never modified — they are SHANA-owned and another agent owns them.** ⛔ **SHANA has not agreed to CloudLight's proposal.** The integration document states that any statement about CloudLight in SHANA documents is *a proposal, not a description of something that exists*, and CloudLight's [`../shana_integration.md`](../shana_integration.md) likewise does not claim SHANA's agreement. ⛔ **The actual interface is UNKNOWN** — it depends entirely on what the robot vendor permits (`CL-SHANA-001`). No schema has been parsed by code, because no code exists |
| Version applicability | SHANA-owned; changes without notice to this register |
| Confidence | `VERIFIED` |
| Related requirements | `FR-SHANA-01`, `FR-SHANA-02`, `FR-SHANA-03` |
| Related decisions | `D-003`, `D-004`, `D-106` |
| Related open questions | `CL-SHANA-001`, `CL-SHANA-015` |
| Next verification step | ⛔ **Do not design against a schema nobody has parsed.** Ask the robot custodian and the vendor what interface is actually permitted — `CL-SHANA-001`. This blocks integration work and must be answered by people, not by reading YAML |

---

## 10. Sources deliberately NOT used

⚠️ **Recorded so that nobody repeats the search.** "Not used" means considered
and rejected, or searched and not located — **not** that the underlying claim is
false.

| Source considered | Why not used | What therefore remains uncertain |
|---|---|---|
| **WCAG 2.2 (W3C)** | Not consulted during the sibling-document pass. Adding it now would retro-fit a citation onto numbers this project chose rather than claims it sourced | ⛔ **All accessibility targets are unsourced proposals** — see `CR-035`. **The highest-value gap to close** |
| **EN 301 549 / RPvF** | A European standard; out of scope for an Indian school. ❓ Whether any equivalent applies is `CL-UX-004` | Whether any accessibility requirement is binding rather than advisory |
| **Synology model-specific hardware specifications** | Product pages describe models in a range, not the model this school owns — and whether a NAS exists at all is unknown | ❓ Whether any actual NAS supports containers — `CL-NAS-001`, `CL-NAS-002`, `CL-NAS-003` |
| **Smartboard or interactive-panel vendor documentation** | ⛔ **No panel model is known.** Citing a vendor would require inventing the hardware | ❓ Everything about the actual displays — resolution, touch digitisation, brightness, viewing angle — `CL-DASH-018`, `CL-WIN-018` |
| **Published CPU inference benchmarks for specific models** | Highly hardware-specific; a benchmark on other hardware supports nothing about `CL-AI-009`, which is unanswered | ❓ Whether inference is on the table at all — `CL-AI-009` |
| **A cloud-VM hosting provider's documentation** | Cloud hosting is a contingency only, never a primary. Writing provider documentation would imply a decision not made | ❓ Whether the school would accept cloud hosting — `CL-SRV-005`, `CL-SRV-008` |
| **School network documentation, prior incident reports, existing systems documentation** | ⛔ **None has been provided, and none may be requested by probing** | Every network fact in this project — all `CL-NET-*` |
| **Any Indian-specific education-technology regulation** | ❓ **NOT FOUND** — searched, not located, and not established whether such a requirement applies to one school's internal display system | Whether any external obligation applies to storing a staff timetable — `CL-SRV-005`, `CL-DATA-015` |
| **Robot vendor SDK or API documentation** | ⛔ No vendor has been identified, and no robot has been contacted. Citing one would imply a partnership that does not exist | ❓ The entire SHANA interface — `CL-SHANA-001`, `CL-SHANA-015` |
| **The SHANA source register, merged in** | ⛔ It is a **separate bibliography** with different scope, access dates and audience. CloudLight and SHANA cite robot-side claims from their own register | Robot-side claims remain traceable only within SHANA's register |
| **`docs/infrastructure/school_network.md`**, `docs/safety/risk_register.md`, `docs/safety/student_privacy.md` | ⛔ **Cited by existing CloudLight documents but NOT PRESENT in the repository tree at 2026-10-11.** Verified by tree inspection | ⚠️ Three internal citations currently resolve to nothing. Their content is known only through the documents that cite them |
| **`docs/architecture/delivery_state_machine.md`** | ⛔ **No such file.** The nearest is `docs/architecture/task_state_machine.md`, which SHANA owns | The deterministic delivery state machine CloudLight's proposal would be submitted to |

---

## 11. Standing note

⚠️ **Every external source in this register describes systems in general — a
vendor product, a published standard, or a research result. None describes this
school, its hardware, its data, its network or its policy.**

**There is no entry in this register that establishes a fact about this school.**
School-specific knowledge exists only as an answer from a person at the school,
recorded in writing. All 576 `CL-*` questions are currently `UNANSWERED`.

⚠️ **And the last word is not ours.** Where these documents and the school's
written policy disagree, **the school is right and these documents are wrong.**

⚠️ **Filling an `UNKNOWN` with a plausible default is the failure mode this
register exists to prevent.**

**Related documents:** [`../sources.md`](../sources.md) ·
[`research_index.md`](research_index.md) · [`research_log.md`](research_log.md) ·
[`claim_evidence_matrix.md`](claim_evidence_matrix.md) ·
[`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md)