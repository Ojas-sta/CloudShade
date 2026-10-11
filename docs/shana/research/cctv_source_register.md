# SHANA — CCTV Occupancy Research: Sources

Every source consulted for the corridor-occupancy feasibility study.
**All accessed 2026-10-10** unless otherwise noted.

**Source types:** `STD` (standard / specification) · `OFFICIAL-GOV` (government / regulator /
eGazette) · `OFFICIAL-BOARD` (examination board, school regulator) · `LEGAL` (statute) ·
`PEER-REVIEWED` · `OFFICIAL-CODE` (project documentation / official docs) ·
`OFFICIAL-MFR` (manufacturer) · `INDEPENDENT` (third-party test, review, forum) ·
`AGGREGATOR` (commercial summary or price tracker).

---

## Method and its limits

1. **Primary specification PDFs were downloaded and read locally**, then searched as text, not
   skimmed from search snippets. Specifically: `onvif-profile-m-specification-v1-1.pdf`,
   `ONVIF-Analytics-Service-Spec.pdf`, `ONVIF-Streaming-Spec.pdf`, and
   `Digital-Personal-Data-Protection-Act-2023.pdf`. **Negative findings (e.g. "occupancy" does not
   occur in Profile M v1.1; "CCTV"/"surveillance"/"biometric" do not occur in the DPDP Act or
   Rules) are full-text search results, not impressions.**
2. **`docs.opencv.org` blocked direct fetch** with a Cloudflare interstitial on the tutorial pages;
   one URL (`4.x/d6/dc3/tutorial_py_people_detector.html`) returned **404**. OpenCV claims in this
   study are therefore sourced from the **official OpenCV GitHub wiki and `opencv.org`**, not from
   the docs site tutorial pages.
3. **`egazette.gov.in` failed TLS certificate verification** in this environment (curl exit 60) and
   could not be retrieved. The DPDP Act commencement notification **G.S.R. 843(E)** was therefore
   read from the **MeitY-hosted copy of the same instrument**. The DPDP Rules **G.S.R. 846(E)** were
   read from **both** the MeitY-hosted original and a secondary full-text mirror, with the
   Fourth Schedule wording confirmed independently in the official PDF. Both retrievals succeeded.
4. **`indiacode.ecourtsindia.com` returned 403** to direct fetch. §66E text was obtained from the
   IndiaCode-hosted bare Act PDF and cross-checked against Indian Kanoon.
5. **No camera was connected to. No RTSP stream was opened. No CCTV footage was viewed. No
   software was installed. No network was probed. No credentials were used.** This is a
   documentation study.
6. **No legal advice is given.** The Fourth Schedule exemption analysis (§C.3) identifies the
   provision and quotes it verbatim; whether it covers this use case is expressly left to a
   competent Indian data-protection lawyer.
7. **CONTRADICTIONS are recorded in §5, not silently resolved.**

---

## 1. ONVIF — profiles and services (STD)

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 1 | ONVIF Profiles index | `https://www.onvif.org/profiles/` | STD | Conformance model: *"Conformance to profiles is the only way that ensures compatibility between ONVIF conformant products; therefore, only registered products with conformance to a profile are considered to be ONVIF conformant."* Profile set A/C/D/G/M/S/T/V; **Profile Q deprecated 2022-04-01**; profile vs. add-on distinction | Marketing/informational page |
| 2 | **ONVIF Profile M Specification v1.1**, March 2024 | `https://www.onvif.org/wp-content/uploads/2024/04/onvif-profile-m-specification-v1-1.pdf` | STD | **THE central source.** §4 requires ONVIF NI Specs ≥ 21.06. §7.5.3 **"Metadata Streaming Device MANDATORY"**; §8.2.3 **"Video Streaming Device CONDITIONAL"**. §8.15 Line crossing counter topic `tns1:RuleEngine/CountAggregation/Counter`, data `Count [xs:int]`, device+client CONDITIONAL. §7.6 rule configuration operations. §8.5 MQTT event handling. §8.13 face recognition event (CONDITIONAL), §8.14 LPR event (CONDITIONAL), §8.8 human face metadata, §8.9 human body metadata. Revision history: v1.1 changed only 7.8.2 (Get/Set MetadataConfiguration → Optional) | Read in full locally. **Full-text search for "occupancy" returns ZERO matches** |
| 3 | **ONVIF Analytics Service Specification v26.06**, June 2026 | `https://www.onvif.org/specs/srv/analytics/ONVIF-Analytics-Service-Spec.pdf` | STD | **Appendix A.6 "Occupancy Counting rule"**: topic `tns1:RuleEngine/CountAggregation/OccupancyCounter`, data `Count [xs:int]`; parameters `Segments` (unbounded `tt:Polyline`), `ReportTimeInterval`, `ResetTime`. **A.5 "Line crossing counting rule"** topic `CountAggregation/Counter`. **A.3 "Field Detector"** topic `FieldDetector/ObjectsInside`, data `IsInside [xs:boolean]`, *"True if one or more objects matching the parameters are inside"*, with note *"a device signaling ObjectId shall emit changed events whenever the list of objects changes."* A.4 Loitering Detector. A.7 Object Detection. A.2 Line Detector. Appendix B motion/cell-motion. Appendix C motion region detector | Read locally. These are **appendix rule definitions**, not mandated profile features |
| 4 | ONVIF Streaming Specification v26.06 | `https://www.onvif.org/specs/stream/ONVIF-Streaming-Spec.pdf` | STD | Metadata stream transport: SDP encoding names `vnd.onvif.metadata`, `vnd.onvif.metadata+gzip`, `vnd.onvif.metadata.exi.ext`; **SDP media type `"application"` for metadata streams**; *"The metadata streaming container format allows well-defined, real-time streaming of analytics, PTZ status and notification data"*; *"Only UTC timestamps shall be used within the metadata stream"*; container carries `VideoAnalyticsStream`; RTP clock rate 90000 Hz; default encoder Timeout 60 s | Earlier version v2.5 also read; both consistent |
| 5 | ONVIF Streaming Specification v2.5 (dated copy) | `https://www.onvif.org/specs/2206/ONVIF-Streaming-Spec-v2206.pdf` | STD | *"Media control is accomplished over RTSP as defined in RFC 2326"*; *"A set of media streaming (audio, video and meta data) options, all based on RTP [RFC 3550]"* | Used for the RFC 2326 / RFC 7826 tension (§5) |
| 6 | ONVIF Core Specification v26.06 | `https://www.onvif.org/specs/2606/ONVIF-Core-Spec-v2606.pdf` | STD | §5.9.1 Authentication over HTTP and HTTPS; §5.9.5 Username token profile; §9.5/9.6 event handling, topic structure, WS-BaseNotification | TOC-level reading |
| 7 | ONVIF Conformance Process Specification v5.7, April 2026 | `https://www.onvif.org/wp-content/uploads/2026/04/ONVIF-Conformance-Process-Specification-v5-7.pdf` | STD | Conformance claim mechanics; normative reference index; supersession of v5.6 from 2026-04-22 | — |
| 8 | ONVIF Profiles Conformance Device Test Spec v26.06 | `https://www.onvif.org/wp-content/uploads/2026/07/ONVIF_Profiles_Conformance_Device_Test_Specification_26.06.pdf` | STD | Conformance test procedures per profile; discovery added to Profile C/D/G support checks | — |
| 9 | ONVIF Profile S Specification v1.3, Nov 2019 | `https://www.onvif.org/wp-content/uploads/2019/12/ONVIF_Profile_-S_Specification_v1-3.pdf` | STD | §7 Profile Mandatory Features: *"User Authentication — WS-Usernametoken using timestamps and nonce — HTTP Digest"*; §7.7.1 WS-BaseNotification and pull-point support; §8.9 multicast conditional | Historical; S is being retired (§2) |
| 10 | ONVIF Profile M overview page | `https://www.onvif.org/profiles/profile-m/` | STD | Profile M scope; use cases: *"For visitor statistics, crowd control, visitor management or queue management through people counting analytics"*; MQTT room-temperature example; Profile M combines with S/T for video and A/C/D for access control | Marketing/informational |
| 11 | ONVIF Profile M summary deck (May 2022) | `https://www.onvif.org/wp-content/uploads/2022/05/onvif-profile-m.pdf` | STD | Profile M vs Profile T feature matrix — **"Metadata streaming M M"** vs **"Video streaming C C"** (M=mandatory, C=conditional) | Confirms #2 independently |
| 12 | ONVIF Profile Feature Overview v2.6 (Apr 2022) | `https://www.onvif.org/wp-content/uploads/2022/04/onvif-profile-feature-overview.pdf` | STD | Cross-profile feature comparison; M/C notation | Predates Profile M v1.1 |
| 13 | ONVIF Real Time Streaming Device Test Spec v20.06 | `https://www.onvif.org/wp-content/uploads/2021/01/ONVIF_Real_Time_Streaming_Device_Test_Specification_20.06.pdf` | STD | RTSP control + RTP transport options: RTP Unicast/UDP, RTP over RTSP over TCP, RTP over RTSP over HTTP over TCP; explicitly **excludes** HTTP/HTTPS/RTP/RTSP protocol conformance testing | Documents what ONVIF does *not* test |
| 14 | ONVIF Network Interface Specifications index | `https://www.onvif.org/profiles/specifications/` | STD | Canonical list of 29 service specs (Core, Streaming, Analytics, Media2, Event, Security, Thermal, WebRTC, …) | — |

## 2. ONVIF — press releases and corporate (OFFICIAL-MFR / press)

| # | Title | URL | Type | Claims supported |
|---|---|---|---|---|
| 15 | **"ONVIF to End Support for Profile S; Recommends Profile T as Replacement"**, 9 Oct 2025 | `https://www.onvif.org/?p=8621&post_type=pressrelease` | OFFICIAL-MFR | Profile S end support announced. *"Profile S, however, specifies authentication mechanisms that are no longer consistent with current cybersecurity recommendations."* *"After 14 years… more than 33,000 conformant devices and clients."* Leo Levit quote. *"ONVIF strongly encourages, if possible, the discontinuation of the use of the username token authentication in Profile S."* *"The majority of ONVIF conformant devices and clients available on the market today support both ONVIF Profile S and Profile T."* |
| 16 | ONVIF press release (canonical slug) | `https://www.onvif.org/pressrelease/onvif-to-end-support-for-profile-s` | OFFICIAL-MFR | Same announcement, canonical URL |
| 17 | ONVIF Overview deck, October 2025 | `https://www.onvif.org/wp-content/uploads/2025/10/onvif-overview-20251009.pdf` | OFFICIAL-MFR | **"ONVIF to end support for Profile S on March 31, 2027."** Profile at-a-glance: T = advanced video streaming; G = edge storage and retrieval; M = metadata and events for analytics |
| 18 | SDM Magazine coverage of the release | `https://www.sdmmag.com/articles/104741-onvif-to-end-support-for-profile-s-recommends-profile-t-as-replacement` | INDEPENDENT | *"The June 2026 version of the ONVIF conformance test tools, which have a nine-month validity period, will be the last test tool version that enables manufacturers to claim product conformance to Profile S."* *"Introduced in 2018, Profile T contains virtually all the features of Profile S, in addition to other advanced features."* Confirms TLS Add-on upgrade end-2026 |
| 19 | Security Systems News coverage | `https://www.securitysystemsnews.com/article/onvif-ending-support-for-profile-s-recommends-profile-t` | INDEPENDENT | Corroborates #18 verbatim | Trade press |
| 20 | ONVIF Profile M release-candidate press release | `https://www.onvif.org/pressrelease/onvif-introduces-profile-m-release-candidate-for-metadata-and-analytics-for-smart-applications` | OFFICIAL-MFR | Profile M genesis: *"standardizes the communication of metadata and event handling of analytics for smart applications"* | Historical |
| 21 | ONVIF at ISC West 2026 exhibitor presentation (third-party host) | `https://pub-mediabox-storage.rxweb-prd.com/exhibitor/document/exh-278d072d-7de4-4edb-a7ec-7b7565bf0008/f0280d7d-de0d-44aa-8f27-c04d031c6be7.pdf` | INDEPENDENT | *"25,000+ conformant products, 500+ are Profile M conformant"* — useful adoption-scale datapoint | Third-party host; not ONVIF-controlled. **Profile M adoption is ~2% of the conformant population** |

## 3. RTSP (STD)

| # | Title | URL | Type | Claims supported |
|---|---|---|---|---|
| 22 | **RFC 2326 — Real Time Streaming Protocol**, Apr 1998 | `https://www.rfc-editor.org/info/rfc2326/` · `https://www.rfc-editor.org/rfc/rfc2326.txt` | STD | *"This RFC is now obsolete, see RFC 7826."* Status: Standards Track. *"The Real Time Streaming Protocol, or RTSP, is an application-level protocol for control over the delivery of data with real-time properties."* |
| 23 | **RFC 7826 — RTSP Version 2.0**, Dec 2016 | `https://www.rfc-editor.org/rfc/rfc7826.html` | STD | *"RTSP 2.0 is a replacement of RTSP 1.0 [RFC2326] and this document obsoletes that specification. This protocol is based on RTSP 1.0 but is not backwards compatible other than in the basic version negotiation mechanism."* Standards Track |
| 24 | IETF Datatracker entries | `https://datatracker.ietf.org/doc/rfc2326` · `https://datatracker.ietf.org/doc/rfc7826` | STD | Status corroboration | — |

## 4. OpenCV (OFFICIAL-CODE)

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 25 | **OpenCV 4 to 5 migration guide** | `https://github.com/opencv/opencv/wiki/OpenCV-4-to-5-migration` | OFFICIAL-CODE | **"Objdetect: Haar and HOG moved to contrib"**; `cv::CascadeClassifier` and `cv::HOGDescriptor` moved to `xobjdetect`; Python now requires `opencv_contrib`; C++17 minimum | Migration guide, authoritative for the move |
| 26 | **OpenCV 5 release notes wiki** | `https://github.com/opencv/opencv/wiki/OpenCV-5` | OFFICIAL-CODE | **"Objdetect module cleanup. Haar-based and HOG-based detectors have been moved to opencv_contrib (xobjdetect). Modern deep learning-based detectors are both faster and more accurate."** DNN module "largest set of changes since OpenCV 4.x"; ONNX Runtime engine, CPU-only for now. Two active branches (4.x and 5.x) | Project's own assessment — treat as opinion, but it is the maintainers' opinion |
| 27 | OpenCV changelog wiki | `https://github.com/opencv/opencv/wiki/OpenCV-Change-Logs` | OFFICIAL-CODE | **Licence: "Recent OpenCV 4.x (since OpenCV 4.5.0) and also OpenCV 5.0 are now distributed under Apache 2 license. We used BSD license before…"** | — |
| 28 | OpenCV releases page | `https://opencv.org/releases/` | OFFICIAL-CODE | **OpenCV 5.0.0 released 2026-08-19**; 4.14.0 on 2026-07-19; 4.13.0 on 2025-12-31 | — |
| 29 | OE-5. OpenCV 5 enhancement proposal | `https://github.com/opencv/opencv/wiki/OE-5.-OpenCV-5` | OFFICIAL-CODE | Licence change rationale (Apache 2, patent protection); API cleanup rationale | Proposal doc |
| 30 | `cv::VideoCapture` class reference (4.13.0) | `https://docs.opencv.org/4.13.0/d8/dfe/classcv_1_1VideoCapture.html` | OFFICIAL-CODE | *"Opens a video file or a capturing device or an IP video stream for video capturing with API Preference."* *"Note that each video stream or IP camera feed has its own URL scheme."* `apiPreference` values incl. `cv::CAP_FFMPEG`, `cv::CAP_IMAGES`, `cv::CAP_DSHOW` | **Page reached only via search index; docs.opencv.org served Cloudflare challenges to direct fetch** |
| 31 | `cv::HOGDescriptor` reference (4.13.0) | `https://docs.opencv.org/4.13.0/d5/d33/structcv_1_1HOGDescriptor.html` | OFFICIAL-CODE | `getDefaultPeopleDetector()` (64×128), `getDaimlerPeopleDetector()` (48×96); `detect()`/`detectMultiScale()`; `winSize` default `Size(64,128)`. Dalal & Triggs CVPR 2005 cited as the source algorithm | Same access caveat |
| 32 | `xobjdetect` group docs (5.0.0-alpha) | `https://docs.opencv.org/5.0.0-alpha/d4/d54/group__xobjdetect.html` | OFFICIAL-CODE | `xobjdetect` module contains "Cascade Classifier for Object Detection" and "HOG (Histogram of Oriented Gradients) descriptor and object detector" — confirms the contrib destination | Alpha docs |
| 33 | OpenCV HOG people-detector tutorial | `https://docs.opencv.org/4.x/d6/dc3/tutorial_py_people_detector.html` | OFFICIAL-CODE | — | ⛔ **404 NOT RETRIEVED.** Its widely-quoted miss-rate/FPPI figures are therefore **UNVERIFIED** and are **not** cited in the report |
| 34 | OpenCV background-subtraction tutorial | `https://docs.opencv.org/4.x/d1/dc5/tutorial_py_background_subtraction.html` | OFFICIAL-CODE | MOG2 / KNN usage | ⛔ **Cloudflare interstitial to direct fetch.** MOG2/KNN presence in `bgsegm` is **NOT independently verified in this pass** — treat as common knowledge requiring confirmation |

## 5. Indian statute, rules and regulation (OFFICIAL-GOV / LEGAL / OFFICIAL-BOARD)

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 35 | **Digital Personal Data Protection Act, 2023** (No. 22 of 2023, 11 Aug 2023) | `https://www.meity.gov.in/static/uploads/2024/02/Digital-Personal-Data-Protection-Act-2023.pdf` | LEGAL | **§2(t)** *"personal data means any data about an individual who is identifiable by or in relation to such data"*; **§2(x)** definition of "processing"; **§3** Application — reproduced in full, **three paragraphs only, no State/surveillance/law-enforcement exemption**; **§4** lawful grounds (consent or legitimate uses); **§5** notice; **§6** consent; **§8** general obligations incl. 8(8) storage limitation and 8(9)/8(10); **§9** children — (1) verifiable parental consent, (2) no detrimental effect on well-being, **(3) no tracking or behavioural monitoring of children**, (4) exemptions as prescribed, (5) "verifiably safe" notification; penalties in the Schedule (up to ₹250 crore for §9 breach) | Read in full locally. **Full-text search: "CCTV", "surveillance", "biometric", "facial", "face recognition" — ZERO occurrences.** Only "video" hit is the live-video KYC illustration |
| 36 | **Commencement notification G.S.R. 843(E)**, 13 Nov 2025, Gazette No. 757 | Official: `https://egazette.gov.in/WriteReadData/2025/267647.pdf` · **MeitY mirror (RETRIEVED AND READ LOCALLY):** `https://www.meity.gov.in/static/uploads/2025/11/c56ceae6c383460ca69577428d36828b.pdf` | OFFICIAL-GOV | Verbatim three-limb commencement schedule. **(a)** immediate: §1(2), §2, §§18–26, §35, §§38–43, §44(1)&(3). **(b)** +1 year: §6(9), §27(1)(d). **(c) +18 months: §§3–5, §6(1)–(8)&(10), §§7–17**, §27 except (1)(d), §§28–34, §§36–37, §44(2). F. No. AA-11038/1/2025-CL&ES; signed Ajit Kumar, Jt. Secy. | ⚠️ **`egazette.gov.in` FAILED TLS verification in this environment (curl exit 60) and could not be retrieved.** The text above was read from the **MeitY-hosted copy**, which is the same instrument on a different government host. The notification says "eighteen months from the date of publication", **not a calendar date** — derived 13 May 2027 is **INFERRED arithmetic**; the eighteen-month rule itself is VERIFIED |
| 37 | Commencement — independent tracker noting the derivation caveat | `https://dpdprules.org/act/3` · `https://dpdprules.org/act` | AGGREGATOR | Section 3 status "Not yet in force", commencement "13 May 2027"; explicitly flags: *"The computed calendar date, 13 May 2027, is interpretation until officially confirmed"* | Third-party. Corroborates the INFERRED label rather than overriding it |
| 38 | **Digital Personal Data Protection Rules, 2025**, G.S.R. 846(E), 13 Nov 2025 | **MeitY official (RETRIEVED AND READ LOCALLY):** `https://www.meity.gov.in/static/uploads/2025/11/53450e6e5dc0bfa85ebd78686cadad39.pdf` · secondary full-text mirror also read: `https://www.dpdpa.com/DPDP_Rules_2025_English_only.pdf` | OFFICIAL-GOV | **Rule 1(2)–(4)** commencement (immediate / +1yr Rule 4 / **+18mo Rules 3, 5–16, 22, 23**); **Rule 10** verifiable consent for children's data; **Rule 11** for persons with disability; **Rule 12** exemptions from §9(1)&(3); **FOURTH SCHEDULE PART A entry 3** — educational institution exemption, conditions *(a) educational activities / (b) interests of safety of children enrolled*; entries 1, 2 (health), 4 (crèche), 5 (transport); **PART B** purposes; Note (c) definition of "educational institution"; **Third Schedule** erasure timeline | Full text read from **both** the MeitY official PDF and the secondary mirror; **the Fourth Schedule Part A entry 3 wording and the Note (c) definition were confirmed independently in the MeitY-hosted PDF.** Full-text search: "CCTV", "surveillance", "biometric", "facial", "face", "video", "image" — only hit is "payment interface." |
| 39 | PIB release, 14 Nov 2025 | `https://www.pib.gov.in/PressReleseDetailm.aspx?PRID=2190014` · `https://www.pib.gov.in/PressReleasePage.aspx?PRID=2190655` | OFFICIAL-GOV | Confirms Rules notified 13/14 Nov 2025; 18-month phased compliance; 6,915 consultation inputs; Data Protection Board of India; 90-day response deadline | **About the Rules, not the Act's commencement — see #37 correction note** |
| 40 | **Information Technology Act, 2000**, §66E | `https://www.indiacode.nic.in/bitstream/123456789/13116/1/it_act_2000_updated.pdf` · cross-check `https://indiankanoon.org/doc/112223967` | LEGAL | Full §66E text incl. Explanation. **§66E(c): "'private area' means the naked or undergarment clad genitals, public area, buttocks or female breast"**; (a) transmit; (b) capture; (d) publishes; (e) under circumstances violating privacy. **Act No. 21 of 2000, in force 17 Oct 2000** | `indiacode.ecourtsindia.com` returned **403** to direct fetch |
| 41 | UNODC SHERLOC mirror of §66E | `https://www.unodc.org/cld/en/legislation/ind/the_information_technology_act_2000/chapter_xi/section_66e/section_66e.html` | OFFICIAL-GOV | Confirms §66E entry into force 2000-10-17; corroborates text | UN mirror, source: India Code |
| 42 | **CBSE Affiliation Bye-Laws amendment — clause 4.7.10**, Notification CBSE/AFF./Notification/2025/0, 21 Jul 2025 | `https://www.cbse.gov.in/cbsenew/documents/Amendments_Chapter_4_21072025.pdf` | OFFICIAL-BOARD | **Verbatim clause 4.7.10**: CCTV with audio-visual facility at entry/exit, lobbies, **corridors**, staircases, classrooms, labs, library, canteen, store room, playground, common areas **"except toilets and washrooms"**, with **"real time audio-visual recording"**, storage of **at-least 15 days**, accessible by authorities. Grounds the amendment in NCPCR Manual cl. 1(X). Signed Himanshu Gupta, Secretary | The single most operationally relevant regulatory fact in this study. **Scope: CBSE-affiliated schools only** |
| 43 | CBSE Affiliation Bye-Laws index | `https://www.cbse.gov.in/cbsenew/aff_bye_laws.html` | OFFICIAL-BOARD | Confirms the chapter/circular publication structure; page last updated 2026-07-07 | — |
| 44 | NCPCR *Manual on Safety and Security of Children in Schools*, cl. 1(X) | cited within #42 | OFFICIAL-BOARD (secondary quote) | *"CCTV in schools needs to be monitored and maintained regularly"* | **Primary NCPCR manual NOT retrieved.** Quoted only as reproduced in the CBSE notification |
| 45 | **Supreme Court of India order, 25 Jul 2025** (*Suhaas Chaturvedi* line of proceedings) | `https://api.sci.gov.in/supremecourt/2024/18489/18489_2024_3_1502_62660_Order_25-Jul-2025.pdf` | LEGAL | Confirms SC issues binding guidelines to **all educational institutions** — but on **mental health policy**, counsellors, student-to-counsellor ratios, referral protocols. **Says nothing about CCTV.** | Recorded specifically so it is **not** mistaken for authority on CCTV |
| 46 | Commercial summary of Indian court directions on school CCTV | `https://supremetoday.ai/issue/mandatory-cctv-surveillance-indian-school-legal-compliance` | AGGREGATOR | Reports a Uttarakhand direction (*"Every Private School in the State of Uttarakhand is ordered to install CCTV cameras in the school rooms, corridors and every corner…"*) and a 2017 Directorate of Education mandate | ⛔ **UNVERIFIED.** Primary judgments not retrieved. Cited only as evidence of *direction of travel*, never as authority |
| 47 | *K.S. Puttaswamy (Retd.) v. Union of India* (2017) 10 SCC 1 | referenced in #48 and general constitutional law | LEGAL | Right to privacy under Article 21; four-part test (legality / legitimate aim / necessity / proportionality) | **Full judgment NOT retrieved in this pass.** Framework is well-established; cite with care |
| 48 | Legal commentary on Puttaswamy + facial recognition | `https://www.legalserviceindia.com/Legal-Articles/delhi-police-facial-recognition-protests-supreme-court-privacy` | INDEPENDENT | Notes a pending SC challenge on FR/biometric surveillance; *"As of 18 August 2026, the reported development is a Delhi Police affidavit/counter-affidavit and an ongoing constitutional challenge, not a final Supreme Court judgement"* | Legal commentary, not authority. Useful for confirming FR is actively contested |

## 6. mmWave / people-presence sensing (OFFICIAL-MFR / INDEPENDENT)

| # | Title | URL | Type | Claims supported |
|---|---|---|---|---|
| 49 | **HI-Link HLK-LD6001C** product page | `https://www.hlktech.net/index.php?id=1312` | OFFICIAL-MFR | **In/out people counting up to 8** at a doorway; ceiling install; **0.5–8 m**, ±60° az/el, **2.7 m install height**, 3.5 m ground-projection radius; 60–64 GHz; **processing cycle ≤30 ms**; TTL serial; 29.36×28 mm; 1.7 W peak / 0.3 W avg; listed **$28.05** |
| 50 | **TI IWRL6432WMOD** datasheet | `https://www.ti.com/lit/ds/symlink/iwrl6432wmod.pdf` · `https://www.ti.com/product/IWRL6432WMOD` | OFFICIAL-MFR | 60 GHz, 57–61.5 GHz; ±60° FoV both axes; 31×15.5 mm; presence detection 15 m @0°, 8 m @ FoV edge; **"Modular certification with FCC, RED, MIC certifications"**; presence/motion via GPIO or SPI point-cloud. **Not priced** (quote-only) |
| 51 | **TI TIDEP-01000** people-counting reference design | `https://www.ti.com/tool/TIDEP-01000` · `https://www.ti.com/lit/pdf/tidue71` | OFFICIAL-MFR | *"can detect up to 250 objects, and point and track up to 20 people with a field of view (FOV) of ±60º in the azimuth plane"*; detect and track to 15 m; IWR6843 ISK; *"All detection and signal processing handled onboard"* |
| 52 | **Minew MSR01** poster | `https://www.minew.com/wp-content/uploads/2023/08/MSR01-poster.pdf` | OFFICIAL-MFR | 60 GHz 4T4R, 120°; *"Human Presence Detection, People Flow Statistics, People Counting"*; **"100% Anonymous Detection"**; *"Supporting non-contact and anonymous detection, MSR01 helps protect privacy"*; Wi-Fi + BLE; 83×27.1 mm; DC 12V/1A |
| 53 | **Aqara FP1E** | `https://www.aqara.com/eu/product/presence-sensor-fp1e` · listing `https://www.domadoo.fr/en/smart-home-security-alarm/7216-aqara-aqara-presence-sensor-zigbee-30-rtczcgq11lm-6975833356478.html` | OFFICIAL-MFR / AGGREGATOR | **Presence only** (moving/still + duration); 60–61 GHz; 120° FoV; 6 m; **200 ms response**; 360° rotating head; Zigbee 3.0 + Matter over bridge; 57×50×37 mm; **€49.99** |
| 54 | **Aqara FP2** | `https://www.aqara.com/us/product/presence-sensor-fp2` | OFFICIAL-MFR | *"ability to detect up to 5 people at the same time"*; up to 30 zones; ceiling-mount fall detection; IWR6843; Wi-Fi 2.4 GHz; 5V1A continuous power |
| 55 | Aqara FP2 **India price** | `https://pricehistory.app/p/aqara-presence-sensor-fp2-mmwave-radar-wired-oNE3lpw8` | AGGREGATOR | **Amazon.in ₹11,128 on 2026-09-07**; range ₹9,569 (low) – ₹20,000 (high); avg ₹13,222. ⚠️ **Price tracker, not a quote** |
| 56 | **Apollo Automation mmWave comparison chart** | `https://wiki.apolloautomation.com/products/general/mmwave-presence-sensor-comparison` | INDEPENDENT | Cross-device table of price / chip / detection distance / zones / target-tracking count for MSR-2, EP1 Lite, MTR-1, Screek 1U & 2A, Aqara FP1 & FP2. **Target tracking: LD2410-class = 1; LD2450 = 2–3 ("3 not great")** |
| 57 | Home Automation Central review | `https://homeautocentral.com/best-mmwave-presence-sensor` | INDEPENDENT | Price bands: EP1 $30–40, FP2 $80–100, MR24HPC1 $20–30, **LD2410B/LD2450 modules $5–15**, FP1 $60–80. Candid: *"The firmware on the generic Tuya-branded versions is inconsistent, the sensitivity drifts over weeks, and the configuration tooling is fiddly…"* *"The honest cost of a bare module is the chip price plus your weekend, and your weekend isn't free."* |
| 58 | MikroE Radar Click (MM5D91-00) | `https://www.mikroe.com/radar-click` | OFFICIAL-MFR | Presence module *"counts the number of people entering or exiting an entrance"*; 60 GHz; 1Tx/3Rx; ±45°/±40°; 10 m macro / 5 m micro motion; **$196** (1 qty) |

## 7. Algorithms — peer-reviewed (PEER-REVIEWED)

| # | Title | URL | Type | Claims supported |
|---|---|---|---|---|
| 59 | **Chan, Liang & Vasconcelos. "Privacy Preserving Crowd Monitoring: Counting People without People Models or Tracking." CVPR 2008.** DOI 10.1109/CVPR.2008.4587569 | `https://ieeexplore.ieee.org/document/4587569` | PEER-REVIEWED | **THE canonical privacy-preserving crowd-counting paper.** *"We present a privacy-preserving system for estimating the size of inhomogeneous crowds… without using explicit object segmentation or tracking."* Cited **1,576** times. Validated on **49,885 pedestrian instances** + one hour of video |
| 60 | UCSD SVCL crowd-counting project page | `http://svcl.ucsd.edu/projects/peoplecnt` | PEER-REVIEWED (lab) | The thesis in full: *"there is in fact no need for pedestrian detection, object tracking, or object-based image primitives to accomplish the pedestrian counting goal"*; *"it can be implemented with hardware that does not produce a visual record of the people in the scene, i.e. with special-purpose cameras that output low-level features"* |
| 61 | UCSD SVCL publications list | `http://www.svcl.ucsd.edu/publications` | PEER-REVIEWED (lab) | Full citation with venue and date |
| 62 | Dalal & Triggs. "Histograms of Oriented Gradients for Human Detection." CVPR 2005 | `https://www.cs.princeton.edu/courses/archive/fall13/cos429/papers/Dalal05.pdf` · `https://ieeexplore.ieee.org/document/1467360` | PEER-REVIEWED | The HOG + linear-SVM method underlying `cv2.HOGDescriptor`. **Explicit scope limitation: *"taking 'pedestrian detection' (the detection of mostly visible people in more or less upright poses) as a test case"*** — outdoor datasets (INRIA Person, PASCAL VOC) |
| 63 | **Li, Zhang & Chen. "CSRNet: Dilated Convolutional Neural Networks for Understanding the Highly Congested Scenes." CVPR 2018** | `https://openaccess.thecvf.com/content_cvpr_2018/papers/Li_CSRNet_Dilated_Convolutional_CVPR_2018_paper.pdf` · `https://arxiv.org/html/1802.10062v2` | PEER-REVIEWED | Density-map counting. MAE 68.2 ShanghaiTech A / **10.6 Part B**; PSNR 23.79 / 0.76; 16.2M params (per #64). *"the CSRNet is the first implementation using dilated CNNs for crowd counting"* |
| 64 | TransCrowd benchmark (arXiv 2104.09116v3) | `https://arxiv.org/pdf/2104.09116v3` | PEER-REVIEWED | **Table 8, quoted verbatim: "CSRNET 384×384, 16.2M, VGG16, 21.67 FPS" and "BL 384×384, 21.6M, VGG19, 45.66 FPS" — *on a Titan XP***. Also the observation that output **feature-map resolution dominates runtime** (BL emits a 1/16-resolution density map) |
| 65 | Zhang et al., MCNN, CVPR 2016 | referenced in #63, #64 | PEER-REVIEWED | Multi-column CNN baseline; MAE 110.2 (ShanghaiTech A), 26.4 (Part B) |
| 66 | **Crowd counting on an RK3399Pro NPU** | `https://www.sciencedirect.com/org/science/article/pii/S1546221823004551` | PEER-REVIEWED | **Inference times 9–28 ms** via RKNN at 320×184 – 640×360 on Venice / Beijing-BRT / ShanghaiTech B. Explicitly notes CSRNet *"encountered difficulties in deployment on embedded devices"* |
| 67 | **Chen et al. (2024). "Privacy-aware crowd counting by decentralized learning with parallel Transformer."** *Privacy & Security* | `https://www.sciencedirect.com/science/article/abs/pii/S2542660524001082` | PEER-REVIEWED | DLPTNet, federated-learning-based privacy-aware crowd counting; cited 19×. **Addresses training-data privacy, not runtime identity** — orthogonal to this project's constraints |
| 68 | **NWPU-Crowd dataset** | `https://arxiv.org/abs/2001.03360v1` | PEER-REVIEWED | 5,109 images, 2,133,338 annotated heads, density range 0–20,033, *"various illumination scenes"* — establishes that academic crowd benchmarks are an **outdoor mass-crowd regime**, not a school corridor |
| 69 | Bloom-filter + homomorphic-encryption crowd monitoring | `https://dl.acm.org/doi/10.1145/3434770.3459735` | PEER-REVIEWED | *"architectural architecture for crowd-monitoring which allows statistical counting for pedestrian dynamics while considering privacy-preservation"* | Cryptographic DP approach; heavy for this project |
| 70 | Wi-Fi privacy-preserving crowd estimation | `https://www.computer.org/csdl/proceedings-article/mass/2024/639900a314/21kuoytyGo8` | PEER-REVIEWED | MASS 2024; decentralised duplicate removal via private set intersection for crowd sizing | Noted as an adjacent RF-based family; **not investigated further in this pass** |
| 71 | *Unique in the Crowd: The privacy bounds of human mobility* (de Montjoye et al.) | `https://www.media.mit.edu/publications/unique-in-the-crowd-the-privacy-bounds-of-human-mobility` | PEER-REVIEWED | *"four spatio-temporal points are enough to uniquely identify 95% of the individuals"* — the re-identification risk argument against persistent tracking | Mobility, not video. Cited as the general principle |

## 8. CCTV integration — real-world practice (INDEPENDENT)

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 72 | ONVIF camera/RTSP troubleshooting checklist | `https://omninvr.com/guides/rtsp-camera-not-connecting.html` | INDEPENDENT (vendor blog) | *"Paths can differ by model, channel, and main/sub-stream choice, and they are often case-sensitive."* Layered diagnosis order. *"Try the camera's lower-resolution sub stream as a diagnostic."* | Commercial product blog; technique is sound, statistics absent |
| 73 | RTSP recording troubleshooting | `https://omninvr.com/guides/troubleshoot-rtsp-recording.html` | INDEPENDENT (vendor blog) | *"Live view works but NVR recording fails → the camera connection is only partially validated."* Timestamp/mux/codec layer decomposition | Same |
| 74 | Home Assistant core issue #172349 (RTSP transport ignored by FFmpeg) | `https://github.com/home-assistant/core/issues/172349` | INDEPENDENT (bug tracker) | Concrete 2026-05-27 failure: FFmpeg snapshot path did not respect the configured TCP transport, causing UDP failure on a Kubernetes-hosted HA. Closed **stale / not_planned** 2026-09-03 | Single user's environment |
| 75 | IP Cam Talk — camera advertising ONVIF/RTSP that it does not deliver | `https://ipcamtalk.com/threads/troubleshooting-ip-camera-compatibility-with-onvif-and-rtsp-protocols.69773` | INDEPENDENT (forum) | AliExpress-sourced camera; ONVIF/RTSP advertised, not enableable via vendor app, firmware update did not unlock. Suggested testing with ONVIF Device Manager + VLC | Anecdotal |
| 76 | TP-Link VIGI NVR forum — cannot specify RTSP port | `https://community.tp-link.com/en/business/forum/topic/715348` | INDEPENDENT (forum) | ONVIF port and RTSP port configured separately on NVRs; manual add offers only the "Management port" | Anecdotal |
| 77 | TP-Link Omada third-party integration documentation | `https://support.omadanetworks.com/baltic/document/44983` | OFFICIAL-MFR | *"When the ONVIF Profile is not supported on the third-party device, Omada Cameras can also enable live streaming on third-party platforms via the RTSP protocol. However, the Omada Camera settings cannot be modified at this time."* URLs `rtsp://IP/stream1`, `rtsp://IP/stream2`, port 554. Security warning: *"the RTSP/ONVIF protocols are not highly secure, so it is not advisable to expose the camera to a public IP via port forwarding for long periods"* | Vendor-specific, but illustrative |
| 78 | OpenCV RTSP read failures (issue #17734) | `https://github.com/opencv/opencv/issues/17734` | INDEPENDENT (bug tracker) | Stream plays in VLC but OpenCV/FFmpeg cannot read it — decoder availability differs per platform | 2020; illustrates the general class |

## 9. Safety-system precedent (STD / INDEPENDENT)

| # | Title | URL | Type | Claims supported | Limitations |
|---|---|---|---|---|---|
| 79 | **ISO 13849-1:2023** official preview (table of contents) | `https://webstore.ansi.org/preview-pages/ISO/preview_ISO+13849-1-2023.pdf` · `https://www.iso.org/obp/ui#!iso:std:iso:13849:-1:ed-4:v1:en` | STD | **§6.2.3 "Unknown PFH values"** exists as a distinct clause; §5 specification of safety functions; §6.1.2 PL↔SIL correlation; §6.1.5 diagnostic coverage; Annex E DC estimates | ⛔ **Clause body paywalled — TOC entry only.** The *existence* of an unknown-value clause is VERIFIED; **what it requires is INFERRED** |
| 80 | ISO/TC 299 robotics catalogue | `https://www.iso.org/committee/5915511/x/catalogue/p/1/u/1/w/1/d/1` | STD | ISO 10218-1:2025, ISO 10218-2:2025, ISO 13482:2014 (service robots), ISO/FDIS 13482 under development, ISO 19649:2017, ISO 3691-4 context | **None verified as applying to this deployment.** Cited only to show the standard landscape exists |
| 81 | IEC 61508 / 62061 / 13849 / 62998 correspondence (Pilz Robotics Summit deck) | `https://www.therobotreport.com/wp-content/uploads/2019/06/lidar-navigation.pdf` | INDEPENDENT | PL a–e ↔ SIL CL 1–3 ↔ SIL 1–4 correspondence table; **IEC 62998 = "Safety-related sensors used for the protection of persons"** — directly applicable framing; SRS/SRSS performance classes A–F; *safe detection of persons with 2D lidar* as the worked example | Industry conference material, 2019. PL/SIL numbers for this project **not** established |
| 82 | NIST — health degradation in robot workcells | `https://www.nist.gov/publications/assessment-novel-position-verification-sensor-identify-and-isolate-robot-workcell-health-degradation-0` | OFFICIAL-GOV | *ASSESSMENT OF A NOVEL POSITION VERIFICATION SENSOR TO IDENTIFY AND ISOLATE ROBOT WORKCELL HEALTH DEGRADATION*, Weiss & Kaplan, ASME J. Mfg. Sci. Eng., 2021 — precedent for using a verification sensor to detect degradation and isolate a cell | Not CCTV/occupancy specific |

## 10. School CCTV policy comparators (INDEPENDENT — **non-Indian, drafting precedent only**)

| # | Title | URL | Type | Claims used |
|---|---|---|---|---|
| 83 | **Madill Board of Education, "School Security Surveillance Cameras"** (adopted 8 Apr 2024) | `https://files.smartsites.parentsquare.com/10122/ckaj.pdf` | INDEPENDENT (US school district policy) | **Drafting precedent for §E.3: "Surveillance cameras also shall not utilize automatic identification, facial recognition, or automatic tracking technologies. Only authorized school personnel… shall be involved with and/or have access to surveillance cameras."** Also: cameras not in restrooms/showers/lockers/private offices; 1-month retention; sole property of the district | ⛔ **Not Indian law.** Cited only as a model clause, never as authority |
| 84 | UK ICO *In the picture* / *Video Surveillance* guidance, as cited in school CCTV policies | referenced in #85/#86 | OFFICIAL-GOV (UK, via secondary) | The guidance school CCTV policies in India actually cite in adapted form — the Indian practice borrows UK/UK-adjacent norms | **Primary ICO guidance NOT retrieved.** Existence established only via citation in school policies |
| 85 | Crofton / Danes / William de Ferrers / Whitehorse school CCTV policies (2024–2026) | `https://4905753ff3cea231a868-376d75cd2890937de6f542499f88a819.ssl.cf3.rackcdn.com/croxleydanes/uploads/document/Surveillance-and-CCTV-Policy-DET-Nov-2025.pdf` · `https://d1zw7qp7axllbi.cloudfront.net/files/CCTV-Policy_V4.0_2026-03.pdf` | INDEPENDENT (UK school policies) | Typical structure: DPIA before installation, 30-day retention, signage identifying controller + contact, DPO involvement, annual review of necessity/proportionality. **None is an Indian instrument** | Corroborates that the Indian practice gap identified in §C.4 is real |

## 11. Corroborating / contextual sources

| # | Title | URL | Type | Use |
|---|---|---|---|---|
| 86 | PIB backgrounder, DPDP Rules notified 17 Nov 2025 | `http://static.pib.gov.in/WriteReadData/specificdocs/documents/2025/nov/doc20251117695301.pdf` | OFFICIAL-GOV | Confirms the MeitY-hosted Rules PDF URL in #38 and the 18-month timeline |
| 87 | AffairsCloud — DPDP Rules phased rollout summary | `https://affairscloud.com/meity-notifies-digital-personal-data-protection-rules-2025` | INDEPENDENT | Corroborates Rule commencement grouping (immediate / +1yr / +18mo) |
| 88 | ONVIF Newsletter Oct 2025 | `https://www.onvif.org/?page_id=8652` | OFFICIAL-MFR | Corroborates the Profile S announcement date |
| 89 | ONVIF staging site homepage | `https://staging.onvif.org/` | OFFICIAL-MFR | Independent capture of the 9 Oct 2025 press-release headline and 2026 committee/ISC schedule |
| 90 | OpenCV 4.x HOGDescriptor reference | `https://docs.opencv.org/4.8.0/d5/d33/structcv_1_1HOGDescriptor.html` | OFFICIAL-CODE | Corroborates API surface pre-5.0 |

---

## 5. Contradictions and tensions recorded (not resolved)

**C1 — ONVIF's RTSP basis is formally obsolete.** RFC 2326 is obsolete per RFC 7826 (§3 #22, #23).
ONVIF's Streaming Specification still builds on RFC 2326 (§1 #5). **Practical resolution:** implement
RTSP 1.0 client semantics, because that is what deployed cameras speak. Do not implement RTSP 2.0
negotiation. Not a blocker; a documentation trap.

**C2 — "Occupancy counting" exists in ONVIF but not in ONVIF's analytics profile.** The Analytics
Service Specification v26.06 Appendix A.6 defines `CountAggregation/OccupancyCounter` (§1 #3), but
Profile M v1.1 contains **zero occurrences** of the word "occupancy" and mandates only the
line-crossing counter (§1 #2). **Consequence:** availability must be checked per-device via
`GetSupportedRules`; it cannot be assumed from profile conformance. This is the most likely source of
a wrong integration claim in this project.

**C3 — ONVIF profile versioning runs on a different clock from ONVIF service versioning.** Core and
Analytics specs are at **26.06** (June 2026) while **Profile M is still v1.1 (March 2024)** (§1 #2,
#6). "Latest ONVIF version" is therefore an ambiguous phrase. Profile M correctly targets ONVIF NI
≥ 21.06, which is satisfied by 26.06.

**C4 — The Go2 camera frame rate is already flagged as contradicted in the prior phase.** The Phase 0
research (`research/sdk_and_software_research.md`, `source_register.md` §6 C2) records 15 fps (About Go2)
against 1080p 30 fps / 720p 60 fps (developer FAQ). §D.2 of the report uses **720p at ≥15 fps**,
which is true under *either* reading — deliberately chosen to sidestep the contradiction.

**C5 — The DPDP Rules were notified on 13 November and announced on 14 November 2025**, and the Act
commencement notification is a *separate* instrument (G.S.R. 843(E), Gazette No. 757) from the Rules
notification (G.S.R. 846(E), Gazette No. 760). Secondary trackers have conflated them. #37 explicitly
corrects this. The 13 May 2027 date remains **derived**, not notified.

**C6 — Camera fps contradiction (this study).** Sources #30/#31 were reachable only via search
index; the tutorial pages (#33, #34) were 404 or Cloudflare-blocked. Any OpenCV performance figure
sourced only from those pages is **UNVERIFIED** and has been excluded rather than repeated.

---

## Coverage against the brief's questions

| # | Question | Answered | Location |
|---|---|---|---|
| 1 | What is ONVIF; spec version; conformance structure | Yes | §A.1 |
| 2 | Analytics/metadata profiles; event framework; doc name + version | Yes | §A.2 |
| 3 | Anonymous metadata events without video; actual event names | Yes | §A.3, §0 |
| 4 | RTSP definition, standard, practical Python/OpenCV mechanism | Yes | §A.4 |
| 5 | Profile T / S / G; does it matter for a school camera | Yes | §A.5 |
| 6 | Digest / Basic / HTTP Digest; what school NVRs use | Partly — #6 answered, "typical NVR config" **UNVERIFIED** | §A.6 |
| 7 | Failure modes; realistic success rate | Failure modes yes; success rate **NOT FOUND** | §A.7 |
| 8 | OpenCV functions; maintenance; limits; licence | Yes | §B.1, §B.2 |
| 9 | Documented accuracy in cluttered indoor corridors | **NOT FOUND** — stated honestly | §B.3 |
| 10 | Modern alternatives; privacy comparison | Yes | §B.4 |
| 11 | Single-camera reliability; failure modes | Yes | §B.7 |
| 12 | Privacy-preserving crowd counting papers | Yes — Chan et al. CVPR 2008 + 8 more | §B.5 |
| 13 | Latency and compute; modest CPU | Partly — GPU/NPU verified, CPU **INFERRED** | §B.6 |
| 14 | Streaming with no video written; how to assert it | Yes — 5 techniques | §C.1 |
| 15 | Metadata-only processing; ONVIF facilities | Yes | §C.2 |
| 16 | Indian law: DPDP, IT Act 66E, DPDP Rules 2025; does it apply to a school | Yes — in depth | §C.3 |
| 17 | Supreme Court guidance on CCTV in schools | **NOT FOUND** — stated honestly | §C.4 |
| 18 | Comparative options (a)–(e) | Yes | §D.1, §D.4 |
| 19 | mmWave products, India cost, count vs presence | Yes | §D.3 |
| 20 | Robot's own 720p/15 fps camera | Yes | §D.2 |
| 21 | Failed-close; sensor confidence / unknown state precedent | Yes | §E.1, §E.2 |
| 22 | FP/FN measurement and reporting | Yes | §E.4 |