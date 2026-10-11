# Security and Trust Boundaries

**Owner:** Agent 3 (Integration Lead)
**Status:** 🔴 `DESIGN_ONLY` — **no system exists to secure**
**Last updated:** 2026-10-11
**Read with:** `AGENTS.md` §K (secrets) · §L (safety and privacy) ·
[`interface_contracts.md`](interface_contracts.md)

---

## 🔴 Standing caveat

**There is nothing deployed.** No host, no service, no database, no network
segment, no credential store, no endpoint. Every boundary below is *designed*,
not *enforced*.

A security architecture with no implementation is a set of intentions. It is
worth writing down — intentions become implementation or they do not — but it
must never be described as protection in place.

---

## 1. Trust boundaries

```
┌────────────────────────────────────────────────────────────────┐
│ TRUST ZONE 1 — School LAN                    🔴 NOT TRUSTED    │
│                                                                 │
│  Classroom displays · teacher devices · unknown hosts           │
│  ❌ No authentication assumed                                    │
│  ❌ No authorisation assumed                                    │
│  ⚠️ Client isolation status UNKNOWN (CL-NET-008)               │
├────────────────────────────────────────────────────────────────┤
│ TRUST ZONE 2 — CloudLight host                                  │
│                                                                 │
│  ✅ Authenticates teachers                                       │
│  ✅ Enforces authorisation                                      │
│  ✅ Holds the authoritative request log                          │
│  ❌ MUST NOT hold any robot control capability                   │
├────────────────────────────────────────────────────────────────┤
│ 🔴 BOUNDARY — the integration interface                          │
│                                                                 │
│  Mutual authentication REQUIRED (unimplemented)                 │
│  Request direction only: CloudLight → SHANA                     │
│  🔴 Any unauthenticated path here lets a host MOVE THE ROBOT    │
├────────────────────────────────────────────────────────────────┤
│ TRUST ZONE 3 — SHANA                                            │
│                                                                 │
│  ✅ Independently validates every request                         │
│  ✅ Owns all safety decisions                                    │
│  ✅ Must be safe when Zone 2 is unreachable                      │
├────────────────────────────────────────────────────────────────┤
│ TRUST ZONE 4 — Robot / physical world                           │
│                                                                 │
│  Sensors · actuators · the corridor · the students              │
│  🔴 NO safety function may depend on Zones 1–3                  │
└────────────────────────────────────────────────────────────────┘
```

🔴 **The boundary between Zone 2 and Zone 3 is the most important line in the
project.** Crossing it with an unauthenticated command path converts a network
compromise into physical motion in an occupied corridor.

---

## 2. 🔴 The unauthenticated-endpoint problem

CloudLight's own specification
(`docs/cloudlight/shana_integration.md` §9) states it plainly:

> 🔴 **P0 control.** Without mutual authentication, any device that can reach
> SHANA's interface can create tasks.

It also correctly bounds the worst case:

> Combined with the absent motor-command path, the worst case is still "an
> unauthorised delivery request" — which is serious, but bounded. **It is far
> better than the alternative.**

⚠️ That bounding only holds while
[`ADR-014`](../architecture/architecture_decisions/ADR-014-cloudlight-never-controls-robot-motion.md)
holds. **The absence of a motion path is the thing keeping a network compromise
from becoming an injury.** It is therefore a security control, not merely an
architectural preference, and must be protected accordingly.

🔴 **Status: no authentication exists. No transport exists. No key custody
policy exists. No identity source exists.**

---

## 3. 🔴 The teacher-identity problem

| Requirement | Status |
|---|---|
| CloudLight must authenticate teachers | 🔴 Blocked — `CL-AUTH-001`: *does a staff identity directory exist?* |
| SHANA must authenticate CloudLight | 🔴 Not specified by SHANA — `IC-06` |
| Offboarding must revoke access | 🔴 Blocked — two stores, drift risk |

⚠️ A shared six-digit code may **authorise one action** or act as a **second
factor**. It is **never proof of identity** — it may not be remembered, guessed
from context, or socially engineered.

---

## 4. Authentication requirements (specified, unimplemented)

| Aspect | Requirement | Status |
|---|---|---|
| Caller authentication | mTLS, or a signed short-expiry token | 🔴 `DESIGN_ONLY` |
| Why not a shared static key alone | Single-key systems are single points of compromise | ✅ agreed in principle |
| Teacher identity to SHANA | A **reference** resolved by CloudLight; never a credential SHANA trusts | ⚠️ conflicts with SHANA's `teacher role id` — `CONF-10` |
| `system.stop` authorisation | 🔴 **None required.** It must be honoured unconditionally and immediately | ✅ designed |

⚠️ **`system.stop` is deliberately exempt from authorisation checks.** A stop
that can be refused is not a stop.

---

## 5. 🔴 Privacy position

Absolute, and consistent across both subsystems.

| # | Rule | Status |
|---|---|---|
| 1 | No facial recognition | ✅ agreed by both |
| 2 | No identity recognition | ✅ |
| 3 | No biometrics | ✅ |
| 4 | No persistent individual tracking | ✅ |
| 5 | No automated misconduct detection or reporting | ✅ |
| 6 | No raw video ever leaves the robot | ✅ |
| 7 | No student data collected | ✅ |
| 8 | No person is ever identified | ✅ |
| 9 | No CCTV use without explicit authorisation | ✅ + 🔴 **`CONF-08` in SHANA's register recommends against CCTV occupancy entirely** |
| 10 | A photograph of a whiteboard is never an authoritative record | ✅ |

⚠️ **This is the best-evidenced area of the project.** Two agents working
independently reached the same privacy position, and it is more restrictive
than either was asked for. 🔴 **It has not been legally reviewed.**

---

## 6. 🔴 Secrets handling

### Prohibited from Git, issues and documents

Wi-Fi keys · passwords · CCTV credentials or stream URLs · API tokens · robot or
server credentials · private network topology · IP assignments · VLAN
configuration · student or staff personal data · unapproved floor plans.

### Known exposure

| Item | Detail |
|---|---|
| File | `start_opencode_server.sh` |
| Issue | 🔴 Hardcoded plaintext administrative password; binds a server to `0.0.0.0:4096` |
| Committed? | 🔴 **No** — excluded by `.gitignore` before and after the baseline commit |
| Quoted anywhere? | 🔴 **No** — the value was never read into or reproduced in any document |
| 🔴 **Still required?** | ✅ **YES — ROTATE IT.** Being gitignored does not un-expose a cleartext password on disk |

⚠️ Owner action, not agent action. No agent may quote the value.

### Hygiene verified in Phase A

| Check | Result |
|---|---|
| Credential-pattern scan over every file proposed for staging | ✅ No live secret staged |
| Staged-file review before commit | ✅ 159 files reviewed |
| Machine-specific artefacts excluded | ✅ `.obsidian/`, `.DS_Store` |

⚠️ These were **manual, one-off checks**. They are not automated and will not
re-run unattended. See [`GIT_AND_AGENT_WORKFLOW_SPEC.md`](GIT_AND_AGENT_WORKFLOW_SPEC.md) §9.2.

---

## 7. 🔴 School infrastructure is out of bounds

**No agent may probe, scan, enumerate or connect to school infrastructure.**

| Prohibited | Why |
|---|---|
| Network scanning | Unauthorised access |
| CCTV connection | No authorisation exists |
| Lift-system contact | No authorisation exists; modification is unlawful |
| Device enumeration | Unauthorised access |
| `winver` on a board | ⚠️ Permitted **only** by an on-site human, on a non-critical board |

🔴 **Every school fact is a question with a named owner — never a finding.**

---

## 8. Update security

| Control | Status |
|---|---|
| Signed releases | 📄 Specified (`FR-OTA-01`). 🔴 No release mechanism exists |
| Rollback / freeze / fast-forward defence | 📄 TUF identified as a published specification |
| Key custody policy | 🔴 **Does not exist.** Deferred |
| Version verification endpoint | 📄 Specified (`GET /version`). Not implemented |

⚠️ A school LAN is not trusted. 🔴 Update signing is **deferred to after the
first pilot** — which means the first pilot would run unsigned software on
school hardware. That should be an explicit, recorded decision, not an
oversight.

---

## 9. 🔴 Threats that exist today, with no mitigation

| # | Threat | Severity | Mitigation |
|---|---|---|---|
| 1 | Cleartext credential on disk | 🔴 High | Owner rotates 🔴 **outstanding** |
| 2 | No remote → no review gate before a change lands | 🟠 Medium | Local commits only; 1 commit exists |
| 3 | No CI → a broken link or leaked secret ships unnoticed | 🟠 Medium | Manual checks performed once |
| 4 | No directory-ownership enforcement | 🟠 Medium | Convention only (`AGENTS.md` §C.3) |
| 5 | Photographic evidence misrepresented | 🟠 Medium | Recorded in the claim-evidence matrix |
| 6 | 🔴 Any future unauthenticated integration path | 🔴 Critical | 🔴 **Blocked** — no code may be written before gates pass |

---

## 10. Security status summary

| Area | Status |
|---|---|
| Secret hygiene | 🟡 Manual only; one rotation outstanding |
| Trust boundaries | ✅ Well-argued, **unenforced** |
| Authentication | 🔴 `UNKNOWN` — no identity source exists |
| Authorisation | 📄 Specified; 🔴 no approver named for substitutions |
| Privacy position | ✅ **Strongest area** — 🔴 not legally reviewed |
| Update security | ⛔ Deferred past the first pilot |
| Incident response | 🔴 **No system to respond to** |
| 🔴 Enforced controls | **0** |
| 🔴 Vulnerabilities | **0** — because there is no software |
| 🔴 Real risk | **1 cleartext credential awaiting rotation** |