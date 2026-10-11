# SHANA — Safety Case

**Status: DRAFT. No safety claim in this document has been demonstrated,
verified, or approved.**

---

## What a safety case is

A safety case is an argument that a system is acceptably safe, supported by
evidence. **This document is the skeleton of that argument. The evidence does not
yet exist.**

⚠️ **It would be dishonest to present this as a completed safety case.** The
gates in `docs/shana/feasibility/go_no_go_gates.md` exist precisely because the
evidence has not been gathered.

---

## The top-level claim

> **SHANA is acceptably safe to operate in a school among students, under active
> human supervision, on a single verified floor, at reduced speed, within a
> physically enforced exclusion zone, carrying only approved payloads, with a
> tested emergency stop available at all times.**

⚠️ **Every clause of that claim is currently unevidenced.**

Note what the claim contains: supervised, single floor, reduced speed,
exclusion zone, approved payloads. **Those are not concessions — they are the
design.**

---

## Manufacturer constraints we adopt as requirements

✅ VERIFIED, Go2 User Manual:

> *"Hand may be **pinched at the motion joints**, such as at the knee joint."*

> *"Please keep it under control in the users' view… and keep the robot maintain a
> **safe distance of at least 2 meters** from obstacles, complex ground, crowds,
> water, and other objects."*

> *"**This product is not a toy and is not intended for use by persons under the
> age of 14.** Keep out of reach of children and be careful when operating in the
> presence of children."*

> *"it is **forbidden** to use the robot on the occasions where the terrain is
> complicated, the ground is wet… steps higher than 16cm…"*

Additional verified behaviour: **at low battery the robot crouches within 10
minutes.**

⚠️ **We adopt every one of these as a design requirement, not merely as guidance.**
The manufacturer has described the environment this robot is intended for, and
we are designing within it.

---

## The safety argument

```
CLAIM 1: The robot cannot move to an unverified location
  ├── Gate 5: every destination physically walked and approved
  ├── Gate 5: every route segment authorised by a named person
  ├── Design: destinations are allow-listed; free text never accepted
  └── Design: relocation events invalidate the pose and force re-localisation
        Evidence: NONE YET — Gate 5

CLAIM 2: The robot moves slowly
  ├── Gate 9: approved maximum speed
  ├── Design: software clamps all velocity to the cap
  ├── Design: the speed cap cannot be overridden by any interface
  └── Manufacturer: rated to 3.7 m/s; we operate far below that
        Evidence: NONE YET — Gate 9

CLAIM 3: People stay away from the robot
  ├── Gate 8: physically enforced exclusion zone, tested
  ├── Gate 0: named supervisor with unilateral stop authority
  ├── Gate 0: no operation without a supervisor present
  └── Manufacturer: 2 m minimum distance from crowds
        Evidence: NONE YET — Gates 0, 8

CLAIM 4: The robot can always be stopped
  ├── Gate 8: emergency stop physically tested
  ├── Gate 8: remote control authority tested before every run
  ├── Gate 8: hard power reachable within seconds
  └── Gate 8: dashboard stop for administrators
        ⚠️ Five independent stop methods; one start method
        Evidence: NONE YET — Gate 8

CLAIM 5: The robot cannot do dangerous things to the payload or environment
  ├── Gate 7: approved item list with absolute prohibitions
  ├── Gate 7: corridor hazard states, treated conservatively
  ├── Gate 7: no automatic resumption after a closure
  ├── Design: UNKNOWN is never treated as AVAILABLE
  └── Design: cameras are advisory; humans declare hazards
        Evidence: NONE YET — Gate 7

CLAIM 6: The robot never acts on unvalidated input
  ├── Deterministic validator; unknown entities rejected
  ├── Human confirmation for every model-derived task
  ├── Teacher roles carry no motor control
  ├── Safety supervisor independently vetoes
  └── The demonstration works with the LLM disabled
        Evidence: NONE YET — not implemented

CLAIM 7: Failures are safe by default
  ├── Loss of comms → hold position
  ├── Localisation loss → stop
  ├── Low battery → refuse tasks, recovery or hold
  ├── Sensor failure → UNKNOWN, never clear
  └── No route available → stop and request human assistance
        Evidence: NONE YET — not implemented
```

---

## The five controls that carry the argument

| # | Control | Why it is load-bearing | Status |
|---|---|---|---|
| 1 | **Human supervisor with unilateral stop** | The only control that does not depend on any technology | ⚠️ Not established |
| 2 | **Physical exclusion zone** | Distance alone is insufficient; the space must be kept empty | ⚠️ Not established |
| 3 | **Speed cap enforced in software** | Operator discipline is not a control | ⚠️ Not implemented |
| 4 | **Approved item and destination lists** | Bounds what the robot can do to people and property | ⚠️ Not agreed |
| 5 | **Conservative failure default** | Every automatic failure degrades to "stop and ask" | ⚠️ Not implemented |

⚠️ **The first two are physical and organisational. No amount of software can
substitute for them.** That is a deliberate design choice: the controls that
matter most are the ones that cannot fail silently.

---

## Hazard categories

| Category | Principal hazard | Primary control |
|---|---|---|
| **Mobility** | Collision, tripping over legs | Speed cap, exclusion zone, supervision |
| **Payload** | Falling or shifting objects | Two restraints, tilt test, approved items, ≤2 kg |
| **Balance/fall** | Robot falls | ⚠️ Go2 self-recovers; G1 could not |
| **Vertical** | Stair and lift transit | ⛔ **Excluded from pilot** |
| **Environmental** | Wet floor, damage, poor light | Human-declared hazards; conservative default |
| **Infrastructure** | Closed corridor, construction | Closure mechanism; no auto-resumption |
| **System** | Comms loss, server crash, sensor failure | Hold position; physical backstops |
| **Human** | Student interference, tampering | Spotter; unilateral stop; response procedure |
| **Battery** | Thermal event, depletion | Temperature limits; supervised charging; quarantine |
| **Privacy** | Data misuse | ⛔ No identification; no raw video; no automated enforcement |

Full analysis: `docs/shana/safety/hazard_analysis.md` and `docs/shana/safety/risk_register.md`.

---

## Safety invariants

Testable properties that must hold at all times.

| # | Invariant |
|---|---|
| S-01 | The robot never moves unless a human supervisor is present and on station |
| S-02 | The robot never moves outside a physically enforced exclusion zone |
| S-03 | The robot never exceeds the approved speed, regardless of interface |
| S-04 | The robot never moves to a location not verified by a named person |
| S-05 | The robot never carries an unapproved item |
| S-06 | The robot never moves while a corridor is `HAZARD`, `CLOSED`, or `UNKNOWN` |
| S-07 | The robot always holds position when it cannot determine where it is |
| S-08 | The robot can always be stopped without the software |
| S-09 | No delivery completes without explicit recipient confirmation |
| S-10 | Every operation, fault, and near-miss is logged |
| S-11 | No AI output ever produces a motion command |
| S-12 | Any person may call for the robot to stop, without justification |

---

## What a completed safety case requires

| # | Requirement | Gate |
|---|---|---|
| 1 | Written authorisation and named owners | Gate 0 |
| 2 | Confirmed hardware and tested connection | Gate 1 |
| 3 | Permitted, isolated, tested network | Gate 2 |
| 4 | Privacy determination | Gate 3 |
| 5 | Vertical transit excluded or proven feasible | Gate 4 |
| 6 | Verified maps, routes, checkpoints | Gate 5 |
| 7 | Measured battery behaviour and tested safe stop | Gate 6 |
| 8 | Reviewed hazard and crowd design | Gate 7 |
| 9 | Approved controlled testing plan | Gate 8 |
| 10 | Approved pilot acceptance criteria | Gate 9 |
| 11 | Separate student-facing authorisation | Gate 10 |

**0 of 11 gates passed.**

---

## The honest statement

⚠️ **SHANA has no safety case today. It has a plan for building one.**

The plan requires a school to authorise a project, IT to build a network,
facilities to survey a building, a custodian to identify a robot, a manufacturer
to answer questions about lifts, and every one of those to have happened before
a single motor turns.

**That is not an obstacle to be overcome by engineering. It is the engineering.**

---

**Related:** `docs/shana/safety/risk_register.md` · `docs/shana/safety/hazard_analysis.md` ·
`docs/shana/safety/emergency_procedures.md` · `docs/shana/safety/operating_restrictions.md` ·
`docs/shana/feasibility/go_no_go_gates.md`