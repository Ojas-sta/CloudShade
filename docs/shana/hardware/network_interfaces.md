# Network Interfaces

⚠️ **All proposed. None configured, none tested, no network access obtained.**

---

## 1. 🔴 The primary constraint: wired Ethernet

| Fact | Detail |
|---|---|
| Primary robot link | 🔴 **Wired Ethernet** |
| Rationale | Reliability, deterministic latency, and school security policy |
| Decision | ADR-004 — Accepted |
| Wireless as primary | ⛔ **Not permitted for the control link** |

Verbatim constraint context: wireless operation is restricted on school
infrastructure and is not acceptable for a safety-relevant control path.

⚠️ **Consequence: the robot must physically trail a cable, or a cable-free
dock-and-run design must be agreed with the school.** This has never been
discussed with the school and may be a decisive practical objection.

---

## 2. Proposed topology

```text
                    ┌──────────────────────────────┐
                    │      SCHOOL LAN (unverified) │
                    └──────────────────────────────┘
                         │              │
        ┌────────────────┘              └───────────────┐
        │                                               │
┌───────▼────────┐                            ┌─────────▼────────┐
│  SHANA station  │◄────── task / status ─────►│  CloudLight      │
│  (control PC)   │                            │  (other agent)   │
└───────┬────────┘                            └──────────────────┘
        │
        │  🔴 ETHERNET (cabled)
        │
┌───────▼────────┐
│   Unitree Go2  │
│   EDU (presumed)│
└────────────────┘
```

⚠️ **This diagram is a proposal. No network of this shape has been built.**

---

## 3. Proposed interfaces

| # | Interface | Direction | Protocol | Status |
|---|---|---|---|---|
| 3.1 | SHANA ↔ Go2 | Bidirectional | Cyclone DDS over Ethernet | `RESEARCH_ONLY` |
| 3.2 | SHANA ↔ CloudLight | Bidirectional | Proposed JSON messages | `DESIGN_ONLY` |
| 3.3 | SHANA ↔ School systems | Inbound | HTTP/notification | `BLOCKED` — no permissions |
| 3.4 | CCTV | Inbound | ONVIF/RTSP | ⛔ `DEPRECATED` — recommended against |
| 3.5 | Lift | Bidirectional | — | 🔴 **No sanctioned API exists** |
| 3.6 | Radio telemetry | Outbound | LoRa/ESP32 | `RESEARCH_ONLY` — deferred, ADR-012 |

---

## 4. 🔴 Design rules

| # | Rule | Rationale |
|---|---|---|
| 4.1 | 🔴 **Safety functions never depend on the network** | A dropped link must never cause unsafe motion |
| 4.2 | 🔴 **SHANA degrades safely when CloudLight is unreachable** | Invariant 4.6 |
| 4.3 | 🔴 **`system.stop` honoured even if the network is degraded** | Invariant 4.2 |
| 4.4 | 🔴 **No student-identifying data on any interface** | Absolute privacy position |
| 4.5 | 🔴 **No raw video leaves the robot** | ADR-010 |
| 4.6 | ⚠️ **No credentials in code or configuration committed to the repository** | See [`../safety/security.md`](../safety/security.md) |
| 4.7 | ⚠️ **All traffic logged without payloads** | Auditability without surveillance |

---

## 5. 🔴 Status

| Question | Answer |
|---|---|
| Has a network been designed? | ⚠️ Proposed only |
| Has a network been built? | ❌ **No** |
| Has a network been tested? | ❌ **No** |
| Does the school permit it? | ❌ **Unknown — 0 of 102 permissions granted** |
| Is the school's IP plan known? | ❌ **Unknown** |
| Is a VLAN required? | ❌ **Unknown — school IT decision** |

⚠️ **Nothing about the school's network is known.** The topology above is
entirely hypothetical and must be validated with school IT before any
implementation.

---

## 6. 🔴 Required before any network work

| # | Requirement | Who |
|---|---|---|
| 6.1 | School network policy and IP plan | School IT |
| 6.2 | VLAN or subnet allocation for the robot | School IT |
| 6.3 | Firewall rules | School IT |
| 6.4 | Wireless policy confirmation | School IT |
| 6.5 | Cable-routing approval (if wired) | School facilities |
| 6.6 | Cybersecurity review | School IT |

See [`../administration/technical_assessment_request.md`](../administration/technical_assessment_request.md).