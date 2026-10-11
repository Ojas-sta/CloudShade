# SHANA — Elevator Integration

**Status: RESEARCH ONLY. No lift has been approached. No vendor or service
provider has been contacted.**

> ## ⛔ ABSOLUTE CONSTRAINT
>
> **No modification to a lift is ever permitted.** No panel opening. No wiring. No
> servo. No button actuator. No interlock bypass. No shaft, pit, or car-top
> entry.
>
> Any such request is out of scope and must not be made.

---

## What the school reported

Two lifts, described as "Johnson elevators" — one in C Block, one in B Block.
**Model, controller, interfaces, installation date, and service arrangements are
all unconfirmed.**

---

## Manufacturer identity — a correction

Two distinct entities exist:

| Entity | Detail |
|---|---|
| **Johnson Lifts Pvt. Ltd.** | Chennai, established 1963, 80+ branches, 205+ service centres |
| **Toshiba Johnson Elevators (India) Pvt. Ltd.** | Joint venture — Toshiba 51% / Johnson 49% |

⚠️ **Determine which nameplate is on the installed equipment.** The integration
counterparty differs.

⚠️ **A common error to avoid:** *"SIRIO"* is **not** a Johnson product — it is an
Italian hydraulic home lift by Extrema. The Toshiba JV's destination control
system is called **FLOORNAVI**.

---

## Which standard applies

⚠️ **The governing Indian family is IS 17900 (2022 series)**, which Johnson itself
cites.

| Standard | What it actually is |
|---|---|
| **IS 17900 (2022 series)** | ✅ **The lift standard family.** Part 1 safety rules; Part 4 control devices and buttons; Part 6 maintenance; Part 11 safety norms for existing lifts |
| IS 14447 | ❌ *Refractories for use in foundry industry* — **not** a lift standard |
| IS 6914 | ❌ *Cast billets ingots* — **withdrawn**, superseded by IS 2830 |

⚠️ **Confirm with the service provider which standard the specific installed lifts
were built to** (`SH-LIFT-002`).

---

## What Johnson actually offers

### ✅ EyeRIS — the only sanctioned path found

A contactless call-booking system. Official site, verbatim:

> *"Once the Johnson EyeRIS app is downloaded, it connects the smartphone to the
> lift system using QR code technology and the user operates the lift by just
> tapping the buttons on the phone's screen. The user will only need to indicate
> the 'Present Floor' and the 'Destination Floor' on the app and once done, the
> lift arrives to take the user to the destination floor."*

> *"Just place the mobile in front of the QR code available in the Landing
> Operating Panels… First select 'present floor' and then press 'destination
> floor'."*

**Why this matters:** a manufacturer-sanctioned client that issues both hall and
destination calls, with **no panel modification and no wiring.**

⚠️ **Three unknowns — all P0:**

| # | Question | Why it matters |
|---|---|---|
| 1 | Is EyeRIS retrofittable to lifts installed earlier? | Determines whether it exists on these lifts |
| 2 | Is the QR element a **passive label** or an **active component**? | ⚠️ If active, fitting it **is** a modification |
| 3 | Can it be provisioned to a **non-human client**? | Documented as authenticating a person's phone |

### ✅ WATCH — monitoring only

Lift → Johnson's IoT data centre → technicians. **No inbound third-party command
path found.** Monitoring is not control.

### ⛔ No public API

**No Johnson API, developer portal, SDK, or robot-integration product was
found.** High-confidence negative.

---

## Industry alternatives — vendor-locked

| Vendor | Product | Applicability |
|---|---|---|
| Otis | Otis Integrated Dispatch™ | ❌ Vendor-locked |
| TK Elevator | MAX Robot Calls API | ❌ Vendor-locked |
| Schindler | BuilT-In Robot Service Connection | ❌ Vendor-locked |

> **Otis:** *"Our technology, obviously, only works with Otis equipment."*

⚠️ **There is no cross-vendor elevator-robot standard.** A Johnson lift is outside
all three.

---

## The statutory position

**Maharashtra Lifts, Escalators and Moving Walks Act, 2017, Section 10:**

> *"No additions or alterations, which cause any change in the characteristics of
> lift or escalator or moving walk like change in passenger capacity, speed,
> stops, steps, etc. … shall be made to any lift or escalator or moving walk
> installation **except with the prior permission in writing of an Electrical
> Inspector (Lifts)**."*

**Delhi / Bombay Lift Rules, Rule 6**, cited as binding by the Supreme Court in
2026: *"The owner shall not carry out any additions or alternations to an
installed Lift…"* and *"No person shall willfully interfere with any mechanism…"*

**Comparable provisions** exist in the Tamil Nadu (1997), Karnataka (2012) and
Haryana (2008) Acts. ⚠️ **The school's State Act has not been identified.**

---

## The liability finding

***Otis Elevator Co. (India) v. Rashmi Handa***, Supreme Court of India,
**29 July 2026**. An officer died when MES personnel manually released the
mechanical brake during a rescue from a lift stalled between floors.

> *"A party that undertakes the comprehensive maintenance of a machine, which is
> in the nature of a vehicle, **owes a heightened duty of care towards its
> users**."*

Liability apportioned **70% Otis / 25% MES / 5% the occupier**.

**Implications:**

1. ⚠️ **Unauthorised third-party intervention does not exonerate anyone.**
2. ⚠️ **The school, as occupier, was held liable — for failure of oversight.**
3. ⚠️ **A robot tampering with a panel would be far more egregious than a human
   rescue reflex** — it is planned, repeatable, and not an emergency.
4. The manufacturer's "do not interfere" stickers were held **insufficient** as a
   safety measure alone.

---

## Physical reality — fit is not the problem

| Dimension | Robot | School lift | Clearance |
|---|---|---|---|
| Width | 310 mm | ~900 mm door | ~3× |
| Length | 700 mm standing / 760 crouched | ~1400 mm | ~2× |
| Mass | 15 kg | ~544 kg rated | 2.8% |
| Step | 160 mm | ±10 mm levelling | Not a barrier |

⚠️ **The genuine physical problem:** a quadruped must **turn around inside the car**
to exit facing the door. A real manoeuvre-planning problem, shared with wheeled
robots.

⚠️ **Actual car dimensions, door opening and sill gap are NOT FOUND.** Measure
on site with a tape.

---

## The defining risk

> **A lift stops between floors with the robot inside.**

This is the *Otis v. Rashmi Handa* scenario exactly. **The robot's battery is dead
in ≤4 hours. A mechanic's rescue will take far longer. The robot must be assumed
dead by the time help arrives.**

⚠️ **This risk is not technical. It is procedural. The defence is a named human
escort on every trip.**

---

## The human-escort protocol

```
1. Robot escorted by a NAMED adult to the lift lobby
2. Robot waits in a marked box, clear of the door swing
3. Human presses the hall call
4. Human verifies the car is EMPTY — if occupied, ABORT (do not share)
5. Robot enters, centres, rotates to face the door
6. Human presses the destination button
7. Human STAYS IN THE CAR and rides with the robot
8. Human confirms the car is level and stationary before the robot exits
9. Human exits last and confirms the robot is clear
```

### Failure modes

| # | Failure | Consequence |
|---|---|---|
| 1 | **Lift stalls with robot inside** | The defining risk. Mechanical rescue |
| 2 | **Fire alarm → Phase I recall** | Car recalled, lift removed from service; robot may block a landing door |
| 3 | **Doors close on the robot** | Reversal or impact |
| 4 | **Supervisor walks away or is distracted** | ⚠️ **The dominant real-world failure** |
| 5 | **Student crowd** | Occupancy management needed |
| 6 | **Wrong floor pressed** | Caught by the human present — a benefit of the protocol |
| 7 | **Lift out of service** | Documented abort; restrict to single floor |

---

## The button-actuator question

⚠️ **We ask what the manufacturer permits. We do not assume permission, and we do
not build one.**

Academic work uses a servo pressed over the panel. Even that *"non-invasive"
technique is a prohibited addition to the installation* under the statutory
provisions above. Academics get away with it by owning the building; **a school
does not have that position.**

⚠️ **Otis — the world's largest elevator company — abandoned button-actuator
integration in favour of sanctioned APIs.** The industry moved away from this
approach.

**If an approved external actuator is not supported, the alternatives are:**

- (a) **Human-assisted operation** — a person presses the buttons
- (b) **Staff-controlled lift calls**, on request
- (c) **Manufacturer-supported integration**, if one is ever offered
- (d) **Avoiding vertical transitions** in the initial pilot ✅ *recommended*

---

## Industry precedent — unanimous

| Company | Behaviour |
|---|---|
| **Relay** | Autonomous, **requires** service company + OEM involvement, written proposal. Does **not** modify the lift. **Johnson not supported** |
| **Otis / TK / Schindler** | Sanctioned APIs, vendor-locked |
| **Starship** | Sidewalk-only. **Does not use passenger elevators.** *"if you could, please press the button"* |

⚠️ **No commercial operator anywhere modifies an elevator without the service
company and the OEM in the loop.**

---

## Questions

Full request: `docs/shana/administration/technical_assessment_request.md` Part 1.

| ID | Question | Priority |
|---|---|---|
| `SH-LIFT-001` | Exact manufacturer and model | **P0** |
| `SH-LIFT-002` | Which Indian standard? | **P0** |
| `SH-LIFT-005` to `-008` | Car dimensions, sill, load, network | P1 |
| `SH-LIFT-009` | **Any sanctioned third-party or robot call interface?** | **P0** |
| `SH-LIFT-014` | **Is EyeRIS retrofittable?** | **P0** |
| `SH-LIFT-015` | **Is the QR passive or active?** | **P0** |
| `SH-LIFT-016` | **Can EyeRIS be provisioned to a robot client?** | **P0** |
| `SH-LIFT-019` | **Is a physical button actuator permitted?** | **P0 — expected answer: no** |
| `SH-LIFT-021` | Is unattended robot travel permitted? | **P0** |
| `SH-LIFT-025` | Approved fault workflow if the lift stalls with the robot inside | **P0** |
| `SH-LIFT-026` | Insurance, warranty, certification impact | P1 |

---

⚠️ **Default position: human escort, permanently. This is not a failure state —
it is what every comparable operator does.**