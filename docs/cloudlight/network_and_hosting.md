# CloudLight — School Network, Server Hosting and Reachability

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ NOTHING IN THIS DOCUMENT HAS BEEN TESTED
>
> **No school network has been accessed, probed, port-scanned, pinged, sniffed or
> otherwise interrogated.** No packet capture was taken. No device was connected
> to. No credential was attempted. **Nothing was installed on school hardware.**
>
> Every school-specific statement below is phrased as a question for the IT
> department, not as a finding. If a sentence here reads like a fact about this
> school, it is wrong — this project has no facts about this school.
>
> 🔒 **Scanning and probing the school network is prohibited by this project's own
> rules.** If something needs checking, IT checks it. See
> `docs/infrastructure/school_network.md` §"What must never happen".

---

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in authoritative vendor documentation or standards, quoted with a URL in §Sources |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | A vendor documents it; we have not confirmed it on any device this school actually owns |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No vendor claim. Our engineering judgement |
| ❓ **UNKNOWN** | No evidence either way. Not assumed. To be answered by IT |
| ⛔ **NOT RECOMMENDED / NOT PERMITTED** | Rejected by our own judgement, by vendor guidance, or by school policy once known |

---

## 1. Three separate things that are constantly confused

CloudLight's premise — *local-first, running on school infrastructure* — rests on
three distinct things that are routinely treated as one. They are not one. They
have different owners, different failure modes, and different approval paths.

### 1.1 The three things

| # | Thing | What it actually is | Who owns it | What it cannot do |
|---|---|---|---|---|
| **(a)** | **Network infrastructure** | Switches, routers, access points, VLANs, DHCP, DNS, firewall — the Layer-2 and Layer-3 fabric that moves frames and packets between devices | School IT | Run an application, hold data, or exist as a place to install software |
| **(b)** | **The host** | A general-purpose, always-on computer that runs the CloudLight backend process, its database and its search index | Whoever IT and the Principal name (`CL-SRV-004`) | Display anything by itself. It is invisible to a teacher unless a display points a browser at it |
| **(c)** | **The display** | The Windows smartboard PC in a classroom — a **client**. It renders the frontend and talks to (b) over the network | School IT / classroom owner (`CL-WIN-*`) | Store authoritative state, survive a dead host, or act as a server without school policy saying so |

### 1.2 The non-implication, stated plainly

> ### ⚠️ **(a) being true does NOT imply (b) exists, and neither implies (c) can reach (b).**

This is the single most common reasoning error in local-first deployments, and it
is worth stating as three separate implications because each fails differently:

| Implication people assume | Why it is false |
|---|---|
| *"The classroom is on the school network, therefore there is somewhere to run the server."* | **Being on a network is not being given a machine.** (a) exists whether or not (b) is authorised. `CL-SRV-001` asks whether a persistent always-on service is permitted at all; `CL-SRV-007` asks whether an existing computer may be used. Both are unanswered. |
| *"A device can reach the gateway, therefore it can reach the server."* | **Reachability to a gateway is not reachability to a host.** Inter-VLAN and intra-VLAN forwarding, firewall state and client isolation all sit between. See §5. |
| *"If the server exists, the display will show it."* | **A display needs name resolution, a permitted route, a trusted TLS certificate and a running service** — four independent links, each of which can fail alone. See §5.1. |

⚠️ **Consequence for planning:** the network is *not* the bottleneck. The
bottleneck is (b) — permission, power, ownership, uptime and recovery. A perfectly
fine network with no authorised host is an unshippable product.

### 1.3 What each layer must independently establish

| Layer | Minimum thing that must be true | Question ID |
|---|---|---|
| (a) Network | A permitted path exists from a classroom segment to a server segment | `CL-NET-003`, `CL-NET-014` |
| (b) Host | A powered, named, always-on machine is authorised to run the service | `CL-SRV-001`, `CL-SRV-002`, `CL-SRV-011` |
| (c) Display | The board is wired or has reliable wireless, and can be configured to reach a local name | `CL-NET-005`, `CL-WIN-003`, `CL-WIN-009` |

---

## 2. What an access point actually provides — and what it does not

The premise that "all smartboards can reach the same access point" is treated
throughout early CloudLight thinking as if it substantially de-risked hosting.
It does not. This section corrects that respectfully but precisely.

### 2.1 The technical correction

An access point is a **Layer-2 bridge**. It forwards Ethernet frames between
wireless stations and the wired network, giving wireless clients a presence on a
broadcast segment that looks like an Ethernet segment. Vendor documentation
describes exactly this: Cisco describes an access point as *"the connection point
between wireless and wired networks"* and operating *"in bridging and routing
mode"*; Aruba describes Client Isolation as the feature that decides whether the
AP *"forwards"* client traffic at all.

✅ **VERIFIED** — see §Sources 1, 2, 4.

### 2.2 What an AP is not

| An access point does **NOT**… | Consequence for CloudLight |
|---|---|
| Run a web server, application, database or container runtime | (b) does not exist on an AP. There is no place to install the backend. |
| Store school data | A student timetable must never be written to infrastructure nobody administers. |
| Provide persistent, reliable compute | An AP reboots on firmware upgrades, PoE events and power loss. It is not an application host. |
| Act as a proxy or terminate TLS for our application | Standard AP captive portals authenticate *users*. They are not application reverse proxies. |

### 2.3 The three variants that get mistaken for a host

| Variant | What it really is | Verdict |
|---|---|---|
| **Consumer router with a bundled web UI** | A router admin page — status, Wi-Fi settings, port forwarding. It configures the router. It does not serve our application. | ⛔ Not a platform. |
| **Enterprise "cloud-managed" AP** | Some vendors offer captive-portal and authentication services from a cloud console. These are **authentication and session services**, not application hosts. | ⛔ Not an application host. |
| **Router "port forwarding"** | Forwards inbound traffic **to a device that already exists on the LAN**. | ⛔ **It cannot conjure a host.** You cannot forward to a machine that has not been authorised, installed and powered. |

### 2.4 The conclusion that matters

> ### ⚠️ *"All smartboards can reach the same access point" establishes **at most a shared Layer-2 segment.** It does not establish that a backend can exist, and it does not establish that devices can reach each other.*

| The statement establishes | The statement does **not** establish |
|---|---|
| The boards associate with a common SSID and are bridged onto some segment | That any machine on that segment is permitted to run a service |
| A Layer-2 path exists from board to wired LAN | That the board can reach a *specific* host on that LAN |
| Some connectivity exists | That an authorisation system accepts traffic from the board to that host |
| — | That a host exists to be reached at all |

⚠️ **This silently breaks any peer-hosted design.** If the intended architecture
ever became "one classroom board hosts the backend for the others", the premise
above is exactly the assumption that would be wrong — and it would fail at
installation time, in front of staff, with no useful diagnostic.

### 2.5 Client isolation — the silent killer

**Client isolation** (also called AP isolation, station isolation, intra-VLAN
deny, "Deny inter-user bridging") is common on managed networks and the default
on most **guest** and many **BYOD** SSIDs.

✅ **VERIFIED** — Aruba documents it as isolating *"clients from one another"*
and disabling *"all peer-to-peer communication within the network"*, allowing
*"only client to gateway traffic from clients to flow"*. WatchGuard documents
that it blocks clients *"on the same AP, different radios of the same AP, and on
different APs"*, and on newer firmware also blocks communication *"with wired-side
hosts on the same network"*.

| Isolation variant | What still works | What breaks, silently |
|---|---|---|
| Guest SSID with client isolation | Board → gateway → internet | **Board → local server.** The typical failed pilot |
| Managed SSID with intra-VLAN deny | Board → gateway | **Board ↔ board**, and possibly board → wired host |
| NAT-mode SSID | Outbound only | **Anything inbound.** Devices outside cannot initiate a connection |

⚠️ **Two consequences:** any design where one display serves another fails with no
error shown, and some implementations extend isolation to wired-side hosts — which
breaks the core local-first design while the board still appears to be
"connected to Wi-Fi".

❓ **UNKNOWN** — whether this school enables any of these. This is `CL-NET-008`
(P1) and `CL-NET-009` (P1), and it must be answered **before** any pilot, because
it is invisible until the day it matters.

---

## 3. Hosting options compared

⚠️ **No option below has been costed, quoted or approved. Every row's "blocking
unknowns" column is live as of the date above.**

### 3.1 Comparison

| Option | Typical form | Pros | Cons | Blocking unknowns | Verdict |
|---|---|---|---|---|---|
| **A. Authorised always-on school computer** | Existing desktop or laptop in an office/lab, repurposed | No purchase; staff already on site; most likely to be permitted | Must not be a machine anyone needs; power and security; updates fight school IT patching | `CL-SRV-001`, `CL-SRV-002`, `CL-SRV-007`, `CL-SRV-009` | 🟡 **Preferred starting option** |
| **B. Dedicated small server** | Mini-tower or 1U box, bought and installed by us | Clean, quiet, low power, ours to manage, correct for a 24/7 service | Procurement, cost, install permission, patch policy, who owns it long-term | `CL-SRV-001`, `CL-SRV-002`, `CL-BUD-*` | 🟡 **Preferred steady-state option** |
| **C. NAS-hosted service** | Container on a NAS, fronted by the NAS reverse proxy | Already powered, backed up and monitored; zero new hardware | **Platform-dependent** capability; durability and supportability concern | `CL-NAS-001`, `CL-NAS-002`, `CL-SRV-016` | 🧪 **Possible, must be confirmed against the actual model** |
| **D. Smartboard as the server** | A classroom PC hosts the backend | No extra hardware | Display must reboot, patch and update; any classroom interruption is an outage; shared classroom = shared credentials | `CL-WIN-003`, `CL-WIN-022`, `CL-OPS-005` | ⛔ **Rejected** |
| **E. Access point or router as the server** | — | — | Not a host. See §2 | — | ⛔ **Rejected in principle** |
| **F. VM on an existing authorised host** | Hypervisor or container platform already in use | Cheap, isolated, snapshot-friendly | Tied to someone else's patching and reboot schedule; a shared host is a shared failure | `CL-SRV-009`, `CL-SRV-019` | 🧪 **Acceptable if the host owner agrees in writing** |

### 3.2 Option A — An authorised always-on school computer

An existing PC in the office or a lab, explicitly written into a school policy as
permitted to run a persistent background service for the whole working day.

- ✅ No procurement. `CL-GEN-002` and `CL-SRV-001` are permission questions, not
  capital questions.
- ✅ Somebody is physically present during the school day, which makes
  `CL-SRV-011` (who restarts it) answerable in a realistic way.
- ⚠️ It must be a machine **nobody needs for anything else**. A "temporary"
  arrangement that outlives the pilot is the normal failure mode of this option.
- ⚠️ School patching policy will eventually reboot it. `CL-SRV-019` (maintenance
  window) governs whether that is a non-event.
- ⚠️ Physical security of the room determines the security of the data
  (`CL-SRV-005`).

**Verdict: 🟡 recommended as the pilot form.** It is the option that asks the
least of the school, and it is the only one that can be answered by a yes/no
rather than by a purchase.

### 3.3 Option B — A dedicated small server

A low-power mini-tower or 1U machine, ours, placed in a secure, ventilated,
temperature-suitable room.

- ✅ Quiet, low power draw, correct physical form factor for a 24/7 service.
- ✅ We control the OS, the patch cadence and the hardware.
- ⚠️ Procurement and installation permission are separate approvals
  (`CL-SRV-002`, `CL-BUD-*`).
- ⚠️ It creates a long-term obligation: someone must own it after the project team
  moves on. This is the single biggest non-technical risk of Option B.
- ⚠️ UPS is effectively mandatory here — see §9.

**Verdict: 🟡 recommended as the steady-state form** if the pilot succeeds and
Option A proves the service has real daily use.

### 3.4 Option C — A NAS-hosted service ⚠️ platform-dependent

Some NAS platforms can run containers and provide a reverse proxy with TLS
termination and an access-control profile.

✅ **VERIFIED (vendor-documented, not verified on our hardware)** — Synology
documents **Container Manager** as running *"virtualized applications directly on
your NAS"* with Compose-based multi-container projects, and documents the
**Login Portal → Advanced** page as the place where *"Your Synology NAS can act
as a reverse proxy server"*, with rules specifying source and destination
protocol, hostname and port, plus HSTS and selectable **access control profiles**
that restrict access by source IP. See §Sources 5, 6, 7.

**This is a genuinely viable hosting option. It is also platform-dependent, and
the project's own rule applies: a vendor capability must be confirmed against the
actual unit.**

| Must be confirmed, specifically | Why it is not a given |
|---|---|
| **The exact NAS model** | Container Manager's availability and limits vary by model and by DSM version. ❓ |
| **The exact DSM version** | The reverse-proxy feature moved between DSM generations (Application Portal in DSM 6 → Login Portal in DSM 7). ❓ |
| **Available RAM for the container** | Some models ship with low memory that is largely consumed by DSM itself. `CL-SRV-013`. |
| **Who may administer it** | Container management is a privileged operation. `CL-NAS-006` asks who administers shared folders today. |
| **Whether backup snapshots the data volume** | Not automatic on every model or every volume type. `CL-SRV-016`. |
| **Vendor upgrade tolerance** | A DSM update can change behaviour or break a container. |

⚠️ **Two further concerns that are not technical capability:**

1. **Durability.** NAS appliances are frequently consumer-grade hardware on a
   five-to-ten-year replacement cycle. Building a school's timetable system on a
   device that IT plans to replace is a supportability problem, not an
   availability one.
2. **Supportability.** When the NAS fails, the person who fixes it may only know
   DSM, not the application inside it. The runbook must survive that person.

**Verdict: 🧪 viable but conditional.** Acceptable if — and only if — the actual
model and DSM version are confirmed, the NAS is not on a replacement schedule
that outruns the project, and a named person owns the container. Otherwise prefer
Option B and use the NAS only as a *backup target* (see §9).

### 3.5 Option D — A Windows smartboard acting as the server ⛔ rejected

Reject for five independent reasons, any one of which is sufficient:

| # | Reason |
|---|---|
| 1 | **Availability.** Classroom PCs sleep, restart for Windows Update mid-lesson (`CL-WIN-013`), lock, and get switched off by staff. A server that sleeps is an outage. |
| 2 | **Blast radius.** The host is in front of a class. Every restart, every patch, every failed update happens in front of students and interrupts a lesson. |
| 3 | **Session model.** Boards are typically shared. A persistent server identity on a shared machine is a credentials problem (`CL-WIN-022`, `CL-DATA-004`). |
| 4 | **Ownership.** Nobody is on the hook for keeping a classroom PC patched as a server. |
| 5 | **Recovery.** If it is down, recovery is somebody walking to a classroom — during a lesson, or after hours (`CL-OPS-005`, `CL-OPS-006`). |

⛔ **The smartboard is a client. That is its job.**

### 3.6 Option E — An access point or router acting as the server ⛔ rejected in principle

This is rejected **before** any school-specific consideration, because it rests on
the physics of what those devices are.

| Reason | Detail |
|---|---|
| **Capability** | An AP is a Layer-2 bridge (§2.1). It has no application runtime, no database, no persistent storage we may use, and no update path we control. |
| **Storage** | Storing student and staff timetable data on a device with no defined backup, no defined owner and no defined retention policy is not acceptable (`CL-SRV-005`, `CL-SRV-006`). |
| **Administration** | Its firmware is vendor-managed and updated out from under us. A platform that can change or vanish without notice cannot hold a system of record. |
| **Security** | Its admin surface is a network-infrastructure admin surface. Co-locating application data with it widens the compromise surface for no benefit. |
| **The port-forwarding fallacy** | Forwarding to "the router" or "the AP" is not hosting. It requires a host that already exists. See §2.3. |
| **Likelihood of being proposed** | High, because it sounds like the least hardware. It is worth naming explicitly so that it is recognised and rejected rather than quietly attempted. |

⛔ **Not an option. Do not cost it. Do not plan it.**

### 3.7 Option F — A VM on an existing authorised host

A virtual machine or container on a machine the school already runs and already
patches.

- ✅ Isolation and snapshotting; a bad update can be rolled back.
- ✅ No new hardware.
- ⚠️ **Shared failure.** A reboot for someone else's Windows Update takes
  CloudLight down too.
- ⚠️ **Shared patch cadence.** The host's owner decides when it reboots
  (`CL-SRV-019`).
- ⚠️ **Permission complexity.** Someone must agree that our workload is allowed
  to run on their host. `CL-SRV-009` asks whether a domain join is required.

**Verdict: 🧪 acceptable if the host owner agrees in writing**, a resource
allocation is defined, and the maintenance window is agreed. Otherwise it is
Option A with extra steps and a shared blast radius.

---

## 4. Minimum host requirements

⚠️ **These are provisional engineering sizings, not verified requirements.**
Nothing has been run. The real answer depends on `CL-SRV-002` (where the host
physically lives), `CL-SRV-013` (what storage exists) and `CL-AI-009` (whether a
GPU-capable machine exists at all).

### 4.1 Baseline host

| Capability | Minimum | Recommended | Why |
|---|---|---|---|
| **CPU** | 2 cores | 4 cores | Timetable rendering and search indexing are bursty; 2 cores makes a Monday-morning reindex visible to teachers (`CL-SRV-021`) |
| **RAM** | 4 GB | 8–16 GB | Backend process, database, indexer and search index resident at once; also absorbs OS and container overhead |
| **Storage — system** | 64 GB SSD | 256 GB SSD | An SSD is close to mandatory: a spinning disk under an index rebuild is a visible stall |
| **Storage — data + index** | 100 GB free | 250 GB+ free, or a separate volume | The index is derived but not free; `CL-SRV-013` and `CL-SRV-015` govern actual capacity |
| **Network interfaces** | 1× 1 GbE | 2× 1 GbE (second for management or a future second service) | One NIC is workable. Two allows a separate management path or later a physically separate SHANA-facing interface — see §8 |
| **UPS** | ⚠️ Effectively mandatory | Small inline UPS with monitoring | Schools have power cuts. See §9.4 and `CL-NET-019`, `CL-SRV-003`, `CL-SRV-012` |
| **OS** | Any mainstream Linux or current Windows Server | Whichever the school will actually patch | `CL-SRV-009`. Not our preference — the school's patching capability is the constraint |
| **Remote console** | Out-of-band or physical | Physical plus documented recovery | `CL-SRV-011` — someone must be able to restart it |

### 4.2 What changes if local AI inference is added

This is the single largest swing factor in host sizing, and it is entirely
unresolved. `CL-AI-009` asks whether a GPU-capable machine exists; the answer is
❓ **UNKNOWN**.

| Workload | Memory behaviour | Size |
|---|---|---|
| **Local embedding model** (for search) | Typically **well under 1 GB** resident | Comfortable on any machine in §4.1 |
| **Local generative model at 4-bit quantisation** | Roughly **0.6–0.7 GB of RAM per billion parameters**, plus KV-cache headroom | A 3B-class model ≈ 2–3 GB; a 7B-class model ≈ 5–6 GB before overhead |
| **Same model at 8-bit** | Roughly double | Doubles the memory requirement |
| **Same model at fp16** | Roughly quadruple | A 7B-class model ≈ 14 GB of weights alone |

✅ **VERIFIED (general rule of thumb)** — Hugging Face documents the load rule as
roughly `4 × X` GB for float32 and `2 × X` GB for bfloat16/float16 for an
`X`-billion-parameter model, and documents 8-bit and 4-bit quantisation as the
path to smaller-hardware inference.

⚠️ **The trap that sizing estimates usually miss:** quantising the *weights* does
not quantise the **KV cache**, which remains in full precision and grows with
context length. A model that fits when empty can fail at long context. Any sizing
must budget for a realistic context length, not a zero one.

| Host class | What it can plausibly run | Verdict |
|---|---|---|
| Baseline (§4.1) | Embeddings, search, timetable, substitution, all core features | ✅ **Core product is affordable on a baseline host** |
| +16 GB RAM, no GPU | A small CPU-inference model; slow but functional | 🧪 Only if `CL-AI-023` tolerance is generous |
| GPU-capable | A capable local model at usable speed | ❓ Entirely depends on `CL-AI-009` |

⚠️ **Recommendation:** do **not** size the host around inference until `CL-AI-009`
is answered. Size it for the core product, and treat local inference as an
upgrade that may require different hardware.

---

## 5. Addressing, naming and reachability

### 5.1 The chain that must hold

For a smartboard to display the timetable, **five links** must all succeed, in
order. Any one of them can fail alone, and each failure looks different.

| # | Link | How it fails independently | Symptom a teacher would describe |
|---|---|---|---|
| 1 | **Smartboard resolves a name** to an address | No local DNS record; resolver does not serve internal names; hosts file not set | *"It says it can't find the site."* |
| 2 | **The board reaches the host address** | Client isolation; wrong VLAN; no route; firewall default-deny | *"It just spins."* |
| 3 | **The firewall permits it** | Inbound default-deny with no rule for this flow | *"Nothing happens at all."* |
| 4 | **TLS is presented and trusted** | Self-signed certificate not trusted by the board; hostname mismatch | *"It says the connection is not private."* |
| 5 | **The backend responds correctly** | Service not running; wrong port; crash on start; authentication rejecting the session | *"It shows an error."* |

⚠️ **These five failures have different owners.** Link 2 and 3 belong to IT.
Links 1, 4 and 5 belong to us. Link 3 also belongs to whoever administers the
firewall, which may be an external ISP (`CL-NET-002`).

### 5.2 ⚠️ A working ping is not proof of working application access

| What a ping proves | What it does not prove |
|---|---|
| The host's NIC is up and the IP is reachable at Layer 3 | The **application port** is open |
| A return route exists for ICMP | The **firewall** permits the app's port (`CL-NET-014`) |
| The host is powered and on the right subnet | The **DNS name** resolves |
| — | The **TLS certificate** is trusted by the board |
| — | The **service** is running and returns a correct response |

⛔ **Therefore: never accept "it pings" as evidence that CloudLight works.** The
only acceptable test is **the board loading the actual page and showing the actual
timetable**. Design the acceptance test that way from the start.

### 5.3 DHCP reservation vs static addressing

| Approach | Pros | Cons | Recommendation |
|---|---|---|---|
| **Static on the host itself** | Works with zero involvement from IT; survives DHCP changes | ⚠️ Collision risk — someone may later DHCP-assign the same address; invisible to IT's inventory; easy to break during reimaging | 🧪 Acceptable for a pilot **only**, with a recorded address |
| **DHCP reservation** (IT binds MAC → fixed IP) | Managed; visible in the school's inventory; survives reimaging; no collision risk | Requires IT's time and their process | 🟡 **Preferred.** `CL-NET-004`, `CL-NET-006`, `CL-NET-010` |
| **DHCP dynamic, discovered by DNS service discovery** | Zero host configuration | ⚠️ Browsers do not reliably use mDNS/Bonjour for this; fragile in a managed network | ⛔ Not recommended |

⚠️ **Whatever is chosen, record it.** `CL-NET-011` asks whether the school's
addressing convention is documented anywhere. If it is not, that documentation
becomes our deliverable.

### 5.4 Local DNS vs hosts file vs hard-coded IP

| Mechanism | Who maintains it | Robustness | Recommendation |
|---|---|---|---|
| **School-managed local DNS zone** | IT (`CL-NET-013`) | ✅ Best — one place to change, every classroom benefits | 🟡 **Preferred** |
| **Hosts file on each display** | Us, per machine | 🧪 Works; breaks on reimaging; does not scale past a few boards | Fallback for a small pilot only |
| **Hard-coded IP in the frontend config** | Us | 🧪 Works; opaque; painful after any addressing change | ⛔ Last resort — and it makes the address *unforgettable*, which is the real cost |
| **FQDN with an internal wildcard record** | IT | ✅ If the school already runs one | 🟡 Worth asking (`CL-NET-012`, `CL-NET-013`) |

⚠️ **The first question to IT is not "can you add a DNS record?" but "who must
approve it, and how long does it take?"** `CL-NET-006` asks exactly this. A
thirty-day DNS lead time is an architecture constraint, not an administrative
detail.

### 5.5 ⚠️ Why a memorable local hostname matters for maintenance

This is a maintainability argument that is routinely skipped, and it is cheap to
get right.

| Choice | What a technician types at 07:40 | Failure rate |
|---|---|---|
| `https://192.168.10.57:8443` | Types the IP from memory, or scrolls a wiki | ⛔ High. Transposed digits. Wrong host entirely. Unsearchable. |
| `https://cloudlight.school.local` | Types the name | ✅ Low. Self-documenting, memorable, searchable in logs, safe to read aloud over a phone |

⚠️ **Additional benefits:** the name appears in logs and support requests instead
of an opaque address; it survives an IP change with one DNS edit rather than a
site visit; and it makes "is it the server or the network?" answerable in one
step.

⚠️ **Naming constraint:** the hostname must be **memorable when spoken aloud over
a telephone to someone who has never heard it.** Short, one word, no digits, no
ambiguous spelling.

### 5.6 Firewall rule direction

| Direction | Rule needed? | Explanation |
|---|---|---|
| **Board → server (initial request)** | ✅ **This is the one that gets blocked.** | Under a **default-deny inbound** policy, the initial packet has no prior state and is dropped. This is the single most common reason a local-first display cannot reach its own server (`CL-NET-003`, `CL-NET-014`). |
| **Server → board (return traffic)** | ✅ Not normally | Firewalls are **stateful**: once a connection is established, return traffic is permitted automatically. No separate outbound rule is needed for the response. |
| **Anyone → administration interfaces** | ⛔ Should be **denied** | Admin surfaces must not be reachable from classroom segments. See §6. |

⚠️ **The intuition to correct:** "the reply is automatic, so a one-way rule should
be enough." It is not. A **default-deny inbound** policy blocks the *initial*
packet, and there is no state for a reply to match. An explicit inbound allow rule
is required — and someone with authority must create it. `CL-NET-002` establishes
**who that is**; it may be an external ISP, not school staff.

⚠️ **Design consequence:** rule-writing is an organisational task with lead time.
Budget for it in the pilot schedule, not just in the network diagram.

### 5.7 Gateway reachable ≠ server reachable

| Statement | What it actually proves |
|---|---|
| *"The board can reach the gateway"* | The board has an IP, a mask and a default route. It can send traffic **off its subnet**. |
| *"The board can reach the server"* | Every link in §5.1 from 1 to 5 holds. |

⚠️ **These diverge constantly.** The board reaching the gateway and getting a DHCP
lease is a **network-healthy** signal — and it is exactly the signal a
non-networker uses to conclude the server is unreachable "when it's not". The two
failures have opposite diagnoses and completely different owners.

---

## 6. Segmentation

### 6.1 The principle

> ### ⚠️ **A school LAN must NOT be assumed trusted.**

| Why a school LAN is not trusted | Detail |
|---|---|
| Student devices of unknown provenance | Students bring laptops, phones and tablets. Compromised or simply misconfigured devices are normal. |
| Large, mixed user population | Students and staff share infrastructure. |
| Guest and vendor devices | Guest networks, contractor laptops, camera systems, attendance terminals. |
| Shared credential practice | Password sharing across a classroom is common. |
| Unknown management capability | ❓ We do not know whether the school can even enforce a segmentation policy (`CL-NET-001`, `CL-NET-007`). |

⚠️ **Operational consequence:** on a flat network, **any compromised display is
one hop from the control plane.** `CL-SEC-023` states this as a P0 question and is
one of the first questions the questionnaire routes to school IT.

### 6.2 Mechanisms available

| Mechanism | What it does | Who can do it | Question |
|---|---|---|---|
| **VLAN** | Splits one Layer-2 switch into separate broadcast domains | IT, if switches are managed and VLANs are authorised | `CL-NET-007`, `CL-NET-023` |
| **Client isolation** | Blocks wireless station-to-station (and sometimes wired-side) traffic | IT | `CL-NET-008` |
| **Guest network** | A pre-isolated segment, often with no route to internal networks | Already exists? | `CL-NET-009` |
| **Firewall policy** | Controls which segment may reach which | IT or the ISP (`CL-NET-002`) | `CL-NET-003`, `CL-NET-014` |
| **Nothing** | Flat LAN | — | `CL-NET-001` — ❓ **UNKNOWN, and it is P0** |

### 6.3 Recommended logical segmentation plan

⚠️ **This is a request to school IT, not a plan we can execute. It is stated so
that IT can react to a specific proposal, not a vague one.**

| Segment | Contains | May reach | Must **not** reach |
|---|---|---|---|
| **Display segment** | Classroom smartboards, teacher devices in classrooms | Display segment (subject to isolation), **server segment on the application port only**, DNS | Management segment, NAS admin interfaces, other school server roles |
| **Server segment** | The CloudLight host | Display segment for the application port, DNS, NAS file-share ports | Management segment except from the management address, CCTV, guest networks |
| **Management segment** | IT workstations, switch/AP/firewall administration | Everything | — | 

| Rule | Detail |
|---|---|
| **Administration interfaces reachable only from the management segment** | `CL-SEC-023`. Host admin console, database admin, container manager, NAS DSM — none of these from a classroom. |
| **Narrow inbound allow** | Display → server, application port, **specific source segment only**. Not "any". |
| **No outbound internet from the server segment** (unless needed) | Reduces exposure and makes exfiltration obvious. ❓ Depends on `CL-NET-015`. |
| **Robot subnet stays separate** | The SHANA robot link is `192.168.123.0/24` and must not share a broadcast domain with anything else. See §8 and `docs/infrastructure/school_network.md`. |

### 6.4 The dependencies

⚠️ **This entire section depends on answers nobody has given:**

| ID | Question | Why it blocks |
|---|---|---|
| `CL-NET-001` | Does the school segment classroom devices today? | If the answer is no, segmentation must be *created* — a much bigger ask |
| `CL-NET-007` | Are VLANs in use, and who may create one? | Determines whether the plan is even expressible |
| `CL-NET-008` | Is client isolation enabled? | Determines whether the display segment works at all |
| `CL-SEC-023` | Which administration functions must be unreachable from a student-facing segment? | Defines the actual security requirement |
| `CL-NET-023` | Is the switch/AP estate managed or unmanaged? | An unmanaged switch estate cannot support this plan |

⚠️ **And above all: whether school IT will permit any of it.** None of these are
technical questions we can answer by reading documentation.

---

## 7. Internet dependency, by feature

### 7.1 Feature dependency table

| Feature | Needs internet? | Degrades gracefully? | Notes |
|---|---|---|---|
| **Timetable display** | ❌ No | N/A | Core. Must work on a LAN with no uplink at all |
| **Substitution display** | ❌ No | N/A | Core. Same |
| **Resource browsing over SMB/file share** | ❌ No | N/A | The share is on the LAN or the NAS |
| **Local search index** | ❌ No | N/A | Indexed once, queried locally |
| **Local AI inference** | ❌ No | N/A | Runs on the host; see §4.2 |
| **Model / embedding download** | ✅ Yes — **once** | 🧪 Partially | Must be fetched while connected, then served locally thereafter. A school that is offline for a term may never fetch it |
| **Software updates** | ✅ Yes | ⛔ Not for security | ⚠️ See §7.3 |
| **Certificate renewal** | ✅ Yes, **unless an internal CA is available** | 🧪 Partially | `CL-SEC-024` asks whether the school can issue an internal certificate. An internal CA removes this dependency entirely |
| **External AI inference** | ✅ Yes | ⛔ No | ⚠️ **And likely prohibited.** Sends school content to a third party. Requires explicit approval (`CL-SEC-*`, `CL-GEN-004`). Default: ❌ **off** |

### 7.2 The principle

> ### ⚠️ **Core operation must not require the internet.**

| Must work with **no** internet | Must work with **no** local server |
|---|---|
| Timetable, substitution, countdown, resource browsing, search | Local AI inference |

The reasoning: school networks are closed, filtered, proxied or powered off at
times. `CL-NET-015` asks whether the public internet is even reachable from a
classroom, and how it is filtered. The answer changes nothing about the
architecture — because the answer must be assumed to be "no" for design purposes.

⚠️ **The design test:** if the school uplink were unplugged for a full week, would
a teacher still be able to run a lesson with CloudLight on the board? That is the
test the design must pass.

### 7.3 ⚠️ The counter-risk: an offline system cannot patch

This is stated as a **counter-risk**, not a footnote. An offline-first system has
a genuine security problem, and a design that ignores it is not more secure — it is
less secure, because it has no patch path at all.

| Risk | Detail |
|---|---|
| **No security updates** | A permanently offline host never receives OS, database, browser or library security fixes. Any vulnerability discovered in year one remains open in year three. |
| **Certificate expiry** | If renewal requires internet, an offline system eventually presents an expired certificate to every board — which browsers treat as untrusted. The dashboard dies on all screens simultaneously. |
| **Stale model and index** | Indexes and models drift as content changes. Offline, drift is invisible until a teacher reports a wrong answer. |
| **Long-lived credentials** | Credentials that never rotate are credentials that are eventually compromised. |

**Therefore, an offline-capable system still requires a documented, tested update
path.** It must record how updates reach the host, who performs them, how long
they take and what happens when they fail; it must have a scheduled window —
`CL-SRV-019` asks when the host may be taken down without disrupting classes; and
it must have been **run at least once, end to end**. If an **internal CA** is
available (`CL-SEC-024`), certificate renewal becomes a local operation and one of
the sharpest offline failure modes disappears.

⚠️ **Offline-first is a resilience property, not a security posture.** It must not
be used as a reason to skip patching.

---

## 8. How SHANA would communicate with CloudLight

⚠️ **No interface exists. Nothing has been implemented, parsed or tested.**
This section records the intended direction so the network requirements are known
in advance.

### 8.1 Intended direction

| Property | Value | Label |
|---|---|---|
| **Direction of request** | CloudLight → SHANA | 🧪 Proposed |
| **Content** | **High-level only** — "deliver this item to that room" | ✅ Mandated by `CL-SHANA-002` |
| **Never included** | Motor, path, speed, actuator, localisation, navigation commands | ⛔ **Absolute** |
| **Idempotency** | Every request carries a unique idempotency key; a retried request must not create a second delivery | 🧪 Proposed — a duplicated physical delivery is a real failure |
| **Response** | SHANA returns status: accepted, in progress, delivered, failed, cancelled | 🧪 Proposed. See `CL-SHANA-008` |
| **Authentication** | SHANA must verify the request came from *this school's* CloudLight instance | ❓ `CL-SHANA-015` (P0) |
| **Display** | CloudLight shows status on the classroom board | 🧪 Proposed. Staleness is a UX risk — `CL-SHANA-005` |

### 8.2 ⚠️ CloudLight must operate fully when SHANA is offline

| Requirement | Reason |
|---|---|
| Timetable, substitution, search, resources must not depend on robot availability | The robot is a subsystem, not a dependency |
| When SHANA is unreachable, the board must show a **clear, non-alarming** "robot unavailable" state — not an error | A teacher must be able to tell "no robot" from "CloudLight is broken" |
| No CloudLight feature may block, queue or degrade on a pending SHANA call | A stalled integration must not stall a lesson |
| SHANA remains safe and independently validated | Safety ownership never transfers to CloudLight (`CL-SHANA-003`) |

### 8.3 ⚠️ The robot link is a completely different network path

> **SHANA's robot link is wired Ethernet over Cyclone DDS on an isolated
> `192.168.123.0/24` subnet. This is a different network path from the CloudLight
> LAN, and the two must not be conflated.**

| Property | SHANA robot link | CloudLight LAN |
|---|---|---|
| **Subnet** | `192.168.123.0/24` | ❓ Unknown — school numbering |
| **Transport** | **Wired Ethernet** (manufacturer requirement) | Wired and/or wireless, ❓ unknown |
| **Protocol** | Eclipse Cyclone DDS, domain 0 | HTTPS |
| **Discovery** | UDP multicast, **frequently blocked by default** | DNS name resolution |
| **Failure if the WLAN drops** | ⚠️ **None** — the link is wired | Total loss of the dashboard |
| **Documentation** | `docs/infrastructure/school_network.md` | This document |

✅ **VERIFIED** — Unitree documents *"Go2 only supports wired connection for
development"*, and Cyclone DDS SPDP discovery uses multicast, which is frequently
blocked on managed networks.

⚠️ **The documented failure mode to avoid:** Unitree records that
`192.168.123.0/24` sharing a broadcast domain with another subnet causes discovery
to succeed while **unicast communication fails** — a fault that looks like a
network, firewall or robot fault and is very hard to diagnose. **The robot subnet
must not share a broadcast domain with the smartboard LAN.**

⚠️ **Host implication:** if the CloudLight host ever also serves SHANA, it needs
**two network interfaces** — one for the school LAN, one dedicated to the robot —
with the robot subnet on its own isolated segment. This is exactly the SHANA
topology in `docs/infrastructure/school_network.md`, and it is why §4.1
recommends a second NIC.

### 8.4 Cross-references

| Document | Scope |
|---|---|
| `docs/infrastructure/school_network.md` | The robot Ethernet link, the `192.168.123.0/24` subnet, the multicast problem, SHANA's IT questions (`SH-NET-*`) |
| `docs/cloudlight/shana_integration.md` | SHANA's side of the ownership boundary and proposed interface |
| `docs/cloudlight/shana_integration.md` | CloudLight's side of the boundary. ⚠️ **Not present in this repository at the date above** — referenced by `questionnaires/Additional_CloudLight_Questionnaire.md` |

---

## 9. Backup, recovery, power loss and storage

### 9.1 What must be backed up

| Item | Rebuildable? | Backup priority | Rationale |
|---|---|---|---|
| **Database** (timetable, substitutions, users, permissions, audit) | ⛔ **No** | 🔴 **Critical** | ⚠️ **The index is rebuildable. The database is not.** It is the only system of record |
| **Configuration** (server config, feature flags, role mappings) | 🧪 Partly | 🔴 High | Small, but reconstructible only with effort and full knowledge of the original |
| **Search index** | ✅ **Yes** | 🟡 Low | Derived data. Rebuild from source documents. Backing it up saves time, not correctness |
| **Cached model weights** | ✅ Yes | 🟢 Low | Re-downloadable — but see §7.3 if offline |
| **Logs** | 🧪 Partly | 🟡 Medium | Retention rules apply (`CL-SRV-006`). Needed for audit |

⚠️ **The common mistake is treating the index as the important thing because it
is the largest.** The index is the *derived* artefact. The database is the
*authoritative* one.

### 9.2 Where the second copy should live

| Requirement | Reason |
|---|---|
| ⛔ **Not on the same disk as the original** | A disk failure destroys both |
| ⛔ **Not in the same directory tree** | A mistaken delete takes both |
| ✅ **A different device or a scheduled, removable copy** | Survives host loss, theft and accidental deletion |
| ✅ **Off-site if at all possible** | Survives fire and flood. `CL-DATA-009` asks whether off-site is acceptable |
| ✅ **Encrypted** | Backups inherit the sensitivity of what they contain. `CL-DATA-010` |
| ✅ **Retention-bounded** | `CL-SRV-006` / `CL-DATA-*` — data must be deletable on schedule, including in backups |

⚠️ **A NAS is an excellent backup target and a poor primary host** — the inverse of
§3.4. That asymmetry is worth stating to IT, because it is often the arrangement
that satisfies everyone.

### 9.3 Who restores it, and has a restore ever been tested?

| Question | ID | Status |
|---|---|---|
| Who holds the system administrator account? | `CL-SRV-004` | ❓ **UNKNOWN** |
| Who would restart the host after an unplanned shutdown? | `CL-SRV-011` | ❓ **UNKNOWN** |
| Who would perform a restore? | `CL-SRV-017`, `CL-DATA-011` | ❓ **UNKNOWN** |
| What recovery time is acceptable? | `CL-DATA-011` | ❓ **UNKNOWN** |
| What data-loss window is acceptable? | `CL-SRV-018` | ❓ **UNKNOWN** |

⚠️ **A backup with no named restorer is not a plan.** It is a file.

⚠️ **The institutional-knowledge risk:** if exactly one person can restore
CloudLight, the system is personal knowledge, not institutional knowledge, and it
collapses when that person is absent (`CL-OPS-006`).

### 9.4 Behaviour on power loss mid-write

| Property | Finding |
|---|---|
| **Transaction atomicity** | ✅ **VERIFIED** — SQLite states that transactions are *"atomic, consistent, isolated, and durable, even if the transaction is interrupted by a program crash, an operating system crash, or a power failure"*, and that *"all changes within a single transaction either occur completely or not at all"*. |
| **The visible guarantee** | A power cut does **not** leave a half-written substitution record in the live timetable. The transaction either committed or did not. |
| **The honest limit** | ✅ SQLite also states that a power loss in the middle of a sector write *"opens SQLite up to the possibility of database corruption"*, and that *"there is nothing that SQLite can do to test for or remedy the situation"* — it assumes the operating system works as advertised. |
| **WAL caveat** | With write-ahead logging, SQLite documents that if you do not sync content to disk on commit, *"transactions are no longer durable and might rollback following a power failure"*. The durability setting matters and must be chosen deliberately. |
| **What must be decided** | `CL-DATA-013` (P0): *"What should happen to a record that is being written when power is lost mid-write?"* — discard, or recover the last complete version. |

⚠️ **Recommendation:** keep the database on a single file with proper durability
settings, and treat power loss as a **tested** scenario — not as a hypothetical.

### 9.5 UPS

⚠️ **A UPS is effectively mandatory, not a luxury.** A cut during a write is
survivable (§9.4); a cut during a write plus a failed disk is not.

| Aspect | Requirement or question | ID / rationale |
|---|---|---|
| **Is UPS or backup power available at the candidate location?** | Without it, an ordinary power cut is an outage | `CL-SRV-003` (P0) |
| **Is there a generator, and how fast is takeover?** | A generator that takes minutes is functionally a no-UPS for a database | `CL-SRV-012` |
| **What is the power-cut history?** | Sets how frequent and how long outages are | `CL-NET-019` |
| **Minimum capacity** | Enough runtime to survive a typical cut **plus** a clean shutdown | §9.4 |
| **Prefer** | A **monitored** UPS that logs an event — so "the server went down" and "the power went out" are distinguishable | Silent outages are the main risk for an unattended host |
| **Also require** | Automatic restart on power return, or a documented person-and-time | `CL-SRV-011` |

### 9.6 ⚠️ A backup that exists vs a backup that has been restored

| State | What it proves | What it does not prove |
|---|---|---|
| **Backup exists** | Something was copied | Nothing about whether it is complete, current or restorable |
| **Backup has been restored** | A restore completed successfully at least once | Nothing about future restores |
| **Backup restored and verified against a running system** | ✅ Recovery is real | — |

> ### ⚠️ **The only meaningful test is a successful restore onto a clean machine, followed by opening the database and confirming the timetable is correct.**

⚠️ **This must be scheduled, not left to judgement** — a restore test that happens
"when there is time" does not happen. It is also the cheapest possible disaster
drill: one spare machine and a few hours.

---

## 10. What must be confirmed by school IT

Each item names the `CL-*` ID that carries it in
`questionnaires/Additional_CloudLight_Questionnaire.md`, and what changes in the
design depending on the answer. ⚠️ **None of these has been answered. None may be
inferred.**

| # | ID | Question | Why the answer changes the design |
|---|---|---|---|
| 1 | `CL-SRV-001` | Is a persistent always-on service permitted on school premises? | **If no, there is no local-first design.** Everything else in this document is moot; the architecture moves to `CL-SRV-008` |
| 2 | `CL-GEN-002` | Does the school permit a persistent background service for the whole working day? | This is the operational form of #1. "Permitted" and "tolerated" differ |
| 3 | `CL-NET-001` | Does the school segment classroom devices today? | Decides whether segmentation is *requested* or *created* — a 10× difference in IT effort |
| 4 | `CL-NET-002` | Is the firewall/gateway administered in-house or by an ISP? | Determines whether we can talk to the decision-maker or must go through a vendor with lead time |
| 5 | `CL-NET-003` | May the server subnet accept connections from the classroom VLAN or wireless segment? | **The most common single cause of a failed local-first pilot.** If no, no display will ever load the dashboard |
| 6 | `CL-NET-008` | Is client isolation or AP isolation enabled? | If yes, and boards are wireless, board → server may be blocked. Invisible until it matters |
| 7 | `CL-NET-007` / `CL-NET-023` | Are VLANs in use, are switches managed, and who may create one? | Determines whether the §6.3 segmentation plan is expressible at all |
| 8 | `CL-NET-004` | Is DHCP central and managed, or static and hand-entered? | Determines whether the host gets a stable address without manual configuration |
| 9 | `CL-NET-006` | **Who must approve adding a device to DNS and DHCP?** | Determines pilot lead time. A thirty-day approval cycle is an architecture constraint |
| 10 | `CL-NET-012` / `CL-NET-013` | Can the school's resolver serve local hostnames? Is a local zone maintained? | Decides DNS vs hosts file vs hard-coded IP (§5.4), and whether §5.5's memorable hostname is achievable |
| 11 | `CL-NET-014` | Which ports and protocols are currently permitted between classroom devices and the server subnet? | Identifies the exact rules to request, and who implements them |
| 12 | `CL-NET-005` | Are the classroom boards wired or Wi-Fi? | Wireless boards make coverage and capacity fundamental feasibility questions, not preferences |
| 13 | `CL-NET-009` | Is there a guest network, and what would be needed to permit internal traffic on it? | The fallback segment if the main network cannot be modified |
| 14 | `CL-NET-016` | For how many hours is the network powered and routed each day? | Bounds the availability window the dashboard can promise |
| 15 | `CL-NET-019` | What is the power-cut history and typical outage duration? | Determines whether a UPS is mandatory and what capacity |
| 16 | `CL-NET-015` | Can the public internet be reached from a board, and is it filtered or proxied? | Affects model downloads and external inference. ❌ **Does not change the core architecture** — that stays offline-first |
| 17 | `CL-NET-024` | Is IPv6 enabled anywhere? | Determines whether the service must be dual-stack or IPv4-only |
| 18 | `CL-SRV-002` | Which physical locations are candidates for a host? | Determines whether on-premise hosting is realistic at all |
| 19 | `CL-SRV-003` / `CL-SRV-012` | Is there UPS or generator backup power, and how fast is takeover? | §9.5. A generator with a slow start is functionally a no-UPS |
| 20 | `CL-SRV-007` / `CL-SRV-009` | Is there an existing computer we may use, and what OS/domain rules apply? | Decides Option A vs Option B vs Option F |
| 21 | `CL-SRV-011` | Who physically restarts the host, and are they reachable during school hours? | Decides whether downtime is minutes or until the next working day |
| 22 | `CL-SRV-013` | What storage is available at the candidate location? | §4.1 sizing. Insufficient storage blocks resource search entirely |
| 23 | `CL-NAS-001` / `CL-NAS-002` | Does a NAS or file server exist, and what is it? | Decides Option C and the whole resource-browsing feature |
| 24 | `CL-SEC-023` | Which administration functions must be unreachable from a student-facing segment? | Defines the segmentation requirement itself |
| 25 | `CL-SEC-024` | Is HTTPS required internally, and can the school issue an internal certificate? | Determines whether the TLS link in §5.1 is satisfiable **offline** (§7.3) |
| 26 | `CL-SRV-016` / `CL-DATA-008` | Where is the second copy kept, how often, and who holds it? | §9.2. A backup on the same disk is not a backup |
| 27 | `CL-AI-009` | Is there a GPU-capable machine, or is CPU-only the realistic ceiling? | §4.2. Decides whether local inference is on the table at all |
| 28 | `CL-OPS-006` | If the usual person is absent for a week, who restores service? | Decides whether this is a system or a personal dependency |

⚠️ **Items 1, 3, 5, 6, 8, 9, 18 and 21 are the ones that stop a pilot.** The rest
shape the design; these determine whether there is one.

---

## 11. Highest-risk assumptions in this document

| # | Assumption | If false, what breaks | How to test cheaply |
|---|---|---|---|
| 1 | **A persistent always-on host is permitted** | ⛔ **The entire product.** There is no local-first platform without one | A written question to the Principal, then the office (`CL-SRV-001`, `CL-GEN-002`). **No hardware needed. Free. Ask first.** |
| 2 | **An authorised machine physically exists and can be powered** | No host → no backend → blank boards. Even with permission, an unpowered or unsuitable room ends the project | A site walkthrough with facilities (`CL-SRV-002`). Zero cost |
| 3 | **The displays can reach that host** | The dashboard never loads. The pilot burns a week before anyone notices | Once a host exists, open the exact URL from **one** classroom board and load the actual page. One board, one test |
| 4 | **No client isolation blocks board → host** | ⚠️ **Silent.** Boards associate, show signal strength, and still cannot reach the server | Read the wireless SSID configuration, or ask IT directly (`CL-NET-008`). **A question, not a scan** |
| 5 | **DHCP and DNS are manageable by the school** | Hard-coded IPs everywhere; every addressing change becomes a site visit; §5.5's memorable hostname is lost | Ask who approves records and how long it takes (`CL-NET-004`, `CL-NET-006`, `CL-NET-012`, `CL-NET-013`) |
| 6 | **The firewall can be changed by someone we can reach** | No permitted inbound path. The single most common local-first failure | Identify the administrator and the change process (`CL-NET-002`, `CL-NET-003`) |
| 7 | **The NAS supports what we would need of it** | Option C collapses to Option B; a procurement cycle appears late, when it hurts | Check the **exact model and DSM version** against vendor documentation (§3.4, §Sources 5–7). Document review, no hardware |
| 8 | **Power is stable enough, or UPS exists** | Outages mid-lesson; possible corruption during writes; repeated manual restarts | Ask for the power-cut history (`CL-NET-019`, `CL-SRV-003`). One question, no equipment |
| 9 | **The host can be restarted by a person who is present** | An outage lasts until the next working day — or until the project team returns | Ask the question and get a **name** (`CL-SRV-011`, `CL-OPS-006`) |
| 10 | **Segmentation is permitted** | A flat network; any compromised display is one hop from the control plane (`CL-SEC-023`) | Ask. If the answer is no, the design must compensate with application-layer controls and accept the risk explicitly |
| 11 | **The display can autostart and self-recover** | Every lesson needs a human. The product is unusable as a dashboard (`CL-WIN-008`, `CL-WIN-009`, `CL-WIN-012`) | Test kiosk mode on **one non-critical machine** — the questionnaire already asks permission for exactly this (`CL-WIN-007`) |
| 12 | **HTTPS can work offline** | Certificates expire; every board shows a security warning simultaneously; the dashboard dies everywhere at once | Ask whether an internal CA exists (`CL-SEC-024`) |
| 13 | **A backup can actually be restored** | ⚠️ **A backup exists and proves nothing.** Recovery is theoretical until a restore is performed | Restore one backup onto a clean machine and open the timetable. Hours of work; the entire disaster-recovery position |

⚠️ **Nine of these thirteen are answerable by asking a question.** The cheapest
sequence in the entire project is items 1, 4, 5, 6 and 8 — all of which are
existing questionnaire entries, cost nothing, and each one can independently
invalidate the design. **Ask before building.**

⛔ **And none of them may be resolved by probing.** Probing the school network
prohibits itself the answer: the school IT department is the only legitimate
source of every fact in this table.

---

## Sources

**Types:** `OFFICIAL-VENDOR` (network or storage vendor documentation) ·
`OFFICIAL-MFR` (manufacturer documentation) · `OFFICIAL-OPEN` (open-source
project documentation) · `PEER-REVIEWED` · `INTERNAL` (this repository).

**Access date for all external sources: 2026-10-11.**

| # | Title | URL | Type | What it supports |
|---|---|---|---|---|
| 1 | **Client Isolation — Aruba Central** | `https://arubanetworking.hpe.com/techdocs/central/2.5.7/content/aos10x/cfg/aps/client-isolation.htm` | OFFICIAL-VENDOR | §2.5 — Client isolation *"isolates clients from one another and disables all peer-to-peer communication within the network"*, allowing *"only client to gateway traffic"*; trusted-destination table means unlisted destinations are dropped |
| 2 | **Configuring Client Isolation / Wireless Network Profiles — Aruba Central** | `https://arubanetworking.hpe.com/techdocs/central/latest/content/nms/access-points/cfg/networks/conf_wlan_ssid.htm` | OFFICIAL-VENDOR | §2.5 — "Deny inter-user bridging" and "Deny Intra VLAN Traffic" are **per-SSID settings**; *"the clients can connect to the Internet, but cannot communicate with each other"* |
| 3 | **About AP Client Isolation — WatchGuard** | `https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Firmware/wireless/ap_station_isolation_c.html` | OFFICIAL-VENDOR | §2.5 — isolation blocks clients on the same AP, across radios, and across APs; on firmware 8.6.x+ it also blocks communication *"with wired-side hosts on the same network"* |
| 4 | **Wireless LAN Configuration Guide, Cisco IOS XE — Wireless LAN Overview** | `https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/wlan/configuration/xe-16-7/wl-16-7-book/wl-overview.html` | OFFICIAL-VENDOR | §2.1 — an access point serves *"as the connection point between wireless and wired networks"* and operates *"in bridging and routing mode"* |
| 5 | **Container Manager — Synology** | `https://www.synology.com/en-us/dsm/feature/docker` | OFFICIAL-VENDOR | §3.4 — containers run *"directly on your NAS"*; Compose-based multi-container projects; CPU/RAM/network monitoring |
| 6 | **Advanced — Login Portal, DSM 7 (Synology Knowledge Center)** | `https://kb.synology.com/en-global/DSM/help/DSM/AdminCenter/system_login_portal_advanced?version=7` | OFFICIAL-VENDOR | §3.4 — *"Your Synology NAS can act as a reverse proxy server"*; source and destination protocol/hostname/port; HSTS; selectable **access control profiles** restricting access by source IP |
| 7 | **Login Portal — DSM 7 (Synology Knowledge Center)** | `https://kb.synology.com/en-global/DSM/help/DSM/AdminCenter/system_login_portal` | OFFICIAL-VENDOR | §3.4, §5.4 — custom alias/domain and reverse-proxy or access-control-profile management; supports the DNS-naming approach in §5.5 |
| 8 | **SQLite Is Transactional** | `https://sqlite.org/transactional.html` | OFFICIAL-OPEN | §9.4 — transactions are *"atomic, consistent, isolated, and durable"* even across a power failure; *"all changes within a single transaction either occur completely or not at all"* |
| 9 | **Atomic Commit In SQLite** | `https://sqlite.org/atomiccommit.html` | OFFICIAL-OPEN | §9.4 — and the honest limit: a power loss mid-sector-write *"opens SQLite up to the possibility of database corruption"*, which SQLite cannot test for or remedy |
| 10 | **Write-Ahead Logging — SQLite** | `https://www3.sqlite.org/wal.html` | OFFICIAL-OPEN | §9.4 — WAL durability trade-off; without syncing on commit, *"transactions are no longer durable and might rollback following a power failure"* |
| 11 | **Optimizing LLMs for Speed and Memory — Hugging Face** | `https://huggingface.co/docs/transformers/llm_tutorial_optimization` | OFFICIAL-OPEN | §4.2 — the memory rule of thumb (`~4×X` GB float32, `~2×X` GB bfloat16/float16 for `X` billion parameters); 8-bit and 4-bit quantisation as the path to smaller hardware; KV-cache growth as a separate, hidden cost |
| 12 | **QLoRA: Efficient Finetuning of Quantized LLMs — Dettmers et al.** | `https://arxiv.org/abs/2305.14314` | PEER-REVIEWED | §4.2 — 4-bit quantisation as an established technique for running capable models on limited hardware |
| 13 | **`docs/infrastructure/school_network.md`** | *(this repository)* | INTERNAL | §2.5, §8.3 — the robot link is wired Ethernet over Cyclone DDS on an isolated `192.168.123.0/24`; the multicast constraint; **"Scan or probe the school network — Prohibited. IT checks what needs checking"** |
| 14 | **`docs/architecture/architecture.md`** | *(this repository)* | INTERNAL | §4.2, §8 — Phase 0 evidence-label conventions; the "local application server" component; the no-internet-dependency design principle; the AI/deterministic boundary |
| 15 | **`docs/cloudlight/shana_integration.md`** | *(this repository)* | INTERNAL | §8 — SHANA-side ownership boundary; the proposed interface; the rule that safety never transfers to CloudLight |
| 16 | **`questionnaires/Additional_CloudLight_Questionnaire.md`** | *(this repository)* | INTERNAL | §4, §6, §9, §10 — every `CL-*` identifier cited in this document, with its priority, respondent and evidence requirement |

---

⚠️ **Standing note.** Every source above is either vendor documentation about
*vendor products in general*, or this repository's own research. **None of it
describes this school.** No source was obtained by accessing, probing or scanning
any school network. The questions in §10 remain questions, and the assumptions in
§11 remain assumptions, until the school's IT department answers them in writing.

**Tests actually executed to date: zero.**