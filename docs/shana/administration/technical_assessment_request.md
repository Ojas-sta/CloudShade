# SHANA — Technical Assessment Request

**For issue to:** Lift service provider · IT department · CCTV administrator ·
Robot owner
**Date:** 2026-10-10
**Status:** DRAFT. **Not yet sent. No external party has been contacted.**

---

## Before you send this

- [ ] Repository owner has approved issuing this document
- [ ] The correct named contacts have been identified
- [ ] No school network, CCTV, or security details are included in the covering
      message
- [ ] The lift questions are routed **through the school's authorised service
      provider**, not to the manufacturer directly, unless the provider directs you

⚠️ **Do not offer, propose, or build any physical modification to a lift.** The
questions below ask what the manufacturer *permits*. A "no" is a legitimate and
completely acceptable answer, and is the answer we expect to design around.

---

# Part 1 — Lift service provider

**To:** Authorised lift maintenance contractor
**Re:** Two lifts serving a school. Written technical assessment request.

## Context

A school project proposes an indoor delivery robot. The robot is a quadruped
approximately **700 × 310 mm** standing, weighing **approximately 15 kg**. It has
**no lifting or manipulation capability** and would travel only as a passenger.

**We are not proposing any modification to your equipment.** We are asking what
is permitted, so that we can design within the answer.

## Section A — Identification and physical survey

1. Please confirm the **manufacturer and model** of each lift, and the Indian
   standard to which each was built and certified.
   *(Reference note: IS 14447 and IS 6914 are not lift standards — we believe the
   applicable family is IS 17900. Please confirm the standard actually applied
   to your installation.)*
2. Please provide **car internal dimensions (W × D × H)** and the **door clear
   opening width** for each lift.
3. Please provide the **rated car load** and typical observed occupancy.
4. Please confirm whether **car position, direction of travel, and door state**
   are available as a readable output by any existing documented means.
5. Please confirm whether **firefighter's service / Phase I fire recall** is
   fitted to either lift.

## Section B — Interfaces

6. Does the manufacturer offer **any supported interface** allowing a
   third-party device or an autonomous robot to place a **hall call** or a
   **destination call**?
7. If a building-management or monitoring interface exists, does it permit
   **outbound control**, or is it **read-only**?
8. Does a **remote-control or mobile application** exist for these lifts? If so,
   is it **restricted to maintenance staff**, and is it intended as a
   third-party integration path?
9. Is there a **documented SDK, protocol, or integration guide** available to
   third parties?
10. Is any **approved external call interface** available to third-party devices?

## Section C — EyeRIS

> Reference: the manufacturer is understood to offer an "EyeRIS" contactless
> call-booking system that operates via a QR code at the landing operating panel.

11. Is EyeRIS **available for retrofit** onto lifts installed earlier, or is it
    limited to lifts originally specified with it?
12. Is the EyeRIS QR element at the landing panel a **passive printed label**, or
    an **active electronic component** requiring wiring or power?
13. Can EyeRIS be **provisioned to a non-human client** — for example an
    unattended device carried by a robot — rather than only to a person's phone?
14. If a non-human client is possible, what are the **authentication and
    authorisation requirements**?

## Section D — Physical button actuation

> ⚠️ **We are not proposing this and do not expect the answer to be yes.** We ask
> because it is a common academic approach and we need to record the
> manufacturer's written position rather than assume it is prohibited.

15. **Does the manufacturer and the building owner explicitly permit a
    purpose-built, externally mounted actuator to press the existing lift
    buttons?**
16. If not permitted, we will treat it as **out of scope permanently**. Please
    confirm that in writing.

## Section E — Operation, safety and assurance

17. Would the manufacturer permit a **small autonomous robot to travel as a
    passenger with a named human escort** operating the panel, as a supervised
    trial?
18. Can the lift's door and passenger-sensor systems accommodate a 15 kg
    quadruped entering and leaving?
19. What is the approved **emergency or fault workflow** if a lift stops between
    floors with a passenger inside?
20. Would robot involvement affect the **maintenance contract, warranty, insurance
    or statutory certification**?
21. Please confirm your position on attaching **any device to, over, or into the
    car or landing operating panel**, or connecting to lift wiring or safety
    circuits.

## Section F — Alternatives

If no sanctioned integration exists, which of these does the manufacturer consider
acceptable?

- (a) Human-assisted operation — a person presses the buttons, the robot rides as a passenger
- (b) Staff-controlled lift calls, on request
- (c) A future manufacturer-supported integration path
- (d) Avoiding vertical transitions in the initial project entirely

**We will design around a "no".** Our default assumption is that a human
escort is the permanent arrangement, which is what comparable delivery robots do.

## Section G — Safety observation

21. Please advise whether the car door dwell time is sufficient for a quadruped
    to enter and exit.
22. Please advise on any lobby space requirement for a robot waiting for a lift.

---

# Part 2 — IT department

**To:** School IT / Network Administrator
**Re:** Read-only network assessment for an indoor robot project

⚠️ **No scanning, probing, or connecting has been or will be performed.** This is
a request for existing documentation and policy information.

## Section A — Existing infrastructure

1. Please provide the **existing wired LAN topology**, including switch model and
   any VLAN structure.
2. Are **VLANs or client isolation** enabled that would prevent device-to-device
   communication on the school LAN?
3. Do firewall rules permit **UDP multicast** on internal segments? *(Multicast
   discovery is required for the robot's communication protocol.)*
4. Please provide the **Wi-Fi access point inventory** with floor and block
   coverage, and a per-floor coverage map if one exists.

## Section B — The robot network

> Reference: the robot's manufacturer specifies development on an isolated
> `192.168.123.0/24` subnet, with the robot at one address and the development
> host at another. **The robot connection must be wired Ethernet.**

5. Is it permissible to create a **dedicated network segment** for the robot,
   isolated from the smartboard and general school LAN?
6. Is a **static IP assignment** possible for a server host on that segment?
7. Can a **dedicated Ethernet NIC** be added to a server host for this purpose?
8. Could the robot be connected by **direct cable** to a server host, with no
   change to the school network at all? **This is our preferred option** and we
   would like to know whether it is acceptable.

## Section C — Local server

9. Can a **locally hosted server** be reachable by teacher devices on the school
   LAN **without internet access**?
10. Are there firewall rules that would block a browser-based tool on a local
    subnet?
11. May a **local DNS name or fixed IP** be assigned for the dashboard?
12. Is a **local-only network with no external exposure** acceptable? *(No
    component of this project will be exposed to a public network.)*
13. Does a **NAS or existing local application server** exist that could host a
    project service?

## Section D — Wireless and coverage

14. **Is there any specific room, corridor, stairwell, hall, or courtyard where
    Wi-Fi does not reach or is unreliable?** *This is the single most important
    answer in this section — it determines whether additional radio technology is
    justified at all.*
15. Are **additional access points** permissible? Who installs and maintains them?
16. Can access points be **powered through approved PoE infrastructure**?
17. Is a **private wireless network** for this project permissible?
18. Is **wireless backhaul or bridging** permissible?
19. Are stairwells and lifts known **wireless dead zones**?
20. What is the school's required **coverage survey method** before any
    additional access point is approved?

## Section E — Resilience and policy

21. Are there **scheduled network outages, maintenance windows, or load-shedding
    patterns** during the school day?
22. What are the school **network acceptable-use policies** applicable to a new
    networked device?
23. Does the school have an **RF or wireless device policy** that would apply to
    any additional radio hardware?
24. What are the requirements for **credential storage and rotation**?
25. ⚠️ Does the school have a **policy prohibiting storage of network credentials
    in source code repositories or documentation**?

---

# Part 3 — CCTV system owner and privacy authority

**To:** CCTV system administrator and School Data Fiduciary
**Re:** Assessment for a proposed anonymous corridor-occupancy feature

⚠️ **No CCTV system has been accessed, scanned, connected to, or tested.** This is
a request for documentation and a policy determination only.

## Section A — System information

1. Please identify the CCTV **brand, model, and management software**.
2. Does the system provide an **ONVIF-conformant interface**? If so, which
   profiles?
3. Do cameras or the recorder provide **RTSP streams**, and is the URL format
   documented?
4. What **authentication** is used — ONVIF Digest, RTSP Basic, or HTTP Digest?
5. Do the **NVR channels re-stream**, or is each camera a separate point-to-point
   stream?
6. Do the streams carry **audio**?

## Section B — Coverage and capability

7. Do cameras cover **all relevant corridor segments** on the routes in question?
8. What are the **resolution, frame rate, and field of view** of those cameras?
9. Are there **glare, low-light, or occlusion** problems in the corridors?
10. Does any camera or the system already emit **anonymous occupancy or
    line-crossing metadata events** without video?
11. Does the school already run **any people-counting analytics**?

## Section C — Policy and permission — the critical section

12. **Does school policy permit any analytics access to CCTV streams?**
13. Who is the **Data Fiduciary** for student data, and have they authorised this
    project in principle?
14. Is a **privacy impact assessment** required before any processing?
15. ⚠️ **Does any available exemption cover corridor-routing analytics?** We
    understand there may be an exemption for educational activity or child safety,
    and that corridor traffic routing may fall outside both. **Please advise.**
16. What **retention** requirements apply to derived occupancy data?
17. Do cameras **record by default**, and may the analytics process streams
    without recording?
18. Is any **debug video capture** permissible, and if so under what authority and
    for how long?

## Section D — Safeguards we propose

For the school's assessment, the following would apply **if** analytics were
ever approved:

- **No facial recognition.** No identity recognition. No biometric identification.
- **No tracking of individuals.** Anonymous corridor states only —
  `CLEAR`, `MODERATE`, `BUSY`, `UNKNOWN`.
- **No raw video stored** by default. Frames processed in-stream and discarded.
- **Audio is never decoded.**
- **No automatic misconduct, discipline, or attendance reporting.** Any incident
  report requires human review and school approval.
- **Confidence thresholds are explicit.** Low confidence or feed failure yields
  `UNKNOWN`, **never** `CLEAR`.
- **CCTV is never the robot's only obstacle-avoidance system.**
- **No real student footage is used during research.**

## Section E — Alternatives

19. Would the school consider a **dedicated mmWave or time-of-flight presence
    sensor** instead of CCTV analytics? *This produces the same
    `CLEAR`/`MODERATE`/`BUSY` answer, involves no camera, no video, and no
    imagery of any person ever formed, at lower cost and lower privacy burden.*
20. Would the school accept **manually reported corridor status** from staff
    during the initial pilot? *This requires no cameras, no sensors, no privacy
    determination, and is the simplest option.*

---

# Part 4 — Robot owner / custodian

**To:** School robot custodian
**Re:** Hardware and configuration identification request

⚠️ **This is an identification request only.** No connection to the robot has
been made and none will be made without written authorisation.

## Section A — Identification

1. Please provide a photograph of the **robot nameplate** and confirm the exact
   model designation.
2. Please provide the **serial number**.
3. Please provide a screenshot of the app's **Device → Service Status** page,
   showing all firmware version numbers.
4. Is the robot the **EDU** variant?

## Section B — Installed options

5. Is the **expansion compute dock** present? If so, which model?
6. Is the **depth camera** fitted?
7. Are **foot force sensors** fitted?
8. Please photograph the **battery label**. *(The battery determines the endurance
   budget and it differs between variants.)*
9. Is a **charging pile or dock** present? How many **spare batteries** exist?

## Section C — Authorisation and support

10. Does the school hold **written authorisation** to use the Unitree secondary
    development interface?
11. Which **Unitree SDK version** is approved for use?
12. Is there a school-approved **Linux host** available for robot development?
13. What **warranty and service arrangements** exist, and who is the local
    service partner?
14. Please provide the **repair and fault history** since purchase.
15. Please provide all **documentation supplied with the robot**.

## Section D — Physical inspection request

16. Where is the robot normally stored, and is that room access-controlled?
17. Is there a mains power point available at the proposed server location?
18. Is a UPS available?

---

## Response handling

Responses to this document become evidence in
`questionnaires/SHANA_Feasibility_QA.md`, in the `Answer` and
`Evidence / Source / Attachment` columns.

⚠️ **Where a response is "unknown", "not permitted", or "no", that is recorded as
the answer and the project designs around it.** A negative response is a valid
and useful outcome, not a failure.

Responses must **not** contain credentials, stream URLs, network topology details
that the school has not approved for circulation, or any personal data.