# Safety and Failure Analysis

For a school environment. **Read this before anyone operates either robot.**

All vendor safety statements quoted verbatim. Evidence labels per `README.md`.

---

## 1. The finding that shapes everything: there is no vendor safety case

**Neither robot has any independent safety assessment.** No ISO 10218
(industrial), no ISO/TS 15066 (collaborative), no UL/CE robotics certification.
Unitree publishes only a liability disclaimer:

> "This product is a civilian robot. We kindly request that all users refrain from
> making any dangerous modifications or using the robot in a hazardous manner."
> — `unitree.com/go2`

> "The humanoid robot has a complex structure and extremely powerful power. Users
> are asked to keep a sufficient safe distance between the humanoid robot and
> people. Please use with caution."
> — `unitree.com/g1`

> "This product is not a toy and is not intended for use by persons under the
> age of 18."
> — G1 User Manual

**Implication:** the exclusion zone, the spotter, and the abort procedure are
**our responsibility, not Unitree's**. They cost the same for both robots and are
not a differentiator in the decision matrix — but they are mandatory.

---

## 2. G1 — vendor safety documentation, verbatim

These are the strongest statements Unitree makes, and they are unusually
directive.

### 2.1 The protective frame is mandatory

> ⚠️ "**Do not use the emergency braking function when the robot is not protected
> by the protective frame and protective rope provided by Unitree in the software
> manual. Otherwise, it will cause the robot to fall to the ground and cause
> serious damage!!!**

> ⚠️ "**When the robot is not protected by the protective frame and protective
> rope provided by Unitree** (or there is no artificial support for the robot),
> **please do not cut off the power** of the robot by pressing the battery power
> button, otherwise the robot will fall to the ground!!!**

> "In developer mode, the user must ensure that the robot protects the robot by
> the **protection frame and the protection rope**."

**Operational consequences:**
- Emergency braking is only safe **while framed**. A framed emergency stop
  becomes an unframed fall.
- The robot must never be powered off while standing unsupported. Power off only
  while suspended on the frame, or seated, in damping.
- Long-term standby: suspend on the frame in damping, or it falls as the battery
  drains.

### 2.2 Distance and supervision

> "When using the robot, keep it within your field of vision and a safe distance
> of **at least 2 meters** from obstacles, complex terrain, crowds, water and
> other objects."

> "When users use their own programs, there is a lot of uncertainty, the robot may
> do some unexpected actions (**Uncontrolled to the crowd**, rushing to the water,
> high places Fall, leg movements, tumbling)."

> ⚠️ Custom-code risk is named explicitly by the manufacturer. This is
> self-reported, and it should be taken at face value.

### 2.3 Emergency stop

`L1 + A` → damping mode; the robot slowly falls.
⚠️ **The official manual is internally inconsistent** — some sections state
`L2 + B`. **Verify on hardware before relying on it.** Do not discover this during
an exhibition.

### 2.4 Environment

> "**The robot is not waterproof**, so do not run it with water on the ground, in
> rain, snow, or wet conditions! **The robot is not dustproof**, please do not run
> it on gravel floors, dusty environments!"

- Operating temperature **0–40 °C**
- **No IP rating published** — only these negative statements
- **Stairs are not supported**: "Avoid attempting to climb stairs to prevent damage"
- **Max incline: NOT FOUND** for the legged G1

> ⚠️ **Documentation contamination warning:** the G1 User Manual contains an
> embedded **quadruped safety section** — 5–35 °C, 25° slopes, 10 cm steps, 3 m
> safe distance, "since the robot does not have a visual perception system at
> present." That is **Go2/B2 material in a G1 document**. Several of those
> figures circulate online as G1 specifications. **They are not.**

---

## 3. Go2 — safety documentation

Deliberately thin. Unitree publishes no distance requirement, no human-proximity
statement, and no leg-injury risk statement. What exists:

| Source | Content |
|---|---|
| `about_Go2` | "This product is a civilian robot… refrain from making any dangerous modifications or using the robot in a hazardous manner." |
| `about_Go2` footnote | "Limited to the current technique and computing power resources, **part of function shall be realized human operation or secondary development**." |
| `AI_motion_service` | `FreeAvoid`: "In this mode, the robot can avoid obstacles while moving; **when stationary, it can dodge objects in front.**" |
| `Motion_Services_Interface_V2.0` | `SwitchAvoidMode`: "In avoidance mode, disable obstacle avoidance in front when the joystick is not pushed, as well as obstacle avoidance behind… **It is generally not recommended to use this mode.**" |
| Motion doc | `TrotRun()`: "somewhat risky… prone to falling on uneven ground" |
| Motion doc | `WalkUpright()`, `CrossStep()`: "The motors in this gait are prone to overheating" |
| `Payload` | `AutoRecoverSet`: "When the device has a payload, it is recommended to **disable** this setting. If the device falls and flips violently, it may damage the payload." |
| Operating temperature | **5–35 °C**, "in good weather" |

### The uncommanded-dodge hazard

⚠️ **This deserves specific attention.** `FreeAvoid` documentation states the
robot "can dodge objects in front" **when stationary**. That is an uncommanded
lateral motion, triggered by a person walking into its space.

**If a student walks into a stationary Go2's space, it may move sideways into
another student.** This is the single most likely real injury mechanism for this
platform and it is documented by the manufacturer.

**Mitigation:** during demonstrations, **do not rely on `FreeAvoid` for
stationary safety.** Use the physical exclusion zone. Consider whether
`FreeAvoid` should be disabled entirely and the RC used instead — noting that the
built-in avoidance is on by default and **switching it off removes a safety layer**,
which needs its own justification.

### LED state readout (useful for the spotter)

| Colour | State |
|---|---|
| **Green** | Obstacle avoidance ON (default) |
| **Blue** | Obstacle avoidance OFF |
| **Purple** | Companion mode active |
| **Yellow** | Range mode |

Priority: purple > yellow > blue. **Teach the spotter to read these.**

---

## 4. Battery safety (both platforms)

From the G1 User Manual; the Go2 class is comparable:

| Rule | Value |
|---|---|
| **Discharge temperature** | −20 °C to 60 °C |
| **Charge temperature** | 0 °C to 55 °C |
| **Exceeding these** | "may cause the battery to ignite or even explode" |
| **Do not charge immediately after running** | Cool to room temperature first |
| **Ideal charge ambient** | 5–40 °C |
| **Ideal storage** | 22–28 °C; **never above 40 °C** |
| **Stop use below** | 10 % remaining |
| **Long-term storage** | Recharge to 70 % if below 30 % |
| **Prohibited** | Charging where electromagnetic/electrostatic fields are strong |
| **Reject the pack if** | Bulging, water ingress, or damaged |

**Operational rule for this project: charge overnight, on a non-flammable surface,
supervised, never unattended overnight, never below 10 % or above 90 % for
storage.**

⚠️ Go2 batteries: BT2-05 is 2.0 kg; BT2-06 is 2.5 kg — **17% of the robot's
total mass**. Handle as a two-person job if the pack is awkward.

---

## 5. Realistic failure scenarios

Ranked by likelihood × severity. Each has a concrete mitigation.

### F1 — The robot's controller dies mid-task 🔴 HIGH LIKELIHOOD, HIGH SEVERITY

**Scenario.** The external host crashes, the network cable is knocked, or the
DDS session drops while the robot is executing a route.

**Why it is realistic.** Independent Go2 operator reports include *"powered off
completely in the middle of its movement and collapsed to the floor."*

**Consequence.** A 15 kg quadruped collapses at ankle height near people; a 35 kg
humanoid falls from standing.

**Mitigation.**
- Software watchdog on the host: no heartbeat for 300 ms → `StopMove()` then
  `Damp()`
- ⚠️ **But if the host itself has crashed, the watchdog is dead too.** The
  backstops are the robot's built-in avoidance (on by default) and the physical RC.
- **RC authority must be tested immediately before every run**, not assumed
- Spotter holds the RC with priority over any other input
- Plan the route so a stop-in-place leaves the robot in a corridor, not in a
  doorway or a crowd

### F2 — Payload falls off 🔴 MEDIUM-HIGH SEVERITY

**Scenario.** A tray-mounted load shifts or tips during locomotion and lands on
the floor or a person's foot.

**Why it is realistic.** ⚠️ **No published G1 or Go2 load-retention data exists.**
Community Go2 carrier reports note that above ~3 kg or off-centre, gait visibly
changes. Corridor thresholds and door frames cause vertical shocks.

**Mitigation.**
- **Ratcheting cradle plus a secondary retention strap.** Two independent
  restraints.
- Payload ≤2 kg for the initial demonstration — well inside the ~3 kg usable
  envelope
- Centre of mass within the spine hardpoint quadrilateral; measure and mark
- **Never carry glass, liquid, sharp, hot, toxic, fragile, or unsealed items** —
  not in the initial demonstration, and arguably not ever
- Contents declared in advance and checked against an allow-list
- **Drop-test the tray rig before the robot touches it** — tilt it to 45° and
  check retention

### F3 — The Go2 collides with a person 🟠 MEDIUM SEVERITY

**Scenario.** A person steps into the robot's path; the avoidance layer either
does not fire, fires late, or the robot is moving in a mode with avoidance
disabled.

**Why it is realistic.** ⚠️ An independent operator reports the Go2 "run into a
door … while it was supposed to be in autonomous navigation and obstacle
avoidance mode." `SwitchAvoidMode()` explicitly makes the robot blind in front
when the joystick is idle and blind behind — Unitree says it is not recommended.

**Consequence.** At 15 kg and ankle height, a collision is a bruise or a trip,
not a serious injury — provided speed is capped.

**Mitigation.**
- **Speed capped at 0.4–0.6 m/s** — never faster than a walking person
- Exclusion zone of 1.5 m either side, physically marked with cones
- **Nobody in the corridor during a demonstration run**
- Do not use `SwitchAvoidMode()` — keep default avoidance, which is on
- Pre-mapped route; no exploration
- The 2D LiDAR's **minimum detection is 0.05 m**, but 0.05 m is floor-level —
  **a desk, a chair, or a child's bag above 20 cm may not be seen reliably by
  the obstacle-avoidance layer.** Use the camera as a supplementary check.

### F4 — The Go2 trips over a threshold or cable 🔴 HIGH LIKELIHOOD, LOW-MEDIUM SEVERITY

**Scenario.** A door threshold, a floor cable, a chair leg, or a rug edge.

**Why it is realistic.** Corridors are full of these. The Go2 handles 16 cm steps
well, which helps — but a rug edge or a cable is a dynamic event, not a step.

**Consequence.** The robot falls and self-rights (`RecoveryStand()`). Mostly a
delay, occasionally a dropped payload.

**Mitigation.**
- **Physically survey and clear the route before every run.** Remove cables,
  move rugs, move chairs.
- Keep the demonstration route short — a 10–15 m loop, not a building tour
- Speed reduced to 0.3 m/s on the first few runs
- `RecoveryStand()` tested manually before the demonstration
- Treat a payload drop as a *successful* safety test to be reported, not hidden

### F5 — The G1 falls mid-demonstration 🔴 LOW LIKELIHOOD, **VERY HIGH SEVERITY**

**Scenario.** The G1 loses balance, or overheats and shuts down, while
demonstrating.

**Why it is realistic.** This is the G1's defining failure mode and it is
independently documented:
- Munro's base G1 "proved unable to stand back up" after being pushed over
- An Auckland University lab's unit was "unusable" with an unresolved stability fault
- **NVIDIA tracker #121 (open since April 2026):** G1 + Dex3 overheats "even when
  it is just standing idle," auto-shutdowns, and *"the robot has fallen multiple
  times"*

**Consequence.** A 35 kg biped falling. A frame and rope reduce but do not
eliminate the risk. **The Unitree manual explicitly warns that the robot may move
"Uncontrolled to the crowd."**

**Mitigation — non-negotiable:**
- **Protective frame + protective rope, fitted and used for every operation**
- **Fenced pen. No public. No students. No unsupervised operation. Ever.**
- Dedicated spotter whose only job is the robot
- Speed capped at 0.2 m/s
- **Damp() rather than emergency brake when unframed** — emergency brake causes
  a fall
- Never power off while standing unsupported
- A written go/no-go before each session: frame present, rope present, spotter
  present, battery above 60%
- **A planned abort on any instability indicator** — stop and frame the robot

### F6 — Localization loss in a corridor 🟠 HIGH LIKELIHOOD, LOW SEVERITY

**Scenario.** The robot's pose estimate degrades in a long corridor — the classic
LiDAR SLAM failure case of parallel walls and low texture.

**Why it is realistic.** Long corridors are exactly where 2D LiDAR localisation
struggles. No published Go2 corridor repeatability figure exists.

**Mitigation.**
- Prefer a route with at least some visual or structural variety — doorways,
  corners, a distinctive feature
- Use the desk-height depth camera to supplement LiDAR (EDU only)
- **Define localisation loss as an explicit abort condition**, not something to
  keep driving through
- Re-localisation attempt, then give up and report honestly
- **Report the localisation failure rate as a result.** It is a genuine finding.

### F7 — Battery exhaustion mid-route 🟠 MEDIUM LIKELIHOOD, LOW SEVERITY

**Scenario.** The robot reaches a low state of charge away from the dock.

**Why it is realistic.** ~2 h realistic on the EDU long battery, less with a
payload and with the vision pipeline running. ⚠️ Unitree quotes long-battery
runtime three different ways (2–4 h, 3–5 h, up to 4 h).

**Mitigation.**
- **Battery check before every run; abort below 40 %**
- Reserve calculated from measured consumption, not from spec sheets — measure
  in Phase 8
- Charging station or a swapped spare pack
- The task manager must refuse a task it cannot complete on the remaining charge
- Note: `EconomicGait()` extends endurance but **raises ride height** — check it
  still clears the route's thresholds

### F8 — Human leg injury from the Go2 🔴 LOW SEVERITY, HIGH LIKELIHOOD

**Scenario.** A child's foot or hand is caught under a moving leg or in the wheel
region.

**Why it is realistic.** Twelve legs moving continuously at ankle height in a
space children want to walk into.

**Mitigation.**
- Exclusion zone, physically marked — this is the primary control
- Nobody within the zone during a run
- Speed capped
- **The spotter's job is specifically to keep people out, not just to watch the
  robot**
- Consider one or two demonstration runs with the robot stationary while
  explaining, so the crowd is not moving around a moving robot

### F9 — The AI layer causes an unsafe action 🔴 LOW LIKELIHOOD, HIGH SEVERITY

**Scenario.** A language model or a compromised input causes an action the user
did not intend.

**Why it is realistic.** Not because the model will spontaneously misbehave, but
because prompt injection is a real and demonstrated attack class.

**Mitigation — architectural, not procedural:**
- The LLM process has **no network route to the robot**; verifiable by inspection
- The LLM has **no tool access** — it emits JSON to a pipe
- The validator is a **pure, deterministic function with no network access**
- **Unknown entities are rejected, never guessed**
- **Human confirmation on every LLM-derived task**
- An adversarial test suite with a measured pass rate
- **The exhibition demonstration runs with the LLM layer switchable off**, and
  must still work

### F10 — Software or firmware mismatch 🔴 MEDIUM LIKELIHOOD, MEDIUM SEVERITY

**Scenario.** An OTA update, or a driver/library version mismatch, changes
behaviour mid-project.

**Why it is realistic.** ⚠️ Go2 firmware ≥ 1.1.6 is required for the V2.0 motion
interface, and the service name changes from `sport_mode` to `mcf` at that
boundary. G1 `rt/sportmodestate` needs ≥ 1.5.1. G1 FSM 801 became 802 after
8.6.x.x. Both robots support automatic cloud OTA.

**Mitigation.**
- **Disable automatic OTA before development begins**
- **Pin and record firmware versions** in the repository
- Test upgrades in simulation (`unitree_mujoco`) before applying to hardware
- Keep a known-good firmware image

---

## 6. Risk register

| ID | Risk | L | S | Rating | Primary control |
|---|---|---|---|---|---|
| F1 | Controller/network loss mid-task | H | H | 🔴 High | RC + watchdog + built-in avoidance |
| F2 | Payload falls | M-H | H | 🔴 High | Cradle + strap + ≤2 kg + pre-test |
| F3 | Collision with a person | M | M | 🟠 Medium | Speed cap + exclusion zone |
| F4 | Trips on threshold/cable | H | L-M | 🟠 Medium | Route survey + clearance |
| F5 | **G1 falls** | L | **V-H** | 🔴 High (G1 only) | **Frame + rope + fenced pen** |
| F6 | Localisation loss | H | L | 🟠 Medium | Abort condition + re-localise + report |
| F7 | Battery exhaustion | M | L | 🟠 Medium | 40% floor + measured reserve |
| F8 | Leg injury | H | L | 🟠 Medium | Exclusion zone + speed cap |
| F9 | AI causes unsafe action | L | H | 🟠 Medium | Architectural boundary |
| F10 | Firmware mismatch | M | M | 🟠 Medium | Disable OTA + pin versions |

**No risk is acceptable at "High" without a named, tested control.** F1, F2, and
F5 are the three that must be retired before any public demonstration.

---

## 7. Operating rules

### 7.1 Absolute rules — no exceptions

1. **Never operate either robot unsupervised.**
2. **The G1 requires its protective frame and rope for every operation.** No
   frame, no operation.
3. **Never power off the G1 while it is standing unsupported.**
4. **Never use emergency braking on an unframed G1** — it causes a fall.
5. **Speed caps are absolute:** Go2 ≤0.6 m/s (0.3 m/s on early runs); G1 ≤0.2 m/s.
6. **An exclusion zone is physically marked and enforced at all times.**
7. **RC authority is tested immediately before every run.**
8. **Only approved, sealed, non-hazardous payloads.** No glass, liquid, chemicals,
   reagents, sharps, heated apparatus, or fragile items.
9. **Battery above 40% before any run; above 60% before a public demonstration.**
10. **No food, no drink, no personal items on the robot.**
11. **Automatic OTA is disabled.** Firmware is pinned.
12. **The AI layer can be disabled at any time, and the system remains
    demonstrable.**

### 7.2 Pre-run checklist — run it every time, no exceptions

```
□  Exclusion zone marked and clear
□  Route physically surveyed — cables, rugs, chairs removed
□  Spotter present, briefed, RC in hand
□  RC authority tested (robot responds, then stops)
□  Battery above threshold
□  Payload secure — cradle AND strap — contents allow-listed
□  Firmware version matches the pinned value
□  Dashboard reachable; task log recording
□  E-stop procedure known to everyone present
□  (G1 only) Frame fitted, rope fitted, damping confirmed
□  Abort criteria agreed and known to the spotter
□  Camera running / recorded
```

### 7.3 Abort criteria — any of these means STOP

- Robot deviates from the mapped corridor
- Any person enters the exclusion zone
- Robot behaviour differs from expected
- Localisation is lost and not recovered within the timeout
- Any abnormal noise (this is the primary cue for a Go2 joint or motor fault)
- Battery below the abort threshold
- Payload shifts
- Communications lost
- **Any judgement that something is not right — no justification required**

> **The spotter has unilateral authority to stop the robot and does not need to
> justify it.** Nothing in the project requires the robot to keep running.

### 7.4 Speed guidance

| Robot | Demonstration speed | Maximum |
|---|---|---|
| Go2 | 0.3–0.4 m/s | **0.6 m/s** |
| Go2 with 2 kg payload | 0.3 m/s | 0.5 m/s |
| G1 | 0.1–0.2 m/s | **0.2 m/s** (and only with frame + rope) |

These are demonstration caps. `SportClient.Move()` accepts up to 3.7 m/s on X/EDU
and `ObstaclesAvoidClient.Move()` documents x up to ±1.5 m/s — **neither is an
appropriate demonstration speed, and neither is a safety limit.**

---

## 8. What must be tested before anyone is near the robot

A commissioning programme. Each item is a hold point.

| # | Test | Pass criterion | Risk retired |
|---|---|---|---|
| T1 | Low-level telemetry subscription only, robot on the stand | Battery, FSM, motor temps read correctly | Network config |
| T2 | Stand / damp / balance transitions, framed (G1) or tethered (Go2) | No unexpected motion | Actuator interface |
| T3 | Velocity command at 0.1 m/s in a clear area | Robot responds as commanded | F1 partial |
| T4 | **Emergency stop, both robots** | Robot stops within the stated time | F1 |
| T5 | RC authority takeover | Robot stops on RC input every time | F1 |
| T6 | Navigation on a 10 m mapped loop, empty | Completes the loop N times | F4, F6 |
| T7 | Obstacle injection at a known point | Stops or reroutes; **latency measured** | F3 |
| T8 | Localisation loss injected | Detects, stops, recovers or aborts | F6 |
| T9 | Battery consumption over a full route | Reserve calculated from measurement | F7 |
| T10 | **Carrier rig drop test, tilted to 45°** | Nothing retained is lost | F2 |
| T11 | Payload route test, 2 kg, 10 loops | No drop; displacement measured | F2 |
| T12 | Full task loop end-to-end, 20 runs | Completion rate measured | All |
| T13 | Uncommanded-motion check | Robot never moves without a command | F1 |
| T14 | G1: frame + rope + damp, no power-off standing | Confirmed by spotter | F5 |
| T15 | Endurance: 2 h continuous operation | No thermal or mechanical fault | F7 |

**T4, T5, T10, and T13 are hold points. Nothing proceeds to a public audience
until all four pass.**

---

## 9. Incident reporting

Any of the following is logged and reviewed, whether or not it caused harm:

- Robot contact with a person, object, or wall
- Any fall (either platform)
- Payload drop or shift
- Uncommanded motion
- Battery event (swelling, unexpected shutdown)
- Communications loss during a task
- Any near-miss

Record: time, location, task ID, robot state log, video if available, what
happened, what the immediate cause appears to be, and what control changes.

**Near-misses are the most valuable data the team will collect.** Reporting them
is a sign of a competent project, not a failed one.

---

## 10. A note on how this affects the recommendation

Safety does **not** change the robot choice — both robots need an exclusion zone, a
spotter, speed caps, and a written abort procedure, and that cost is identical.

What changes is the **magnitude**. The G1 adds a mandatory frame, a mandatory
rope, a fenced pen, no public access, and a fall that cannot be recovered from
without human intervention. The Go2 adds a trip hazard and a documented
uncommanded-dodge behaviour.

**A G1 demonstration is a laboratory demonstration. A Go2 demonstration can be a
school demonstration.** That difference is reflected in the 10% safety weight in
`weighted_decision_matrix.md`, and it is worth considerably more than 10% when
judging what kind of event the school can actually hold.
