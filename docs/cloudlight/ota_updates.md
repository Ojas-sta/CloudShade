# CloudLight — Over-the-Air Updates, Integrity Verification and Rollback

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ NOTHING IN THIS DOCUMENT HAS BEEN TESTED
>
> **No build has been produced, signed, published, deployed or rolled back. No
> smartboard has been enrolled. No release key exists. No certificate has been
> issued. No update agent has been installed anywhere.** Every vendor, model and
> policy referred to is a hypothesis about this school, not a finding.
>
> Every school-specific statement below is an **open question** routed to a named
> human. If a sentence here reads like a fact about this school, it is wrong.

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in a published specification or authoritative vendor document, cited in §Sources |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | A specification or vendor documents it; not confirmed against any device this school owns |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No source claims it. Our engineering judgement |
| ❓ **UNKNOWN** | No evidence either way. Deliberately not assumed |
| ⛔ **NOT RECOMMENDED / NOT PERMITTED** | Rejected by our judgement, by published guidance, or by a school policy boundary once known |

---

## 1. The core framing

> ### ⚠️ **"OTA update" is not one thing in this system.**
>
> **Conflating a React bundle refresh with robot firmware flashing is a category
> error with safety consequences.**

⚠️ One phrase covering "refresh the web page", "deploy a backend service",
"reconfigure a kiosk", "correct tomorrow's timetable" and "flash new code onto a
robot" produces a process that is **simultaneously too heavy for the first four
and too light for the fifth.** `CL-OTA-001` (P0) asks the school to settle the
scope; `CL-OTA-002` (P0) asks whether robot firmware is in scope at all.

⚠️ **The separation that matters is between artefacts that carry different blast
radius, different reversibility and different consequence.** A timetable typo is
corrected in a minute and forgotten. A substituted backend build executes code in
every classroom in the building. A bad firmware flash on a robot is a
safety-relevant physical event. Treating those three as one thing called "a
deploy" is the error this document exists to prevent.

---

## 2. Seven distinct update channels

⚠️ **No channel below has been built, and no approval authority has been
established for any of them.** `CL-OTA-003` (P0) asks who may approve a production
deployment, and is ⚠️ **the single control that separates a managed platform from
an unreviewed one** — if request and approval collapse into one role, a
compromised or careless account can ship to every classroom with no independent
check.

| # | Channel | What changes | Who approves | Blast radius | Rollback | Risk |
|---|---|---|---|---|---|---|
| **1** | **Frontend web bundle** | JavaScript/CSS/HTML the smartboards load | ❓ `CL-OTA-003` | 🟡 Every classroom that refreshes | 🟢 Republish previous bundle; trivial | 🟡 A bad bundle breaks every display at once. A **substituted** bundle is arbitrary code execution in every classroom |
| **2** | **Backend service** | The process serving the timetable, substitution and delivery APIs | ❓ `CL-OTA-003` | 🟡 **Every display depends on it.** Total loss of the product | 🟢 Redeploy previous version — ⚠️ *if* cached frontends remain compatible (§5) | 🔴 **The highest-consequence software channel.** Everything visible on a wall comes from here |
| **3** | **Smartboard kiosk configuration** | Home URL, display timeout, lockdown settings, autostart | ❓ `CL-OTA-003` | 🟢 One device at a time | 🟢 Re-push previous config | 🟡 Low individually, fleet-wide if wrong. §10 recommends shipping this as **signed data, not an install** |
| **4** | **Timetable / substitution data** | The actual lessons, rooms, teachers, periods | ❓ Timetable office | 🔴 **Every screen in the school, immediately** | 🟢 Revert the data | 🔴 **A wrong timetable is acted upon by children** on walls they cannot question. Also the *most frequently changed* artefact — §11 |
| **5** | **NAS resource search index** | Derived index over NAS documents | ❓ IT | 🟢 Search results only | 🟢 Rebuild from source | 🟢 **Derived data.** Rebuildable, so a corrupted index is an inconvenience, not an incident |
| **6** | **AI model / prompt set** | Local model weights, system prompt, retrieval config | ❓ `CL-OTA-003` + a competent reviewer | 🟡 Answer quality across the school | 🟢 Revert to previous weights/prompt | 🟡 Highest *reputational* risk: a prompt change can alter what a teacher-facing surface says. See `../architecture/architecture_decisions/` — **ADR-001: no AI model may reach the robot** |
| **7** | **SHANA integration layer** | The client that talks to the robot | ❓ Joint — CloudLight **and** SHANA owners | 🟢 Robot subsystem only | 🟢 Revert client | ⟡ **Must never become a dependency of the timetable.** A stalled integration must not stall a lesson. See `./shana_integration.md` |

### 2.1 ⛔ (8) Robot firmware — NOT part of CloudLight OTA

> **Robot firmware flashing is a higher-risk physical activity with its own
> approval, window, recovery and rollback plan.**
>
> **CloudLight must never be its delivery path.**

⚠️ `CL-OTA-002` (P0) asks the school whether it treats firmware flashing as part
of a CloudLight update or as a separate on-site activity. The reasons for keeping
them apart are not administrative preferences:

| Reason | Detail |
|---|---|
| **It is irreversible on a physical asset** | A failed flash can leave a robot that does not move at all |
| **It is safety-relevant** | Firmware governs how the machine behaves. Its approval standard cannot be the same as a CSS change |
| **Recovery is physical** | Someone goes to the robot with a cable. CloudLight has no role in that |
| **It must not sit behind a UI approval** | ⚛️ **A release approval dialog is the wrong instrument for a motor-safety decision.** Fusing web delivery with firmware would put safety decisions behind a UI approval |
| **Blast radius is physical** | Every other channel's worst case is a broken screen. This one's worst case is a machine that moves wrongly |

⛔ **The recommendation is unambiguous:** SHANA firmware is flashed by an
on-site activity with its own plan, run by its own owner, and **CloudLight is not
involved in triggering, scheduling, approving or delivering it.**

---

## 3. Why an untrusted LAN changes the update design

### 3.1 The premise

> **A school LAN is not trusted merely because it is internal.**

`./network_and_hosting.md` §6.1 establishes why: student devices of unknown
provenance, guest and vendor devices, contractor laptops, camera systems,
attendance terminals, shared credential practice, and a large mixed user
population. ⚠️ Anyone able to write to a web root, a file share or a kiosk's local
storage can substitute a build that **every classroom will load**.

⚠️ **This is not defence-in-depth theatre. On an untrusted LAN it is the primary
control.** Internal-only systems are exactly the ones that never learn they were
compromised, because nothing tells them.

### 3.2 The consequence

> **A smartboard must accept an update only if it is validly signed, and must
> refuse an otherwise well-formed unsigned build even when served from an internal
> address.**

⚠️ `CL-OTA-004` (P0) asks whether this is acceptable here. `CL-SEC-013` (P1) asks
which control stops someone without signing authority shipping a change to every
screen — client-side signature verification, an internal deployment credential,
or both.

| Design | What it delivers | Assessment |
|---|---|---|
| **Internal address trusted, signature optional** | ⛔ Convenience. Any writer to a share reaches every classroom | ⛔ **Rejected.** The untrusted-LAN premise makes this the primary vulnerability |
| **Signature verified, origin not trusted** | 🟢 The internal address becomes irrelevant to security | 🟢 **This is the design.** Origin convenience is preserved *and* integrity is enforced |

⚠️ **The rule to hold on to:** an unsigned but perfectly well-formed build served
from `192.168.x.x` must be **refused**. If the client accepts it, the signature
check is theatre.

---

## 3.1 🔴 Pilot constraint — no self-updating before signing exists

⚠️ Agent 3 asked whether update signing belongs in first-release scope. **CloudLight's
position: signing is a precondition for *self-updating*, and self-updating is out
of first-release scope.**

| | First release / pilot |
|---|---|
| Self-download and auto-apply | ⛔ **No.** Decision `D-027` |
| Who applies updates | School IT, through a controlled channel |
| Integrity of what is applied | 🔴 **Every artefact is verified before it runs** |
| If signing infrastructure is not ready | ⛔ **Updates are frozen for the pilot.** Running unsigned is not the fallback |

⚠️ **A frozen device is an inconvenience. An unsigned update on an untrusted LAN
is a fleet-wide compromise** (§3). The pilot chooses the inconvenience.

This narrows §10's "pin previous version" control: with no self-update path there
is no silent fleet drift to re-pin, so `CL-OTA-021` drops from 🟡 to 🟢 for the
pilot. It returns to 🟡 the moment self-updating is enabled.

---

## 4. Signed releases and key custody

### 4.1 The roles

| Role | Question it answers |
|---|---|
| **Who may sign?** | ❓ `CL-OTA-005` (P0). A key anyone can use is not a control |
| **Where does the signing key live?** | ❓ `CL-OTA-005` — vendor infrastructure, the school's environment, or both |
| **Must more than one signature be required?** | 🧪 For production releases, yes. Threshold signing means a single compromised key is not sufficient |
| **Who is accountable for custody?** | ❓ ⚠️ `CL-OTA-005` names no default answer. A key that is copyable, or jointly held without accountability, converts signature verification into **a rubber stamp** |
| **How often is it rotated?** | 🧪 Scheduled, and immediately on any suspected exposure |
| **How is a leaked key neutralised?** | ⚛️ **A key cannot be un-leaked. Rotation only works if clients can *refuse* an old key — which requires a revocation mechanism, not just a new key** |

### 4.2 ⚠️ Compromise resilience

> **System design must remain safe even when a signing key is compromised.**

⚠️ **This is the property that separates signing from a rubber stamp.** A
signature answers "did this come from the holder of the key?" — it does **not**
answer "should the holder of the key have signed *this*?" If a key leaks, an
attacker produces **perfectly valid signatures on arbitrary software**, and every
verification client accepts it. The only remaining control is what the software
is *permitted* to do.

| Control | Effect when the key is compromised |
|---|---|
| **Key rotation + revocation** | Bounds the window, but ⛔ does not stop the attacker inside it |
| **Signed *metadata* separately from signed *code*** | ⚠️ An attacker with the code key may still not be able to repoint clients at a new metadata server |
| **Separate keys per role** (§5) | ⛔ Compromising one role's key does not grant the others |
| **A held-back root of trust** | ⛔ **The strongest available answer**: the leaked key can be revoked because a higher authority says so |
| **🧪 Least privilege in what the artefact may do** | 🔴 **The deepest answer.** If the update system cannot grant itself new capabilities, a leaked key buys far less |

⚠️ **Offline versus online keys:**

| Key type | Use | Rationale |
|---|---|---|
| **Offline** | Anything that can sign a production release | 🟢 The key is not present on any reachable machine, so a network compromise cannot reach it |
| **Online** | Frequently reissued, lower-consequence metadata | 🟡 Convenience, against a reduced consequence if stolen |

### 4.3 ⚠️ The practical consequence for a school project

> **The signing-key custody decision is a school decision** (`CL-OTA-004`,
> `CL-OTA-005`).

⚠️ This is stated plainly because it is easy to miss: **we cannot simply choose
this.** A signing key held by a school, in a school, by school staff, is a
**responsibility the school must accept in writing** — including who holds it,
where it is stored, who can use it, and who reviews that it was used.
⛔ **An unreviewed signing key in a school environment is worse than no signing at
all**, because it manufactures a guarantee nobody is checking.

⚠️ A vendor-held key with school-revocable vendor-signed builds is the other
shape, and `CL-OTA-005` asks for both. ⚠️ **Both shapes have a failure mode, and
neither is ours to choose.**

---

## 5. Threats an update system must resist

⚠️ `CL-OTA-017` (P0) asks who would **notice and be able to block** a rogue build
reaching the smartboards. ⚠️ The school's LAN is not assumed trusted, so detection
and containment of a substituted build must be **somebody's named
responsibility**.

| Threat | What the attacker does | What defeats it |
|---|---|---|
| **Rollback attack** | Presents an **older, known-vulnerable** version and claims it is newer | Version numbers must be **monotonic**, and a client must refuse a version it has already superseded |
| **Freeze attack** | Blocks the client from ever seeing an update — indefinitely serving the same old build | Metadata with an **expiry**, so an old signed snapshot eventually becomes invalid |
| **Fast-forward attack** | **Inflates version numbers** so every future legitimate update looks like a rollback | Version numbers must be issued **only** by a higher authority that the attacker does not hold |
| **Mix-and-match attack** | Combines files from **different repository states** — a signed `index.html` with an unsigned script | One signature must cover the **complete consistent set**, not one file |
| **Arbitrary-software attack** | Compromises the **signing key** and signs their own build | Higher-authority revocation; held-back root; least privilege in what the artefact may do |
| **Key-compromise recovery** | Uses a leaked key **before** it is rotated | ⛔ Rotation alone does not help. Revocation and expiry are the actual controls |

> ### ⚠️ **A naive "just check the signature" design resists NONE of the first four.**

⚠️ **This is the most important sentence in this section.** A signature proves
*authenticity and integrity*. It does **not** prove **freshness** or
**completeness**. An old, correctly-signed build passes every signature check
perfectly. A build whose signature covers only `index.html` passes while the
script it loads is unsigned. **Signature verification is necessary and nowhere
near sufficient.**

### 5.1 The Update Framework (TUF) — the reference design

⚠️ **TUF is presented here as the reference design, not as a decision.** This
document does **not** assert that CloudLight will adopt TUF — that is a
dependency, a staffing and a scope question (`CL-OTA-003`, `CL-OTA-012`, and the
available support hours, `CL-OPS-019`).

✅ **VERIFIED** — The Update Framework is a **published specification**, not a
product. It is explicitly designed to resist the attacks above, and the
specification text names them as its threat model. ✅ **VERIFIED** — TUF
separates trust across **four roles with separate keys**: **root**, **snapshot**,
**targets** and **timestamp**, and requires **expiry times** on each role's
metadata, so that a frozen or stale repository state becomes invalid on its own.
✅ **VERIFIED** — §Sources 1, 2.

⚠️ **What TUF buys, mapped to this table:** the **root** role holds the highest
authority, so a compromised `targets` key can be revoked by a role the attacker
does not hold. **Expiring metadata** defeats the freeze attack. **Monotonic,
threshold-issued versions** defeat rollback and fast-forward. **A signature over
the complete target set** defeats mix-and-match.

⚠️ **What it costs, stated honestly:** four roles, four keys, expiry discipline,
and a client implementation that is more than "call verify()". For a school
platform run by a handful of technical staff, that is a real cost, and the honest
position is that **a simplified TUF-shaped design may be more maintainable than a
faithful one** — while acknowledging that a simplified design has, by definition,
removed some of the protection.

---

## 6. Version compatibility between frontend and backend

⚠️ `CL-REACT-010` (P1) asks whether the API must be **versioned** so future
changes cannot break deployed displays. ⚠️ `CL-OTA-006` (P1) asks what the school's
tolerance is for a smartboard running a **cached frontend one version behind** the
live backend.

### 6.1 The skew problem

⚠️ **A smartboard caches its bundle.** A cached bundle can outlive a backend
rollback. ⚠️ **Version skew is the usual reason a rollback becomes impossible** —
and that is the failure that matters most, because rollback is the recovery
mechanism every other plan in this document assumes exists.

| Relationship | Consequence |
|---|---|
| **Backend rolls back to version N, frontends cached at N+1** | 🔴 The new-old backend must answer an API the cached bundle expects. If it cannot, **every classroom breaks and the rollback fixed nothing** |
| **Backend rolls forward, frontends stale and never refreshing** | 🟡 Silent degradation: displays show slightly old data with no error |
| **Unversioned API, ever-changing shapes** | 🔴 Every backend change is potentially a fleet-wide breaking change |

### 6.2 The requirements

| Requirement | Rationale |
|---|---|
| **The API is versioned** | The contract must not change underneath a deployed device. `CL-REACT-010` |
| **The frontend reports its own version** | ⚠️ Without this, "which version is Room 12 running?" is unanswerable — and every support call becomes guesswork. `CL-GIT-018` (P1) |
| **The backend stays compatible with the oldest supported cached bundle**, within a fixed grace window | ⛔ Otherwise a rollback strands clients permanently |
| **Or the backend forces a refresh** when the cached bundle is too old to be safe | 🟢 Viable, ⚠️ but only if the display can reach the server at that moment — and it may not |
| **The grace window is defined and enforced, not assumed** | ❓ `CL-OTA-006` — the school sets the tolerance |

⚠️ **The honest statement of the trade-off:** a backend that remains backward
compatible is more expensive to build and constrains future API design; a backend
that forces refreshes is simpler but breaks any display that is offline at the
moment of the change. 🧪 **For a school where the LAN is unreliable, backward
compatibility for a defined window is the safer choice.**

---

## 7. Rollback

⚠️ `CL-OTA-007` (P1) asks **how quickly** the previous version must be restored if
a release breaks timetable display, and what classroom disruption is acceptable
to achieve it. ⚠️ **The acceptable cost of rollback determines whether rollback is
designed in or bolted on after the first outage.**

### 7.1 ⚠️ Rollback must be tested, not intended

⚠️ **`./network_and_hosting.md` §9.6 makes the same argument for backups: a
mechanism that has never been exercised is not a recovery plan, it is a hope.**

| Artefact | How it is reverted | 🧪 What must be tested |
|---|---|---|
| **Backend service** | Redeploy the previous version | ⛔ That the previous version still works against the **current** database schema and the **currently cached** frontends (§6) |
| **Frontend bundle** | Republish the previous bundle; force refresh where possible | That cached displays actually pick up the reversion rather than serving from cache indefinitely |
| **Kiosk configuration** | Re-push the previous config as **signed data** (§10) | That a kiosk which has already applied a bad config will accept the corrected one |
| **Timetable / substitution data** | Revert the data change | ⚠️ Fastest of all, and most often needed — which is exactly why §11 gives data its own path |
| **Search index** | Rebuild from source | 🟢 The only genuinely trivial rollback, because the index is derived |
| **AI model / prompt set** | Revert to previous weights/prompt | ⚠️ That reverting the artefact also reverts its *behaviour* — a partially-reverted prompt is a new artefact, not an old one |

### 7.2 The design requirement

> **Rollback speed is a business commitment, not purely a technical capability.**

⚠️ The number the school sets in `CL-OTA-007` is the number that determines
whether the design needs two application versions running side by side, whether
releases can be prepared in advance, and whether a rollback can happen at all
inside a maintenance window.

---

## 8. Health checks and staged rollout

⚠️ `CL-OTA-008` (P1) asks which checks must pass before an update is called
complete. ⚠️ Each check corresponds to a dependency that **fails silently rather
than loudly**, which is why each needs a named pass condition.

### 8.1 Gates

| Stage | Check | ⚠️ Why this specific one |
|---|---|---|
| **Before deployment** | Database connectivity | ⛔ Without it, the backend starts and serves nothing |
| **Before deployment** | **NAS mount health** | ⟡ A NAS mount that is absent at boot produces an empty search feature and an error nobody reads |
| **Before deployment** | Timetable feed freshness | ⟡ Deploying against stale data can silently publish wrong lessons |
| **After deployment** | **A signed reconnect from the pilot classroom** | 🟢 Proves the whole chain end to end — signature, transport, session, rendering — on real hardware |
| **After deployment** | AI availability (if enabled) | 🟡 Local inference failing should degrade the assistant, not the timetable |

⚠️ **The principle:** a check that has no **named pass condition** is not a check.
⚠️ `CL-REACT-019` (P1) asks the same question of health checking generally, and
⚠️ `CL-SEC-*` §7 of `./classroom_dashboard.md` establishes that **silent partial
failure is indistinguishable from "nothing is scheduled"** — which on a wall
display in front of children is a confident falsehood.

### 8.2 Staged rollout

⚠️ `CL-OTA-009` (P1) asks whether the school would accept piloting an update on
**one nominated smartboard for a full school day** before all classrooms receive
it. ⚠️ **A single-room pilot is the cheapest available blast-radius control**, and
it is the only mechanism that catches a bad release **before it reaches every
simultaneous period**.

⚠️ **One full school day, not one hour.** The value comes from the pilot board
going through a real day — first period, breaks, substitution changes, end of day.
An hour at 11:00 tests nothing that a morning would not.

### 8.3 Automatic re-pinning

⚠️ `CL-OTA-021` (P2) asks whether the platform should **automatically re-pin
failed devices to the last known-good version** rather than leave them mixed.

⚠️ **Recommendation: yes.** Without automatic re-pinning, part of the school is
silently running unapproved software indefinitely, and **mixed-version classrooms
make every support call unanswerable** — nobody can say whether the fault is the
new version, the old version, or the network between them.

| Property | Requirement |
|---|---|
| **Failure detection is per-device** | One failed board must not roll back the fleet |
| **The target is the last *known-good* version**, not simply "one version back" | ⛔ "One version back" from a bad release is correct only if the release before it was good |
| **Re-pinning is itself audited** | ⚠️ It is a silent change to what a classroom is running, and §14 requires it recorded |
| **The school may override** | `CL-OTA-021` is conditional and is a school decision |

---

## 9. Failure modes

⚠️ `CL-OTA-013` (P1) and `CL-OTA-022` (P2) cover these; `CL-OPS-018` (P0) covers
certificates.

| Failure | Required behaviour | ⚠️ Why |
|---|---|---|
| **Network drops mid-update** | 🟢 **Abort cleanly to the previous verified version.** ⛔ ⛔ Never resume from a partial download without re-verification | Resuming from a partial download without integrity re-verification can leave a kiosk unbootable |
| **Disk full during update** | 🟡 **Check free space before starting**, and fail before writing anything | ⛔ A full disk is also a silent-failure mode elsewhere: `CL-OPS-016` — an unattended display box with a full disk is a common and entirely avoidable outage |
| **Power loss mid-update** | 🟢 The previous version must still be bootable | ⚠️ Schools have power cuts. `./network_and_hosting.md` §9.5 — UPS is effectively mandatory on the host |
| **Certificate expiry makes the update server unreachable** | 🟡 **An offline-capable update path must survive it**, or expiry must be tracked and alarmed | ⛔ `CL-OPS-018` (P0) — an expired certificate makes a perfectly good system unreachable, **on every display at once** |
| **Kiosk reimaged or returned from repair** | 🟢 **Re-enrol automatically onto the last approved version** (`CL-OTA-022`) | ⚠️ A rebuild wipes local trust state; without re-enrolment the fleet **silently drifts from what was approved** |

⚠️ **The common thread:** every one of these produces a **partially applied state**,
and the only defence is that the previous known-good version is never destroyed
until the new one is verified. 🧪 **That is a design property to build in from the
start, not a behaviour to add after the first incident.**

---

## 10. Windows permissions and school IT restrictions

⚠️ `CL-OTA-010` (P0) asks whether kiosk accounts have the **Windows privilege to
install an updating agent and write to a machine-wide application folder**.
`CL-OTA-012` (P0) asks whether policy permits **automatic** deployment or requires
every deployment to be an explicit logged human action.

### 10.1 The privilege question

| Answer | Consequence |
|---|---|
| **Kiosk has elevation** | 🟢 A self-updating agent is possible; routine changes need no human |
| **Kiosk is a standard account** | ⛔ **Every update becomes a manual IT visit** |

⚠️ **The second row changes the cost model entirely.** A per-device manual visit,
repeated for every classroom on every release, is a different product: rollout time
scales with room count, and an update is only as fast as the technician's round.
`CL-OTA-010` is therefore **not a permissions formality — it decides whether
automated updates are possible at all.**

### 10.2 The policy constraints

| Constraint | Question | ID |
|---|---|---|
| **Unattended automatic deployment permitted?** | 🧪 If not, the design needs a deliberate, auditable **push-with-confirmation** flow — a silent agent is ⛔ prohibited, not merely discouraged | `CL-OTA-012` (P0) |
| **Scheduled-task restrictions** | ⚠️ Whether an updating task may run without a logged-in user | `CL-OTA-010` |
| **Group Policy / MDM constraints** | ⚠️ Whether a third-party agent may install, update and self-protect | `CL-OTA-010`, `CL-OTA-012` |
| **Software installation policy** | ⚠️ Whether *anything* may be installed on classroom devices by a non-IT party | `CL-OTA-012` (P0) |

⚠️ **Recommendation if automation is prohibited:** do not attempt to defeat it.
Design a **push-with-confirmation** flow — the update is proposed, visible, and
requires an explicit logged action to apply — so that the process is auditable
rather than hidden.

---

## 11. Configuration and data need their own promotion paths

### 11.1 Configuration as signed data, not as an install

⚠️ `CL-OTA-011` (P1) asks whether kiosk configuration — home URL, display timeout,
lockdown settings — can be delivered as **signed configuration data** rather than
as a software install.

> ### ⚠️ **Recommendation: yes. Routine configuration changes should not require the privileged install path at all.**

| Benefit | Reason |
|---|---|
| **No elevation required** | ⚠️ If `CL-OTA-010` comes back "no elevation available", signed-data config is the **only** path that still works |
| **Independently auditable** | A config change is a data record, reviewable without reading a build |
| **Independently reversible** | Revert the data, not the software |
| **Signed** | 🟢 The same integrity control as code — an attacker who substitutes the config file still cannot forge a signature |
| **Narrower blast radius** | ⛔ A config change cannot install code, because there is no code in it |

⚠️ `CL-OTA-011` is marked **conditional** in the questionnaire, and ❓ depends on
`CL-OTA-010`. ⚠️ **If elevation is unavailable, this stops being an optimisation
and becomes the design.**

### 11.2 Data updates must not queue behind a software release

⚠️ `CL-OTA-019` (P1) asks whether a correction to tomorrow's timetable is an
**urgent operational fix** that bypasses the normal update window, or follows the
same process as a software deployment.

> ### ⚠️ **A timetable correction is an urgent operational fix. It must NOT be blocked behind a software-release approval process.**
>
> **Data and code need separate approval and promotion paths, with separate audit
> trails.**

⚠️ **The reasoning is a matter of proportionality.** A timetable data change is a
record correction, made by the people who own the timetable, publishable in
minutes, and fully reversible by reverting a record. Putting it through a software
release pipeline means **an urgent room swap is delayed by engineering release
discipline it does not need** — and the predictable response is for someone to
bypass the pipeline, at which point the audit trail is gone too.

| Artefact | Approval path | Audit trail | Reversal |
|---|---|---|---|
| **Timetable / substitution data** | Timetable office, on the school's own authority | ⚠️ **Separate** — who corrected what, when | Revert the record |
| **Configuration** | 🧪 IT or a scoped administrator (`CL-ADMIN-020`) | ⚠️ **Separate** | Revert the config data |
| **Software release** | `CL-OTA-003`, a distinct approver from the requester | ⚠️ **Separate** | Redeploy previous version |

⚠️ `CL-OTA-020` (P2) asks who promotes configuration from staging to production,
and whether that can happen without a full software deployment. ⚠️ **The answer
should be yes**, and `CL-ADMIN-020` (P3) is the related question of whether the
timetable department, library and facilities each hold their own limited admin
rights rather than routing every request through one person.

---

## 12. Change windows and the school calendar

⚠️ `CL-OTA-018` (P1) asks which windows are safe, given that **no update may
interrupt a scheduled lesson or an examination period**. ⚠️ `CL-OPS-020` (P0) asks
whether the platform may be down or under change **during exam weeks**.

> **⛔ Frozen maintenance window during examinations.**
> **⛔ No update may interrupt a scheduled lesson.**

⚠️ **An update that collides with an exam is an incident regardless of technical
correctness.** This is a school-calendar constraint that determines the entire
rollout strategy, and it must be settled **before** a release cadence is chosen —
because a cadence that does not fit the calendar will simply be violated.

| Requirement | Rationale |
|---|---|
| **A published, school-agreed change window** | `CL-OTA-018`. Without one, every deployment is an argument |
| **A frozen window during exam periods** | `CL-OPS-020`. ⚠️ Exam weeks differ from the software calendar and are ❓ **UNKNOWN** |
| **The window is bounded by lesson periods, not by clock time** | ⚠️ A "4pm–6pm" window is meaningless in a school running evening classes or Saturday batches |
| **Emergency data fixes are exempt** | ⚠️ §11 — an urgent timetable correction is not a maintenance-window activity. **This exemption must be explicit, or the freeze will be bypassed illegitimately** |
| **The host's patching schedule is aligned** | ⚠️ `network_and_hosting.md` §3.2 — school patching will eventually reboot the host, and `CL-SRV-019` asks when it may be taken down |

---

## 13. Offline recovery

⚠️ `CL-OTA-014` (P1) asks **how long** update infrastructure may be unreachable
before the platform must remain **fully usable on cached bundles and cached data**.
❓ **UNKNOWN**, and it is a school tolerance, not a design assumption.

### 13.1 Two requirements that pull in opposite directions

> **A permanently offline system cannot receive security patches.**
>
> **Therefore an offline-capable system still requires a documented, tested update
> path.**

⚠️ `./network_and_hosting.md` §7.3 states this as a **counter-risk**: offline-first
is a resilience property, **not a security posture**, and it must not be used as a
reason to skip patching. A vulnerability discovered in year one remains open in
year three.

| Requirement | Consequence |
|---|---|
| **The platform stays fully usable on cached bundles and cached data** for the offline window the school sets | 🟢 This is what makes local-first real. ❓ The window is `CL-OTA-014` |
| **A documented path for security updates must exist**, even if the updates themselves are delivered locally | 🟢 The path may be manual and local; **it must exist and must have been exercised** |
| **Certificate renewal must not depend on the internet** | 🟢 An internal CA removes this dependency entirely (`CL-SEC-024`, `CL-OPS-018`) |
| **The path must record how, who, how long, and what happens on failure** | ⚠️ An update path that has never been run end to end has not been tested, only written |

⚠️ **The question worth asking the school is not "can updates be local?" but "what
is the security-update path when the internet is unavailable for a term?"** A
school that never has reliable internet still needs to receive a security fix, and
someone must own delivering it.

---

## 14. Audit

⚠️ `CL-OTA-015` (P1) asks **for how long, and in whose custody**, the record is kept
of which update reached which smartboard, when, by whom, with what result. ⚠️
After an incident, the deployment log is **the only evidence that bounds which
devices were exposed and which were safe.**

| Must be recorded | Why |
|---|---|
| **What was deployed** — artefact, version, and — for releases — its signature |
| **Which devices** received it, and which did not | ⚠️ Blast radius cannot be bounded after the fact without this |
| **When**, per device | ⚠️ Devices update at different times; "the 14:00 release" is not a per-device fact |
| **By whom, and approved by whom** | ⚠️ `CL-OTA-003` — request and approval are separate facts |
| **The result** — success, failure, rollback, re-pin | ⚠️ A failure that was silently retried is not visible without this |

⚠️ `CL-OTA-016` (P2) asks whether the deployment log must be **append-only or
externally verifiable**, so that someone with administrator rights cannot quietly
edit or delete a past deployment record.

> ### ⚠️ **Insider tampering is a realistic threat precisely because whoever administers updates also has database access.**

⚠️ **They can change their own history if the log is an ordinary table they can
write to.** The log must therefore be append-only or externally verifiable —
hash-chained, written to a destination the deployer does not control, or exported
somewhere they cannot reach. ⚠️ This is the same requirement as `CL-SEC-019` (P0)
for the authentication audit log, and **the same argument applies exactly.**

⚠️ And once more: ⚠️ **protecting the record does not ensure anyone reads it.**
`CL-ADMIN-011` asks whether administrator activity is reviewable by someone else —
for updates, `CL-ADMIN-004` and `CL-ADMIN-019` ask who may deploy at all and
whether vendor-made changes are approved like any other.

---

## 15. Prevention of unauthorised updates

⚠️ `CL-OTA-017` (P0): **who here would notice and be able to block a rogue build
reaching the smartboards** — for example a replaced release file on a share, or a
fake updater installed locally on a kiosk?

⚠️ **The threat is assumed realistic precisely because the LAN is not trusted.**
This is not paranoia about a determined attacker; it is the ordinary consequence of
students, guests and contractors all sharing a network with classrooms.

| Layer | Prevention | Requires |
|---|---|---|
| **Client-side signature verification** | 🟢 **The primary control.** An unsigned or wrongly-signed build is refused at the device, whatever its origin | `CL-OTA-004` (P0), `CL-SEC-013` (P1) |
| **Restricting who can write to the update location** | 🟡 Defence in depth. Reduces opportunity; does not remove it | School IT policy |
| **An internal deployment credential** | 🟡 A second, independent check on the publisher side | `CL-SEC-013` |
| **Monitoring** | 🟡 Detects a substituted build after the fact | ⚠️ **Only if someone is watching** — `CL-OPS-007`, `CL-OPS-008` ask how faults are noticed today and which channels are genuinely read |
| **Approval separation** | 🟡 Requires a different person to approve | `CL-OTA-003`, `CL-ADMIN-004` |
| **Append-only deployment log** | 🟡 Makes tampering visible after the fact | `CL-OTA-016`, `CL-SEC-019` |

⚠️ **The honest position:** the primary control is the signature, and the secondary
controls are detection. **Detection without a named watcher is not a control** —
which is why `CL-OTA-017` asks for a *person*, not a process.

---

## 16. What is not decided

⚠️ **Nothing here is ours to decide.** ⚠️ **All are recorded as `UNANSWERED` in
`../../questionnaires/Additional_CloudLight_Questionnaire.md` as of 2026-10-11.**
⚠️ Rows sharing an answerer and a blocking theme are grouped.

| ID | Question | Who can answer | What it blocks |
|---|---|---|---|
| `CL-OTA-001`, `CL-OTA-002` | Which artefacts count as an update; is robot firmware in scope | Administration / robotics | ⛔ The scope of this entire document, and the firmware boundary |
| `CL-OTA-003` | Who may approve a production deployment; must it be a different person from the requester | Administration | ⛔ **The control that separates a managed platform from an unreviewed one** |
| `CL-OTA-004`, `CL-OTA-005`, `CL-SEC-013` | Must updates be validly signed; where does the key live; which control stops an unauthorised build | IT | ⛔ **The primary integrity control on an untrusted LAN**, and key custody |
| `CL-OTA-006`, `CL-REACT-010` | Tolerance for a cached frontend behind the live backend; must the API be versioned | IT | ⛔ Whether a rollback is possible at all (§6) |
| `CL-OTA-007` | How quickly must the previous version be restored, and what disruption is acceptable | Administration | Whether rollback is designed in or bolted on after the first outage |
| `CL-OTA-008`, `CL-REACT-019` | Which checks must pass before an update is called complete | IT | ⚠️ Each check is a dependency that fails silently rather than loudly |
| `CL-OTA-009`, `CL-OTA-021` | Will one board pilot for a full day; should failed devices auto re-pin | Administration / IT | The only blast-radius control, and silent fleet drift without it |
| `CL-OTA-010`, `CL-OTA-012` | Does the kiosk account have elevation; is automatic deployment permitted | IT | ⛔ **If there is no elevation, every update becomes a manual IT visit and the cost model changes entirely** |
| `CL-OTA-011`, `CL-OTA-020` | Can config be signed data; who promotes config from staging | IT | Whether routine changes need the privileged install path at all |
| `CL-OTA-013`, `CL-OTA-022` | Behaviour when the network drops mid-update; should a reimaged board re-enrol automatically | IT | ⛔ A partially applied update is the standard bricking path |
| `CL-OTA-014` | How long update infrastructure may be unreachable before the platform must stay fully usable | IT | ⚠️ The local-first claim is only meaningful against a real outage duration |
| `CL-OTA-015`, `CL-OTA-016`, `CL-SEC-019` | Retention and custody of the deployment record; must it be tamper-evident | IT / administration | ⛔ Without it, a log the deployer can edit is not evidence |
| `CL-OTA-017` | Who would notice and block a rogue build | IT | ⛔ P0. Detection needs a named person, not a process |
| `CL-OTA-018`, `CL-OPS-020` | Safe change windows; is the platform frozen during exam weeks | Administration | ⚠️ An update colliding with an exam is an incident regardless of technical correctness |
| `CL-OTA-019` | Is a timetable correction an urgent fix that bypasses the release window | Timetable office | ⛔ Data and code must have separate promotion paths, or urgent fixes get delayed |
| `CL-OPS-018` | Which certificates or tokens the school operates, and who tracks expiry | IT | ⛔ P0. An expired certificate takes every display down at once |
| `CL-ADMIN-004`, `CL-ADMIN-019` | Who may deploy; must vendor changes be approved like any other | IT / administration | ⚠️ Deployment rights should not follow from ordinary configuration rights |

---

## The invariants

Five rules survive every open question above. If the answers arrive and they are
inconvenient, these five do not move.

> **1. "OTA update" is not one thing.** Seven channels, seven blast radii, seven
> rollback stories. Robot firmware is an eighth thing entirely, and CloudLight is
> not its delivery path.

> **2. An unsigned build is refused, whatever its origin.** On an untrusted LAN,
> signature verification is not defence in depth. It is the control.

> **3. Rollback is designed in and tested, or it does not exist.** A mechanism that
> has never been exercised is a hope, not a recovery plan.

> **4. Data and code have separate promotion paths.** An urgent timetable
> correction is never queued behind a software-release approval.

> **5. The deployment log is append-only.** Whoever administers updates also has
> database access — so they can change their own history unless they cannot.

---

## Sources

**Types:** `SPECIFICATION` (published open standard) · `INTERNAL` (this
repository). **Access date for all external sources: 2026-10-11.**

| # | Title | URL | Type | What it supports |
|---|---|---|---|---|
| 1 | **The Update Framework (TUF) — Specification** | `https://theupdateframework.io/specification/latest/` | SPECIFICATION | §5.1 — TUF is a **published specification**, not a product; it defines **root, snapshot, targets and timestamp** roles with **separate keys**, and requires **expiry** on metadata, so a frozen or stale repository state becomes invalid on its own. Presented as the reference design, **not** as a decision to adopt |
| 2 | **The Update Framework — About / Overview** | `https://theupdateframework.io/about/` | SPECIFICATION | §5 — TUF's stated threat model, which **names** the rollback, freeze, fast-forward and mix-and-match attacks the specification exists to resist, confirming that "check the signature" alone is not the design |
| 3 | **`./network_and_hosting.md`** | *(this repository)* | INTERNAL | §3, §9, §12, §13 — a school LAN is not trusted merely because it is internal; **certificate expiry as a simultaneous school-wide failure**; §7.3 offline-capability vs patchability; §9.5 UPS and power-cut behaviour; §9.6 "a backup that has never been restored is not a backup" |
| 4 | **`./shana_integration.md`** | *(this repository)* | INTERNAL | §2.1 — SHANA-side ownership boundary; **safety never transfers to CloudLight**; CloudLight operates fully when SHANA is offline |
| 5 | **`./classroom_dashboard.md`** | *(this repository)* | INTERNAL | §8.1 — **silent partial failure is indistinguishable from "nothing is scheduled"**, which on a wall in front of children is a confident falsehood; the read-only, unauthenticated display property |
| 6 | **`../shana/architecture/architecture.md`** and **`../architecture/architecture_decisions/`** | *(this repository)* | INTERNAL | §2 — the AI/deterministic boundary; **ADR-001: no AI model may reach the robot**, which is why channel 6 carries a different risk class from the rest |
| 7 | **`../../questionnaires/Additional_CloudLight_Questionnaire.md`** | *(this repository)* | INTERNAL | Every `CL-OTA-*` identifier in §16, plus `CL-SEC-013`, `CL-REACT-010`, `CL-REACT-019`, `CL-GIT-018`, `CL-OPS-016`, `CL-OPS-018`, `CL-OPS-020`, `CL-ADMIN-004`, `CL-ADMIN-019`, `CL-ADMIN-020`, with priority, respondent, evidence requirement and `UNANSWERED` status |

⚠️ **Note on Windows platform sources.** §10 discusses kiosk elevation, scheduled
tasks and Group Policy constraints. ⚠️ **No vendor source is cited for those
claims**, because none was verified during this pass, and an unverified citation is
worse than an absent one. `CL-OTA-010` and `CL-OTA-012` are the questions that
settle them, and they must be answered by school IT rather than inferred here.

---

⚠️ **Standing note.** Every external source above is a **general published
specification or vendor document**. **None of it describes this school.** No build
was produced, signed, published or deployed; no release key exists; no smartboard
has been enrolled; no update agent has been installed. The questions in §16 remain
questions until school IT and school administration answer them in writing.

**Tests actually executed to date: zero.**