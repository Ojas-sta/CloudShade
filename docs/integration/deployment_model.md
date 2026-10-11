# Deployment Model

**Owner:** Agent 3 (Integration Lead)
**Status:** 🔴 `DESIGN_ONLY` — **nothing is deployed. No host exists. Nothing is
permitted.**
**Last updated:** 2026-10-11

---

## 🔴 The deployment question is not technical

🔴 **It has never been asked of the school.**

> **May a persistent local service run on school premises at all?**

`CL-SRV-001`. Roughly ten minutes with the Principal. 🔴 **It is Gate 1 for
CloudLight and nothing can be designed around until it is answered.**

Everything below is a *conditional*: "if permitted, then…". 🔴 **It is not a
deployment plan, because there is nothing yet to deploy and no permission to
deploy it.**

---

## 1. Deployment status

| Thing | Status |
|---|---|
| CloudLight host | 🔴 **Does not exist.** `CL-SRV-001` — not even permitted |
| CloudLight service | 🔴 Does not exist |
| Database | 🔴 Does not exist. No schema, no engine chosen |
| Search index | 🔴 Does not exist |
| Classroom displays configured | **0** |
| NAS accessed | **0** — 🔴 and whether one exists is unknown |
| Robot operated | **0** |
| AI service contacted | **0** |
| CI / monitoring / alerting | 🔴 Do not exist |
| 🔴 **Systems deployed** | **0** |
| 🔴 **Systems demonstrated** | **0** |

---

## 2. Unanswered questions that block any deployment

| # | Question | ID | Owner | If unanswered |
|---|---|---|---|---|
| 1 | 🔴 May a persistent local service run at all? | `CL-SRV-001` | Principal + IT | 🔴 **The local-first premise fails** |
| 2 | Can classroom devices reach it? | `CL-NET-003` | IT | 🔴 The project cannot proceed — 🔴 **and may fail silently** |
| 3 | Is client/AP isolation enabled? | `CL-NET-008` | IT | *"The most likely quiet failure in the whole project"* |
| 4 | Is a dedicated robot segment permitted? | SH-NET | IT | Robot link and classroom LAN share a broadcast domain |
| 5 | Is a Linux host with a spare Ethernet port available? | SH-NET | AI Lab / IT | No wired robot link |
| 6 | Does a NAS or shared file server exist **at all**? | `CL-NAS-001` | IT + Library | 🔴 The entire resource track closes permanently |
| 7 | Does a staff identity directory exist? | `CL-AUTH-001` | IT | 🔴 No authentication source |
| 8 | What Windows edition is on the boards? | `CL-WIN-001` | IT | Kiosk lockdown may be impossible |
| 9 | Do kiosk accounts have administrator rights? | `CL-WIN-003` | IT | Kiosk design blocked |

---

## 3. 🔴 A verified fact worth carrying forward

From CloudLight's research, `VERIFIED` against Microsoft and standard sources:

> **An access point is a Layer-2 bridge and cannot host an application.**

> *"All smartboards can reach the same access point" establishes at most a shared
> network segment — **it does not establish that a backend can exist, nor that
> one device can reach another.**

⚠️ This is exactly the kind of assumption that produces a project which appears
to work in a meeting and fails silently in a classroom.

🔴 **Corroborating constraint:** a Windows Service **cannot display UI**, so it
cannot be the kiosk launcher. Kiosk mode needs Assigned Access or Shell
Launcher — 🔴 **both edition-dependent, and they cannot be combined.**

---

## 4. Conditional topology

🔴 **Hypothetical. Shown only so the constraints are visible.**

```
                    SCHOOL LAN  (client isolation status UNKNOWN)
   ┌────────────────────────────────────────────────────────────────┐
   │                                                                │
   │   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   │
   │   │ Display  │   │ Display  │   │ Display  │   │ Display  │   │
   │   │ Board 1  │   │ Board 2  │   │ Board 3  │   │ Board N  │   │
   │   │ Windows? │   │ Windows? │   │ Windows? │   │ Windows? │   │
   │   │ UNKNOWN  │   │ UNKNOWN  │   │ UNKNOWN  │   │ UNKNOWN  │   │
   │   └────┬─────┘   └────┬─────┘   └────┬─────┘   └────┬─────┘   │
   │        │              │              │              │          │
   └────────┼──────────────┼──────────────┼──────────────┼──────────┘
            │              │              │              │
            └──────────────┴──────┬───────┴──────────────┘
                                  │  🔴 REACHABILITY UNCONFIRMED
                                  │     (CL-NET-003, CL-NET-008)
                                  ▼
   ┌────────────────────────────────────────────────────────────┐
   │  CLOUDLIGHT HOST                        🔴 PERMISSION UNKNOWN│
   │  modular monolith (ADR-013, provisional)                   │
   │  ├ timetable · substitutions · resources · auth            │
   │  └ ✋✋✋ NO ROBOT CONTROL CAPABILITY ✋✋✋                    │
   └───────────────────────────┬────────────────────────────────┘
                               │  request only · mutual auth REQUIRED
                               │  🔴 separate segment (permit unknown)
                               ▼
   ┌────────────────────────────────────────────────────────────┐
   │  SHANA (robot)                        🔴 SEPARATE ETHERNET   │
   │  🔴 Go2 documentation states WIRED development connections │
   └────────────────────────────────────────────────────────────┘
```

⚠️ **`N` is unknown.** No board has been counted, no Windows edition verified.

---

## 5. 🔴 Segment separation is a safety requirement

From `docs/cloudlight/security_and_privacy.md` §7.1:

> The robot link is a **separate wired Ethernet segment** that **must not share
> a broadcast domain with the classroom LAN**.

⚠️ **Not a hardening preference.** A robot segment on the classroom LAN means
student devices share a broadcast domain with a system that moves physical
objects through a building.

🔴 **Blocked on IT** (`SH-NET`). 🔴 **No network topology detail may be
committed** — `.gitignore` excludes it, correctly.

---

## 6. Deployment sequence (conditional)

🔴 **Step 0 has not started.** Steps 1–2 require answers the school has not
given.

| # | Step | Gate | Owner | Status |
|---|---|---|---|---|
| 0 | 🔴 Answer `CL-SRV-001` — is a local service permitted? | **Gate 1** | Principal | 🔴 `BLOCKED` |
| 1 | Confirm classroom devices can reach the host | Gate 2 | IT | 🔴 `BLOCKED` |
| 2 | Confirm identity source | Gate 3 | Administration + IT | 🔴 `BLOCKED` |
| 3 | Confirm substitution approval path | Gate 4 | Timetable | 🔴 `BLOCKED` |
| 4 | One authorised on-site test: `winver` on a non-critical board | — | IT | 🔴 `BLOCKED` |
| 5 | Deploy to a **single** non-critical board | — | — | ⛔ Not reachable |
| 6 | Add displays incrementally | — | — | ⛔ Not reachable |
| 7 | 🔴 Enable SHANA integration **last** | — | — | ⛔ Not reachable |

⚠️ **The integration is last deliberately.** CloudLight's own recommendation
(`shana_integration.md` §12):

> **Build CloudLight without the SHANA integration, and add it last.**

Because a SHANA integration that is unreachable, rejected or refused would take
every classroom screen down with it if the dependency ran the other way. 🔴 **The
dependency arrow must point CloudLight → SHANA, and it must be optional.**

---

## 7. Offline and degraded behaviour

⚠️ 🔴 **Not one of these has been implemented.** They are requirements, and they
are requirements the project has reasoned about unusually well.

| Failure | Required behaviour | Status |
|---|---|---|
| Network down | Display shows last snapshot **with a visible staleness marker** | 📄 `DESIGN_ONLY` |
| No snapshot ever received | Show an explicit unavailable state, not an empty screen | 📄 |
| CloudLight unreachable from displays | 🔴 **A classroom screen must never blank because the robot failed** | 📄 |
| SHANA unreachable | CloudLight fully usable; robot panel neutral with last-known age | 📄 |
| Quarantined document | Model the degraded state explicitly | 📄 |
| Stale data | 🔴 Shown only with a **visible marker**. *A blank screen is better than a confidently wrong answer* | 📄 |

⚠️ **The last line is a genuinely good design decision** and should be preserved
through any implementation pressure.

---

## 8. 🔴 What the deployment model needs before it becomes a plan

| # | Requirement | Cost | Owner |
|---|---|---|---|
| 1 | Permission to run a persistent local service | ~10 min | 🔴 Principal |
| 2 | Reachability + client isolation confirmation | ~15 min | IT |
| 3 | Identity source | ~10 min | IT |
| 4 | Substitution approver + timetable owner | ~15 min | Timetable |
| 5 | Windows edition + kiosk admin rights | ~10 min | IT |
| 6 | Does a NAS exist at all? | ~10 min | IT + Library |
| 7 | External AI service permitted, in writing? | ~5 min | Principal |

⚠️ **Total: ~25 minutes of a Principal's time and ~40 minutes of IT's time.**

🔴 **That is the entire remaining cost of learning whether this project is
deployable.** Everything after it is only worth spending once these return. This
is the single highest-leverage action available to anyone in the project.

---

## 9. Deployment status summary

| Metric | Value |
|---|---|
| Deployment model | `DESIGN_ONLY` |
| 🔴 Systems deployed | **0** |
| 🔴 Systems demonstrated | **0** |
| 🔴 Deployment permissions granted | **0** |
| 🔴 Blocking questions | **9**, all answerable in ~1 hour of human time |
| 🔴 Technical blockers | **0** — the blockers are all institutional |