# SHANA — Authorised CCTV Integration for Anonymous Corridor-Occupancy Estimation

**Status:** Research complete. No camera was connected to, no stream was pulled, no footage was viewed,
no software was installed, no package was downloaded.

**Date of evidence collection:** 2026-10-10. All web evidence reflects that date.

**Companion file:** [`../research/cctv_source_register.md`](../research/cctv_source_register.md) — every URL, source type,
and the specific claim it supports.

---

## Evidence grading

Same scheme used across the Phase 0 and feasibility documents.

| Label | Meaning |
|---|---|
| **VERIFIED** | Read directly from a primary source: an ONVIF specification PDF, an official Gazette notification, a statute, an official board circular, official API docs, or a peer-reviewed paper. |
| **INFERRED** | Follows from verified facts but is not itself stated by a source. |
| **UNVERIFIED** | Claimed by marketing or a secondary source, with no specification or independent test located. |
| **NOT FOUND** | Searched for and not located. Absence of evidence, explicitly not evidence of absence. |
| **CONTRADICTED** | Sources disagree; the disagreement is documented rather than silently resolved. |

---

## The three honest answers, up front

The brief asked for particular care on three points. Here they are first, in plain terms,
before the long evidence tables.

### (a) Is classical OpenCV people detection accurate enough in a cluttered school corridor?

**We could not find any published accuracy figure for this task, and we are not going to invent one.**
See §B.4. What we *can* say with evidence:

- OpenCV itself has demoted HOG and Haar out of the main library. In OpenCV 5 they moved to
  `opencv_contrib`, and the project states in terms: *"Modern deep learning-based detectors are both
  faster and more accurate."* (VERIFIED — OpenCV 5 migration guide.)
- The classical detectors are trained on upright, walking pedestrians at pedestrian scale. A school
  corridor at 3–8 m produces partially-occluded, seated, backlit, and edge-clipped people.
- **There is no benchmark we could find for HOG or Haar on cluttered indoor school corridors.**
  Every number we could verify is from outdoor pedestrian datasets (PASCAL VOC, INRIA Person) or
  from academic crowded-scene datasets. Transferring those to a school corridor is an assumption,
  not a measurement.

So the honest position is: **classical OpenCV is not defensible as the primary occupancy
estimator.** It may be acceptable as a coarse sanity signal layered under a better estimator,
but nobody should promise the school that it works until it has been measured on the school's own
corridor with a consented, non-student test cohort.

### (b) What exactly does Indian law say about CCTV of children?

Three findings, all from primary sources, and the second one changes the project's timeline:

1. **Neither the DPDP Act 2023 nor the DPDP Rules 2025 mentions CCTV, surveillance, biometrics or
   facial recognition anywhere.** We searched the full text of both. The only occurrence of
   "video" is an illustration about live video-based KYC. (VERIFIED — negative finding.)
2. **The operative sections are not yet in force.** Commencement notification G.S.R. 843(E) of
   13 November 2025 brings sections 3–17 of the Act — including the children's-data section 9 —
   into force **eighteen months** after publication, i.e. approximately **13 May 2027**. Rule 12 of
   the DPDP Rules 2025, which contains the children's exemptions, is on the same eighteen-month
   clock. **As of today, 2026-10-10, neither is in force.** (VERIFIED — eGazette G.S.R. 843(E).)
3. **There is, however, a purpose-built exemption for exactly this kind of institution.** DPDP Rules
   2025, Rule 12(1) read with the Fourth Schedule, Part A, entry 3 exempts an educational institution
   from the section 9(1) verifiable-parental-consent duty and the section 9(3) prohibition on
   tracking and behavioural monitoring of children — conditioned on processing being restricted to
   tracking or behavioural monitoring (a) for the educational activities of the institution, or
   (b) **in the interests of safety of children enrolled with the institution**. (VERIFIED.)

**The honest reading:** corridor-occupancy routing is a *safety-adjacent* purpose, not a clearly
"educational activity" purpose. Whether (b) covers "avoid driving a robot down a crowded corridor"
is a judgement call on which a competent Indian data-protection lawyer should opine. **Do not treat
the exemption as settled.** Also note: CBSE clause 4.7.10 already *mandates* CCTV in corridors for
CBSE-affiliated schools, so if this is a CBSE school the corridor footage already exists and the
school is already the Data Fiduciary for it. That cuts both ways.

### (c) Is the whole concept worth the permission burden versus a mmWave presence sensor?

**Our reading of the evidence: no — not as the primary mechanism.** See §D.4 for the full
comparison. The short form:

- The CCTV route requires: written authorisation from the school's Data Fiduciary, a defined purpose
  that survives a proportionality test, credentials for existing security infrastructure, an
  integration that must not degrade the CCTV system's own security posture, and an accuracy
  claim the team cannot currently evidence.
- The mmWave route requires: mounting hardware, a small electrical/DIY task, and no personal data
  at all, because **no image of any person is ever created**.
- For a CLEAR/MODERATE/BUSY decision, a presence/zone-count sensor at a corridor threshold answers
  the operational question adequately, at a fraction of the cost and none of the legal surface.

**The CCTV case is strongest for exactly one configuration:** if the school already runs an
analytics-capable NVR that natively emits an ONVIF occupancy count, and the school is willing to
expose that single integer over its existing authenticated ONVIF channel. In that case the robot
never touches video at all and the permission burden is one line in an existing policy. **That
configuration should be probed before any of this is built.**

---

## 0. The single most important technical finding

**ONVIF Profile M makes metadata streaming MANDATORY and video streaming CONDITIONAL.**

From ONVIF Profile M Specification v1.1, March 2024:

> §7.5.3 Function list for devices — **"Metadata Streaming  Device MANDATORY"**
> §8.2.3 Function list for devices — **"Video Streaming  Device CONDITIONAL"**
> (https://www.onvif.org/wp-content/uploads/2024/04/onvif-profile-m-specification-v1-1.pdf)

This means a Profile M conformant analytics device is permitted to be conformant **while streaming
no video whatsoever**. That is the standards-level basis for the "metadata-only, no video" design.
It is not a workaround; it is the intended architecture of the profile.

### 0.1 The occupancy event exists in the Analytics spec — but is NOT in Profile M

This is the finding most likely to mislead an integrator, so it is stated precisely:

| Artefact | Topic | Data | Status |
|---|---|---|---|
| ONVIF **Analytics Service Spec v26.06**, Appendix A.6 "Occupancy Counting rule" | `tns1:RuleEngine/CountAggregation/OccupancyCounter` | `Count [xs:int]` | **VERIFIED** — defined |
| ONVIF **Profile M Spec v1.1**, §8.15 "Line crossing counter" | `tns1:RuleEngine/CountAggregation/Counter` | `Count [xs:int]` | **VERIFIED** — defined |
| ONVIF **Profile M Spec v1.1** — occupancy counter section | — | — | **VERIFIED NEGATIVE** — the string "occupancy" does not occur anywhere in the document |

A full-text search of Profile M v1.1 for `occupancy` returns **zero matches**. Profile M mandates
only the *line-crossing* counter (`CountAggregation/Counter`), which is a *cumulative flow count*,
not an occupancy level. The occupancy counter is defined in the Analytics Service Specification
appendix as an *optional rule*, not as a profile feature.

**Consequence:** you cannot write "the camera is Profile M conformant, therefore it supports
`OccupancyCounter`." You must call `GetSupportedRules` (Analytics service) and check whether
`tt:OccupancyCounting` is actually offered by that specific device.

The rule as specified (Analytics Spec v26.06, Appendix A.6, verbatim):

> "The occupancy counting rule counts the number of objects inside an enclosure by counting objects
> passing in and out of a set of line segments (barriers). The counter increments when objects pass
> from left to right and decrements when objects pass from right to left."
>
> **TOPIC:** `tns1:RuleEngine/CountAggregation/OccupancyCounter`
> **DATA:** `Count [xs:int]` — "Number of objects counted since last reset."
>
> Parameters: `Segments` (unbounded `tt:Polyline`), `ReportTimeInterval` (optional `xs:duration`),
> `ResetTime` (optional, unbounded `xs:time`, interpreted as localtime).
>
> (https://www.onvif.org/specs/srv/analytics/ONVIF-Analytics-Service-Spec.pdf)

**Read that carefully: it is a differential counter maintained by the device from line crossings.
It is not a perception result we compute.** The device never has to give us video. The entire data
exposure is one 32-bit integer plus a timestamp. This satisfies every hard constraint in the brief
except that we cannot independently verify what the device is counting.

---

## A. ONVIF and RTSP — the integration surface

### A.1 What is ONVIF, and what is the current specification version?

**VERIFIED.** ONVIF is an open industry forum producing standardised interfaces for IP-based physical
security products.

> "ONVIF is an open industry forum that provides and promotes standardized interfaces for effective
> interoperability of IP-based physical security products and services."
> — https://opensecurityandsafetyalliance.org/ (the organisation's own domain, linked from onvif.org)

Conformance is registry-based and profile-scoped:

> "Conformance to profiles is the only way that ensures compatibility between ONVIF conformant
> products; therefore, only registered products with conformance to a profile are considered to be
> ONVIF conformant."
> — https://www.onvif.org/profiles/

**Current versions (VERIFIED, read from the documents themselves):**

| Document | Version | Date | URL |
|---|---|---|---|
| ONVIF Core Specification | **26.06** | June 2026 | `https://www.onvif.org/specs/2606/ONVIF-Core-Spec-v2606.pdf` |
| ONVIF Analytics Service Specification | **26.06** | June 2026 | `https://www.onvif.org/specs/srv/analytics/ONVIF-Analytics-Service-Spec.pdf` |
| ONVIF Streaming Specification | **26.06** | June 2026 | `https://www.onvif.org/specs/stream/ONVIF-Streaming-Spec.pdf` |
| ONVIF Conformance Process Specification | **5.7** | April 2026 | `https://www.onvif.org/wp-content/uploads/2026/04/ONVIF-Conformance-Process-Specification-v5-7.pdf` |
| ONVIF Profiles Conformance Device Test Spec | **26.06** | June 2026 | `https://www.onvif.org/wp-content/uploads/2026/07/ONVIF_Profiles_Conformance_Device_Test_Specification_26.06.pdf` |
| ONVIF Profile M Specification | **1.1** | March 2024 | `https://www.onvif.org/wp-content/uploads/2024/04/onvif-profile-m-specification-v1-1.pdf` |

ONVIF versions by calendar stamp (`YYMM`). Note the **split versioning**: service specifications
are versioned on the ONVIF calendar, while **profiles carry their own slower version line**
(Profile M is still at v1.1 from March 2024 while its base service specs are at 26.06).

**Profile M requires** *"Implementation of ONVIF Network Interface Specifications, version 21.06 or
later"* (Profile M v1.1 §4, VERIFIED).

**Current profiles:** A, C, D, G, M, S, T, and V (Release Candidate). Profile Q is deprecated as of
1 April 2022 (https://www.onvif.org/profiles/, VERIFIED).

### A.2 Which profiles matter, and does ONVIF define an analytics event framework?

**Yes.** The analytics framework is the **ONVIF Analytics Service Specification**
(https://www.onvif.org/specs/srv/analytics/ONVIF-Analytics-Service-Spec.pdf), and the profile that
exposes it is **Profile M**, titled *"Metadata and events for analytics applications"*.

Profile M's own scope statement (https://www.onvif.org/profiles/profile-m/, VERIFIED):

> "ONVIF Profile M supports analytics configuration and information query for metadata, as well as
> filtering and streaming of metadata. It has interfaces for generic object classification, and
> specified metadata for geolocation, vehicle, license plate, human face and human body."

And its stated use cases include, verbatim:

> "For visitor statistics, crowd control, visitor management or queue management through people
> counting analytics"

**The profile's metadata/analytics operations** (Profile M v1.1, VERIFIED):

| Operation | Service | Requirement |
|---|---|---|
| `GetSupportedAnalyticsModules`, `GetAnalyticsModules`, `CreateAnalyticsModules`, `DeleteAnalyticsModules` | Analytics | M (client) |
| `GetSupportedRules`, `GetRules`, `CreateRules`, `DeleteRules` | Analytics | M (client) |
| `GetMetadataConfigurations` | Media 2 | M |
| `SetMetadataConfigurations` | Media 2 | O (optional — changed from mandatory in v1.1 editorial correction) |
| `SetSynchronizationPoint`, `CreatePullPointSubscription`, `PullMessages` | Event | M |
| MQTT event publishing | Event | CONDITIONAL |

The Profile M v1.1 revision history records the one change from v1.0:
*"Editorial correction 7.8.2: Get/Set MetadataConfiguration TSC is Optional"* (VERIFIED).

### A.3 Can a camera emit anonymous metadata events WITHOUT video? Which event names?

**Yes — with an important caveat about which events are actually standardised for this use.**

**Events defined in the Analytics Service Specification v26.06 (VERIFIED — appendix A of that
document):**

| Rule | Topic | Payload |
|---|---|---|
| Motion detector | `tns1:RuleEngine/DeclarativeMotionDetector/Motion` | — |
| Line detector | `tns1:RuleEngine/LineDetector/Crossed` | — |
| **Field detector** | **`tns1:RuleEngine/FieldDetector/ObjectsInside`** | **`IsInside [xs:boolean]`** — "True if one or more objects matching the parameters are inside." |
| Loitering detector | `tns1:RuleEngine/LoiteringDetector/ObjectIsLoitering` | `Since [xs:dateTime]` |
| **Line crossing counting** | **`tns1:RuleEngine/CountAggregation/Counter`** | **`Count [xs:int]`** |
| **Occupancy counting** | **`tns1:RuleEngine/CountAggregation/OccupancyCounter`** | **`Count [xs:int]`** |
| Object detection | `tns1:RuleEngine/ObjectDetection/Object` | object class list |

The **Field detector** deserves attention for this project, because it is a *boolean* rather than a
count and is defined with an explicit `Field [tt:Polygon]` region:

> "A FieldDetector is defined by a simple non-intersecting polygon. The FieldDetector determines if
> each object in the scene is inside or outside the polygon. This information is put into a property."
> — Analytics Spec v26.06 §A.3

**`ObjectsInside` with `IsInside` is a plausible fit for a CLEAR/MODERATE/BUSY three-way decision**
and requires strictly less of the device than a calibrated counter. It is *not* in Profile M
either, so the same `GetSupportedRules` check applies.

**Privacy assessment of these events — honest reading:**

- `OccupancyCounter` and `Counter` carry **only an integer**. No image, no geometry, no time series
  per person. This is the cleanest possible anonymous signal.
- `FieldDetector/ObjectsInside` carries a boolean. Also clean. But the spec adds: *"Note: a device
  signalling ObjectId shall emit changed events whenever the list of objects changes."* If a device
  also signals `ObjectId`, you are receiving per-object identifiers. **The client must therefore
  filter the metadata stream to exclude the `ObjectId` element explicitly, and the design must treat
  that filter as a correctness requirement, not an optimisation.** (VERIFIED quote; the design
  requirement is INFERRED.)
- `LoiteringDetector` and `FieldDetector` carry per-object events. They are the ones to avoid.
- Profile M also defines `Face recognition event` (§8.13) and `License plate recognition event`
  (§8.14), both **CONDITIONAL**. The design must simply never subscribe to those topics. Presence of
  a capability on the device is not consent to use it.

### A.4 RTSP — what it is, and the practical Python/OpenCV mechanism

**VERIFIED.** RTSP is the IETF application-layer control protocol for real-time media.

- **RFC 2326**, *Real Time Streaming Protocol*, April 1998 — **OBSOLETE.** The RFC Editor's status
  page states: *"This RFC is now obsolete, see RFC 7826."*
  (https://www.rfc-editor.org/info/rfc2326/)
- **RFC 7826**, *Real-Time Streaming Protocol Version 2.0*, December 2016 — Standards Track.
  *"RTSP 2.0 is a replacement of RTSP 1.0 [RFC2326] and this document obsoletes that specification.
  This protocol is based on RTSP 1.0 but is not backwards compatible other than in the basic version
  negotiation mechanism."*
  (https://www.rfc-editor.org/rfc/rfc7826.html)

**A documented tension worth recording (CONTRADICTED-adjacent):** the ONVIF Streaming Specification
still builds on the obsolete RFC. The ONVIF Streaming Spec (v2.5 text) states *"Media control is
accomplished over RTSP as defined in RFC 2326"* and *"A set of media streaming (audio, video and meta
data) options, all based on RTP [RFC 3550]"* (https://www.onvif.org/specs/2206/ONVIF-Streaming-Spec-v2206.pdf).
RTP itself (RFC 3550) is the current standard. **Practical meaning: ship RTSP 1.0 client behaviour,
because that is what deployed cameras speak.** Do not implement RTSP 2.0 semantics.

**Practical mechanism (VERIFIED from OpenCV API docs):**

```python
import cv2
cap = cv2.VideoCapture(
    "rtsp://user:pass@host:554/Streaming/Channels/101",
    cv2.CAP_FFMPEG,
)
if not cap.isOpened():
    raise RuntimeError("stream open failed")

while True:
    ok, frame = cap.read()          # frame is a numpy array in RAM
    if not ok:
        break
    # ... count occupancy ...
    # frame goes out of scope here; never written
```

OpenCV `VideoCapture` documentation (https://docs.opencv.org/4.13.0/d8/dfe/classcv_1_1VideoCapture.html),
verbatim:

> "Opens a video file or a capturing device or an IP video stream for video capturing with API
> Preference."
> "Note that each video stream or IP camera feed has its own URL scheme. Please refer to the
> documentation of source stream to know the right URL."

Backends selectable via `apiPreference`, including `cv::CAP_FFMPEG`, `cv::CAP_GSTREAMER` and
`cv::CAP_IMAGES` (same URL). **INFERRED:** force `CAP_FFMPEG` explicitly rather than `CAP_ANY`,
because a vendor GStreamer path can silently fall back and behave differently on the school network.

**Latency note.** The ONVIF Streaming Spec sets a default encoder timeout of 60 seconds and
documents RTSP `PLAY` with `Range: npt=now-` (VERIFIED, Streaming Spec §5.2.1.1.3). In practice a
sub-stream RTSP pull adds one to several frame intervals. If low latency matters, request the
**sub-stream**, not the main stream — but note that a sub-stream is usually lower resolution, which
directly harms the accuracy question in §B.

### A.5 Profile S / Profile T / Profile G — does the distinction matter for a school camera?

**Profile S — VERIFIED, and it is being retired:**

> "SAN RAMON, Calif. – Oct. 9, 2025. ONVIF®, the leading global standardization initiative for
> IP-based physical security products, announces that it will end support for ONVIF Profile S, and
> is recommending the use of its successor, Profile T."
> — https://www.onvif.org/?p=8621&post_type=pressrelease

Three specific facts from the same release:

> "The June 2026 version of the ONVIF conformance test tools, which have a nine-month validity
> period, will be the last test tool version that enables manufacturers to claim product conformance
> to Profile S."
> "Although the deprecation of Profile S has no effect on the operation of deployed Profile S-based
> systems, ONVIF strongly encourages, if possible, the discontinuation of the use of the username
> token authentication in Profile S."
> "The majority of ONVIF conformant devices and clients available on the market today support both
> ONVIF Profile S and Profile T."
> — https://www.staging.onvif.org/ and https://www.securitysystemsnews.com/article/onvif-ending-support-for-profile-s-recommends-profile-t

The end-of-support date is given in ONVIF's own October 2025 overview deck:
**"ONVIF to end support for Profile S on March 31, 2027."**
(https://www.onvif.org/wp-content/uploads/2025/10/onvif-overview-20251009.pdf, VERIFIED)

Profile S Specification v1.3 mandates, as profile-mandatory features:
> "User Authentication — WS-Usernametoken using timestamps and nonce — HTTP Digest"
> (https://www.onvif.org/wp-content/uploads/2019/12/ONVIF_Profile_-S_Specification_v1-3.pdf, VERIFIED)

**Profile T** — *"contains virtually all the features of Profile S, in addition to other advanced
features for video surveillance"* (SDM Magazine, quoting the ONVIF release, VERIFIED via
https://www.sdmmag.com/articles/104741-...). Introduced 2018. Adds H.264/H.265, imaging settings,
motion alarm and tamper events, **metadata streaming**, bi-directional audio
(ONVIF Overview deck, VERIFIED).

**Profile G** — edge storage and retrieval: recording configuration and control. ONVIF's own site:
*"ONVIF Profile G — For edge storage and retrieval"* (VERIFIED).

**Does the distinction matter here? Honestly, less than expected, and not for the reason people
assume.** A school that bought cameras five or more years ago will have Profile S gear that
predates Profile M entirely. A school that bought recently may have Profile T but almost certainly
**not Profile M**, because Profile M is an analytics-edge-device profile and school cameras are
bought as dumb cameras. **The practical consequence is that the analytics capability we need will
usually have to come from the NVR/VMS, not the camera.** See §D.1.

### A.6 Authentication: Digest vs Basic, and what school NVRs actually do

**VERIFIED from the specifications:**

- **ONVIF device management** uses HTTP Digest and/or WS-Security UsernameToken. Profile S mandates
  both (Profile S Spec v1.3 §7.1, quoted above). The ONVIF Core Spec v26.06 has a dedicated section
  *"5.9.1 Authentication over HTTP and HTTPS"* (https://www.onvif.org/specs/2606/ONVIF-Core-Spec-v2606.pdf).
- **RTSP authentication** is separate from ONVIF SOAP authentication and separate again from the
  video stream's own transport. In practice all three are the same credentials on most devices but
  are enforced by different components. **INFERRED: a device can be configured to allow ONVIF
  digest login while rejecting RTSP login, or to require RTSP Basic while ONVIF uses digest.**
- **HTTP Digest** is the standard for RTSP and HTTP video paths because credentials are not sent in
  clear. **HTTP Basic** over plain HTTP sends credentials base64-encoded — this is obfuscation, not
  protection.

**Which is typical for school NVRs — honest answer: UNVERIFIED.** We found no survey or authoritative
study of Indian school NVR authentication practice. What we can say from vendor documentation is
that the RTSP URL is typically `rtsp://user:pass@host:554/<vendor-path>` and that behaviour varies.
For example, TP-Link's Omada documentation (https://support.omadanetworks.com/baltic/document/44983,
VERIFIED as vendor documentation):

> "When the ONVIF Profile is not supported on the third-party device, Omada Cameras can also enable
> live streaming on third-party platforms via the RTSP protocol. However, the Omada Camera settings
> cannot be modified at this time."

and the same document's warning:

> "Please note that the RTSP/ONVIF protocols are not highly secure, so it is not advisable to
> expose the camera to a public IP via port forwarding for long periods."

**Design constraint from the brief — "no bypass of CCTV auth" — is fully compatible with all of the
above.** The correct posture is: use the school's issued ONVIF credentials, read-only, on the
existing management VLAN, with no port forwarding and no credential sharing outside the school. That
is *narrower* than what most integrators do, which is a feature.

### A.7 Common failure modes, and the realistic success rate of "just try the RTSP URL"

**Failure modes — VERIFIED as documented, real occurrences:**

1. **Vendor-specific, model-specific, often case-sensitive paths.** ONVIF-independent troubleshooting
   guidance states: *"The scheme, host, port, and vendor-specific path must all be correct. Paths can
   differ by model, channel, and main/sub-stream choice, and they are often case-sensitive."*
   (https://omninvr.com/guides/rtsp-camera-not-connecting.html)
2. **Cameras that claim ONVIF and do not deliver it.** A documented AliExpress-sourced camera case where
   ONVIF and RTSP were advertised, could not be enabled in the vendor app, and a firmware update did
   not unlock them. (https://ipcamtalk.com/threads/troubleshooting-ip-camera-compatibility-with-onvif-and-rtsp-protocols.69773)
3. **ONVIF and RTSP ports are separate and separately configurable**, and NVRs frequently expose only
   one. A TP-Link VIGI NVR forum thread documents users unable to specify the RTSP port on manual
   camera add. (https://community.tp-link.com/en/business/forum/topic/715348)
4. **RTSP transport mismatch.** A real Home Assistant issue where FFmpeg did not respect the
   configured TCP transport and UDP then failed on the target network.
   (https://github.com/home-assistant/core/issues/172349, closed as stale/not_planned 2026-09-03)
5. **NVR re-stream.** Whether an NVR will re-stream a channel to a third-party RTSP client is a
   **per-model feature, not a standard**. ONVIF Profile G governs recording control; it does not
   impose a re-stream contract. **NOT FOUND:** any specification text requiring NVR re-stream.
   **INFERRED:** many NVRs will not re-stream at all.
6. **"Works in the player but not in the recorder"** is a well-documented distinct failure class
   involving timestamps, muxing and codec negotiation.
   (https://omninvr.com/guides/troubleshoot-rtsp-recording.html)

**The realistic success rate of "just try the RTSP URL": NOT FOUND, and we decline to guess.**

We searched for a quantified success rate and did not find one from a credible source. Anyone quoting
"80% of cameras work with a standard RTSP URL" is quoting folklore. What *is* supportable:

- Cameras reachable through **ONVIF** can have their stream URI discovered properly via
  `GetStreamUri` rather than guessed. ONVIF Profile M requires `GetProfiles` and `GetStreamUri`
  (Media 2, M) — VERIFIED. **This is the entire argument for using ONVIF rather than guessing
  paths:** discovery replaces guessing, and the credential path is the same one the school already
  issued.
- The failure rate is dominated by two things the integrator cannot control: what model the school
  bought, and whether the installer enabled RTSP at all. Many Indian school CCTV installations are
  **installer-configured, turnkey, with RTSP and ONVIF never enabled and the installer account
  password unknown to the school.** **INFERRED — this is a procurement and governance risk before it
  is a technical one.**

**This is why the first action item in §D.5 is not code. It is a question to the school.**

---

## B. OpenCV and algorithms — anonymous occupancy without identity

### B.1 Which OpenCV functions are relevant, and are they still maintained?

| Function | Module | Status | Licence |
|---|---|---|---|
| `cv2.createBackgroundSubtractorMOG2` / `KNN` | `bgsegm` | Main library, active | Apache-2.0 |
| `cv2.HOGDescriptor` (people detector) | **moved to `opencv_contrib` `xobjdetect` in OpenCV 5** | Demoted | Apache-2.0 |
| `cv2.CascadeClassifier` (Haar, incl. people) | **moved to `opencv_contrib` `xobjdetect` in OpenCV 5** | Demoted | Apache-2.0 |
| `cv2.dnn` (MobileNet-SSD, YOLO, etc.) | `dnn` | Main library, **"largest set of changes since OpenCV 4.x"** | Apache-2.0 |

**Licence — VERIFIED:** OpenCV switched from BSD to **Apache 2.0**. From the OpenCV changelog wiki:
*"Recent OpenCV 4.x (since OpenCV 4.5.0) and also OpenCV 5.0 are now distributed under Apache 2
license. We used BSD license before, but switched to Apache 2, because the latter is more refined
and provides somewhat better protection (not ideal though) from patent-related issues."*
(https://github.com/opencv/opencv/wiki/OpenCV-Change-Logs)

**Maintenance status — VERIFIED, and this is the decisive fact for this project.** From the official
OpenCV 4→5 migration guide (https://github.com/opencv/opencv/wiki/OpenCV-4-to-5-migration):

> "**Objdetect: Haar and HOG moved to contrib**"
> "cv::CascadeClassifier (Haar-based) and cv::HOGDescriptor (for pedestrian detection) have been
> moved to the xobjdetect module in opencv_contrib."
> Python: *"Haar/HOG detectors: cv2.CascadeClassifier and cv2.HOGDescriptor require opencv_contrib."*

And from the OpenCV 5 release notes (https://github.com/opencv/opencv/wiki/OpenCV-5):

> "**Objdetect module cleanup.** Haar-based and HOG-based detectors have been moved to
> opencv_contrib (xobjdetect). **Modern deep learning-based detectors are both faster and more
> accurate.**"

Also VERIFIED: OpenCV maintains two active branches, *"the previous stable 4.x branch and 5.x branch
(now also stable)"*, with OpenCV 5.0.0 released **2026-08-19**
(https://opencv.org/releases/).

**Honest reading:** the upstream project has made an architectural statement about this exact
choice. If this project builds on HOG or Haar, it is building on code the OpenCV maintainers have
moved out of the core library because they consider it superseded, and it now carries the
dependency burden of `opencv_contrib`. That is not disqualifying — `opencv_contrib` is Apache-2.0
and perfectly usable — but it should be a conscious decision, not an accident.

### B.2 Limits of each approach for a school corridor

| Approach | Honest limit in this scenario |
|---|---|
| **MOG2 / KNN background subtraction** | Counts *foreground blobs*, not people. A student carrying a large bag, a swinging door, a cleaning trolley, or a shadow all register. A person standing still for 30 seconds is **absorbed into the background model and disappears** — which is catastrophic for corridor occupancy, where people queue and wait. **INFERRED from the algorithm's definition; we did not find a corridor-specific published evaluation.** |
| **HOG + linear SVM** | 2005 method. Trained on upright pedestrians at pedestrian scale. Partial occlusion, sitting people, backlit silhouettes and edge-clipped people all degrade it badly. Now in contrib. |
| **Haar cascade people** | Weaker than HOG by the project's own assessment (implied by recommending HOG historically and by the general ordering of the literature). Now in contrib. |
| **DNN detector (MobileNet-SSD etc.)** | Best of the OpenCV-native options. But a *detector* produces boxes, which invites tracking. A detector is only privacy-preserving if the tracking layer is deliberately never built. |
| **Density map (CSRNet etc.)** | Produces a count directly, no per-person boxes, therefore no identity by construction. See §B.5. |

**The "people standing still" problem deserves emphasis.** School corridor occupancy is not a
transit flow — it is dominated by *stationary clusters*: children waiting for a class, a queue at a
door, a group waiting for a lift. This is the single scenario where MOG2 fails outright and where a
flow-counter (`OccupancyCounter`) derived from line crossings can be correct while a
presence-detector is also correct, but a background-subtractor is wrong. **Inferring occupancy from
background subtraction in a school corridor is a category error.**

### B.3 Is classical OpenCV people detection accurate enough in a cluttered indoor corridor?

**This is the question the brief specifically flagged, so here is the direct answer:**

**NOT FOUND — no published accuracy figure exists for this specific task.**

We searched for benchmark data on classical OpenCV people detection in cluttered indoor school
corridors and found none. What exists:

- **Dalal & Triggs, CVPR 2005**, the original HOG paper, evaluated on the **INRIA Person** and
  **PASCAL VOC** pedestrian datasets — outdoor, upright, walking people.
  (https://www.cs.princeton.edu/courses/archive/fall13/cos429/papers/Dalal05.pdf)
  The paper's own framing: *"taking 'pedestrian detection' (the detection of mostly visible people in
  more or less upright poses) as a test case."* That is not a school corridor during changeover.
- **Crowded-scene datasets** (UCSD, ShanghaiTech, UCF-QNRF, NWPU-Crowd, JHU-Crowd++) are academic
  benchmarks dominated by outdoor mass-crowd imagery. NWPU-Crowd contains *"various illumination
  scenes"* and a density range of 0–20,033 (https://arxiv.org/abs/2001.03360), which is a different
  regime from a 30-student secondary-school corridor.
- **The OpenCV tutorial's own performance figures** for HOG+SVM on the 2008 PASCAL VOC person class
  are widely quoted second-hand. **We were unable to retrieve the current tutorial page** —
  `https://docs.opencv.org/4.x/d6/dc3/tutorial_py_people_detector.html` returned **404**, and
  `docs.opencv.org` served a Cloudflare interstitial to direct fetch on the background-subtraction
  tutorial. **We therefore do not quote the 0.84-miss-rate figure as VERIFIED, because we could not
  read it from the source in this pass.** Marked UNVERIFIED pending direct retrieval.

**What this means practically:** any accuracy claim for this project must come from *measuring on
the school's own corridor*, with a consented cohort of adults and with no students present. There
is no shortcut and no published number to lean on.

### B.4 Modern alternatives, compared on privacy grounds

| Family | Identity risk | Privacy posture | Suitability here |
|---|---|---|---|
| **Track-by-detection (SORT/DeepSORT/ByteTrack)** | **Highest.** DeepSORT in particular embeds a *re-identification* vector whose entire purpose is matching a person across cameras and time. | Poor. Directly violates "no persistent tracking of identifiable persons". | **EXCLUDE.** |
| **Per-frame detector, no tracker** | Low. Detections are ephemeral and discarded. | Good, provided no tracker is constructed and no frames are retained. | Acceptable, with hard engineering controls. |
| **Density map (no boxes)** | **None by construction.** A density map is a spatial heatmap; summing it yields a number. There is no per-person object to re-identify. | **Best.** | **Preferred, if compute allows (§B.6).** |
| **Holistic / motion-segmentation counting** | None — the canonical privacy-preserving formulation. | Best. | Well-established, older, see §B.5. |
| **mmWave / ToF / thermal** | **None. No image of any person is ever formed.** | **Best possible.** | See §D. |

**Density maps are the technically strongest privacy-preserving option that still uses a camera**,
and they map almost exactly onto this task: you want a number in `CLEAR / MODERATE / BUSY`, not a
list of people. The constraint is compute, addressed next.

### B.5 Published work on privacy-preserving crowd counting

**The canonical citation — VERIFIED:**

> **A. B. Chan, Z. S. J. Liang, and N. Vasconcelos. "Privacy Preserving Crowd Monitoring:
> Counting People without People Models or Tracking."** *IEEE Conference on Computer Vision and
> Pattern Recognition (CVPR), Anchorage, June 2008.* DOI: **10.1109/CVPR.2008.4587569**
> — https://ieeexplore.ieee.org/document/4587569
> Lab page: http://svcl.ucsd.edu/projects/peoplecnt

Abstract, verbatim:

> "We present a privacy-preserving system for estimating the size of inhomogeneous crowds, composed
> of pedestrians that travel in different directions, without using explicit object segmentation or
> tracking."

The authors' framing of the thesis — from the lab page (http://svcl.ucsd.edu/projects/peoplecnt):

> "This is a canonical example of a problem that vision technology addresses with privacy invasive
> methods: detect the people in the scene, track them over time, and count the number of tracks.
> **Unlike these methods, we show that there is in fact no need for pedestrian detection, object
> tracking, or object-based image primitives to accomplish the pedestrian counting goal**, even when
> the crowd is sizable and inhomogeneous, e.g. has sub-components with different dynamics."

And the most directly transferable sentence in the whole paper for this project:

> "The system is also privacy-preserving in the sense that **it can be implemented with hardware
> that does not produce a visual record of the people in the scene**, i.e. with special-purpose cameras
> that output low-level features (e.g. segmentations, edges, and texture)."

**Evaluation scale (VERIFIED):** validated *"on a large dataset of pedestrian video, containing
49,885 pedestrian instances"*, plus a demonstration *"on an hour of video"*.

**A caveat we must state:** this is 2008 work on outdoor pedestrian crowds. Its method (mixtures of
dynamic textures, segmenting the crowd into homogeneous-motion components) assumes people are
*moving*. A school corridor full of stationary children is close to adversarial for that method.
The paper's *thesis* — that counting need not require identification — is what transfers. Its
*method* is not obviously what we should implement.

**Modern density-map state of the art, for context:**

- **Zhang et al., MCNN**, CVPR 2016 — multi-column CNN, the canonical baseline.
- **Li, Zhang & Chen, CSRNet**, CVPR 2018, *"CSRNet: Dilated Convolutional Neural Networks for
  Understanding the Highly Congested Scenes"* — https://openaccess.thecvf.com/content_cvpr_2018/papers/Li_CSRNet_Dilated_Convolutional_CVPR_2018_paper.pdf
  Reported: MAE 68.2 on ShanghaiTech Part A, **10.6 on Part B**; PSNR 23.79.
- Privacy-aware federated variant: **Chen et al. (2024), "Privacy-aware crowd counting by
  decentralized learning with parallel Transformer"**, *Privacy & Security* —
  https://www.sciencedirect.com/science/article/abs/pii/S2542660524001082
  — *"We propose a novel framework called the Decentralized Learning with Parallel Transformer network
  (DLPTNet). Based on the federated learning mechanism."* (VERIFIED as existing and relevant; the
  federated framing addresses *training-data* privacy, not *runtime* identity, and is therefore
  orthogonal to this project's constraints.)

### B.6 Latency and compute — can it run on a modest CPU?

**Honest answer: the verified numbers are all for GPUs or NPUs. We did not find a verified
CPU-only figure.**

What is verified:

- **CSRNet at 384×384, 16.2M parameters, 21.67 FPS on a Titan XP** (a 2016 Pascal GPU), benchmarked
  in TransCrowd (https://arxiv.org/pdf/2104.09116v3, Table 8: *"Comparison with BL and CSRNET using
  the same input image resolution on a Titan XP"*). The same table notes *"the FPS of VGG19-based BL
  outperforms the VGG16-based CSRNet"* because BL *"generates a small-resolution density map (1/16 of
  the input image)"* — i.e. **output feature-map resolution dominates runtime**.
- **NPU deployment on an RK3399Pro-class SoC**: inference times of **9–28 ms** at 320×184 to
  640×360 on Venice and Beijing-BRT datasets via RKNN.
  (https://www.sciencedirect.com/org/science/article/pii/S1546221823004551)

**INFERRED from the above:** 20–35 FPS on a discrete GPU; single-digit to low-teens FPS on a
RK-class NPU at reduced resolution; **a modest x86 or ARM CPU without AVX-accelerated ONNX runtime
would be materially slower**, plausibly low single digits at useful resolution. This is a
reasonable engineering inference from the feature-resolution sensitivity the benchmarks themselves
document, but it is **not a measurement and must not be presented to the school as one.**

**Practical mitigation, and the reason the school robot's constraints matter:** the occupancy task
does not need 20 FPS. A corridor's occupancy state changes on a timescale of tens of seconds.
**Sampling the stream at 1–2 FPS and running the estimator on that is both sufficient and a large
privacy win**, because it means the pipeline is architecturally incapable of reconstructing a
video of anyone. That is a design choice that serves both goals at once. (INFERRED.)

### B.7 Can occupancy be estimated from a SINGLE camera reliably?

**Single-camera estimation is viable. Single-camera *counting* is not reliable in general. The
distinction is the whole design.**

Failure modes, ranked by likelihood in a school corridor:

| Failure mode | Severity here | Mitigable? |
|---|---|---|
| **Stationary people** (waiting, queuing, sitting) | **Critical.** Defeats background subtraction outright; defeats transit counting. | Partially — density map or presence sensor. |
| **Partial / full mutual occlusion** in a cluster | High. Density maps degrade; a detector counts visible heads, not people. | Partially — head-count annotations help; corridor geometry matters. |
| **People at the frame edge** | High. Half-observed humans are routinely missed. | Geometry — mount the camera down-corridor, not across it. |
| **Lighting change** (flickering fluorescent, sunlight through a window) | Medium-high. Destroys HOG's gradient assumption and MOG2's background model. | Partially — time-of-day learning, or radar which is immune. |
| **Shadows and reflections** (polished floors, glass partitions) | Medium. Produce false foreground. | Partially. |
| **Occlusion by the robot itself** if using the onboard camera | High for §D.1(b) only. | Yes — mount high. |
| **Calibration drift** | Low-medium. | Yes — periodic re-check. |

**Single-camera is not the fundamental problem. A corridor is a constrained, high-value scene:
fixed camera, known geometry, known camera height, mostly-side-on people, bounded density.** This
is about as favourable as crowd counting gets. The honest caveat is that **single-camera counting is
occlusion-limited in an absolute sense and no algorithm removes that** — hence the third value in
the CLEAR/MODERATE/BUSY enum is doing important work. A binary CLEAR/BUSY decision is far more
defensible than a number.

---

## C. What is deliverable without storing video

### C.1 Can occupancy be computed streaming, with frames discarded and no video written?

**Yes. This is straightforward engineering, and it is assertable — but the assertion must be
designed for, not assumed.**

The mechanism:

```python
def process_once(cap):
    ok, frame = cap.read()          # frame lives in RAM only
    if not ok:
        return Occupancy.UNKNOWN
    count = density_sum(frame)      # int, or a 3-band label
    return classify(count)          # frame is unreachable after return
```

**The technical assertion and verification of "no video written":**

1. **Static assertion — never call a writer.** Assert at import time and in CI that no
   `cv2.VideoWriter`, no `cv2.imwrite`, no `ffmpeg` invocation, and no `tempfile` write of frame
   data exists in the codebase. This is a lintable, testable invariant.
2. **Process-level assertion — deny write syscalls.** Run the estimator under a sandbox where the
   frame-processing code has no filesystem write permission (`chmod`/seccomp/container mount with a
   read-only filesystem except for the metrics socket). **If the process physically cannot write a
   file, "no video stored" is a property of the system, not a promise.** This is the strongest
   practical guarantee and is the one to recommend.
3. **Runtime assertion — measure what leaves.** Monitor the process's write syscalls and its network
   egress, and assert total bytes written to disk and bytes sent over the network are bounded and
   consistent with metrics only. Alert on any anomaly.
4. **Artifacts assertion — the pipeline emits nothing image-shaped.** The only outputs permitted are
   the enum, an integer count, a timestamp, and a confidence value. Add a schema validation on the
   output message that rejects any field exceeding the declared type — so an accidental image
   payload cannot silently ship.
5. **Independent negative test.** Run the pipeline against a synthetic stream, then assert the total
   size of all files created anywhere on the filesystem during the run is zero (or bounded to
   metrics). This catches anything the static analysis missed.

**The honest limit (INFERRED):** this proves the *robot's* pipeline stores nothing. It does not and
cannot prove anything about the school's NVR, which — under CBSE clause 4.7.10 — is recording
audio-visual footage in the corridor anyway with a 15-day archive. **The privacy delta this project
delivers is "we did not add storage," not "no video exists."** That distinction must be stated to
the school in exactly those words, or the project oversells itself.

### C.2 "Metadata-only processing" and the ONVIF facilities that support it

**Metadata-only processing** means: the camera performs the perception at the edge, and the only
thing that crosses the network is structured, non-image analytics output.

**ONVIF facilities that support it, all VERIFIED:**

1. **Profile M makes the metadata stream mandatory and the video stream conditional** (§0 above).
   The profile is *architecturally* a metadata-first profile.
2. **Metadata Streaming Specification, v26.06.** The metadata stream is transported over RTSP/RTP
   with dedicated payload types, and the specification defines the container:
   - SDP encoding names: `"vnd.onvif.metadata"` (uncompressed), `"vnd.onvif.metadata+gzip"` (GZIP),
     `"vnd.onvif.metadata.exi.ext"` (EXI). (Streaming Spec §5.2.2.4, VERIFIED)
   - *"The metadata streaming container format allows well-defined, real-time streaming of analytics,
     PTZ status and notification data."* (VERIFIED)
   - *"Only UTC timestamps shall be used within the metadata stream."* (VERIFIED)
   - The container carries a `VideoAnalyticsStream` element.
   - Notably, the SDP media type for a metadata stream is **`"application"`**, not `video` — the
     distinction is visible at the protocol level.
3. **MQTT event transport (Profile M §8.5, CONDITIONAL).** JSON-formatted events over
   `mqtt`/`mqtts`/`ws`/`wss`, with topic retrieval via `GetEventProperties`. ONVIF's own example use
   case: *"Room temperature control, where a Profile M camera (with MQTT support) detects humans in a
   room and sends an ONVIF event over MQTT to an IoT platform/application."* **INFERRED: for a
   corridor-occupancy integer, MQTT is a far lighter integration than RTSP and is worth probing if
   the school's VMS supports it.**
4. **Pull-point event delivery (Profile M §8.4, CONDITIONAL).** `SetSynchronizationPoint`,
   `CreatePullPointSubscription`, `PullMessages`, `GetEventProperties`, `Unsubscribe`.

**Profile M's own framing of the privacy-relevant feature split** — metadata streaming mandatory,
video conditional, image sending conditional, face recognition event conditional. **Every
privacy-risky feature in Profile M is conditional and every privacy-benign feature is mandatory.**
That is the standard working in the project's favour, and it should be said to the school.

### C.3 Indian law — what it actually says

#### The DPDP Act 2023

Source: **https://www.meity.gov.in/static/uploads/2024/02/Digital-Personal-Data-Protection-Act-2023.pdf**
(Act No. 22 of 2023, dated 11 August 2023). All quotes below read directly from that PDF.

**The definitional hinge — §2(t):**

> "(t) **'personal data' means any data about an individual who is identifiable by or in relation
> to such data**;"

**This is the most important sentence in this entire section for this project.** A bare integer
occupancy count is not "data about an individual who is identifiable by or in relation to such
data". **An aggregate count is not personal data under the DPDP Act.** (INFERRED from the verbatim
definition — but a strong inference, and worth stating plainly to the school, subject to legal
confirmation.)

The danger is entirely in the *pipeline*, not the payload: a video frame **is** personal data; a
detector run on frames produces personal data even if its output is a number.

**§3 — Application. There is no surveillance exemption:**

> "3. Subject to the provisions of this Act, it shall—
> (a) apply to the processing of digital personal data within the territory of India where the
> personal data is collected–– (i) in digital form; or (ii) in non-digital form and digitised
> subsequently;
> (b) also apply to processing of digital personal data outside the territory of India, if such
> processing is in connection with any activity related to offering of goods or services to Data
> Principals within the territory of India;
> (c) **not apply to**— (i) personal data processed by an individual for any personal or domestic
> purpose; and (ii) personal data that is made or caused to be made publicly available by— (A) the
> Data Principal…; or (B) any other person who is under an obligation under any law…"

**VERIFIED NEGATIVE, and worth flagging:** §3 contains **no exemption for the State, no
instrumentality exemption, no law-enforcement exemption, and no surveillance exemption.** The 2022
draft contained a carve-out for state agencies processing in the interests of sovereignty, public
order, etc.; **the enacted 2023 Act does not.** Anyone relying on "we're a school, we're exempt"
will find no such text. Read carefully — §3 is only three paragraphs, and it is reproduced here in
full for that reason.

**§9 — Children (the provision the school cares about):**

> "**9. (1)** The Data Fiduciary shall, before processing any personal data of a child or a person
> with disability who has a lawful guardian obtain **verifiable consent of the parent** of such child
> or the lawful guardian, as the case may be, in such manner as may be prescribed.
> **(2)** A Data Fiduciary shall not undertake such processing of personal data that is **likely to
> cause any detrimental effect on the well-being of a child**.
> **(3)** A Data Fiduciary shall not undertake **tracking or behavioural monitoring of children** or
> targeted advertising directed at children.
> **(4)** The provisions of sub-sections (1) and (3) shall not be applicable to processing of personal
> data of a child by such classes of Data Fiduciaries or for such purposes, and subject to such
> conditions, **as may be prescribed**.
> **(5)** The Central Government may, if satisfied that a Data Fiduciary has ensured that its
> processing of personal data of children is done in a manner that is **verifiably safe**, notify for
> such processing…"

**§9(3) is the provision to design against.** It is a flat prohibition on tracking and behavioural
monitoring of children. §9(4) is the escape hatch, and it is what the Rules then fill in.

**§4 — lawful grounds:** processing only for a lawful purpose, either **(a) consent** or
**(b) certain legitimate uses**; and *"'lawful purpose' means any purpose which is not expressly
forbidden by law."*

#### Commencement — the timeline fact that changes the plan

**Notification G.S.R. 843(E), 13 November 2025**, Gazette No. 757.
Official: **https://egazette.gov.in/WriteReadData/2025/267647.pdf**
MeitY-hosted copy of the same instrument: **https://www.meity.gov.in/static/uploads/2025/11/c56ceae6c383460ca69577428d36828b.pdf**

⚠️ **Retrieval note, stated for accuracy:** `egazette.gov.in` failed TLS certificate verification
in our environment, so the text below was read from the **MeitY-hosted copy**. This is the same
notification on a different government host and is consistent with the eGazette text as indexed;
we did not independently compare the two binaries.

Verbatim:

> "(a) the date of publication of this notification in the Official Gazette as the date on which the
> provisions of sub-section (2) of section 1, **section 2**, sections 18 to 26 sections 35, 38, 39,
> 40, 41, 42, 43, and sub-sections (1) and (3) of section 44 … shall come into force;
> (b) **one year** from the date of publication of this gazette on which the provisions of
> sub-section (9) of section 6 and clause (d) of sub-section (1) of section 27 … shall come into force.
> (c) **eighteen months** from the date of publication of this gazette, on which the provision of
> **sections 3 to 5**, sub-sections (1) to (8) and (10) of section 6, **sections 7 to 10**, sections
> 11 to 17, section 27 except clause (d) … shall come into force."

**Consequence, stated plainly:** as of **2026-10-10**, the operative obligations — §3 (scope), §4
(lawful grounds), §5 (notice), §6 (consent), §8 (general obligations), **§9 (children)** — are
**NOT YET IN FORCE**. They commence approximately **13 May 2027** (eighteen months from
13 November 2025).

⚠️ **One honest caveat:** the *notification* says "eighteen months from the date of publication"; it
does not itself print a calendar date. The derived date of 13 May 2027 is arithmetic from the
published notification date. Third-party trackers (e.g. dpdprules.org, which flags the point
explicitly) treat 13 May 2027 as a *computed* date "until officially confirmed". **Marked
INFERRED on the specific calendar date; the eighteen-month rule itself is VERIFIED.** The
operational conclusion does not depend on the exact day — it depends on "not this year", which is
certain.

#### The DPDP Rules 2025

**G.S.R. 846(E), 13 November 2025**, Gazette issue No. 760.
Official text: **https://www.meity.gov.in/static/uploads/2025/11/53450e6e5dc0bfa85ebd78686cadad39.pdf**
(MeitY-hosted copy of the Gazette PDF). Full text also mirrored at
https://www.dpdpa.com/DPDP_Rules_2025_English_only.pdf

**Rule 1(2)–(4), commencement (VERIFIED):**

> "(2) Rules 1, 2 and 17 to 21 shall come into force on the date of their publication in the
> Official Gazette.
> (3) Rule 4 shall come into force one year after the date of publication of this Gazette.
> **(4) Rules 3, 5 to 16, 22 and 23 shall come into force eighteen months after the date of
> publication of this Gazette.**"

Rule 12 is in the eighteen-month group. Same conclusion: **not yet in force.**

**Rule 12 (VERIFIED):**

> "**12. Exemptions from certain obligations applicable to processing of personal data of child.**
> (1) The provisions of sub-sections (1) and (3) of section 9 of the Act shall not be applicable to
> processing of personal data of a child by such class of Data Fiduciaries as are specified in
> **Part A of Fourth Schedule**, subject to such conditions as are specified in the said Part.
> (2) The provisions of sub-sections (1) and (3) of section 9 of the Act shall not be applicable to
> processing of personal data of a child for such purposes as are specified in **Part B of Fourth
> Schedule**…"

**Fourth Schedule, Part A, entry 3 (VERIFIED — this is the provision that matters):**

> | S. No. | Class of Data Fiduciaries | Conditions |
> |---|---|---|
> | 3. | **A Data Fiduciary who is an educational institution.** | **Processing is restricted to tracking and behavioural monitoring— (a) for the educational activities of such institution; or (b) in the interests of safety of children enrolled with such institution.** |

And the definition (Fourth Schedule, Note (c), VERIFIED):

> "(c) **'educational institution' shall mean and include an institution of learning that imparts
> education, including vocational education**;"

**Honest assessment of whether this covers the project:**

- **It covers the *class* of Data Fiduciary.** A school is an educational institution. (VERIFIED
  that the class exists and includes schools; INFERRED that a specific school falls in it.)
- **It does not automatically cover this *purpose*.** The condition restricts processing to tracking
  or behavioural monitoring (a) for educational activities, or (b) **in the interests of safety**.
  Routing a delivery robot away from a crowded corridor is, on its face, neither an educational
  activity nor obviously a safety-of-children purpose. **This is a genuine legal question and we are
  not going to resolve it.** It needs a competent Indian data-protection lawyer, and it needs to be
  resolved **before** build, not after.
- **Even if resolved, it only disapplies §9(1) and §9(3).** It does **not** disapply §4 (lawful
  basis), §5 (notice), §6 (consent), or §8 (general obligations). Those still need a basis. A
  school may well be able to rely on the "legitimate uses" limb of §4 — but that determination, too,
  is the school's to make, not ours.
- **Timing:** the exemption itself is not in force until ~May 2027. **Building a system that depends
  on it means depending on law that has not started yet.** That is a project risk, not a detail.

**VERIFIED NEGATIVE — worth stating because it is so often assumed:**

> Searching the full text of the **DPDP Act 2023** for `surveillance`, `cctv`, `biometric`,
> `facial`, `face recognition` returns **one** hit, and it is an illustration about live video-based
> KYC.
> Searching the full text of the **DPDP Rules 2025** for `surveillance`, `cctv`, `biometric`,
> `facial`, `face`, `video`, `image` returns **one** hit, and it is the phrase
> *"payment interface"*.
>
> **There is no Indian statutory definition of, or rule specific to, CCTV surveillance.** The
> framework that governs this project is general-purpose data-protection law applied to a
> general-purpose definition of personal data. Anyone citing an "Indian CCTV law" is citing
> something that does not exist.

#### The IT Act 2000, section 66E — commonly mis-cited

**§66E is far narrower than it is usually described.** Full text (India Code,
https://www.indiacode.nic.in/bitstream/123456789/13116/1/it_act_2000_updated.pdf; text
cross-checked at https://indiankanoon.org/doc/112223967):

> "**66E. Punishment for violation of privacy.** Whoever, intentionally or knowingly captures,
> publishes or transmits the **image of a private area** of any person without his or her consent,
> under circumstances violating the privacy of that person, shall be punished with imprisonment which
> may extend to three years or with fine not exceeding two lakh rupees, or with both.
>
> **Explanation.** —For the purposes of this section—
> (a) 'transmit' means to electronically send a visual image with the intent that it be viewed by a
> person or persons;
> (b) 'capture', with respect to an image, means to videotape, photograph, film or record by any means;
> (c) **'private area' means the naked or undergarment clad genitals, public area, buttocks or female
> breast**;
> (d) 'publishes' means reproduction in the printed or electronic form and making it available for
> public;
> (e) 'under circumstances violating privacy' means circumstances in which a person can have a
> reasonable expectation that—"

**Read it: §66E criminalises capture/transmission of an image of a *genital or breast* area.** It is
not a general surveillance statute, it does not create a consent requirement for corridor CCTV, and
it does not apply to a corridor view. **Anyone citing §66E as the Indian CCTV privacy law is
citing it wrongly.** It is worth knowing because it will come up in conversation with school
administration.

#### Regulatory context that does apply

- **CBSE Affiliation Bye-Laws, clause 4.7.10 (21 July 2025)** — VERIFIED from the official CBSE
  notification: https://www.cbse.gov.in/cbsenew/documents/Amendments_Chapter_4_21072025.pdf
  Notification No. **CBSE/AFF./Notification/2025/0**, dated **21.07.2025**, signed (Himanshu Gupta),
  Secretary.

  > "**4.7.10** 'The school should install high resolution CCTV Cameras with audio visual facility at
  > all the entry and exit points of the school, lobbies, **corridors**, Staircases, all the
  > classrooms, labs, library, Canteen Area, store room, playground and other common areas **except
  > toilets and washrooms** with real time audio-visual recording.
  >
  > These CCTV Cameras must be equipped with a storage device with the capacity of holding footage of
  > at-least **15 days**. It shall be ensured that a backup of a minimum 15 days be preserved, which
  > may be accessed by the authorities, if required.
  >
  > All the affiliated schools shall strictly adhere to this provision in letter and spirit for the
  > safety of school children."

  The notification grounds this in the **NCPCR Manual on safety and security of children in schools**,
  clause 1(X): *"CCTV in schools needs to be monitored and maintained regularly."*

  **Three consequences for this project, all material:**
  1. **In a CBSE school, corridor CCTV is already mandatory and already recording audio-visual
     footage with a 15-day archive.** The project would not be introducing surveillance into the
     corridor; it would be reading a feed that must already exist. That is a genuinely favourable
     framing for the permission conversation.
  2. **The school is already the Data Fiduciary** for that footage, with 15-day retention and a
     real-time recording obligation. Adding a consumer to that stream is a much smaller ask than
     introducing a new camera.
  3. **But audio-visual recording is mandated**, meaning the stream the robot would pull contains
     audio. **The design must therefore explicitly disable or ignore the audio track** — not merely
     promise not to listen. That is a concrete, checkable requirement, and a concrete failure mode if
     someone configures a default `cv2.VideoCapture` that decodes everything.

- **The right to privacy** rests on *K.S. Puttaswamy (Retd.) v. Union of India* (2017) 10 SCC 1,
  locating privacy under Article 21. **VERIFIED** as the governing framework, but note: **the DPDP
  Act does not yet contain a savings clause preserving prior privacy jurisprudence**, and
  *Puttaswamy*'s four-part test (legality, legitimate aim, necessity, proportionality) has not
  been re-legislated. **INFERRED:** *Puttaswamy* remains the constitutional ceiling above the statute.
  A proportionality argument must still be made on those four limbs, and §D.4 is written as that
  argument.

### C.4 Indian Supreme Court guidance on CCTV in educational institutions

**NOT FOUND — and this is a real gap in the research, stated rather than papered over.**

We searched for a Supreme Court of India judgment or binding guideline specifically on CCTV
surveillance in educational institutions and **did not locate one**. We want to be precise about
what we did and did not find:

- **Found, and verified as a real Supreme Court order on educational institutions**, but on a
  different subject: *Suhaas Chaturvedi v. Union of India*, order dated 25 July 2025
  (https://api.sci.gov.in/supremecourt/2024/18489/18489_2024_3_1502_62660_Order_25-Jul-2025.pdf),
  which issues guidelines *"All educational institutions shall adopt and implement a uniform mental
  health policy…"*. **It says nothing about CCTV.** Recorded here so it is not mistaken for
  authority on this question.
- **Found at High Court level**, including an Uttarakhand direction that *"Every Private School in
  the State of Uttarakhand is ordered to install CCTV cameras in the school rooms, corridors and every
  corner, more particularly, in the isolated places in the schools."* However, **we could only
  locate this through a secondary commercial summary** (supremetoday.ai), not the primary judgment.
  **UNVERIFIED.** The *direction of travel* — Indian courts pushing schools to install CCTV, not to
  remove it — appears consistent across the secondary sources, but we are not treating it as
  established.
- **No Indian equivalent located** of the UK ICO's *In the picture: a data protection code of
  practice for surveillance cameras and personal information*, which is the guidance most school
  CCTV policies here actually cite (in adapted form). Its existence in UK school policies and its
  absence in India is itself informative about where the practice sits.

**What this means for the project:** there is **no Indian legal precedent either authorising or
restricting this specific use.** That cuts both ways. It is not a green light — the absence of
prohibitive precedent is not permission — but it does mean the school will be making a novel
decision, and should be told so rather than being given a false citation.

⚠️ **A process point that must not be skipped.** Whatever the merits, an automated system that
touches CCTV in a school should have **human review before any incident report** — as the brief
requires. That is not merely good practice here; it is the direct analogue of a widely-used
comparator policy: the **Madill Board of Education (US)** surveillance policy states that
*"Surveillance cameras also shall not utilize automatic identification, facial recognition, or
automatic tracking technologies. Only authorized school personnel… shall be involved with and/or have
access to surveillance cameras"* (https://files.smartsites.parentsquare.com/10122/ckaj.pdf,
adoption date 8 April 2024). **UNVERIFIED as Indian law; cited as a drafting precedent for the
policy clause we recommend in §E.3.**

---

## D. Comparative options

### D.1 Side-by-side

| | (a) Existing CCTV analytics | (b) Robot's own camera + depth | (c) mmWave / ToF people presence | (d) Manual closure / schedule | (e) Combination |
|---|---|---|---|---|---|
| **Privacy** | **Worst.** Reads footage of identifiable children. Mitigated only by metadata-only design (§C.2). | **Medium.** Images the children the robot passes; but *of the robot's own motion*, no retention, no cross-scene linkage. | **Best. No image of any person is ever formed.** | **Best.** No sensing at all. | (c) + (d) best; (a) only if already deployed |
| **Cost (India)** | ₹0 incremental if an analytics NVR exists. **Potentially high** if one must be bought/licensed. | ₹0 — camera already on the robot | **₹2,000–11,000** per corridor (see §D.3) | ₹0 | ₹2,000–11,000 |
| **Latency** | Highest (RTSP decode + NVR hop) | Lowest (local) | Low (<30 ms processing; see §D.3) | None | Low |
| **Reliability** | **Lowest.** Depends on NVR uptime, ONVIF implementation, credentials, and model-specific paths (§A.7). | High, but occluded by the robot itself and blind beyond its FOV. | **High.** Immune to lighting, dust, colour, clothing, and faces. | High but coarse | High |
| **Maintenance** | **Highest.** Credential rotation, firmware changes, vendor updates, re-tuning after any camera move. | Low | **Low.** Fixed install, no software dependency on the school IT estate. | Lowest | Low |
| **Permission burden** | **Highest.** Data Fiduciary authorisation, proportionality argument, purpose limitation, credential issuance, incident-reporting procedure. | **Medium.** Robot-owner policy; but it is still imaging children in a school. | **Low.** No personal data. A facilities decision. | **Lowest.** Pure scheduling. | Low–medium |
| **Counts or presence?** | Both (Count is an int) | Both | **Both** — see §D.3 | Neither (coarse prior) | Both |

### D.2 Option (b) — the robot's own camera

**The hard constraint in the brief — "the robot must NOT depend on CCTV for obstacle avoidance" —
is fully respected here, because this is the robot's own sensor and not the CCTV system at all.**

What it can realistically support, from the Phase 0 research already in this repository
(`research/sdk_and_software_research.md`):

- The Go2 EDU's onboard camera is documented at **1080p 30 fps / 720p 60 fps** in the developer FAQ,
  with F2.2, 120° FOV. (VERIFIED in the prior phase; the same source notes an fps conflict with the
  About-Go2 page — see `source_register.md` §6 C2.)
- **720p at 15 fps is comfortable for this task.** Occupancy is a slow-changing scalar; 720p
  (1280×720) at even 1–2 fps is ample resolution for a corridor-scale presence/density estimate.

**The decisive limits — state these plainly:**

1. **The camera sees only what is ahead of the robot.** It cannot see a corridor around a corner or
   at the far end. If the robot's purpose is to *avoid entering* a crowded corridor, onboard sensing
   arrives too late — it would be reacting, not anticipating.
2. **Occlusion by the robot's own body and payload** is a real problem at low mounting heights.
3. **A 120° wide FOV at close range** means near-field people are heavily clipped at the frame edge —
   exactly the failure mode in §B.7.
4. **Privacy-wise it is genuinely better than CCTV** — no cross-scene correlation, no retention, no
   NVR, no credential to leak — but it still images identifiable children without their being able
   to avoid it, so it is not zero.

**Verdict:** excellent as a *local safety cross-check*, poor as the *primary* corridor-occupancy
signal. It cannot see what it has not reached.

### D.3 Option (c2) — mmWave / people-presence sensors: products, cost in India, count vs presence

**Products and pricing (VERIFIED to the sources shown; prices are observed retail snapshots, not
quotations):**

| Product | Chip / band | Count or presence? | Indicative price |
|---|---|---|---|
| **HLK-LD2410B** | 60 GHz, LD2410 | Presence + **up to 3 distance zones** | **$5–15** module (https://homeautocentral.com/best-mmwave-presence-sensor) |
| **HLK-LD2450** | 60 GHz, LD2450 | **Target tracking 2–3 (3 described as "not great")** | **$5–15** module |
| **HLK-LD6001C** | 60–64 GHz | **In/out counting at a doorway, up to 8 people** — https://www.hlktech.net/index.php?id=1312 | **$28.05** |
| **Seeed MR24HPC1** | 24 GHz | Presence, distance gates | **$20–30** |
| **Everything Presence One (EP1)** | LD2410/LD2450 | Presence + target tracking | **$30–40** |
| **Minew MSR01** | 60 GHz, 4T4R | *"People Flow Statistics, People Counting"*, *"100% Anonymous Detection"*, 120°, Wi-Fi/BLE — https://www.minew.com/wp-content/uploads/2023/08/MSR01-poster.pdf | — |
| **Aqara FP1E** | 60–61 GHz | **Presence only** (moving/still status), 120°, 6 m, 200 ms response, 360° rotating head — https://www.aqara.com/eu/product/presence-sensor-fp1e | €49.99 (domadoo.fr listing) |
| **Aqara FP2** | BGT60TR13C | **Up to 5 people simultaneously**, up to 30 zones | **₹11,128 in India** (Amazon.in price 2026-09-07, pricehistory.app tracker — https://pricehistory.app/p/aqara-presence-sensor-fp2-mmwave-radar-wired-oNE3lpw8) |

**A useful comparative table of detection capability across the price band** (Apollo Automation,
https://wiki.apolloautomation.com/products/general/mmwave-presence-sensor-comparison, VERIFIED):

| Device | Price | mmWave chip | Detection (m) | Zones | Target tracking |
|---|---|---|---|---|---|
| Apollo MSR-2 | $34.99 | LD2410B | 6 | 3 zones + 9 gates | 1 |
| Everything Presence Lite | $37.00 | SEN0395 | 9 | — | 1 |
| Apollo MTR-1 | $36.99 | LD2450 | 6 | 3 zones | 2–3 |
| Screek Human Sensor 1U | $25.99 | LD2410C | 6 | 6 | — |
| Screek Human Sensor 2A | $29.99 | LD2450 | 6 | 6 | ✗ |
| Aqara FP1 | $49.99 | BGT60TR13C | 5 | 5 | ✗ |
| Aqara FP2 | $82.99 | IWR6843 | 6–8 | 6 to 8 | 6 to 8 |

**Answering "count or only presence?" precisely:**

- **Cheap modules (LD2410B, ~$5–15): presence plus zone-gated occupancy, NOT reliable head-count.**
  LD2410 reports presence *per configured zone*; it does not resolve individuals. **But — and this is
  the key point for this project — zone-presence maps directly onto the required enum.** Configure
  three zones along the corridor (near / middle / far) and `occupancy_band = f(zones_occupied)` gives
  CLEAR/MODERATE/BUSY with **no counting at all**.
- **LD2450 and above: multi-target tracking, 2–3 targets.** Enough for a real count.
- **HLK-LD6001C: genuine in/out counting at a doorway, up to 8 people**, at 0.5–8 m, ±60° azimuth
  and pitch, ceiling mount at 2.7 m, **processing cycle ≤30 ms**, TTL serial output, 29.36×28 mm,
  1.7 W peak / 0.3 W average (https://www.hlktech.net/index.php?id=1312, VERIFIED). This is close to
  ideal for a corridor threshold.
- **TI's reference design** goes further: **TIDEP-01000** using IWR6843 *"can detect up to 250
  objects, and point and track up to 20 people with a field of view (FOV) of ±60º in the azimuth
  (horizontal) plane"*, detect and track to 15 m
  (https://www.ti.com/tool/TIDEP-01000, VERIFIED). This is a development platform, not a
  drop-in product.

**The privacy argument is categorical, and it is the strongest single point in this document:**

> mmWave radar forms a **range–velocity–angle point cloud**. It has no aperture in the optical
> sense, produces no image, and cannot produce a picture of a person under any circumstance. There is
> no frame to store, no face to protect, no retention question, no data-protection impact assessment
> triggered by the *image* of a child, and **nothing to leak if the device is stolen.** The privacy
> surface is a small set of integers: zone occupancy, and optionally a target count.

**Honest caveats on mmWave — do not oversell it either:**
- **Zone/occupancy accuracy in a corridor is untested by us.** The LD2450's third target is described
  by an independent community comparison as *"not great"* (Apollo table above). Three targets is
  enough for CLEAR/MODERATE/BUSY; it is not enough to publish a precise headcount.
- **Physical installation matters enormously** — ceiling height, corridor width, mounting angle.
  The HLK device's own spec assumes ceiling installation at 2.7 m with a 3.5 m-radius ground
  projection. **This is a survey task, not a plug-and-play task.**
- **Independent reporting is candid that cheap modules need work:** *"The firmware on the generic
  Tuya-branded versions is inconsistent, the sensitivity drifts over weeks, and the configuration
  tooling is fiddly in a way that eats evenings… The honest cost of a bare module is the chip price
  plus your weekend, and your weekend isn't free."*
  (https://homeautocentral.com/best-mmwave-presence-sensor — VERIFIED quote from an independent
  reviewer.)
- **FCC/RED/MIC modular certification exists for some modules** (TI IWRL6432WMOD: *"Modular
  certification with FCC, RED, MIC certifications"*, https://www.ti.com/lit/ds/symlink/iwrl6432wmod.pdf).
  **In India, the applicable regime is different** and we did **not** locate an authoritative source
  on wireless-equipment certification requirements for 60 GHz radar in an Indian school. Marked
  **NOT FOUND.** *A cheap uncertified module may not be lawful to operate in India; this must be
  checked before procurement.*

### D.4 The proportionality argument (Puttaswamy's four limbs)

The brief asks whether the concept is worth the permission burden. Put through the test that
*Puttaswamy* (2017) 10 SCC 1 established, and that remains the constitutional ceiling (§C.3):

| Limb | CCTV analytics route | mmWave route |
|---|---|---|
| **Legality** | Lawful — DPDP §§4/5/6 once in force; no prohibition. | Lawful — no personal data at all. |
| **Legitimate aim** | Child safety in crowded corridors. Defensible. | Identical. **Same aim.** |
| **Necessity** | **Weakest link.** Is reading children's CCTV *necessary* to know a corridor is busy, when a radar sensor answers that question without imaging anyone? A regulator would ask exactly this. | Strong. A radar sensor is necessary for the aim. |
| **Proportionality** | Adverse effect on every child in frame vs. marginal operational gain. **Hard.** | Near-zero adverse effect. |

**The honest conclusion, and the answer to question 18(e):** the CCTV route and the mmWave route
serve the *same legitimate aim*, but they differ sharply on **necessity and proportionality**. If a
radar sensor can deliver CLEAR/MODERATE/BUSY at ₹2,000–11,000 with no personal data, then processing
children's CCTV to obtain the same enum is **difficult to defend as necessary**, and the
proportionality limb is where the CCTV approach is most likely to fail.

**This does not mean never use CCTV.** It means: **use CCTV only where the school's existing
infrastructure already emits the count as metadata and no new sensing is required** (§0, §D.5). In
that narrow case the marginal processing is genuinely minimal and the proportionality argument is
much stronger. Everywhere else, prefer a sensor that does not form an image.

### D.5 Recommended first action — a question, not a build

Before any code, the school should be asked one question:

> **"Does the existing CCTV/NVR system already have analytics enabled, and does it already expose a
> people-count or people-presence value for the relevant corridor — through ONVIF, MQTT, or an
> existing VMS API?"**

- **If yes** → the best-case architecture applies. Configure an ONVIF analytics rule on the existing
  device, expose only `CountAggregation/OccupancyCounter` (or `FieldDetector/ObjectsInside`), and
  have the robot consume that one integer. No video crosses the network. No new sensor. No new
  capture. The permission ask is one line in the school's existing CCTV policy.
- **If no** → **stop and reconsider.** The cheapest remaining path is a radar sensor at the corridor
  threshold. Building a bespoke vision pipeline on the school's CCTV, in 2026, with no published
  accuracy figure for the scenario (§B.3), against a legal framework whose operative sections are not
  in force until ~May 2027 (§C.3), is a poor trade for any problem a ₹2,000 sensor solves.

**If the school insists on the CCTV route regardless**, the minimum defensible design is:

1. Metadata-only. Subscribe to the counter topic. Never request a video stream URI.
2. Explicitly exclude `ObjectId` from the metadata filter, and treat that filter as a tested
   invariant.
3. Disable/ignore the audio track — CBSE clause 4.7.10 mandates audio-visual recording, so the
   default stream has audio and the pipeline must not decode it.
4. Sample at 1–2 fps. No frame buffer, no writer, no temp file, under a read-only filesystem.
5. Human review before any incident report. The system never files anything on its own.
6. Occupancy is advisory. It must not gate obstacle avoidance (§ brief constraint), and it must not
   be the robot's only basis for any decision with a safety consequence.
7. Document the uncertainty as a first-class output, not a footnote (§E).

---

## E. Failed-close behaviour

### E.1 What precedent exists for "unknown state" handling

**There is strong, directly transferable precedent in functional safety — and it points one way:
toward assuming the worst, not the average.**

**ISO 13849-1:2023, Safety of machinery — Safety-related parts of control systems**, has a section
titled **"6.2.3 Unknown PFH values"** in its published table of contents
(https://webstore.ansi.org/preview-pages/ISO/preview_ISO+13849-1-2023.pdf, VERIFIED as a TOC entry
from the official preview). The existence of a dedicated clause for *unknown* failure probability is
the precedent: **when you cannot establish a failure rate, the standard requires you to treat it as
the unfavourable bound rather than assume it is zero.** (INFERRED from the clause's existence — the
clause body is paywalled and we did not read it. Marked honestly.)

The related standards and their framing, VERIFIED:

- **ISO 13849** — Performance Levels **PL a → PL e**
- **IEC 62061** — **SIL CL 1 → SIL CL 3**
- **IEC 61508** — **SIL 1 → SIL 4**
- **IEC 62998** — *Safety-related sensors used for the protection of persons* — directly on point,
  since this project is proposing a sensor protecting people.
(Correspondence table and framing: https://www.therobotreport.com/wp-content/uploads/2019/06/lidar-navigation.pdf,
S. Rammer / Pilz Robotics Summit material, VERIFIED as industry presentation.)

**The applicable category for this project's status:** ISO 13482 covers *safety requirements for
service robots* and ISO/FDIS 13482 is under development; ISO 10218-1:2025 and ISO 10218-2:2025 cover
industrial robots; ISO 3691-4 covers driverless AMR systems
(https://www.iso.org/committee/5915511/x/catalogue/p/1/u/1/w/1/d/1, VERIFIED catalogue).
**INFERRED:** a school delivery robot in a corridor with children is closest to **ISO 13482**
(service robots) rather than ISO 10218. **Neither standard has been verified as applying to this
specific deployment, and neither should be cited to the school as though it does.** The point is
only that the *concept* of a declared unknown state is standard practice, not invention.

### E.2 The recommended behaviour on feed failure

**Design rule: `UNKNOWN` is a first-class value, never a default, and never silently coerced to a
safe-looking value.**

1. **`UNKNOWN` is a distinct enum member, not a sentinel.** `CLEAR | MODERATE | BUSY | UNKNOWN` —
   the brief already specifies this. Enforce it in the type system so `UNKNOWN` cannot be optimised
   away or mapped to `0`.
2. **Staleness is itself a source of `UNKNOWN`.** Track `last_valid_sample_timestamp`. If the
   estimator's output is older than a threshold, the state is `UNKNOWN` — regardless of what the last
   known value was. **A stale `CLEAR` is the most dangerous possible output**, because it actively
   discourages the robot from going anywhere.
3. **The camera being "online" is not the same as the estimate being valid.** A camera can return
   frames while the analytics engine has silently stopped updating the counter. **INFERRED:** treat
   `last_valid_sample_timestamp` as the health signal, not the transport.
4. **On `UNKNOWN`, what the robot does:** proceed on its *normal* conservative behaviour — its own
   onboard perception, its LiDAR, its normal speed limits. **`UNKNOWN` must not grant the robot
   permission to go faster or take a normal corridor it would otherwise avoid.** This is the
   fail-closed direction: an unknown corridor occupancy state should make the robot *more*
   cautious, not less.
5. **Degrade, do not oscillate.** Require **N consecutive** `UNKNOWN`s before changing mode, and
   **M consecutive** valid samples before returning to normal. Prevents a flapping feed from
   repeatedly toggling behaviour.
6. **Escalate visibly.** Sustained `UNKNOWN` should be logged as an operational fault and surfaced to
   a human, not silently retried forever.
7. **Never a safety input.** Per the brief's hard constraint and §D.5 item 6: occupancy must not
   feed obstacle avoidance. Obstacle avoidance uses the robot's own sensors, always.

### E.3 The policy clause to ask the school for

Whatever route is chosen, the school should add an explicit clause to its CCTV/robot policy. Draft,
addressed to the school's Data Fiduciary, for a competent lawyer to adapt:

> The corridor-occupancy function **receives only an anonymous aggregate count or presence flag** from
> the CCTV analytics system. It **does not receive, request, store, or process video, audio, images,
> or per-person identifiers.** It **does not subscribe to face-recognition, license-plate-recognition,
> object-identification, or tracking event topics.** It is used solely for robot routing decisions and
> **is never the sole basis for any incident report, disciplinary action, or any decision with a safety
> consequence** — such reports require human review by authorised school staff using the school's
> existing CCTV process. The system's only outputs are an integer count or a presence band, a
> timestamp, and a confidence value.

The last clause of the first sentence — **no subscription to tracking topics** — is directly modelled
on the Madill Board of Education comparator (§C.4) and is the single most important sentence in the
whole document.

### E.4 How false positives and false negatives should be measured and reported

**The asymmetry must be stated first, because it drives everything else:**

- **A false NEGATIVE** (busy corridor reported CLEAR) → the robot enters a crowded corridor. Degraded
  experience, possible awkwardness. **Usually benign.**
- **A false POSITIVE** (clear corridor reported BUSY) → the robot refuses or reroutes a clear corridor.
  Degraded service, operator frustration, and — over time — **the system gets switched off**, which
  destroys the value entirely.

**Neither is a safety event, because occupancy does not gate obstacle avoidance.** That design
property is what makes this safe to deploy, and it is why the measurement burden can be modest.

**Measurement method — no real student footage, per the brief:**

1. **Consented adult cohort only.** Never measure on students. Recruit volunteers (staff, or adult
   visitors under a written consent) to occupy the corridor in known configurations. **This is not
   optional and it is the brief's constraint as well as the ethical requirement.**
2. **Label by configuration, not by individual.** Ground truth is *"4 adults present in the far
   third of the corridor"* — a scene descriptor. No per-person annotation, no tracking, nothing that
   could reconstruct identities.
3. **Stratified grid.** Sample across: empty / 1 / 2–3 / 4–6 / 7+ across near / middle / far zones;
   across lighting states (morning, midday, fluorescent-only, overcast); across clothing types;
   with and without static vs. moving occupants. **The static-vs-moving axis is the one that will
   break a background-subtraction approach (§B.2), so it must be explicitly populated.**
4. **Report the confusion matrix, weighted for the asymmetry above.** Report FP and FN separately,
   never as a single accuracy number. A single accuracy figure hides exactly the failure mode that
   matters here.
5. **Report a confidence-conditional metric.** Because the system must be able to say `UNKNOWN`,
   report coverage-versus-error as a curve: *at what confidence threshold does the error rate fall
   below what is acceptable, and what fraction of the time is the system below that threshold?* If the
   answer is "accurate 60% of the time", the honest `UNKNOWN` design is what makes the system viable
   at all — and that is an argument **for** the `UNKNOWN` state, not against it.
6. **Report latency and uptime separately**, with the stale-threshold behaviour exercised
   deliberately (unplug the stream; confirm `UNKNOWN`, not stale `CLEAR`).
7. **Re-measure after any change** — camera move, lighting change, corridor refit, NVR firmware
   update, or model replacement. Occupancy estimators are geometry-dependent; a corridor reconfiguration
   invalidates the measurement.

---

## What we could not determine

Stated plainly, because the brief asked for honesty over completeness.

| Question | Status |
|---|---|
| Published accuracy of classical OpenCV people detection in a cluttered school corridor | **NOT FOUND.** No benchmark exists. Do not let anyone supply one from memory. |
| OpenCV HOG tutorial's own miss-rate/FPPI figures | **UNVERIFIED.** Tutorial page 404'd; docs.opencv.org served a Cloudflare challenge. Not quoted. |
| Real-world success rate of guessing a standard RTSP URL | **NOT FOUND.** No credible survey. Use ONVIF discovery instead of guessing. |
| Whether an NVR must re-stream a channel to a third-party RTSP client | **NOT FOUND** in any specification. It is a per-vendor feature. |
| Typical authentication configuration of Indian school NVRs | **UNVERIFIED.** No survey located. |
| Whether Indian wireless-equipment certification permits a given 60 GHz module in a school | **NOT FOUND.** Must be checked before procurement. |
| Supreme Court of India guidance on CCTV in educational institutions | **NOT FOUND.** A real SC order on educational institutions exists (25 Jul 2025) but concerns mental health, not CCTV. |
| Primary text of the Uttarakhand High Court CCTV-in-schools direction | **UNVERIFIED** — located only via a secondary commercial summary. |
| Whether "in the interests of safety of children" (DPDP Rules Fourth Schedule Part A entry 3(b)) covers corridor-occupancy routing | **NOT FOUND / legal question.** Requires a competent Indian data-protection opinion. Not resolvable from the text. |
| Whether this school's existing CCTV/NVR already emits a people count | **UNKNOWN — and it is the first thing to ask.** |

---

## Reproducing this research

Primary sources, in order of evidential weight:

- ONVIF Profile M Specification v1.1 (March 2024) — `https://www.onvif.org/wp-content/uploads/2024/04/onvif-profile-m-specification-v1-1.pdf`
- ONVIF Analytics Service Specification v26.06 — `https://www.onvif.org/specs/srv/analytics/ONVIF-Analytics-Service-Spec.pdf`
- ONVIF Streaming Specification v26.06 — `https://www.onvif.org/specs/stream/ONVIF-Streaming-Spec.pdf`
- ONVIF Conformance Process v5.7 — `https://www.onvif.org/wp-content/uploads/2026/04/ONVIF-Conformance-Process-Specification-v5-7.pdf`
- ONVIF "End Support for Profile S" release, 9 Oct 2025 — `https://www.onvif.org/?p=8621&post_type=pressrelease`
- DPDP Act 2023 — `https://www.meity.gov.in/static/uploads/2024/02/Digital-Personal-Data-Protection-Act-2023.pdf`
- DPDP Act commencement, G.S.R. 843(E), 13 Nov 2025 — `https://www.meity.gov.in/static/uploads/2025/11/c56ceae6c383460ca69577428d36828b.pdf` (MeitY copy of the instrument; `egazette.gov.in` failed TLS here)
- DPDP Rules 2025, G.S.R. 846(E), 13 Nov 2025 — `https://www.meity.gov.in/static/uploads/2025/11/53450e6e5dc0bfa85ebd78686cadad39.pdf`
- IT Act 2000 §66E — `https://www.indiacode.nic.in/bitstream/123456789/13116/1/it_act_2000_updated.pdf`
- CBSE Affiliation Bye-Laws cl. 4.7.10, 21 Jul 2025 — `https://www.cbse.gov.in/cbsenew/documents/Amendments_Chapter_4_21072025.pdf`
- Chan, Liang & Vasconcelos, CVPR 2008 — `https://ieeexplore.ieee.org/document/4587569`
- OpenCV 4→5 migration guide — `https://github.com/opencv/opencv/wiki/OpenCV-4-to-5-migration`
- RFC 2326 / RFC 7826 — `https://www.rfc-editor.org/info/rfc2326/`, `https://www.rfc-editor.org/rfc/rfc7826.html`

Full source table with types, claims supported and limitations: **[`../research/cctv_source_register.md`](../research/cctv_source_register.md)**.