# Test Inventory

🔴 **The authoritative answer to "what has been tested?"**

## Headline

> ## Tests executed: **ZERO (0)**
>
> Tests written: **ZERO (0)**
>
> Test files: **ZERO**
>
> CI: **does not exist**

There is nothing to run. No test framework has been installed, no test has been
written, and no test has ever been executed.

---

## 1. Verification of this claim

| Check | Method | Result |
|---|---|---|
| Test files | Filesystem scan for `test_*.py`, `*_test.go`, `*.test.ts`, `*.spec.js` | **None** |
| Test directories | Filesystem scan for `tests/`, `test/`, `spec/` | **None** |
| Test framework config | Scan for `pytest.ini`, `jest.config.*`, `vitest.config.*` | **None** |
| CI configuration | Check for `.github/workflows/` | **None** |
| Build configuration | Scan for `package.json`, `pyproject.toml`, `Makefile`, `CMakeLists.txt` | **None** |
| Test report artefacts | Scan for coverage reports, junit XML | **None** |
| Implementation code to test | Scan for all common source extensions | **None** |

⚠️ There is no code, so there is nothing to test. This is not an oversight in
recording — it is the primary finding.

---

## 2. Planned test tiers — none started

| Tier | Scope | Status | Can it start now? |
|---|---|---|---|
| **T1 — Static** | Schema validity, link integrity, linting | ⚠️ **Partially possible** | ✅ **Yes** |
| **T2 — Simulation** | Navigation, avoidance, state machine in simulation | `NOT STARTED` | ✅ **Yes** — no permissions needed |
| **T3 — Hardware bench** | Robot responses, battery, sensors | `BLOCKED` | ❌ Needs robot access + permissions |
| **T4 — Integration** | CloudLight interface, schema conformance | `BLOCKED` | ❌ Needs a partner that does not exist |
| **T5 — Field** | Real school corridor operation | `BLOCKED` | ❌ Needs gates, permissions, survey |

⚠️ **Only T1 and T2 are genuinely unblocked.** T2 is the highest-value next
step, because it produces a working artefact under the current rules.

---

## 3. Checks actually performed

These are the only verifications ever run against this repository. They are
documentation-integrity checks, **not** tests of any SHANA capability.

| # | Check | Result | Date |
|---|---|---|---|
| 3.1 | All relative Markdown links resolve | ✅ **PASS — 0 broken** | 2026-10-11 |
| 3.2 | ADR index entries match files on disk | ✅ **PASS after correction** — index previously claimed 12 ADRs; only 2 existed | 2026-10-11 |
| 3.3 | Questionnaire ID uniqueness | ✅ **PASS — 434 unique IDs, 0 duplicates** | earlier |
| 3.4 | Cross-document consistency after renames | ✅ **PASS** | 2026-10-11 |
| 3.5 | Visual inspection of all 8 image files | ✅ **PASS — all are unrelated display hardware** | 2026-10-11 |

⚠️ These checks validate **documentation only**. They say nothing about whether
SHANA works, because SHANA is not built.

---

## 4. Outstanding test debt

| # | Test needed | Blocked by |
|---|---|---|
| 4.1 | Schema YAML parse validation | ⚠️ **Not blocked** — never done |
| 4.2 | Message schema round-trip | ⚠️ **Not blocked** — needs code |
| 4.3 | State machine transition coverage | ⚠️ **Not blocked** — needs simulation |
| 4.4 | Obstacle avoidance under simulation | ⚠️ **Not blocked** |
| 4.5 | Battery-reserve policy simulation | ⚠️ **Not blocked** |
| 4.6 | Emergency stop timing | ❌ Hardware |
| 4.7 | Obstacle stop distance measurement | ❌ Hardware |
| 4.8 | Stair rejection behaviour | ❌ Hardware |
| 4.9 | Network failure and degraded-mode behaviour | ❌ Hardware |
| 4.10 | CloudLight interface conformance | ❌ Partner |
| 4.11 | Field trial against acceptance criteria | ❌ Everything |

---

## 5. 🔴 Integrity statement

> **No test has ever been run against SHANA. No performance figure has ever been
> measured. No claim of tested behaviour may be made on the basis of anything in
> this repository.**
>
> The only verified facts are documentation-integrity facts, listed in §3.
>
> Any future document claiming test results must cite a run log. **A claim of
> "passing tests" without a run log is fabrication and is prohibited by
> [`../handoff/agent_handoff.md`](../handoff/agent_handoff.md) §4.1.**

See [`known_failures.md`](known_failures.md) for known defects and contradictions.