# SHANA — Operating Restrictions

**Status: PROPOSED. None of these restrictions has been agreed or enforced.**

---

## Absolute — no phase

| # | Restriction | Basis |
|---|---|---|
| R-01 | **No unsupervised operation, ever** | Manufacturer: not a toy, not for under-14s, must be kept in the user's view |
| R-02 | **No movement to unverified locations** | Design principle |
| R-03 | **No teacher access to motor control** | Design principle |
| R-04 | **No AI model reaching the robot** | Design principle |
| R-05 | **No public network exposure** | Design principle |
| R-06 | **No lift modification, ever** | Statutory |
| R-07 | **No facial recognition, biometric identification, or automated enforcement** | Absolute |
| R-08 | **No credentials in the repository** | Absolute |
| R-09 | **No delivery completed without explicit recipient confirmation** | Design principle |
| R-10 | **No operation without a named supervisor present and on station** | Design principle |

---

## Pilot-specific — current scope

| # | Restriction | Reversal condition |
|---|---|---|
| R-11 | **Single floor only** | Multi-floor proven with floor identity (`ADR-002`) |
| R-12 | **No stairs** | `ADR-003` reversal conditions — measured risers ≤160 mm, written manufacturer and school approval, supervised test passing including forward descent |
| R-13 | **No lifts** | `ADR-003` reversal conditions — manufacturer-confirmed sanctioned interface, permissions obtained |
| R-14 | **No doors of any kind** | A specific demonstrated requirement |
| R-15 | **Approved destinations only** | An approved process for adding destinations |
| R-16 | **Approved items only** | An approved change to the item list |
| R-17 | **Payload ≤2 kg** | Measured payload capability |
| R-18 | **Maximum 0.4 m/s** | A measured and approved higher limit |
| R-19 | **Exclusion zone ≥1.5 m either side** | A different physically enforced arrangement |
| R-20 | **Pre-verified routes only** | Never — this is permanent |
| R-21 | **No CCTV analytics** | `SH-PRIV-008` resolves favourably |
| R-22 | **No radio telemetry** | `SH-NET-009` identifies a coverage gap |

---

## Environmental

| # | Restriction | Basis |
|---|---|---|
| R-23 | **No wet or low-friction floor operation** | ✅ Manufacturer manual: *"the ground is wet"* is named as a forbidden condition; slip is the named failure mode |
| R-24 | **No outdoor or rain-exposed routes** | Robot is not weatherproof |
| R-25 | **Operating temperature 5–35 °C** | ✅ Manufacturer specification |
| R-26 | **No charging below 5 °C or above 40 °C ambient** | ✅ Manufacturer limit |
| R-27 | **No stairs above 16 cm** | ✅ Manufacturer limit |
| R-28 | **No slopes above 40°** | ✅ Manufacturer specification |
| R-29 | **No fire or evacuation routes** | Legal and safety |
| R-30 | **No restricted areas** | School policy |

---

## Battery

| # | Restriction | Basis |
|---|---|---|
| R-31 | **No operation below the abort threshold** — TBD by measurement | Verified: crouch within 10 min at low battery |
| R-32 | **No unsupervised charging** | School policy required |
| R-33 | **No charging on a flammable surface** | Manufacturer limits |
| R-34 | **No charging in electromagnetic environments** | ✅ Manufacturer prohibition |
| R-35 | **A swollen, damaged, or dropped pack is quarantined, never used** | Safety |
| R-36 | **No charging below 10% or storing above 40%** | ✅ Manufacturer limits |

---

## Operating hours

| # | Restriction | Status |
|---|---|---|
| R-37 | **No operation outside authorised hours** | ⚠️ Not agreed — `SH-TIME-020` |
| R-38 | **No operation during examinations** | ⚠️ Proposed |
| R-39 | **No operation during assemblies or events** | ⚠️ Proposed |
| R-40 | **No new deliveries after the lunch cutoff** | ⚠️ Lead time TBD — `SH-TIME-006` |
| R-41 | **Return to checkpoint before lunch begins** | ⚠️ Lead time TBD |
| R-42 | **No resumption without an authorised person** | ⚠️ Proposed |

---

## Payload

| # | Restriction | Basis |
|---|---|---|
| R-43 | **No glass** | Fragile; a drop is a hazard |
| R-44 | **No liquids or unsealed containers** | Spill; slip hazard |
| R-45 | **No chemicals, reagents, or hazardous materials** | ✅ Explicit project rule |
| R-46 | **No sharps or pointed objects** | Injury risk |
| R-47 | **No heated apparatus or hot items** | Burn risk |
| R-48 | **No fragile items** | Breakage hazard |
| R-49 | **Nothing that could be mistaken for a weapon or hazard to a member of the public** | Perception |
| R-50 | **Contents declared in advance and checked against the allow-list** | Safety |
| R-51 | **Two independent restraints, tilt-tested before use** | Safety |
| R-52 | **Auto-recovery disabled during payload missions** | `RecoveryStand()` behaviour |

---

## Emergency

| # | Restriction | Basis |
|---|---|---|
| R-53 | **No restart after an abort without explicit supervisor authorisation** | Procedure |
| R-54 | **No resumption after an injury** | Procedure |
| R-55 | **No robot retrieval during a fire evacuation** | Life safety |
| R-56 | **No operation with a modified or tampered robot** | Safety |
| R-57 | **All operation stops if any permission is withdrawn** | Governance |

---

## Enforcement

| Mechanism | Covers |
|---|---|
| Software clamp | R-03, R-04, R-18, R-31 |
| Validator | R-02, R-15, R-16, R-47 to R-50 |
| Route planner | R-11, R-12, R-13, R-20, R-29, R-30 |
| Corridor state | R-23, R-30 |
| Hardware interlocks | R-06 |
| Written procedure | R-01, R-10, R-32 to R-42, R-53 to R-57 |
| **Human supervisor** | **R-01, R-10, R-19, and everything not automatable** |

⚠️ **Some restrictions cannot be enforced in software.** A supervisor who does not
stop the robot when they should is the failure mode this project cannot design
away. That is why the supervisor is a named, trained person rather than a
volunteer.

---

## Review

This list should be reviewed:

- After any measurement that changes a threshold
- After any gate status change
- Before any expansion of scope
- Whenever a fault or near-miss occurs
- At least annually

⚠️ **A restriction may be lifted with evidence. It may never be lifted by
expiry, by pressure, or by convenience.**