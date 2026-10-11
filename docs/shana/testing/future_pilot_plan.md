# Implementation Roadmap

Nine phases. **This is a plan, not a description of work done.** No code has been
written and no robot has been touched.

**Go2 track and G1 track are genuinely different.** §10 gives both.

**Hard rule: no phase begins until the previous phase's hold points pass.**

---

## Phase 1 — Confirm the exact robot model and hardware

**Goal:** establish whether this project is possible at all.

### Why this is first

Both platforms are development-gated. A standard G1 lists
`Secondary Development: /`. A Go2 AIR or PRO has no secondary development
interface. **If the school owns a non-EDU unit, nothing else in this roadmap
matters.**

### Tasks

1. **Physically identify both robots.**
   Photograph each nameplate. Record the exact model string:
   - G1 vs **G1 EDU**
   - Go2 **AIR / PRO / X / EDU**
2. Record serial numbers and firmware versions (app → Device → Service Status).
3. **Go2: confirm the expansion dock is present, and which Orin variant.**
4. **Go2: confirm which battery is fitted** — 8,000 mAh or 15,000 mAh.
5. **Go2 EDU: confirm the depth camera is present** (EDU-only feature).
6. **G1: confirm whether hands are fitted**, and which generation — Dex3-1
   (with or without tactile), Inspire DFQ, or Inspire FTP. If the G1 is Ultimate
   A vs B, the difference is tactile sensors.
7. **G1: confirm the protective frame and protective rope exist.** Without them,
   the G1 cannot be operated at all.
8. Check for existing attachments: carrier tray, third-party sensors, compute
   modules.
9. **Measure the Go2 spine mounting area** — Unitree publishes this as a PNG only.
10. **Survey the demonstration space**: corridor width, floor surface, threshold
    heights, ramp gradients, available power, network drop points.

### Hold point

- [ ] Both robots identified by model
- [ ] **At least one robot confirmed to be an EDU model**
- [ ] Demonstration space surveyed and recorded
- [ ] If Go2 X: written confirmation from Unitree obtained (§1.1)

### If the school owns a non-EDU unit

Stop. Escalate with:
- Unitree's own quote for the Go2 EDU (the recommendation), or
- Unitree's own statement that the G1 EDU is required, or
- A decision that the project is deferred pending hardware

**Do not attempt to work around the gate.** The `unitree_legged_sdk` route used
by old tutorials is not an EDU substitute.

### 1.1 — The Go2 X question

If the school owns an **X** (`◕` "basic support"), ask Unitree **in writing**:

1. Does "basic support" permit use of `unitree_sdk2` and `unitree_ros2`?
2. Are the RJ45, DC 28.8 V output, and SBUS interfaces physically present?
   (Official docs say they are EDU-exclusive.)
3. Is the `slam_operate` service available on X?
4. Is there a documented difference between X and EDU firmware?

**Do not select X on the assumption that a firmware key will open it.** The
documented answer may well be no.

### 1.2 — Early question for the school's IT team

Ask now, because it can disqualify the G1 independently of everything else:

> The Unitree G1 has been independently audited and found to (a) share one
> hardcoded AES cryptographic key across all units, allowing fleet takeover via a
> BLE exploit, and (b) transmit camera, audio, and map telemetry to an external IP
> address in China every five minutes without user consent. Does institutional
> policy permit such a device on our network?

If the answer is no, **the G1 is out** regardless of the rest of this document.

---

## Phase 2 — Read-only connection and status inspection

**Goal:** prove the network, the toolchain, and the topic set. **No actuation.**

### 2.1 Development machine

- Ubuntu 20.04 or 22.04 (⚠️ **Mac and Windows are not supported**)
- **Wired Ethernet** to the robot — Wi-Fi is documented as unreliable for DDS
- Static IP on `192.168.123.0/24`, e.g. `192.168.123.222`
  (⚠️ **never** assign `192.168.123.161` — that is the robot)
- Dedicated NIC if the machine has multiple interfaces; the NIC name is passed
  explicitly to the SDK

### 2.2 Toolchain

```bash
# 1. CycloneDDS (required before SDK2 will import)
git clone https://github.com/eclipse-cyclonedds/cyclonedds -b releases/0.10.x
cd cyclonedds && mkdir build install && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=../install
cmake --build . --target install
export CYCLONEDDS_HOME=~/cyclonedds/install

# 2. SDK2 Python — install FROM GIT, not PyPI
#    PyPI release is ~13 months stale
git clone https://github.com/unitreerobotics/unitree_sdk2_python.git
cd unitree_sdk2_python && pip3 install -e .

# 3. Verify the robot's firmware meets the gate
#    Go2 motion interface V2.0 requires >= V1.1.6
```

### 2.3 Read-only tasks

| # | Task | Interface |
|---|---|---|
| 1 | Subscribe `rt/sportmodestate` — position, velocity, foot forces, BMS | ✅ verified |
| 2 | Subscribe `rt/lowstate` — IMU, motor states, temperatures | ✅ verified |
| 3 | Subscribe `rt/utlidar/cloud_deskewed` — confirm the point cloud flows | ✅ verified |
| 4 | Subscribe `rt/utlidar/height_map_array`, `rt/utlidar/range_info` | ✅ verified |
| 5 | Query `RobotStateClient.ServiceList` — enumerate the robot's services | ✅ verified |
| 6 | Subscribe `/wirelesscontroller` — read the RC without using it | ✅ verified |
| 7 | Log battery voltage, current, SoC, and motor temperatures for 30 min | 🔵 ours |

### 2.4 Also benchmark these unknowns while the robot is stationary

| Question | Why it matters |
|---|---|
| **Camera frame rate — 15, 30, or 60 fps?** | ⚠️ Unitree's docs contradict each other. Determines whether real-time vision is viable. |
| **GStreamer H.264 stream** — establish the pipeline, measure latency | Needed for Concepts 2 and 6 |
| **`VideoClient::GetImageSample`** — measure achievable snapshot rate | Fallback if GStreamer fails |
| **LiDAR point rate and message size** | Sizing the SLAM input |
| **Does `ObstaclesAvoidClient.UseRemoteCommandFromApi` need to precede `Move`?** | ⚠️ Ordering is undocumented |
| **Measured current draw at rest vs under motion** | Basis for the battery budget in Phase 8 |

### Hold point

- [ ] Stable subscription to all state topics for ≥30 min with no drops
- [ ] Point cloud confirmed flowing
- [ ] Camera frame rate **measured and recorded**
- [ ] Battery and motor temperatures logged
- [ ] **Zero actuation commands issued**

### Output

A `docs/shana/hardware/go2_edu_inventory.md` recording: model, serial, firmware, measured camera
fps, measured LiDAR rate, battery behaviour. This file becomes the project's
single source of hardware truth, and it replaces assumptions with measurements.

---

## Phase 3 — Simulation first (zero hardware)

**This phase comes before any physical movement, and it is the single largest
lever on project success.** Because `unitree_mujoco` speaks the same DDS channels
as the real robot, the entire software stack can be built, debugged, and
demonstrated with no robot and no cost.

### 3.1 Bring up the simulator

```bash
git clone https://github.com/unitreerobotics/unitree_mujoco.git
# Go2 model, CPU physics — no GPU required
```

Set `RMW_IMPLEMENTATION=rmw_cyclonedds_cpp` and the loopback interface, exactly
as `unitree_ros2/setup_local.sh` does. Your existing `unitree_ros2` nodes then
run against the simulator **unchanged**.

### 3.2 Build and validate the software stack in simulation

- [ ] Publish `LowCmd` / read `LowState` against the simulated Go2
- [ ] Confirm `SportModeState` is available in simulation (it is, for the Go2)
- [ ] Subscribe the simulated LiDAR point cloud
- [ ] Write the **task manager, validator, and executor** — the entire
      application layer — and exercise it with simulated faults
- [ ] Write the **safety supervisor** and verify it vetoes correctly
- [ ] Write the **dashboard** against simulated telemetry
- [ ] Write the **metrics collector** and run a synthetic 20-task campaign
- [ ] Inject faults: robot unreachable, battery low, destination unknown,
      obstacle present, task timeout

### 3.3 Milestone

**A complete, working, demonstrated software stack that has never touched the
robot.** This is the goal of Phase 3.

⚠️ **G1 caveat:** `unitree_mujoco` supports the G1 at **low level only** — there
is no high-level walking controller in simulation, so a locomotion policy must be
trained or sourced, which is a GPU-bound research task. The G1 team cannot do
Phase 3 this way, which is itself a strong practical argument for the Go2.

### Hold point

- [ ] Full task loop demonstrated in simulation, including a simulated obstacle
- [ ] Validator correctly rejects every adversarial input in the test suite
- [ ] Safety supervisor veto test passes
- [ ] Fault injection produces a clean abort in every case

---

## Phase 4 — Basic movement and navigation, supervised

**Goal:** move the real robot, slowly, in a cleared area, with a spotter.

### Preconditions (all mandatory)

- [ ] Phase 3 hold points passed
- [ ] Phase 2 hold points passed
- [ ] Spotter briefed; RC in hand; RC authority tested
- [ ] Exclusion zone marked
- [ ] Area cleared of cables, rugs, chairs
- [ ] Battery above 40%
- [ ] Com commissioning tests T1–T5 passed (§`safety_and_failure_analysis.md` §8)

### Tasks, in this order — **do not skip ahead**

| Step | Action | Speed |
|---|---|---|
| 1 | Stand / damp / balance transitions | 0 |
| 2 | Velocity command, cleared area, tethered | 0.1 m/s |
| 3 | Emergency stop test | 0.1 m/s |
| 4 | RC takeover test | 0.1 m/s |
| 5 | Build the map (Route B: `slam_toolbox` on the built-in LiDAR) | 0.3 m/s |
| 6 | Navigate a 10 m out-and-back, 10 consecutive runs | 0.3 m/s |
| 7 | Navigate the full demonstration route | 0.3 m/s |
| 8 | Obstacle injection at a known point | 0.3 m/s |
| 9 | Localisation-loss injection | 0.3 m/s |
| 10 | Increase to demonstration speed **only after** everything above is reliable | 0.4–0.6 m/s |

### Deliverables

- A mapped route
- **Measured**: localisation failure rate, obstacle-response latency, success
  rate over 10 loops
- A written list of the robot's actual failure behaviours, observed

### Hold point

- [ ] 10 consecutive successful route completions
- [ ] Emergency stop and RC takeover both verified
- [ ] Obstacle response detected and measured
- [ ] Localisation loss handled cleanly

### Stop conditions

Stop this phase and reassess if: localisation fails more than 2 times in 10 runs;
the robot exhibits uncommanded motion; or any abnormal noise is observed.

**If navigation cannot be made reliable here, pivot to Concept 5**
(stationary classroom assistant), which needs no locomotion. Decide this here,
not on the exhibition day.

---

## Phase 5 — Minimal end-to-end task

**Goal:** one task, start to finish, without human intervention.

### Tasks

- [ ] Fit the carrier tray (test retention tilted to 45° first)
- [ ] Mount arrival fiducials at each destination
- [ ] Implement arrival detection
- [ ] Implement the executor state machine for a single route
- [ ] Implement load confirmation (contact switch, or visual, or operator button)
- [ ] Run 20 consecutive end-to-end tasks
- [ ] Log every outcome with a failure reason

### Deliverables

- **Task completion rate over 20 runs**
- **Average and p95 task duration**
- **Navigation error at arrival** (fiducial pose vs robot pose)
- **A failure taxonomy** from the 20 runs

### Hold point

- [ ] ≥90% completion over 20 runs (an internal target, not a published claim)
- [ ] Every failure has a classified reason
- [ ] No uncommanded motion in any run

---

## Phase 6 — Dashboard, task queue, and perception

### 6.1 Dashboard and queue

- [ ] Web dashboard: task submission, live status, battery, queue, history
- [ ] Task queue with multiple requests, ordering, and deadlines
- [ ] Notification routing (dashboard + speaker)
- [ ] Full audit log

### 6.2 Perception — add only what a concept needs

| Concept | Adds |
|---|---|
| 1 Courier | Arrival fiducial detection |
| 2 Sensor platform | Environmental sensor driver, calibration protocol |
| 3 NL scheduling | LLM extraction, schema validation, registry, confirmation gate |
| 5 Classroom assistant | ASR, corpus index, TTS, refusal logic |
| 6 Vision lab | Detection model, live overlay, evaluation set |
| 7 Digital twin | Telemetry pipeline, time-series store, trend analysis |

⚠️ **Add perception last, and only where it earns its place.** Every added
component is a new failure mode. Concept 3 in particular should be added *after*
the manual dashboard is fully working, so the demonstration never depends on it.

### 6.3 Language layer safety

- [ ] LLM process has **no network route to the robot** — verify by inspection
- [ ] Validator is a pure function with no network access
- [ ] Unknown entities rejected, never guessed
- [ ] Human confirmation on every LLM-derived task
- [ ] Adversarial test suite of ≥100 crafted inputs
- [ ] Kill switch; **the demonstration works with the LLM disabled**

### Hold point

- [ ] Dashboard functional under concurrent use
- [ ] Queue handles 5 pending tasks correctly
- [ ] **Zero unauthorised actions across the adversarial suite**
- [ ] Demonstration completes with the LLM layer switched off

---

## Phase 7 — Error handling, diagnostics, safety supervision

**Goal:** the system fails safely and explains itself.

### Tasks

- [ ] Safety supervisor complete: watchdog, battery floor, heartbeat, speed
      clamp, zone enforcement, operator e-stop
- [ ] Every executor state has a defined abort path
- [ ] Health endpoint: battery, connectivity, topic rates, fault flags
- [ ] Structured logging with correlation IDs from dashboard to robot
- [ ] Alerting for: battery low, localisation lost, comms loss, fault flags
- [ ] Replayable incident log for every run
- [ ] Failover test: kill the host mid-task and verify the robot stops safely

### Commissioning tests T13–T15

- [ ] **Uncommanded-motion check** — robot never moves without a command
- [ ] Endurance run — 2 h continuous operation, no thermal or mechanical fault
- [ ] (G1) Frame + rope procedure verified; damping confirmed; no power-off
      standing

### Hold point

- [ ] Host-kill test passes: robot stops within the watchdog interval
- [ ] Every failure mode in `safety_and_failure_analysis.md` §6 has a tested
      response
- [ ] All logs recoverable and readable

---

## Phase 8 — Repeatability testing and measurement

**Goal:** produce defensible numbers under a written protocol.

### 8.1 Write the protocol first

A one-page methods document, agreed before any data is collected:

- What is being measured
- How many trials
- What counts as a success, a failure, an intervention
- Where the obstacles are injected
- How latency is timestamped
- What is excluded and why

⚠️ **Write this before collecting data, not after.** Writing it afterwards
invites the accusation that the protocol was fitted to the results.

### 8.2 Measurements

| Metric | Method | Trials |
|---|---|---|
| Task completion rate | Completed / submitted | ≥30 |
| Delivery success rate | Correct destination, correct item, undamaged | ≥30 |
| Average and p95 task duration | Wall-clock, submit → confirmation | ≥30 |
| Navigation error | Fiducial pose vs robot pose at arrival | ≥30 |
| Obstacle-response latency | Video timestamp → stop-command timestamp | ≥20 |
| Localisation failure rate | Injected + naturally occurring | ≥30 |
| Human interventions | Count per run | ≥30 |
| Perception accuracy | Detection rate per approach angle | ≥50 samples |
| Payload retention | Drop tests + displacement measurement | ≥20 |
| Battery consumption | Measured, not calculated from spec | ≥5 full routes |

### 8.3 Statistics

Report confidence intervals (Wilson intervals for proportions), not bare
percentages. A 20/20 result means something very different from a 200/200 one.

### 8.4 Publish the failures

**Include the failures, with the failure taxonomy.** A project reporting a 70%
completion rate and a clear taxonomy of why the other 30% failed is stronger
evidence of competence than one reporting 95% with no detail.

### Hold point

- [ ] Protocol written and signed off before data collection
- [ ] ≥30 trials per primary metric
- [ ] Confidence intervals reported
- [ ] Failure taxonomy published alongside the success figures

---

## Phase 9 — Exhibition preparation

### 9.1 Rehearsal

- [ ] **20 consecutive full rehearsals** with no failures — this is the real
      acceptance test
- [ ] Rehearse the failure paths too: kill the network, inject an obstacle,
      force a validation rejection
- [ ] Rehearse with the LLM layer disabled
- [ ] Time every element; build the demo to ~3 minutes of robot time
- [ ] Prepare a **shortened demo** for time pressure

### 9.2 Documentation

- [ ] One-page dashboard explainer for non-technical judges
- [ ] Methods poster: protocol, results, confidence intervals, failure taxonomy
- [ ] Architecture diagram showing the AI/deterministic boundary
- [ ] Safety procedures visible — this signals professionalism to judges
- [ ] README for anyone who wants to build on it

### 9.3 Logistics

- [ ] Battery charging schedule; spare pack charged
- [ ] Spare NIC, cable, and host configured
- [ ] Venue walkthrough with the actual floor surface
- [ ] Fiducials mounted and tested under exhibition lighting
- [ ] **Network verified at the venue, not assumed** — carry a tested fallback
- [ ] Spotter briefed; abort authority explicitly confirmed to them
- [ ] Contingency: if navigation fails on the day, **Concept 5 (stationary) is the
      fallback and must be ready to run**

### 9.4 Final acceptance

- [ ] All Phase 1–8 hold points passed
- [ ] Commissioning tests T1–T15 passed
- [ ] 20 consecutive rehearsals clean
- [ ] Fallback demo tested
- [ ] Safety checklist signed
- [ ] **Every judge-facing claim matches a measured number**

### The claim discipline rule

**Every number shown to a judge must come from the Phase 8 dataset.**

Do not present a target as a result. Do not present a simulation number as a
hardware number. Do not present the vendor's spec sheet as your measurement.
If a number is an estimate, label it as an estimate. This single rule
distinguishes an engineering project from a demonstration, and judges do notice.

---

## 10. Divergent paths

The roadmap is not the same for both robots. Key divergences:

| Phase | Go2 EDU | G1 EDU |
|---|---|---|
| **1** | Confirm EDU, dock, battery, depth camera | Confirm **EDU** (standard is unusable), hands, **frame + rope** |
| **2** | Read `rt/sportmodestate`, `rt/utlidar/*`, camera | Read `rt/lowstate`, `rt/sportmodestate`; ⚠️ no camera or point-cloud client exists — must be sourced |
| **3** | ✅ Full stack in CPU-only MuJoCo | ⚠️ Low-level only — needs a locomotion policy (GPU-bound). **Much harder.** |
| **4** | ✅ Vendor `slam_operate` *or* self-built Nav2 | 🔨 Self-built only; nothing published |
| **5** | ✅ Carrier tray | 🔨 Trays are undocumented; carrying means holding in the hands |
| **6** | Fiducials, sensors, LLM | 🔨 **Plus** the entire manipulation stack: IK, grasp, verification, retry |
| **7** | Standard safety supervisor | 🔨 **Plus** frame/rope confirmation, motion whitelist, attitude watch |
| **8** | Delivery metrics | 🔨 **Plus** grasp success over ≥50 attempts, Wilson CIs, failure taxonomy |
| **9** | Public, supervised | **Laboratory only. Fenced. No students.** |

**The Go2 reaches a demonstrable end-to-end task in Phase 5. The G1 does not reach
a demonstrable manipulation task until Phase 6+, and then only at a measured
20–30% success rate.**

If the G1 is selected regardless, the honest framing is: this is a manipulation
research project with logistics as a secondary goal — the reverse of the brief.

---

## 11. Timeline shape

**Do not read this as a schedule.** These are *sequential dependencies*, and the
durations depend on team size, prior experience, and hardware availability.

```
P1 Confirm hardware         ▓
P2 Read-only connection     ▓▓▓
P3 Simulation               ▓▓▓▓▓▓▓      ← build everything here
P4 Movement + navigation    ▓▓▓▓▓
P5 Minimal end-to-end       ▓▓▓▓
P6 Dashboard + perception   ▓▓▓▓▓▓
P7 Errors + safety          ▓▓▓
P8 Measurement              ▓▓▓▓▓
P9 Exhibition               ▓▓▓▓
```

**The critical path is P3.** Everything in P4–P6 depends on software that P3
produces. If P3 is rushed, every later phase inherits the debt. If P3 is skipped
entirely — as teams sometimes do — the project becomes a robot remote control
with a web interface, which is not what the brief asks for.

**Phases 8 and 9 must not be compressed.** A project that demos well but publishes
no measurements has demonstrated a robot, not done research. A project that
measures honestly has done both.

---

## 12. What to do if things go wrong

| Situation | Response |
|---|---|
| Navigation proves unreliable in P4 | **Pivot to Concept 5** (stationary classroom assistant). It needs no locomotion. Better a good demo than a broken one. |
| Battery life insufficient | Shorten the route. Re-measure. Reduce to one destination. |
| Fiducials unreliable under venue lighting | Add lighting; fall back to a coded target or a fixed arrival pose. |
| LLM unavailable on the day | **Kill switch — the manual dashboard works.** This is why the fallback exists. |
| Robot damaged | Stop. Do not attempt field repair. Log it. The project continues in simulation or on the other robot. |
| School owns a non-EDU unit | Stop and escalate (Phase 1). There is no workaround. |
| Deadline compresses | Cut scope, not safety. Concepts 5 and 7 can be delivered in far less time than Concept 1. **Never cut the safety work.** |

---

## 13. The five actions, restated

1. **Physically identify both robots.** Model, serial, firmware, compute, hands,
   battery. Checklist in `hardware_configuration_matrix.md` §6.
2. **Get written confirmation from Unitree on Go2 X**, if that is what the school
   owns. And ask the IT team about the G1's network posture.
3. **Establish a read-only DDS connection** and measure the unknowns — camera
   frame rate above all. No actuation.
4. **Run the whole stack in `unitree_mujoco`.** Zero risk, zero cost, and it
   validates every line of code before the robot is switched on.
5. **Write the one-page demonstration protocol** — route, exclusion zone, obstacle
   injection point, measurement definitions, abort criteria — and confirm the
   space and supervision arrangement with the school.
