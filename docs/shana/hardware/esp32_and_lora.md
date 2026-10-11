# SHANA — ESP32, LoRa and Optional Radio Infrastructure

**Status: RESEARCH ONLY. No radio has been configured, transmitted on, or
installed. This technology is DEFERRED and may never be needed.**

---

## The conclusion first

> **SHANA probably does not need LoRa. Do not build it until a specific gap is
> demonstrated.**

| Reason | Detail |
|---|---|
| **The robot link is Ethernet, not Wi-Fi** | A WLAN failure **does not stop the robot**. Radio telemetry would not improve that. |
| **Wi-Fi beats LoRa for coverage indoors** | Peer-reviewed 16-storey study found LoRa *still lost the link across floors*, with near-zero packet delivery in the far zone. |
| **Floor attenuation varies enormously** | Published studies range 5.5–21.7 dB per floor — a 4× spread. No single range figure is meaningful. |
| **The project is likely to be single-floor** | Which removes the hardest propagation case entirely. |

**LoRa buys independence from the WLAN, not range.** That is a genuine benefit —
but only if a WLAN outage has consequences we have not yet identified.

**Gate question `SH-RAD-001` is a P0 justification question.** If no demonstrated
problem exists, the other 19 questions in this category are moot and the
hardware should not be bought.

---

## When it *would* be justified

Any one of these:

- The robot operates in areas where Wi-Fi does not reach — stairwells, sports
  hall, courtyard, gate (`SH-NET-009`)
- A distinction is needed between *"the robot is fine and the network is down"*
  and *"the robot is lost"* during a demonstration
- An independent channel is required that cannot be knocked over by whatever
  broke the primary link

⚠️ **Battery efficiency is NOT a justification here.** The robot is docked and
mains-charged between missions, so node battery life is irrelevant. A published
comparison found Wi-Fi association costs ~3.9 seconds per message against 564 ms
for a low-power link — irrelevant for a docked device.

---

## Indian regulatory constraints

⚠️ **These are the errors an international source will make.**

| # | Constraint | Consequence |
|---|---|---|
| 1 | Licence-exempt band is **865–868 MHz**, not 868 MHz | ⚠️ **The default EU868 channel, 868.1 MHz, is OUTSIDE the band.** An unmodified EU868 configuration is non-compliant. |
| 2 | India cites **EN 300 220**, not the modern EN 303 708 | A CE/RED test report is built on the wrong standard. |
| 3 | Power limited in **e.r.p., not EIRP** — "relative to a half-wave dipole" | 500 mW e.r.p. = **29.15 dBm EIRP**. Quoting 500 mW as EIRP overshoots by 2.15 dB. |
| 4 | Duty cycle: **2.5% end device, 10% gateway, 1% at 25 mW** | At SF12 a 350-byte report fits only every ~2 minutes. |
| 5 | ⚠️ **A LilyGO T-Beam at full 32 dBm is illegal** | ~1531 mW e.r.p. against a 500 mW ceiling — **4.9 dB over**. Must be capped to ~24–27 dBm with Adaptive Power Control. |
| 6 | **Telecommunications (Radio Equipment Possession Authorisation) Rules, 2026** — G.S.R. 592(E), 8 Jul 2026; applications from 06.08.2026 | ⚠️ **Telecom Act 2023 §3(1)(c) now requires authorisation to *possess* radio equipment.** Almost all secondary sources are stale. **Needs written DoT clarification.** |
| 7 | **ETA is now 100% self-declaration**, ₹10,000, lifetime, since 09.09.2024 | Low cost for project quantity. |
| 8 | ⚠️ The common claim that India caps 2.4 GHz Wi-Fi at 20 dBm EIRP is **wrong** for the 2005 rules | 30 dBm TX / 36 dBm EIRP. Genuine instrument ambiguity — do not rely on blogs. |

⚠️ **`wpc.gov.in` does not exist.** Several sources cite it. The correct host is
`wpc.dot.gov.in`.

---

## What a heartbeat would carry

~35 bytes, and it is genuinely useful:

| Field | Bytes |
|---|---|
| Device ID, sequence | 8 |
| Battery % and mV | 2 |
| Charging state | 1 |
| Robot state enum — IDLE / MOVING / CHARGING / FAULT / E-STOP | 1 |
| Fault code | 2 |
| Position X, Y | 8 |
| ⚠️ **Pose confidence — dead-reckoned vs last-known** | **1** |
| Zone or task ID | 2 |
| CRC | 2 |

⚠️ **The `pose confidence` field is mandatory.** It exists precisely to stop a
stale last-known position being mistaken for the robot's current position.

---

## What is impossible on this link

| Cannot | Why |
|---|---|
| Camera stills | JPEG ≈ 20–60 kB → 20–60 s airtime per frame. Duty cycle >100% |
| Video | Not a closeable question |
| LiDAR | Even a decimated occupancy grid is kilobytes |
| Motor control or closed loop | Hundreds of ms to seconds, non-deterministic, no QoS |
| Data needed within 1 second | SF7 minimum airtime ~38 ms; SF9–SF12 typical |
| ⛔ **Positioning** | ⛔ Signal strength must **never** drive navigation or floor identity |

---

## Hardware, if ever justified

| Component | Note |
|---|---|
| Radio | Semtech SX1262 or similar, **865–868 MHz variant** |
| Board | ESP32 + LoRa, or a dedicated LoRa node |
| ⚠️ Verify | Heltec V3 HF spans 863–928 MHz, but its own datasheet lists "IN868" as **865–867 MHz** — narrower than the gazette. **Verify on the bench.** |
| TX power | Capped to remain within e.r.p. limits; Adaptive Power Control implemented |
| Antenna | Must remain within gain limits; placement avoids floor voids and stairwells |
| Power | **Must be mains-powered.** Battery life is not a factor on a docked robot. |
| Gateway | Secure location with power and network back to the server |

---

## Security requirements if ever built

| Requirement | Why |
|---|---|
| Authenticated messages (HMAC or AEAD) | Unauthenticated telemetry is a spoofing vector |
| Monotonic sequence numbers | Replay protection |
| Per-device keys | Prevents one node impersonating another |
| Rate limiting | Prevents channel abuse |
| ⚠️ **Gateway can never command the robot** | **Telemetry only, inbound to the server.** This is a hard constraint, not a preference. |

---

## Prohibited uses

- ⛔ As the robot's control link
- ⛔ As a navigation data channel
- ⛔ As a positioning source
- ⛔ For camera, video, or LiDAR
- ⛔ Operating outside 865–868 MHz
- ⛔ Exceeding the e.r.p. limit or omitting Adaptive Power Control
- ⛔ Operating without a written DoT possession-authorisation clarification

---

## Questions

| ID | Question | Priority |
|---|---|---|
| `SH-RAD-001` | **Is there a demonstrated communication failure this would solve?** | **P0 — gates the category** |
| `SH-RAD-012` | What measured floor attenuation exists here? | P1 |
| `SH-RAD-016` | Confirm the radio path can never command the robot | **P0** |
| `SH-RAD-018` | Does the school need a 2026 possession authorisation? | **P0** |
| `SH-RAD-020` | Does school policy permit an additional RF device? | P1 |

Full regulatory research with gazette citations: `docs/shana/research/appendix_india_rf_research.md`.