# SHANA — Network Architecture

**Status: PROPOSED. No network has been probed, scanned, or connected to.**

---

## The single most important network fact

⚠️ **The robot-to-server link is wired Ethernet over Cyclone DDS. This is not a
preference — it is the only manufacturer-supported path.**

> *"No, Go2 only supports wired connection for development."*
> — Unitree developer FAQ, `https://support.unitree.com/home/en/developer/faq`

Discovery uses multicast, which is unreliable over Wi-Fi. **A design that put the
robot on Wi-Fi would be non-conformant with the manufacturer and would fail
intermittently in ways that are very hard to diagnose.**

### What follows from that

Because the robot link is Ethernet:

- ✅ **A WLAN failure does not stop the robot.** This is a safety benefit.
- ⚠️ **A cable, port, or NIC failure does stop it** — hence the physical
  supervisor.
- ❌ **LoRa cannot substitute for it.** Radio telemetry is a watchdog, not a
  control path.

---

## Proposed topology

```
                        SCHOOL NETWORK
     ┌───────────────────────────────────────────────────────┐
     │                                                       │
     │   ┌──────────────┐                                    │
     │   │ Smartboards  │──────── existing LAN ──────────┐   │
     │   └──────────────┘                                  │   │
     │   ┌──────────────┐                                  │   │
     │   │ Teacher PCs  │── Wi-Fi / wired ──┐              │   │
     │   └──────────────┘                  │              │   │
     │                                      ▼              │   │
     │                          ┌───────────────────────┐  │   │
     │                          │  SHANA SERVER         │  │   │
     │                          │  (AI Lab, CP-C3)      │  │   │
     │                          │  eth0 ──► school LAN  │  │   │
     │                          │  eth1 ──► robot subnet│  │   │
     │                          └───────────┬───────────┘  │   │
     └──────────────────────────────────────┼──────────────┘   │
                                            │                  │
              ═══════════ ISOLATION BOUNDARY ══════════════   │
                                            │                  │
                              ┌─────────────▼─────────────┐   │
                              │  ROBOT SUBNET             │   │
                              │  192.168.123.0/24          │   │
                              │                            │   │
                              │  Server eth1 : .222       │   │
                              │  Go2 onboard  : .161      │   │
                              │                            │   │
                              │  ← WIRED ETHERNET ONLY →   │   │
                              └────────────────────────────┘   │
```

---

## Robot subnet specification

| Property | Value | Source |
|---|---|---|
| Subnet | `192.168.123.0/24` | Unitree documentation ✅ |
| Robot onboard | `192.168.123.161` | Unitree documentation ✅ |
| Development host | `192.168.123.222` (or `.99`) | Unitree documentation ✅ |
| DDS domain | `0` | `unitree_sdk2` source ✅ |
| Discovery | CycloneDDS SPDP multicast | ✅ |
| Transport | **Wired Ethernet** | Unitree FAQ ✅ |
| Operating system | Ubuntu 20.04 / 22.04 | Unitree FAQ ✅ |
| NIC selection | Passed explicitly at startup | SDK source ✅ |

⚠️ **Do not assign `192.168.123.161` to the host.** Unitree states this directly:
it is the robot's own address.

### Known failure mode, documented by Unitree

> *"192.168.123.161/24 and 192.168.1.10/24 on the same VLAN … you can discover
> each other, but unicast communication fails."*

**Mitigation:** the robot subnet must not share a broadcast domain with any other
network, including the smartboard LAN.

---

## Three link options, in preference order

### Option A — Direct cable (preferred) ✅

A dedicated USB-Ethernet NIC on the server, cabled **directly** to the robot.

| Advantage | Consequence |
|---|---|
| No school network change at all | Requires physical cable routing |
| Maximum isolation | Server needs two NICs |
| Nothing for IT to maintain | School approval still needed for the cable |
| Simplest thing that could work | |

⚠️ **This is the recommended option and it requires the least from the school.**

### Option B — Dedicated VLAN and switch port

| Advantage | Consequence |
|---|---|
| Serves multiple robots | Requires IT VLAN configuration |
| Proper network engineering | Requires IT to permit a new VLAN |

Requires: `SH-NET-003`, `SH-NET-005`, `SH-NET-007`.

### Option C — Wireless bridge 🔵 not recommended

⚠️ **Rejected as the robot link.** Even via a bridge, the DDS discovery remains
fragile and the manufacturer's support position is for wired connections. May be
considered only as a last resort for a fixed installation, and only with written
IT approval.

---

## Teacher access path

| Property | Value |
|---|---|
| Protocol | HTTPS to a local address |
| Reachability | School LAN only |
| Public exposure | **None.** No port forwarding, no public hosting |
| Internet dependency | **None.** Core operation works offline |
| Authentication | School accounts or standalone; role-based |
| Destination selection | **Allow-list only.** Free text never accepted |

⚠️ The server binds to the LAN interface only. A local DNS name would be
convenient but is not required.

---

## What the network must NOT do

| Prohibited | Why |
|---|---|
| Route the robot subnet to the school LAN | Robot could reach school systems |
| Bridge the robot subnet to Wi-Fi | Fragile discovery; violates manufacturer guidance |
| Expose the server publicly | Design principle; student safety |
| Add the robot to the smartboard VLAN | Not a supported configuration |
| Enable developer mode on school Wi-Fi | Unnecessary exposure |
| Store credentials in the repository | Absolute rule |

---

## Radio telemetry (optional, deferred)

⚠️ **Deferred.** Do not build unless `SH-NET-009` identifies a specific room with
no Wi-Fi coverage.

With one working access point per floor, **Wi-Fi outperforms LoRa for coverage**
in a multi-storey building. Radio telemetry buys **independence from the WLAN**,
not range — and the robot link is Ethernet anyway, so a WLAN outage does not stop
the robot.

If built, it is:

- **Telemetry only, inbound to the server.** ⛔ Never commands the robot.
- ~35 bytes per report at a low reporting rate.
- Never used for positioning. ⛔ RSSI is not a positioning input.
- Independently authenticable, with replay protection.

Full regulatory detail, including Indian spectrum constraints, is in
`docs/shana/hardware/esp32_and_lora.md` and `docs/shana/research/appendix_india_rf_research.md`.

---

## CCTV network path (not requested, not approved)

⚠️ **No CCTV system has been accessed.** If analytics were ever approved, it would
require its own network segment, processing on the server or a dedicated host,
and no recording by default.

Full analysis in `docs/shana/infrastructure/cctv_feasibility.md`.

---

## Network questions that block everything

| ID | Question | Why it blocks |
|---|---|---|
| `SH-NET-005` | Are VLANs or client isolation enabled? | Would silently break the link |
| `SH-NET-007` | Is UDP multicast permitted on the subnet? | ⚠️ **Discovery fails without it — a common default-blocked protocol** |
| `SH-NET-003` | Is a dedicated segment permitted? | Determines isolation |
| `SH-NET-002` | Is a free Ethernet port available? | Determines physical feasibility |
| `SH-NET-014` | Is PoE available for access points? | Determines teacher-access coverage |
| `SH-NET-009` | Is there a Wi-Fi dead zone? | Determines whether radio is justified |

---

## Pre-connection checklist

Before any connection to the robot is attempted:

- [ ] Written authorisation to use the development interface (`SH-GO2-008`)
- [ ] Written approval for the network segment (`SH-NET-003`)
- [ ] Static IP assigned and verified not to collide
- [ ] Multicast confirmed permitted
- [ ] **Gate 0 and Gate 1 passed**
- [ ] A direct-cable option attempted **before** any VLAN work, as a fallback
- [ ] No scanning or probing of any network beyond what IT has explicitly
      authorised

⚠️ **The last point is absolute.** No network scanning, port sweeping, or probing
of school infrastructure. If something needs checking, IT checks it.