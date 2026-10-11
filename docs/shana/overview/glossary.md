# SHANA — Glossary

Terms used across the SHANA documentation. Where a term has a common
misunderstanding, that is stated.

---

## Robot and hardware

**Go2 EDU** — Unitree's quadruped robot, EDU variant. 12 degrees of freedom,
approximately 15 kg, approximately 8 kg rated payload. Only EDU-and-above
variants may be programmed with custom software. *Note: the "EDU" designation
governs development access, not physical capability.*

**Quadruped** — A four-legged robot. Unlike a humanoid it does not balance
through an upright posture; it is stable by design. This makes it a good
carrier and a poor manipulator.

**Sport mode / motion service** — Unitree's closed on-board motion controller.
SDKMotion requests are sent to it; the gait itself is proprietary and cannot be
modified. *Sometimes called `sport_mode` (firmware < 1.1.6) or `mcf` (≥ 1.1.6).*

**Cyclone DDS** — The middleware the robot uses to communicate. All commands and
telemetry travel over it. ⚠️ **Requires a wired Ethernet connection**; discovery
is unreliable over Wi-Fi.

**IMU** — Inertial Measurement Unit. Reports orientation and acceleration.
**Does not report altitude on this robot.** See "floor identification".

**LiDAR (L2, "4D")** — The robot's built-in rotating sensor. 360° × 96° field of
view, 21,600 samples per second, 0.05 m minimum range. Despite being marketed as
2D, its wide vertical field makes it a genuinely three-dimensional sensor.

**BMS** — Battery Management System. Reports state of charge, current,
temperature, and per-cell voltages. *Battery telemetry is on this, not on the
motion topic.*

**ArUco / AprilTag** — A square printed marker that a camera can identify by its
numeric ID. Useful for identifying which floor the robot is on, or which
checkpoint it has reached. *Does not make a route safe — it identifies.*

**Checkpoint (CP)** — An approved location where the robot may be stationed,
recovered, and charged. Provisional names such as `CP-C3` are **names only, not
verified navigation coordinates.**

**Payload carrier** — The tray or cradle that holds the delivered item. Must have
two independent restraints and be tilt-tested before use.

**Docking / charging pile** — An ArUco-marked platform the robot can drive onto.
⚠️ **There is no autonomous return-to-dock capability**, despite the mechanism
existing.

---

## Navigation and mapping

**Slam / SLAM** — Simultaneous Localisation and Mapping. Builds a map of an
unknown environment while tracking position within it.

**Localisation** — Determining the robot's pose *within a known map*. A
within-floor problem.

**Floor identification** — Determining *which map applies*. A discrete
classification between floors. ⚠️ **A harder and separate problem. The Go2 cannot
do it with its own sensors.**

**Relocalisation** — Recovering a pose estimate after it has been lost.
*Frequently confused with floor identification. They are different problems.*

**Map** — A surveyed, version-controlled representation of one floor's
traversable area.

**Route graph** — An explicit network of approved corridor segments and
intersections. Contains **only** segments a person has physically verified.

**Named destination** — A database record such as `C3_AI_LAB`. ⚠️ **A name is
not a location, and a location is not a verified goal.** See `docs/shana/architecture/route_graph.md`.

**Waypoint** — A specific pose within a map. *A waypoint may be valid while its
containing segment is closed.*

**Odometry** — Position estimated by integrating motion over time. Accumulates
error and resets on power loss. ⚠️ **Not an absolute position.**

**Verified navigation goal** — A pose on a validated map, physically reached and
confirmed by a person. **The only kind of goal the robot may be sent to.**

**Dead reckoning** — Estimating position from known motion history. Used by
odometry; unreliable over long periods.

---

## Communications

**Ethernet (wired)** — The only manufacturer-supported connection for
development. ⚠️ **Required for the robot link.**

**WLAN / Wi-Fi** — School wireless network. Used by teachers and, potentially,
backup telemetry. ⚠️ **Never for the robot link.**

**VLAN** — A logically separated network segment. Required to isolate the robot
subnet from the school LAN.

**Multicast** — Network traffic sent to a group. ⚠️ **Required by the robot's
discovery protocol and frequently blocked by default on school networks.**

**Subnet** — Here, `192.168.123.0/24` for the robot, per manufacturer
specification.

**LoRa** — A low-power, long-range radio technology. Sub-GHz in India
(865–868 MHz). ⚠️ **Not suitable for video, navigation data, or control loops.**
Low bandwidth; high airtime.

**Duty cycle** — The fraction of time a radio may transmit, regulated. In India,
limits apply to sub-GHz devices.

**e.r.p.** — Effective Radiated Power, measured **relative to a half-wave
dipole**. ⚠️ **Not the same as EIRP.** Vendors frequently conflate them.

**Heartbeat** — A periodic signal proving the server is alive and in control.
Its absence triggers a safe stop.

**Watchdog** — A mechanism that acts when a heartbeat is missing.

**Orphaned** — The robot's state when the server has lost contact. It holds
position and accepts no new tasks.

---

## Delivery and workflow

**Delivery request** — A teacher-submitted request to move an approved item
between approved locations.

**Allow-list** — The set of destinations the robot may be sent to. ⚠️ **Free
text destinations are never accepted**, because the robot cannot verify them.

**Task validator** — A deterministic, auditable function that checks a request
against every rule. ⚠️ **The only thing that may authorise a task.** No model
output ever bypasses it.

**Delivery state machine** — The defined sequence of states a delivery passes
through. See `docs/shana/architecture/task_state_machine.md`.

**Completion** — Delivery marked finished. ⚠️ **Requires explicit recipient
confirmation. Never inferred from arrival.**

**Recipient confirmation** — A named person asserting they have received the
item.

**Task queue** — Ordered list of accepted requests awaiting execution.

**Relocation event** — A recorded instance of a person manually handling the
robot. ⚠️ **Invalidates the pose estimate and forces re-localisation.** Without
this, a carried robot may confidently navigate the wrong map.

---

## Safety and operations

**Safety supervisor** — An independent software component that can veto any
command. ⚠️ **Not the same as the robot's own obstacle avoidance**, and not the
same as the human supervisor.

**Human supervisor (spotter)** — A named person whose only job is the robot, with
**unilateral stop authority**. ⚠️ **Load-bearing for safety. Not optional.**

**Exclusion zone** — A physically marked area the robot stays inside and people
are kept out of. The primary physical safety control.

**Safe stop** — The defined behaviour on any unrecoverable condition: stop
moving, hold, notify a human.

**Abort** — Deliberately stopping an activity before it completes. ⚠️ **There
is no penalty for aborting.**

**Near-miss** — An event that could have caused harm but did not. ⚠️ **The most
valuable data a project collects.**

**Decentralised decision** — A decision made by the nearest authorised person
rather than escalating. Used for hazard declaration and abort.

**Corridor state** — A segment's condition: `AVAILABLE`, `BUSY`, `HAZARD`,
`CLOSED`, or `UNKNOWN`.

**`UNKNOWN`** — The state used when a sensor fails, data is stale, or confidence
is low. ⚠️ **Never treated as clear.** This is a safety-critical design decision.

---

## Governance

**Go/no-go gate** — A checkpoint that must be passed before work proceeds. A
gate is marked `PASSED`, `BLOCKED`, or `NOT APPLICABLE` — **never passed without
evidence.**

**Data Fiduciary** — The entity legally responsible for personal data under
applicable law. Holds approval authority for any data processing.

**CCTV analytics access** — Permission to derive anonymous occupancy states from
security camera feeds. ⚠️ **Not requested, not granted, and subject to a privacy
determination.**

**Assumption** — A statement treated as true so work can proceed, which is not
established. Tracked in `docs/shana/feasibility/assumptions.md`.

**Reversal condition** — The recorded circumstance under which a decision would
be revisited. ⚠️ **A decision without one becomes a decision nobody dares
revisit.**

---

## Acronyms

| Term | Meaning |
|---|---|
| BMS | Battery Management System |
| COP | Car Operating Panel (lift) |
| DHCP | Dynamic Host Configuration Protocol |
| DoT | Department of Telecommunications (India) |
| ETA | Equipment Type Approval (India) |
| IMU | Inertial Measurement Unit |
| LOP | Landing Operating Panel (lift) |
| Nav2 | ROS 2 Navigation Stack |
| ONVIF | Open Network Video Interface Forum |
| RTT | Round-Trip Time |
| SDK | Software Development Kit |
| SF | Spreading Factor (LoRa) |
| SLAM | Simultaneous Localisation and Mapping |
| WPC | Wireless Planning and Coordination (India) |
| QoS | Quality of Service |
| UDP | User Datagram Protocol |