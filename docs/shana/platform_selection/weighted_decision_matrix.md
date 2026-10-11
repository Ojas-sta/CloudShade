# Weighted Decision Matrix

Scored comparison with per-score evidence, three separate rankings, and a
sensitivity analysis that identifies precisely where the recommendation changes.

Collected 2026-10-10. Full evidence in `robot_comparison.md`,
`sdk_and_software_research.md`, and `source_register.md`.

---

## 0. The assumption that dominates everything

**All scores assume EDU models.** A standard G1 lists `Secondary Development: /`
and a Go2 AIR or PRO has no secondary development interface at all. Scoring a
non-EDU robot on software accessibility would be scoring a product that cannot
run the project. §6 handles that case separately.

---

## 1. Scoring rubric

| Score | Meaning |
|---|---|
| 9–10 | Fully supported by verified interfaces, works out of the box |
| 7–8 | Supported; some significant custom work, no unknowns |
| 5–6 | Possible but requires substantial development with material unknowns |
| 3–4 | Marginal; major unknowns or major work; unreliable |
| 1–2 | Not achievable, or contradicted by evidence |
| 0 | Physically impossible on this platform |

---

## 2. Criterion 1 — Practical school-task automation (20%)

| | G1 EDU | Go2 EDU |
|---|---|---|
| **Score** | **2** | **8** |

**Evidence for Go2 (8):** ≈8 kg rated payload (max ~12 kg) on the body, with
field reports of ~3 kg centred on the spine producing no measurable gait change.
Steps over 16 cm thresholds. Handles 40° slopes. `ObstaclesAvoidClient` exposes
`MoveToAbsolutePosition` in the world frame, so waypoint following is a direct
call. Autonomous navigation via `slam_operate`. Battery 15,000 mAh on EDU.
Low CoM makes payload carriage stable by construction.

**Evidence for G1 (2):** arm load 2–3 kg and posture-dependent by Unitree's own
footnote. **Measured** maximum sustained unbraced end-effector force is **13.5 N
(≈1.38 kgf)** — arXiv 2609.25486. Carrying requires holding load at the end of a
0.45 m arm span while walking on 12 DOF, which is the configuration in which the
documented overheating → shutdown → fall occurs. **No autonomous navigation
interface is published for the G1.** Independent measurement puts real walking
runtime at ~90 minutes, less with manipulation.

**Uncertainties:** Go2's usable payload figure (~3 kg) is from community field
data, not Unitree. Unitree publishes no CoM envelope. The G1's 13.5 N is a single
measurement under one bracing condition.

**Assumes extra hardware?** Go2: yes — a carrier tray (~low cost, fabricated).
G1: yes — Dex3-1 hands and an EDU compute module.
**Assumes custom software?** Go2: navigation stack, significant. G1: navigation
stack *plus* a manipulation stack — both custom, neither available.

---

## 3. Criterion 2 — Autonomous mobility and navigation (15%)

| | G1 EDU | Go2 EDU |
|---|---|---|
| **Score** | **3** | **8** |

**Evidence for Go2 (8):** Unitree publishes a documented navigation service
`slam_operate` v`1.0.0.1` with API IDs 1801 start mapping, 1802 end mapping,
1804 init pose, 1102 pose navigation, 1201 pause, 1202 resume, 1901 close slam —
plus published operating limits (0.2–1.0 m/s, ≥0.8 m navigable width,
obstacles ≤0.5 m wide and ≥20 cm tall, target within 10 m). Separately,
`ObstaclesAvoidClient` provides `Move(x,y,yaw)` with documented ranges,
`MoveToIncrementPosition(±2.0 m)`, and `MoveToAbsolutePosition(world frame)`.
Built-in 4D LiDAR (21,600 samples/s, 360°×96°, 0.05 m min) publishes point clouds
to DDS on `rt/utlidar/cloud_deskewed` in a ROS 2 `PointCloud2`-compatible type.
Gaits exist for stairs, gravel, and slippery surfaces.

⚠️ The vendor `slam_operate` path requires **EDU + expansion dock + separately
purchased MID-360 or XT16 LiDAR**. The built-in L2 is **not** the navigation
LiDAR.

**Evidence for G1 (3):** `LocoClient` provides walking with a documented speed cap
of 3.0 m/s (`SetSpeedMode(3)`). **No SLAM service, no navigation service, no
localisation service, no obstacle-avoidance service is published for the G1.**
`unitree_ros2` — the official ROS 2 bridge — depends on no navigation package
whatsoever. Reaching a destination requires building the entire stack. `GetMimicMotion`
and `RecoveryStand` cover physical recovery only.

**Uncertainties:** Go2's `slam_operate` was still receiving patches in April 2026
and needs a Unitree-issued YSN. Its interaction with a ROS 2 graph is
undocumented. G1 navigation is theoretically possible via community work
(`grove-g1`), but that is one person's multi-month effort.

**Assumes extra hardware?** Go2: **yes for Route A** (dock + LiDAR, ~$2–3k).
**No for Route B** (self-built Nav2 on the built-in L2's DDS point cloud).
G1: no extra hardware, but a compute module is strongly implied.
**Assumes custom software?** Go2 Route A: no. Route B: substantial.
G1: substantial in all cases.

---

## 4. Criterion 3 — Object manipulation and laboratory assistance (15%)

| | G1 EDU | Go2 EDU |
|---|---|---|
| **Score** | **6** | **0** |

**Score 0 for Go2 is not a judgement — it is an absence of hardware.** No variant
of the Go2 has an arm (VERIFIED from the product page and from Unitree's own arm
pages, which pair the Z1 with Aliengo/B1 and the D2 with A2/As2, never with Go2).
Any Go2 manipulator would be an unsupported third-party build.

**Why the G1 only scores 6 and not 9:** it is the only platform that *can*
manipulate, and it has real arms, real hands, real tactile sensing, and a genuine
research ecosystem. But:

| Limitation | Evidence |
|---|---|
| **20–30% best published full-task pick-and-place** | arXiv 2610.08119, real G1, held-out object |
| **~7% for native foundation-model action** | same |
| **13.5 N sustained unbraced force** (≈1.38 kgf) | arXiv 2609.25486, measured |
| **No published grasp rate for the Dex3-1** | Both strong results used BrainCo hands. NOT FOUND |
| **No flat-sheet grasping data** in Unitree's ~180 datasets | Pick family is apple/bottle/charger/gum/snack/tissue/doll/drink/toothpaste; grasp family is square/Oreo/Rubik's cube |
| **Cannot reach standard workbenches** | 1,320 mm tall |
| **High-level arm API is 16 opaque animations** | `G1ArmActionClient.action_map` |
| **No `ArmAction` struct, no IK, no MoveIt, no grasp synthesis** | NOT FOUND in any official repo |
| **Hand coupled to locomotion** | Unitree: "not recommended to perform overly intense motions with the dexterous hand installed, such as running or balance tests" |
| **Height limit** | Cannot reliably reach and manipulate simultaneously with a load |

**The strongest positive result** is 61–84% *completion* on 5 held-out objects
with a **fixed** VLA plus a contact-guided supervisor (arXiv 2610.04391) — real G1,
BrainCo Revo2 hand. That is genuine, but it is a task-specific trained policy with
a runtime supervisor, not generalist manipulation, and not the hand the school
would own.

**Assumes extra hardware?** G1: yes — Dex3-1 hands (EDU Ultimate A/B), plus
protective frame and rope, plus VR hardware for teleoperation.
**Assumes custom software?** Yes, overwhelmingly — every layer from joint control
to grasp verification.

---

## 5. Criterion 4 — Software-development accessibility (15%)

| | G1 EDU | Go2 EDU |
|---|---|---|
| **Score** | **6** | **8** |

**Evidence for Go2 (8):** `SportClient` offers 41 typed methods with clean
velocity semantics (`Move(vx, vy, vyaw)`). Python ships runnable
`obstacles_avoid` examples that C++ lacks. LiDAR data needs no client — generic
`ChannelSubscriber` on a ROS 2-compatible type. **`unitree_mujoco` simulates the
Go2 on CPU only, on the same DDS channels as the hardware**, so the whole stack
can be built and debugged with zero hardware and zero cost. The Python README is
broadly accurate. A mature community ecosystem exists (1057★ `go2_ros2_sdk`,
374★ `BotBrain`, 181★ `go2_ros2_toolbox`).

**Evidence for G1 (6):** The G1 has the **richest official ROS 2 example set**
(8 nodes including arm SDK, Dex3, dual-arm, audio) — genuinely better here. But:
the Python `LocoClient` is a strict subset (no `StandUp()`, no `Squat()`), and
the Python README's parity claim is false. `G1ArmActionClient` is missing 2 of 4
APIs in Python. C++ and Python disagree on action IDs and JSON payload keys.
Real joint control requires raw DDS, hand-transcribed joint indices, mandatory
CRC32, a motion-switcher release, and a physical gamepad interlock.
`unitree_mujoco` supports the G1 at **low level only** — there is no high-level
walking controller in simulation, so a locomotion policy must be trained or
sourced. That is a large, GPU-dependent barrier for a school team.

**Uncertainties:** Both platforms are pinned to ROS 2 **Foxy and Humble only**
(confirmed by CI); Jazzy is the current LTS. `unitree_sdk2` ships a closed ~28 MB
static library neither team can audit.

**Assumes extra hardware?** Go2: no. G1: a compute module is strongly implied.
**Assumes custom software?** Go2: navigation stack (Route B) or none (Route A).
G1: navigation **and** manipulation stacks.

---

## 6. Criterion 5 — Reliability and repeatability (15%)

| | G1 EDU | Go2 EDU |
|---|---|---|
| **Score** | **2** | **7** |

**Evidence for G1 (2) — five independent sources:**

1. **Munro & Associates teardown** (MotorTrend, 2026-06-26): a base G1 was "very
   buggy and needed several software updates before it would do anything," and
   after being pushed over "proved unable to stand back up." Non-strain-wave
   joints with backlash; exposed greased ankle Cardan joint; *"definitely not
   ready for prime time."*
2. **Auckland University of AUT** (2026-09): a university lab's G1 described as
   *"stumbling heavily… This instability makes the robot unusable."* Hip motors
   needed replacement; **Unitree's engineer, present in the lab, called back to
   the factory in China**.
3. **NVIDIA `GR00T-WholeBodyControl` issue #121**, open since 2026-04-27: G1+Dex3
   overheats "after a few minutes, **even when it is just standing idle**," then
   auto-shutdown, then *"the robot has fallen multiple times."* A follow-up five
   months later reads *"no finger use for now, but it's still not perfect."*
4. **A $100k research buyer abandoned the platform** (2026-02) after a knee fault
   requiring a **2-month parts wait** plus "frequent hardware failures,
   particularly in its arms."
5. **No fall-rate statistic has ever been published** by anyone. NOT FOUND.

**Evidence for Go2 (7):**

1. **Teardown endorsement**: *"emphasis on robustness and long-term
   durability… leg modules designed for rapid field replacement… prioritised
   vibration resistance, shock absorption, and overall impact survivability."*
   Unitree's most field-proven actuator line.
2. **Deployed in anger**: Hangzhou Asian Games javelin/discus transport. Unitree
   ships a quadruped inspection product line; no comparable G1 product ships.
3. **Self-righting**, with `RecoveryStand()`, `AutoRecoverSet()`, and documented
   recovery gaits.
4. **But** — an independent operator reports failing to stand on first power-up
   three times, "flailing its legs around wildly"; powered off without warning
   mid-movement; and "run into a door … while it was supposed to be in autonomous
   navigation and obstacle avoidance mode."
5. **No published report of tipping under payload.** NOT FOUND.
6. **No rigorous independent runtime test.** NOT FOUND.

**Uncertainties:** Go2's 7 reflects a design and deployment record, not a
controlled repeatability study. Neither robot has a published reliability figure.
**This score is the least well-evidenced of the seven and should be revisited if
either team can run an endurance test.**

**Assumes extra hardware?** Neither.
**Assumes custom software?** Neither — this is hardware and firmware behaviour.

---

## 7. Criterion 6 — Safety and suitability for a supervised school demonstration (10%)

| | G1 EDU | Go2 EDU |
|---|---|---|
| **Score** | **2** | **6** |

**Evidence for G1 (2):** Unitree's own G1 User Manual:

> ⚠️ "**Do not use the emergency braking function when the robot is not protected
> by the protective frame and protective rope provided by Unitree** in the software
> manual. Otherwise, it will cause the robot to fall to the ground and cause
> serious damage!!!"

> ⚠️ "In developer mode, the user must ensure that the robot protects the robot by
> the **protection frame and the protection rope**."

> "When users use their own programs, there is a lot of uncertainty, the robot may
> do some unexpected actions (**Uncontrolled to the crowd**…)"

> "This product is not a toy and is not intended for use by persons under the age
> of 18."

Plus: ≥2 m mandated safe distance from crowds; 35 kg mass; no ISO 10218 or
ISO/TS 15066 certification; cannot self-recover from a fall.

**Evidence for Go2 (6):** 15 kg, low CoM, no rigid upper body, self-righting.
Real injuries would be trips and leg pinching, not crushing. But:
`FreeAvoid` documentation states the robot *"can dodge objects in front"* when
stationary — an **uncommanded lateral motion**, a genuine bystander hazard.
**No vendor safety case exists.** No certification. Operating temperature 5–35 °C.

**Uncertainty:** neither robot has any independent safety assessment.

**Assumes extra hardware?** G1: **yes — a protective frame and rope, mandatory.**
**Assumes custom software?** Neither.

**Note:** both robots require an exclusion zone, a dedicated spotter, and a
documented abort procedure. That cost is identical and is not a differentiator.

---

## 8. Criterion 7 — Exhibition impact and educational value (10%)

| | G1 EDU | Go2 EDU |
|---|---|---|
| **Score** | **9** | **7** |

**Evidence for G1 (9):** A 1.32 m humanoid draws an immediate crowd. It answers
the question every visitor has — "how does it not fall over?" Its depth camera,
LiDAR, 4-microphone array, and **offline ASR and TTS on DDS** (service `voice`,
APIs 1001/1002 — a genuine capability the Go2 lacks) support a rich interactive
story. Depth camera plus 3D LiDAR is the richer perception story.

**Evidence for Go2 (7):** A quadruped is still visually striking, and it reads as
a *working machine* rather than a prototype — arguably better for a judging panel
concerned with engineering. It has a compelling industrial narrative: this is the
platform Unitree actually productises. But it does not generate the same
"look at that" reaction, and its audio interface is limited to volume and
brightness.

**Assumes extra hardware?** Neither.
**Assumes custom software?** Either, for a rich interaction story.

---

## 9. Weighted totals

### Main matrix — the brief's stated weights

| # | Criterion | Weight | G1 | G1 × w | Go2 | Go2 × w |
|---|---|---|---|---|---|---|
| 1 | Practical school-task automation | 20% | 2 | 4.0 | 8 | 16.0 |
| 2 | Autonomous mobility & navigation | 15% | 3 | 4.5 | 8 | 12.0 |
| 3 | Object manipulation & lab assistance | 15% | 6 | 9.0 | 0 | 0.0 |
| 4 | Software-development accessibility | 15% | 6 | 9.0 | 8 | 12.0 |
| 5 | Reliability & repeatability | 15% | 2 | 3.0 | 7 | 10.5 |
| 6 | Safety for a supervised demo | 10% | 2 | 2.0 | 6 | 6.0 |
| 7 | Exhibition impact & educational value | 10% | 9 | 9.0 | 7 | 7.0 |
| | **TOTAL** | **100%** | | **40.5** | | **63.5** |

### **Unitree Go2 EDU wins, 63.5 / 100 against 40.5 / 100.**

---

## 10. Ranking A — Delivery-focused automation

Weights re-derived for the delivery question: automation 35%, navigation 30%,
reliability 20%, safety 15%.

| Criterion | Weight | G1 | G1 × w | Go2 | Go2 × w |
|---|---|---|---|---|---|
| Practical school-task automation | 35% | 2 | 7.0 | 8 | 28.0 |
| Autonomous mobility & navigation | 30% | 3 | 9.0 | 8 | 24.0 |
| Reliability & repeatability | 20% | 2 | 4.0 | 7 | 14.0 |
| Safety for a supervised demo | 15% | 2 | 3.0 | 6 | 9.0 |
| **TOTAL** | 100% | | **23.0** | | **75.0** |

### **Go2 wins decisively, 75.0 against 23.0.**

The gap widens from 23 points to 52 points once the question is narrowed to
delivery. This is not a marginal preference — on delivery specifically, the G1 has
no published autonomous navigation interface at all, while the Go2 has two.

---

## 11. Ranking B — Manipulation-focused laboratory assistance

Weights: manipulation 45%, automation 15%, navigation 15%, dev access 15%,
reliability 10%.

| Criterion | Weight | G1 | G1 × w | Go2 | Go2 × w |
|---|---|---|---|---|---|
| Object manipulation & lab assistance | 45% | 6 | 27.0 | 0 | 0.0 |
| Practical school-task automation | 15% | 2 | 3.0 | 8 | 12.0 |
| Autonomous mobility & navigation | 15% | 3 | 4.5 | 8 | 12.0 |
| Software-development accessibility | 15% | 6 | 9.0 | 8 | 12.0 |
| Reliability & repeatability | 10% | 2 | 2.0 | 7 | 7.0 |
| **Raw total** | 100% | | **45.5** | | **43.0** |

### **G1 wins narrowly on raw score — but Go2 is INELIGIBLE for this ranking.**

This is the most important nuance in the matrix, and it must not be read as
"45.5 vs 43.0, basically even."

**A weighted average cannot express "physically impossible."** The Go2 scores 0 on
the defining criterion because *no Go2 variant has an arm* — verified from the
product page and from Unitree's own arm-compatibility pages, which pair the Z1
with Aliengo/B1 and the D2 with A2/As2 and never with the Go2. A Go2 carrying a
manipulation-led concept is not a worse solution; it is not a solution.

**Correct reading:** for a project whose *defining task* is picking something up,
the G1 is the only eligible platform of the two. Its score of 45.5 also reflects
that it would be an expensive, unreliable way to do it — 20–30% published
pick-and-place, 13.5 N measured force, unresolved thermal-fall defect.

**For laboratory assistance that does not require manipulation** — transporting
equipment, carrying sensor payloads, mobile measurement, cleaning — the Go2 is
the better platform by a wide margin, and that is what Concepts 2 and 4 in
`exhibition_project_ideas.md` are built on.

---

## 12. Ranking C — Overall school exhibition value

The brief asks for a separate overall exhibition ranking. Under the main weights
this is identical to §9:

| | Total |
|---|---|
| **Go2 EDU** | **63.5** |
| G1 EDU | 40.5 |

**Go2 EDU wins.**

### Overall recommendation

**Select the Go2 EDU.**

The project vision is dominated by logistics and delivery, which is where the
platform difference is widest (75.0 vs 23.0). Laboratory assistance can be
delivered *without* manipulation — transport, mobile sensing, and mobile
measurement all work on a quadruped. The G1's manipulation, which is its one
genuine advantage, is the single least reliable capability either platform
offers, and it would be the load-bearing step of the entire project.

---

## 13. Sensitivity analysis — precisely where this flips

Six weightings, each summing to 100%. All scores held constant; only weights vary.

| # | Framing | Weights (A/N/M/D/R/S/E) | G1 | Go2 | Winner |
|---|---|---|---|---|---|
| S1 | **Brief as written** | 20/15/15/15/15/10/10 | **40.5** | **63.5** | **Go2** |
| S2 | Production / reliability emphasis | 25/20/8/15/20/8/4 | 34.0 | 69.6 | **Go2** |
| S3 | Equal weights | 14.3 each | 42.9 | 62.9 | **Go2** |
| S4 | Drop-manipulation emphasis | 28/22/5/16/18/8/3 | 32.7 | 72.3 | **Go2** |
| S5 | **Science-fair spectacle** | 10/5/35/8/12/5/25 | **55.2** | 47.3 | **G1** |
| S6 | **Manipulation-maximal** | 5/3/45/7/10/5/25 | **58.6** | 39.5 | **G1** |

*(A = automation, N = navigation, M = manipulation, D = dev access,
R = reliability, S = safety, E = exhibition impact.)*

### Flip point

Interpolating between S1 and S5, the crossover sits at roughly:

| Weights | G1 | Go2 |
|---|---|---|
| manip 30, exhib 20, auto 10, nav 8, dev 10, rel 15, safety 7 | 50.8 | 51.1 |
| manip 32, exhib 20, auto 9, nav 7, dev 10, rel 15, safety 7 | 51.5 | 49.5 |

> **The G1 overtakes only when object manipulation carries ≥ ~31% of total
> weight AND exhibition impact carries ≥ ~20%, while reliability still retains
> meaningful weight.**

**That combination describes a science-fair spectacle. It does not describe the
project vision in the brief**, which is "a robot that can perform repetitive
school tasks, transport materials, assist teachers, and support laboratory
activities" — a utility specification, not a spectacle one.

### Robustness check on the headline scores

All rows hold every other score and weight constant, at the brief's baseline.

| Perturbation | G1 | Go2 | Winner |
|---|---|---|---|
| Go2 reliability 7 → 5 | 40.5 | 60.5 | Go2 |
| Go2 navigation 8 → 6 | 40.5 | 60.5 | Go2 |
| Go2 automation 8 → 6 | 40.5 | 59.5 | Go2 |
| Go2 safety 6 → 4 | 40.5 | 61.5 | Go2 |
| G1 manipulation 6 → 8 | 43.5 | 63.5 | Go2 |
| G1 exhibition 9 → 10 | 41.5 | 63.5 | Go2 |
| Go2 automation 8 → 6 **and** navigation 8 → 6 | 40.5 | 57.5 | Go2 |
| Go2 reliability 7 → 4 **and** navigation 8 → 6 | 45.0 | 56.0 | Go2 |
| **All six of the above simultaneously** | **45.5** | **56.0** | **Go2** |

The recommendation is **robust to substantial error in every individual score**.
Even compounding every adverse change — the Go2 being simultaneously less
reliable, less navigable, less automatable and less safe than evidenced, *while*
the G1 is better at manipulation and even more impressive than scored — the Go2
still wins by 10.5 points.

It would take an error larger than any plausible reading of the evidence to
change the answer. That is the confidence level this research supports.

---

## 14. Hardware-reality variants

If the school does not own EDU models, the matrix changes as follows.

### Standard G1 ($13,500)

| Criterion | Score | Change |
|---|---|---|
| Dev access | 6 → **0** | `Secondary Development: /` |
| Manipulation | 6 → **0** | No hands (`Single Hand DOF: /`) |
| Automation | 2 → **1** | No compute module |
| Navigation | 3 → **2** | No compute module |
| Reliability / Safety / Exhibition | 2 / 2 / 9 unchanged | Most generous reading — a standard G1 still walks, and can be app-teleoperated |
| **TOTAL** | | **19.0 / 100** |

**The standard G1 cannot run this project.** This is not a score — it is a
closed door. The 19.0 is the *most generous* treatment available; a stricter
reading, in which reliability and safety-under-autonomy also score 0, gives
**14.0**. Either way the outcome is identical: excluded.

### Go2 AIR ($1,600) / PRO ($2,800)

| Criterion | Score | Why |
|---|---|---|
| Automation, Navigation, Dev access, Manipulation | **0** | No secondary development interface |
| Reliability under autonomous operation | **0** | It cannot be given an autonomous task |
| Safety under autonomous operation | **0** | Same |
| Exhibition impact as a physical object | **7** | Retained — it is still a striking machine |
| **TOTAL** | **7.0 / 100** | |

**Closed.** No secondary development interface, and no RJ45 / DC-out / SBUS
hardware to attach one to.

### Go2 X ($4,500)

| Criterion | Score |
|---|---|
| Dev access | **UNVERIFIABLE** — `◕` "basic support" has no official definition |

**Do not select X on assumption.** Ask Unitree in writing: does `◕` permit
`unitree_sdk2` and `unitree_ros2` usage, and are the RJ45 / DC-out / SBUS
interfaces physically present? If the answer is no, the X is a $4,500 paperweight.

### If the school owns a G1-D

A G1-D changes the analysis materially: it is a wheeled chassis with the G1 arms,
~3–5× the price, and its vendor autonomy stack (SLAM, navigation, ±5 cm
docking, HTTP REST API, RoboStudio) is documented as a locked product with
chassis data ports marked "not currently open."

| Criterion | G1-D score (provisional) |
|---|---|
| Automation | 9 |
| Navigation | 7 (vendor-locked, not programmable via SDK) |
| Manipulation | 5 (arms, but 13.5 N and 20–30% still apply) |
| Dev access | 4 |
| Reliability | 6 (no legs — cannot fall) |
| Safety | 7 (no fall hazard) |
| Exhibition | 8 |
| **TOTAL** | **~66 / 100** — comparable to the Go2, from a different direction |

⚠️ **This is provisional and depends entirely on whether the vendor HTTP REST API
is actually accessible.** That contract is unpublished. **Verify before
selecting.** If the school owns a G1-D, re-run this analysis.

---

## 15. What would change the recommendation

| Finding | Effect on recommendation |
|---|---|
| School owns Go2 AIR / PRO | Go2 option **closed** — re-run with G1 EDU only, which yields ~40.5 and a weak project |
| School owns Go2 X, `◕` confirmed insufficient | As above |
| School owns Go2 **EDU** | **Recommendation confirmed** |
| School owns G1 **standard** | G1 option **closed** |
| School owns G1 **EDU** | Go2 still recommended (63.5 vs 40.5) |
| School owns **G1-D** | Re-run — potentially comparable, but blocked on the HTTP API contract |
| School requires a humanoid for presentation | Brief change, not a technical argument. Disclose the 20–30% manipulation figure to judges up front. |
| IT policy blocks devices with known telemetry | G1 excluded on governance grounds regardless of merit |
| Go2 EDU turns out to have the 8,000 mAh battery | Endurance margin halves; plan for 1–2 h |
| Navigation proves unreliable in Phase 3 | Fall back to Concept 5 (stationary classroom assistant), which needs no locomotion |

---

## 16. Confidence statement

**High confidence** — the recommendation follows from four independently verified
facts: the G1 spec table's `Secondary Development: /`; Unitree's Go2 FAQ stating
EDU-only development; Unitree's documented `slam_operate` navigation service for
the Go2 and its absence for the G1; and the published 20–30% G1 pick-and-place
rate from a peer-reviewed preprint with real-hardware closed-loop trials.

**Moderate confidence** — the reliability scores (G1 2, Go2 7). These rest on
teardown reports, forum reports, and one university lab rather than controlled
testing. **No fall-rate statistic exists for either robot.** This is the weakest
part of the matrix.

**Low confidence** — the manipulation scores for the specific Dex3-1 hand. No
published grasp rate exists for it. The G1's 6/10 rests on results obtained with
different hardware.

**Not assessed** — cybersecurity. The G1 has documented fleet-wide cryptographic
and telemetry issues which would likely be disqualifying under most institutional
IT policies. This was not scored because it is a governance gate, not a
performance attribute, but it should be checked with the school immediately.
