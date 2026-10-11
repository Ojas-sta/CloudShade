# CloudLight — Windows Smartboard Startup, Kiosk and Recovery

**Phase 0 · Documentation only · 2026-10-11**

This document sets out how CloudLight could be made to start automatically, survive
reboots and failures, and degrade honestly when it cannot reach the school-hosted
backend on Windows classroom smartboards.

⚠️ **Nothing is built. No smartboard has been touched, imaged, configured,
probed or observed.** Every school-specific fact below is either a question posed
to the school or an assumption explicitly marked for confirmation. Every platform
behaviour claim is sourced to Microsoft or MDN documentation and labelled with the
same evidence scheme used in `docs/architecture/architecture.md`.

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in official vendor documentation, quoted with URL in §11 |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | Vendor documents it; we have not confirmed it on the school's actual boards |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No vendor claim; our engineering judgement, needs an actual test |
| ❓ **UNKNOWN** | No evidence found. Not assumed either way |
| ⛔ **NOT PERMITTED / NOT RECOMMENDED** | Not acceptable, or carries unacceptable risk in this context |

---

## 1. The one fact that must be recorded before anything is designed

> **CloudLight cannot be designed until the Windows edition, build number,
> administrator rights, device ownership and MDM state of every board are
> recorded from the school's own records.**

This is not a formality. The features this document depends on are **edition-gated**,
and Microsoft documents the gating explicitly:

| Feature | Windows Pro | Enterprise / Enterprise LTSC | Education | IoT Enterprise / IoT Enterprise LTSC |
|---|---|---|---|---|
| **Assigned Access** (kiosk) | ✅ | ✅ | ✅ | ✅ |
| **Shell Launcher** (kiosk) | ⛔ Not supported | ✅ | ✅ | ✅ |

✅ **VERIFIED** — Microsoft states: *"Assigned Access is supported on Windows Pro
and Enterprise/Education. Shell Launcher is only supported on Windows Enterprise
and Education editions."*
Source: `learn.microsoft.com/en-us/windows/configuration/kiosk/`

⚠️ **Consequence.** If the boards run **Windows 10 Home**, Assigned Access and Shell
Launcher are both unavailable and the entire managed-kiosk recommendation in §2 collapses
to option (a) — a plain browser someone has to launch. That is not a theoretical branch;
it is a real estate-wide failure mode that must be checked first.

⚠️ **A second constraint, equally hard:** *"You can't configure both Shell Launcher and
Assigned Access on the same system."* ✅ **VERIFIED** — same source. One lockdown
mechanism per board, and the choice is effectively permanent for that device.

### The blocking questions

These are `CL-WIN-001` through `CL-WIN-007` in
`questionnaires/Additional_CloudLight_Questionnaire.md`, and **all seven are P0 with
`Blocks? = YES`**:

| ID | Question | Respondent | Why it blocks |
|---|---|---|---|
| `CL-WIN-001` | Exact Windows edition and build on the display computer | IT | Determines whether Assigned Access / kiosk mode exist at all |
| `CL-WIN-002` | Which feature updates are running, and who decides when to move | IT | Autostart and shell-launch behaviour changes across versions |
| `CL-WIN-003` | Does the display account have **local administrator** rights | IT | Without elevation we cannot install, self-start or repair anything |
| `CL-WIN-004` | Do the PCs belong to school / vendor / donor / lease — and who holds admin credentials | ADM | Software may not be installable on vendor-managed hardware at all |
| `CL-WIN-005` | Is **Intune** or any MDM platform enrolled and authorised | IT | Changes deployment from local install to policy push |
| `CL-WIN-006` | Which lockdown or kiosk tooling is already active | IT | Existing policy may forbid or dictate our approach |
| `CL-WIN-007` | Can we test kiosk mode on **one non-critical** machine | IT | The cheapest way to discover whether the design survives contact with policy |

**Recommended first action:** ask IT for a `winver` screenshot (`CL-WIN-001`) and a
local group-membership screenshot (`CL-WIN-003`). Both are single screenshots and
they determine whether the rest of this document is actionable.

### Windows 10 IoT Enterprise LTSC 2021 — a real option, and a licence trap

Smartboards are **fixed-purpose classroom appliances**, which is precisely the
category Windows IoT Enterprise exists to serve. It is entirely plausible that these
boards already run an IoT edition, and it would be a good fit if they do.

⚠️ **But it must not be assumed, and it must not be newly licensed casually.**

✅ **VERIFIED** — Microsoft: *"Windows IoT Enterprise LTSC is intended for use for
fixed-function specialized devices only and cannot be used as a replacement of
Windows desktop for general purpose computing."*
Source: `learn.microsoft.com/en-us/windows/iot/iot-enterprise/deployment/volume-license`

What that edition buys us, all ✅ **VERIFIED**: **Assigned Access** (supported),
**Shell Launcher** (supported), and **Unified Write Filter** (the `UWF_Volume` /
`UWF_Filter` edition table reads Home No, Pro No, **Enterprise Yes, Education Yes,
IoT Enterprise Yes**).

✅ **VERIFIED** — Windows 10 IoT Enterprise LTSC 2021 is build 19044, serviced
2021-11-16 to **2032-01-13**, Fixed Lifecycle Policy, equivalent to Windows 10 21H2.
Source: `learn.microsoft.com/en-us/windows/iot/iot-enterprise/whats-new/windows-10-iot-enterprise-ltsc-2021`

⛔ **NOT PERMITTED without a licensing answer.** IoT Enterprise LTSC is sold as an
upgrade license through Volume Licensing; volume licences are non-transferrable and
not valid for bare-metal general-purpose PCs. **If these boards currently run
general-purpose Windows and we are considering IoT Enterprise, that is a licensing
decision for the school's IT and asset owner — not a technical preference for us.**
This is exactly `CL-WIN-004`.

⚠️ **A second risk with LTSC families:** they receive no feature updates and no
Microsoft Store app updates by default. A browser-based kiosk on a stale LTSC image is
a security liability. See `CL-WIN-002`.

---

## 2. The four candidate approaches compared

| Option | How it works | Update path | Failure recovery | Admin effort | School-policy risk | Verdict |
|---|---|---|---|---|---|---|
| **(a) Local web app, opened manually** | A teacher or IT member opens a browser and navigates to a URL | Ordinary web deploy — no per-device change | **None.** If nobody opens it, the board is blank | Lowest. No install, no policy | Lowest | 🧪 Baseline. Viable only if manual start is accepted |
| **(b) Managed kiosk browser** 🏆 | Edge in kiosk mode, launched by Assigned Access or Shell Launcher | Web deploy + occasional Edge/policy update | App relaunches automatically on close; idle timeout resets session | Moderate. One-time config per board; needs elevation | **Moderate.** Changes what staff can do on the board | 🟡 **Recommended** |
| **(c) Packaged desktop client** | Electron / Tauri / WebView2 wrapper installed per board | Per-device installer + per-device update | Whatever we build, plus a new class of failure we own | **Highest.** Installer, versioning, support on every device | **Higher.** Installed third-party software on school hardware | ⛔ Not justified on current evidence |
| **(d) Hybrid** | Browser-first; thin wrapper only where a specific requirement fails | Web deploy, with targeted exceptions | Inherits (b), plus complexity at the seam | Moderate, rising | Moderate | 🟡 Fallback if a requirement is proven unmet |

### (a) Browser-accessible local web application only

A teacher or IT member opens a URL on the board. That is the entire mechanism.

**For:** no installation, no elevation, no policy change. Works on **every** Windows
edition including Home. Nothing to break and nothing to restore. Deployable and
testable before any IT approval beyond network access.

**Against:** ⛔ **nothing happens automatically.** Every lesson depends on a human
remembering. If a student closes the browser, the board stays closed. A reboot mid-day
leaves the board showing a desktop until someone intervenes. The display is therefore
not dependable enough to be the school's primary timetable surface.

⚠️ **This is not a design failure. It is a service-level decision.** If the school
answers "a person is present at every lesson and will open it", (a) is completely
acceptable and the cheapest thing to operate. That question is `CL-WIN-012` (recovery)
and `CL-WIN-008` (how it is launched today).

### (b) Managed kiosk browser 🏆 RECOMMENDED

Microsoft Edge in kiosk mode, launched by Windows Assigned Access or Shell Launcher.

#### Edge kiosk lockdown experiences

✅ **VERIFIED** — Edge kiosk mode offers two lockdown experiences, selectable by flag:

```text
msedge.exe --kiosk <url> --edge-kiosk-type=fullscreen --no-first-run
msedge.exe --kiosk <url> --edge-kiosk-type=public-browsing --no-first-run
```

`fullscreen` = **Digital/Interactive Signage** (a specific site, full screen).
`public-browsing` = **Public-Browsing** (a limited multi-tab Edge). And: *"Both
experiences run a Microsoft Edge InPrivate session, which protects user data. Autofill is
disabled when Microsoft Edge runs in InPrivate mode to ensure user data such as form
entries and passwords aren't retained."*
Source: `learn.microsoft.com/en-us/deployedge/microsoft-edge-configure-kiosk-mode`

⚠️ **The InPrivate session is a double-edged property for CloudLight.** It protects
student-device privacy, which is good. It also means browser-persisted state is not a
dependable place to keep a timetable snapshot. See §5 and §7.

#### The feature matrix — and why `fullscreen` is the right choice

✅ **VERIFIED** — Microsoft publishes a per-experience support matrix (Edge 89+ unless
noted; 90+ marked ★). Condensed:

| Feature | Digital/Interactive Signage | Public-Browsing |
|---|---|---|
| InPrivate navigation · Reset on inactivity | Y | Y |
| F11 blocked (full-screen toggle) | Y | Y |
| F12 blocked (DevTools) | Y | Y |
| CTRL+N blocked (new window) | Y | Y |
| CTRL+T blocked (new tab) | Y | **N** |
| Multi-tab support | **N** | Y |
| Read-only address bar *(policy)* | **N** | Y |
| Show home button *(policy)* | **N** | Y |
| Manage favourites *(policy)* | **N** | Y |
| End session button | **N** | Y |
| Internal `edge://` URLs blocked | **N** | Y |
| ★ Restrict launch of other applications | Y | Y |
| ★ UI print settings lockdown | Y | Y |
| Allow URL / Block URL *(policy)* | Y | Y |
| Enable printer *(policy)* | Y | Y |

Same source.

⚠️ **Reading this table correctly matters.** `fullscreen` blocks **navigation affordances**
(no address bar policy, no tabs, no home button, no end-session button) but does **not**
block the internal `edge://` pages. `public-browsing` blocks those internal pages but
adds multi-tab and keeps a read-only address bar.

🧪 **Our judgement:** for a classroom timetable board, `fullscreen` on a single fixed
URL is the correct lockdown experience — it removes every visible control that lets a
student wander. **But we must verify what `edge://` pages are actually reachable in
practice on the board's Edge version.** The matrix marks this `N` for signage, and
whether it behaves identically on this estate is `DOC_NOT_HW_VERIFIED` at best.

⚠️ **The most useful control in the whole document is the policy layer.** ✅ **VERIFIED**
— applying to **both** experiences: `URLAllowlist`, `URLBlocklist`, `SmartScreen`,
`ConfigureKeyboardShortcuts`, `DownloadDirectory`, printer enablement. **Public-browsing
only:** `HomePageLocation`, `ShowHomeButton`, `NewTabPageLocation`, `FavoritesBarEnabled`,
`ManagedSearchEngines`, `UserFeedbackAllowed`, `VerticalTabsAllowed`. Same source.

`URLAllowlist` restricts the kiosk to our dashboard origin and nothing else — a strong,
policy-backed containment control needing no third-party software. ⚠️ **The open question
is `CL-WIN-006`:** applying Edge policies requires local administrator rights
(`CL-WIN-003`) or MDM (`CL-WIN-005`).

#### `--kiosk-idle-timeout-minutes` — and the trap in it

```text
msedge.exe --kiosk <url> --edge-kiosk-type=public-browsing --kiosk-idle-timeout-minutes=5
```

✅ **VERIFIED** — the flag changes the time in minutes from the last user activity before
Edge kiosk mode resets the user's session by closing the browser. Defaults: **full-screen
0 (turned off)**, **public browsing 5 minutes**. Allowed values: `0` turns the timer off;
`1`–`1440` minutes.

⚠️ **The trap — quoted verbatim from Microsoft:**

> *"Note: this flag will not restart Microsoft Edge after it's closed. A separate
> technology, such as Assigned Access or Shell Launch is required to automatically
> restart Edge after the idle timeout."*

This is a very common misreading. The idle timeout **resets the session; it does not
relaunch Edge**. Setting `--kiosk-idle-timeout-minutes` alone produces a kiosk that closes
itself to a black screen and stays there until a human intervenes.

✅ **Design consequence:** `--kiosk-idle-timeout-minutes` and the host mechanism are
**two separate mandatory components**. The idle timeout is only ever useful when
paired with Assigned Access or Shell Launcher.

#### Assigned Access (single-app kiosk)

✅ **VERIFIED** — Microsoft: *"Assigned Access: used to execute a single Universal
Windows Platform (UWP) app or Microsoft Edge in full screen above the lock screen. When
the kiosk account signs in, the kiosk app launches automatically. If the UWP app is
closed, it automatically restarts."* Source: `learn.microsoft.com/en-us/windows/configuration/kiosk/`

Requirements, ✅ **VERIFIED** — *Configure a Single-App Kiosk with Assigned Access*:

- **UAC must be enabled**
- **You must sign in from the console.** *"The kiosk experience isn't supported over a
  remote desktop connection."* ⚠️ Direct operational consequence: **an IT member
  assisting remotely over RDP cannot validate the kiosk.** Validation must be in person.
- Supported editions: Pro, Enterprise/LTSC, Education, IoT Enterprise/LTSC

Configuration routes, ✅ **VERIFIED** — `Set-AssignedAccess` PowerShell cmdlet, Settings
UI, or the Assigned Access CSP XML applied via Intune/MDM, a provisioning package, or the
MDM Bridge WMI Provider. ⚠️ **The WMI Bridge route requires the client to run as SYSTEM** —
✅ **VERIFIED**: *"For all device settings, the WMI Bridge client must be executed as
SYSTEM (LocalSystem) account."*

⚠️ **A documented advantage over Shell Launcher:** *"a set of policy settings are
automatically applied to the device to optimize the lock-down experience. Shell Launcher
doesn't have any default lockdown policies."* ✅ **VERIFIED** — same kiosk overview page.

#### Assigned Access breakout and relaunch behaviour

✅ **VERIFIED** — verbatim:

> *"By default, to exit the kiosk experience, press `Ctrl + Alt + Del`. The kiosk app
> exits automatically. If you sign in again as the Assigned Access account, or wait for
> the sign in screen time-out, the kiosk app relaunches. The default time-out is
> 30 seconds, but you can change the time-out with the registry key:
> `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\LogonUI`"*

⚠️ **The breakout sequence is configurable** — ✅ **VERIFIED**: *"The Breakout Sequence of
`Ctrl + Alt + Del` is the default, but this sequence can be configured to be a different
sequence of keys… modifiers + keys."* An example given is `CTRL + ALT + A`.

⚠️ **Design requirement:** the breakout sequence is a **support procedure**, not a user
convenience. It must be chosen, written down, agreed with IT, and held by someone who can
type it at the front of a classroom. See §8.

⚠️ **A precise trap worth recording now.** ✅ **VERIFIED**: *"Note: `IdleTimeOut` doesn't
apply to the Microsoft Edge kiosk mode."* So there are **two separate resume timers**, and
they are not the same mechanism:

| Timer | Governs | Applies to Edge kiosk mode? |
|---|---|---|
| `LogonUI` `IdleTimeOut` (default 30 s) | How long until the kiosk app relaunches after breakout | ❌ **No** — explicitly excluded |
| `--kiosk-idle-timeout-minutes` | How long idle before Edge closes itself | ✅ Yes |

⚠️ **Never conflate these two in a configuration procedure.** Doing so produces a kiosk
that exits on breakout and does not come back.

#### Shell Launcher

✅ **VERIFIED** — Microsoft: *"Shell Launcher: used to configure a device to execute a
Windows desktop application as the user interface. The application that you specify
replaces the default Windows shell (`Explorer.exe`) that usually runs when a user signs
in. This type of single-app kiosk doesn't run above the lock screen."*

Shell Launcher is **an optional component, not enabled by default** ✅ **VERIFIED**;
configuring it via the Assigned Access CSP enables it automatically. OMA-URI:
`./Vendor/MSFT/AssignedAccess/ShellLauncher`. The `Shell` element ✅ **VERIFIED**
carries the executable path (may use `%variableName%` variables and parameters),
`V2:AppType` (`Desktop` | `UWP`) and `V2:AllAppsFullScreen`.

⚠️ **Shell Launcher's distinguishing weakness:** it does not run above the lock screen,
so a lock or sleep event leaves the classroom without a display. See §4.

The `ReturnCodeActions` mechanism ✅ **VERIFIED** — *"You can specify at most four
custom actions mapping to four exit codes, and one default action for all other exit
codes. When an app exits, and if the exit code isn't found in the custom action mapping,
or there's no default action defined, **nothing happens**. For this reason, you should
at least define `DefaultAction`."* Supported actions: **`RestartShell`**,
**`RestartDevice`**, **`ShutdownDevice`**, `DoNothing`.

⚠️ **The recovery power here is real and specific.** `RestartShell` on a normal exit
relaunches the kiosk; `RestartDevice` on a crash exit reboots the machine. That is
genuinely stronger recovery than Assigned Access alone, and it is the main technical
argument for Shell Launcher over Assigned Access.

⚠️ With `AutoLogonAccount`, Shell Launcher *"creates and manages a user account to
automatically sign in after a device restarts. The account is a local standard user
named `Kiosk`."* ✅ **VERIFIED**. A shared, named, standard-privilege account on every
classroom board has implications for `CL-SEC-001` and `CL-WIN-022`.

### (c) Packaged desktop client — Electron, Tauri or WebView2

**This is only justified if one of the following is genuinely required and cannot be
achieved in-browser:**

| Requirement | Can a browser do it? | Status |
|---|---|---|
| Offline timetable cache | Yes — service worker + storage, subject to §7 | 🧪 Must be tested on this Edge version |
| Direct OS integration (serial, USB HID, local files) | **No** | ❓ No such requirement has been stated |
| Kiosk lockdown the browser cannot achieve | Mostly no — Edge kiosk + Assigned Access is purpose-built | ✅ VERIFIED capability exists |
| Deeper OS-level write protection | UWF can do this **without** an app | ✅ VERIFIED |

⚠️ **The honest position.** Based on everything currently known about CloudLight's
requirements — a timetable, a substitution sheet, a countdown, a resource browser and an
assistant — **a browser can cover them all.** No requirement has been raised that needs
native code.

And the wrapper adds real, permanent costs: an **installer** on every board (meaning
`CL-WIN-003` elevation and `CL-WIN-004` permission); a **per-device update problem** — a
web app updates once on the server, a desktop app once *per board, per deployment*; a
**new failure mode we would own** (updater failure, partial upgrade, version skew between
boards showing different data); and a larger **support surface** for school IT, who did
not ask for it.

⛔ **NOT RECOMMENDED on current evidence.** If ever needed, it should be a *thin* wrapper
over the same web application, not a parallel codebase.

### (d) Hybrid

Browser-first as the default for every board; a thin wrapper only on boards where a
named, documented requirement cannot be met in-browser.

🧪 **This is the correct answer to keep available, not the answer to start with.** The
discipline is that any wrapper must justify itself against a **written requirement
ID**, never against a preference.

### Recommendation

> **Adopt (b), Managed kiosk browser, via Assigned Access with Edge kiosk mode in
> `fullscreen` — conditional on `CL-WIN-001`, `CL-WIN-003`, `CL-WIN-006` and
> `CL-WIN-007` being answered favourably.**

Why Assigned Access rather than Shell Launcher as the *default*:

| Factor | Assigned Access | Shell Launcher |
|---|---|---|
| Auto-applied lockdown policies | ✅ Yes | ❌ None by default |
| Runs above the lock screen | ✅ Yes | ❌ No |
| Supported on Windows Pro | ✅ Yes | ⛔ No |
| Relaunch on app close | ✅ Documented | ✅ Via `RestartShell` |
| `RestartDevice` on crash | ❌ Not available | ✅ Available |
| Edge kiosk `--kiosk-idle-timeout-minutes` | ✅ Works | ✅ Works |

Assigned Access wins on three of the four decisive criteria. Shell Launcher's single real
advantage — `RestartDevice` on crash — can be covered by Task Scheduler (§3) without
giving up the lockdown benefits.

#### Reversal condition

**Reverse to (a) — manual browser launch — if any of the following is true:**

1. `CL-WIN-001` reveals **Windows Home** on any board in the pilot scope (Assigned Access
   and Shell Launcher are both unavailable ✅ **VERIFIED**)
2. `CL-WIN-006` reveals existing lockdown policy that **forbids** kiosk configuration
3. `CL-WIN-004` reveals the boards are **leased or vendor-managed** and we may not install
   or reconfigure them
4. `CL-WIN-007` is refused — the school will not let us test on one non-critical board
5. IT requires a change-control window we cannot obtain

**Reverse to (d) Hybrid** if a specific requirement emerges that the browser cannot meet,
and only with a written requirement ID attached.

---

## 3. Autostart mechanisms on Windows — what exists

| # | Mechanism | What it does | Needs elevation? | Survives reboot? | Failure mode |
|---|---|---|---|---|---|
| 1 | Registry `Run` | Executes a command at user logon | ✗ (user-writable) | ✅ | Silent. No restart, no supervision, no logging. **Worst visibility** |
| 2 | Registry `RunOnce` | Executes once after installation | ✗ | ✅ | Runs once, then deletes itself. **Disappears** — an anti-pattern for a kiosk |
| 3 | Startup folder | Executes a shortcut at logon | ✗ | ✅ | Same as `Run`. Easily deleted by a user |
| 4 | **Task Scheduler** (logon trigger) | Runs a program on user logon, or at boot, or on idle | ✅ To create | ✅ | **Silent if misconfigured.** Lowest interactive priority — may not run |
| 5 | **Task Scheduler** (restart on failure) | Retries a failed task a configured number of times | ✅ To create | ✅ | Retries then gives up. **Silent give-up** |
| 6 | Windows Service | Background process managed by the SCM, starts at boot before logon | ✅ Yes | ✅ | ⛔ **Cannot display UI.** See below |
| 7 | **Shell Launcher** | Replaces `explorer.exe` — the app *is* the desktop shell | ✅ Yes | ✅ | If the shell app exits and no `DefaultAction` is set, **nothing happens — blank desktop** |
| 8 | **Assigned Access** | Kiosk app launches when the kiosk account signs in | ✅ Yes | ✅ | Does not run if the account does not sign in. Lock screen issue (§4) |
| 9 | Group Policy | Pushes configuration, including Edge kiosk policies | ✅ Yes | ✅ | Silent policy application failure |
| 10 | MDM / Intune kiosk profile | Pushes kiosk configuration from the tenant | ✅ Tenant admin | ✅ | Depends on device check-in and internet reachability |
| 11 | Unified Write Filter (UWF) | Write-protection: redirects writes to a virtual overlay | ✅ Admin | ✅ (overlay cleared) | ⛔ **Overlay exhaustion.** See below |

### ⛔ A Windows Service cannot be the kiosk launcher — this is not a preference

✅ **VERIFIED**, stated directly by Microsoft:

> *"Services cannot directly interact with a user as of Windows Vista."*

> *"All services run in Terminal Services session 0. Therefore, if an interactive service
> displays a user interface, it is visible only to the user who connected to session 0.
> Because there is no way to guarantee that the interactive user is connected to session
> 0, do not configure a service to run as an interactive service under Terminal Services
> or on a system that supports fast user switching."*

Microsoft also documents that `NoInteractiveServices` under
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Windows` **defaults to 1**, meaning
no service is allowed to run interactively.

**Therefore: a Windows Service has no place in the CloudLight display path.** A service
could legitimately run a *server-side* helper — health reporting, a status agent — but it
cannot put pixels on a classroom panel, and any design that tries fails silently and
expensively. ⚠️ The only workaround Microsoft documents — a hidden GUI app launched via
`CreateProcessAsUser` and talking over IPC — is precisely the complexity a browser kiosk
avoids entirely. ⛔ **Not recommended here.**

### Task Scheduler — the one mechanism worth understanding properly

✅ **VERIFIED** — Task Scheduler supports triggers including: *"When a specific system
event occurs… When the computer enters an idle state… When the task is registered… When
the system is booted. **When a user logs on.** When a Terminal Server session changes
state."* Source: `learn.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-start-page`

✅ **VERIFIED** — restart-on-failure is first-class: `ITaskSettings::put_RestartCount` —
*"Gets or sets the number of times that the Task Scheduler will attempt to restart the
task."* Paired with a restart interval.

Two properties make Task Scheduler attractive, one makes it dangerous:

- ✅ At-boot and at-logon triggers cover both the auto-logon and manual sign-in cases
- ✅ Restart-on-failure gives supervised retry without a human
- ⛔ **Tasks run at low interactive priority by default and may be starved.** A task that
  never runs looks identical to one that was never created. **This is why `CL-WIN-009`
  asks whether a scheduled task actually *works*, and why the evidence required is
  *"Screenshot of a successfully running scheduled task"* — not a screenshot of the task
  definition.**

✅ **VERIFIED** — `schtasks` requires **Administrators group** membership to schedule,
view or change all tasks on the local computer. This is `CL-WIN-003` again, and it is why
several rows above say "needs elevation".

### Shell Launcher — the configuration trap worth stating up front

⚠️ Microsoft is explicit ✅ **VERIFIED**: *"if the exit code isn't found in the custom action
mapping, or there's no default action defined, **nothing happens**. For this reason, you
should at least define `DefaultAction`."*

Because Shell Launcher **replaces `explorer.exe`**, "nothing happens" means **a
desktop-less, taskbar-less screen**. If Edge is never launched, the classroom screen is
simply black, with no obvious way back.

⛔ **Consequence:** a CloudLight Shell Launcher configuration without an explicit
`DefaultAction` is a **board-bricking risk**, not a minor omission. Any such configuration
must be treated as a change requiring a tested restore procedure.

### Unified Write Filter — powerful, and easy to misuse

✅ **VERIFIED** — UWF *"protects the contents of a volume by intercepting write attempts to
a protected volume and redirects those write attempts to a virtual overlay."* Source:
`learn.microsoft.com/en-us/windows/iot/iot-enterprise/customize/uwfoverlay`

⚠️ **Four documented behaviours that matter for a classroom board:**

| Behaviour | Consequence for CloudLight |
|---|---|
| **RAM overlay is the default**, cleared after reboot | The offline snapshot **cannot** live on a protected volume unless explicitly configured — an offline timetable that vanishes at reboot is worse than useless |
| **Overlay default size 1024 MB** | Fills quickly with browser and Edge writes |
| **Overlay exhaustion**: *"any write attempts will fail… your device may become unresponsive and sluggish, and you may need to restart your device"* | ⛔ A slow, unresponsive board in a lesson |
| **Overlay-full shutdown can take *"up to an hour or longer"*** | ⛔ Discovered by a teacher at 3pm, not by IT in a test window |

✅ **VERIFIED** — Windows 10 19H1 and later *"will automatically restart if the maximum
size of the overlay is exceeded."* That is a **forced reboot** — precisely the mid-lesson
restart the school is trying to avoid (`CL-WIN-013`).

✅ **VERIFIED** — enabling or configuring UWF **requires an administrator account**.

⚠️ **Our position: UWF is a candidate for phase 2, not phase 1.** It is the right tool
for a genuinely fixed-purpose board, but it introduces a class of failure (overlay
exhaustion) that a classroom cannot tolerate during a lesson, and it constrains where
an offline snapshot may live. **It must not be enabled before `CL-WIN-007` has been
answered on a sacrificial machine.**

---

## 4. Recovery matrix

| # | Failure scenario | What happens | What recovers it | Residual risk |
|---|---|---|---|---|
| 1 | **Browser crashes** | Edge process exits | Assigned Access: app relaunches if closed ✅. Shell Launcher: `ReturnCodeActions` → `RestartShell` ✅. Task Scheduler restart-on-failure as backup | 🟡 With Assigned Access the relaunch is documented for **UWP apps**. Edge relaunch in the same account should be validated on our hardware (`CL-WIN-007`) |
| 2 | **Browser closed by user/student** | Same as a crash — an unattended student taps the X | Identical to #1 | ⚠️ Under `fullscreen`, there is no visible close control; `CTRL+N` and `F11` are blocked ✅. Residual risk is external input (task manager, Alt-Tab) |
| 3 | **Windows restart (planned or power cycle)** | Full boot, then autostart must re-engage | Assigned Access autologon / Shell Launcher autologon ✅. Task Scheduler at-boot trigger as a third net | ⚠️ Depends on `CL-WIN-008`. If the board requires a manual sign-in, nothing starts |
| 4 | **Windows Update restart mid-lesson** | ⚠️ **Screen goes dark during a lesson.** This is the highest-impact everyday failure | Update deferral / active hours (an IT policy, `CL-WIN-013`); UWF restart-to-protect-volume is **not** a mitigation here | ⚠️ **We cannot fix this from software we control.** It is a school IT policy decision. Record it as accepted risk |
| 5 | **Power cut** | Abrupt loss; risk of filesystem corruption on the host PC | Reboot on power return. UWF would reduce write risk | ⚠️ If the *server* is on the same circuit, both go down (`CL-NET-019`, `CL-SRV-003`) |
| 6 | **Network unavailable at boot** | ⚠️ **The PC boots faster than the network comes up.** Edge launches, cannot resolve or connect, and shows a **browser error page** | ⚠️ **The correct fix is an application-level offline state, not a browser flag.** See the callout below and §5 | 🟡 Residuals: the first paint may briefly show a browser-native error before our JS runs |
| 7 | **Server unavailable** | Application loads, fetch fails, must show cached snapshot with staleness | Offline snapshot (§5), service worker | 🟡 Shows stale data — honest, but see the §5 warning |
| 8 | **Backend healthy but slow** | Long hangs; teacher thinks it is broken | Client-side request timeout → offline tier | ⚠️ **Timeouts are not verified against this network.** `CL-NET-018` asks what response time teachers need |
| 9 | **Certificate expiry** | ⚠️ **Hard fail.** Browser interstitial blocks the page entirely — worse than an outage because there is no degraded path | Not recoverable client-side. Needs server-side alerting on cert expiry + a renewal process | 🔴 **The most under-planned failure in this list.** Must be an explicit ops item, not an assumption |
| 10 | **Kiosk account password change / expiry** | Autologon fails; board stops at the sign-in screen | IT password reset | ⚠️ **Unattended board, dark screen, until IT arrives.** A long-password policy that expires credentials is actively harmful for kiosks |
| 11 | **Disk full** | Writes fail; Edge or the OS may fail to start | Disk cleanup; UWF would *cause* this (§3) | ⚠️ Logging and crash-dump growth are the usual culprits |
| 12 | **Screen locked or asleep** | Display off; nothing visible | Power and lock policy (`CL-WIN-014`) | ⚠️ **Assigned Access runs above the lock screen ✅; Shell Launcher does not ✅.** This is a decisive difference and must match `CL-WIN-014` |
| 13 | **Display scaling / resolution change** | Layout reflows; touch targets may not match the new panel | Density-independent rendering (§6) | ⚠️ Touch misalignment after a scale change is a known panel-calibration issue, not a browser bug |
| 14 | **Board host hardware failure** | Nothing. No software recovery | Manual replacement | ⚠️ Depends on `CL-WIN-018` — if the boards are thin clients, failure may be at the panel or the host |

### ⚠️ The known real-world problem: the PC boots faster than the network

> **This is the single most common kiosk failure in real deployments, and it is
> predictable enough to design against.** The sequence, observed repeatedly in the field:

```
0s     Power on
~10-40s Windows begins booting
~45-90s Explorer/shell starts, autostart fires, Edge launches in kiosk mode
~50s   Edge tries to resolve cloudlight.school.local  ──► DNS NOT YET ANSWERING
~51s   Browser renders its OWN network error page
...    ~30-180s later the network is genuinely up
       Edge is still sitting on a browser error page, and nobody refreshes it
```

**The critical observation:** the browser error page is not our application. It is a
browser-native screen showing a DNS or connection failure. From the back of a classroom
it looks exactly like "CloudLight is broken" — and no amount of our application code will
ever run, because our code is not what is on screen.

⛔ **What does NOT fix this:**

| Non-fix | Why it fails |
|---|---|
| Increasing Edge's network timeout | Browser flags do not govern our own `fetch()` retry behaviour |
| Adding `--kiosk-idle-timeout-minutes` | Closes Edge; does not make it wait for the network |
| A longer Task Scheduler delay before launch | **Merely hides the symptom by guessing a delay.** Tuned to one board on one morning, wrong the next time DHCP or a switch takes longer. A magic number, not a design |
| Retrying DNS resolution | Still loads a browser error page if the *document* fetch fails |

🟡 **What DOES fix it — the application-level offline state:**

1. The CloudLight application **must render successfully even when every request fails.**
   The shell, layout, fonts and offline tier logic are all bundled and must not depend
   on network.
2. The **document request itself** must be served from a local cache (service worker or
   equivalent), so the browser never navigates to its error page at all.
3. Once running, the client must **retry with backoff** until the backend appears, then
   transition silently to live data.

⚠️ **This is an architectural requirement on the application, not a deployment setting.**
It means the offline shell must be designed and built from the first commit —
retrofitting it later is not realistic, because it changes how the whole frontend is
structured.

🧪 **And it is not yet verified.** Whether Edge kiosk mode on this estate will serve an
application shell from cache without ever touching the network error page is **exactly**
what `CL-WIN-007` — testing on one non-critical machine — is for.

---

## 5. Network unavailability and stale-data display

### The design rule

> **The display must remain useful and honest when it cannot reach the backend.
> It must never show a confidently wrong timetable. It must always show when the
> data was last synchronised.**

⚠️ **A confidently wrong timetable is worse than no timetable.** A board showing a
substitution that was cancelled this morning is actively misinformation displayed to
students. A board showing "I cannot reach the server, last updated 09:14" is merely
degraded — and a teacher can act on that.

This is a school-policy question, not an engineering preference. It is `CL-SEC-018`:
*"How old may timetable, substitution and robot-status data be on a classroom screen
before this school considers it **misleading rather than merely late**?"* — P1,
respondent ADM. ⚠️ **We must not invent this number.** It is a school decision and the
questionnaire correctly leaves it blank.

### The three offline tiers

| Tier | Condition | Display | Must show | Rationale |
|---|---|---|---|---|
| **(i) Cached, within tolerance** | Snapshot age ≤ staleness threshold | Normal display, no alarming styling | `Last synced: HH:MM` | Data is acceptable by school policy. No need to alarm the room |
| **(ii) Cached, beyond tolerance** | Snapshot age > staleness threshold | ⚠️ **Visibly marked stale** | `Last synced: HH:MM (STALE — n minutes old)` | Honest, but the school has said this is misleading. It must be labelled, not styled away |
| **(iii) No snapshot at all** | No usable cached data ever stored | ⚛️ **Explicit "no timetable data" state** | Why, and what to do | ⛔ **Never a blank screen. Never a spinner that never ends** |

⚠️ **Tier (iii) is the one teams skip and the one that matters most in a crisis.**
"Blank screen" is indistinguishable from "broken board". "Cannot reach the timetable
server — last attempt HH:MM — please tell IT" is a diagnosable state that any member
of staff can act on.

🧪 **Tier (ii) is risky and must be treated as a product risk, not a solved problem:**
it is *honest*, but honesty is not safety. A stale substitution displayed for 40 minutes
could send students to a room where the teacher is not, or a teacher to a class that is
not there. And the visible warning is read by adults, not students — **a student
glancing at a stale board sees a timetable with a small grey timestamp nobody mentioned.**

⚠️ **Therefore the staleness threshold is a safety parameter with a real cost, and it
must be decided by the school — `CL-SEC-018` — not defaulted by us.**

### The threshold must be configurable, and visible

🧪 **Recommendation:** the threshold is a server-side configuration value pushed to each
board, not a constant compiled into the frontend. The school sets it (`CL-SEC-018`);🧪
different **data types** should not share one threshold — a period ending in 20 minutes
has a different urgency from a long-term notice board; and if the answer changes it must
be a configuration change, not a release.

⚠️ **Whatever value is chosen, it must be visible on the screen itself.** A hidden
threshold produces a system where nobody can tell whether the board is fresh.

### Related questions this raises

| Question | ID | Note |
|---|---|---|
| What must the board do when the network is down at all? | `CL-WIN-011` | *"show a cached timetable, or display nothing at all?"* — the school may legitimately choose "nothing" |
| Would teachers accept an occasionally out-of-date board? | `CL-GEN-011` | Sets the whole caching target |
| What response time do teachers need at a period change? | `CL-NET-018` | Sets the timeout before tier (ii)/(iii) |
| Does an unauthenticated read-only home state apply when no teacher is signed in? | `CL-SEC-001` | Determines what is student-visible at all |

---

## 6. Display scaling, resolution and touchscreen

### Four different things that are constantly confused

| Concept | What it is | Who controls it |
|---|---|---|
| **Panel native resolution** | Physical pixels. 3840×2160 on a 4K panel | The panel |
| **OS display scaling** | How many device pixels map to one logical point. 100%, 125%, 150%, 200% | Windows Display settings; per-monitor |
| **Browser zoom** | Page magnification, independent of OS scaling | `Ctrl`+`+`, or per-site setting |
| **`devicePixelRatio`** | The ratio the page reads via `window.devicePixelRatio` | Derived from OS scaling × browser zoom |

✅ **VERIFIED** — `window.devicePixelRatio` is a standard Web API exposing the ratio of
physical pixels to CSS pixels. MDN documents it under `Window.devicePixelRatio`.

⚠️ **Consequence that bites classroom dashboards:** on a 4K panel at 200% OS scaling,
a naive `px` layout can appear at half its intended size if the developer confused
device pixels with CSS pixels. This is the most common large-display bug, not a
hypothetical.

### Recommendations

1. **Render density-independent.** 🟡 Specify and type sizes in **CSS pixels and relative
   units**, never device pixels. Read `window.devicePixelRatio` only to choose *asset*
   resolution, never to size layout.
2. **Set a minimum legible type size for the intended viewing distance.** ⚠️ The intended
   distance is **unknown** — it is `CL-UX-001`: *"What is the farthest distance from which
   students are expected to read text on a classroom display?"* (P0, respondent TCH). 🧪
   A classroom timetable has no business using body-copy sizing, but **we must not guess
   the number** — the school must state it.
3. **Test on the worst-case panel in the estate.** ⚠️ Not the newest, not the flagship —
   the **worst**: smallest physical size, lowest resolution, most unusual aspect ratio. If
   it works there, it works everywhere.
4. **Never rely on hover.** 🟡 A touch panel has no hover state. Anything reachable only
   on `:hover` is unreachable on a touch board. Every interaction must work on tap.
5. **Assume no pointer.** 🟡 No right-click menu, no middle-click, no drag-to-select, no
   context menu. Long-press is not discoverable for a teacher starting a lesson.
6. **Design for aspect-ratio variation, not one fixed layout.** ⚠️ Ultrawide classroom
   panels, 16:9 boards and 4K boards in portrait are all plausible. A fixed pixel layout
   will crop on something.

### The panel inventory is unknown

⚠️ **Not one of these is known** — stated plainly because it means **no layout decision
can be finalised**.

| ID | Question | Respondent |
|---|---|---|
| `CL-WIN-015` | Panel sizes, resolutions and scaling settings | IT |
| `CL-WIN-018` | Dedicated PC, thin client, or embedded slot | IT |
| `CL-DASH-018` | Do displays vary in size, resolution, orientation or aspect ratio | IT |
| `CL-UX-001` | Farthest reading distance | TCH |

`CL-WIN-018` deserves particular attention: if the boards are **thin clients or
embedded-slot panels**, the failure domain and the update path are different from a
dedicated PC, and the "install something" assumption weakens further. ❓ **UNKNOWN.**

⚠️ **Related, and easily forgotten:** touch digitisation itself is unknown
(`CL-WIN-016` — native touch, vendor driver, or USB overlay), as is who recalibrates it
and how long it takes (`CL-WIN-017`). A board whose touch calibration drifts is a support
problem that appears as "the CloudLight buttons are in the wrong place", and it is a
facilities task, not a software task.

---

## 7. Browser security restrictions that matter

⚠️ **This section assumes TLS will be added.** The design is local-first over the school
LAN, and the questionnaire treats HTTPS as the intended shape (`CL-NET-014` covers
permitted ports and protocols). Several of these constraints only bite once TLS exists,
which means **they will be discovered late unless designed for now**.

### Content Security Policy

🟡 **Recommendation:** ship a strict CSP from the first release — no `unsafe-inline`, no
`unsafe-eval`, explicit `default-src 'self'`, no wildcard origins. A strict CSP on a
local-only application costs nothing and closes a large class of injection risk on an
unattended device that students can touch.

### Mixed content — the constraint that appears the day TLS is switched on

✅ **VERIFIED** — MDN documents that browsers **auto-upgrade** *upgradable* mixed
content (images, audio, video, `<source>`) from HTTP to HTTPS, and **block** all
*blockable* mixed content — which includes `<script>`, stylesheets, `<iframe>`,
`fetch()`, `XMLHttpRequest`, and **CSS `url()` values including `@font-face`**.

And critically: *"Mixed content requests that would otherwise be upgraded are blocked
if the URL's host is an IP address rather than a domain name."* Source: MDN.

Two practical consequences:

1. ⚠️ **Never serve the dashboard over `http://<ip-address>`.** If the board is reached
   by IP rather than hostname, HTTPS cannot be enabled cleanly at all. This makes
   `CL-NET-012`/`CL-NET-013` (name resolution) a **TLS prerequisite**, not a convenience.
2. ⚠️ **Web fonts are blockable mixed content.** A font loaded by absolute `http://` URL
   will be blocked silently under HTTPS. Fonts must be same-origin or HTTPS.

⛔ **NOT RECOMMENDED: running HTTPS on a bare IP address.** Use a local hostname.

### Third-party cookies, autoplay and notifications

🟡 Modern Chromium-family browsers — Edge included — restrict or phase out third-party
cookies as a privacy default. 🧪 **CloudLight's own session cookie is first-party** — set
by, and only sent to, the origin the kiosk points at — so this should not affect it.

⚠️ **But it will affect anything the dashboard embeds:** a federated sign-in provider, an
analytics tag, a third-party font, an embedded NAS browser. ⚠️ **None of these should be
on a classroom board anyway.** The correct response is architectural: **keep the display
surface entirely first-party.** Every external dependency is one more thing that can
break, leak, or be blocked by policy.

🟡 **Autoplay** of media is restricted without user interaction. Any audio/video element
— a welcome announcement, an embedded resource preview — may be silently blocked. ⚠️ **A
student-facing board that silently fails to play an announcement is a real failure mode.**
If audio is ever used, it must have a visible fallback state.

🟡 **Notifications** require explicit permission and are blocked by default. ⛔ **Do not
build classroom-critical behaviour on web notifications.** A countdown that only alerts
via notification is a countdown that will not alert.

### Service workers and local storage in kiosk mode — ⚠️ the critical unknown

⚠️ **This is the highest-risk technical unknown in the document.**

✅ **VERIFIED** — both Edge kiosk lockdown experiences **run an InPrivate session**, and
**autofill is disabled**.

⚠️ **An InPrivate session is by design a session in which user data is not retained.**
Whether an InPrivate-context service worker can install a durable offline cache, and
whether `localStorage` persists across kiosk restarts, is **version-dependent behaviour
on this specific Edge build that we have not tested and cannot test today.**

| Requirement | Depends on | Status |
|---|---|---|
| Offline timetable snapshot survives a browser restart | Durable storage in InPrivate context | 🧪 **UNVERIFIED — must be tested** |
| Application shell served from cache when the network is down | Service worker in kiosk context | 🧪 **UNVERIFIED — must be tested** |
| Silent transition from offline to live once network returns | Both of the above | 🧪 **UNVERIFIED** |

⚠️ **If neither works, §4's boot-race fix is not implementable as designed**, and the
architecture must change — likely to option (c), a packaged client with real disk storage.

⛔ **This is the strongest technical argument that could ever emerge for a desktop wrapper.**
It should be evaluated on evidence, not assumed.

### ⛔ Session tokens on an unattended device

⚠️ **The panel in a classroom may be touched by any student.** That is the baseline risk
of the whole product, and it is `CL-SEC-001` — a P0 question about whether the board is
student-usable or must fall back to an unauthenticated read-only state.

**Required cookie configuration** (✅ **VERIFIED** from MDN `Set-Cookie` semantics):

| Attribute | Required | Why |
|---|---|---|
| **`HttpOnly`** | ✅ **Mandatory** | *"Forbids JavaScript from accessing the cookie… This mitigates attacks against cross-site scripting (XSS)."* |
| **`SameSite=Strict`** | ✅ **Mandatory** | Blocks cross-site request forgery; the kiosk makes no cross-site requests |
| **`Secure`** | ✅ Mandatory once TLS exists | Cookie sent only over HTTPS |
| **Short `Max-Age`** | ✅ Mandatory | Bounds the damage of a stolen token |
| **Bound to the server session** | ✅ Mandatory | Server-side revocation must be possible |
| `__Host-` prefix | 🟡 Strongly consider | ✅ **VERIFIED**: requires `Secure`, `Path=/`, and **no `Domain`** — *"as close as possible to treating the origin as a security boundary"* |

⚠️ **Note the limitation of `HttpOnly` honestly.** ✅ **VERIFIED** MDN: a cookie
created with `HttpOnly` *"will still be sent with JavaScript-initiated requests."* It
stops **reading** the cookie, not **using** it. It mitigates XSS; it does not
eliminate it. A strict CSP is the other half of that defence.

⛔ **NEVER store a session token in `localStorage` or `sessionStorage`.** Those are
readable by any JavaScript running on the origin. On a student-touchable panel with
a known origin, that is a credential handed over by design. **Use `HttpOnly` cookies
exclusively.**

---

## 8. Device lockdown and preventing disruption of other smartboard functions

⚠️ **This is the risk most likely to end the project — not technically, politically.**
A smartboard is shared teaching equipment. If CloudLight's kiosk configuration breaks
the whiteboard, the video conferencing, or the document camera, the school will
remember that first and permanently.

### The three lockdown options compared

| Option | Locks down | Supported editions | Runs above lock screen | Auto-applied policies | Verdict |
|---|---|---|---|---|---|
| **Assigned Access** (single-app) | Everything except one app | Pro, Ent, Ent LTSC, Edu, IoT Ent | ✅ Yes | ✅ Yes | 🏆 **Recommended default** |
| **Assigned Access** (multi-app / restricted) | Everything except a defined app list | Same | ✅ Yes | ✅ Yes | 🧪 Consider — allows whiteboard + CloudLight together |
| **Shell Launcher** | Replaces the shell entirely | **Ent and Edu only** | ❌ No | ❌ None | Secondary; use for its `RestartDevice` recovery |
| **Group Policy / MDM** | Configuration, not containment | Ent, Ent LTSC, Edu, IoT Ent | N/A | N/A | 🟡 Complementary — `URLAllowlist` is powerful here |

⚠️ **A thought worth putting to the school early:** the *multi-app* Assigned Access
configuration allows a **restricted user experience** — a customised Start menu containing
an allow-list of applications, with policies and AppLocker rules enforced. ✅ **VERIFIED**
— *"Different policy settings are applied to the device… AppLocker rules are enforced…
users can access a familiar Windows desktop, while limiting their access."*

That could plausibly allow **CloudLight + the vendor whiteboard app + the video conferencing
app** in one lockdown. 🧪 **Plausible, requires testing, and not yet raised with the
school.** It may be the answer that makes lockdown acceptable rather than disruptive.

### AppLocker, WDAC and USB/keyboard filters

🟡 These exist and can restrict what runs on the board. ✅ **VERIFIED** that AppLocker
rules are part of the Assigned Access restricted user experience.

⚠️ **But:** they **require elevation**, apply policy **machine-wide**, and need a **reboot
or reconfiguration window**. Combined with `CL-WIN-004` (do we even have permission to
change these machines?), this is firmly a later-phase conversation.
⛔ **Do not attempt AppLocker or WDAC before a restore procedure is written and tested.**

### The four rules for not breaking the school's equipment

> **Rule 1 — Pilot on ONE non-critical board first.** Never a board in a regularly
> timetabled classroom. `CL-WIN-007` asks exactly this.

> **Rule 2 — Write and rehearse the restore procedure BEFORE writing any policy.**
> ⛔ **No policy change is applied before its inverse is documented and someone has
> walked through it.** A lockdown without a known escape is a device that may not come back.

> **Rule 3 — Keep a documented manual escape, recorded with IT.** The breakout sequence
> (`Ctrl+Alt+Del` by default ✅ **VERIFIED**, configurable to another modifier+key
> combination ✅ **VERIFIED**) must be written down, agreed with IT, and held by named
> people who can type it at the front of a classroom.

> **Rule 4 — Do not touch the whiteboard, conferencing or document camera configuration.**
> CloudLight's configuration must be additive. If our kiosk makes the vendor's whiteboard
> app unreachable, that is a failed pilot regardless of how well the dashboard works.

⚠️ **One more risk specific to shell replacement:** with Shell Launcher, `explorer.exe`
**is gone**. Any school process or vendor tool that depends on the Windows shell — a
vendor tray agent, a screenshot utility, some whiteboard annotation tools — may not
behave. 🧪 **Requires testing on the sacrificial board.**

---

## 9. What is NOT decided

Every item below is open. None has an answer, and **none may be assumed** for the
purpose of design.

| ID | Open question | Blocks | Who can answer |
|---|---|---|---|
| `CL-WIN-001` | Exact Windows edition and build | **Everything.** Determines if Assigned Access / kiosk exist at all | IT |
| `CL-WIN-002` | Feature updates in use; who decides on upgrades | Autostart and shell-launch stability | IT |
| `CL-WIN-003` | Local administrator rights on display accounts | Install, self-start, repair | IT |
| `CL-WIN-004` | Device ownership and credential custody | Whether we may configure at all | ADM |
| `CL-WIN-005` | Intune / MDM enrolled and authorised | Deployment method | IT |
| `CL-WIN-006` | Existing lockdown / kiosk tooling in force | Whether kiosk config is permitted | IT |
| `CL-WIN-007` | Can kiosk be tested on one non-critical machine | Everything — the cheapest de-risking step available | IT |
| `CL-WIN-008` | How the browser is launched today | Whether unattended start is realistic | IT |
| `CL-WIN-009` | Are scheduled tasks permitted and do they actually run | Autostart mechanism choice | IT |
| `CL-WIN-010` | How classroom PCs are imaged, patched, monitored | Deploy-once vs push-with-image | IT |
| `CL-WIN-011` | What the display must do when the network is down | Offline tier design | TCH |
| `CL-WIN-012` | How boards recover after a crash today | Whether self-recovery is required | TCH |
| `CL-WIN-013` | What happens on an update restart mid-day | Accepted-risk statement | IT |
| `CL-WIN-014` | Sleep / screensaver / lock policy; who can change it | Lock-screen behaviour of the kiosk | IT |
| `CL-WIN-015` | Panel sizes, resolutions, scaling | Layout, type size | IT |
| `CL-WIN-016` | Which boards have digitised touch; driver type | Touch-first vs remote-first design | IT |
| `CL-WIN-018` | Dedicated PC, thin client, or embedded slot | Where the kiosk client lives; failure domain | IT |
| `CL-WIN-019` | Browser and version; is auto-update permitted | Kiosk flag and policy stability | IT |
| `CL-WIN-020` | Number of display devices now and in three years | Rollout waves | ADM |
| `CL-WIN-022` | Shared student/staff use; saved profiles | Session and profile design | TCH |
| `CL-WIN-023` | Max acceptable power-on-to-timetable delay | Boot performance target | TCH |
| `CL-SEC-001` | Is the board student-usable, or read-only when signed out | The entire student-visible surface | ADM |
| `CL-SEC-018` | **Staleness threshold** — when data becomes misleading | §5 offline tiers | ADM |
| `CL-UX-001` | Farthest reading distance | §6 minimum type size | TCH |
| `CL-DASH-018` | Do displays vary in size, resolution, orientation, aspect | §6 responsive layout | IT |
| `CL-NET-012` | DNS resolver; can it resolve local hostnames | **TLS viability** (§7 mixed content) | IT |
| `CL-NET-014` | Permitted ports and protocols | Reachability | IT |

⚠️ **The shortest path to progress is four questions:** `CL-WIN-001`,
`CL-WIN-003`, `CL-WIN-006` and `CL-WIN-007`. All four are IT, all four need a
screenshot or a yes/no, and together they determine whether §2's recommendation is
viable at all.

---

## 10. Facts to record before any implementation

| # | Fact to record | How to obtain it | Who owns it | Blocking? |
|---|---|---|---|---|
| 1 | Windows edition and build of every board | `winver` screenshot, per board model | IT (`CL-WIN-001`) | 🔴 **Yes** — determines if kiosk is possible |
| 2 | Local administrator rights on display accounts | Screenshot of local group membership | IT (`CL-WIN-003`) | 🔴 **Yes** |
| 3 | Device ownership and who holds admin credentials | Asset register | ADM (`CL-WIN-004`) | 🔴 **Yes** |
| 4 | Existing lockdown / kiosk tooling | Screenshot of current kiosk configuration | IT (`CL-WIN-006`) | 🔴 **Yes** |
| 5 | Whether a non-critical board is available to test on | Ask directly | IT (`CL-WIN-007`) | 🔴 **Yes** — the de-risking gate |
| 6 | MDM / Intune enrolment status | Tenant screenshot or licence record | IT (`CL-WIN-005`) | 🟡 Changes deployment method |
| 7 | Power, sleep and lock policy | Screenshot of power settings | IT (`CL-WIN-014`) | 🟡 Decides Assigned Access vs Shell Launcher |
| 8 | Update restart policy | Screenshot of Windows Update settings | IT (`CL-WIN-013`) | 🟡 Accepted-risk item |
| 9 | Panel size, resolution, scaling, aspect ratio | Photo of display properties | IT (`CL-WIN-015`, `CL-DASH-018`) | 🟡 Blocks layout work |
| 10 | Touch digitisation type and calibration owner | Photo of touch settings | IT / FAC (`CL-WIN-016`, `CL-WIN-017`) | 🟡 Blocks interaction design |
| 11 | Browser and version on the boards | About page screenshot | IT (`CL-WIN-019`) | 🟡 Blocks kiosk-flag decision |
| 12 | Whether scheduled tasks run and are permitted | Screenshot of a **successfully running** task | IT (`CL-WIN-009`) | 🟡 Blocks autostart fallback |
| 13 | Server reachability and permitted ports | Test result from IT | IT (`CL-NET-003`, `CL-NET-014`) | 🔴 **Yes** — no server, no display |
| 14 | Local DNS resolution of a local hostname | Successful resolution test | IT (`CL-NET-012`) | 🟡 Blocks TLS (§7) |
| 15 | Staleness threshold | School policy decision | ADM (`CL-SEC-018`) | 🟡 Blocks §5 design |
| 16 | Intended reading distance | Ask teachers | TCH (`CL-UX-001`) | 🟡 Blocks §6 type sizing |
| 17 | Acceptable power-on-to-timetable delay | Ask teachers | TCH (`CL-WIN-023`) | 🟡 Blocks performance target |
| 18 | Certificate renewal ownership and alerting | Server ops decision | IT (`CL-SRV-004`) | 🟡 §4 failure #9 |
| 19 | Power-cut history and UPS availability | Facilities record | FAC (`CL-NET-019`, `CL-SRV-003`) | 🟡 Availability target |
| 20 | Number and location of boards | Asset inventory | ADM (`CL-WIN-020`) | 🟡 Rollout planning |

⛔ **Before item 5 is answered, no kiosk configuration may be written to any board.**

---

## 11. Sources

| # | Title | URL | Type | What it supports |
|---|---|---|---|---|
| S1 | Windows Single-App and Multi-App Kiosk Configuration Options Overview | https://learn.microsoft.com/en-us/windows/configuration/kiosk/ | Microsoft Learn — vendor documentation | §1 edition support table (Assigned Access on Pro/Ent/Edu/IoT; Shell Launcher Ent/Edu only); the "cannot configure both Shell Launcher and Assigned Access" constraint; Shell Launcher replacing `explorer.exe` and not running above the lock screen; Assigned Access auto-applying lockdown policies while Shell Launcher does not; Edge kiosk as an Assigned Access experience |
| S2 | Configure a Single-App Kiosk with Assigned Access | https://learn.microsoft.com/en-us/windows/configuration/assigned-access/configure-single-app-kiosk | Microsoft Learn — vendor documentation | §2 UAC and console sign-in requirements (no RDP); supported editions; `Set-AssignedAccess`; CSP/Intune/provisioning-package/WMI Bridge delivery routes and the SYSTEM requirement; **default `Ctrl+Alt+Del` breakout sequence and its configurability**; `LogonUI` `IdleTimeOut` default of 30 seconds and its **explicit non-application to Edge kiosk mode**; Edge relaunching on re-sign-in |
| S3 | Create a Shell Launcher configuration file | https://learn.microsoft.com/en-us/windows/configuration/assigned-access/shell-launcher/configuration-file | Microsoft Learn — vendor documentation | §2 `Shell` element properties (`Shell`, `V2:AppType`, `V2:AllAppsFullScreen`); **`ReturnCodeActions` with `RestartShell` / `RestartDevice` / `ShutdownDevice` / `DoNothing`**; the four-custom-actions-plus-one-default limit and the "nothing happens" warning; `AutoLogonAccount` creating a local standard `Kiosk` user |
| S4 | Configure Shell Launcher | https://learn.microsoft.com/en-us/windows/configuration/shell-launcher/configure | Microsoft Learn — vendor documentation | §3 Shell Launcher as an optional component, enabled by the Assigned Access CSP; OMA-URI `./Vendor/MSFT/AssignedAccess/ShellLauncher`; interaction with `Run` / `RunOnce` |
| S5 | Quickstart: Configure a kiosk experience with Shell Launcher | https://learn.microsoft.com/en-us/windows/configuration/shell-launcher/quickstart-kiosk | Microsoft Learn — vendor documentation | §2 the canonical Edge-in-Shell-Launcher command line including `--kiosk-idle-timeout-minutes=2`; PowerShell/MDM Bridge application requiring SYSTEM |
| S6 | Configure Microsoft Edge kiosk mode | https://learn.microsoft.com/en-us/deployedge/microsoft-edge-configure-kiosk-mode | Microsoft Learn — vendor documentation | §2 the two lockdown experiences (Digital/Interactive Signage, Public-Browsing); **both run an InPrivate session with autofill disabled**; `--kiosk`, `--edge-kiosk-type=fullscreen`, `--edge-kiosk-type=public-browsing`, `--no-first-run`; **`--kiosk-idle-timeout-minutes` defaults (full-screen 0, public browsing 5) and the 0 / 1–1440 value range**; **the verbatim warning that the idle timeout does not restart Edge and that Assigned Access or Shell Launcher is required**; the per-experience feature support matrix (F11/F12/CTRL+N/CTRL+T blocked, multi-tab, end session, `edge://` blocking, version 89/90 availability); the kiosk-mode Group Policy table (`URLAllowlist`, `URLBlocklist`, `ConfigureKeyboardShortcuts`, `DownloadDirectory`, printer, and the public-browsing-only policies) |
| S7 | Kiosk settings for Windows in Microsoft Intune | https://learn.microsoft.com/en-us/intune/intune-service/configuration/kiosk-settings-windows | Microsoft Learn — vendor documentation | §2 Intune-delivered kiosk profile: Edge Kiosk URL, kiosk mode type, and "Refresh browser after idle time" 0–1440 minutes |
| S8 | Windows IoT Enterprise LTSC in Volume License | https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/deployment/volume-license | Microsoft Learn — vendor documentation | §1 the verbatim fixed-function restriction: *"intended for use for fixed-function specialized devices only and cannot be used as a replacement of Windows desktop for general purpose computing"*; upgrade-licence-only availability through Volume Licensing |
| S9 | What's new in Windows 10 IoT Enterprise LTSC 2021 | https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/whats-new/windows-10-iot-enterprise-ltsc-2021 | Microsoft Learn — vendor documentation | §1 build 19044, servicing 2021-11-16 to 2032-01-13, Fixed Lifecycle Policy, equivalence to Windows 10 21H2 |
| S10 | Windows IoT Enterprise release history | https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/whats-new/release-history | Microsoft Learn — vendor documentation | §1 confirmation of IoT LTSC lifecycle dates and the fixed-purpose-device framing |
| S11 | Unified Write Filter (UWF) overlay location and size | https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/customize/uwfoverlay | Microsoft Learn — vendor documentation | §3 UWF overlay behaviour; RAM overlay default cleared on reboot; 1024 MB default size; **overlay exhaustion causing failed writes and an unresponsive device**; **shutdown taking "up to an hour or longer" when the overlay is full**; automatic restart when the maximum overlay size is exceeded (Win10 19H1+) |
| S12 | `UWF_Filter` WMI provider | https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/customize/uwf-filter | Microsoft Learn — vendor documentation | §3 UWF edition support table (Home No, Pro No, Enterprise Yes, Education Yes, IoT Enterprise Yes); **administrator account required to change UWF configuration** |
| S13 | Interactive Services (Win32) | https://learn.microsoft.com/en-us/windows/win32/services/interactive-services | Microsoft Learn — vendor documentation | §3 the ⛔ finding that **services cannot directly interact with a user since Windows Vista**; **all services run in Terminal Services session 0**; `NoInteractiveServices` defaulting to 1; the `CreateProcessAsUser` workaround and its risk |
| S14 | Task Scheduler for developers | https://learn.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-start-page | Microsoft Learn — vendor documentation | §3 documented triggers — at logon, at boot, on idle, on event, on registration, on session state change |
| S15 | `ITaskSettings::put_RestartCount` | https://learn.microsoft.com/en-us/windows/win32/api/taskschd/nf-taskschd-itasksettings-put_restartcount | Microsoft Learn — vendor API reference | §3 restart-on-failure as a first-class Task Scheduler setting |
| S16 | `schtasks` commands | https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks | Microsoft Learn — vendor documentation | §3 the Administrators-group requirement for scheduling, viewing and changing tasks |
| S17 | Mixed content (MDN) | https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content | MDN — web platform reference | §7 upgradable vs blockable content; **CSS `url()` values and web fonts are blockable**; mixed content to an **IP-address host is blocked rather than upgraded**; localhost treated as a secure origin |
| S18 | `Set-Cookie` header (MDN) | https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie | MDN — web platform reference | §7 `HttpOnly` (and its documented limitation — the cookie is still *sent* on JavaScript-initiated requests), `SameSite=Strict/Lax/None`, `Secure`, `__Host-` prefix requirements |
| S19 | `questionnaires/Additional_CloudLight_Questionnaire.md` | (repository file) | Internal — requirements source | §1, §5, §6, §9, §10 every `CL-*` identifier and its priority, respondent and blocking status |

⚠️ **All platform claims above are `✅ VERIFIED` against the cited vendor or MDN
documentation. None has been tested on this school's hardware, because no board has
been touched.** Every behavioural claim about the school's actual estate, network or
panels is `❓ UNKNOWN` and is recorded in §9.

---

**Next step is not a document. It is a question.** Ask IT for a `winver` screenshot
(`CL-WIN-001`) and a local group-membership screenshot (`CL-WIN-003`) from one
classroom board, and ask whether one non-critical machine is available to test on
(`CL-WIN-007`). Until those three answers exist, §2's recommendation is a hypothesis
— a well-founded one, built entirely on Microsoft's own documentation, but still a
hypothesis about hardware nobody in this project has seen.