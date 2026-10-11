# ADR-008: No lift modification, ever

**Status:** `Accepted`
**Date:** 2026-10-10
**Decides:** Whether any physical modification to a lift installation may be made
to enable robotic operation
**Reversal condition:** 🔴 **None.** This decision is not reversible by any
technical, commercial or schedule development. Reversal would require a change to
the law itself, or a sanctioned manufacturer interface — which is not the same
thing as a reversal of this ADR. 🔴 It is a legal and ethical constraint, not an
engineering preference.
**Evidence:** `docs/shana/feasibility/decision_log.md` `D-015` ·
`docs/shana/infrastructure/elevator_feasibility.md` ·
[`ADR-003`](ADR-003-stairs-and-lifts-excluded.md)

---

## Context

To reach destinations on other floors, the robot must use a lift. Several
"non-invasive" academic techniques circulate: pressing a call button with a
servo, bridging a contact, or simulating a QR-scan at the floor indicator.

Each is a physical alteration to a lift installation.

---

## Decision

🔴 **No lift modification. Ever.**

Not a servo. Not a button actuator. Not a contact bridge. Not a QR spoof.
Not "just for one test".

| # | Rule |
|---|---|
| 1 | 🔴 No physical addition to or alteration of any lift installation |
| 2 | 🔴 No modification "temporarily", "for a test", or "reversible afterwards" |
| 3 | Only a **sanctioned, manufacturer-provided, non-human-provisionable** interface may be considered |

---

## Evidence

| Fact | Class | Source |
|---|---|---|
| *"No additions or alterations… shall be made to any lift… installation except with the prior permission in writing of an Electrical Inspector (Lifts)"* | `VERIFIED` | Maharashtra Lifts Act 2017 s.10 |
| Comparable prohibitions | `VERIFIED` | Delhi/Bombay Rules 6(v), 6(xv); Tamil Nadu Act s.15 |
| The school, as occupier, was held **5 % liable** for failure of oversight in a lift fatality | `VERIFIED` | *Otis Elevator Co. (India) v. Rashmi Handa*, Supreme Court of India, 29 July 2026 |
| Otis — the world's largest elevator company — **abandoned** button-actuator integration in favour of sanctioned APIs | `VERIFIED` | The Robot Report |
| No Johnson public API, SDK, or developer portal exists | `VERIFIED` (negative) | `infrastructure/elevator_feasibility.md` |
| Johnson's WATCH IoT is **monitoring-only**, with no inbound command path | `VERIFIED` | same |
| **EyeRIS** (QR contactless call) is the only sanctioned interface found | `VERIFIED` | `infrastructure/elevator_feasibility.md` |
| Whether the EyeRIS QR is **passive**; whether it can be **provisioned to a non-human client** | `NOT FOUND` | ⚠️ if active, fitting it is a modification |

---

## Rationale

**Why "non-invasive" does not mean permitted.** A servo pressed over the panel
without touching wiring is still a **prohibited addition to the installation**. The
legislation is not written in terms of invasiveness.

**Why the industry precedent is decisive.** Otis — the world's largest elevator
company — moved *away* from button actuators. When the organisation with the most
to gain from automation abandoned the technique, the technique is not the safe
choice.

**Why the liability finding matters specifically here.** The school, as occupier,
was held liable in a lift fatality. Introducing a robot into a lift shifts
liability exposure onto exactly the party least able to bear it, for exactly the
marginal convenience of one subsystem.

**Why rule 2 exists.** "Temporary" and "just for a test" are the standard
justifications by which absolute rules erode. There is no test that justifies a
prohibited modification.

---

## Consequences

**Accepted** — multi-floor operation is excluded by default (ADR-003);
human-escorted lift operation is the only conceivable future path; zero legal
exposure from the robot's lift use.

**Rejected** — every non-sanctioned integration technique, permanently.

⚠️ **Honest consequence:** if the school requires multi-floor delivery, the answer
is that SHANA cannot deliver it, and no engineering work will change that.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Servo over call button, no wiring | 🔴 A prohibited addition. Non-invasive ≠ permitted. |
| Simulate the EyeRIS QR | ⛔ If the QR is active, fitting it is a modification. **Not established either way.** |
| Ask the school for Electrical Inspector permission | 🔴 Permission for a modification remains a modification. This ADR rules out the *activity*, not just the paperwork. |
| Human escort, operator presses the button | ⚠️ **Not rejected — this is the only acceptable future path.** See ADR-003. |

---

## Related

- [`../../infrastructure/elevator_feasibility.md`](../../infrastructure/elevator_feasibility.md)
- [`ADR-003`](ADR-003-stairs-and-lifts-excluded.md)
- [`../../feasibility/decision_log.md`](../../feasibility/decision_log.md) — `D-015`, `D-016`
- `AGENTS.md` §L.8 — 🔴 *"No lift modification. Ever."*