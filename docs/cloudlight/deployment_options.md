# CloudLight — Deployment Options

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ NOTHING HAS BEEN DEPLOYED
>
> **No server has been bought, borrowed, installed, configured or powered on. No
> container has been run. No NAS has been contacted. No smartboard has been
> configured as a kiosk. No cloud instance has been created. No cost has been
> quoted, and no purchase has been requested.**
>
> Every verdict in this document is a **recommendation about a decision that has
> not been made**, on hardware that has not been identified, for a school that has
> answered **none** of the 576 questions in
> [`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md).
>
> **Evidence labels:** ✅ VERIFIED · 🟡 DOCUMENTED, NOT VERIFIED ON OUR HARDWARE ·
> 🧪 PLAUSIBLE, REQUIRES TESTING · ❓ UNKNOWN · ⛔ NOT RECOMMENDED / NOT PERMITTED

---

## 1. Decision frame

### 1.1 What is actually being chosen

> ### ⚠️ **"How do we deploy CloudLight?" is three separate decisions made by three different people on three different timelines.**

The word "deployment" hides a network decision, a host decision and a display
decision. They have different owners, different lead times, and different
approval paths. Bundling them into one question is how a school ends up
approving a network change in March and discovering the host needs procurement
in September.

| | **(a) Network** | **(b) Host** | **(c) Client** |
|---|---|---|---|
| **What it is** | Switches, routers, access points, VLANs, DHCP, DNS, firewall | An always-on machine running the backend, database and index | The Windows smartboard PC — a browser or app pointed at (b) |
| **Who decides** | School IT, possibly an external ISP | School administration + facilities + IT | School IT + classroom owner |
| **Who pays** | `CL-BUD-019` | `CL-BUD-001`, `CL-BUD-002` | `CL-BUD-011` |
| **Lead time** | ❓ Unknown — `CL-NET-006` asks who approves and how long it takes | Procurement cycle, or zero if an existing machine is authorised | ❓ Unknown — `CL-WIN-003` |
| **Can fail independently?** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Failure looks like** | Board associates to Wi-Fi with full signal and cannot reach the server | Boards load nothing, or show a cached page going stale | One room is dark while others work |
| **Detailed in** | [`network_and_hosting.md`](network_and_hosting.md) §5–§7 | [`network_and_hosting.md`](network_and_hosting.md) §3–§4, §9 | [`windows_startup.md`](windows_startup.md) |

### 1.2 The three implications people collapse

| Assumption | Why it fails |
|---|---|
| *"The classrooms are on the network, so there is somewhere to run the server."* | Being on a network is not being given a machine. `CL-SRV-001` asks whether a persistent always-on service is permitted **at all** |
| *"The board can reach the gateway, so it can reach the server."* | Reachability to a gateway is not reachability to a host. Client isolation, VLAN and firewall state sit between — `CL-NET-003`, `CL-NET-008`, `CL-NET-014` |
| *"A server exists, so the boards will show it."* | Five independent links must all hold: DNS, route, firewall, trusted TLS, running service. Each fails alone, and each looks different to a teacher |

⚠️ **The bottleneck is the host, not the network.** A perfect network with no
authorised machine is an unshippable product. See
[`network_and_hosting.md`](network_and_hosting.md) §1.2.

### 1.3 Relationship to the hosting options already documented

[`network_and_hosting.md`](network_and_hosting.md) §3 lists six hosting options
under letters **A–F**. This document uses letters **(a)–(f)** for a slightly
different cut of the same space. ⚠️ **The two sets are not interchangeable** and the
mapping is:

| This document | [`network_and_hosting.md`](network_and_hosting.md) §3 | Same thing? |
|---|---|---|
| **(a)** static web server on an existing always-on PC | **A** — authorised always-on school computer | ✅ Yes, same option |
| **(b)** containerised deployment on a NAS | **C** — NAS-hosted service | ✅ Yes |
| **(c)** dedicated small server | **B** — dedicated small server | ✅ Yes |
| **(d)** smartboard as the server | **D** — smartboard as the server | ✅ Yes, and rejected there too |
| **(e)** cloud VM | **F** — VM on an *existing authorised host* | ⚠️ **No.** A cloud VM is a different jurisdiction, a different data boundary and a different outage model |
| **(f)** offline-capable appliance | *(not listed there)* | ✅ New option, §2.7 |

⚠️ **Option E in that document (an access point as the server) is not carried
forward here.** It is rejected on the physics of what an access point is, not on
school-specific considerations. Do not cost it. Do not plan it.

---

## 2. Options matrix

⚠️ **No row below has been costed. No row is approved. The "Cost class" column is
a relative ordering, not a figure.**

### 2.1 The matrix

| | Option | Server host | Client form | Update mechanism | Offline capability | Operational burden | Cost class | Suitability for a small school | Verdict |
|---|---|---|---|---|---|---|---|---|---|
| **(a)** | **Static web server on an existing always-on PC** | An existing office or lab PC, authorised in writing for a persistent background service | Browser in kiosk mode on the boards | Manual, by a named person, in an agreed window | 🟡 Full, if the model survives being repurposed | 🟢 **Lowest** — no new hardware to own, but a machine nobody can repurpose mid-pilot | 💲💲 Lowest | ✅ **Best fit for a pilot.** Asks the least of the school, and is answerable by a yes/no rather than a purchase | 🟡 **Recommended baseline** |
| **(b)** | **Containerised deployment on a NAS** | A container on an existing NAS, fronted by the NAS reverse proxy | Same as (a) | Container update by whoever administers the NAS | 🟡 Full, subject to the model | 🟠 **Medium** — container management is a privileged operation, and the NAS owner may only know DSM | 💲 Low | 🟡 Good **if** the model supports it. Usually better as a *backup target* than as the primary host | 🧪 **Viable but conditional** |
| **(c)** | **Dedicated small server** | A low-power mini-tower or 1U machine, bought and installed | Same as (a) | Manual, or scripted, by us | ✅ Full, plus a UPS | 🟠 **Medium-high** — we own the OS, patch cadence, UPS and eventual replacement | 💲💲💲 Highest | 🟡 Correct for a 24/7 service, but creates a **long-term obligation** nobody has agreed to | 🟡 **Recommended steady state** |
| **(d)** | **Windows smartboard doubling as the server** | A classroom PC hosts the backend | That same machine | Whatever happens to happen | ⛔ Unreliable — the host sleeps | ⛔ **Very high** — every failure is in front of a class | 💲 Cheapest to buy | ⛔ **Poor choice.** Losing or re-imaging one classroom board would take down **every** classroom | ⛔ **Rejected — see §2.5** |
| **(e)** | **A cloud VM** | An external provider | Same as (a) | Our own pipeline or the provider's | ❌ **Core features would need a local component anyway** | 🟠 Medium for us, but it is now **someone else's availability** | 💲💲💲 Recurring | 🟠 Only as a documented fallback — and it moves school data off premises, which `CL-SRV-005` may forbid | 🧪 **Contingency only.** ⛔ *Not* as a primary |
| **(f)** | **Offline-capable appliance** | A small pre-built box that runs the whole stack, shipped as an image | Same as (a) | Reflash or scripted update by a named person | ✅ Best of all options | 🟢 **Low operationally**, 🟠 **high at handover** — the school inherits a black box it cannot inspect | 💲💲 Low-to-mid | 🟡 Attractive for a school with very little technical staff, **but** it maximises our dependency on the vendor | 🧪 **Consider at handover, not at pilot** |

### 2.2 Option (a) — the baseline

An existing desktop or laptop in an office or lab, explicitly written into a school
policy as permitted to run a persistent background service for the whole working
day.

- ✅ **No procurement.** `CL-GEN-002` and `CL-SRV-001` are *permission* questions, not capital questions.
- ✅ **Somebody is physically present** during the school day, which makes `CL-SRV-011` answerable realistically.
- ⚠️ **It must be a machine nobody needs for anything else.** A "temporary" arrangement that outlives the pilot is the normal failure mode.
- ⚠️ **School patching will eventually reboot it.** `CL-SRV-019` governs whether that is a non-event.
- ⚠️ **Physical security of the room sets the security of the data** — `CL-SRV-005`.
- ⚠️ **We do not know whether a suitable machine exists.** `CL-SRV-007`, `CL-SRV-009`.

**Verdict: 🟡 the recommended starting point.** It is the option that asks the
least of the school, and the only one that can be answered by a conversation
rather than a purchase.

### 2.3 Option (b) — containerised on a NAS ⚠️ platform-dependent

✅ **VERIFIED (vendor-documented, not verified on our hardware)** — vendor
documentation describes container runtimes with Compose-based multi-container
projects and a reverse proxy with TLS termination and source-IP access control
profiles. See [`sources.md`](sources.md) §2.2.

⚠️ **Vendor capability is not platform capability.** This must be confirmed against
the *actual model and DSM version*, which we have not seen.

| Must be confirmed | Why it is not a given |
|---|---|
| **The exact NAS model** | Container runtime availability and limits vary by model and OS version. ❓ `CL-NAS-001`, `CL-NAS-002` |
| **Available RAM for the container** | Low-memory models spend most of their RAM on the NAS OS itself. `CL-SRV-013` |
| **Who may administer it** | Container management is privileged. `CL-NAS-006` |
| **Whether backup snapshots the data volume** | Not automatic on every model or volume type. `CL-SRV-016` |
| **Durability** | NAS appliances are often consumer-grade on a five-to-ten-year replacement cycle. Building a timetable system on a device IT plans to replace is a supportability problem |

⚠️ **The asymmetry worth saying to IT:** a NAS is an **excellent backup target and
a poor primary host**. That inversion is often the arrangement that satisfies
everyone — [`network_and_hosting.md`](network_and_hosting.md) §9.2.

### 2.4 Option (c) — dedicated small server

- ✅ Quiet, low power, correct physical form factor for a 24/7 service.
- ✅ We control the OS, the patch cadence and the hardware.
- ⚠️ Procurement and install permission are **separate approvals** — `CL-SRV-002`, `CL-BUD-001`, `CL-BUD-002`, `CL-BUD-007`.
- ⚠️ **UPS becomes effectively mandatory** here — `CL-SRV-003`, `CL-SRV-012`, `CL-NET-019`.
- ⛔ **The biggest non-technical risk is ownership after we leave.** `CL-BUD-010`, `CL-OPS-022`. A server nobody will own in year three is an asset and a liability.

**Verdict: 🟡 the steady state** if the pilot proves daily use, and if
`CL-BUD-010` has a credible answer.

### 2.5 Option (d) — the smartboard as the server ⛔ rejected

> ### ⚠️ **This option is rejected for a specific, concrete reason: losing or re-imaging one classroom board would take down every classroom, not just its own.**

A classroom PC is a *shared, visible, restorable* device. School IT reimages
boards. A board is swapped. A board is returned from repair. **In each of those
events, the entire timetable for the whole school disappears**, and it disappears
in front of a class.

Five independent reasons, any one of which is sufficient:

| # | Reason | Detail |
|---|---|---|
| 1 | **Blast radius** | One board in one room hosts the state for every room. The failure of any single display is now a whole-school outage |
| 2 | **Availability** | Classroom PCs sleep, restart mid-lesson, lock, and are switched off by staff. A server that sleeps is an outage — `CL-WIN-013` |
| 3 | **Ownership** | Nobody is on the hook for keeping a classroom PC patched as a server. It is not the job it was bought for |
| 4 | **Session model** | Boards are shared. A persistent server identity on a shared machine is a credentials problem — `CL-WIN-022`, `CL-DATA-004` |
| 5 | **Reimaging** | A routine IT action on one classroom machine becomes a full-service outage with no diagnostic and no recovery |

⛔ **The smartboard is a client. That is its job.**
See [`windows_startup.md`](windows_startup.md).

⚠️ **This option gets proposed often**, because it sounds like the least hardware
and appears to remove the entire procurement question. It is named here so that it
is **recognised and rejected rather than quietly attempted**.

### 2.6 Option (e) — a cloud VM

- ❌ **It does not remove the local requirement.** If a classroom board cannot reach a local host, it cannot reach a cloud host either — the same DNS, routing, firewall and TLS chain applies, plus a WAN hop the school may not have.
- ⛔ **It moves school data off premises**, which is a decision for `CL-SRV-005` and possibly a data-processing agreement, not for a deployment document.
- ⛔ **It inverts the availability model.** An outage is now partly somebody else's uptime, and the school cannot fix it.
- ⚠️ It **inherits the offline problem** rather than solving it.

**Verdict: 🧪 contingency only**, recorded here because `CL-SRV-008` asks whether
the school would accept it as a fallback. ⛔ **It must not be the primary
architecture**, and no design in this repository assumes it.

### 2.7 Option (f) — an offline-capable appliance

A small box, pre-imaged, that runs the whole stack and needs no assembly on site.

- ✅ **Operationally the lightest option** — the least for school staff to go wrong, which matters more than elegance at a school with very little technical staff.
- ✅ **Version-drift between machines is eliminated.**
- ⛔ **It is a black box.** At handover, the school cannot inspect, patch or extend what they were given.
- ⚠️ It **maximises our dependency** rather than minimising the school's. That is the opposite of what `CL-OPS-022` and `CL-BUD-020` ask for.
- ⛔ **Supply-chain and support end-game are unanswered.** `CL-BUD-010`.

**Verdict: 🧪 consider at handover, not at pilot.** If `CL-OPS-022` resolves to
"the school owns this and we do not", an appliance is a defensible answer. While
we remain responsible, it is not.

---

## 3. Client form comparison

### 3.1 The three forms

| Aspect | Browser in kiosk mode | Desktop wrapper (Electron/TWA-style shell) | Native application |
|---|---|---|---|
| **Update** | 🟢 Server-side; the next page load fetches the new bundle. No per-machine action | 🟠 The wrapper itself updates separately from the app — two update mechanisms | ⛔ Per-machine install, per-machine update |
| **Offline** | 🟢 Service-worker caching is possible; ⚠️ behaviour in a kiosk session is `❓ UNKNOWN` | 🟢 Local asset bundle; predictable cache | 🟢 Predictable, if built for it |
| **Accessibility** | 🟢 Standard web semantics; widest assistive-technology support | 🟡 Chromium-based, so close to web — but adds a layer | 🟠 Depends entirely on the toolkit chosen |
| **Security surface** | 🟢 Single origin; browser sandbox applies | 🟡 Node/Electron integrations can widen it substantially | 🔴 Full native privilege |
| **Maintenance** | 🟢 **One codebase** for all classrooms | 🟠 Two artefacts to keep in step | ⛔ Per-platform, per-version |
| **School IT acceptance** | 🟢 Familiar | 🟡 Needs explaining | ⛔ "Installing software on the board" invites refusal |
| **Likely to break their existing board functions** | 🟢 Low | 🟠 Medium | 🔴 High — `.msi`/`.exe` install on a school board |
| **Storage of session tokens** | ⚠️ Must not use `localStorage` on an unattended device | 🟢 Controllable | 🟢 Controllable |

### 3.2 Recommendation

🟡 **Browser in kiosk mode**, with the constraints that
[`windows_startup.md`](windows_startup.md) already establishes:

| Constraint | Source |
|---|---|
| Kiosk lockdown **requires** a shell — Assigned Access or Shell Launcher — not just a browser flag | `CL-WIN-*`; vendor documentation |
| ⛔ **A Windows service cannot be the kiosk launcher.** Services cannot interact with a user session since Windows Vista; all services run in session 0 | `CL-WIN-*` |
| ⚠️ **The kiosk idle timeout does not restart the browser by itself** — a shell is required for that, or the dashboard silently stops responding mid-lesson | Vendor-documented; verified against documentation, not on this hardware |
| ⛔ **Not on a display**: login prompts, browser chrome, and the OS taskbar. A teacher must not have to dismiss them mid-lesson | `CL-DASH-021`, `CL-UX-018` |
| ⚠️ **Unified Write Filter is powerful and easy to misuse.** A full overlay can make a device unresponsive and shutdown take a very long time | Vendor-documented; ⚠️ never enabled without a tested recovery path |
| ⛔ **Never break the school's existing board functions.** Four rules, in that document, govern this | `CL-WIN-*` |

⚠️ **Two unknowns must be settled before this can even be trialled:** whether the
boards are managed by school IT or are unmanaged (`CL-AUTH-018`), and whether one
non-critical machine can be made available to test on (`CL-WIN-007`). The
questionnaire already asks for exactly that permission.

### 3.3 Offline behaviour is the honest gap

> ### ⚠️ **Browser offline caching in a locked-down kiosk session is `❓ UNKNOWN` and is recorded as such.**
>
> Vendor documentation describes kiosk lockdown experiences that run in a
> restricted, private browsing context. Whether a service worker can register and
> persist a cache **in that specific configuration on this school's boards** has
> not been established and cannot be established without touching a board.
>
> This is a **contradiction between two requirements that are individually
> non-negotiable**: the display must keep working when the host is unreachable
> (`NFR-OFF-02`), and the display must be locked down (`FR-AUTH-02`, C7). If the
> locked-down session forbids persistent caching, one of them must give.
>
> **The resolution is not a design decision — it is a test on one non-critical
> machine**, and it is cheap. Until then, the offline tier that is actually
> achievable is unproven. See [`sources.md`](sources.md) §5.

---

## 4. Recommended baseline deployment

### 4.1 What it is

The simplest deployment that could satisfy the requirements in
[`requirements.md`](requirements.md), if — and only if — the blocking questions
are answered favourably.

**One authorised always-on PC, on the school network, serving a browser client on
the classroom smartboards, with a named backup copy elsewhere on the network.**

### 4.2 Topology

```
                     SCHOOL NETWORK
  ┌──────────────────────────────────────────────────────────────────────┐
  │                                                                      │
  │   ┌──────────────────┐                                               │
  │   │   School router  │  ──► internet ──► (updates only, never       │
  │   │   / firewall     │        for core   required for core)        │
  │   └──────────────────┘                     CL-NET-002, CL-NET-015   │
  │        ▲                                    CL-SRV-001              │
  │        │ inbound: allow, specific source                           │
  │        │ segment only, app port only      CL-NET-003, CL-NET-014   │
  │   ═════╪═══════════════  firewall / VLAN boundary  ═══════════════   │
  │        │                                    CL-NET-001, CL-NET-007  │
  │  ┌─────┴───────────────────────────────────────────┐                │
  │  │  SERVER SEGMENT                    (no admin    │                │
  │  │                                      interfaces  │                │
  │  │                                       reachable   │                │
  │  │                                       from here)  │                │
  │  │                                       CL-SEC-023  │                │
  │  │   ┌──────────────────────────────────────┐     │                │
  │  │   │  (a) CLOUDLIGHT HOST   ❓ UNIDENTIFIED│     │                │
  │  │   │                                      │     │                │
  │  │   │   existing always-on PC — OPTION (a) │     │                │
  │  │   │   ───────────────────────────────    │     │                │
  │  │   │   cloudlight.school.local:443       │     │                │
  │  │   │      backend  · database (SQLite)   │     │                │
  │  │   │      search index  · static assets  │     │                │
  │  │   │   ───────────────────────────────    │     │                │
  │  │   │   managed DNS + DHCP record          │     │                │
  │  │   │   CL-NET-004/006/010/012/013         │     │                │
  │  │   └───────────────┬──────────────────────┘     │                │
  │  │                   │ HTTPS, app port only       │                │
  │  │                   │                            │                │
  │  │   ┌───────────────┴──────────┐                 │                │
  │  │   │  ✗ NEVER  (d) BOARD-AS-SERVER            │                │
  │  │   │  One board hosts → whole school dies      │                │
  │  │   └──────────────────────────┘                 │                │
  │  │                                               │                │
  │  │   BACKUP COPY (second device, not this disk)   │                │
  │  │   CL-SRV-016 · CL-DATA-008 · encrypted        │                │
  │  └───────────────────────────────────────────────┘                │
  │                        ▲                                           │
  │                        │ client only — no state, no writes          │
  │  ┌─────────────────────┴──────────────────────────────┐             │
  │  │  DISPLAY SEGMENT                                    │             │
  │  │                                                   │             │
  │  │  ┌────────────┐  ┌────────────┐  ┌────────────┐    │             │
  │  │  │ Board 1    │  │ Board 2    │  │ Board 3    │    │             │
  │  │  │ kiosk      │  │ kiosk      │  │ kiosk      │    │             │
  │  │  │ READ-ONLY  │  │ READ-ONLY  │  │ READ-ONLY  │    │             │
  │  │  │ un-auth'd  │  │ un-auth'd  │  │ un-auth'd  │    │             │
  │  │  └────────────┘  └────────────┘  └────────────┘    │             │
  │  │                                                   │             │
  │  │  ❓ how many? Do not know.  CL-DASH-001          │             │
  │  │  ❓ wired or Wi-Fi?  CL-NET-005                  │             │
  │  │  ⚠️ client isolation may block this entirely      │             │
  │  │     CL-NET-008 — invisible until the day it      │             │
  │  │     matters                                      │             │
  │  └───────────────────────────────────────────────────┘             │
  │                                                                      │
  │  ROBOT LINK — SEPARATE SEGMENT, SHARED BROADCAST DOMAIN ❌           │
  │  192.168.123.0/24 · wired Ethernet · Cyclone DDS                     │
  │  ../infrastructure/school_network.md                          │
  │  (No interface exists. Nothing built. CL-SHANA-015 unresolved.)      │
  └──────────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────────────┐
  │  ❌ NOT IN THE BASELINE                                                 │
  │  ✗ AI assistant        — out of scope release 1 (§6 #2 requirements)  │
  │  ✗ Vector search       — out of scope release 1                       │
  │  ✗ OTA self-update     — out of scope release 1                       │
  │  ✗ SHANA submission    — out of scope release 1                       │
  │  ✗ Cloud VM            — contingency only, never primary              │
  │  ✗ Board-as-server     — rejected, §2.5                               │
  └──────────────────────────────────────────────────────────────────────┘
```

### 4.3 What this baseline assumes

> ### ⚠️ **Eleven assumptions. Every one of them is unverified, and five of them can end the project on their own.**

| # | Assumption | If false | Question |
|---|---|---|---|
| 1 | **A persistent always-on service is permitted** | ⛔ **There is no local-first product.** Everything else is moot | `CL-SRV-001`, `CL-GEN-002` |
| 2 | **A suitable physical machine exists and can be powered** | No host → no backend → blank boards | `CL-SRV-002`, `CL-SRV-007` |
| 3 | **The firewall permits the display segment to reach the host** | The dashboard never loads. **The single most common local-first failure** | `CL-NET-003`, `CL-NET-014`, `CL-NET-002` |
| 4 | **Client isolation does not block board → host** | ⚠️ **Silent.** Boards associate, show full signal, and cannot reach the server | `CL-NET-008` |
| 5 | **A trusted TLS certificate can exist offline** | Every board shows a security warning at once; the dashboard dies everywhere simultaneously | `CL-SEC-024`, `CL-AUTH-010` |
| 6 | **DHCP/DNS records can be created, and reasonably quickly** | Hard-coded IPs everywhere; every address change becomes a site visit | `CL-NET-004`, `CL-NET-006`, `CL-NET-012`, `CL-NET-013` |
| 7 | **A timetable exists in a form we can read** | Core feature has no input | `CL-TIME-001`, `CL-TIME-002`, `CL-TIME-003` |
| 8 | **A storage share exists for resources** | Browsing and search have nothing to browse | `CL-NAS-001`, `CL-NAS-002`, `CL-SRV-014` |
| 9 | **Power is stable enough, or a UPS exists** | Outages mid-lesson; possible corruption during writes | `CL-NET-019`, `CL-SRV-003`, `CL-SRV-012` |
| 10 | **A person can restart the host during school hours** | An outage lasts until the next working day | `CL-SRV-011`, `CL-OPS-006` |
| 11 | **One non-critical machine can be tested on** | Kiosk mode is untestable, so the client form stays hypothetical | `CL-WIN-007`, `CL-WIN-003` |

⚠️ **Assumptions 1, 3, 4 and 11 are answerable by asking a question and cost
nothing.** They should be asked before anything is bought.

### 4.4 What is deliberately absent from the baseline

| Absent | Why |
|---|---|
| Any AI capability | Out of scope release 1 — [`requirements.md`](requirements.md) §6 |
| Any vector index | Out of scope release 1 |
| Backend self-update | Out of scope release 1 |
| Any SHANA network interface | Built last, and `CL-SHANA-015` is unresolved |
| A second host | Single host is a deliberate **single point of failure**, accepted at pilot scale and revisited in §6 |
| Redundant power | ⚠️ Should not be absent. `CL-SRV-003` must answer this before the pilot |

---

## 5. Progression

How a pilot grows, and what each stage actually requires. ⚠️ **Stages are
gated on evidence, not on dates.**

### Stage 0 — Questions, before hardware

| Requires | Unlocks |
|---|---|
| Written answers to `CL-SRV-001`, `CL-NET-003`, `CL-NET-008`, `CL-TIME-001`, `CL-NAS-001` | The decision to proceed at all |
| `CL-WIN-007` — permission to test on one non-critical machine | The first real technical experiment |
| **Nothing bought, nothing installed** | ⛔ If assumption 1 fails, **stop here.** It costs nothing to find out |

⚠️ **Stage 0 is the cheapest and most decisive stage in the entire project, and it
is routinely skipped.**

### Stage 1 — Single host, single smartboard

| Requires | Unlocks |
|---|---|
| An authorised host (option (a)) · DNS + DHCP records · firewall rule · a trusted certificate · one named display | Proof that the five links in [`network_and_hosting.md`](network_and_hosting.md) §5.1 hold |
| A second copy of the data on a different device | The first genuine backup |
| ⛔ **Accept the display count of one.** Do not plan for the school | Evidence about real usage, real latency, real failures |

⚠️ **The acceptance test is not "it pings".** It is *the board loading the actual
page and showing the actual timetable*. Design it that way from the start.

### Stage 2 — Single host, several rooms

| Requires | Unlocks |
|---|---|
| Bindings for each display · kiosk lockdown verified on each · concurrent-access testing · the period/day profile answered | Real classroom use; the first honest latency numbers |
| `CL-DASH-018` — whether displays vary in size, resolution or orientation | Layout work driven by evidence rather than by one panel |
| A staging → production promotion step for timetable data | `FR-ADMIN-07` |
| `CL-OPS-020` — an agreed change window | Updates that cannot interrupt a lesson |

### Stage 3 — Redundant host, or a proven restore

| Requires | Unlocks |
|---|---|
| **Option (c)** a dedicated server, **or** a scheduled restore onto a clean machine that has actually been performed | Removing the single point of failure |
| A UPS with monitoring, so "the host went down" is distinguishable from "the power went out" | `NFR-AVL-03` |
| A named restorer who is not the person who set it up | Institutional rather than personal knowledge |

⚠️ **Stage 3 is the stage where projects quietly fail to arrive.** It costs money,
buys nothing visible, and is dropped when enthusiasm runs out. It is also the
stage that determines whether the school can still run its timetable if the
project team leaves.

### Stage 4 — Optional AI node

| Requires | Unlocks |
|---|---|
| `CL-AI-009` — is there a GPU-capable machine at all? · `CL-AI-023` — what latency would teachers tolerate? · `CL-SRV-013` — storage · written authorisation for the data | Semantic search, and eventually the assistant |
| ⚠️ **The model must run on a different machine or be gated behind an explicit switch** | Nothing about the core product changes |
| `CL-BUD-006` — the school accepts the compute and power cost | A decision with a running electricity bill attached |

⚠️ **Do not size the host around inference before `CL-AI-009` is answered.** Size
for the core product; treat inference as a possible different machine.

### Stage 5 — Optional SHANA integration

| Requires | Unlocks |
|---|---|
| `CL-SHANA-015` — how does SHANA verify the request came from *this* instance · `CL-SEC-016` — replay and idempotency · a separate NIC and an isolated robot segment | The delivery-request interface |
| A human approval step that is **not** CloudLight's to skip | The boundary remaining intact under load |

⛔ **The non-negotiables do not change at this stage.** CloudLight still never
controls motors. SHANA still independently validates safety. CloudLight still
operates fully when SHANA is offline. Stage 5 adds a capability; it does not move
the boundary one centimetre.

---

## 6. Upgrade triggers

> ### ⚠️ **Each threshold below is a proposal. None is measured, because nothing is
> deployed. Each names the question that would set the real number.**

| # | Trigger | Threshold *(proposal)* | Response |
|---|---|---|---|
| T1 | **Host disk usage** | > 80% on the data volume | Free space first; add storage only if growth is real, not if the alarm is noisy. `CL-OPS-016`, `CL-SRV-013` |
| T2 | **Backup failure — any failure** | **One occurrence** | ⛔ Not a threshold. A failed backup is an immediate incident until it is explained. `CL-DATA-008`, `CL-SRV-017` |
| T3 | **Backup never restored** | Still true at handover | Stage 3 is mandatory. A backup that has never been restored is a file, not a plan |
| T4 | **Response-time degradation** | Sustained > 2× the proposed dashboard target | Profile before scaling hardware. Degradation is usually the index, not the host. `CL-UX-011`, `CL-SRV-021` |
| T5 | **Number of displays** | Beyond what one host comfortably serves, **or** beyond what one person can administer | Measure before adding hardware. A display count we do not know yet: `CL-DASH-001` |
| T6 | **Staff change** | The person who can restart or restore is unavailable | ⛔ Immediate. This is the difference between a system and a personal dependency. `CL-OPS-006`, `CL-OPS-022` |
| T7 | **Certificate expiry** | Within 60 days with no renewal path | ⛔ Urgent. Every board fails at once. An internal CA removes this entirely. `CL-SEC-024`, `CL-OPS-018` |
| T8 | **Power-cut frequency** | Any cut during a write | UPS before anything else. `CL-NET-019`, `CL-SRV-003`, `CL-SRV-012` |
| T9 | **The existing host is needed for something else** | Any competing claim | ⛔ Migrate to option (c). A "temporary" host that outlives the pilot is the predictable failure mode of option (a) |
| T10 | **Permission or policy change** | School requires HTTPS internally, segmentation, or data location rules | Re-architect, do not patch around it. `CL-SEC-023`, `CL-SRV-005` |
| T11 | **End of project-team involvement** | Any date | Handover (§7 of this document) is mandatory, not optional. `CL-OPS-022`, `CL-BUD-020` |

⚠️ **T2, T6, T7 and T11 are not thresholds to be tuned. They are events that
require a response on the day they occur.**

---

## 7. Decommissioning and handover

> ### ⚠️ **This section is the requirement most likely to be dropped, which is why
> `NFR-MNT-01` is written as a requirement rather than as a hope.**
>
> A system that cannot be run by the school after the team leaves has not been
> delivered; it has been lent.

### 7.1 Questions that gate the whole section

| ID | Question | Status |
|---|---|---|
| `CL-BUD-013` | If the project stopped, would the school expect to **retain its data and timetable records, and in what form?** | ❓ UNANSWERED |
| `CL-BUD-020` | If the school wanted to continue **without the project team**, what would it need — training, documentation, or a support contact? | ❓ UNANSWERED |
| `CL-OPS-022` | Once the project team has finished, **who in the school owns this system** — a person, a department, or nobody at all? | ❓ UNANSWERED |
| `CL-BUD-010` | If the project team were to leave, **who would pay for maintenance** in the second and third year? | ❓ UNANSWERED |
| `CL-BUD-014` | Who would **own the hardware** if the project ended, and would it be redeployed elsewhere? | ❓ UNANSWERED |
| `CL-SRV-022` | What should happen to **hosted data and accounts** if the school stops using CloudLight? | ❓ UNANSWERED |

⚠️ **"Nobody at all" is a legitimate answer to `CL-OPS-022`.** If that is the
answer, the honest response is a clean export and a documented shutdown — not a
system left running with nobody watching it.

### 7.2 Data export expectations

| Category | Expected format | Rationale |
|---|---|---|
| **Timetable and substitutions** | Plain structured export — CSV plus a documented schema, or an open database format. **Not** a proprietary binary dump | A school must be able to open this in a spreadsheet years later with tools that no longer exist. `CL-BUD-013` |
| **Audit records** | Append-only export in a documented, re-readable form | They are the school's records, not ours. `CL-DATA-015` |
| **Resource metadata** | Documented schema, including the permission model | The NAS retains the files; only the derived index is ours. `CL-DATA-020` |
| **Search index** | ⛔ **Not exported.** It is derived and rebuildable | Exporting it would preserve extracted copies of documents that may have been deleted |
| **Configuration** | Documented, human-readable, version-controlled | Reproducibility, not just portability |
| **Documentation and runbooks** | Plain documents, not only a working system | "The system works" is not handover |
| **Release signing keys and secrets** | ⛔ **Never exported or transferred in a handover package** | A handover document that leaks a key is an incident. `CL-SEC-006` |

### 7.3 Ownership

| Thing | Owner | Question |
|---|---|---|
| **The hardware** | ❓ Unassigned | `CL-BUD-014` — and note a redeployed office PC becomes *someone else's* machine the moment we leave |
| **The data** | The school, throughout | `CL-BUD-013`, `CL-SRV-022`. It is the school's timetable; we are custodians, never owners |
| **The system** | ❓ Unassigned | `CL-OPS-022` |
| **Ongoing maintenance cost** | ❓ Unfunded | `CL-BUD-010` |
| **The backup copies** | The school | `CL-DATA-008`, `CL-DATA-009` |
| **Certificates** | Whoever holds the internal CA | `CL-SEC-024`, `CL-OPS-018` |

### 7.4 How the school continues without us

| Requirement | Detail | Verified by |
|---|---|---|
| **A restore has been performed by a school person** | Not by us, on our machine | `CL-OPS-015`, `CL-DATA-012` |
| **Written instructions for the common failures** | Written by someone who is leaving, then followed by someone who is not | `CL-OPS-006`, `CL-BUD-020` |
| **Named owner, named deputy** | Names, not roles | `CL-OPS-022` |
| **The timetable survives us** | Export tested by opening it in a spreadsheet | `CL-BUD-013` |
| **A documented manual fallback already exists and is used** | If the backend is down, the school still runs its timetable on paper | `CL-SEC-022`, `FR-ADMIN-10` |
| **The host's operating system can be reinstalled from documentation** | Otherwise the project ends with the machine, not with the software | `CL-SRV-009` |

⚠️ **The test of handover is not "we gave them the passwords".** It is: *a member
of staff who never met us can restore service from a backup using only written
instructions.* If that has not been demonstrated, the handover has not happened.

### 7.5 If the project stops

| Step | Action |
|---|---|
| 1 | Export the data in the documented formats of §7.2 |
| 2 | Confirm the export opens correctly in tools the school actually has |
| 3 | Hand over documentation, runbooks and the named owner |
| 4 | Remove credentials, tokens, signing keys and access from school systems |
| 5 | ⛔ **Delete derived index entries and extracted text** — they are copies of documents that were not ours to keep |
| 6 | Return or redeploy the hardware per `CL-BUD-014` |
| 7 | Record what was done, and when, in a closing note |

---

## 8. What is not decided

Everything below is undecided because the question is unanswered. ⚠️ **None of it
may be inferred, and none of it may be resolved by probing the network.**

### 8.1 Hosting — `CL-SRV-*`

| ID | Question | Blocks |
|---|---|---|
| `CL-SRV-001` | Is a persistent always-on service permitted on school premises? | ⛔ **Everything.** If no, there is no local-first design |
| `CL-SRV-002` | Which physical locations are candidates? | Whether on-premise hosting is realistic |
| `CL-SRV-003` / `CL-SRV-012` | Is UPS or generator backup power available? | Whether a database survives an ordinary cut |
| `CL-SRV-004` | Who holds the system administrator account? | Who can fix anything at all |
| `CL-SRV-007` | Is there an existing computer we may use? | Option (a) vs (c) vs (f) |
| `CL-SRV-008` | Could a cloud VM be accepted as a fallback? | Option (e) as a contingency |
| `CL-SRV-009` | What OS is allowed? Domain join required? | Everything on the host |
| `CL-SRV-011` | Who physically restarts it, and are they reachable in school hours? | Whether downtime is minutes or a day |
| `CL-SRV-013` | What storage exists at the candidate location? | Whether search is possible at all |
| `CL-SRV-016` | Where is the second copy kept? | Whether the backup is a backup |
| `CL-SRV-017` / `CL-DATA-012` | How often do backups run, and has anyone tested a restore? | Whether recovery is real |
| `CL-SRV-019` | When may the host be down without disrupting classes? | Whether updates can happen at all |
| `CL-SRV-022` | What happens to data and accounts if the school stops using CloudLight? | §7 |

### 8.2 Client — `CL-WIN-*`

| ID | Question | Blocks |
|---|---|---|
| `CL-WIN-003` | Local group membership on a board — is the needed privilege present? | Whether a kiosk agent can be installed |
| `CL-WIN-007` | Is one **non-critical** machine available to test on? | Whether the client form can be trialled at all |
| `CL-WIN-008` / `CL-WIN-009` | Autostart and self-recovery — can they be configured? | Whether every lesson needs a human |
| `CL-WIN-011` | Display timeout / screensaver behaviour | Whether the board is usable unattended |
| `CL-WIN-012` | Boot into kiosk mode unattended — is a password required? | Whether the board is usable at 08:00 |
| `CL-WIN-013` | Can automatic Windows Update restarts be deferred on the boards? | Whether a lesson is ever interrupted by an OS update |
| `CL-WIN-018` | Native panel resolution, scaling and touch | Layout, and whether kiosk mode renders correctly |
| `CL-WIN-022` | Per-device accounts and permissions | Whether the shared-board credential problem is solvable |
| `CL-AUTH-018` | Are boards enrolled and managed by school IT? | Whether we may configure them at all |
| `CL-OTA-010` / `CL-OTA-011` | May a kiosk agent be installed and kiosk config be delivered as signed configuration? | Whether §3's client form is deployable |

### 8.3 Network — `CL-NET-*`

| ID | Question | Blocks |
|---|---|---|
| `CL-NET-001` | Does the school segment classroom devices today? | Whether segmentation is *requested* or *created* — an order-of-magnitude difference in IT effort |
| `CL-NET-002` | Is the firewall in-house or with an ISP? | Whether we can reach the decision-maker |
| `CL-NET-003` | May the server subnet accept connections from the classroom segment? | ⛔ **The most common single cause of a failed pilot** |
| `CL-NET-004` | DHCP central or hand-entered? | Whether the host gets a stable address |
| `CL-NET-005` | Are the boards wired or Wi-Fi? | Whether wireless coverage becomes a fundamental feasibility question |
| `CL-NET-006` | **Who approves adding a device to DNS and DHCP, and how long does it take?** | ⛔ A thirty-day approval cycle is an architecture constraint |
| `CL-NET-007` / `CL-NET-023` | Are VLANs in use, and may one be created? | Whether the segmentation plan is expressible |
| `CL-NET-008` / `CL-NET-009` | Is client isolation enabled? Is there a guest network? | Whether the display segment works at all — ⚠️ **invisible until the day it matters** |
| `CL-NET-010` / `CL-NET-011` | Which subnet is available? Is the addressing convention written down? | Whether we can address the host at all |
| `CL-NET-012` / `CL-NET-013` | Can the school's resolver serve local hostnames? | Whether a memorable hostname is achievable |
| `CL-NET-014` | Which ports are permitted between classroom and server? | The exact rules to request, and who writes them |
| `CL-NET-019` | Power-cut history? | Whether a UPS is mandatory, and of what size |
| `CL-NET-020` | Does anything monitor the network, and does an alert reach someone? | Whether an outage is noticed before a teacher notices |
| `CL-SEC-024` | Can the school issue an internal certificate? | ⛔ Whether TLS works offline — certificate expiry as a simultaneous school-wide failure |

⚠️ **`CL-NET-001`, `CL-NET-003`, `CL-NET-006`, `CL-NET-008` and `CL-SRV-001`
are the five questions that decide whether there is a pilot at all.** All five
cost nothing to ask.

---

**Related documents:** [`requirements.md`](requirements.md) ·
[`network_and_hosting.md`](network_and_hosting.md) · [`windows_startup.md`](windows_startup.md) ·
[`ota_updates.md`](ota_updates.md) · [`shana_integration.md`](shana_integration.md) ·
[`authentication_and_authorisation.md`](authentication_and_authorisation.md) ·
[`security_and_privacy.md`](security_and_privacy.md) · [`sources.md`](sources.md)

**Deployments planned: zero. Hosts identified: zero. Costs quoted: zero. Questions answered: zero.**