# SHANA — CCTV Integration and Corridor Occupancy

**Status: NOT REQUESTED. NOT APPROVED. NO CCTV SYSTEM HAS BEEN ACCESSED.**

⚠️ **This document exists to help the school decide whether to say no.**

---

## The proposal, stated plainly

An authorised local application may estimate **approximate anonymous corridor
occupancy** from approved feeds, producing one of four states — `CLEAR`,
`MODERATE`, `BUSY`, `UNKNOWN` — used only to influence route selection.

**Purpose: route congestion estimation. Not identifying people.**

---

## Absolute constraints

| # | Constraint |
|---|---|
| 1 | **No facial recognition** |
| 2 | **No identity recognition** |
| 3 | **No biometric identification** |
| 4 | **No persistent tracking** of identifiable persons |
| 5 | **No automatic misconduct accusations** |
| 6 | **No automatic disciplinary or attendance reports** |
| 7 | **No raw video stored by default** — frames processed in-stream and discarded |
| 8 | **No bypassing CCTV authentication or access restrictions** |
| 9 | **No real student footage used during research** |
| 10 | **Human review and school approval** for any incident report |
| 11 | **Audio is never decoded** |
| 12 | **CCTV is never the robot's sole obstacle-avoidance system** |
| 13 | Low confidence or feed failure → `UNKNOWN`, **never `CLEAR`** |
| 14 | Sensor may **raise a prompt**, never open or close a corridor |

⚠️ **Constraints 1–6 are not negotiable for convenience. Ever.**

---

## Research findings

### It is technically possible — standards-native

✅ **ONVIF Profile M v1.1 §7.5.3 makes *Metadata Streaming* MANDATORY, and §8.2.3
makes *Video Streaming* CONDITIONAL.** A conformant analytics device can stream
zero video.

✅ The ONVIF Analytics Service Spec v26.06 §A.6 defines
`tns1:RuleEngine/CountAggregation/OccupancyCounter` carrying only
`Count [xs:int]`.

⚠️ **But Profile M v1.1 contains zero occurrences of "occupancy."** It mandates
only line-crossing counters. **An integrator who assumes Profile M implies
occupancy counting is wrong** — you must call `GetSupportedRules` per device.

### Indian legal position — three verified negatives

✅ **Neither the DPDP Act 2023 nor the DPDP Rules 2025 mentions CCTV, surveillance,
biometrics, or facial recognition anywhere in their text.**

✅ **The operative sections, including §9 on children, do not come into force
until approximately May 2027** per G.S.R. 843(E)(c).

✅ **DPDP Rules 2025 Rule 12 and Fourth Schedule Part A grant educational
institutions an exemption — but conditioned on processing for educational
activities or "in the interests of safety of children."** ⚠️ **Corridor traffic
routing is not obviously either.**

✅ **IT Act 2000 §66E is narrow** — it criminalises images of *"genitals, public
area, buttocks or female breast"*, not corridor CCTV.

⚠️ **No Supreme Court judgment on CCTV in schools was found.** The one real SC
order on educational institutions located (25 July 2025) concerns mental health
policy and is recorded here specifically so it is not mistaken for authority.

### An existing regulatory fact worth knowing

✅ **CBSE clause 4.7.10 (21 July 2025) already mandates audio-visual CCTV in
corridors, with 15-day retention.**

In a CBSE school, the footage **already exists** and the school is **already the
Data Fiduciary**. ⚠️ **Which means the stream carries audio, which the pipeline
must explicitly not decode.**

### OpenCV people detection — honesty required

⚠️ **No published accuracy benchmark for OpenCV people detection in a cluttered
school corridor was found. NOT FOUND.**

⚠️ **OpenCV moved HOG and Haar people detection to `opencv_contrib` in OpenCV
5.0.0**, stating *"Modern deep learning-based detectors are both faster and more
accurate."*

⚠️ The commonly quoted HOG miss-rate figure was **not verifiable** — the source
page returned an error and the documentation site served a challenge. **It is not
cited here.**

Known failure modes regardless of detector: partial occlusion, seated people,
people at frame edge, lighting change, shadows, reflections.

---

## The honest comparison

| Option | Privacy | Cost | Latency | Reliability | Permission burden |
|---|---|---|---|---|---|
| **(a) Existing CCTV analytics** | Medium — needs justification | Low if NVR already supports it | Medium | Depends on camera placement | ⚠️ **High** |
| **(b) Robot camera + depth** | High — it's our own view | ₹0 | Low | ⚠️ Narrow field of view; only sees ahead | Low |
| **(c) mmWave / ToF presence** | ⛔ **Highest — no image ever formed** | ₹2,000–11,000 | Low | Good | **Low** |
| **(d) Manual staff report** | ⛔ **Highest — no sensing at all** | ₹0 | Medium — human latency | Depends on staff | **Very low** |
| **(e) Combination** | Varies | Medium | Best | Best | High |

### On mmWave

⛔ **No image of any person is ever formed.** Published products include
HLK-LD6001C (in/out counting to ~8 people, ≤30 ms processing, ~$28) and Aqara FP2
(multi-person tracking, ₹11,128 on Amazon India). Many provide **presence rather
than count** — which may be entirely sufficient for `CLEAR`/`MODERATE`/`BUSY`.

### The finding that should drive the decision

> **The same aim — `CLEAR`/`MODERATE`/`BUSY` — can be achieved by CCTV analytics, by
> a mmWave sensor, or by a staff member pressing a button.**

The CCTV route is strongest in exactly one configuration: **an existing NVR that
already emits the count as metadata with no additional privacy burden.**

⚠️ **In every other configuration, a simpler option exists that avoids the
privacy question entirely.**

---

## Recommended decision sequence

1. **Ask whether corridor congestion awareness is needed at all.** It is a
   refinement, not a core capability (`SH-DEC-010`).
2. **If not needed → skip it.** The project proceeds without it.
3. **If needed → try manual staff reporting first** (`SH-DEC-011`). Zero privacy
   burden, zero cost, no analytics to maintain.
4. **If manual is insufficient → evaluate mmWave.** No image ever formed.
5. **Only if all of the above fail → consider CCTV analytics**, and only with a
   written privacy determination and a Data Fiduciary determination on whether an
   exemption covers corridor routing.

⚠️ **The recommended first action is a question, not a build.**

---

## Degraded behaviour

| Condition | Resulting state |
|---|---|
| Feed unavailable | `UNKNOWN` |
| Confidence below threshold | `UNKNOWN` |
| Data stale beyond expiry | `UNKNOWN` |
| Sensor failure | `UNKNOWN` |
| No permission | Feature does not exist |

⚠️ **Route planning must treat `UNKNOWN` as blocked.** A failed occupancy sensor
must never make a corridor look empty.

---

## Questions

| ID | Question | Priority |
|---|---|---|
| `SH-CCTV-001` to `-005` | Brand, model, ONVIF/RTSP, authentication | P1 |
| `SH-CCTV-006` | Coverage of candidate routes | P1 |
| `SH-CCTV-009` | **Does any camera emit occupancy or line-crossing metadata without video?** | P1 |
| `SH-CCTV-012` to `-015` | **Anonymous counts only; no biometrics; no retention; no bypass** | **P0** |
| `SH-CCTV-016` | **Does policy permit any analytics access?** | **P0** |
| `SH-CCTV-017` | Who is the Data Fiduciary, and have they approved? | **P0** |
| `SH-CCTV-018` | Is a privacy impact assessment required? | P1 |
| `SH-CCTV-020` | ⚠️ **Does any exemption cover corridor routing?** | **P0** |
| `SH-CCTV-021` | **CCTV confirmed not to be the sole avoidance system** | **P0** |
| `SH-CCTV-023` | **No automatic misconduct reporting** | **P0** |
| `SH-CCTV-024` | Has mmWave been considered instead? | P1 |

---

**Full research:** `docs/shana/infrastructure/cctv_research_evidence.md`
**Sources:** `docs/shana/research/cctv_source_register.md`

⚠️ **Nothing in this category has been requested, approved, or accessed. The
default answer is that SHANA does not use CCTV.**