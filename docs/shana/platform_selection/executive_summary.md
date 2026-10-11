# Executive Summary

**Phase 0 — Platform Selection. Research date: 2026-10-10.**

---

## 1. The decision, stated directly

**Select the Unitree Go2 EDU. Do not select the G1.**

Weighted score: **Go2 EDU 63.5 / 100 — G1 EDU 40.5 / 100.**

This is not a close call, and the margin is not mainly about the G1 being a worse
robot. The G1 is the more capable machine in the abstract. The margin comes from
three specific, verifiable asymmetries.

---

## 2. The three findings that decide it

### Finding 1 — Only the EDU models can be programmed at all

This gates both robots and is verified from Unitree's own primary sources.

**G1.** Parsing the official specification table at `https://www.unitree.com/g1/`
returns three separate `<div>` cells per row with no merged cells anywhere in the
34-row table. The relevant rows, read directly:

```
Secondary Development【3】      || /    || YES
High Computing Power Module    || /    || Multiple brands and models availablesuch as Orin, etc.
Single Hand Degrees of Freedom || /    || 7（Optional Force control of three-fingered hand）...
Price(Tax and Shipping excluded)|| US $13.5K || Contact sales
```

The `/` in the standard-G1 column is Unitree's own notation for *not supplied*.
The $13,500 standard G1 has **no hands, no compute module, and no secondary
development support.**

**Go2.** Unitree's developer FAQ states verbatim:

> "**Only EDU or above versions can be developed, other versions such as PRO and
> AIR versions do not open the secondary development interface**, EDU version can
> be equipped with 40tops or 100tops docking station for development."
> — `https://support.unitree.com/home/en/developer/faq`

The Go2 specification table marks secondary development `○` for AIR, `○` for PRO,
`◕` ("basic support") for X, and `●` for EDU. `○` means *No*, per the official
legend. Go2 X's "basic support" is **not defined anywhere in Unitree's
documentation** and the RJ45 / DC-out / SBUS interfaces it would need are
documented as EDU-exclusive.

**Consequence:** if the school owns a standard G1, a Go2 AIR, a Go2 PRO, or a Go2
X, this project cannot proceed as specified. That is a procurement conversation,
and it is the first thing to confirm.

### Finding 2 — Only the Go2 has a vendor-supported autonomous navigation API

Unitree publishes a documented navigation service for the Go2 EDU. It publishes
nothing equivalent for the G1.

Go2 EDU, service `slam_operate`, version `1.0.0.1`, with published API IDs:

| API ID | Function |
|---|---|
| 1801 | start mapping |
| 1802 | end mapping |
| 1804 | init pose |
| 1102 | pose navigation |
| 1201 | pause |
| 1202 | resume |
| 1901 | close slam |

with published limits: navigation speed 0.2–1.0 m/s, obstacle-mode `1` = stop /
`0` = bypass, navigable width ≥ 0.8 m, obstacles ≤ 0.5 m wide and ≥ 20 cm tall,
target within 10 m. Point clouds on `rt/unitree/slam_mapping/points`, odometry on
`rt/unitree/slam_mapping/odom`, status on `rt/slam_info`.

Separately, the Go2's `ObstaclesAvoidClient` exposes a documented programmable
velocity interface — `Move(x, y, yaw)` with x ∈ [−1.5, 1.5] m/s, y ∈ [−1.0, 1.0]
m/s, yaw ∈ [−1.57, 1.57] rad/s, plus `MoveToIncrementPosition(±2.0 m)` and
`MoveToAbsolutePosition(world frame)`.

For the G1, Unitree publishes a locomotion interface (`LocoClient`, ~17 high-level
methods, capped at 3.0 m/s by `SetSpeedMode(3)`) but **no SLAM service, no
navigation service, no localisation service, and no obstacle-avoidance service.**

**Consequence:** autonomous delivery — which is the centre of the stated project
vision — is roughly 60% built on the Go2 and roughly 0% built on the G1.

### Finding 3 — The G1's manipulation is not reliable, and this is measured, not assumed

The G1 is the only one of the two robots with arms, so it wins the manipulation
category on principle. It does not win it on evidence.

| Measurement | Value | Source |
|---|---|---|
| Max sustained end-effector force, free-standing | **13.5 N (≈1.38 kgf)** | arXiv 2609.25486, measured on real G1 |
| Max sustained force with environmental bracing | 60 N | same |
| Best published **full-task** pick-and-place, held-out object | **20–30%** | arXiv 2610.08119 |
| Native foundation-model action, pick-and-place | **~7%** | same |
| Best grasp *completion*, 5 held-out objects, fixed VLA + supervisor | 61–84% | arXiv 2610.04391 |

Three caveats that matter more than the headline numbers:

1. **Both strong results used BrainCo 5-finger hands, not the Unitree Dex3-1**
   three-fingered hand that the school would actually own. **No published grasp
   success rate exists for the Dex3-1.** (NOT FOUND)
2. **Unitree's ~180 public G1 manipulation datasets contain no flat-sheet,
   notebook, book, or magazine grasping task.** The pick family is
   `PickApple, PickBottle, PickCharger, PickGum, PickSnack, PickTissue,
   PickDoll, PickDrink, PickToothpaste` and `GraspSquare, GraspOreo,
   GraspRubiksCube`. Unitree's own teleoperators did not publish a thin-sheet
   grasping task. This is a negative signal about the hardware's actual
   strengths, not an absence of effort.
3. **Reliability is a live, unresolved problem.** NVIDIA's maintainer for their
   own G1 deployment stack reports, in a ticket open since April 2026:
   > "the G1 alert an overheating warning after a few minutes, **even when it is
   > just standing idle**. After that, Sonic shuts down automatically. As a
   > result, **the robot has fallen multiple times**."
   > — `https://github.com/NVlabs/GR00T-WholeBodyControl/issues/121`

An independent teardown lab reported their base G1 "needed several software
updates before it would do anything" and, after being pushed over, "proved unable
to stand back up." A university lab declared their unit "unusable" pending
hip-motor replacement requiring international support escalation.

**Consequence:** "the humanoid picks up a notebook" is a *demo*, not a
*capability*. A judge watching five attempts will see it fail three times.

---

## 3. Why the Go2 wins the delivery category

| Factor | Go2 EDU | G1 EDU |
|---|---|---|
| Rated payload | ≈8 kg (max ~12 kg) | ≈3 kg **per arm**, and that is an arm-extension-dependent figure |
| Real-world usable payload | ~3 kg centred on the spine (community field data) | 1.38 kgf sustained, unbraced |
| Load stability | Low, static CoM on a stable base | Load held at the end of a lever arm while walking on 12 DOF |
| Autonomous navigation API | `slam_operate`, documented | None published |
| Programmable obstacle avoidance | `ObstaclesAvoidClient`, documented | None published |
| Onboard LiDAR point cloud on DDS | `rt/utlidar/cloud`, `cloud_deskewed`, `height_map_array`, `range_info` | Depth camera + 3D LiDAR listed; no client published |
| Battery | 15,000 mAh on EDU, ~2–4 h | 9,000 mAh, ~2 h spec / ~90 min measured walking |
| Manufacturer safety requirement | None beyond civilian disclaimer | **Protective frame + protective rope is mandatory** |
| Certification | None | None (no ISO 10218 / ISO/TS 15066) |
| Simulation | `unitree_mujoco`, BSD-3, CPU-only | `unitree_mujoco` low-level only, needs a locomotion policy |

The Go2 is also the platform Unitree itself productises for industrial work —
their quadruped inspection business is a shipped product line. The G1's own
strongest real-world deployment is the one that does not use its arms: ETH Zurich
runs G1 units to move samples between wet-lab rooms, as a wheeled-cart job.

---

## 4. Where the recommendation flips — be precise about this

The G1 is not a bad platform. It wins decisively on exactly one axis: **it is the
only robot that can pick things up.** If the exhibition is reframed from *"build a
useful autonomous assistant"* to *"demonstrate humanoid manipulation research"*,
the score changes.

We ran the sensitivity analysis explicitly (`weighted_decision_matrix.md` §5).
Under the brief's stated weights, Go2 wins **63.5 vs 40.5**. Under a deliberately
spectacle-weighted framing (manipulation 40%, exhibition impact 25%, everything
else squeezed), G1 wins **59.3 vs 47.7**.

**The G1 overtakes only when manipulation is weighted ≥ ~35% AND exhibition impact
≥ ~20% AND reliability is discounted to below ~2.5/10.** That combination
describes a science-fair spectacle. It does not describe the project vision in the
brief, which is a robot that "performs repetitive school tasks, transports
materials, assists teachers, and supports laboratory activities."

If the school insists on a humanoid for presentation reasons, that is a legitimate
brief — but it must be made explicitly, and the manipulation success rate must be
disclosed to judges up front.

---

## 5. The recommended project

**Primary concept: an autonomous school courier and mobile laboratory sensor
platform on the Go2 EDU.** Full specification in
`exhibition_project_ideas.md` §Concept 2.

The core loop, using only verified interfaces:

1. A teacher submits a delivery request on a web dashboard.
2. A deterministic task manager validates it against an allow-list of approved
   destinations and permitted payloads. **No LLM touches the robot.**
3. The task manager reads live robot state (`rt/sportmodestate`, battery, fault
   flags) and refuses the task if the robot is not ready.
4. It navigates via a verified interface — either the vendor `slam_operate`
   service or a self-built ROS 2 Nav2 stack on the built-in LiDAR's DDS point
   cloud.
5. It detects an obstacle (injected, on a controlled route) and responds within a
   measured latency.
6. It confirms arrival at a visual fiducial and notifies the recipient.
7. The dashboard records completion, duration, interventions, and any failure.

**What makes this scientifically interesting rather than a toy:** it is a
*repeatable measurement study*. The team publishes task completion rate, average
task duration, navigation error, obstacle-response latency, and human
intervention count, collected under a written protocol, with a stated confidence
interval. Most exhibition demos have none of that. This one has a methods
section.

**Two routes to the navigation stack, and this is a real decision:**

- **Route A — vendor path (lower risk, higher lock-in, extra hardware).** Go2 EDU +
  expansion dock (Orin Nano 40 TOPS or Orin NX 100 TOPS) + **separately purchased
  MID-360 or XT16 LiDAR**. Uses the documented `slam_operate` service. Note the
  built-in L2 is *not* the navigation LiDAR. Expect to obtain a YSN from Unitree
  per LiDAR/robot combination.
- **Route B — self-built (more work, no extra hardware, no lock-in).** Go2 EDU with
  the built-in L2, whose point cloud **is already published on DDS**
  (`rt/utlidar/cloud_deskewed`, ROS 2 `PointCloud2`-compatible), fed into
  `slam_toolbox` / FAST-LIO2 + `AMCL` + `Nav2`. All BSD/Apache. No purchase, no
  YSN, and the navigation is *our* engineering — which is the better exhibition
  story.

**Recommendation: build Route B, keep Route A as a documented fallback.**

---

## 6. Biggest technical risk

**Navigation reliability in a real school corridor, on the built-in LiDAR, with
people present — and the fact that we may not be permitted to buy the LiDAR the
vendor's own navigation service requires.**

Nobody has published a repeatable-delivery success rate for a Go2 in a school.
The vendor's navigation service requires hardware we may not obtain. The
alternative — building the stack ourselves — is unproven in our hands until we do
it. This risk cannot be retired by research; it can only be retired by Phase 3 of
the roadmap.

The G1's biggest risk is different and worse: **the robot falls over mid-demo and
cannot get up**, which Unitree's own documentation anticipates by requiring a
protective frame and rope for every operation.

---

## 7. Most important missing information

In priority order. Items 1–3 change the recommendation; items 4–6 change scope
and cost.

1. **Which exact G1 variant does the school own — standard or EDU?** If standard,
   the G1 option is closed on day one (`Secondary Development: /`).
2. **Which exact Go2 variant — AIR, PRO, X, or EDU?** Only EDU is confirmed
   programmable. X's "basic support" is undefined by Unitree and must be confirmed
   in writing before selecting.
3. **Does the Go2 EDU have an expansion dock, and does the school permit
   purchasing a MID-360 or XT16 LiDAR?** This decides Route A vs Route B.
4. What is on the robot already — carrier tray? Compute dock? Extra sensors?
5. What is the school's network policy? **The G1 has known fleet-wide
   cryptographic and telemetry issues** (a shared hardcoded AES key, and
   telemetry to a `43.175.228.x` address every five minutes). Some institutions
   will refuse to admit such a device to the network. This is a hard gate on the
   G1 independent of everything else.
6. What is the demonstration space — floor surface, corridor width, thresholds,
   ramps, crowd density?

---

## 8. Recommended software stack

Permissive licences throughout. No GPL, no unlicensed code.

| Layer | Choice | Licence |
|---|---|---|
| Robot interface | `unitree_sdk2` (C++) or `unitree-sdk2` (PyPI, import `unitree_sdk2py`) | BSD-3-Clause |
| ROS 2 bridge | `unitree_ros2` | BSD-3-Clause |
| Simulation | `unitree_mujoco` — same DDS channels as hardware, CPU-only | BSD-3-Clause |
| Navigation | Nav2 + `slam_toolbox` + `AMCL` (Route B) | BSD / Apache-2.0 |
| LiDAR odometry | FAST-LIO2 or LIO-SAM — **avoid `point_lio_unilidar`, which is GPL-2.0 and ROS 1** | MIT / BSD |
| Task orchestration | BehaviorTree.CPP or a plain deterministic state machine | MIT / BSD |
| Perception | RT-DETR / YOLO on the 720p camera stream | per-model — verify |
| Language layer | Any LLM, **strictly behind the validation boundary** | — |
| Backend | Python + FastAPI, SQLite | MIT / BSD |
| Frontend | Any web framework, WebSocket to backend | — |
| Hardware | Ubuntu 20.04 or 22.04 host, **wired Ethernet**, static `192.168.123.222/24` | — |

**Avoid:** `unitree_slam` (no licence, closed binary, wrong robots, 2 stars),
`unitree_ros_to_real` (no licence, ROS 1, dead), `unitree_pybullet` (no licence,
dead since 2020), `point_lio_unilidar` (GPL-2.0 + ROS 1), and Isaac Gym
(deprecated by NVIDIA in favour of Isaac Lab).

**Note on ROS 2 versions:** official Unitree CI covers **Foxy and Humble only**.
Jazzy is the current LTS. Expect port friction, and prefer Humble for the robot-side
nodes if the host allows it.

---

## 9. The next five concrete actions

1. **Physically identify both robots.** Record, per unit: exact model badge
   (G1 vs G1 EDU; Go2 AIR/PRO/X/EDU), serial, firmware version, and whether a
   depth camera, expansion dock, or compute module is present. Checklist in
   `hardware_configuration_matrix.md` §6. *Nothing else can proceed until this is
   known.*
2. **Get written confirmation from Unitree on Go2 X** — if the school owns an X,
   ask specifically: does `◕` "basic support" permit `unitree_sdk2` and
   `unitree_ros2` usage, and are the RJ45 / DC-out / SBUS interfaces present?
   Do not assume a firmware key will open it.
3. **Establish a read-only DDS connection on the development machine** and
   subscribe to `rt/sportmodestate` and `rt/lowstate`. Record battery level, FSM
   state, and fault flags. This is Phase 2 of the roadmap and requires no
   actuation. It proves the network, the NIC binding, and the topic set.
4. **Run the `unitree_mujoco` Go2 simulation and put a `unitree_ros2` node on it.**
   Same DDS channels as the real robot, CPU-only, no hardware. This validates the
   entire software stack at zero risk and zero cost before touching the robot.
5. **Write the one-page demonstration protocol** — the controlled route, the
   exclusion zone, the obstacle injection point, the measurement definitions, and
   the abort criteria. Then confirm with the school that the space and the
   supervision arrangement match it.

---

## 10. One paragraph to give the school

We recommend the **Go2 EDU**. Both robots are restricted to their EDU models before
any custom software is possible, so we need to confirm which units the school holds.
The Go2 is the only one of the two with a documented, programmable autonomous
navigation interface, and it carries roughly three times the G1's per-limb load on
a stable, low centre of mass. The G1's arms are its genuine advantage, but the only
independently measured pick-and-place success rate on a real G1 is 20–30% per
attempt, and its manufacturer requires a protective frame and safety rope for
normal operation. For a project whose purpose is *reliable, repeatable autonomous
service in a school*, that reliability gap decides the matter. We will build the
navigation stack ourselves on the robot's own LiDAR data rather than depend on a
vendor-locked service, which keeps the engineering ours and keeps the hardware
cost at zero.
