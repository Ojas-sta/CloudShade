# Current Project State

🔴 **Read this before any other document in this repository.**

This file exists because SHANA produced a great deal of documentation and it is
easy for a reader — or an agent — to mistake **volume of writing** for
**progress**. This file corrects that impression.

**Audit date:** 2026-10-11
**Audit method:** direct repository inspection, not recollection.

---

## The one-paragraph version

SHANA is a **feasibility study**, not a system. It contains approximately 100
markdown documents, four proposed data schemas, and a 434-question
questionnaire. It contains **no application code, no tests, no simulation, and
no hardware access**. Everything documented is either *researched from vendor
documentation*, *proposed*, or *unknown*. **The project cannot be demonstrated,
deployed, exhibited, or judged as a working system.**

---

## Verified repository facts

| Fact | Value | How verified |
|---|---|---|
| Branch | `main` | `git branch --show-current` |
| Commits | **0** | `git log` returns nothing |
| Git remotes | **0** | `git remote -v` returns nothing |
| Tracked files | **0** | nothing is staged or committed |
| Implementation code files | **0** | filesystem scan for `.py`, `.js`, `.ts`, `.tsx`, `.cpp`, `.go`, `.rs`, `.java`, `.cs` |
| Shell scripts | 1 (`start_opencode_server.sh`) | filesystem scan — unrelated to SHANA, and see the security note below |
| YAML schemas | 4 | `data/schemas/` |
| Markdown documents | ~100 | filesystem scan |
| Tests written | **0** | no test directory, no test files |
| Tests executed | **0** | nothing runnable exists |
| CI configuration | none | no `.github/workflows/` |
| Build system | none | no `package.json`, `pyproject.toml`, `Makefile`, `CMakeLists.txt` |
| `docs/cloudlight/` | **empty** at audit time | directory listing |
| Robot accessed | **never** | no hardware interaction has occurred |
| Network probed | **never** | no scanning, no CCTV connection, no lift contact |

---

## What exists, honestly assessed

| Artefact | Exists | Real status |
|---|---|---|
| Problem framing and goals | Yes | Researched, coherent, not validated by the school |
| Architecture design | Yes | Design only, never implemented |
| Hardware feasibility research | Yes | Strong. Grounded in vendor documentation |
| Platform selection (G1 vs Go2) | Yes | Complete decision, retained for its findings |
| Risk register | Yes | 21 risks registered, **0 mitigated** |
| Safety case | Yes | **Draft only — no evidence gathered** |
| Privacy position | Yes | Well-argued, not legally reviewed |
| Source register | Yes | Real sources with honest evidence classes |
| Data schemas | Yes | **Proposed YAML. Never validated. Never parsed by code.** |
| Feasibility questionnaire | Yes | **434 questions. 0 answered.** |
| Permission checklist | Yes | **102 permissions. 0 granted.** |
| Go/No-Go gates | Yes | **11 gates. 0 passed.** |
| Assumptions register | Yes | **34 assumptions. 30 unverified.** |
| Code | **No** | — |
| Tests | **No** | — |
| Simulation | **No** | — |
| Photographs or video of the robot | **No** | see below |
| School surveys | **No** | protocol written, never executed |

---

## ⚠️ The repository contains no usable evidence

Eight JPEG files exist at the repository root: `Image.jpeg` and `Image 2.jpeg`
through `Image 8.jpeg`.

**All eight were visually inspected. All eight are close-up photographs of
LCD and display panel assemblies — flex ribbon cables, LCD modules, and driver
boards.** They have no relationship to a quadruped robot, a school, a map, or
any SHANA subject.

Consequences:

- ⚠️ **There is no photograph of the Go2 robot in this repository.**
- ⚠️ **There is no photograph of the school, corridors, or destinations.**
- ⚠️ **There are no screenshots of any system.**
- ⚠️ **These images must not be used as exhibition evidence**, must not be
  captioned as SHANA output, and must not be included in any demonstration
  material.

Recorded in
[`../handoff/exhibition_evidence_inventory.md`](../handoff/exhibition_evidence_inventory.md).

---

## ⚠️ Security finding requiring owner action

`start_opencode_server.sh` at the repository root is unrelated to SHANA, but it
is inside the shared repository and it **must not be committed or shared**.

It contains a **hardcoded plaintext administrative password** in an
`export OPENCODE_SERVER_PASSWORD=...` line, and it starts the OpenCode server
bound to `0.0.0.0:4096`, exposing it on every network interface including
public ones.

Required actions:

1. 🔴 **Rotate that password.** Treat it as compromised — it is written to disk in
   cleartext and must be assumed exposed.
2. Add the script to `.gitignore`.
3. Bind the server to a specific interface or `127.0.0.1`, not `0.0.0.0`.
4. Move the credential to an environment variable supplied at runtime, never a
   literal in the script.

⚠️ The credential value is deliberately **not reproduced** in any SHANA
document. No SHANA document, and no agent, should quote or copy it.

SHANA has **no other code, so it has no other credentials** — but the same
discipline applies to anything added later. See
[`../safety/security.md`](../safety/security.md).

---

## The gap between documentation and reality

| The documentation reads as… | The reality is… |
|---|---|
| "The navigation stack uses…" | No navigation stack exists |
| "Battery monitoring alerts at 20%" | No battery monitoring code exists |
| "The lift module reports status via…" | No lift module exists, and no lift API exists to integrate |
| "CCTV provides corridor occupancy" | Not implemented — and **recommended against** on privacy grounds |
| "Integration tests verify…" | No integration tests exist |
| "Acceptance criteria 1–52" | **0 of 52 met** |
| "The deployment pipeline supports…" | No pipeline exists |

⚠️ **This gap is the single most important thing for any incoming agent or
judge to understand.** Phrasing throughout this documentation set deliberately
uses conditional and proposed language to keep the gap visible. If a sentence
appears to assert that something exists, it should be corrected rather than
trusted.

---

## What would meaningfully change this state

In priority order — and note that **none of these are software tasks**:

| # | Action | Unblocks |
|---|---|---|
| 1 | Answer the **school-critical** subset of the 434 questions | Almost everything |
| 2 | Obtain the **Go2 EDU model number, firmware, and SDK licence status** | Hardware feasibility, gates 1–2 |
| 3 | Get written answers from the **lift manufacturer** | Cross-floor scope, gate 6 |
| 4 | Conduct the **route survey** on foot at the school | Navigation design, gates 3–4 |
| 5 | Obtain a **privacy assessment** for any camera or CCTV use | Gate 8 |
| 6 | Run a **simulation-first** proof of concept | Any code at all |

Full ordering: [`phase_summary.md`](phase_summary.md) §Next 10 actions.

---

## If you are an agent about to act

| Do | Do not |
|---|---|
| Read [`../handoff/agent_handoff.md`](../handoff/agent_handoff.md) first | ❌ Begin implementation — the gates forbid it |
| Verify claims in [`../research/claim_evidence_matrix.md`](../research/claim_evidence_matrix.md) | ❌ Trust a document's phrasing as proof of existence |
| Add to [`../research/research_log.md`](../research/research_log.md) | ❌ Modify anything under `docs/cloudlight/` |
| Update [`implementation_inventory.md`](../handoff/implementation_inventory.md) when status changes | ❌ Commit or push without explicit instruction |
| Mark unknowns as `UNKNOWN` rather than guessing | ❌ Fabricate test results, screenshots, or photographs |