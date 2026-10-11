# ADR-004: The robot-to-server link is wired Ethernet

**Status:** `Accepted`
**Date:** 2026-10-10
**Decides:** The physical and data link between the robot and the application
server
**Reversal condition:** Reversed if **all** hold: (1) the school confirms a
sanctioned wireless path for a safety-relevant control link, (2) measured wireless
latency and packet loss under school load satisfy the abort budget, and (3) the
safety case shows the system remains fail-safe on link loss with no wireless path
at all. ⚠️ Condition 1 is unlikely to be met; wireless is unlikely ever to be
sanctioned for a control link in a school.
**Evidence:** `docs/shana/feasibility/decision_log.md` `D-003` ·
`docs/shana/hardware/network_interfaces.md`

---

## Context

The robot must exchange state with a server: task assignment, status, telemetry,
and stop commands. That link is safety-relevant — a stop command traverses it.

Wireless is the default expectation for a mobile robot. A tethered robot is
operationally awkward and may be rejected on that basis alone. The decision must
therefore be made on evidence rather than convenience.

---

## Decision

**🔴 The robot-to-server link is wired Ethernet, using Cyclone DDS.**

| # | Rule |
|---|---|
| 1 | The control link is **cabled**. Wireless is not used as the primary path |
| 2 | Wi-Fi may carry **non-safety** traffic (telemetry for dashboards) |
| 3 | 🔴 Loss of the link produces the defined safe behaviour (`D-008`), never a best-effort continuation |
| 4 | The radio link, if any, is independent of the control link — it cannot carry it |

---

## Rationale

**Why determinism matters more than mobility here.** A stop command that arrives
late is a stop command that failed. Predictable latency, predictable delivery, and
a link whose failure mode is observable are worth more than untethered operation.

**Why wireless is a poor safety path in a school.** School WLAN is shared,
managed for office use, and subject to policy. Interference, roaming, and
authentication expiry are routine. None of these are acceptable on a control path
for a robot among children.

**Why rule 2 exists.** Excluding wireless entirely would be overreach. Telemetry
for a dashboard has no safety consequence and tolerates loss.

**Why rule 3 is the important one.** Whatever the link, losing it must produce a
defined safe behaviour. The link choice matters less than this property.

⚠️ **Operational consequence, stated plainly:** a cabled robot trails a cable, or
the school accepts a cable-free design that must be agreed with facilities.
⚠️ **This has never been discussed with the school** and may be a decisive
practical objection.

---

## Evidence

| Fact | Class | Source |
|---|---|---|
| Cyclone DDS is the Go2 interface | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | `hardware/sdk_and_interfaces.md` |
| School network policy and IP plan | `UNKNOWN` | `infrastructure/school_network.md` |
| Latency/jitter budget under load | `UNKNOWN` | ⚠️ **never measured** |
| 🔴 No link exists, configured or tested | `VERIFIED` | repository audit 2026-10-11 |

---

## Consequences

**Accepted** — predictable link behaviour; observable failure; no WLAN dependency
on the safety path.

**Rejected** — Wi-Fi as the control link; any design assuming the robot is
unobstructed by a cable.

⚠️ **Cable management is a real operational cost** and must be presented to the
school explicitly, not discovered later.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Wi-Fi as the control link | Non-deterministic latency on a shared school network, on a safety-relevant path. |
| LoRa as the control link | ~350-byte heartbeat at SF9 every 60 s. Telemetry-grade, not control-grade. Also deferred (ADR-012). |
| Ethernet with a wireless fallback for control | 🔴 A fallback that is *less* safe than the primary invites using it. Rejected. |

---

## Related

- [`../../hardware/network_interfaces.md`](../../hardware/network_interfaces.md)
- [`ADR-012`](ADR-012-radio-telemetry-deferred.md)
- [`../../feasibility/decision_log.md`](../../feasibility/decision_log.md) — `D-003`, `D-008`