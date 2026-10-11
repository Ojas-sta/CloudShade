# SHANA — Battery, Charging and Recovery

**Status: RESEARCH ONLY. No battery has been measured. Thresholds are TBD.**

---

## Where battery telemetry actually lives

⚠️ **A common and consequential error: `SportModeState` contains no battery
fields at all.**

Its complete field list is: `stamp`, `error_code`, `imu_state`, `mode`,
`progress`, `gait_type`, `foot_raise_height`, `position`, `body_height`,
`velocity`, `yaw_speed`, `range_obstacle`, `foot_force`, `foot_position_body`,
`foot_speed_body`.

**Battery is on `rt/lowstate` → `LowState.bms_state`.**

### `BmsState` fields

| Field | Type | Meaning |
|---|---|---|
| `soc` | uint8 | **State of charge, percent** — the mission-critical field |
| `current` | int32 | **Signed** — enables coulomb counting |
| `cell_vol` | uint16[15] | Per-cell millivolts — enables imbalance and health detection |
| `bq_ntc` | int8[2] | Battery temperatures |
| `status` | uint8 | Pack fault code |
| `cycle` | uint16 | Charge cycle count |

Plus, on `LowState`: `power_v`, `power_a`, `temperature_ntc1/2`.

⚠️ **Not available:** watt-hours remaining, estimated runtime, distance remaining.
All three must be derived.

---

## The runtime contradiction

⚠️ **Manufacturer figures span 1–5 hours across four official documents.**

| Source | Figure |
|---|---|
| `about_Go2` battery table, BT2-06 | 3–5 h |
| `about_Go2` product matrix, EDU | "About 2–4h" |
| `EconomicGait()` documentation | "up to 4 hours on a full charge" |
| **Go2 User Manual** | **"about 1-2 hours with no load static standing and upright walking alternately"** |

⚠️ **Design assumption: plan on ~90 minutes of active mission per charge**, with a
mandatory charge or swap between missions.

⚠️ **This is an engineering estimate, not a specification.** It is superseded by
measurement (`SH-BAT-005`).

---

## Battery pack variants

| | BT2-05 (standard) | **BT2-06 (EDU standard)** |
|---|---|---|
| Capacity | 8000 mAh | **15,000 mAh** |
| Energy | 236.8 Wh | **432 Wh** |
| Weight | 2.0 kg | **2.5 kg** |
| Rated voltage | 29.6 V | 28.8 V |
| Charge current | 3.5 A | **9 A** |
| Charge time | 2 h | **1 h 15** |

⚠️ **Which pack is fitted is unknown and consequential.** The 8,000 mAh pack
halves the endurance budget. ⚠️ **Read the label. It takes a photograph.**

⚠️ EDU ships **one** battery in the box. A spare is not included.

⚠️ The 2.5 kg long pack is **17% of the robot's total mass.** Mass distribution
affects payload capacity.

---

## Low-battery behaviour — architecturally important

✅ VERIFIED, Unitree pilot-lamp table:

> *"Low flash rate in yellow — **Low battery warning, will automatically crouch
> down within 10 minutes**"*

| Consequence | Design implication |
|---|---|
| It **crouches**; it does not collapse instantly | There is a reaction window |
| No controlled safe-landing is documented | **Never rely on the robot's own logic** |
| ⚠️ **A crouch on a staircase is a fall** | On stairs, 10 minutes is the *entire* budget |
| 20% is **not** a safe threshold | Use a higher abort threshold |

⚠️ **Abort thresholds are deliberately recorded as TBD.** They must be established
by measurement, not guessed. See `SH-BAT-006`, `SH-BAT-008`.

---

## Recovery behaviours and payload safety

| Method | Documented behaviour | With payload |
|---|---|---|
| `Damp()` | *"All motor joints stop moving and enter damping state. **This mode has the highest priority and is used for emergency stops**"* | ✅ Use for e-stop. Accepts payload drop |
| `RecoveryStand()` | *"**Regardless of whether it is fallen or not, it will recover to standing**"* | ⛔ **UNSAFE.** Rising from a heap with 8 kg is violent |
| `StandDown()` / `Sit()` / `BalanceStand()` / `StopMove()` | Controlled | ✅ Safe |
| `AutoRecoverSet(bool)` | Enable/disable auto-recovery | ⚠️ Unknown with payload. **Recommendation: OFF during payload missions** |

---

## Charging

### Procedure and limits

| Rule | Value |
|---|---|
| Discharge temperature | −20 °C to 60 °C |
| **Charge temperature** | **0 °C to 55 °C** |
| Exceeding these | ⚠️ *"may cause the battery to ignite or even explode"* |
| After running | Cool to room temperature before charging |
| Ideal charge ambient | 5–40 °C |
| Ideal storage | 22–28 °C; **never above 40 °C** |
| Stop use below | 10% remaining |
| Long-term storage | Recharge to 70% if below 30% |
| Prohibited | Charging in strong electromagnetic or electrostatic fields |

### Swap

✅ VERIFIED, Go2 User Manual:

> *"Lay Go2 on a flat surface, insert the battery pack into the battery pack's
> from the side of the robot, pay attention to the direction of installation,
> with the power switch button facing upwards. If the battery pack cannot be
> fully inserted, please adjust the direction of the battery packs and **do not
> press forcibly**… When you hear a 'click' sound, the battery pack installation
> is complete."*

**VERIFIED:** the pack inserts from the side and is mechanically latched. The
robot must be **lying on a flat surface**.

**NOT FOUND:** any hot-swap-while-powered procedure. **Treat swap as a
power-off, ~2-minute, lay-it-down procedure performed by a trained person.**

⚠️ **If the charging station is on one floor and missions need several, swap time
becomes the dominant floor-transition cost.**

---

## Return to base — a verified negative

| Item | Finding |
|---|---|
| Charging pile support | ✅ X and EDU only |
| **Autonomous return-to-dock** | ⛔ **NOT FOUND.** No charging or dock service in the documented service list (`sport_mode`/`mcf`, `basic_service`, `chat_go`, `net_switcher`, `obstacles_avoidance`, `ota_box`, `unitree_lidar`, `utrack`, `video_hub`, `webrtc_bridge`, `vui_service`) |
| **An "Aruco Recharge Service Interface" documentation page EXISTS** | 🟡 Page body would not render through 8 URL variants. **Not read** |
| Third-party implementation | 🧪 Topic `rt/aruco_cmd`, `StartRecharge(repeat, period_sec)` / `StopRecharge(...)`. **Not official** |
| Mechanism | 🧪 The pile presents an ArUco marker; the robot's camera recognises it and drives onto the pile |

⚠️ **The mechanism is marker-based, driven by a command from outside.** Something
else must decide when to go and route it there. That is our work.

> **This is a useful convergence:** the same ArUco marker pipeline that enables
> automatic docking is the same technology needed for **floor identification**.
> One investment, two capabilities.

**Conclusion: recovery is human-assisted by design.** Architect it, do not assume
otherwise.

---

## The reserve rule

**PROPOSED — thresholds are TBD until measured.**

```
Before starting a task:
  1. Read SOC and, if available, coulomb-counted consumption
  2. Check the task is completable on the remaining charge
  3. Check the RETURN route is feasible on the remaining charge after the task
  4. Check the nearest safe checkpoint is reachable and safe
  5. If ANY check fails → refuse the task with a specific reason

At the abort threshold:
  6. Stop accepting new tasks
  7. Return to a verified, reachable, safe checkpoint — or hold
  8. If return is not safely feasible → safe stop + human assistance request
```

⚠️ **A low battery must never cause the robot to take an unverified stair route,
enter a lift without approval, or cross a closed or hazardous corridor.** These
are hard constraints, not preferences (`SH-BAT-012`).

---

## Recovery workflow when the robot cannot continue

```
1. Robot holds position, does not wander
2. Dashboard shows: last known position, battery, state, fault, time since last contact
   ⚠️ position age shown prominently — a 40-minute-old position is not current
3. Supervisor notified with the location and the reason
4. Supervisor retrieves the robot by the agreed manual method
5. If the robot is unreachable, facilities search begins from the last
   confirmed checkpoint, labelled as possibly stale
6. Incident or fault logged
```

⚠️ **A checkpoint's last-known position is not necessarily the robot's current
position.** Any manual handling is recorded as a relocation event.

---

## Questions

| ID | Question | Priority |
|---|---|---|
| `SH-GO2-006` | Which battery is fitted? | P1 — **blocks endurance planning** |
| `SH-GO2-003` / `-007` | Dock or charging pile present? Spares? | P2 |
| `SH-BAT-001` to `-004` | Telemetry source, accuracy, calibration, failure behaviour | P1 |
| `SH-BAT-005` to `-009` | Consumption modelling, reserve, thresholds, recovery | P0 |
| `SH-BAT-010` to `-014` | Checkpoint selection, mid-task abort, inaccessible checkpoint | P1 |
| `SH-BAT-015` to `-019` | Charging permission, procedure, location, swap, spares | P0 |
| `SH-BAT-020` to `-022` | Health logging, quarantine, docking interface | P2 |

---

## Nothing here has been measured

⚠️ **Every number in this document is either a manufacturer figure — several of
which contradict each other — or an engineering estimate.** No battery has been
measured, no threshold has been validated, and no charging has been performed.