# SHANA — Student Privacy

**Status: POLICY PROPOSAL. No determination has been made by any authority.**

⚠️ **This document proposes constraints. It does not grant, confirm, or establish
that any of them have been approved.**

---

## The position in one paragraph

SHANA's purpose is to move objects, not to observe people. Every design decision
follows from that. Where a feature would require observing people to be useful,
**the feature is deferred rather than the privacy requirement relaxed.**

---

## Absolute prohibitions

⚠️ **These are not configurable. Not for convenience, not for a better demo, not
for a policy exception.**

| # | Prohibition |
|---|---|
| 1 | **No facial recognition** |
| 2 | **No identity recognition** |
| 3 | **No biometric identification** of any kind |
| 4 | **No persistent tracking** of individuals across time |
| 5 | **No automatic accusation** of any student as suspicious or misbehaving |
| 6 | **No automatic identification, ranking, or reporting** of individual students |
| 7 | **No automatic disciplinary, attendance, or behavioural reporting** |
| 8 | **No storing raw video** by default |
| 9 | **No bypassing CCTV authentication** or access restrictions |
| 10 | **No use of real student footage** during research |
| 11 | **No collection of student personal data** by SHANA |
| 12 | **No use of signal strength** for identifying or tracking individuals |

---

## Research findings that shape the position

### Indian legal position

✅ **Neither the DPDP Act 2023 nor the DPDP Rules 2025 mentions CCTV,
surveillance, biometrics, or facial recognition anywhere in their text.**

✅ **The operative sections, including §9 on children, do not come into force
until approximately May 2027** per G.S.R. 843(E)(c).

✅ **DPDP Rules 2025 Rule 12 and Fourth Schedule Part A grant educational
institutions an exemption — conditioned on processing for educational activities
or "in the interests of safety of children".** ⚠️ **Corridor traffic routing is
not obviously either.**

✅ **IT Act 2000 §66E is narrow** — it criminalises images of *"genitals, public
area, buttocks or female breast"*, not corridor CCTV.

⚠️ **No Supreme Court judgment on CCTV in schools was found.** The one real SC
order on educational institutions located (25 July 2025) concerns mental health
policy and is recorded here so it is not mistaken for authority.

⚠️ **These are three verified negatives and a qualified exemption — not legal
advice.** A qualified determination is required: `SH-PRIV-008`, `SH-CCTV-020`.

### An existing regulatory fact

✅ **CBSE clause 4.7.10 (21 July 2025) already mandates audio-visual CCTV in
corridors with 15-day retention.**

⚠️ In a CBSE school the footage **already exists** and the school is **already
the Data Fiduciary**. ⚠️ **Which means the stream carries audio — which the
pipeline must explicitly not decode.**

---

## What is proposed, and what is not

| Feature | Privacy cost | Recommendation |
|---|---|---|
| Anonymous corridor states from CCTV | High — requires a determination | ⛔ **Deferred.** Try simpler options first |
| mmWave presence sensor | ⛔ **Lowest — no image ever formed** | Consider before CCTV |
| **Manual staff-reported corridor status** | ⛔ **None — no sensing at all** | ✅ **Recommended first** |
| Robot's own camera for marker detection | Medium — we see the corridor | Only if required; **no recording** |
| Robot's own camera for general perception | High | ⛔ **Not proposed** |

⚠️ **The default answer is that SHANA does not use CCTV.** See
`docs/shana/infrastructure/cctv_feasibility.md`.

---

## Design safeguards — if analytics were ever approved

| Safeguard | Implementation |
|---|---|
| Anonymous states only | `CLEAR`, `MODERATE`, `BUSY`, `UNKNOWN`. Never a count per person. Never an ID. |
| Frames discarded in-stream | No raw video written to disk, by default |
| Audio never decoded | Explicit pipeline configuration |
| Low confidence → `UNKNOWN` | Never `CLEAR` |
| No persistent IDs | No tracking across frames or cameras |
| No individual inference | No age, gender, uniform, or group detection |
| No enforcement output | Any incident requires human review and school approval |
| Minimal retention | ⚠️ TBD — `SH-PRIV-006`, `SH-PRIV-012` |
| Access control | Results never shown to students; restricted access |

---

## Data we do retain, and why

| Data | Retention | Why | Risk |
|---|---|---|---|
| Delivery records | Per school policy | Dispute resolution, audit | Low — no personal data |
| Mission telemetry | Per mission | Measurement | ⚠️ **Position reveals layout** |
| Battery health | Long-term | Lifecycle | Low |
| Fault history | Long-term | Reliability analysis | Low |
| Corridor status reports | Per school policy | Audit of closures | Low — named author, no students |
| **Point clouds** | ⛔ **Not retained** | — | Avoids a high-resolution layout record |
| **Camera or CCTV imagery** | ⛔ **Not retained by default** | — | Privacy |

⚠️ **Position telemetry is treated as sensitive** because it reveals the building
layout and, aggregated over time, could reveal staff movement patterns.

---

## Prohibited data uses

- ⛔ Occupancy data → individual assessment
- ⛔ Occupancy data → attendance
- ⛔ Occupancy data → behaviour monitoring
- ⛔ Any count → identifying a specific person
- ⛔ Any image → any display students can see
- ⛔ Any data → automated enforcement

⚠️ **Anonymous does not mean unlinkable.** A count of one person in a corridor at
3:15 p.m. can be identifying. Aggregated data is still data.

---

## Transparency

| Question | Answer |
|---|---|
| Are students told? | ⚠️ School policy decision — `SH-CCTV-016` |
| May teachers see occupancy results? | ⚠️ Restricted — never in a student-facing view |
| May photographs include students? | ⚠️ School policy — `SH-PRIV-017` |
| May the demonstration be filmed? | ⚠️ Organiser policy — `SH-EXPO-018` |

⚠️ **SHANA should be able to explain to any student, in plain language, what it
does and does not record.** If it cannot, that is a design failure.

---

## The Data Fiduciary

⚠️ **No Data Fiduciary has been identified.** Under applicable law the school
holds that role.

| Requirement | Status |
|---|---|
| Identified in writing | ❌ Not done |
| Project authorised in principle | ❌ Not requested |
| Privacy impact assessment completed | ❌ Not done |
| Legal determination on any exemption | ❌ Not done |
| Retention policy agreed | ❌ Not done |
| Breach process agreed | ❌ Not done |

Questions: `SH-PRIV-001`, `-002`, `-003`, `-008`, `-018`, `-022`

---

## Repository hygiene

⛔ **Never committed to Git:**

- Student or staff personal data
- CCTV credentials or stream URLs
- Building layouts not approved for circulation
- Floor plans, route maps, or telemetry
- Credentials of any kind

See `SECURITY.md` and `.gitignore`.

---

## The honest position

⚠️ **SHANA does not need to see people to be useful, so it does not look at them.**

The entire delivery function — transport, routing, checkpoints, battery,
scheduling — works with **no perception of people at all**. Corridor congestion is
a refinement, and a manual staff report achieves it with **zero privacy cost**.

If the simplest approach that works is chosen, the privacy question largely
disappears. That is the intended design, not an accident.