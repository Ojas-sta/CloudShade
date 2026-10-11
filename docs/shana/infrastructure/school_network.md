# SHANA — School Network Integration

**Status: RESEARCH ONLY. No school network has been accessed, probed, or
scanned. Every question in this document is for the IT department to answer from
existing records.**

---

## The one fact that determines the design

> **The robot link is wired Ethernet over Cyclone DDS. This is not a preference.**
>
> *"No, Go2 only supports wired connection for development."*
> — Unitree developer FAQ

Discovery uses multicast, which is unreliable over Wi-Fi. **A robot on Wi-Fi
would be non-conformant with the manufacturer and would fail in ways very hard to
diagnose.**

### The consequence that matters

⚠️ **Because the robot link is Ethernet, a WLAN failure does not stop the robot.**

This is a safety benefit and it substantially weakens the case for radio backup
telemetry. See `docs/shana/hardware/esp32_and_lora.md`.

---

## Proposed topology

```
Teacher devices ──┐
                  ├──► SHANA SERVER ──eth0──► school LAN (dashboard access)
Existing LAN ─────┘                 └──eth1──► robot subnet (isolated)
                                                        │
                                                 WIRED ETHERNET
                                                        │
                                                   Go2 EDU
```

**Two network interfaces on the server. The robot subnet is isolated from
everything else.**

---

## Robot subnet specification

| Property | Value | Class |
|---|---|---|
| Subnet | `192.168.123.0/24` | VERIFIED — Unitree documentation |
| Robot onboard | `192.168.123.161` | VERIFIED |
| Server | `192.168.123.222` | VERIFIED |
| DDS domain | `0` | VERIFIED — SDK source |
| Discovery | CycloneDDS SPDP multicast | VERIFIED |
| Transport | **Wired Ethernet** | VERIFIED |

⚠️ **Never assign `192.168.123.161` to the host** — it is the robot's address.

### Known failure mode, documented by Unitree

> *"192.168.123.161/24 and 192.168.1.10/24 on the same VLAN … you can discover
> each other, but unicast communication fails."*

**Mitigation:** the robot subnet must not share a broadcast domain with anything
else, including the smartboard LAN.

---

## ⚠️ The multicast problem

The robot's discovery protocol **requires UDP multicast**, which is frequently
blocked by default on managed networks.

⚠️ **If multicast is blocked, the connection fails in a way that looks like a
network fault, a firewall fault, or a robot fault.** It is the single most likely
cause of a "nothing works" scenario.

**IT must confirm this explicitly** (`SH-NET-007`).

---

## Options in preference order

### Option A — Direct cable ✅ recommended

A dedicated USB-Ethernet NIC on the server, cabled directly to the robot.

- ✅ No school network change at all
- ✅ Maximum isolation
- ✅ Nothing for IT to maintain
- ⚠️ Requires two NICs and physical cable routing
- ⚠️ Still requires school approval for the cable

**This is the option that asks the least of the IT department.**

### Option B — Dedicated VLAN

- ✅ Proper engineering, supports multiple robots
- ⚠️ Requires VLAN configuration and a switch port
- ⚠️ Requires multicast to be permitted within the VLAN

### Option C — Wireless bridge 🔵 not recommended

⚠️ **Rejected for the robot link.** Even via a bridge, DDS discovery remains
fragile and the manufacturer's support position is for wired connections. May be
considered only as a last resort, with written IT approval.

---

## Teacher access

| Property | Value |
|---|---|
| Protocol | HTTPS to a local address |
| Reachability | School LAN only |
| **Public exposure** | **None.** No port forwarding |
| **Internet dependency** | **None.** Core operation works offline |
| Destination selection | **Allow-list only** — free text never accepted |
| Local DNS | Optional convenience, not required |

---

## What must never happen

| Prohibited | Why |
|---|---|
| Route the robot subnet to the school LAN | Robot could reach school systems |
| Bridge the robot subnet to Wi-Fi | Fragile; violates manufacturer guidance |
| Add the robot to the smartboard VLAN | Not a supported configuration |
| Expose the server publicly | Design principle; student safety |
| Enable developer mode on school Wi-Fi | Unnecessary exposure |
| Store credentials in the repository | Absolute rule |
| **Scan or probe the school network** | **Prohibited. IT checks what needs checking.** |

---

## Questions for IT

| ID | Question | Priority |
|---|---|---|
| `SH-NET-001` | Existing wired LAN topology? Switch model? VLANs? | **P0** |
| `SH-NET-002` | Free Ethernet port, or can a USB-Ethernet adapter be added? | **P0** |
| `SH-NET-003` | Is a dedicated robot segment permitted? | **P0** |
| `SH-NET-004` | Can a static IP be assigned? | P1 |
| `SH-NET-005` | **Are VLANs or client isolation enabled?** | **P0** |
| `SH-NET-006` | Can teachers reach a local server? | P1 |
| `SH-NET-007` | **Is UDP multicast permitted on the subnet?** | **P0 — most likely failure point** |
| `SH-NET-008` | Wi-Fi inventory and per-floor coverage | P1 |
| `SH-NET-009` | **Is there any room with no Wi-Fi?** | P1 — decides LoRa |
| `SH-NET-011` | Is a private wireless network permitted? | P1 |
| `SH-NET-012` | Are additional access points permitted? | P2 |
| `SH-NET-014` | Is PoE available? | P2 |
| `SH-NET-015` | Is a local-only network without internet permitted? | P1 |
| `SH-NET-018` | Is there a rule against storing credentials in repositories? | **P0** |
| `SH-NET-022` | Scheduled outages, maintenance, load-shedding? | P1 |

Full request: `docs/shana/administration/technical_assessment_request.md` Part 2.

---

## Pre-connection checklist

- [ ] Written authorisation to use the development interface
- [ ] Written approval for the network segment
- [ ] Static IP assigned, verified not to collide
- [ ] ⚠️ **Multicast confirmed permitted**
- [ ] Client isolation confirmed not to block the link
- [ ] Direct-cable option attempted **before** any VLAN work, as a fallback
- [ ] **Gate 0 and Gate 1 passed**
- [ ] No scanning or probing of any network beyond what IT has authorised

⚠️ **The last item is absolute.** If something needs checking, IT checks it.