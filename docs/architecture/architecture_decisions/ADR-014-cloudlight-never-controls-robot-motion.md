# ADR-014: CloudLight never controls robot motion

**Status:** Accepted
**Date:** 2026-10-11
**Decides:** The maximum capability CloudLight may ever have over SHANA
**Reversal condition:** None. This extends ADR-001 to the new subsystem and is
**not reversible by the CloudLight team**

---

## Context

CloudLight is being designed as a platform that includes a request interface to
the SHANA robot subsystem. It is therefore necessary to state, explicitly and in
advance, the boundary — before any integration code exists, because a boundary
defined after a capability exists is a negotiation rather than a constraint.

The physical stakes are asymmetric. A software defect in a timetable display is an
inconvenience. A software defect on the path to a motor command is a 15 kg
machine with twelve moving legs in an occupied school corridor.

## Decision

> **CloudLight may request a delivery. CloudLight may never command the robot.**

Concretely, CloudLight shall have **no mechanism whatsoever** to express:

- motor commands · gait selection · velocity or acceleration
- waypoints, paths, checkpoints or route segments
- localisation, map corrections or floor identity
- battery thresholds, charging or docking
- obstacle avoidance, hazard suppression or sensor behaviour
- emergency stop, safe stop or recovery behaviour

⚠️ The boundary is expressed as an **absence of capability**, not as a policy. A
low-level command path that exists "only for debugging" or "only for the
exhibition" converts a software defect into physical motion.

SHANA retains, without exception and without delegation: navigation,
localisation, route safety, battery reserve, hazard handling, safe stopping,
emergency behaviour and physical delivery execution. CloudLight must offer **no
fallback that could degrade any of them**.

## Evidence

| Claim | Class | Source |
|---|---|---|
| No AI model may reach the robot | VERIFIED (accepted decision) | `ADR-001-no-ai-reaches-the-robot.md` |
| Teachers never receive motor-control access | VERIFIED (accepted decision) | `ADR-001`; project ground rules |
| Loss of communications produces a defined safe behaviour | VERIFIED (accepted decision) | SHANA ground rules |
| An unauthenticated task endpoint lets any host on the network move the robot | PLAUSIBLE_TEST | Standard service-security reasoning |
| A duplicate request produces a duplicate physical task | PLAUSIBLE_TEST | Standard idempotency reasoning |
| The actual SHANA interface available to us | UNKNOWN | `CL-SHANA-001` — **REQUIRES_VENDOR** |

## Consequences

**Easier:**

- The worst consequence of a full CloudLight compromise is an unauthorised
  *delivery request* — serious, but bounded. It is not motion.
- CloudLight can be reasoned about, tested and deployed without any robot
  present.
- The safety case lives entirely in SHANA, where it can be argued and audited.

**Harder:**

- CloudLight cannot offer features a user might expect, such as "send it there
  now" urgency or route preferences. This is intentional.
- The integration requires mutual authentication and idempotency from day one,
  which is more work than a naive endpoint.
- A product limitation must be explained to users rather than engineered around.

## Alternatives considered

**Give CloudLight read access to robot telemetry.** Rejected. Telemetry is
SHANA's to expose, and read access blurs the boundary toward write access.

**Let CloudLight request urgency or priority.** Rejected. Any priority concept
invites a future priority that SHANA must honour over its own safety rules.

**Allow a maintenance/debug channel.** Rejected. It is the mechanism by which
every such boundary is eventually crossed.

## What this decision does NOT settle

- The interface itself. `CL-SHANA-001`
- SHANA's behaviour when CloudLight is unreachable — **SHANA decides**, not us.
- Cancellation semantics. `CL-SHANA-013`, `CL-SHANA-014`

See [`docs/cloudlight/shana_integration.md`](../../cloudlight/shana_integration.md) §1 and §10.