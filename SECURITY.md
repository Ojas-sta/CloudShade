# Security Policy

## Reporting

Report suspected vulnerabilities or accidental exposure of sensitive information
to the **repository owner** directly. Do not open a public issue containing
sensitive details.

---

## SHANA-specific security concerns

SHANA is a physical device operating in a school among children. Several
concerns are not conventional software vulnerabilities.

### 1. Physical safety is the primary concern

Security failures in this project have physical consequences. A compromised or
malfunctioning control path could move a 15 kg machine among students.

**Design rules that exist for this reason:**

- No AI model may reach the robot. A deterministic validator sits between every
  request and every motion command.
- Teachers never receive motor-control access.
- The software cannot be the only means of stopping the robot. A human spotter
  with unilateral authority is required at every stage.
- The server is never exposed to a public network.

⚠️ **None of these has been implemented.** They are requirements. See
`docs/shana/feasibility/go_no_go_gates.md`.

### 2. Credential and configuration hygiene

**Never commit, and never paste into an issue or document:**

- Wi-Fi keys or passwords
- CCTV credentials or stream URLs
- API tokens or certificates
- Robot or server credentials
- Private network topology, IP assignments, or VLAN configuration
- Student or staff personal data
- Building layouts not approved for circulation

⚠️ If any of these have already been committed, treat it as an incident: notify
the repository owner, and follow the school's breach-notification process
(`SH-PRIV-022`).

### 3. Threats against the robot's control path

| Threat | Required control |
|---|---|
| Unauthenticated movement command | Commands only accepted from an authenticated server on an isolated subnet |
| Teacher account misuse | Role separation; no motor-control role for teachers |
| Prompt injection reaching the robot | LLM output validated against a schema and a closed registry; no tool access; human confirmation |
| Replay of a movement command | Nonce and sequence validation on the control path |
| Rogue device on the robot subnet | Isolated VLAN; no bridging to the school LAN |
| Physical tampering with the robot | Spotter presence; tamper response procedure |

### 4. Threats against student privacy

| Threat | Required control |
|---|---|
| Facial recognition | **Prohibited.** Not implemented, not planned |
| Identity tracking | **Prohibited.** Anonymous counts only |
| Automated misconduct reporting | **Prohibited.** Human review required for any incident report |
| Raw video retention | Frames processed in-stream and discarded by default |
| Data exposure via repository | Floor plans, routes, and telemetry are treated as sensitive |
| Misuse of occupancy data | Derived counts are never used for individual assessment or attendance |

### 5. Threats against the school's network

SHANA must not become an attack path into the school network. The robot subnet
must be isolated. Any wireless component requires explicit approval
(`SH-NET-012`).

### 6. Threats against the lifts

⛔ **Any modification to a lift is prohibited.** Not as a security control — as a
legal and safety matter. A robot tampering with a lift controller would be a
planned, repeatable intervention, and the school as occupier inherits liability.

See `docs/shana/infrastructure/elevator_feasibility.md`.

---

## Known risks

This project is in feasibility discovery. The following are **known and
unresolved**, documented rather than hidden:

| Risk | Status |
|---|---|
| Platform-level cryptographic weaknesses in Unitree hardware | Documented in the sibling platform study. **Network isolation is mandatory.** |
| Telemetry behaviour of the platform | Unverified. Must be assessed against school IT policy before any network connection. |
| No application-level authentication exists yet | Nothing has been built. |
| No physical security assessment exists | Nothing has been built. |
| Lift integration risk | Unresolved and potentially legally prohibited. |

⚠️ **A risk that has not been assessed is not the same as a risk that does not
exist.** See `docs/shana/safety/risk_register.md`.

---

## Responsible disclosure

- **Do not** publicly disclose an unresolved safety or privacy concern.
- **Do** report it to the repository owner and, where student safety or privacy
  is affected, to the school's designated authority.
- Remember: **this is a school project involving children.** Escalate quickly.