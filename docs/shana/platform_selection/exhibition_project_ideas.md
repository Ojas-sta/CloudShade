# Exhibition Project Concepts

Nine distinct concepts. **Not variations of one delivery robot.** Three need
physical manipulation, six do not.

Assumption: **Go2 EDU** unless stated otherwise. Where the G1 is the better
platform for a concept, that is stated and the score reflects it.

Difficulty: ★ trivial · ★★ moderate · ★★★ hard · ★★★★ research-grade
Effort: student-weeks, single competent student + supervision. **These are
estimates, not measurements.**

---

## Ranking at a glance

| # | Concept | Robot | Manipulation? | Difficulty | Effort | Rank |
|---|---|---|---|---|---|---|
| 1 | **Autonomous School Courier** | Go2 EDU | No | ★★★ | 14–20 | **1** |
| 2 | **Mobile Lab Sensor Platform** | Go2 EDU | No | ★★ | 8–12 | **2** |
| 3 | **Natural-Language Task Scheduling** | Go2 EDU | No | ★★ | 8–12 | **3** |
| 4 | **Lab Equipment Courier** | Go2 EDU | No | ★★★ | 14–20 | 4 |
| 5 | **Interactive Classroom Assistant** | Go2 EDU | No | ★★ | 8–12 | 5 |
| 6 | **Computer-Vision Lab Demonstration** | Go2 EDU | No | ★★ | 6–10 | 6 |
| 7 | **Digital Twin & Fleet Dashboard** | Go2 EDU | No | ★★ | 6–10 | 7 |
| 8 | **Humanoid Grasp Demonstration** | G1 EDU | **Yes** | ★★★★ | 30–50 | 8 |
| 9 | **Multi-Robot Handoff (G1 + Go2)** | Both | Yes (G1) | ★★★★★ | 50+ | 9 |

Concepts 8 and 9 exist to be honestly scored, not recommended. See §Concept 8 for
why the G1 concept is ranked below concepts that are strictly easier.

---

# Concept 1 — Autonomous School Courier ★ Rank 1

## Project name
**"Room-to-Room: Autonomous Document Courier for School Operations"**

## Problem
Staff and students walk between classrooms, offices, and the library to collect
and deliver notebooks, worksheets, stationery, and small sealed containers.
This is low-value repetitive walking that consumes teaching time and generates
congestion in corridors at changeover.

## Robot
**Unitree Go2 EDU.** Chosen because it is the only platform of the two with a
vendor-supported autonomous navigation API (`slam_operate`), a programmable
obstacle-avoidance interface (`ObstaclesAvoidClient`), and a programmable LiDAR
point cloud already on DDS.

## Why this robot is appropriate
- Carries ~3 kg centred on the spine without altering gait measurably
- Steps over 16 cm thresholds and handles 40° slopes — corridors have both
- Self-recovers from trips; a fall is a minor incident, not a catastrophic one
- CPU-only simulation (`unitree_mujoco`) lets the entire stack be built and
  debugged before the robot is switched on
- 15 kg total mass means a fall or collision is not a hazard to people

## Exact user interaction
1. A staff member opens the dashboard (laptop or tablet) and picks an origin and
   destination from **dropdown lists** — never free text for the location.
2. They select an item from an **allow-list of approved containers** with a
   displayed weight.
3. They press **Submit**.
4. They get an immediate validation result: *"Accepted — task 0042"* or
   *"Rejected: destination 'Lab 2' is not approved for autonomous delivery."*
5. They watch live status, ETA, and battery.
6. On arrival the recipient gets a dashboard notification and a spoken
   announcement; they confirm receipt with one button press.

## End-to-end task workflow
```
Submit → Validate (deterministic) → Readiness check → Load check
  → Navigate to origin → Wait for "loaded" confirmation → Navigate to destination
  → Fiducial arrival confirmation → Unload prompt → Recipient confirmation
  → Notify → Log → Return to dock
```
Every arrow is a state transition in a deterministic state machine. No LLM
participates in any transition.

## Required hardware
- Go2 EDU
- Custom carrier tray: 3 mm 6061 aluminium on 6 mm standoffs, 4× M5 to the spine
  hardpoints, with a lip and a sprung retaining strap
- 1× AprilTag family marker per destination (printable; ~free)
- 1× fiducial camera solution — either the built-in 720p camera, or a small USB
  camera if the EDU depth camera proves inadequate
- Optionally a cheap contact switch or load cell in the tray for load confirmation

## Required software
- Python (task manager, dashboard backend, validators) — team strength
- `unitree_sdk2` / `unitree_sdk2_python` for robot interface (BSD-3)
- ROS 2 Humble + `unitree_ros2` for LiDAR subscription and state
- Nav2 + `slam_toolbox` + `AMCL` (Route B) **or** `slam_operate` (Route A)
- AprilTag detection (AprilTag family via OpenCV/ROS 2)
- SQLite for task history and metrics
- FastAPI + WebSocket for the dashboard

## Required external equipment
- Ubuntu 20.04/22.04 host laptop, **wired Ethernet**
- Network drop or a dedicated router with a static `192.168.123.222/24` host
- Charging station (Go2 EDU supports the charging pile) or a spare 15,000 mAh pack
- Cones or barrier tape for the exclusion perimeter

## Existing capabilities reused
- ✅ `SportClient.Move(vx, vy, vyaw)` locomotion
- ✅ `ObstaclesAvoidClient` reactive avoidance (built-in, **on by default**)
- ✅ `rt/utlidar/cloud_deskewed` point cloud on DDS
- ✅ `rt/sportmodestate` battery / state / foot-force telemetry
- ✅ Built-in obstacle detection, no work required
- ✅ Simulation parity via `unitree_mujoco`

## Features requiring custom development
- 🔨 Task manager, validator, and queue (all of it)
- 🔨 Dashboard and notification routing (all of it)
- 🔨 Carrier tray design and fabrication (all of it)
- 🔨 Arrival fiducial detection and arrival logic
- 🔨 Obstacle-injection detection logic and latency measurement
- 🔨 Full metrics, logging, and diagnostics

## Difficulty
★★★ — the difficulty is entirely in the software we write, not in the robot
interface. That is the point.

## Main engineering risks
1. **Navigation reliability in a real corridor with people.** Highest risk.
2. LiDAR point-cloud quality under fluorescent lighting and low-texture walls
   (corridors are hard for LiDAR SLAM — long parallel surfaces)
3. Fiducial not visible at the approach angle
4. Battery exhaustion mid-route with 15,000 mAh on EDU (~2 h realistic)
5. Wi-Fi dropouts between robot and dashboard

## Safety measures
- Speed capped at **0.4–0.6 m/s** for demonstration — never above walking pace
- Physical exclusion perimeter of 1.5 m either side; nobody in the corridor
  during a run
- Named spotter whose only task is the robot, with a working e-stop
- Payload limited to ≤2 kg in a latched tray; nothing glass, liquid, sharp, hot,
  toxic, fragile, or unsealed
- Tray lip and strap; load must not be able to leave the tray
- Pre-mapped route; no exploration during the exhibition
- Battery check before each run; abort below 40%
- Manual RC always in the spotter's hand, joystick authority tested before each run

## Estimated implementation effort
**14–20 student-weeks.** Roughly: 4 weeks simulation-only bring-up · 4 weeks
navigation · 3 weeks carrier and fiducials · 3 weeks dashboard and task manager ·
2–3 weeks safety, metrics, and rehearsal.

## What can be demonstrated reliably
- Task accepted → robot departs → navigates → arrives → notifies → returns
- Obstacle injected at a known point → robot stops or reroutes → **measured
  response latency displayed on screen**
- A deliberate simulated failure (destination unreachable, battery too low, robot
  not ready) → task rejected before departure, with a clear explanation
- Same task run 10 times in a row with a logged completion rate

## What makes it scientifically interesting
It is a **repeatability study**, not a demo. The team publishes task completion
rate, average and p95 task duration, navigation error at arrival, obstacle-response
latency, and human-intervention count — collected under a written protocol, with
stated confidence intervals and a documented failure taxonomy. It also enables an
honest comparison against published indoor-quadruped benchmarks, which is a real
contribution at school level.

## How performance is measured
| Metric | Method |
|---|---|
| Task completion rate | Completed / submitted over ≥20 trials |
| Average task duration | Wall-clock, submit → recipient confirmation |
| Navigation error | Euclidean distance between fiducial pose and robot pose at arrival |
| Obstacle-response latency | High-speed video timestamp → stop-command timestamp |
| Human interventions | Count of operator actions per run |
| Delivery success rate | Correct destination, correct item, undamaged |
| Perception accuracy | Fiducial detection rate per approach angle |

**Do not invent expected values.** Run a baseline week, publish what you measure,
and compare to it.

## Why compelling at a school exhibition
It solves a problem the judges recognise from their own morning. It is visibly
autonomous — nobody is pushing a joystick. It has a dashboard that looks like
real operations software. And it produces numbers on a screen, which is what
separates a project from a video.

---

# Concept 2 — Mobile Laboratory Sensor Platform ★ Rank 2

## Project name
**"LabSentry: Autonomous Environmental Monitoring for School Laboratories"**

## Problem
Laboratories need periodic environmental readings — temperature, humidity, and
light — at multiple locations. Currently a student or technician walks a clipboard
route. Readings are sparse, inconsistent, and unrecorded.

## Robot
**Unitree Go2 EDU.** The sensor payload is light, needs no manipulation, and the
benefit is entirely in reliable autonomous repositioning between sample points.

## Why this robot is appropriate
- A sensor payload of ~1–2 kg sits comfortably inside the usable envelope
- Low, stable CoM — a mast-mounted sensor is stable on a quadruped in a way it
  would not be on a humanoid
- Autonomous navigation is the entire value; the Go2 is the platform that has it
- No manipulation requirement, so none of the G1's problems apply

## Exact user interaction
1. Operator opens the dashboard and selects a monitoring route from a saved list.
2. Presses **Start Monitoring**. The robot visits each of 4–6 sample points,
   dwells 30 s at each, and streams readings.
3. A live chart shows temperature and humidity against time, annotated with the
   robot's location.
4. At the end, a CSV report is generated and stored with the route name, date,
   and battery state at start.
5. If a reading exceeds a threshold the dashboard raises an alert.

## End-to-end task workflow
```
Select route → Validate points → Readiness check → Navigate to point 1
  → Dwell 30 s, sample at 1 Hz → Navigate point 2 → … → point N
  → Return to dock → Generate report → Store
```

## Required hardware
- Go2 EDU
- Custom sensor mast on the spine: aluminium upright + mounting plate
- Sensor: **environmental only.** For example a combined
  temperature/humidity/light sensor module (I²C or analog over USB)
- Optional: a particulate (PM2.5) sensor — note these need a pump or a fan and
  draw meaningful power
- 1× AprilTag per sample point
- Weatherproof enclosure — **IP rating matters**, and the Go2 is not weatherproof

## Required software
- Everything from Concept 1, minus the tray and load logic
- Sensor driver reading at 1 Hz
- Time-series store: SQLite is sufficient at this data rate
- Charting: any web charting library
- Threshold alerting

## Required external equipment
- Host machine, wired network, charging
- Printed fiducials
- Battery budget: sensor + navigation together

## Existing capabilities reused
- All Concept 1 navigation components
- `rt/sportmodestate` for accurate timestamped position at each sample

## Features requiring custom development
- 🔨 Sensor mast and enclosure design
- 🔨 Sensor driver + calibration + calibration schedule
- 🔨 Dwell-and-sample state machine
- 🔨 Time-series logging and charting
- 🔨 **Calibration methodology** — the scientifically interesting part

## Difficulty
★★ — reuses Concept 1's navigation. The new work is instrumentation.

## Main engineering risks
1. Sensor drift — the dominant scientific risk. Mitigated by a documented
   co-location protocol against a reference instrument.
2. Sensor power draw reducing available runtime
3. Airflow from the robot's own fans and motors biasing temperature readings
4. Moisture ingress

## Safety measures
- All Concept 1 measures
- **No hazardous materials.** No chemicals, no heated apparatus, no glassware on
  the robot
- Sensors are non-invasive: they measure air, they do not touch anything
- Mast height kept low enough that it cannot strike a doorway lintel — measure
  before deployment

## Estimated implementation effort
**8–12 student-weeks** including the navigation reuse.

## What can be demonstrated reliably
- A live chart filling in as the robot moves between points — visually striking
  and clearly scientific
- A calibration comparison against a reference thermometer
- An alert firing when a threshold is deliberately crossed (e.g. by placing a
  warm object near the sensor)
- A generated CSV report

## What makes it scientifically interesting
This is the strongest **genuine science** in the whole set. The interesting
questions are not "can the robot move" but: *how much does sensor drift dominate
measurement uncertainty? How does the robot's own heat sources bias readings?
What is the confidence interval on a spatial measurement made by a moving
platform?* A student team that measures and reports the drift honestly has done
real experimental method, and most exhibition projects cannot.

## How performance is measured
- Sensor drift vs reference, over ≥48 h
- Bias introduced by the robot's own heat sources, with and without motors running
- Repeatability of a spatial measurement (same point, N times)
- Measurement uncertainty propagation
- Navigation repeatability (does the robot sample the same point each run?)

## Why compelling at a school exhibition
A live chart drawing itself as a robot moves through a room is visually
immediate. And it connects to something the school genuinely cares about — its
own laboratories — rather than to a toy task.

---

# Concept 3 — Natural-Language Task Scheduling and Execution ★ Rank 3

## Project name
**"Ask Robo: Natural-Language Request Ingestion with Deterministic Execution"**

## Problem
The dashboard of Concept 1 requires the user to fill in forms. Staff want to
type "take the physics notebooks to the lab before period 3" and have the system
work out what that means — but acting on unparsed natural language is dangerous.

## Robot
**Unitree Go2 EDU** (the concept is robot-agnostic in principle but requires
autonomous execution to demonstrate its point).

## Why this robot is appropriate
The concept is about the **software architecture**, not the hardware. It works on
either platform. Go2 is chosen because autonomous execution must actually work
for the demonstration to prove anything.

## Exact user interaction
1. User types free text into the dashboard: *"Take the physics notebooks to the
   physics lab before period 3."*
2. The system returns a **structured interpretation for confirmation**:
   ```
   Parsed:
     action      : DELIVER
     item        : physics notebooks        (from approved item list)
     destination : Physics Lab — Room 114
     deadline    : 11:40 (period 3 start)
   Confidence: 0.94
   [ Confirm ]  [ Edit ]  [ Cancel ]
   ```
3. The user **confirms or edits**. Nothing executes without confirmation.
4. Execution proceeds exactly as in Concept 1.

## End-to-end task workflow
```
Free text → LLM structured extraction → Schema validation → Entity resolution
  against a registry → DETERMINISTIC validator (permissions, destination
  allow-list, payload allow-list, schedule feasibility)
  → Human confirmation → Task queue → Execution (as Concept 1)
```

## The architecture boundary — the whole point of this concept
```
┌─────────────────────────────────────────────────────────────┐
│  LLM LAYER (untrusted input, advisory only)                │
│  Free text → {action, item, destination, deadline}          │
│  CANNOT: emit commands, set velocities, name topics,        │
│          bypass the validator, or execute anything          │
└──────────────────────┬──────────────────────────────────────┘
                       │ structured JSON only
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  DETERMINISTIC VALIDATOR (trusted, complete)               │
│  - JSON schema check                                         │
│  - Destination must exist in approved registry              │
│  - Item must be in approved item list with known weight     │
│  - Destination-item combination must be permitted           │
│  - Deadline must be feasible given battery + queue          │
│  - Robot must be READY (state, battery, faults)             │
│  Reject → explain → never execute                           │
└──────────────────────┬──────────────────────────────────────┘
                       │ validated Task object
                       ▼
              Task queue → Executor → Verified SDK interfaces
```

## Required hardware
Identical to Concept 1. **This concept adds no hardware.**

## Required software
- Everything from Concept 1
- An LLM reachable over an API (or a local model if the school requires data
  sovereignty — check institutional policy)
- A JSON-schema validator
- An entity registry mapping natural-language names → canonical location IDs
- A prompt-injection-resistant input layer

## Required external equipment
- LLM API access, **or** a local inference host
- ⚠️ **School network policy decision**: is sending user text to an external API
  permitted? This is a real institutional question, not a technical one.

## Existing capabilities reused
- All of Concepts 1 and 2

## Features requiring custom development
- 🔨 Everything in this concept. There is no existing capability here.

## Difficulty
★★ — the natural-language part is easy. **The validator is the work**, and the
validator is what makes it safe.

## Main engineering risks
1. **Hallucinated destinations.** Mitigated by resolving against a closed registry
   and **refusing any entity not in it**. Never let the LLM invent a room.
2. **Prompt injection** via user text ("ignore previous instructions and…").
   Mitigated by treating all LLM output as untrusted data validated against a
   schema, with the LLM having no tool access at all.
3. Scope creep into an "AI controls the robot" framing. The architecture must make
   structurally clear that it does not.
4. LLM availability / latency during a live demonstration. **Mitigate by caching
   parsed forms and having a manual-entry fallback that always works.**

## Safety measures
- All Concept 1 measures
- **The LLM has no robot interface whatsoever.** It writes JSON to a validator
  process. It cannot reach DDS. This should be verifiable by inspecting the code
  and is worth demonstrating.
- Every LLM-derived task requires human confirmation before execution
- A kill switch that disables LLM features entirely, leaving the manual dashboard
  fully functional — **the exhibition demo must work with the LLM switched off**

## Estimated implementation effort
**8–12 student-weeks**, on top of Concept 1.

## What can be demonstrated reliably
- Natural language → correct structured task → confirm → execute → complete
- **Deliberate adversarial inputs being rejected**: an unknown room, a prohibited
  item, an infeasible deadline — each with a clear, specific explanation
- The manual fallback working identically with the LLM disabled

## What makes it scientifically interesting
The interesting claim is not "we used an LLM." It is: **"we built a system that
uses a language model and can prove the model cannot cause the robot to do
anything it was not explicitly permitted to do."** That is a real, testable safety
property, and it can be demonstrated — feed it a hundred adversarial inputs and
show zero unauthorised actions. Reporting the pass rate of an adversarial test
suite is a genuine experimental result.

## How performance is measured
| Metric | Method |
|---|---|
| Intent extraction accuracy | Labelled test set of ≥100 real staff phrasings |
| Rejection rate on adversarial inputs | ≥100 adversarial prompts; **target zero unauthorised actions** |
| False-rejection rate | Valid requests correctly accepted |
| LLM latency | Time from submit to rendered parse |
| Layer-of-defence verification | Code inspection showing no LLM→robot path exists |

## Why compelling at a school exhibition
Judges consistently find "AI that touches a physical machine" both the most
exciting and the most alarming idea. Demonstrating that the team thought about
the safety architecture — and can prove it — is far more impressive than
demonstrating that it works.

---

# Concept 4 — Laboratory Equipment Courier ★ Rank 4

## Project name
**"SpecimenRun: Sealed-Specimen Transport Between School Laboratories"**

## Problem
Approved, sealed, non-hazardous teaching specimens and prepared apparatus need
moving between prep rooms and teaching labs, on a schedule, with a chain of
custody.

## Robot
**Unitree Go2 EDU.**

## Why this robot is appropriate
Same as Concept 1, with one addition: **the G1's manipulation would be required if
the container had to be picked up off a bench and handed over.** Here the
container is pre-placed in the tray by a human, so no manipulation is needed —
which is precisely why this concept is realistic on the Go2 and would be
unreliable on the G1.

## Exact user interaction
Identical to Concept 1, plus a **chain-of-custody record**: every handoff is
logged with a timestamp, a camera snapshot, and a recipient confirmation.

## End-to-end workflow
As Concept 1, plus custody events appended to the task record.

## Required hardware / software / external equipment
Concept 1, plus: sealed containers in a ratcheting tray cradle; a cheap webcam
pointed at the tray for custody snapshots (optional but valuable).

## Existing capabilities reused
All of Concept 1.

## Features requiring custom development
- 🔨 Tray cradle with secondary retention
- 🔨 Custody log with tamper-evidence (hash-chained records)

## Difficulty
★★★ — same as Concept 1 plus a modest custody-logging addition.

## Main engineering risks
All Concept 1 risks, plus: **container movement during transit**. This is
mitigated by a cradle, not by robot capability, and must be measured.

## Safety measures
All Concept 1 measures, plus:
- **Only sealed, inert, non-hazardous materials.** No chemicals, no reagents, no
  glassware, no sharps, no heated apparatus, nothing toxic.
- Container contents declared in advance and checked against an allow-list
- Secondary retention strap independent of the primary cradle
- Containers must not be glass. If any laboratory demonstration uses glass, the
  robot does not carry it.

## Estimated effort
**14–20 student-weeks.**

## What can be demonstrated reliably
- Timed scheduled run between two labs
- Obstacle handling mid-route
- Complete custody record exported at the end
- **Measured container displacement** — this is the interesting number

## What makes it scientifically interesting
The measurable claim is: *how much does a container actually move during
quadruped locomotion, and does that exceed the tolerance of the contents?* That
is a small, honest, answerable experiment that connects robot dynamics to
laboratory practice. Publish displacement vs route, vs speed, vs payload mass.

## How performance is measured
- Container displacement (mm) per transit, via accelerometer or video tracking
- Spillage/damage rate over N transits
- Custody-record completeness
- Delivery success rate
- Energy per transit

## Why compelling at a school exhibition
"Labs get their specimens delivered reliably and we can prove the contents were
not disturbed" is a concrete, defensible claim a school actually wants.

---

# Concept 5 — Interactive Classroom Assistant ★ Rank 5

## Project name
**"Ask the Robot: Supervised Classroom Q&A and Announcement Delivery"**

## Problem
Teachers repeat the same procedural announcements and standard questions
("where is the fire assembly point", "what is today's homework") in every lesson.

## Robot
**Unitree Go2 EDU.**

## Why this robot is appropriate
Presentation and speech only. **No locomotion during Q&A** — the robot stands in a
fixed spot for the session. This deliberately avoids the mobility reliability
problem entirely and concentrates on the parts that work well.

⚠️ **Note the trade-off this concept exposes:** the G1 has a genuinely better
audio story — offline ASR *and* TTS on DDS via the `voice` service. The Go2's
`vui` service exposes only volume and screen brightness. **On the Go2, speech must
come from an external host** (a USB speaker or a room microphone plus a host-side
ASR/TTS pipeline). That is a real, and occasionally Go2-winning, argument for the
G1 — but it does not outweigh the navigation and reliability gaps.

## Exact user interaction
1. The robot is positioned at the front of the room, stationary, on a marked spot.
2. The teacher asks a question verbally.
3. The system transcribes, retrieves from an **approved school-material corpus
   only**, and answers aloud.
4. Anything outside the corpus produces: *"I don't have that in my approved
   materials — please ask your teacher."* **Never improvise.**
5. The teacher can push a scheduled announcement to the robot.

## End-to-end workflow
```
Microphone → ASR → RAG retrieval over approved corpus (local index)
  → TTS → speaker → display of the answer text
Rejection path if retrieval score below threshold
```

## Required hardware
- Go2 EDU on a marked floor spot
- USB microphone (or the robot's built-in mic — the Go2's audio interface is
  limited to `vui` volume/brightness, so a **host-side mic is likely required**)
- External speaker, or a room PA
- A tablet or small display for the answer text (optional)

## Required software
- Concept 1's task manager and dashboard, reused
- Host-side ASR (Whisper or equivalent — verify licence) and TTS
- A local vector or keyword index over approved school material
- **A strict relevance threshold** — below it, refuse

## Required external equipment
- Host machine
- Microphone and speaker

## Existing capabilities reused
- Concept 1's dashboard, logging, network

## Features requiring custom development
- 🔨 Everything in the audio path
- 🔨 The corpus and its curation process
- 🔨 The refusal logic

## Difficulty
★★

## Main engineering risks
1. **Hallucination in a Q&A system shown to students.** This is the defining risk.
   Mitigation: approved corpus only, retrieval threshold, explicit refusal, and the
   answer text displayed so a teacher can see what was retrieved.
2. ASR accuracy in a noisy classroom
3. Latency
4. Students asking inappropriate questions — a content-policy question the school
   must own, not the team

## Safety measures
- **Stationary.** No locomotion during student-facing sessions. This removes the
  entire fall/collision category.
- Physically anchored or on a marked spot with the spotter present
- No student may approach unsupervised
- Battery above 60% before a session

## Estimated effort
**8–12 student-weeks.**

## What can be demonstrated reliably
- Ask a prepared question → correct answer from the corpus
- Ask an out-of-scope question → clean, correct refusal
- Teacher-pushed announcement delivered audibly and displayed
- A visible latency measurement

## What makes it scientifically interesting
The interesting result is the **refusal behaviour**: an honest measurement of how
often the retrieval system declines to answer, and an argument for why refusing is
the correct engineering choice in an educational setting. A system that says "I
don't know" well is more valuable than one that always answers.

## How performance is measured
- Answer accuracy on a labelled question set
- **Correct-refusal rate** on out-of-scope questions
- False-answer rate (should be zero — this is the safety metric)
- ASR word error rate in classroom noise
- End-to-end latency

## Why compelling at a school exhibition
Every judge has seen an AI chatbot hallucinate. A team that demonstrates an
educationally-deployed AI which *knows when to shut up* is making a point that
lands.

---

# Concept 6 — Computer-Vision Laboratory Demonstration ★ Rank 6

## Project name
**"Vision Lab: Live Computer-Vision Analysis Published to the Class"**

## Problem
Computer-vision techniques are taught using static images on slides. Students do
not see a vision pipeline running on a moving camera in a real environment,
dealing with lighting change, motion blur, and occlusion.

## Robot
**Unitree Go2 EDU.** The value is the moving, embedded camera — not the robot.

## Why this robot is appropriate
Its built-in camera and autonomous motion make a self-running vision pipeline
possible without a person holding anything. **This concept requires no
manipulation, no navigation reliability beyond a short route, and no payload.**

## Exact user interaction
1. Operator selects an analysis mode (e.g. object detection, fiducial pose,
   colour/light metering).
2. Presses **Start**.
3. The robot follows a short, pre-mapped loop (or stands and pans if the hardware
   permits — the Go2 cannot pan; it must translate).
4. The dashboard shows a **live annotated video feed with detections overlaid**
   and a running log of what was detected, where, and when.
5. Results export as CSV and as annotated snapshots.

## End-to-end workflow
```
Camera stream (GStreamer H.264) → decode → preprocess → model inference
  → NMS → overlay → live display → event log → CSV
```

## Required hardware
- Go2 EDU (EDU depth camera is a bonus; the 720p wide-angle is sufficient)
- A printed scene: coloured objects, fiducials, a scale ruler
- Optional: a small monitor so judges can see the feed without the laptop screen

## Required software
- GStreamer pipeline (verified in `sdk_and_software_research.md` §4.3)
- OpenCV for decode, preprocessing, overlay
- A detection model — **verify the licence of whichever you choose**
- SQLite for the event log
- The dashboard from Concept 1

## Required external equipment
- Host machine (a real GPU is strongly preferred; the Orin NX 100 TOPS option
  enables onboard inference — this is a genuine reason to consider the expansion
  dock)
- A monitor

## Existing capabilities reused
- Concept 1's navigation, dashboard, logging

## Features requiring custom development
- 🔨 The vision pipeline
- 🔨 Model selection, training or fine-tuning, and **evaluation methodology**
- 🔨 Performance measurement under motion

## Difficulty
★★ — the hardest part is the vision, and standard recipes exist.

## Main engineering risks
1. **Frame rate.** ⚠️ Unitree's own docs contradict each other: `about_Go2` says
   15 fps; the FAQ says 1080p@30 / 720p@60. **Bench-test this in Phase 2 before
   committing to any design.** At 15 fps, real-time detection is limited.
2. Motion blur — the single biggest CV problem with a moving camera
3. Lighting variation across a corridor
4. Model licence compatibility
5. GPU availability

## Safety measures
- Short, pre-mapped loop; speed capped at 0.3 m/s for a cleaner feed
- Same perimeter and spotter requirements
- **No hazardous objects in the scene**

## Estimated effort
**6–10 student-weeks** including navigation reuse.

## What can be demonstrated reliably
- A live annotated feed — visually excellent and instantly legible
- Detection accuracy measured over a fixed evaluation set
- A deliberate stress test: reduce lighting, move faster, occlude objects — and
  publish how performance degrades. **This is the scientifically valuable part.**

## What makes it scientifically interesting
The genuinely interesting result is the **degradation curve**: detection accuracy
as a function of platform speed, under varying illumination, with measured
occlusion handling. That is an experiment with a real hypothesis, a real
independent variable, and a real finding. Most student vision projects only
report a static accuracy number.

## How performance is measured
- Detection precision/recall/mAP on a fixed labelled set
- **Accuracy vs platform speed** (the key independent variable)
- Accuracy vs illumination level
- Frames per second actually achieved (measure, do not assume)
- End-to-end latency

## Why compelling at a school exhibition
A live annotated video feed with detections appearing as the robot moves is
immediately impressive to a non-technical judge, and the degradation experiment
gives it substance.

---

# Concept 7 — Digital Twin and Fleet Dashboard ★ Rank 7

## Project name
**"TwinTrack: Live Robot State, Telemetry, and Predictive Maintenance"**

## Problem
Educational robotics projects usually have no observability. When the robot
misbehaves, nobody knows why. Battery health, motor temperatures, and fault codes
are invisible.

## Robot
**Unitree Go2 EDU.** The rich telemetry is the point.

## Why this robot is appropriate
`rt/sportmodestate` and `rt/lowstate` expose IMU, motor states, battery
management system data, and foot forces on the Go2. That is a rich instrumentation
surface, and it is more accessible than anything the G1 publishes.

## Exact user interaction
1. A dashboard shows live battery voltage, current, state of charge, per-motor
   temperature, IMU attitude, foot contact forces, and fault flags.
2. A rolling chart shows motor temperature and current over time.
3. A **predictive-maintenance panel** flags degradation trends.
4. A **mission log** shows every task with outcome, duration, battery at start and
   end, and any operator intervention.
5. Everything exports as CSV for analysis.

## End-to-end workflow
```
Subscribe rt/sportmodestate + rt/lowstate (high rate)
  → ring buffer → time-series store → live charts → trend analysis → alerts
```

## Required hardware
Concept 1's hardware. **No additional sensors needed** — this concept extracts
maximum value from what is already there.

## Required software
- High-rate DDS/ROS 2 subscriber
- Ring-buffer + time-series writer (SQLite adequate at these rates, or a
  round-robin binary log for higher rates)
- Live charting
- Trend analysis

## Existing capabilities reused
- All of `unitree_ros2`'s state readers — this concept is almost entirely
  reuse-plus-dashboards

## Features requiring custom development
- 🔨 Telemetry pipeline, storage, and dashboards
- 🔨 Trend analysis and the predictive-maintenance heuristic

## Difficulty
★★

## Main engineering risks
1. **Storage volume** at 500 Hz — must downsample intelligently and define
   retention
2. Ring-buffer management
3. Clock synchronisation between robot and host — timestamps from the robot and
   host timestamps must be reconciled or all timing analysis is wrong
4. Assuming a fault code means what we think it means

## Safety measures
No additional physical risk — the robot can be stationary or on a bench. Its real
value is that it makes the *other* concepts more reliable to operate.

## Estimated effort
**6–10 student-weeks.**

## What can be demonstrated reliably
- Live telemetry from a moving robot on a big screen
- A detected anomaly — deliberately induced — flagged correctly
- A complete mission log exported and analysed
- **A before/after comparison showing that telemetry-based monitoring measurably
  reduced incident rate**

## What makes it scientifically interesting
This is the layer that turns a demo into engineering. The publishable result is
the relationship between observable telemetry and actual failure — e.g. "motor
temperature exceeded X for Y seconds before 3 of 4 failures." And it is a genuine
contribution to the team's own reliability, which is measurable.

## How performance is measured
- Telemetry coverage — % of mission time with valid data
- Anomaly detection precision/recall against induced faults
- Clock-sync accuracy
- Reduction in incident rate with monitoring on vs off
- Data loss rate

## Why compelling at a school exhibition
Engineering judges respond strongly to observability. And the honest finding
"we built telemetry and then discovered that 80% of our failures had a visible
precondition" is exactly the kind of result that demonstrates engineering maturity.

---

# Concept 8 — Humanoid Grasp Demonstration (G1) ★ Rank 8

## Project name
**"HandControl: Dexterous Manipulation of Lightweight Objects"**

⚠️ **This is included for completeness and honesty, not as a recommendation.**

## Problem
Demonstrating that a humanoid can manipulate objects is a genuine research
question, and the school may have an interest in it.

## Robot
**Unitree G1 EDU** — the only platform of the two with arms.

## Why this robot is appropriate
It is the only one that *can* do this. No Go2 variant has an arm.

## Exact user interaction
Operator places a target object in the operating envelope, selects a scripted
trajectory, and the robot attempts the pick.

## End-to-end workflow
```
Object placed → detect → reach → grasp → lift → transport → place → release
```
Executed via **teleoperation** (`avp_teleoperate`, VERIFIED official, Quest 3 /
Apple Vision Pro supported, G1 23 and 29 DOF confirmed), or by playing back a
recorded trajectory.

## Required hardware
- G1 **EDU** (standard has no hands and no secondary development)
- **Protective frame + protective rope — mandatory per Unitree**
- Dex3-1 hands (tactile variant strongly preferred)
- Quest 3 or Apple Vision Pro for teleoperation
- Lightweight objects: 100–500 g, non-fragile, non-sharp

## Required software
- `unitree_sdk2` raw DDS on `rt/arm_sdk` + `rt/dex3/*/cmd`, with CRC
- `avp_teleoperate` (official, NOASSERTION licence — **verify before use**)
- Optionally `unitree_lerobot` for recording and replaying episodes

## Existing capabilities reused
- `G1ArmActionClient` — 16 pre-recorded animations (wave, clap, high five). These
  are the **only** manipulation that is reliable out of the box.
- `avp_teleoperate` for teleoperation

## Features requiring custom development
- 🔨 **Everything else.** Joint-level arm control, grasp strategy, object
  detection, grasp verification, retry logic, recovery.

## Difficulty
★★★★ — research-grade. Requires months of data collection and policy work to
reach what a commercial system does routinely.

## Main engineering risks — all severe

| Risk | Evidence |
|---|---|
| **Grasp failure** | Best published full-task pick-and-place on real G1: **20–30%** (arXiv 2610.08119). Native foundation-model action: **~7%**. |
| **Force limit** | **13.5 N** sustained unbraced (≈1.38 kgf). Measured, arXiv 2609.25486. |
| **Unmeasured hand** | Both strong published results used **BrainCo** hands. **No published grasp rate exists for the Dex3-1.** |
| **No flat-sheet data** | Unitree's ~180 G1 datasets contain **no** notebook/book/thin-sheet grasping task. Negative signal. |
| **Thermal → falls** | NVIDIA tracker #121, open since Apr 2026: G1 + Dex3 overheats "even when it is just standing idle", auto-shutdown, **falls repeatedly**. |
| **Cannot self-recover** | Munro teardown lab: base G1 "proved unable to stand back up" after falling. |
| **Mechanical maturity** | Non-strain-wave joints with backlash; exposed greased ankle Cardan joint; "definitely not ready for prime time". |
| **Support burden** | A university lab's unit was declared "unusable" pending hip motors, with vendor support requiring international escalation. |
| **Reach** | 1,320 mm tall; cannot reach standard workbenches. |
| **Hand–locomotion coupling** | Unitree: "not recommended to perform overly intense motions with the dexterous hand installed, such as running or balance tests." |
| **Contradictory SDKs** | C++ and Python disagree on action IDs and JSON keys. Neither is authoritative. |

## Safety measures
- **Protective frame + protective rope — non-negotiable, per Unitree's manual**
- Spotter whose only task is the robot, always
- **Fenced pen.** No public, no students, no unsupervised operation. Ever.
- Speed capped at 0.2 m/s; standing-still manipulation only if at all
- Non-fragile objects only. **No glass, no liquids, no sharps.**
- Emergency damping via RC — ⚠️ verify the combo on hardware; the manual is
  internally inconsistent (`L1+A` vs `L2+B`)
- Never power off while standing unprotected

## Estimated effort
**30–50 student-weeks**, plus months of calibration and failure recovery that are
not compressible.

## What can be demonstrated reliably
Realistically: **teleoperated grasping of a light object at low success rate, and
the pre-recorded wave/clap/high-five animations at high success rate.**

Be honest with judges: demonstrate 10 attempts, report the actual count. A 3-in-10
result honestly presented with a discussion of why is a *better* project than a
3-in-10 result hidden.

## What makes it scientifically interesting
Genuinely, this is the most scientifically interesting concept in the set — it is
a hard, open research problem with a real literature. The honest contribution is
a **reproducible measurement** of G1 manipulation reliability with a stated
protocol, filling a genuine gap: **no fall-rate statistic and no Dex3-1 grasp
rate has ever been published.** Producing either would be a real result.

## How performance is measured
- Grasp success rate over ≥50 attempts per object type, with Wilson confidence
  intervals
- Time to successful grasp
- Failure taxonomy (reach / pre-grasp / grasp / lift / place / transport)
- Recovery rate after a fall
- Sustained session duration before thermal event

## Why compelling at a school exhibition
A humanoid picking up an object is the single most attention-grabbing thing a
robot can do. **But the attention is the point, not the engineering.** This
concept maximises spectacle and minimises reliability. That is the wrong trade for
the stated project vision, which is about *useful, repeatable* assistance.

---

# Concept 9 — Multi-Robot Handoff ★ Rank 9

## Project name
**"Relay: Humanoid-to-Quadruped Object Handoff"**

## Problem
Can a humanoid hand an object to a quadruped, or vice versa?

## Robot
**Both** — which requires the school to release *both*, contradicting the brief's
one-robot constraint.

## Why this robot is appropriate
It is the only concept that genuinely needs both, which is why it is last.

## Exact user interaction
Operator-initiated.

## End-to-end workflow
```
G1 picks object → walks to handover point → places/releases → Go2 receives
  → transports to destination
```
Each leg is a Concept 1 or Concept 8 task, plus a coordinated handover protocol
that neither platform has any support for.

## Required hardware / software
Everything from Concepts 1 and 8, plus a shared task bus, a handover protocol, and
clock synchronisation between two robots.

## Difficulty
★★★★★

## Main engineering risks
Everything in Concepts 1 and 8, multiplied, plus:
- The G1 must **place** an object — a manipulation primitive it has not been shown
  to perform reliably
- The Go2 must **receive** an object — it has no manipulator at all, so it must
  be positioned under the object and the G1 must release into the tray. This
  requires G1 placement precision the literature does not support
- Two robots, two failure modes, one exclusion zone

## Estimated effort
**50+ student-weeks**, with a high probability of not completing.

## What can be demonstrated reliably
Almost certainly nothing end-to-end. Possibly a staged, non-integrated sequence:
G1 manipulates, Go2 transports, cut between them.

## Scientifically interesting?
Yes — coordination between embodiments is an open problem. But answering it
requires a research group, not a school project.

## Why ranked last
It violates the one-robot constraint, has the highest failure probability, the
longest timeline, and depends on the G1's least reliable capability as a load-
bearing step. It is included to show it was considered and rejected on evidence,
not overlooked.

---

# Cross-concept comparison

| Concept | Needs manipulation? | Needs reliable long-range nav? | Hardware cost | Estimated completion risk |
|---|---|---|---|---|
| 1 Autonomous Courier | No | **Yes** | Carrier tray + fiducials | Moderate |
| 2 Lab Sensor Platform | No | **Yes** | Sensor mast + fiducials | **Low** |
| 3 NL Task Scheduling | No | Yes | **None** | **Low** |
| 4 Equipment Courier | No | **Yes** | Cradle + fiducials | Moderate |
| 5 Classroom Assistant | No | **No** — stationary | Mic + speaker | **Low** |
| 6 Vision Lab | No | Short route only | Scene + monitor | **Low** |
| 7 Digital Twin | No | No | **None** | **Low** |
| 8 Humanoid Grasp | **Yes** | No | Frame + rope + VR + hands | **High** |
| 9 Multi-Robot | **Yes** + both robots | Yes | Everything | **Very high** |

## Recommended combination

**Concepts 1 + 2 + 3 + 7 combined** is the strongest project, and it is not
concept stapling — the pieces are genuinely one system:

- **7 (telemetry)** is the instrumentation that makes 1 and 2 measurable and safe
- **1 (courier)** is the core autonomy demonstration
- **2 (sensor platform)** is the same robot doing a second scientifically
  distinct job on the same navigation stack
- **3 (NL scheduling)** is the layer that makes it look and behave like real
  operations software, and it is the natural place to demonstrate the AI safety
  architecture

Concept **5** is the fallback if navigation proves unreliable — it needs no
locomotion at all and can be demonstrated whatever happens.

Concept **6** is the cheapest add-on for visual impact.

**Concept 8 is not recommended.** If the school requires a humanoid, that is a
brief change, not a technical argument, and the manipulation reliability numbers
must be disclosed to judges up front.
