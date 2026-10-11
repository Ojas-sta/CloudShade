# Go2 EDU Inventory

🔴 **What the school robot actually is — and what is not confirmed.**

This document separates three very different things that are easy to conflate:

1. What the **Go2 family** is documented to be
2. What **our school's robot** specifically is — ⚠️ largely unconfirmed
3. What we have **verified on our own hardware** — 🔴 nothing

---

## 1. ⚠️ The unconfirmed core

| Attribute | Value | Confidence |
|---|---|---|
| Exact model | **UNKNOWN** | 🔴 Never confirmed |
| Variant (EDU / PRO / AIR) | **Presumed EDU** | 🔴 **Never confirmed** |
| Firmware version | **UNKNOWN** | 🔴 Never confirmed |
| SDK licence status | **UNKNOWN** | 🔴 Never confirmed |
| Serial number | **UNKNOWN** | — |
| Purchase date / warranty | **UNKNOWN** | — |
| School asset register reference | **UNKNOWN** | — |

🔴 **This is Gate 1. If the unit is not EDU-or-above, the secondary development
interface does not open and the entire project is impossible.**

Verbatim vendor constraint:

> *"Only EDU or above versions can be developed, other versions such as PRO and
> AIR versions do not open the secondary development interface."*

⚠️ **Nothing in this repository should be read as confirming the school has the
right unit.** That confirmation does not exist.

---

## 2. Go2 family — documented capabilities

**Class: `DOCUMENTED_NOT_HARDWARE_VERIFIED`.** These come from vendor
documentation. **We have tested none of them.**

### 2.1 Locomotion

| Attribute | Documented |
|---|---|
| Type | Quadruped |
| Locomotion | Walk, trot, run, climb, step |
| Gait configuration | ⚠️ **Stair gait setter removed at firmware ≥1.1.6** |
| Autonomous return-to-dock | ⚠️ Vendor-documented; **no dock configured by us** |

⚠️ **"Vendor supports stairs" is not the same as "our unit can use stairs."**
The SDK support was removed. Stairs are excluded.

### 2.2 🔴 Sensor suite — the critical gap

| Sensor | Present | Consequence |
|---|---|---|
| Depth / perception | ✅ | Obstacle detection, mapping |
| RGB camera | ✅ | ⚠️ Privacy-sensitive; no raw video retention |
| Joint position / state | ✅ | Proprioception, gait |
| IMU | ✅ | Orientation |
| **Altitude** | ❌ | 🔴 **No floor identity from the robot** |
| **Pressure sensor** | ❌ | 🔴 **No floor identity from the robot** |
| **Barometer** | ❌ | 🔴 **No floor identity from the robot** |
| LiDAR (built-in) | ❌ | Must rely on depth camera |

> ## 🔴 The most consequential finding in this repository
>
> **The robot cannot determine which floor it is on.**
>
> There is no altitude, pressure, or barometer sensor. This is not a software
> problem that better code solves — it is a hardware absence. Every multi-floor
> design must therefore add an **external** mechanism, such as ArUco floor markers
> (ADR-006, provisional and unvalidated).
>
> See [`../feasibility/constraints.md`](../feasibility/constraints.md) C-2.

### 2.3 Compute and connectivity

| Attribute | Documented |
|---|---|
| Onboard compute | ✅ Present |
| Development interface | ⚠️ **EDU-only** |
| Primary link | 🔴 **Wired Ethernet required** — see [`network_interfaces.md`](network_interfaces.md) |
| Wireless | ⚠️ Limited; not the primary link |
| External device mounting | ✅ Possible (ESP32 considered, deferred) |

---

## 3. Payload and platform limits

| Attribute | Documented | Relevance |
|---|---|---|
| Payload capacity | ✅ Ample for documents/items | Delivery task is well-matched |
| Dimensions | Compact quadruped | ⚠️ Must confirm corridor fit — never surveyed |
| Operating temperature | Vendor-specified | ⚠️ Indian summer relevance — untested |
| Ingress protection | ⚠️ Limited | ⚠️ Indoor only; no rain corridors |

⚠️ **Physical fit in the school's corridors is entirely unverified.** Door
widths, corridor widths, and turning radii were never measured.

---

## 4. Battery and charging

See [`battery_and_charging.md`](battery_and_charging.md) for detail.

| Attribute | Status |
|---|---|
| Battery telemetry | ⚠️ Vendor-documented; **unverified by us** |
| Reserve threshold (20%) | ⚠️ **Design intent — no code enforces it** |
| Autonomous return-to-dock | ⚠️ Capability documented; **no integration exists** |
| Swap vs charge | `UNKNOWN` — school facilities decision |
| Charging location | `UNKNOWN` — never discussed with the school |

🔴 **There is no configured dock and no charge-management implementation.**
Recovery is human-assisted by design.

---

## 5. 🔴 What we have verified on hardware

> ## Nothing.
>
> **Zero hardware interactions have occurred.** The robot has never been
> accessed, commanded, observed, or tested. Every capability above is
> vendor-documented.
>
> If a document in this repository implies otherwise, that document is wrong.

---

## 6. 🔴 Confirmations required

Priority order. **None of these has been requested.**

| # | Confirmation | Why | Ask whom |
|---|---|---|---|
| 6.1 | 🔴 **Model and variant** | Project viability | School / robot owner |
| 6.2 | 🔴 **Firmware version** | SDK features available | School / robot owner |
| 6.3 | 🔴 **SDK licence status** | Programming eligibility | School / robot owner |
| 6.4 | Physical dimensions and weight | Corridor fit | Robot owner |
| 6.5 | Battery type and expected runtime | Duty-cycle planning | Robot owner |
| 6.6 | Charging setup and location | Recovery workflow | School facilities |
| 6.7 | Asset register and insurance status | Permission basis | School administration |
| 6.8 | Operating temperature limits | Seasonal viability | Robot owner |

⚠️ **Items 6.1–6.3 are Gate 1.** Nothing else matters until they are answered.

---

## 7. Summary

| Statement | Verdict |
|---|---|
| "We use a Unitree Go2 EDU" | ⚠️ **Presumed, not confirmed** |
| "The robot can be programmed" | ⚠️ Only if EDU-or-above — **unconfirmed** |
| "The robot knows which floor it is on" | ⛔ **False — no such sensor** |
| "The robot can use stairs" | ⛔ **False — removed from SDK** |
| "We have tested the robot" | ⛔ **False — never accessed** |