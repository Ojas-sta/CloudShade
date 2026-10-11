# ADR-012: Radio telemetry deferred

**Status:** `Provisional`
**Date:** 2026-10-10
**Decides:** Whether to build LoRa/ESP32 radio telemetry for the pilot
**Reversal condition:** Reversed if `NET-05` identifies a **specific room** with no
WLAN coverage that SHANA must operate in. 🔴 The decision rule is deliberately
narrow: **a room**, not a general wish for independence.
**Evidence:** `docs/shana/feasibility/decision_log.md` `D-017` ·
`docs/shana/hardware/esp32_and_lora.md` ·
`docs/shana/research/appendix_india_rf_research.md`

---

## Context

An ESP32 with a LoRa radio could provide a heartbeat independent of the school
WLAN — useful telemetry if the robot is in a part of the building with poor
coverage, and useful for battery monitoring off-network.

It is also additional hardware, additional regulatory exposure, and additional
failure surface for a project with no implementation.

---

## Decision

⚠️ **`Provisional`:** LoRa is **deferred**. It is not built for the pilot unless
a named condition triggers it.

| # | Rule |
|---|---|
| 1 | 🔴 Build Wi-Fi properly first |
| 2 | Add the LoRa heartbeat **only if** `NET-05` names a specific uncovered room |
| 3 | The heartbeat is **~350 bytes at SF9 every 60 s** — telemetry only, never a control path |
| 4 | It never carries the robot control link (ADR-004) |

---

## Evidence

| Fact | Class | Source |
|---|---|---|
| With one AP per floor, **Wi-Fi beats LoRa for coverage** in a 4-storey building | `PLAUSIBLE_REQUIRES_TESTING` | `decision_log.md` `D-017` |
| A peer-reviewed 16-storey study found LoRa **still lost the link across floors** | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | `esp32_and_lora.md` |
| LoRa buys **independence from the WLAN**, not range | `PLAUSIBLE_REQUIRES_TESTING` | `decision_log.md` `D-017` |
| The robot control link is **Ethernet** — a WLAN outage does not stop the robot | `VERIFIED` | ADR-004 |
| Indian RF frequency allocation and licensing constraints | `VERIFIED` | `research/appendix_india_rf_research.md` |
| 🔴 **No network has been surveyed**, so coverage is unknown | `VERIFIED` | `infrastructure/school_network.md` |

---

## Rationale

**Why independence from the WLAN is not the win it appears to be.** The
attractive argument is "the robot keeps reporting even if the network dies". But
the robot's *control* link is Ethernet (ADR-004), so a WLAN outage does not
degrade robot motion in the first place. The independence is being bought for
telemetry that already works.

**Why deferring is cheap and adding late is cheap.** The heartbeat is small,
periodic, and architecturally inert — it changes nothing about how the robot
behaves. This is exactly the kind of thing that can be added late at low cost,
which is the definition of a good thing to defer.

**Why the decision rule is deliberately narrow.** ⚠️ If the trigger were "we want
more resilience", the rule would fire immediately and the deferral would be
meaningless. Requiring a **named room** makes it evidence-driven.

---

## Consequences

**Accepted** — no additional hardware; no additional regulatory exposure; one
less unvalidated subsystem.

**Rejected** — LoRa as a control path; building it "just in case" without a named
coverage gap.

⚠️ **If `NET-05` names an uncovered room, this decision reverses on that evidence
alone** — no further debate required.

---

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Build LoRa now | No evidence of a coverage gap. Adds hardware, cost and regulatory exposure for telemetry that Wi-Fi already provides. |
| LoRa as control fallback | 🔴 Low bandwidth, non-deterministic latency. Control path is Ethernet (ADR-004). |
| Battery monitoring via LoRa only | 🔴 Battery telemetry is available on the robot's own interface (`D-011`). Remote monitoring is a convenience. |

---

## Related

- [`../../hardware/esp32_and_lora.md`](../../hardware/esp32_and_lora.md)
- [`../../research/appendix_india_rf_research.md`](../../research/appendix_india_rf_research.md)
- [`ADR-004`](ADR-004-wired-ethernet-robot-link.md)
- [`../../feasibility/decision_log.md`](../../feasibility/decision_log.md) — `D-017`
- [`../../handoff/unresolved_dependencies.md`](../../handoff/unresolved_dependencies.md) — `NET-05`