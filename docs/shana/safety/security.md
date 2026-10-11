# Security

⚠️ **SHANA currently has no code, so it has almost no attack surface. One
critical defect exists regardless.**

---

## 1. 🔴 Open critical defect

| Field | Detail |
|---|---|
| **File** | `start_opencode_server.sh` (repository root) |
| **Severity** | 🔴 **Critical** |
| **Issue 1** | Hardcoded **plaintext administrative password** in an `export` line |
| **Issue 2** | Server bound to **`0.0.0.0:4096`** — reachable on every interface, including public ones |
| **Issue 3** | Not listed in `.gitignore` — **can be committed** |

### Required actions

| # | Action | Owner |
|---|---|---|
| 1 | 🔴 **Rotate the password immediately** — treat as compromised | Repository owner |
| 2 | Move the credential to a runtime environment variable | Repository owner |
| 3 | Bind to `127.0.0.1` or a specific interface | Repository owner |
| 4 | Add the script to `.gitignore` | Repository owner |
| 5 | Confirm it was never pushed anywhere | Repository owner |

⚠️ **The credential value is deliberately not reproduced in any SHANA document.**
No SHANA document should ever quote it.

⚠️ **The repository has 0 commits and 0 remotes**, which reduces the exposure
window — but the file is still on disk in cleartext and must be rotated.

---

## 2. Rules for any future SHANA code

| # | Rule | Rationale |
|---|---|---|
| 2.1 | ⛔ **Never commit secrets** — no passwords, tokens, keys, or certificates | The most common serious breach |
| 2.2 | ⛔ **Never commit student-identifying data** | C-7, absolute |
| 2.3 | ⛔ **Never commit raw video or stills of people** | C-7 |
| 2.4 | ⛔ **No AI model reaches the robot** | C-6 / ADR-001 |
| 2.5 | ✅ **Safety functions never depend on the network** | A dropped link must not cause unsafe motion |
| 2.6 | ✅ **Secrets supplied at runtime** via environment or secret store | Keeps them out of Git |
| 2.7 | ✅ **Deny by default** on all interfaces | Least privilege |
| 2.8 | ✅ **Log without payloads** | Auditability without surveillance |
| 2.9 | ✅ **No internal IPs, hostnames, or topology in committed docs** | Information disclosure |
| 2.10 | ✅ **`.gitignore` before first commit, not after** | Prevention over cleanup |

---

## 3. Threat model — proposed

⚠️ **Proposed only. No threat model has been formally reviewed.**

| Threat | Mitigation | Status |
|---|---|---|
| Network-borne command injection | Signed, validated messages; CloudLight cannot command motion | `DESIGN_ONLY` |
| Malicious or faulty peer sends `system.stop` | Honoured unconditionally — **fail safe, not fail open** | `DESIGN_ONLY` |
| Replay of a stale task | Sequence numbers, expiry | `DESIGN_ONLY` |
| Spurious destination | Strict allow-list; reject unknown destinations | `DESIGN_ONLY` |
| Network partition during motion | Local motion continues safely; hold if uncertain | `DESIGN_ONLY` |
| Credential compromise | Runtime secrets, rotation, `.gitignore` | ⚠️ **One live violation found** |
| Physical tampering with the robot | Physical controls — school responsibility | `UNKNOWN` |
| Surveillance of students | C-7 absolute | `DESIGN_ONLY` |

---

## 4. Privacy — non-negotiable

Security and privacy are treated as one topic because in this project they are
the same topic.

| # | Rule |
|---|---|
| 4.1 | ⛔ **No student is ever identified** |
| 4.2 | ⛔ **No raw video leaves the robot** (ADR-010) |
| 4.3 | ⛔ **No persistent identifiers** |
| 4.4 | ⛔ **No biometrics** |
| 4.5 | ⛔ **No CCTV integration** — deprecated on privacy grounds |
| 4.6 | ✅ **Detection and identity are separated**: the system may know *someone* is there; it may never know *who* |

⚠️ **4.6 is the whole design in one line.** See
[`student_privacy.md`](student_privacy.md).

---

## 5. Verification status

| Check | Result |
|---|---|
| Secrets scan of SHANA docs | ✅ **No SHANA document contains a credential** |
| SHANA code security review | N/A — ⚠️ **no code exists** |
| Credential rotation | 🔴 **NOT DONE — requires owner** |
| `.gitignore` protection for the script | 🔴 **NOT DONE** |
| Threat model review | ❌ **Not performed** |

---

## 6. 🔴 Summary

> SHANA has no code, so it has little to attack — but the repository contains
> **one critical credential defect inherited from an unrelated script.**
>
> **That password must be rotated, the script must be gitignored, and the server
> must not bind to all interfaces.**
>
> Nothing about SHANA's own security posture can be verified, because nothing has
> been built. These rules exist so that the first line of code written respects
> them.