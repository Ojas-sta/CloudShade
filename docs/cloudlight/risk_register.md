# CloudLight — Risk Register

**Phase 0 · 2026-10-11**

Risks to the proposed CloudLight platform. **Nothing is built.** No control
described here has been implemented, tested, or demonstrated. Every mitigation
is a *plan*, and every rating is an *assessment made without school input*.

**Risk rating:** `L` likelihood × `S` severity, both 1–5.
🔴 High (15–25) · 🟠 Medium (8–14) · 🟡 Low (1–7)

**Status:** `MITIGATION PLANNED` · `OPEN — APPROVAL REQUIRED` ·
`OPEN — ANSWER REQUIRED` · `MONITOR`

⚠️ **The largest risk is not a bug.** It is that a local-first design with no
operations staff becomes one machine, in one room, on one circuit, that one
person understands. That is structural, not incidental, and no code addresses it.
It is recorded as R-01, R-09, R-34 and R-35 because each is separately
unanswerable.

---

## 1. Executive risk summary

| # | Risk | Domain | Rating | Status |
|---|---|---|---|---|
| R-01 | Local-first collapses into a single host with no redundancy | Technical | 🔴 20 | OPEN — ANSWER REQUIRED |
| R-05 | No always-on host permitted | Network and Hosting | 🔴 20 | OPEN — APPROVAL REQUIRED |
| R-06 | Smartboards cannot reach the server | Network and Hosting | 🔴 20 | OPEN — ANSWER REQUIRED |
| R-08 | Flat school LAN; control plane is one hop from every classroom | Network and Hosting | 🔴 20 | OPEN — ANSWER REQUIRED |
| R-15 | The wrong timetable is published to every screen | Data and Timetable | 🔴 20 | MITIGATION PLANNED |
| R-16 | The wrong substitution is displayed to students | Data and Timetable | 🔴 20 | MITIGATION PLANNED |
| R-21 | No staff identity directory, so nothing is attributable | Security and Privacy | 🔴 20 | OPEN — ANSWER REQUIRED |
| R-45 | No named approver — nothing may lawfully be stored | Permission and Governance | 🔴 20 | OPEN — APPROVAL REQUIRED |
| R-07 | Client isolation silently blocks the whole design | Network and Hosting | 🔴 16 | OPEN — ANSWER REQUIRED |
| R-11 | Windows edition unknown; kiosk design unvalidated | Windows and Device | 🔴 16 | OPEN — ANSWER REQUIRED |
| R-12 | Kiosk account cannot install or self-update | Windows and Device | 🔴 16 | OPEN — ANSWER REQUIRED |
| R-13 | A Windows feature update breaks the kiosk mid-lesson | Windows and Device | 🔴 16 | OPEN — ANSWER REQUIRED |
| R-17 | An OCR misread is silently published | Data and Timetable | 🔴 16 | MITIGATION PLANNED |
| R-18 | Stale data presented as current on a wall screen | Data and Timetable | 🔴 16 | MITIGATION PLANNED |
| R-22 | A shared six-digit code becomes identity in practice | Security and Privacy | 🔴 16 | OPEN — APPROVAL REQUIRED |
| R-23 | An unattended board holds a live teacher session | Security and Privacy | 🔴 16 | OPEN — ANSWER REQUIRED |
| R-24 | The SHANA task channel is unauthenticated | Security and Privacy | 🔴 16 | OPEN — APPROVAL REQUIRED |
| R-26 | An unsigned update is accepted from an untrusted LAN | Security and Privacy | 🔴 16 | MITIGATION PLANNED |
| R-28 | Confidential staff material sits beside teaching resources | Security and Privacy | 🔴 16 | OPEN — APPROVAL REQUIRED |
| R-30 | School data sent to an external AI service | AI and Data Protection | 🔴 16 | OPEN — APPROVAL REQUIRED |
| R-34 | The one person who understands the system leaves | Operations and Support | 🔴 16 | MITIGATION PLANNED |
| R-19 | No shared storage exists at all | Data and Timetable | 🟠 12 | OPEN — ANSWER REQUIRED |
| R-25 | A replayed request creates a duplicate delivery task | Security and Privacy | 🟠 12 | MITIGATION PLANNED |
| R-27 | Secrets leak into the repository | Security and Privacy | 🟠 12 | MITIGATION PLANNED |
| R-29 | Search returns files the user cannot open | Security and Privacy | 🟠 12 | MITIGATION PLANNED |
| R-38 | The budget covers nothing | Programme and Scope | 🟠 12 | OPEN — ANSWER REQUIRED |
| R-39 | Scope grows faster than capability | Programme and Scope | 🟠 12 | MITIGATION PLANNED |
| R-40 | The school rejects the project | Programme and Scope | 🟠 12 | OPEN — ANSWER REQUIRED |
| R-41 | The pilot fails | Programme and Scope | 🟠 12 | OPEN — ANSWER REQUIRED |
| R-43 | The SHANA interface is never agreed | SHANA Integration | 🟠 12 | OPEN — ANSWER REQUIRED |

⚠️ **Nine of the highest-rated risks are `OPEN` and cannot be closed by
research.** Each needs a person, a measurement, or a signature.

---

## 2. Full risk register

### 2.1 Technical

#### R-01 🔴 20 — Local-first collapses into a single host with no redundancy
**OPEN — ANSWER REQUIRED** · Owner: Internal (architecture)

Database, search index, audit log, secrets and signing key on **one machine, one room, one circuit**. No second copy, no failover. `L5` — this is not a failure mode, it is the default outcome of the chosen design. `S4` — total outage, and personal data simultaneously unavailable and undefended.
**Mitigation:** accept concentration explicitly as the price of one-operator operability, and buy back resilience with speed of recovery — off-host encrypted backups, a rehearsed restore, and displays that degrade to a clearly-marked cached timetable without the server.
**Residual:** 🟠 Concentrated **by design**. ⚠️ No signature has been obtained for that choice.
**Refs:** `CL-SRV-001` `CL-SRV-016` `CL-OPS-014` `CL-OPS-015` `CL-DATA-008`

#### R-02 🟠 12 — Frontend/backend version skew across mixed boards
**MITIGATION PLANNED** · Owner: Internal (architecture)

Displays run whatever bundle they last downloaded. A board that misses a deployment runs an old bundle against a new backend and may render nothing. `L3` — likely during any rollout, certain on an unreliable network. `S4` — a blank screen in front of a class.
**Mitigation:** every display reports its version; the backend stays compatible with the oldest bundle it knows about; skew is a defined, tested rendering, not an error; boards that fall too far behind are pinned to the last known-good version.
**Residual:** 🟠 Low once the compatibility window exists and has been tested.
**Refs:** `CL-OTA-006` `CL-OTA-021` `CL-GIT-018`

#### R-03 🟠 12 — The timetable model cannot represent the school's actual day
**MITIGATION PLANNED** · Owner: Internal (architecture)

The designed model assumes a regular grid of equal periods. Real schools hold split and part-time teachers, shared rooms, exam slots, half-days and one-off cancellations. If the model cannot represent them, **correct data gets rejected** — and a rejected correct entry becomes a wrong display. `L4` — irregular days are near-universal. `S3` — bounded, recoverable display errors.
**Mitigation:** design the model against irregularity before regularity; an unscheduled period is a first-class state, never a missing row; rejection messages must be actionable for whoever fixes the source.
**Residual:** 🟠 Cannot retire without the real timetable structure.
**Refs:** `CL-TIME-006` `CL-TIME-012` `CL-TIME-015` `CL-TIME-018` `CL-TIME-020`

#### R-04 🟠 12 — Every sizing number in this project is an estimate
**MONITOR** · Owner: Internal

How many boards, teachers, files, how much RAM, how much storage — every figure in the current documentation is a provisional engineering estimate. None has been measured against this school. `L5` — certain. `S2` — causes over- or under-provisioning, not a safety event.
**Mitigation:** label every provisional sizing as such, as already done in `network_and_hosting.md` §4; re-measure before procurement; never present an estimate to the school as a requirement.
**Residual:** 🟠 Permanent for Phase 0, and the correct state.
**Refs:** `CL-WIN-020` `CL-SEARCH-014` `CL-SRV-013` `CL-SRV-021` `CL-OPS-016`

### 2.2 Network and Hosting

#### R-05 🔴 20 — No always-on host permitted
**OPEN — APPROVAL REQUIRED** · Owner: School administration

CloudLight has no home. Every board, the dashboard, search, the substitution pipeline and the SHANA request interface need a service that survives reboots and runs unattended. If the school will not host a persistent service on its own premises, the local-first premise fails. `L3` — schools reasonably fear always-on machines. `S5` — nothing works without it.
**Mitigation:** ask first, in writing, naming the machine, the power draw and the maintenance obligation. Fallbacks in order — an authorised existing computer; a container or VM on an existing authorised host; a small dedicated machine if budget exists. ⚠️ A cloud-hosted VM contradicts local-first and is a **scope change needing explicit agreement**, not a convenience.
**Residual:** 🟡 Binary — the school permits it, or the architecture changes.
**Refs:** `CL-GEN-002` `CL-SRV-001` `CL-SRV-002` `CL-SRV-007` `CL-SRV-008`

#### R-06 🔴 20 — Smartboards cannot reach the server
**OPEN — ANSWER REQUIRED** · Owner: School IT

Displays must resolve a local name and open a connection to a host on a different subnet. DNS, DHCP, firewall rules and subnet reachability are all school-owned and all unknown. `L4` — the single most common reason a local-first display cannot reach its own server. `S5` — total failure.
**Mitigation:** treat the chain as four things that must *each* be established — addressing, name resolution, route, firewall. ⚠️ A working ping is **not** proof of working application access. Prefer a memorable local hostname and a DHCP reservation over a hard-coded IP, and document the exact rules to request.
**Residual:** 🟡 Resolved by measurement and configuration, not by design.
**Refs:** `CL-NET-003` `CL-NET-004` `CL-NET-006` `CL-NET-012` `CL-NET-014`

#### R-07 🔴 16 — Client isolation silently blocks the whole design
**OPEN — ANSWER REQUIRED** · Owner: School IT

Managed access points commonly enable AP or client isolation as a default. It blocks device-to-device traffic on the wireless network. **Nothing errors.** Boards appear on the network and simply cannot reach a local server on the same LAN. `L4` — a standard managed-AP default. `S4` — silent, total, and diagnosed only by someone who knows to look.
**Mitigation:** ask IT for the isolation setting directly and specifically before anything is deployed; keep a documented wired alternative for at least the pilot rooms; ⚠️ never design around isolation by hoping it is off.
**Residual:** 🟠 Low once measured. The danger is not knowing it is on.
**Refs:** `CL-NET-008` `CL-NET-005` `CL-NET-009`

#### R-08 🔴 20 — The school LAN is flat; the control plane is one hop from every classroom
**OPEN — ANSWER REQUIRED** · Owner: School IT + Principal

If classroom devices are not segmented from office, laboratory, CCTV and guest traffic, then any compromised or merely curious classroom device is **one hop from the control plane** holding staff and student personal data. `L4` — most school networks are flat. `S5` — a breach affecting every staff member.

⚠️ **This risk is honest in an uncomfortable way. A school LAN without segmentation genuinely is high risk, and no amount of application-layer authentication makes a flat network safe for staff data.** The honest options are to segment it, or to accept the exposure in writing.

**Mitigation:** ask what segmentation exists *before* asking for anything else. Where none exists, compensate at the application layer — server-mediated storage access, a separated administrative surface, default-deny egress — and **obtain written acceptance of the residual exposure**, signed by someone who can be asked about it in a year's time.
**Residual:** 🔴 Not eliminable without network work the project cannot perform.
**Refs:** `CL-NET-001` `CL-NET-007` `CL-SEC-023` `CL-SEC-007`

#### R-09 🟠 12 — A power cut takes the server down
**MITIGATION PLANNED** · Owner: Facilities + Internal

One host, one room, one circuit. A power cut stops every display in the school, and recovery requires someone to physically press a button. `L3` — Indian school power interruption is routine. `S3` — minutes of downtime, multiplied across every room.
**Mitigation:** a UPS if one is available; a restart procedure with a named person reachable during school hours; displays that degrade to a clearly-marked cached timetable rather than blanking; clean shutdown and restart needing no console.
**Residual:** 🟠 Long outages are outside our control. Honest degradation is the whole mitigation.
**Refs:** `CL-SRV-003` `CL-SRV-011` `CL-SRV-012` `CL-NET-019` `CL-DATA-014`

#### R-10 🟠 12 — The school network fails during a demonstration
**MITIGATION PLANNED** · Owner: Internal

A demonstration is judged once, in front of people, on a day chosen by someone else. If the network drops, the display is the worst possible place to discover it. `L3` — school networks fail, especially under demonstration load. `S3` — reputational, and programme-threatening if the pilot depends on it.
**Mitigation:** every display works from a cached bundle and cached timetable; the degraded state is a designed, honest screen, not a blank one; demonstration data is labelled as demonstration; a recorded fallback exists.
**Residual:** 🟠 A live outage during a demonstration cannot be engineered away, only survived.
**Refs:** `CL-EXPO-006` `CL-EXPO-007` `CL-OTA-014` `CL-SEC-021`

### 2.3 Windows and Device

#### R-11 🔴 16 — The Windows edition is unknown, invalidating the kiosk design
**OPEN — ANSWER REQUIRED** · Owner: School IT

Assigned Access and kiosk behaviour differ sharply across editions, and the autostart mechanism differs with them. The whole display architecture rests on an edition nobody has confirmed. `L4` — nothing has been checked. `S4` — a missing feature forces a redesign.
**Mitigation:** a photograph of `winver` from one board answers this in under a minute. ⛔ Do not design a kiosk for a specific edition before that photograph exists.
**Residual:** 🟡 Trivial once measured; embarrassing to have assumed.
**Refs:** `CL-WIN-001` `CL-WIN-002` `CL-WIN-006` `CL-WIN-007`

#### R-12 🔴 16 — The kiosk account cannot install or self-update
**OPEN — ANSWER REQUIRED** · Owner: School IT

Without local elevation and a machine-wide writable location, the display agent cannot install, self-update or self-repair. Every future change then requires a human walking to the board. `L4` — least-privilege lockdown is the norm. `S4` — the update model cannot function.
**Mitigation:** establish what privilege exists today before designing around it; where it does not, deliver kiosk configuration as **signed configuration data** rather than as a software install, which needs far less privilege; a browser-first client is the fallback needing least of all.
**Residual:** 🟠 Constrains the update architecture permanently.
**Refs:** `CL-WIN-003` `CL-OTA-010` `CL-OTA-011` `CL-WIN-009`

#### R-13 🔴 16 — A Windows feature update breaks the kiosk mid-lesson
**OPEN — ANSWER REQUIRED** · Owner: School IT

Feature updates change autostart, shell-launch and fullscreen behaviour, and boards restart during the teaching day. The restart can land mid-lesson with the screen dark and the timetable absent. `L4` — mid-day restarts are routine. `S4` — a dark screen in front of a class.
**Mitigation:** agree update deferral windows and an out-of-hours reboot policy with IT *before* deployment; the display must recover to a correct timetable with nobody touching it after an unexpected restart; ⛔ no update may interrupt a scheduled lesson or an examination period.
**Residual:** 🟠 IT owns the restart schedule; we can only be resilient to it.
**Refs:** `CL-WIN-013` `CL-WIN-012` `CL-OTA-018` `CL-OPS-021`

#### R-14 🟠 8 — Boards silently drop off the fleet
**MONITOR** · Owner: School IT

A board is reimaged, replaced, returned from repair or moved to another room. It stops running CloudLight, and because nothing reports its own absence, nobody finds out for days. `L3` — routine over a school year. `S2` — one room degrades quietly.
**Mitigation:** every display reports its version and last-seen time, and the fleet view lists boards that have gone quiet; re-enrolment onto the last approved version must be automatic, not a manual IT step.
**Residual:** 🟠 Depends on the school telling us when hardware changes.
**Refs:** `CL-OTA-022` `CL-WIN-021` `CL-WIN-018` `CL-WIN-024`

### 2.4 Data and Timetable

#### R-15 🔴 20 — The wrong timetable is published to every screen
**MITIGATION PLANNED** · Owner: Timetable department

A timetable error propagates to every classroom display at once, is seen by every student and teacher, and is corrected only if someone notices. The highest-consequence correctness failure in the product. `L3` — timetable handling is manual and error-prone today. `S5` — a confident falsehood on every wall in the building.
**Mitigation:** deterministic parsing before any AI; validation that **rejects** contradictions rather than resolving them; a named human approves every publication; full immutable version history with rollback; ⛔ no AI publishes a timetable change.
**Residual:** 🟠 A wrong entry that passes validation is possible. The defence is that it must also pass a person.
**Refs:** `CL-TIME-001` `CL-TIME-022` `CL-TIME-024` `CL-DATA-017` `CL-ADMIN-003` `CL-DEC-012`

#### R-16 🔴 20 — The wrong substitution is displayed to students
**MITIGATION PLANNED** · Owner: Timetable department

A substitution names a real teacher in a real classroom for a real period. An error is visible, embarrassing and operationally damaging, and hard to withdraw once seen. `L4` — substitutions are the most change-heavy, least-structured data in the school. `S5` — a person's professional standing displayed wrongly to their students.
**Mitigation:** three states that must never be conflated — *proposed*, *approved*, *published*; ⛔ never publish unapproved; ⛔ never auto-select a substitute; approval may require two people; withdrawal from every affected display must be fast and rehearsed; a photograph of a classroom board is never authoritative.
**Residual:** 🟠 Late same-day changes are the irreducible part.
**Refs:** `CL-SUB-001` `CL-SUB-002` `CL-SUB-003` `CL-SUB-007` `CL-SUB-012` `CL-SUB-023`

#### R-17 🔴 16 — An OCR misread is silently published
**MITIGATION PLANNED** · Owner: Timetable department

A scanned or photographed schedule is read by an extraction system. A misread — a `7` for a `1`, a struck-out cell read as valid — is structurally plausible, looks correct, and reaches a display unless a human catches it. `L4` — extraction is uncertain by nature. `S4` — the same outcome as R-15 by a quieter route.
**Mitigation:** strict parsing tiers — deterministic extraction of printed tables first, OCR only where nothing else applies; the reviewer sees the original image beside the extracted values; confidence below the agreed threshold **blocks** rather than suggests; ⛔ no AI-extracted change reaches a display without a named human confirmation.
**Residual:** 🟠 A confident reviewer can still accept a misread. The image-beside-values view is the mitigation that matters.
**Refs:** `CL-AIML-013` `CL-AIML-014` `CL-AIML-017` `CL-AIML-022` `CL-SEARCH-005`

#### R-18 🔴 16 — Stale data is presented as current on a wall screen
**MITIGATION PLANNED** · Owner: Internal

The server dies, the network drops, or a board loses its feed. The display shows yesterday's timetable with no indication it is untrue. The teacher trusts it because it looks exactly like the real thing. `L4` — disconnections are routine. `S4` — invisible precisely when it matters most.
**Mitigation:** every displayed value carries its age and a visible last-synced timestamp; above an agreed threshold the screen changes state and says so; the countdown is hidden entirely when the clock is unsynced; a degraded state is a designed screen, not an error page.
**Residual:** 🟡 A design control. Retires with implementation, not with an answer.
**Refs:** `CL-DASH-012` `CL-DASH-013` `CL-DASH-009` `CL-DASH-010` `CL-SEC-018`

#### R-19 🟠 12 — No shared storage exists at all
**OPEN — ANSWER REQUIRED** · Owner: School IT + Library

It is entirely possible this school has no NAS and no shared file server, and that materials live on laptops, pen drives and personal cloud drives. In that case **resource search and the assistant have nothing to index**, and about a third of the product evaporates. `L3` — plausible and common. `S4` — removes the entire resource and assistant capability set.
**Mitigation:** ask early and plainly; it is one question with a very large answer. If no shared storage exists, the honest response is to **drop search and the assistant from scope** and build a timetable and substitution system that works, rather than to invent a storage layer nobody asked for.
**Residual:** 🟠 Scope reduction, not technical failure.
**Refs:** `CL-NAS-001` `CL-NAS-002` `CL-SRV-014` `CL-GEN-015` `CL-SEARCH-014`

#### R-20 🟠 12 — Downtime during an examination period
**OPEN — ANSWER REQUIRED** · Owner: School administration

During exams every room, display and staff member is committed. A deployment, a restart or an incident in that window is unacceptable — and it is also when the system would be under least pressure to be maintained. `L3` — exams recur every term. `S3` — serious disruption and loss of trust.
**Mitigation:** publish a no-change calendar covering examination periods **before** any deployment; freeze automated updates and reboots in that window; ensure the system is in a known-good state before exams begin, not during.
**Residual:** 🟠 A coordination problem, not a technical one.
**Refs:** `CL-OPS-020` `CL-OTA-018` `CL-TIME-018` `CL-TIME-013`

### 2.5 Security and Privacy

#### R-21 🔴 20 — No staff identity directory, so nothing is attributable
**OPEN — ANSWER REQUIRED** · Owner: School administration

The whole authorisation model — who may publish a timetable, who may approve a substitution, whose session is on a board, who requested a delivery — depends on knowing **which named teacher** did what. If no directory exists and none may be created, the audit trail is fiction. `L3` — many Indian schools have no formal staff identity system. `S5` — the audit trail underpins every consequential control.
**Mitigation:** ask early whether a directory exists, whether CloudLight may create and own a minimal one, and who is authoritative. If the answer is no, the consequence is explicit: shared accounts for everything, and ⛔ **no audit trail worth the name**. That must be stated to the school, not discovered later.
**Residual:** 🟠 Accuracy responsibility moves to a human roster we maintain; drift becomes a real risk.
**Refs:** `CL-AUTH-001` `CL-AUTH-002` `CL-AUTH-004` `CL-DATA-003` `CL-DATA-004` `CL-SEC-019`

#### R-22 🔴 16 — A shared six-digit code becomes identity in practice
**OPEN — APPROVAL REQUIRED** · Owner: School administration

A code designed to authorise one action is, under classroom time pressure, the credential people actually use. Within weeks it becomes the room's password — written on a board, passed between staff, photographed by a student. `L4` — social pressure guarantees convergence on the shortcut. `S4` — actions become unattributable, undoing R-21's mitigation.
**Mitigation:** a code may authorise **one action** or act as a **second factor** — never authenticate a person; short-lived, single-use, rate-limited, absent from logs and URLs, invalidated on substitution; ⛔ no per-classroom shared code; individual accounts for anything consequential.
**Residual:** 🟠 Cannot be eliminated by design — only by school policy and by choosing actions that tolerate it.
**Refs:** `CL-CODE-002` `CL-CODE-003` `CL-CODE-005` `CL-CODE-011` `CL-CODE-015` `CL-CODE-016`

#### R-23 🔴 16 — An unattended board holds a live teacher session
**OPEN — ANSWER REQUIRED** · Owner: School administration + IT

A teacher signs in, walks out, and their authenticated session sits on a wall screen in an empty or student-occupied room. Anything they can do is available to whoever walks up to it. `L4` — teachers leave rooms. `S4` — privilege escalation by proximity.
**Mitigation:** sessions expire at the period change; the signed-out state shows PUBLIC-class content only — no staff surface, no names, no confidential material; prefer re-authentication at the display over a long-lived session; codes expire on room emptiness or substitution.
**Residual:** 🟠 A window always exists between sign-in and sign-out. The signed-out state's content is the real control.
**Refs:** `CL-AUTH-020` `CL-AUTH-021` `CL-SEC-001` `CL-SEC-002` `CL-DASH-002` `CL-SEC-010`

#### R-24 🔴 16 — The SHANA task channel is unauthenticated
**OPEN — APPROVAL REQUIRED** · Owner: Robot custodian + School IT

If the channel between CloudLight and SHANA carries no cryptographic identity, anything on the school network can submit a delivery request, and CloudLight cannot prove to the robot that a request came from this school's instance. `L3` — depends on an interface nobody has agreed. `S5` — an unauthorised party triggers physical robot behaviour in a school.
**Mitigation:** mutual authentication before any integration is built; requests persisted and audit-complete **before** transmission; ⛔ if no authenticated channel can be agreed, **there is no integration** — build CloudLight without SHANA and add it last, if ever.
**Residual:** 🟡 Contingent on a decision nobody has made yet.
**Refs:** `CL-SHANA-015` `CL-SHANA-001` `CL-SEC-016` `CL-SHANA-017`

#### R-25 🟠 12 — A replayed request creates a duplicate physical delivery task
**MITIGATION PLANNED** · Owner: Internal

A retried connection, a double tap, or a captured-and-replayed request causes the same delivery to be created twice. The result is a robot carrying one item on two tasks, or a room receiving an item it did not expect. `L3` — retries are normal engineering. `S4` — a physical, visible, duplicated action.
**Mitigation:** an idempotency key on every request; server-side duplicate suppression; the robot deduplicating on its side as well; ⛔ never rely on the client not double-sending.
**Residual:** 🟠 Mutually authenticated is not non-replayable; a window still exists.
**Refs:** `CL-SEC-016` `CL-SHANA-018` `CL-SHANA-021`

#### R-26 🔴 16 — An unsigned update is accepted from an untrusted LAN
**MITIGATION PLANNED** · Owner: Repository owner + School IT

On an untrusted local network, anyone who can write to a share or run code on one board can replace a release file. If a board installs whatever it is handed, a hostile build reaches every classroom. `L3` — the LAN is assumed untrusted and probably is. `S5` — code execution across the whole estate.
**Mitigation:** client-side signature verification; a well-formed but unsigned build is refused; signing key custody documented, rotation rehearsed, and the school able to revoke vendor-signed builds; version recorded on every display.
**Residual:** 🟠 Key custody is an operational discipline, not a code property. A single compromised key is indistinguishable from a legitimate deployment.
**Refs:** `CL-OTA-004` `CL-OTA-005` `CL-OTA-017` `CL-SEC-013`

#### R-27 🟠 12 — Secrets leak into the repository
**MITIGATION PLANNED** · Owner: Repository owner

NAS credentials, signing keys, certificate material and API keys end up committed to Git, in a URL, in a log, or in browser storage. Git history is permanent. `L3` — an extremely common accident. `S4` — credentials exposed, possibly staff data accessible.
**Mitigation:** a written secrets policy established before any code exists; ⛔ no real credential in any questionnaire column, document or commit; runtime secrets held outside the repository; repository scanning in place before merge; rotation rehearsed so it is known to work.
**Residual:** 🟠 Requires discipline from every contributor, forever.
**Refs:** `CL-GIT-014` `CL-GIT-015` `CL-SEC-006` `CL-CODE-014`

#### R-28 🔴 16 — Confidential staff material sits beside teaching resources
**OPEN — APPROVAL REQUIRED** · Owner: School administration

Appraisal forms, salary records, medical or disciplinary notes stored on the same shares as worksheets. A resource index built across those shares makes confidential material **findable** — a different and worse exposure than it merely being stored there. `L3` — plausible where storage is a general shared drive. `S5` — sensitive personal data exposed to colleagues and students.
**Mitigation:** ask this **before** indexing anything; index an explicitly approved folder list, never a whole share; exclude confidential and restricted classes by default; examination papers and answer keys must never appear in results at all; ⛔ no parallel permission model — enforce the storage's own permissions.
**Residual:** 🟠 The index is a copy of restricted content that outlives the permission check on the file. That is a new thing to protect.
**Refs:** `CL-NAS-011` `CL-NAS-012` `CL-NAS-013` `CL-NAS-008` `CL-SEARCH-005`

#### R-29 🟠 12 — Search returns files the user cannot open
**MITIGATION PLANNED** · Owner: Internal

Permission is evaluated when the index is built, but a file's permission can change later. A search result then advertises a filename, title and path the teacher is no longer entitled to open. The metadata itself is the leak. `L4` — permission drift is normal over a school year. `S3` — disclosure of existence and title without content.
**Mitigation:** permission re-verified **at open time**, not only at search time; the index stored at the entitlement level and filtered at query time; index entries deleted when the source is removed; a deletion takes effect promptly, not at the next rebuild.
**Residual:** 🟠 A race between a permission change and an open always exists; keep the window short.
**Refs:** `CL-SEARCH-002` `CL-SEARCH-003` `CL-DATA-020` `CL-SEC-007` `CL-SEARCH-012`

### 2.6 AI and Data Protection

#### R-30 🔴 16 — School data sent to an external AI service
**OPEN — APPROVAL REQUIRED** · Owner: School administration

The convenient implementation of an assistant calls a hosted model. Timetable data, substitution records, staff availability and retrieved document text leave school infrastructure and enter a third party's systems. `L3` — the path of least resistance, reached for convenience. `S5` — a data-protection breach that cannot be undone.
**Mitigation:** ⛔ external inference is **not assumed permitted**; local inference preferred; if external is permitted it must be explicitly configured not to retain prompts or completions; ask which half may leave — the question only, or document text as well; ⚠️ no student-identifiable data in the assistant at all until answered in writing.
**Residual:** 🟠 If refused, the assistant is dropped rather than weakened. That is the intended outcome.
**Refs:** `CL-AI-001` `CL-AI-002` `CL-AI-010` `CL-AI-018` `CL-SEC-011` `CL-DEC-010` `CL-DEC-011`

#### R-31 🟠 12 — Prompt injection through an indexed document
**MITIGATION PLANNED** · Owner: Internal

A document in the corpus contains text addressed to the assistant — asking it to reveal other files, ignore its rules, or surface a hidden folder. The assistant retrieves it as ordinary content and may act on it. `L3` — plausible if anyone can upload to a shared folder. `S3` — information disclosure and misleading answers; **not** actuation, because the assistant never acts.
**Mitigation:** retrieved content is treated strictly as **data, never as instructions**; the assistant has no capability to grant access, publish, or command anything; deterministic validation sits between the model and every output; where no school document is meant to be obeyed as instruction, that is recorded explicitly.
**Residual:** 🟠 There is no complete defence in a retrieval-augmented system. The blast radius is reduced; the class of attack is not eliminated.
**Refs:** `CL-AI-006` `CL-SEC-009` `CL-AI-007` `CL-AI-024`

#### R-32 🟠 12 — A confident wrong answer in a teacher's voice
**MITIGATION PLANNED** · Owner: Internal

The assistant produces a plausible, well-formatted, wrong answer. A wrong timetable is obviously wrong; a wrong answer in a teacher's voice is not, and is acted upon. `L4` — the failure mode generative systems actually have. `S3` — teacher time wasted, wrong material used.
**Mitigation:** "I don't know" is a first-class, tested outcome; every answer carries a clickable citation to the source; permission is re-checked when the citation is followed; answers are visibly marked as AI-generated; the assistant is built **last**, after permission-correct search works without it.
**Residual:** 🟠 Mitigation reduces impact; it does not make the model reliable.
**Refs:** `CL-AI-005` `CL-AI-004` `CL-AI-016` `CL-AI-017` `CL-SEARCH-021`

### 2.7 Operations and Support

#### R-33 🟠 12 — Nobody notices a display is broken
**OPEN — ANSWER REQUIRED** · Owner: School IT + Facilities

A display has shown an error, or yesterday's timetable, or nothing, since Monday. Nobody knows, because a classroom screen is silent when it fails. `L4` — certainly true today. `S3` — a room is without its information for days.
**Mitigation:** boards report last-seen time and version, and the fleet view lists quiet boards; a visible report-a-problem control usable from a teacher's own device; ⚠️ every alert must reach a channel staff actually read — that channel must be **named**, not assumed.
**Residual:** 🟠 The last mile is organisational.
**Refs:** `CL-DASH-020` `CL-OPS-007` `CL-OPS-008` `CL-OPS-009` `CL-REACT-020`

#### R-34 🔴 16 — The one person who understands the system leaves
**MITIGATION PLANNED** · Owner: School administration

Personal knowledge, not documented knowledge, is what keeps the system running. One absence leaves an unmaintained system holding staff and student data with no one able to restore it. `L3` — schools have few technical staff and turnover is real. `S5` — data becomes unprotectable and unrecoverable.
**Mitigation:** written runbooks a non-technical person can follow; a **rehearsed** restore, not a documented one; a named deputy with the same access; a named school-side owner from day one, not at handover; ⚠️ restoration time by a non-technical member of staff is a **measured** question, not an assumption.
**Residual:** 🟠 Documentation is always weaker than the person who wrote it.
**Refs:** `CL-OPS-006` `CL-OPS-015` `CL-SRV-011` `CL-ADMIN-007` `CL-ADMIN-008` `CL-OPS-022`

#### R-35 🟠 12 — The project team leaves
**MITIGATION PLANNED** · Owner: School administration + Internal

An external team builds a local-first system with no support arrangement, then leaves. The school is left with a system nobody can maintain and no budget to replace anyone. `L3` — the normal end of a project. `S4` — slow, certain decay.
**Mitigation:** ownership assigned **before** the build, not at handover; documentation as a deliverable, not an afterthought; architecture decisions recorded with reversal conditions; year-two and year-three maintenance funding identified early; every reversible decision made reversible, every irreversible one justified in writing.
**Residual:** 🟠 A programme risk only mitigation can reduce.
**Refs:** `CL-BUD-010` `CL-BUD-020` `CL-OPS-022` `CL-GIT-011` `CL-GIT-012`

#### R-36 🟠 12 — Backups exist and have never been restored
**OPEN — ANSWER REQUIRED** · Owner: School IT

A backup regime looks healthy on paper and has never been tested. The first test happens during the outage it was meant to prevent. `L4` — untested backup is the norm everywhere. `S4` — the recovery capability is fictional.
**Mitigation:** ask whether any school backup has **ever** been restored, and by whom; assume the answer is no until proven otherwise; schedule the first test restore and record its outcome and duration; backups off-host, encrypted, with a named holder who is not the person who made them.
**Residual:** 🟠 Testing establishes capability, not permanence.
**Refs:** `CL-OPS-014` `CL-OPS-012` `CL-OPS-013` `CL-DATA-012` `CL-DATA-008`

#### R-37 🟠 8 — Nobody can restart the server
**OPEN — ANSWER REQUIRED** · Owner: Facilities + School IT

After an unplanned shutdown the service stays down until the next working day, because the person who knows how to restart it is absent and the procedure is not written down. `L3` — follows from R-09 and R-34. `S3` — downtime extended from minutes to a day.
**Mitigation:** a one-screen restart procedure needing no console and no technical vocabulary; the person who performs it is named and reachable during school hours; auto-start after power returns.
**Residual:** 🟠 Unrecoverable hardware still needs a human.
**Refs:** `CL-SRV-011` `CL-OPS-005` `CL-OPS-002`

### 2.8 Programme and Scope

#### R-38 🟠 12 — The budget covers nothing
**OPEN — ANSWER REQUIRED** · Owner: School administration

No money for a server, storage, cabling, replacement boards or maintenance. The design assumes hardware the school does not own and may not buy. `L3` — common. `S4` — removes the hosting, storage and rollout options simultaneously.
**Mitigation:** ask about budget before designing against hardware; prefer running on a machine the school already owns; if there is no budget, the honest plan is a smaller system on existing hardware — and the year-two maintenance question must be asked anyway.
**Residual:** 🟠 A zero budget is a scope constraint, not a failure.
**Refs:** `CL-BUD-001` `CL-BUD-002` `CL-BUD-007` `CL-BUD-010` `CL-BUD-018`

#### R-39 🟠 12 — Scope grows faster than capability
**MITIGATION PLANNED** · Owner: Internal + School administration

The aspiration includes timetable, substitution, OCR, resource search, an AI assistant, robot integration, kiosk management and updates. Each is individually reasonable; together they exceed what a small team can deliver to a school's standard. `L4` — enthusiasm is the default state of a new project. `S3` — an unfinished system is worse than a finished smaller one.
**Mitigation:** a signed must-have list and an explicit deferrable list, recorded **before** building; a staged build where each stage is independently useful — timetable and substitutions first, then storage permissions, then permission-aware search, then the assistant; ⛔ the assistant is built last or not at all.
**Residual:** 🟠 Requires someone with authority to say no.
**Refs:** `CL-GEN-021` `CL-DEC-017` `CL-DEC-008` `CL-DEC-002`

#### R-40 🟠 12 — The school rejects the project
**OPEN — ANSWER REQUIRED** · Owner: Principal

After seeing it, the school decides the platform is not wanted, not permitted, or not worth the disruption. `L3` — a real and legitimate outcome. `S3` — the programme ends; nothing else is harmed.
**Mitigation:** ⚠️ **this is not a failure.** A school that declines a wall-display system and keeps its printed timetable has lost nothing. Ask what success means **before** building, and honour the answer even when it shrinks the project.
**Residual:** 🟡 Terminal but not damaging.
**Refs:** `CL-GEN-005` `CL-GEN-006` `CL-EXPO-013` `CL-DEC-005` `CL-DEC-019`

#### R-41 🟠 12 — The pilot fails
**OPEN — ANSWER REQUIRED** · Owner: School administration + Internal

A pilot runs in a few classrooms and produces a decision: abandon, redesign or extend. The likeliest failure is not technical — it is that teachers do not use it, because it added a step rather than removing one. `L3` — most pilots fail in some form. `S3` — but a pilot is explicitly a learning exercise.
**Mitigation:** agree pass/fail criteria in advance, including acceptable and unacceptable failure rates; measure real daily reliability, not demonstration reliability; decide in advance what "abandoned" means and who calls it; keep the pilot small enough that abandoning it costs little.
**Residual:** 🟠 Only a real pilot retires this risk.
**Refs:** `CL-EXPO-014` `CL-EXPO-016` `CL-EXPO-017` `CL-DEC-006` `CL-DEC-007`

#### R-42 🟠 8 — Teachers ignore it
**OPEN — ANSWER REQUIRED** · Owner: Teachers

The system is accurate, available and correct, and teachers keep using paper, because it added a step rather than removing one. `L3` — the default fate of a new institutional system. `S3` — adoption failure makes the project pointless even if it works.
**Mitigation:** ask which single operational problem would make this worth funding, and build against that answer; ask whether a substitution board that is *occasionally* out of date is acceptable — that answer sets the entire accuracy target; make the first screen the thing a teacher wants, not the thing the architecture produced.
**Residual:** 🟠 Cannot be engineered away; only earned.
**Refs:** `CL-GEN-005` `CL-GEN-011` `CL-DASH-004` `CL-SEARCH-001` `CL-AUTH-012`

### 2.9 SHANA Integration

#### R-43 🟠 12 — The SHANA interface is never agreed
**OPEN — ANSWER REQUIRED** · Owner: Robot custodian

Which SHANA interface exists, and which is permitted, is unknown. Until it is, the request contract cannot be finalised, authenticated or tested. `L3` — interface agreements stall. `S3` — affects one feature, not the platform.
**Mitigation:** build the entire platform **without** SHANA and add it last; if no interface is agreed, ship without it; ⛔ CloudLight must operate fully when SHANA is offline — a hard requirement, not a degraded mode.
**Residual:** 🟡 Feature-level. The platform stands without it.
**Refs:** `CL-SHANA-001` `CL-SHANA-002` `CL-SHANA-019` `CL-SHANA-024` `CL-DASH-022`

#### R-44 🟠 8 — CloudLight becomes coupled to SHANA availability
**MITIGATION PLANNED** · Owner: Internal

A timetable, substitution or resource view that waits on the robot subsystem has inherited a dependency it must never have. A robot fault becomes a classroom fault. `L2` — easy to introduce accidentally. `S4` — a robot problem takes down a wall display in every room.
**Mitigation:** the timetable is a wall sign and must behave like one; no CloudLight feature may block, slow or fail because a robot is unreachable; the robot panel shows a clearly unavailable state, or is hidden — it is never load-bearing.
**Residual:** 🟡 A design invariant; retires only with implementation.
**Refs:** `CL-DASH-022` `CL-SHANA-024` `CL-SEC-017`

### 2.10 Permission and Governance

#### R-45 🔴 20 — No named approver, so nothing may lawfully be stored
**OPEN — APPROVAL REQUIRED** · Owner: Principal

Nobody has been identified who can authorise storing student and staff personal data, identifiable substitution records, or timetable data on school infrastructure. Without that person there is no lawful basis and nothing may be built. `L3` — the question is often simply never asked. `S5` — the project cannot start at all.
**Mitigation:** **ask this first.** One question, and it gates everything. Obtain written approval naming the accountable **role**, not just a person. Record the motor-command boundary, the data-handling basis, retention rules and pilot scope as separate signed approvals rather than one blanket permission.
**Residual:** 🟡 Binary. No approver means no project.
**Refs:** `CL-GEN-001` `CL-GEN-003` `CL-GEN-004` `CL-SRV-005` `CL-SRV-006` `CL-DEC-001` `CL-DEC-021`

#### R-46 🟠 12 — A requested feature would violate an invariant
**MONITOR** · Owner: Internal + School administration

A legitimate stakeholder asks for something that breaks an invariant — student marks on a public display, an assistant that publishes the timetable, an auto-selected substitute. Pressure to comply is strong and the request is rarely malicious. `L4` — feature requests arrive continuously. `S4` — the invariants are the safety property.
**Mitigation:** the invariants are written down and short. ⛔ *Nothing is shown on a student-facing screen that has not been confirmed; no number is shown the system does not currently believe; the timetable is never a dependant; the assistant never acts.* If a request cannot be met within those, say so in writing rather than quietly bending one.
**Residual:** 🟠 Political, not technical.
**Refs:** `CL-DASH-003` `CL-AI-007` `CL-SUB-012` `CL-DEC-017` `CL-GEN-014`

---

## 3. Risks that cannot be mitigated

Recorded so nobody spends effort on them, and so nobody later claims the design solved them.

| # | Risk | Why it cannot be mitigated |
|---|---|---|
| U-1 | **A child reads whatever is on a shared screen** | A wall display is visible to whoever stands in front of it. The only control is *what the signed-out state shows*. A camera in the room can photograph any screen, including one showing a staff name. ⛔ Not eliminable in a room with a door |
| U-2 | **A six-digit code is guessable in principle** | A million possibilities is not an authentication factor. Rate limiting, single use and short expiry raise the cost; they do not make it identity. The code must never be the thing that matters |
| U-3 | **A determined user with legitimate access can read a document they are entitled to** | No access control prevents this. The design goal is that they *were* entitled, and that the access is recorded |
| U-4 | **Everything lives on one host** | Database, index, audit log, secrets, signing key — one room. Concentration is the direct price of local-first. Only time, restoration and honest acceptance address it |
| U-5 | **An administrator with full host control defeats software-only log tampering** | Tamper-evident chaining and an off-host copy raise the cost. Someone who owns the machine owns the evidence. Only an external party could close this, and there is none |
| U-6 | **Prompt injection has no complete defence** in a retrieval-augmented system | Retrieved content is untrusted input to a model that processes instructions. Giving the assistant no power to act reduces the blast radius; it does not eliminate the class of attack |
| U-7 | **A network outage during a live demonstration** | Survived, never prevented. The only mitigation is an honest degraded screen and a recorded fallback |
| U-8 | **A teacher will forget which button does what** | Mitigated by design clarity, not eliminated. The substitute teacher using it alone on day one is the permanent residual |

⚠️ **Eight residual exposures, none eliminable.** That is the honest state of a
school-grade local-first platform in 2026. A risk register claiming otherwise
would be worth less than none.

---

## 4. Risks that are SHANA's, not CloudLight's

Out of scope here, named so nobody double-counts them. Full detail in
[`../safety/risk_register.md`](../safety/risk_register.md).

| Concern | Owner | Note |
|---|---|---|
| Stair riser height vs the robot's documented limit | SHANA — `R-01`, `R-11` | Physical measurement; nothing to do with a wall display |
| Localising onto the wrong floor's map | SHANA — `R-03` | Silent and dangerous; entirely inside the robot's sensor fusion |
| Robot disabled mid-mission with no watchdog | SHANA — `R-07` | A CloudLight outage does **not** stop the robot, by design |
| Lift integration is legally impossible | SHANA — `R-02`, `R-15` | Statutory prohibition; CloudLight has no standing in it |
| Robot stranded in a stalled lift | SHANA — `R-09` | The escort protocol is the defence |
| Battery thermal event | SHANA — `R-08` | Documented manufacturer limits |
| Student injury from a 15 kg machine with 12 moving legs | SHANA — `R-05` | Named adult spotter, every stage, without exception |
| Secondary development not authorised | SHANA — `R-06` | Technical capability and permission are different questions |
| Payload drop; unstable load | SHANA — `R-14` | Carry limit, ratchet and strap |
| Corridor congestion and human intervention | SHANA — `../operations/` | Physical world, not a database |

⚠️ **The boundary is asymmetric and deliberate.** CloudLight can *cause* SHANA
risk only by sending a malformed, unauthorised or duplicated request — which is
why R-24, R-25 and R-28 sit in this register. **CloudLight can never cause
navigation, localisation, battery or physical-safety risk, because it has no
capability to express one.** That is an absence of capability, not a promise.

---

## 5. Outstanding approvals

**None of these may be assumed. None has been requested.** Full text in
[`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md),
where every `Status` is `UNANSWERED`.

| # | Approval | From | Questionnaire | Status | Blocks |
|---|---|---|---|---|---|
| A-01 | **Named accountable approver** for storing personal data on school infrastructure | Principal | `CL-GEN-001` `CL-DEC-001` | ⛔ NOT REQUESTED | Everything |
| A-02 | **Persistent always-on service** permitted on school premises | Principal | `CL-GEN-002` `CL-SRV-001` | ⛔ NOT REQUESTED | The whole platform |
| A-03 | **Written acknowledgement** that CloudLight never sends motor commands | Principal + robot custodian | `CL-GEN-003` `CL-SHANA-002` | ⛔ NOT REQUESTED | SHANA integration |
| A-04 | **Lawful basis** for identifiable staff availability and substitution records | Principal | `CL-GEN-004` `CL-SRV-005` | ⛔ NOT REQUESTED | Substitution feature |
| A-05 | **Server room allocation** and named sysadmin account holder | Principal + Facilities | `CL-SRV-002` `CL-SRV-004` | ⛔ NOT REQUESTED | All software |
| A-06 | **Network rules** permitting classroom devices to reach the server subnet | IT | `CL-NET-003` `CL-NET-014` | ⛔ NOT REQUESTED | Every display |
| A-07 | **DNS/DHCP record approval** for the CloudLight host | IT / Administration | `CL-NET-006` `CL-NET-012` | ⛔ NOT REQUESTED | Stable addressing |
| A-08 | ⚠️ **Written acceptance of a flat, unsegmented LAN** | IT + Principal | `CL-NET-001` `CL-SEC-023` | ⛔ NOT REQUESTED | Holding staff data at all |
| A-09 | **Local administrator rights** on classroom display PCs | IT | `CL-WIN-003` `CL-OTA-010` | ⛔ NOT REQUESTED | Self-update, self-repair |
| A-10 | **Kiosk / Assigned Access trial** on one non-critical board | IT | `CL-WIN-006` `CL-WIN-007` | ⛔ NOT REQUESTED | Display deployment |
| A-11 | **Update deferral window** covering lessons and examinations | IT + Administration | `CL-WIN-013` `CL-OTA-018` `CL-OPS-020` | ⛔ NOT REQUESTED | Update model |
| A-12 | **Approved folder list** permitted for indexing | Principal + Library | `CL-NAS-008` `CL-NAS-012` | ⛔ NOT REQUESTED | Search, assistant |
| A-13 | ⚖️ **Permission to copy file contents into an index** | Principal | `CL-NAS-015` `CL-SEARCH-005` | ⛔ NOT REQUESTED | OCR and full-text search |
| A-14 | **Per-room permission to photograph** the substitution board or sheet | Administration | `CL-SUB-006` `CL-SUB-007` | ⛔ NOT REQUESTED | Substitution ingestion |
| A-15 | ⚖️ **Whether any AI service may receive school content** | Principal | `CL-AI-001` `CL-AI-010` `CL-DEC-010` | ⛔ NOT REQUESTED | The assistant |
| A-16 | **Release signing key custody** and revocation authority | IT + repository owner | `CL-OTA-005` `CL-SEC-013` | ⛔ NOT REQUESTED | Any deployment |
| A-17 | **Mutual authentication** for the SHANA task channel | Robot custodian + IT | `CL-SHANA-015` `CL-SEC-016` | ⛔ NOT REQUESTED | SHANA integration |
| A-18 | **Pilot permission**, scope, rooms and duration | Principal | `CL-EXPO-013` `CL-EXPO-014` `CL-GEN-021` | ⛔ NOT REQUESTED | Anything user-facing |
| A-19 | **Backup location and named holder**, off-host | Principal + IT | `CL-SRV-016` `CL-DATA-008` `CL-DATA-010` | ⛔ NOT REQUESTED | Recovery |
| A-20 | **Named school-side owner** after handover | Principal | `CL-OPS-022` `CL-ADMIN-008` `CL-BUD-010` | ⛔ NOT REQUESTED | Year two onward |

⚠️ **A-01 gates everything.** Until a named person can authorise this project,
no other question is worth asking.

---

## 6. How these risks retire

| Risk group | Retires when |
|---|---|
| R-01, R-05, R-09 | A host is authorised **and** a restore has been rehearsed |
| R-06, R-07, R-08 | Reachability, isolation setting and segmentation are **measured**, not assumed |
| R-11 – R-14 | A `winver` photograph exists **and** a kiosk trial has run on a non-critical board |
| R-15 – R-18 | A named approver exists, validation rejects contradictions, and a human confirms every publication |
| R-19, R-28 | The school answers what storage exists and which folders may be indexed |
| R-20 | A no-change examination calendar is agreed before any deployment |
| R-21 – R-23 | An identity decision is recorded — or shared access is accepted **in writing** |
| R-24, R-25 | An authenticated, non-replayable channel is agreed — or integration is dropped |
| R-26, R-27 | A secrets policy and key custody model are documented and rehearsed |
| R-30 – R-32 | A written AI policy; the assistant built last and citation-bound |
| R-33, R-34, R-36, R-37 | A rehearsal has happened **and the runbook was followed by someone else** |
| R-38 – R-42 | A signed scope, a named owner, and a real pilot with pre-agreed pass criteria |
| R-43, R-44 | An agreed SHANA interface — or the platform ships without it |
| R-45, R-46 | A named approver signs the boundary and the invariants |

⚠️ **No risk in this register retires through further research.** Every one
requires a measurement, a decision, or a signature. That is the correct state for
Phase 0, and the reason this register will look uncomfortable for as long as it
exists.