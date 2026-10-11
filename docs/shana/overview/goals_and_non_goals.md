# SHANA — Goals and Non-Goals

A project without explicit non-goals has no boundary. This document draws that
boundary.

---

## Goals

### G-01 — Deliver approved items safely and reliably
The primary objective. Approved items reach approved destinations, with the
recipient confirming receipt. **Success is measured, not asserted.**

### G-02 — Make the request workflow simple for teachers
A teacher should be able to request a delivery in under a minute, without
training, and understand what happened afterwards.

### G-03 — Make the system's state visible
When the robot, a request, or a fault is happening, a teacher or administrator
should be able to see it without asking anyone.

### G-04 — Never move somewhere unverified
The robot travels only on routes that a person has walked, measured, and
approved. It never explores.

### G-05 — Fail safely and visibly
Every failure produces a clear state, a logged reason, and a human notification.
The robot never fails silently.

### G-06 — Protect student privacy absolutely
No identification, no tracking, no automatic enforcement. This is a constraint,
not a feature.

### G-07 — Be honest about capability
Every published figure comes from measurement under a written protocol.
Failures are reported alongside successes.

### G-08 — Leave the school better documented than we found it
Measured routes, identified weak points, and a maintained questionnaire.

### G-09 — Be teachable
A student should be able to understand, operate, and extend what we built.

---

## Non-goals

### N-01 — Not a general-purpose robot
One platform, one payload type, a bounded set of destinations. Generality is the
enemy of reliability.

### N-02 — Not a surveillance system
No facial recognition. No identity recognition. No biometrics. No persistent
tracking. No automatic misconduct or attendance reporting. **No exception will be
made for convenience.**

### N-03 — Not a stair-climbing robot
Not in the pilot. Stair capability is not established for this hardware in this
building.

### N-04 — Not an autonomous lift user
No sanctioned third-party lift interface has been identified, and physical
modification is prohibited by law. Human-assisted operation is the design.

### N-05 — Not multi-floor
The robot has no sensor capable of determining which floor it is on. Multi-floor
operation requires added infrastructure that does not yet exist.

### N-06 — Not an object manipulator
There is no arm. SHANA carries; it does not pick up, open, pour, insert, or
rearrange.

### N-07 — Not dependent on the internet
Core operation runs entirely on a local school network. No internet, no cloud,
no external service.

### N-08 — Not publicly exposed
No component is reachable from outside the school network. No port forwarding.
No public hosting.

### N-09 — Not hands-off
A named, trained supervisor is present at every operation with unilateral stop
authority. This is not negotiable in any phase.

### N-10 — Not a replacement for human judgement
SHANA moves objects. It does not make decisions about people, and it must never
generate output that punishes, accuses, or embarrasses anyone.

### N-11 — Not fully autonomous in any sense
There is always a human who can stop it, a human who approved the route, and a
human who confirms the delivery.

### N-12 — Not a demonstration of cutting-edge robotics
It is a practical delivery system. Where the honest and impressive choices
conflict, the honest one wins.

---

## Explicitly out of scope for the initial pilot

| Excluded | Reason | Revisit when |
|---|---|---|
| Stairs | Not established; manufacturer forbids complex terrain | `SH-STAIR-012` answered and `SH-STAIR-020` reversed |
| Lifts | No sanctioned interface; modification prohibited | A manufacturer confirms a permitted path in writing |
| Multiple floors | No floor-identification sensor exists | ArUco floor markers installed and tested |
| Doors of any kind | Not tested, not required | A specific requirement arises |
| Cameras streaming to the dashboard | Not required for delivery | A specific requirement arises |
| Natural-language requests | Not required; adds risk without value at this stage | Manual workflow is proven |
| Any AI model controlling motion | Prohibited by design principle | **Never** |
| Teacher motor control | Prohibited by design principle | **Never** |
| CCTV analytics | Privacy determination not made | `SH-PRIV-008` resolves favourably |
| Radio telemetry | Not justified without a coverage gap | `SH-NET-009` identifies a gap |
| Outdoor routes | Robot is not weatherproof | **Never**, for this project |

---

## Constraints that are not negotiable

These are not goals, which can be traded off. They are constraints, which
cannot.

| # | Constraint |
|---|---|
| C-01 | No unsupervised operation |
| C-02 | No facial recognition or biometric identification |
| C-03 | No automatic misconduct, discipline, or attendance reporting |
| C-04 | No lift modification |
| C-05 | No movement to unverified locations |
| C-06 | No public network exposure |
| C-07 | No teacher access to motor control |
| C-08 | No AI model reaching the robot |
| C-09 | No credentials in the repository |
| C-10 | No delivery completed without explicit recipient confirmation |
| C-11 | No claimed performance number without measurement |
| C-12 | No gate passed without evidence |

---

## Success criteria

SHANA is successful if, at the end of the pilot:

1. A defined set of deliveries completed reliably, with **measured** completion
   rate and duration.
2. Every delivery required and received explicit recipient confirmation.
3. Every failure produced a logged reason and a human notification.
4. Zero incidents and zero unplanned interventions beyond the agreed baseline.
5. Teachers who use it report it as useful.
6. **Every published figure is traceable to measured data under a written
   protocol.**
7. The school has a documented route survey, a maintained questionnaire, and a
   clear record of what SHANA cannot yet do.

⚠️ **Criterion 6 and 7 matter as much as 1.** A system that performs well but
cannot be trusted, measured, or explained has not succeeded.

---

## What "done" means

**Not** "the robot can deliver things." That is a demonstration.

**Instead:** "the school has a measured, documented, safe, supervised delivery
system for one floor, a complete record of what was learned, and a clear
statement of what comes next and what is still not possible."