# CloudLight — Overview

**Phase 0 · Documentation only · 2026-10-11**

> **⚠️ NOTHING IN THIS DIRECTORY IS BUILT, DEPLOYED OR TESTED.**
> No smartboard has been configured. No school network has been probed. No NAS has
> been accessed. No robot has been operated. No AI service has been contacted. No
> school credential exists anywhere in this repository.
>
> This directory contains **research, analysis and questions**. That is deliberate.

---

## 1. What CloudLight is proposed to be

An adaptable, **school-local** digital platform that runs on Windows classroom
smartboards and authorised school devices connected to the school's own network.

Its proposed capabilities are:

| # | Capability | First-release status |
|---|---|---|
| 1 | Classroom information displays | **In scope** |
| 2 | Current and upcoming class information | **In scope** |
| 3 | Class timings and countdowns | **In scope** |
| 4 | Teacher and substitution information | **In scope** |
| 5 | Timetable and substitution-sheet ingestion | **In scope** |
| 6 | AI/ML-assisted document conversion to structured data | **Conditional** — assistive only |
| 7 | Teacher resource browsing | **Deferred** — gated on shared storage existing |
| 8 | One-click access to authorised NAS folders | **Deferred** — same gate |
| 9 | Search across authorised classroom resources | **Deferred** — same gate |
| 10 | AI assistant for approved information | **Deferred** — build last, if ever |
| 11 | Teacher authentication and authorisation | **In scope** — but see §4 |
| 12 | Requests to SHANA for approved deliveries | **Deferred** — build last |
| 13 | Over-the-air update mechanism | **Deferred** — after first pilot |
| 14 | React-based frontend | **Accepted direction**, constraints unconfirmed |
| 15 | Local server on an authorised school network | **In scope** |
| 16 | Extensible modules for future workflows | **Architecture goal, not a feature** |

⚠️ **This table is a proposal, not a plan.** Nine of sixteen capabilities are
deferred, and four of those are deferred because **a precondition has not been
confirmed to exist at all** — not because they are hard.

---

## 2. Relationship to SHANA

> **CloudLight is the platform. SHANA is one subsystem within it.**
> **Both live in this one repository. Neither gets a separate one.**

```
┌──────────────────────────────────────────────────────────────────────┐
│  CLOUDLIGHT — the school-facing platform                             │
│  classroom display · timetable · substitution · resources · search   │
│  assistant · authentication · administration · updates              │
│                                                                      │
│   ┌────────────────────────────────────────────────────────────┐     │
│   │  SHANA — the robotics subsystem                            │     │
│   │  navigation · localisation · route safety · battery        │     │
│   │  hazards · safe stopping · delivery execution              │     │
│   │                                                            │     │
│   │  ⚠️ SHANA validates every safety decision independently.   │     │
│   │  ⚠️ CloudLight never sends a motor command.                 │     │
│   └────────────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────────────┘
```

Two rules govern the boundary, and both are inherited from SHANA's own accepted
architecture decisions rather than invented here:

1. **CloudLight may request a delivery. It may never command the robot.**
2. **CloudLight must operate fully when SHANA is offline.**

The second rule is the one that most affects architecture. It means the dependency
arrow points from CloudLight to SHANA, never the reverse, and it means **a robot
failure must never blank a classroom screen.** See
[`shana_integration.md`](shana_integration.md).

⚠️ **This directory does not modify SHANA.** SHANA's documentation, decisions and
safety case remain exactly as they were. Where CloudLight depends on a SHANA
convention, it cites the existing document rather than restating or relaxing it.

---

## 3. The shape of the system

```
   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
   │ Smartboard A │  │ Smartboard B │  │ Smartboard C │   read-only display
   └───────┬──────┘  └──────┬───────┘  └──────┬───────┘
           │                │                 │  HTTPS, school LAN
           └────────────────┼─────────────────┘
                            ▼
   ┌──────────────────────────────────────────────────────────────┐
   │  CLOUDLIGHT BACKEND  (one always-on school host)             │
   │                                                              │
   │  display · timetable · substitution · resources · search     │
   │  AI assistant · auth · admin · audit · update verification   │
   │                                                              │
   │            ┌──────────────┐        ┌──────────────┐         │
   │            │   Database   │        │ Search index │         │
   │            └──────────────┘        └──────────────┘         │
   └───────────┬──────────────────────────────┬───────────────────┘
               │ SMB (read-only svc account)  │ optional, signed request
               ▼                              ▼
      ┌─────────────────┐          ┌──────────────────┐
      │  NAS / shares   │          │  SHANA interface │
      │  teaching files │          │  request + status│
      └─────────────────┘          └──────────────────┘
```

Full detail in [`architecture.md`](architecture.md).

---

## 4. The three findings that shape everything

### 4.1 An access point cannot host a platform

The brief states that all smartboards and computers can reach the same school LAN
or access point. **That assumption is not verified, and even if true it does not
establish what is needed.**

An access point is a **Layer-2 bridge**: it forwards Ethernet frames. It does not
run a web server, a database, or an application. A router's "port forwarding"
only forwards to a device that already exists.

> **"All smartboards can reach the same access point" establishes at most a shared
> Layer-2 segment. It does not establish that a backend can exist, and it does
> not establish that one device can reach another.**

Three separate things are being conflated and must each be answered separately:

| | What it is | What proves it exists |
|---|---|---|
| **Network infrastructure** | Switches, APs, VLANs, DHCP, DNS, firewall | Connectivity *between devices* |
| **The host** | An always-on machine running the backend | Someone is permitted to run one |
| **The display** | A Windows smartboard rendering the frontend | It can reach *the host* |

⚠️ The first being true implies neither of the other two. See
[`network_and_hosting.md`](network_and_hosting.md).

### 4.2 A six-digit code is not an identity

The brief proposes a teacher using a secret passcode or six-digit code to request
a SHANA delivery.

> **A shared six-digit code proves only that someone in the room knows six
> digits. It identifies nobody.**

A million possibilities is guessable by a student with a phone during a free
period, absent attempt limits. A code observed by anyone who walks past the board
is captured. And a classroom display is **shared and frequently unattended**.

Such a code may only ever be **(a) an authorisation for one specific action**, or
**(b) a second factor behind a real first factor**. It may never stand in for a
named teacher. Individual teacher accounts are required for anything
consequential. See [`authentication_and_authorisation.md`](authentication_and_authorisation.md).

### 4.3 A photograph of a board is not a record

The brief contemplates ingesting photographs of timetable or substitution boards.

> **A document is authoritative because a person with authority approved it —
> not because it was photographed.**

An image is an **input to a proposal**. A human must confirm it. No OCR result
publishes at any confidence level. **Only a human-approved record reaches a
classroom display**, and an unapproved substitution is never displayed as a
confirmed teacher. See
[`timetable_and_substitutions.md`](timetable_and_substitutions.md).

---

## 5. What this directory contains

| File | Purpose |
|---|---|
| [`overview.md`](overview.md) | This file — what CloudLight is and its relationship to SHANA |
| [`requirements.md`](requirements.md) | Requirements by class: confirmed, proposed, pending answer, out of scope |
| [`architecture.md`](architecture.md) | Modules, boundaries, data flow, failure behaviour, and the modular-monolith argument |
| [`classroom_dashboard.md`](classroom_dashboard.md) | Display content hierarchy, countdowns, the scheduled-vs-confirmed contract, degraded states |
| [`timetable_and_substitutions.md`](timetable_and_substitutions.md) | Ingestion pipeline, parsing tiers, schema validation, corrections and rollback |
| [`windows_startup.md`](windows_startup.md) | Windows edition gating, kiosk options, autostart mechanisms, the recovery matrix |
| [`network_and_hosting.md`](network_and_hosting.md) | What an AP is, hosting options compared, addressing, segmentation, reachability |
| [`resource_browser.md`](resource_browser.md) | NAS reality check, protocols, permissions, the resource directory |
| [`resource_search.md`](resource_search.md) | Search tiers, permission-aware retrieval, ranking, stale indexes |
| [`ai_assistant.md`](ai_assistant.md) | Retrieval-augmented answers, local vs external inference, prompt injection, refusal |
| [`authentication_and_authorisation.md`](authentication_and_authorisation.md) | Identity, MFA, the six-digit code rules, unattended displays, sessions |
| [`ota_updates.md`](ota_updates.md) | The eight distinct update channels, signed releases, rollback, staged rollout |
| [`security_and_privacy.md`](security_and_privacy.md) | Threat model, data classification, logging, incident response, accepted risks |
| [`shana_integration.md`](shana_integration.md) | The integration boundary, request contract, failure behaviour in both directions |
| [`deployment_options.md`](deployment_options.md) | Hosting and client options compared, baseline deployment, progression |
| [`feasibility_report.md`](feasibility_report.md) | The conclusions, each classified by evidence strength |
| [`risk_register.md`](risk_register.md) | Risks by domain with rating, mitigation and residual risk |
| [`open_questions_register.md`](open_questions_register.md) | The blocking questions, sequenced by who can answer them |
| [`decision_log.md`](decision_log.md) | Decisions made, provisional defaults, open decisions, rejected options |
| [`first_release_scope.md`](first_release_scope.md) | The smallest useful first release, and what is postponed |
| [`next_actions.md`](next_actions.md) | The next ten actions in priority order |
| [`sources.md`](sources.md) | CloudLight bibliography, sources deliberately not used, unsourced judgement |

The questionnaire that gathers the answers lives outside this directory:
`../../questionnaires/Additional_CloudLight_Questionnaire.md`.

---

## 6. Where to start

| If you are… | Read |
|---|---|
| **The Principal** | This file → [`requirements.md`](requirements.md) → `CL-GEN`, `CL-DEC` in the questionnaire |
| **School IT** | [`network_and_hosting.md`](network_and_hosting.md) → [`windows_startup.md`](windows_startup.md) |
| **Timetable / administration** | [`timetable_and_substitutions.md`](timetable_and_substitutions.md) → [`classroom_dashboard.md`](classroom_dashboard.md) |
| **Teachers** | [`classroom_dashboard.md`](classroom_dashboard.md) → [`resource_search.md`](resource_search.md) |
| **The robot custodian** | [`shana_integration.md`](shana_integration.md) |
| **Evaluating feasibility** | [`feasibility_report.md`](feasibility_report.md) → [`open_questions_register.md`](open_questions_register.md) |
| **A new contributor** | This file → [`decision_log.md`](decision_log.md) → `CONTRIBUTING.md` |

---

## 7. Ground rules this project follows

1. **No capability is described as operational before it is demonstrated.**
2. **No AI may publish timetable data, approve substitutions, grant permissions,
   or issue robot commands.** It may only propose; deterministic code decides.
3. **Deterministic parsing precedes AI.** AI never invents a missing value.
4. **No AI output may ever reach the robot.**
5. **The classroom display never shows an authentication secret, and never
   displays an unapproved substitution as a confirmed teacher.**
6. **Stale data is shown only with a visible marker.** A blank screen is better
   than a confident wrong answer.
7. **Search never returns a file the user cannot open**, re-verified at open time.
8. **No external AI service receives school data** without written school
   authorisation.
9. **A school LAN is not trusted.** Updates are signed; administration surfaces
   are separated; secrets live nowhere near Git, URLs, logs or browser storage.
10. **CloudLight never controls robot motors**, and SHANA's safety authority is
    never delegated.
11. **A gate is never marked verified without evidence.**

---

## 8. Licence

Not yet determined. See [`LICENSE_REVIEW.md`](../../LICENSE_REVIEW.md) — this
remains an open decision and no licence has been chosen.