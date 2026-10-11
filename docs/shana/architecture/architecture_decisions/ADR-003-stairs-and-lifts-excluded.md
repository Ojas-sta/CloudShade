# ADR-003: Stairs and lifts excluded from the pilot

**Status:** Provisional
**Date:** 2026-10-10
**Decides:** Whether stair or lift transitions are in scope for the initial pilot
**Reversal condition:** See below, per modality

---

## Context

The school spans three blocks and three to four levels. Vertical transit is
therefore a natural requirement — and the least established capability on the
platform.

---

## Stairs

### Evidence

| Claim | Class | Source |
|---|---|---|
| Maximum climb/drop height **16 cm** | VERIFIED | `about_Go2`; corroborated by User Manual |
| *"not currently recommended that robot walk stairs higher than 16cm… likely to trip over the foot"* | VERIFIED | Go2 User Manual §5 |
| *"it is forbidden to use the robot on the occasions where the terrain is complicated… steps higher than 16cm"* | VERIFIED | Go2 User Manual §9 |
| **`SwitchGait` — the only documented stair-specific API — existed on firmware < 1.1.6 and is ABSENT from the current interface** | VERIFIED | Motion Services Interface V2.0; `SportClient` source |
| `SwitchGait(3)` = forward climbing, `SwitchGait(4)` = reverse climbing, **firmware < 1.1.6 only** | VERIFIED | `sports_services` doc |
| `ClassicWalk` *"supports climbing stairs, gravel, meadows, and slippery surfaces"* | VERIFIED | Motion Services Interface V2.0 |
| Independent reports consistently describe **descent as the worse case** | UNVERIFIED | DroneBlocks; Miraenda; mybotshop forum |
| At low battery the robot **crouches within 10 minutes** | VERIFIED | `about_Go2` pilot-lamp table |
| *"Hand may be pinched at the motion joints"* | VERIFIED | Go2 User Manual |

### Step-height arithmetic

| Typical riser | vs 160 mm limit | Verdict |
|---|---|---|
| 150 mm | +10 mm (6.7 %) | Inside spec — **10 mm of tolerance** |
| 160 mm | **0 mm** | Exactly at limit. **Zero margin** |
| 170 mm | −10 mm | **Exceeds** |
| 180 mm | −20 mm | **Exceeds** |

### Aggravating factors

- The 16 cm figure is a **no-load** figure. No derated figure under payload is
  published.
- On the current SDK there is **no stair gait setter**. Stair capability would be
  an emergent property of `ClassicWalk` with no documented step-height parameter
  or safety envelope.
- **A crouch on a staircase is a fall.** The 10-minute low-battery warning is the
  entire reaction budget on stairs.
- ⚠️ **There is no report anywhere of a Go2 EDU descending stairs carrying 8 kg
  with a person present.** That is the case we would need.

---

## Lifts

### Evidence

| Claim | Class | Source |
|---|---|---|
| **No Johnson public API, SDK, or developer portal exists** | VERIFIED (negative) | Johnson site and product pages |
| Johnson's **WATCH** IoT is monitoring-only, lift → vendor, with **no inbound command path** | VERIFIED | johnsonliftsltd.com; Times of India; Elevator World |
| **EyeRIS** — a QR-code contactless call system — is the only sanctioned interface found | VERIFIED | `https://www.johnsonliftsltd.com/eyeris/` |
| EyeRIS retrofit availability for older lifts | **NOT FOUND** | — |
| Whether the EyeRIS QR is passive or active | **NOT FOUND** | ⚠️ If active, fitting it is a modification |
| Whether EyeRIS can be provisioned to a non-human client | **NOT FOUND** | — |
| ⚠️ *"No additions or alterations… shall be made to any lift… installation except with the prior permission in writing of an Electrical Inspector (Lifts)"* | VERIFIED | Maharashtra Lifts Act 2017 s.10 |
| Delhi/Bombay Rules 6(v), 6(xv); Tamil Nadu Act s.15 — comparable prohibitions | VERIFIED | cited in `docs/shana/infrastructure/elevator_feasibility.md` |
| **The school, as occupier, was held 5% liable** for failure of oversight in a lift fatality | VERIFIED | *Otis Elevator Co. (India) v. Rashmi Handa*, Supreme Court of India, 29 July 2026 |
| Otis — the world's largest elevator company — **abandoned** button-actuator integration in favour of sanctioned APIs | VERIFIED | The Robot Report |

### Industry precedent

No commercial operator integrates with a lift without the service company and
the OEM in the loop. **Starship** — the closest analogue — does not use passenger
elevators at all; it asks a human to press the button.

---

## Decision

**Stairs and lifts are excluded from the initial pilot.**

The default assumption, if the relevant questions go unanswered, is that both
remain permanently out of scope.

---

## Reversal conditions

### For stairs

Reversed **only** if **all** of the following hold:

1. **Measured** riser heights on every candidate route are ≤160 mm
   (`SH-STAIR-001`)
2. Written **manufacturer** confirmation that the proposed use is permitted
   (`SH-STAIR-014`)
3. Written **school** approval for a stair test (`SH-STAIR-013`, `-016`)
4. Written **insurer** approval
5. A supervised, unloaded test completed **including forward descent**, with
   zero unrecovered falls
6. Even then, stairs are **descended unloaded only**

⚠️ **Criterion 1 alone may settle the matter.** If any required riser exceeds
160 mm, no further work is needed — stairs are outside the manufacturer envelope
before payload, before supervision, and before wear.

### For lifts

Reversed **only** if **all** of the following hold:

1. The manufacturer confirms in writing that **EyeRIS is retrofittable**, its QR
   is **passive**, and it **can be provisioned to a non-human client**
   (`SH-LIFT-014`, `-015`, `-016`)
2. The school authorises the approach in writing
3. Electrical Inspector permission is obtained where State rules require it
4. Maintenance contract compatibility is confirmed
5. Insurance and certification impact are confirmed
6. An approved fault and emergency workflow exists for a robot inside a car
7. A **human escort protocol** is agreed and staffed for every trip

⚠️ **If the manufacturer answers "no", this becomes permanent.** Human-assisted
operation is what every comparable operator has converged on, and it is an
acceptable permanent design.

---

## Consequences

**Accepted:** a single-floor system. Cross-block reach depends on a verified
ground-floor route existing.

**Rejected:** any plan assuming autonomous vertical transit, and any plan
requiring a physical lift modification.

⚠️ **Both exclusions are reversible with evidence, and both are cheap to keep
for now.** The alternative — designing for vertical transit before it is
established — would put an unverified capability at the centre of the entire
project.