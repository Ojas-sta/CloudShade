# Known Failures, Defects, and Contradictions

🔴 **An honest register of everything wrong, uncertain, or self-contradictory in
this project.**

Most projects hide this file. It exists so that the next person does not
rediscover these problems the hard way, and so that no defect is mistaken for
working behaviour.

---

## 1. 🔴 Corrected defects

| # | Defect | Impact | Status |
|---|---|---|---|
| 1.1 | 🔴 **ADR index claimed 12 ADRs; only 2 existed** | A reader would believe 10 decisions were documented when they were not | ✅ **CORRECTED 2026-10-11.** Index now records the gap explicitly and points to the decision log |
| 1.2 | ⚠️ **12 broken relative Markdown links** | Navigation broken across the documentation set | ✅ **CORRECTED 2026-10-11.** Now 0 broken links |
| 1.3 | ⚠️ **`docs/shana/infrastructure/cctv_research_evidence.md` referenced a non-existent source register** | Citation chain incomplete | ✅ **CORRECTED 2026-10-11** |
| 1.4 | ⚠️ **Documentation tree restructured under `docs/shana/`** | SHANA and CloudLight namespaces were mixed | ✅ **CORRECTED 2026-10-11.** All links revalidated |

---

## 2. 🔴 Open security defects

| # | Defect | Severity | Status |
|---|---|---|---|
| 2.1 | 🔴 **`start_opencode_server.sh` contains a hardcoded plaintext password** | **Critical** | ⚠️ **OPEN — requires owner action** |
| 2.2 | 🔴 **Script binds a server to `0.0.0.0:4096`**, exposing it on all interfaces including public ones | **High** | ⚠️ **OPEN** |
| 2.3 | ⚠️ **Script is not in `.gitignore`** and could be committed | **High** | ⚠️ **OPEN** |
| 2.4 | ⚠️ **The credential must be treated as compromised** — it is on disk in cleartext | **Critical** | ⚠️ **Requires rotation** |

🔴 **The credential value is deliberately not reproduced in any SHANA document.**
Required actions are listed in
[`../overview/current_project_state.md`](../overview/current_project_state.md).

⚠️ SHANA has **no code**, so it has no other credentials. The same discipline
applies to anything added later.

---

## 3. 🔴 Content defects — the repository contains misleading assets

| # | Defect | Impact | Status |
|---|---|---|---|
| 3.1 | 🔴 **8 root JPEG files are unrelated LCD/display panel photographs** | Could be mistaken for SHANA evidence at an exhibition | ✅ **DOCUMENTED** — excluded via [`../handoff/exhibition_evidence_inventory.md`](../handoff/exhibition_evidence_inventory.md). **They should also be moved out or deleted by the owner** |
| 3.2 | 🔴 **Zero code, zero tests, zero build system** | "Implementation-ready" reading is impossible | ✅ **DOCUMENTED** — [`test_inventory.md`](test_inventory.md) |
| 3.3 | ⚠️ **`docs/cloudlight/` was empty at audit** | No real integration exists | ⚠️ **OPEN** — owned by another agent |

---

## 4. Structural gaps — promised but missing

| # | Gap | Why it matters |
|---|---|---|
| 4.1 | ⚠️ **10 owed ADRs** — decisions recorded in the decision log but never written as ADRs | Real decisions risk being lost or silently reversed |
| 4.2 | 🔴 **No simulation, though ADR-011 mandates simulation-first** | The project's own rule is being violated |
| 4.3 | 🔴 **No CI, no build system, no dependency manifest** | Nothing is reproducible |
| 4.4 | ⚠️ **Schemas never validated as parseable YAML** | A schema that does not parse is not a schema |
| 4.5 | ⚠️ **`data/` templates are empty by design** | Nothing to test against |

---

## 5. Contradictions within the documentation

⚠️ These are places where two documents could be read as saying different things.

| # | Contradiction | Resolution |
|---|---|---|
| 5.1 | "SHANA delivers to classrooms" vs "nothing is built" | 🔴 The first is **design intent**. Always read it as "proposes to" |
| 5.2 | "Battery telemetry alerts at 20%" vs no code exists | 🔴 Design intent. No threshold is enforced anywhere |
| 5.3 | "CCTV provides occupancy" vs privacy position opposes CCTV | 🔴 **CCTV is `DEPRECATED` and recommended against.** Any surviving CCTV text is historical |
| 5.4 | "Lifts excluded" vs lift integration documents exist | 🔴 Lifts are excluded **because no sanctioned API exists.** The documents explain why exclusion is correct |
| 5.5 | "Robot model is Go2 EDU" vs model never confirmed | 🔴 The **school's** robot is presumed EDU but **unconfirmed**. Gate 1 |
| 5.6 | "52 acceptance criteria" vs 0 met | No contradiction — the criteria define the work. **0 met** |
| 5.7 | Vendor documents autonomous docking vs "no autonomous return-to-dock" | ⚠️ Not a contradiction: the **capability** is vendor-documented; the **integration is not configured**. Always qualify |

⚠️ **Contributing factor: the documentation was authored primarily by one agent
working from vendor documentation, without a second reviewer.** Single-author
documentation is the root risk behind most items in §5.

---

## 6. Known design weaknesses

| # | Weakness | Consequence |
|---|---|---|
| 6.1 | 🔴 **No floor identity sensor** | Multi-floor is impossible; mitigations are proposed but unproven |
| 6.2 | 🔴 **No autonomous dock** | Battery recovery is human-assisted — an operational cost |
| 6.3 | ⚠️ **Occupancy is human-declared** | Depends on staff compliance; no sensing backup |
| 6.4 | ⚠️ **Safety case has zero evidence** | Cannot claim any safety property |
| 6.5 | ⚠️ **30 of 34 assumptions unverified** | The design rests on unverified premises |
| 6.6 | ⚠️ **Self-built navigation stack** (ADR-005, provisional) | Higher risk than vendor SLAM; unvalidated |
| 6.7 | ⚠️ **Marked-in-the-environment localisation** (ADR-006, provisional) | Needs survey to validate |

---

## 7. 🔴 Integrity statement

> This project currently has:
>
> - **0** passing tests — because **0** tests exist
> - **0** implemented components
> - **0** passed gates
> - **0** granted permissions
> - **0** answered questions of 434
> - **1** critical security defect requiring owner action
> - **10** owed ADRs
> - **8** unrelated images that must never be presented as evidence
>
> **None of these are hidden. That is the point of this file.**