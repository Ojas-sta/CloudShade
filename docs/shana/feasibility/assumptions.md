# SHANA — Assumptions Register

**Every assumption the project is currently running on. None is verified.**

An assumption is a statement we are treating as true so that work can proceed,
when in fact it is not yet established. Each one below carries the question that
would confirm or refute it, and a **reversal condition** — what happens if it
turns out to be false.

⚠️ **An assumption that is never written down becomes an assumption that is
never tested.** This register exists so that cannot happen here.

---

## Class A — Assumptions about the robot

| ID | Assumption | Evidence class | Confirms it | If false |
|---|---|---|---|---|
| A-01 | The school robot is an EDU variant | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | `SH-GO2-001` | ⛔ **Project impossible.** Only EDU-and-above may be programmed |
| A-02 | Firmware is ≥ 1.1.6 | `DOCUMENTED_NOT_HARDWARE_VERIFIED` | `SH-GO2-002` | `SwitchGait` still exists; stair question changes |
| A-03 | The 15,000 mAh battery is fitted | `UNKNOWN` | `SH-GO2-006` | Endurance budget **halves** to ~1–2 h |
| A-04 | An expansion dock is present | `UNKNOWN` | `SH-GO2-003` | Vendor navigation impossible; self-built path is mandatory |
| A-05 | A depth camera is fitted | `UNKNOWN` | `SH-GO2-004` | Stair-edge and low-obstacle perception reduced |
| A-06 | A Linux host is available for development | `UNKNOWN` | `SH-GO2-011` | Development blocked until one is provisioned |
| A-07 | The robot's published 16 cm step limit is accurate for our unit | `DOC_NOT_HW_VERIFIED` | `SH-STAIR-001` | Stairs even less feasible |
| A-08 | No autonomous return-to-dock exists | `VERIFIED (negative)` | `SH-BAT-022` | Recovery design changes |
| A-09 | Battery telemetry is on `rt/lowstate` | `VERIFIED` | `SH-GO2-019` | — |
| A-10 | The robot self-recovers from minor trips | `PLAUSIBLE_TEST` | `SH-GO2-021` | Manual recovery becomes routine |

⚠️ **A-03 is the most expensive assumption.** It changes the entire endurance
budget and it is resolved by reading a label.

---

## Class B — Assumptions about the school

| ID | Assumption | Evidence class | Confirms it | If false |
|---|---|---|---|---|
| B-01 | The school will authorise development access | `UNKNOWN` | `SH-GO2-008` | ⛔ **Project impossible** |
| B-02 | A room will be approved for the server | `UNKNOWN` | `SH-SRV-001` | Server location unresolved |
| B-03 | A dedicated network segment will be permitted | `UNKNOWN` | `SH-NET-003` | Fallback: direct cable, no school network change |
| B-04 | Wi-Fi covers all floors and blocks | `UNKNOWN` | `SH-NET-008` | Coverage gaps; possibly justifies radio telemetry |
| B-05 | A single-floor route to most needed rooms exists | `UNKNOWN` | `SH-STAIR-012` | ⛔ Vertical transit becomes mandatory, and unproven |
| B-06 | Stair risers are ≤160 mm | `UNKNOWN` | `SH-STAIR-001` | ⛔ **Stairs permanently out of scope** |
| B-07 | The Data Fiduciary will approve the project in principle | `UNKNOWN` | `SH-PRIV-002` | Data-dependent features closed |
| B-08 | Teachers will actually use the system | `UNKNOWN` | `SH-DEL-017` | No product, only a demonstration |
| B-09 | Budget exists | `UNKNOWN` | `SH-BUD-001` | Scope constrained to existing hardware only |
| B-10 | Insurance will cover robot operation | `UNKNOWN` | `SH-BUD-020` | ⛔ Operation may be impossible |

---

## Class C — Assumptions about school infrastructure

| ID | Assumption | Evidence class | Confirms it | If false |
|---|---|---|---|---|
| C-01 | No VLAN or client isolation blocks the robot subnet | `UNKNOWN` | `SH-NET-005` | Connection fails silently; must be diagnosed |
| C-02 | UDP multicast is permitted on the robot subnet | `UNKNOWN` | `SH-NET-007` | ⛔ Robot link fails entirely |
| C-03 | A local server is reachable by teacher devices | `UNKNOWN` | `SH-NET-006` | Dashboard unusable |
| C-04 | The school permits no-internet local operation | `UNKNOWN` | `SH-NET-015` | Design conflict |
| C-05 | CCTV analytics would be permitted if requested | `UNKNOWN` | `SH-CCTV-016` | Occupancy-by-sensing is closed; **manual occupancy becomes mandatory** |
| C-06 | Any exemption might cover corridor-routing analytics | `UNKNOWN` | `SH-PRIV-008` | Legal exposure; feature must be excluded |
| C-07 | The lift has no sanctioned third-party interface | `PLAUSIBLE_TEST` | `SH-LIFT-009` | If one exists, Stage 3 changes materially |
| C-08 | EyeRIS cannot be provisioned to a robot client | `PLAUSIBLE_TEST` | `SH-LIFT-016` | If it can, a sanctioned path exists |

⚠️ **C-02 deserves emphasis.** Multicast discovery is required by the robot's
protocol and is frequently blocked by default on school networks. Discovering
this on demonstration day would stop the project.

---

## Class D — Assumptions about our own design

| ID | Assumption | Evidence class | Confirms it | If false |
|---|---|---|---|---|
| D-01 | Self-built navigation is feasible on the built-in LiDAR | `PLAUSIBLE_TEST` | `SH-GO2-013` | Vendor path or dock purchase required |
| D-02 | An ArUco marker per floor solves floor identification | `PLAUSIBLE_TEST` | `SH-MAP-016` | Single-floor only |
| D-03 | Manual corridor-status input is sufficient for the pilot | `PLAUSIBLE_TEST` | `SH-DEC-011` | Automation required, with all its privacy burden |
| D-04 | The whole stack can be built in simulation first | `VERIFIED` | `SH-GO2-016` | — |
| D-05 | A deterministic validator can be complete and auditable | `PLAUSIBLE_TEST` | `SH-DEC-013` | Safety architecture needs redesign |
| D-06 | Spotter presence is enforceable in practice | `PLAUSIBLE_TEST` | `SH-SAFE-005` | ⛔ Unsupervised risk; operations must be suspended |

---

## Class E — Assumptions we have explicitly rejected

Recorded so they are not silently reintroduced.

| Rejected assumption | Why it is rejected |
|---|---|
| "The robot can identify which floor it is on" | ✅ **Verified negative.** No altitude, pressure, or barometer field exists anywhere in the Go2 message set. |
| "The robot can reach stairs or lifts autonomously" | Neither established nor permitted. |
| "Cameras can determine whether a floor is safe" | RGB and depth cannot determine friction. Visual detection of water is not proof of safety. |
| "A maintenance app means remote control is available" | A maintenance application is not a third-party control API. |
| "Occupying a busy corridor means an alternate route exists" | Not established. Without a verified alternate, a busy corridor is a hard block. |
| "Radio telemetry is more reliable than Wi-Fi indoors" | Peer-reviewed evidence points the other way. Wi-Fi with one AP per floor outperforms LoRa for coverage. |
| "More sensors means more safety" | Every added sensor is a new failure mode. Independence is worth more than density. |
| "A higher platform score means a better deliverable" | Demonstrations are judged on reliability and honesty. |

---

## Assumption review triggers

Re-open this register when any of the following occurs:

- A `P0` question is answered
- A gate changes status
- A measurement differs from an assumption by more than 10%
- The project scope changes
- A supplier, vendor, or authority changes its position
- Any assumption is more than 30 days old and still unverified

⚠️ **An assumption older than 30 days and still unverified should be treated as
UNKNOWN, not as a working assumption.** If the project cannot function without
an answer, the answer must be obtained.

---

## Summary

| Class | Count | Verified | Unverified |
|---|---|---|---|
| A — Robot | 10 | 3 | 7 |
| B — School | 10 | 0 | 10 |
| C — Infrastructure | 8 | 0 | 8 |
| D — Own design | 6 | 1 | 5 |
| **Total** | **34** | **4** | **30** |

⚠️ **30 of 34 assumptions are unverified, and not one of the school-side
assumptions has been tested.** That is the accurate state of the project at the
end of the feasibility phase. It is not a criticism of the work — it is the
reason the work was done.