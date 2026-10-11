# SHANA — Cameras and Depth Sensors

**Status: RESEARCH ONLY. No camera has been accessed, tested, or configured.**

---

## What the robot is documented to have

| Sensor | Status | Evidence class |
|---|---|---|
| **HD wide-angle camera** | Present on all Go2 variants. F2.2, 120° FOV | VERIFIED |
| **Depth camera** | **EDU only.** Model **not named in any official source** | VERIFIED presence / NOT FOUND model |
| **4D LiDAR L2** | All variants. 360° × 96°, 21,600 samples/s, 0.05 m min | VERIFIED |

⚠️ **Whether the school robot has the depth camera is unknown.** It is an EDU-only
feature and must be confirmed by inspection (`SH-GO2-004`).

---

## The frame-rate contradiction

⚠️ **Unitree's own documentation disagrees.**

| Source | Claim |
|---|---|
| `about_Go2` | *"720p, 15fps, 1080p, 15fps dual streams… smooth 720p 15fps transmission"* |
| Developer FAQ | *"1080P(HD) 30fps or 720p 60fps"* |

**This must be measured before any perception-dependent design** (`SH-GO2-015`,
`TEC-02`).

⚠️ **Design assumes the worse figure: 15 fps.** What that supports and does not:

| Task | At 15 fps |
|---|---|
| ArUco marker detection for floor identity | ✅ **Likely fine.** Markers are large, static, high-contrast |
| AprilTag checkpoint confirmation | ✅ Likely fine |
| Coarse object presence | ⚠️ Marginal |
| Person tracking | ⛔ Not viable |
| Any fine manipulation perception | ⛔ Not viable |

⚠️ **The floor-identity requirement is deliberately designed around markers rather
than general vision precisely because markers tolerate a low frame rate.**

---

## How camera video is actually obtained

There is no video streaming API. There are two documented paths.

### Path 1 — Snapshot via `VideoClient`

```cpp
int32_t GetImageSample(std::vector<uint8_t>&);
```

Service `videohub`, API 1001. Returns **a single compressed JPEG frame**. The
official example writes timestamped `.jpg` files and sleeps between calls.

⚠️ **This is a snapshot API, not a stream.** Polling it at a usable rate is
questionable.

### Path 2 — GStreamer over UDP

Documented pipeline:

```
udpsrc address=230.1.1.1 port=<n> ! application/x-rtp,media=video,
  encoding-name=H264 ! rtph264depay ! video/x-raw,format=NV12,
  width=1280,height=720,framerate=15/1 ! videoconvert ! appsink drop=1
```

Requires OpenCV compiled with `WITH_GSTREAMER`.

⚠️ **Not yet verified on our hardware.** Latency and stability are unknown.

---

## What the depth sensor can and cannot do

| It can | It cannot |
|---|---|
| Detect nearby obstacles | Determine surface friction |
| Provide a range image | Prove a floor is safe |
| Support grasp or approach planning | See through glass or transparent surfaces |
| Detect stair edges at close range | Reliably detect a thin wire or cable |
| — | Provide absolute altitude |

⚠️ **Depth sensors are defeated by transparent and specular surfaces.** Water
puddles, polished floors, and glass are exactly the hazards of concern.

---

## The hazard-detection limitation

> **RGB and depth cameras cannot determine whether a floor is safe to drive on.**

| Hazard | Camera detection | Is that sufficient? |
|---|---|---|
| Water puddle | Reflection, specular highlight | ⚠️ **No.** Many surfaces reflect without being wet |
| Wet floor sign | Text or shape recognition | Advisory only — prompts human confirmation |
| Cleaning in progress | Equipment, person, wet floor | Advisory only |
| Floor damage | Visual crack, deformation | ⚠️ Partial at best |
| **Surface friction** | **Not detectable by any camera** | ⛔ **Not detectable at all** |
| Cable on floor | Too thin, usually below resolution | ⛔ Not reliably |
| Low friction, no visible water | **Not detectable** | ⛔ Not detectable |

⚠️ **The Go2 manual explicitly forbids operation on wet or low-friction ground
and names foot slip as a failure mode.** The manufacturer's prohibition exists
precisely because this cannot be sensed.

### Therefore

Corridor hazard handling is **built on human declaration, not on sensing.**

```
Corridor state
   │
   ├── HAZARD  ← declared by an authorised person  (authoritative)
   ├── CLOSED  ← declared by an authorised person  (authoritative)
   ├── UNKNOWN ← no data, stale data, or sensor failure  (conservative)
   └── AVAILABLE ← declared, and no hazard reported
```

Sensors may **raise a prompt for human confirmation**. They may **never** open
or close a corridor themselves.

---

## Vision requirements for the proposed system

| Requirement | Status |
|---|---|
| Floor identity via marker | Needs marker detection — likely viable at 15 fps |
| Checkpoint arrival confirmation | Needs marker detection — likely viable |
| Marker-free arrival confirmation | **Not required** — markers are the design |
| Obstacle detection | **Handled by the robot's own avoidance**, not our vision |
| Corridor occupancy | **Deferred.** See `docs/shana/infrastructure/cctv_feasibility.md` |

⚠️ **We deliberately minimise our dependence on our own vision.** Every additional
vision dependency is another failure mode that must be tested, monitored, and
degraded safely.

---

## Questions

| ID | Question |
|---|---|
| `SH-GO2-004` | Is the EDU depth camera fitted? |
| `SH-GO2-015` | What camera frame rate has actually been measured? |
| `SH-GO2-018` | What are the depth camera's documented limits? |
| `SH-CCTV-016` | Does school policy permit any analytics access? |
| `SH-ENV-005` | What hazards can the camera detect, and at what distance? |
| `SH-ENV-006` | What can the depth sensor detect, and what does it miss? |
| `SH-ENV-007` | Confirm camera detection is never proof of floor safety |
| `SH-MAP-017` | Marker size, height and standoff distance |
| `SH-MAP-018` | Lighting conditions at each marker location |
| `SH-PRIV-018` | What may the robot's own camera capture? |

---

## Nothing here is verified on our hardware

⚠️ **Every capability in this document is documented by the manufacturer, not
demonstrated by us.** Nothing has been tested. `SH-GO2-015` and `SH-GO2-004` must
be answered before any perception-dependent design is committed.