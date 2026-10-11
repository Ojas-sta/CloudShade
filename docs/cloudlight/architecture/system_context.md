# CloudLight — System Context

**Phase 0 · Design only · 2026-10-11**

A C4-style **context view**: what sits outside CloudLight, what CloudLight needs
from each, and what happens when each is absent.

> **⚠️ Nothing in this diagram exists.** No host has been provisioned, no display
> configured, no storage reached, no identity provider contacted, no update source
> used, no robot integrated. Every box marked ❓ is a question, and most of them
> are questions for a person rather than for an engineer.

Deeper levels are deliberately **not** in this document. Module internals are in
[`../architecture.md`](../architecture.md); deployment topology is in
[`../deployment_options.md`](../deployment_options.md) §4.2.

---

## 1. The system in one picture

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│  SCHOOL — physical and organisational environment                                   │
│                                                                                    │
│   ┌───────────┐   ┌───────────┐   ┌───────────┐   ┌───────────┐   ┌───────────┐     │
│   │ TEACHERS  │   │ TIMETABLE │   │   ADMIN / │   │    IT     │   │ STUDENTS  │     │
│   │           │   │  CLERK    │   │ PRINCIPAL │   │           │   │ (passive) │     │
│   └─────┬─────┘   └─────┬─────┘   └─────┬─────┘   └─────┬─────┘   └─────┬─────┘     │
│         │               │               │               │               │           │
└─────────┼───────────────┼───────────────┼───────────────┼───────────────┼───────────┘
          │  view         │  propose &     │  approve &    │  approve &     │ see
          │  (no login)   │  approve       │  authorise     │  operate       │ (PUBLIC only)
          │               │                                │                │           │
    ══════╪═══════════════╪════════════════╪════════════════╪════════════════╪════════════
    TRUST BOUNDARY — everything above is outside CloudLight's control
    ══════╪═══════════════╪════════════════╪════════════════╪════════════════╪════════════
          │               │                │               │                │
          ▼               ▼                ▼               ▼                │
┌──────────────────────────────────────────────────────────────────────────────────┐
│                                                                                    │
│  ┌──────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                              │  │
│  │                        C L O U D L I G H T                                    │  │
│  │                                                                              │  │
│  │   classroom display · timetable · substitution · resources · search          │  │
│  │   authentication · administration · audit · update verification             │  │
│  │   SHANA adapter (request + status ONLY — never a command)                    │  │
│  │                                                                              │  │
│  │   ⚠️ ONE PROCESS · ONE DATABASE · ONE HOST · 0 lines of code written          │  │
│  │                                                                              │  │
│  └──────────────────────────────────────────────────────────────────────────────┘  │
│         ▲            ▲                ▲               ▲               ▲            │
│         │            │                │               │               │            │
└─────────┼────────────┼────────────────┼───────────────┼───────────────┼────────────┘
          │            │                │               │               │
   ═══════╪════════════╪════════════════╪═══════════════╪═══════════════╪══════════════
   TRUST BOUNDARY — school network is NOT trusted (C6)
   ═══════╪════════════╪════════════════╪═══════════════╪═══════════════╪══════════════
          │            │                │               │               │
          ▼            ▼                ▼               ▼               ▼
┌──────────────────┐ ┌──────────────┐ ┌──────────────────┐ ┌──────────────┐ ┌──────────┐
│  SCHOOL LAN      │ │     NAS /    │ │ IDENTITY         │ │ UPDATE       │ │  SHANA   │
│  switches, APs,  │ │    SHARES    │ │ PROVIDER         │ │ SOURCE       │ │ (robot)  │
│  VLANs, DHCP,    │ │              │ │                  │ │              │ │          │
│  DNS, firewall   │ │ ❓ EXISTS?   │ │ ❓ EXISTS?       │ │ ❓ PERMITTED? │ │ ❓ IFACE? │
│  ❓ isolation?   │ │ CL-NAS-001   │ │ CL-AUTH-001/002  │ │ CL-OTA-012   │ │CL-SHANA-1│
└──────────────────┘ └──────────────┘ └──────────────────┘ └──────────────┘ └──────────┘

  ═════════════════════════════════════════════════════════════════════════════════
  EXTERNAL SYSTEMS CLOUDLIGHT EXPLICITLY DOES **NOT** DEPEND ON
  ═════════════════════════════════════════════════════════════════════════════════
    ✗ PUBLIC INTERNET          — core operation does not require it (C1)
    ✗ ANY EXTERNAL AI SERVICE  — prohibited without written authorisation (C2)
    ✗ CLOUD DATABASE / SaaS    — a scope change, not a fallback
    ✗ CLOUD VM AS PRIMARY      — contingency only, never primary
    ✗ ROBOT FIRMWARE DELIVERY  — CloudLight must never be the path (N-18)
    ✗ ANY STUDENT DEVICE       — students are viewers, never users
    ✗ SSO / CLOUD IDENTITY     — only if the school already runs it
    ✗ ANY THIRD-PARTY SAAS     — nothing in a lesson may depend on a vendor's uptime
```

---

## 2. People — what CloudLight needs, assumes, and does without

| Actor | Needs from CloudLight | CloudLight assumes | If absent |
|---|---|---|---|
| **Teacher** | The lesson, times, room, and who is actually covering it. ⚠️ **Without a login prompt at lesson start** — that gets bypassed or abandoned | That a teacher reads a wall rather than a phone, and that the screen is more useful than asking a colleague | ⛔ **The core user is absent.** Nothing else matters |
| **Timetable clerk** | One place to enter and approve; no whiteboard alongside; no re-keying | That a recorded decision is *faster* than a phone call and a marker | ⚠️ **The system is routed around.** A second, less authoritative source — the worst outcome |
| **Administration** | Approval authority; a record of who approved what; screens with no personal data | That a named person will actually be the approver, and that they will be there | ⛔ **Nothing may lawfully be stored.** `CL-GEN-001` gates everything |
| **IT generalist** | One process, one database, restore instructions a non-specialist can follow, a named person to restart the host | ⚠️ **One person, not a team.** No on-call rota, no second person who could read the code | ⛔ **Nobody will maintain it.** `CL-OPS-022` — the one answer that should stop the project rather than shrink it |
| **Students** | Nothing. They are viewers | ⚠️ **They are never users.** No login, no personalisation, no tracking | N/A — they are not a stakeholder to be designed for |

⚠️ **The timetable clerk is the load-bearing user.** Everything upstream and
downstream of the approval gate exists to make *their* decision reach a screen
without transcription error. A system that adds review burden to them without
removing transcription will be bypassed within a fortnight.

---

## 3. Systems — what CloudLight needs, assumes, and does without

### 3.1 School LAN — switches, APs, VLANs, DHCP, DNS, firewall

| | |
|---|---|
| **Needs** | Connectivity between displays and host; name resolution; a stable address; a firewall rule permitting one port from one segment; a certificate the displays trust |
| **Assumes** | ❓ That client/AP isolation is **not** enabled — `CL-NET-008`. ❓ That cross-subnet access is permitted — `CL-NET-003`. ❓ That a DNS name and DHCP reservation can be created |
| **If absent** | ⛔ **Every display is blank — silently.** Boards associate, show full signal, and never load. ⚠️ This is the most likely quiet failure in the project, and the reason it is a written confirmation rather than something we test ourselves (`C13`) |
| **Constraint** | The LAN is **not trusted** (`C6`). Releases are signed, admin surfaces are separated, authorisation is enforced server-side regardless of network location |

### 3.2 NAS / shared file server

| | |
|---|---|
| **Needs** | A reachable share; a dedicated **read-only service account**; an explicit approved folder list; an ACL export; a directory listing and a size measurement |
| **Assumes** | ❓ That one exists **at all** — `CL-NAS-001`. ⚠️ **A school may keep every teaching file on laptops, pen drives and personal cloud drives**, and *"no"* is a valid answer |
| **If absent** | ⛔ **Resource browsing, resource search and the assistant do not exist.** ✅ **The timetable and substitution product is unaffected.** This is a shape change, not a project-ending one |
| **Constraint** | ⛔ CloudLight holds no parallel permission model. It defers to the share's ACLs, honouring DENY over ALLOW, and re-verifying at open time |

### 3.3 Identity provider — school directory via OIDC/SAML

| | |
|---|---|
| **Needs** | One list of teachers with stable identifiers; written permission to authenticate against it; a group-membership source for query-time permission filtering |
| **Assumes** | ❓ That a directory exists — `CL-AUTH-001`. ❓ What it is — `CL-AUTH-002`. ❓ That SSO is permitted — `CL-AUTH-003` |
| **If absent** | ⚠️ **CloudLight becomes the system of record for teacher identity**, and therefore owns provisioning *and offboarding*. HR and CloudLight diverge silently; annual provisioning becomes a security event |
| **Constraint** | ⛔ A shared classroom code may **never** substitute for this. It may authorise one action or act as a second factor — never identify a person |

See [`../integrations/identity_provider.md`](../integrations/identity_provider.md).

### 3.4 Update source

| | |
|---|---|
| **Needs** | A place releases come from; a signing key with a custody policy; a permitted rollout method |
| **Assumes** | ❓ That automatic unattended deployment is permitted — `CL-OTA-012`. ❓ Where the signing key lives — `CL-OTA-005` |
| **If absent** | ✅ **The platform remains fully usable.** Manual, logged, human-triggered deploys are the conservative default. ⚠️ But offline-first is a resilience property, not a security posture — a permanently offline system cannot be patched, and that tension is real |
| **Constraint** | Eight distinct update channels, each signed, each staged, each rollback-able. ⛔ **Robot firmware is not one of them** |

### 3.5 SHANA — the robotics subsystem

| | |
|---|---|
| **Needs** | High-level delivery intent only: *what*, *to where*, *by when*. And status back. Nothing else |
| **Assumes** | ❓ That an interface exists and is permitted — `CL-SHANA-001`. ❓ How the robot verifies a request came from this school's CloudLight — `CL-SHANA-015` |
| **If absent** | ✅ **Entirely unaffected.** The robot panel shows unavailable-with-age, or is hidden. ⚠️ **This is the single most important architectural property in the project**: a robot failure must never blank a classroom screen |
| **Constraint** | ⛔ **No motor, path, speed, actuator or localisation command may cross this boundary.** SHANA validates safety independently. The boundary is an *absence of capability*, not a policy — ADR-014 |

---

## 4. Trust boundaries

⚠️ **Every serious failure mode lives on a boundary line, not inside a component.**

| # | Boundary | What crosses it | The control |
|---|---|---|---|
| **1** | **School environment → CloudLight** | Approvals, refusals, operational decisions | ⚠️ **The project team may not decide for the school.** A decision requiring school authority stays `OPEN` until answered in writing |
| **2** | **CloudLight ⇄ school LAN** | HTTPS requests from displays | The LAN is **not trusted**. Short-lived session cookies; `httpOnly`, `Secure`, `SameSite=Strict` |
| **3** | **CloudLight → NAS** | SMB reads | ⛔ Read-only **dedicated service account**, never a staff credential. Permission re-verified at open time |
| **4** | **Display segment ⇄ administration** | Admin sessions | ⚠️ **The highest-value boundary.** The administration surface must be unreachable from a student-facing segment; if the network cannot be changed, compensate in the application |
| **5** | **CloudLight → external service** | Anything, if ever permitted | ⛔ **Prohibited by default.** Requires written school authorisation for a *named* service. No outbound call in any code path, including error handlers |
| **6** | **CloudLight → update source** | Release artefacts | Signature verified **on the consuming device**, not at the distribution point. Expiry on metadata |
| **7** | **CloudLight ⇄ SHANA** | Request and status | Mutually authenticated; idempotency keys; ⛔ **no motor-command path exists** |
| **8** | **Signed-out display ⇄ staff surfaces** | Anything shown in public | ⛔ **PUBLIC-class content only.** No staff surface, no search, no assistant, no settings without authentication |

⚠️ **Boundary 5 is the one that must be expressed in code rather than in policy.**
FR-AI-02 requires the default to be fully local inference with no outbound call
in any code path — *including error handlers*. A default-on with a kill switch is a
different and weaker thing.

⚠️ **Boundary 4 is the one most likely to be unavailable.** On a flat network, any
compromised display is one hop from the control plane. If segmentation cannot be
changed, the application must refuse the connection itself — and that refusal must
be tested, not assumed.

---

## 5. The systems CloudLight deliberately does not depend on

⚠️ **This list is a design statement, not a limitation.** Each item is a way the
product could have been easier to build and worse to run.

| Not a dependency | Why | What it forecloses |
|---|---|---|
| **Public internet** | Core operation must survive a week with the uplink unplugged | Any design where a lesson depends on an external service |
| **Any external AI service** | ⛔ Prohibited without written authorisation for a named service | "We will try it and see". The default is local-only |
| **A cloud database or SaaS** | A scope change, not a fallback | Managed availability someone else can revoke |
| **A cloud VM as primary hosting** | Contingency only | Local-first, and data stays on school premises |
| **Student devices** | Students are viewers | ⚠️ Any per-student surface at all |
| **Robot firmware delivery** | Higher-risk physical activity, separately governed | Conflating a bundle refresh with a flash — a category error with safety consequences |
| **Third-party uptime** | Nothing in a lesson may depend on a vendor | Any "if the vendor is down, no timetable" path |
| **A photographed board as a record** | Visibility is not authority | Any ingestion path where a misread publishes |

⚠️ **The item most likely to be quietly reintroduced is the first.** Once a demo
works, "let's just use a hosted thing for staging" is a two-minute decision that
turns a local-first product into a cloud product with a school LAN attached. That
is a scope change and needs an explicit decision, not a shortcut.

---

## 6. Related documents

| Document | Covers |
|---|---|
| [`../architecture.md`](../architecture.md) | Module internals, data flow, failure behaviour — level below this one |
| [`../deployment_options.md`](../deployment_options.md) | The baseline topology and what it assumes |
| [`../network_and_hosting.md`](../network_and_hosting.md) | Network reachability, segmentation, backup and power |
| [`../shana_integration.md`](../shana_integration.md) | The CloudLight/SHANA boundary in detail |
| [`../integrations/identity_provider.md`](../integrations/identity_provider.md) | The identity models, compared |
| [`../integrations/timetable_sources.md`](../integrations/timetable_sources.md) | Candidate timetable inputs |
| [`../data_model.md`](data_model.md) | The entities the system must model — design only |
| [`../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md`](../../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md) | The boundary, stated as an absence of capability |