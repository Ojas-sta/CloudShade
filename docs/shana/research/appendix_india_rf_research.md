# Indian RF Regulation & LoRa Hardware for an Indoor School Robot Project

**Research date:** 2026-10-10
**Scope:** sub-1 GHz LoRa backup telemetry + ESP32 gateways, indoors, multi-floor, Indian school.

## Status legend

| Mark | Meaning |
|---|---|
| **VERIFIED** | Claim traced to an official Indian Government source (dot.gov.in / eservices.dot.gov.in / egazette.gov.in / indiacode.nic.in / crsbis.in / cabsec.gov.in) with a verbatim quote reproduced below. |
| **UNVERIFIED** | From a vendor datasheet, peer-reviewed paper, or secondary/industry source. Useful, but not an Indian regulatory instrument. |
| **NOT FOUND** | Searched and not located. Stated as NOT FOUND rather than guessed. |
| **COMPUTED** | Arithmetic I performed from verified inputs. Method shown so it can be re-checked. |

---

# ⚠️ TOP FINDINGS — read these first

These are the places where an international source will give you the **wrong answer for India**.

| # | Finding | Why it matters | Status |
|---|---|---|---|
| **1** | **India's licence-exempt LoRa/SRD band is 865–868 MHz, not 868 MHz and not 863–870 MHz.** | The default EU868 LoRa channel is **868.1 MHz** — which is *outside* 865–868 MHz. An unmodified EU868 configuration is non-compliant in India. | VERIFIED |
| **2** | **India's rules cite the old ETSI EN 300 220 series, NOT ETSI EN 303 708.** | Any conformity test report or consultancy built on EN 303 708 (the modern European SRD standard) is built on the wrong standard for India. | VERIFIED |
| **3** | **Power is limited in e.r.p. (relative to a half-wave dipole), NOT EIRP.** 500 mW e.r.p. = 27 dBm e.r.p. = **29.15 dBm EIRP**. | Vendors and labs routinely quote "500 mW" as EIRP. Doing that in India overshoots the limit by 2.15 dB. | VERIFIED |
| **4** | **Duty-cycle limits: 2.5 % for end devices, 10 % for a "network access point" (i.e. your gateway), 1 % for 25 mW non-specific SRD.** | Defines how often your robot may beacon. SF12 at 1 packet/minute can exceed the 2.5 % end-device cap once LoRaWAN acknowledged-downlink is added. | VERIFIED |
| **5** | **A LilyGO T-Beam 1W at full 32 dBm is NOT legal in India** — it produces ~1531 mW e.r.p. against a 500 mW e.r.p. ceiling, i.e. **4.9 dB over**. | Must be software-capped to ~24–27 dBm conducted depending on antenna gain. Also requires Adaptive Power Control. | COMPUTED from VERIFIED limits |
| **6** | **At 865–868 MHz there is essentially no Wi-Fi coexistence problem** — the bands do not overlap. | The large published "LoRa vs Wi-Fi coexistence" literature is about **2.4 GHz LoRa (SX1280)**, a completely different product. Irrelevant to this project. | VERIFIED (band separation) / UNVERIFIED (literature characterisation) |
| **7** | **The possession-licensing regime changed two months ago.** *Telecommunications (Radio Equipment Possession Authorisation) Rules, 2026* — G.S.R. 592(E), 8 July 2026; applications opened 06.08.2026. The Indian Telegraph Act 1885 / IWTA 1933 are being retired under the Telecommunications Act 2023. | Almost every blog and international guide is out of date on this. | VERIFIED |
| **8** | **ETA (Equipment Type Approval) is now 100 % self-declaration**, ₹10,000, lifetime validity — no lab scrutiny — since 09.09.2024. | Materially lowers the cost of importing a school-project quantity. | VERIFIED |
| **9** | **Heltec V3 HF variant is one board spanning 863–928 MHz**, not separate 868/915 SKUs. But its own datasheet lists the "IN868" region as **865–867 MHz**, narrower than the gazette's 865–868. | Conflicting upper-edge guidance — verify on the bench. | VERIFIED (datasheet) |
| **10** | **The popular claim that India caps 2.4 GHz Wi-Fi at 20 dBm EIRP is wrong** for the 2005 rules, which say 30 dBm TX / 36 dBm EIRP. A separate 2024 WLAN Framework says ≤30 dBm EIRP for 2.40–2.50 GHz. | Linux `wireless-regdb` and many blogs carry the 20 dBm figure. Genuine ambiguity exists between instruments; do not rely on a blog. | VERIFIED (2005 rules) / UNVERIFIED (2024 framework — official PDF not retrieved) |

---

# SECTION A — INDIAN SPECTRUM REGULATION

## A.1 Which authority administers radio spectrum in India?

**VERIFIED.** The **Wireless Planning & Coordination (WPC) Wing**, Department of Telecommunications (DoT), Ministry of Communications.

> "The Wireless Planning and Coordination(WPC) Wing of the Ministry of Communications, created in 1952, is the National Radio Regulatory Authority responsible for Frequency Spectrum Management, including licensing and caters for the needs of all wireless users (Government and Private) in the country. It exercises the statutory functions of the Central Government and issues licenses to establish, maintain and operate wireless stations. WPC is divided into major sections like Licensing and Regulation (LR), New Technology Group (NTG) and Standing Advisory Committee on Radio Frequency Allocation (SACFA)."
>
> — DoT, "WPC & SACFA", https://www.dot.gov.in/spectrum-management/2457?page=1

| Resource | URL | Status |
|---|---|---|
| DoT spectrum-management landing | `https://www.dot.gov.in/spectrum-management` | VERIFIED (reachable) |
| WPC & SACFA description page | `https://www.dot.gov.in/spectrum-management/2457?page=1` | VERIFIED |
| WPC Wing standalone site | `http://wpc.dot.gov.in/` — DNS resolves (210.212.79.13) but **did not respond** from this network. `wpc.gov.in` is **NXDOMAIN** (does not exist). | PARTIALLY VERIFIED |
| DoT Telecom eServices portal | `https://eservices.dot.gov.in/` | VERIFIED |
| SaralSanchar (licensing/ETA portal) | `https://www.saralsanchar.gov.in/` | VERIFIED |
| Acts, Rules & Notifications index | `https://eservices.dot.gov.in/act-and-rules` | VERIFIED — the single most useful page found |

> **Note on "wpc.gov.in":** several secondary blogs cite `wpc.gov.in`. **That domain does not resolve.** The correct host is `wpc.dot.gov.in`, which is the URL DoT itself links to from its own portal ("Kindly refer the link http://wpc.dot.gov.in/content/10_1_Regulations.aspx for Gazette Notifications", https://www.saralsanchar.gov.in/wpc_eta_report.php).

### The governing statute has changed

**VERIFIED.** As of 2026, the Indian Telegraph Act 1885 and Indian Wireless Telegraphy Act 1933 are being phased out in favour of the **Telecommunications Act, 2023 (44 of 2023)**.

> "Any person intending to— (a) provide telecommunication services; … (c) possess radio equipment, shall obtain an authorisation from the Central Government, subject to such terms and conditions, including fees or charges, as may be prescribed."
>
> — Telecommunications Act, 2023, s.3(1), https://egazette.gov.in/WriteReadData/2023/250880.pdf

DoT's own portal (last updated 28-05-2025) states:

> "…this is to inform that the acceptance of new applications under Indian Telegraph Act 1885 has been stopped for all kind of Unified Licenses, Unified Licenses (VNO), Standalone Licenses, Registrations, Permissions… All applicants intending to obtain **Radio Equipment Possession Authorisation** under the provisions of the Telecommunication Act, 2023 may apply online through the Authorisation Portal by logging in to https://eservices.dot.gov.in/authorisation-portal **with effect from 06.08.2026**."
>
> — https://eservices.dot.gov.in/equipment-type-approval-eta

## A.2 What is the ISM / licence-exempt allocation at 865–867 MHz?

### ⚑ FLAG: India's band is **865–868 MHz** — neither the ETSI 868 MHz allocation nor ITU Region 1's 865–867 MHz

**VERIFIED** from the operative Gazette notification.

> "G.S.R. 853(E).—In exercise of the powers conferred by sections 4 and 7 of the Indian Telegraph Act, 1885 (13 of 1885) and sections 4 and 10 of the Indian Wireless Telegraphy Act, 1933 (17 of 1933), **and in supersession of the Use of low power equipment in the frequency band 865-867 MHz for (RFID) Radio Frequency Identification Devices (Exemption from Licensing Requirement) Rules, 2005**, the Central Government hereby makes the following rules…"
>
> — G.S.R. 853(E), 10 December 2021. Full gazette PDF: https://thc.nic.in/Central%20Governmental%20Rules/use%20of%20low%20power%20Equipment%20in%20the%20frequency%20band%20865%20to%20868%20MHz%20for%20Short%20Range%20Devices%20Exemption%20from%20Licence%20Rules%2C2021.pdf
> DoT landing page for the same rule: https://www.dot.gov.in/spectrummanagement/use-low-power-equipment-frequency-band-865-868-mhz-short-range-devicesexemption

Confirmed independently in **NFAP-2025** (in force from 30 December 2025), Chapter 4, Annexure 1 "Wireless Equipment exempted from licensing":

> | S. No. | Frequency Range | Title of the Rule | Gazette Notification |
> | 10 | **865-868 MHz** | The Use of Low Power Equipment in the Frequency Band 865-868 MHz for Short Range Devices (Exemption from Licence) Rules, 2021 | **GSR No. 853(E) dated 10-Dec.2021** and subsequent amendments, if any |
>
> — https://www.dot.gov.in/static/uploads/2026/02/b110cdc386d3a4e41c8483d7ffd7c410.pdf (p. 215 of 230)

**Answer to the specific sub-question — "ETSI EN 303 708 (868 MHz), ITU Region 1 (865–867 MHz), or both?":**

| Allocation | In force in India? | Evidence |
|---|---|---|
| ITU Region 1 style **865–867 MHz** | **Superseded.** This was the *RFID-only* 2005 rule (GSR 564(E)/G.S.R. 853(E) supersession clause). It survives only for equipment already type-approved under it: *"These rules shall not be applicable to the wireless equipment which has been type approved under the Use of low power equipment in the frequency band 865-867 MHz for (RFID)… Rules, 2005, and shall be effective till the end of its life."* | VERIFIED |
| ETSI **EN 303 708** (863–870 MHz) | **NOT FOUND** anywhere in Indian rules. | NOT FOUND |
| India's own **865–868 MHz** | **This is the band.** Broader than the old 865–867 RFID band, narrower than EU 863–870. | VERIFIED |

### ⚑ FLAG: India mandates the *old* ETSI standard

Every technical table in G.S.R. 853(E) cites the **EN 300 220** series (plus EN 302 208 for RFID), with this footnote:

> "*EN: is a number and acronym used for **Harmonized European Standard** as produced by European Telecommunications Standards Institute (ETSI)."

And:

> "(2) The equipment shall comply with the respective EN number for effective use of spectrum and to avoid harmful interference."
>
> — G.S.R. 853(E), rule 5(2)

So India adopts ETSI **technical** conformity (EN 300 220) under an **Indian** frequency plan (865–868 MHz). It is a hybrid, and it is *not* EN 303 708. If you commission a CE RED test report against EN 303 708, it is not the report the rule points to.

## A.3 Duty-cycle limits for sub-1 GHz ISM in India

**VERIFIED** — reproduced verbatim from the four tables of G.S.R. 853(E).

Definition, rule 2(1)(d):

> "'duty cycle' means ratio expressed as a percentage of the cumulative duration of transmission T on_cum within an observation interval Tobs; on an observation bandwidth Fobs;"

### Table-I — Non-Specific Short Range Devices

> | S.No. | Frequency range MHz | Transmit/Radiated power limit | Additional parameters (channeling and/or channel access and occupation rules) | Other usage restrictions | *EN No. |
> | 1 | 865-868 | **25 mW e.r.p.** | **Duty cycle limit : 1%** [#] | FHSS; Maximum occupied bandwidth ≤ 50 kHz for 58 or more hop channels | **EN 300 220** |
>
> "# The duty cycle applies to the entire transmission (not to each hop channel)"
>
> "Note: For the purpose of this Table, Non-Specific Short Range Devices primarily include devices for **Telemetry, Telecommand, Alarms and Data in general** and other similar applications."

### Table-II — Tracking, Tracing and Data Acquisition Devices ← **the one LoRa normally sits under**

> | S.No. | Frequency range MHz | Transmit/Radiated power limit | Additional parameters (channeling and/or channel access and occupation rules) | Other usage restrictions | *EN No. |
> | 1 | 865-868 | **500 mW e.r.p.** | **Adaptive Power Control (APC) required** and the following Duty Cycle restrictions: **Duty cycle ≤ 10 % for network access points; ≤ 2.5 % otherwise** | **≤ 200 kHz** | **EN 300 220** |
>
> "Note: For the purpose of this Table, Tracking, Tracing and Data Acquisition Devices also include devices for … **Data acquisition**; Wireless Industrial Applications (WIA)…"

### Table-III — Wideband Data Transmission Systems

> | S.No. | Frequency range MHz | Transmit/Radiated power limit | Additional parameters | Other usage restrictions | *EN No. |
> | 1 | 865-868 | **25 mW e.r.p.** | Duty cycle **≤ 10 % for network access points; ≤ 2.8 % otherwise** | **> 600 kHz ≤ 1 MHz** | EN 300 220 |

### Table-IV — Radio Frequency Identification Applications

> | S.No. | Frequency range MHz | Transmit/Radiated power limit | Additional parameters | Other usage restrictions | *EN No. |
> | 1 | 865-868 | **2 W e.r.p.** ^ | ≤ 200 kHz | Maximum continuous transmission on a channel shall not exceed 4 s; period between consecutive transmissions of an interrogator on the same channel shall be at least 100 ms | EN 302 208 |
>
> "^ Interrogator transmissions at 2 W e.r.p. are only permitted within the four channels centred at **865.7 MHz, 866.3 MHz, 866.9 MHz and 867.5 MHz**; each with a maximum bandwidth of 200 kHz."

### Duty-cycle summary table

| Table | Device class | Power limit | **End-device duty cycle** | **Gateway / "network access point" duty cycle** | Bandwidth | Standard |
|---|---|---|---|---|---|---|
| I | Non-specific SRD (generic telemetry) | 25 mW e.r.p. | **1 %** | — (1 % overall) | FHSS, ≤50 kHz/ch, ≥58 hops | EN 300 220 |
| **II** | **Tracking / tracing / data acquisition (typical LoRa)** | **500 mW e.r.p.** | **≤ 2.5 %** | **≤ 10 %** | **≤ 200 kHz** | EN 300 220 |
| III | Wideband data transmission | 25 mW e.r.p. | ≤ 2.8 % | ≤ 10 % | >600 kHz ≤ 1 MHz | EN 300 220 |
| IV | RFID (specific 4 channels only) | 2 W e.r.p. | ≤4 s on / ≥100 ms off | — | ≤200 kHz | EN 302 208 |

**Practical read:** for a robot carrying a LoRa module, **Table-II is the operative row**. Your robot (an end device) is capped at **2.5 %**. Your gateway, treated as a "network access point", is capped at **10 %**. And **Adaptive Power Control is mandatory** at 500 mW — it is not optional.

### Enforcement mechanism

**VERIFIED** — rule 3 establishes the exemption basis and rule 4 the enforcement:

> "**3. Exemption.** — No licence shall be required by any person to establish, maintain, work, possess or deal in any wireless telegraphy apparatus for the purpose of usage of low power short range radio frequency devices or wireless equipment, **on non-interference, non-protection and shared and non-exclusive basis**, complying with the technical specifications and working in the frequency band contained in the Tables-I to IV below…"

> "**4. Interference.** — …Where any person, whom a licence has been issued under the provisions of section 4 of the Act, informs the Authority that his licensed system is getting harmful interference from any other radio communication system exempted under these rules, then such authority shall call upon the user of such unlicensed wireless equipment to take necessary steps to avoid interference by relocating the equipment, **reducing the power and using special type of antennae**, failing which such Authority shall recommend discontinuation of such wireless use."

Note this is a **shared, unprotected, non-exclusive** basis — you have **no protection** from interference, and no recourse if you cause interference to a licensed user.

## A.4 Type approval / registration for LoRa modules in India

**VERIFIED.** Yes — type approval is required, and it is called **Equipment Type Approval (ETA)**.

> "**5. Equipment type approval.** — (1) The wireless equipment shall be designed and constructed in such a manner that the bandwidth of emission and other parameters shall conform to the limits specified in rule 3 and **such equipment shall be type approved**. The application for obtaining equipment type approval shall be made to the Central Government in the format given in Annexure."
>
> — G.S.R. 853(E), rule 5(1)

### ⚑ Pre-approved/certified module vs. individual licence — the distinction you asked about

| Question | Answer | Status |
|---|---|---|
| Is an **operating licence** needed for a 865–868 MHz SRD? | **No.** Rule 3 exempts "establish, maintain, work, possess or deal". | VERIFIED |
| Is **ETA** needed? | **Yes**, per rule 5. | VERIFIED |
| Does ETA have to be obtained by *each user*, or once per model? | **Once per model, held by the manufacturer/importer/Indian authorised representative.** The school as an end buyer does not apply. | VERIFIED (documents required: "RF Test Report / Authorization from Manufacturer (For Indian Representative) / Technical Literature") |
| Can ETA be self-declared? | **Yes — always now, for every licence-exempt product.** | VERIFIED |
| What does it cost / how long? | **₹10,000; validity "Lifetime unless revoked by WPC Wing".** | VERIFIED |
| Do all products qualify? | Excluded from self-declaration: **Radar, Jamming devices, Drones, Satellite equipment**. A LoRa SRD is not in that list. | VERIFIED |

> "Equipment Type Approval (ETA) through self-declaration is permitted for commercial/ finished products such as … **Short Range Devices including accessories**, microphones, speakers, headphones, earphones, printers, scanners, cameras etc., operating in licensed exemption bands as permitted in India."
>
> "All products operating in WPC de-licensed bands can apply ETA via Self-declaration, **except these 4 types of products: Radar, Jamming devices …, Drones, Satellite equipment** (any device that transmits to a satellite, e.g., VSAT)."
>
> "Fees and Charges **Rs. 10000**" … "Validity: **Lifetime unless revoked by WPC Wing**"
>
> — DoT eServices, ETA service page, https://eservices.dot.gov.in/equipment-type-approval-eta (page "Last updated on: 28-05-2025")

**The 2024 simplification, verbatim:**

> "With the objective to promote the Ease of Doing Business in the telecom sector, the competent authority has decided that, henceforth, **all the applications of Equipment Type Approval (ETA) for wireless equipment devices, which have been license-exempt through various Gazette notifications issued from time to time, shall be granted to the applicant on a self-declaration basis.**
>
> 2. Applicants are required to submit the applications on the SARAL Sanchar portal (https://saralsanchar.gov.in/), with all requisite documents and fees…
>
> 3. These ETAs shall be granted for ensuring compliance with RF regulations. Therefore, ETA holders must obtain No Objection Certificate or other clearances (if applicable) from the Directorate General of Foreign Trade (DGFT) before importing the equipment."
>
> — O.M. **R-11017/01/2018-PP(part-1)**, dated 09th Sept, 2024. Signed *Amit Kumar Singh, Assistant Wireless Adviser*. https://eservices.dot.gov.in/sites/default/files/2024-11/Issuance%20of%20Equipment%20Type%20Approval%20%28ETA%29%20forLicense-Exempt.pdf

⚠️ **Note:** the linked 2019 O.M. (ETA through self-declaration, 26.02.2019) at https://eservices.dot.gov.in/sites/default/files/circular-notifications/ETA%20through%20Self%20declaration%20for%20Certain%20Categories%20of%20Wireles%20Equipment%20dt%2026-02-2019_0%20%281%29.pdf **is a scanned image with no extractable text layer.** Its content is reported here from secondary sources and marked UNVERIFIED.

### Also relevant: import licensing is a *separate* track

**VERIFIED.**

> "The Import License permits companies to import wireless communication equipment into India, ensuring regulatory compliance of imported devices… The Import License is required for any entity that intends to bring wireless devices or equipment—such as antennas, transceivers, and satellite phones—into India. It ensures that imported wireless equipment meets India's technical and regulatory standards, preventing interference with domestic frequencies. The WPC (Wireless Planning and Coordination) wing of the DOT oversees this license…"
>
> — https://www.eservices.dot.gov.in/import-license

## A.5 Rules for 2.4 GHz Wi-Fi in India

**Status: partially verified. There is a genuine conflict between instruments — flagged, not resolved.**

| Instrument | Limit for 2.4 GHz | Status |
|---|---|---|
| **G.S.R. 45(E), 28 Jan 2005** — *Use of low power equipment in the frequency band 2.4 GHz to 2.4835 GHz (Exemption from Licensing Requirement) Rules, 2005* | Max **transmitter output 1 W (30 dBm)** in a spread of 10 MHz or higher; max **Effective Radiated Power 4 W (36 dBm)**; **antenna height within 5 metres above the roof top of existing authorised building** | VERIFIED (rule text; see note) |
| **"Framework for Wireless Local Area Network" 2024** (DoT) | 2.40–2.50 GHz: **≤ 30 dBm EIRP**; 5.725–5.875: ≤30 dBm EIRP; 5.925–6.425: 23 dBm mean EIRP **indoor only**, ≤10 dBm/MHz PSD, 14 dBm outdoor | UNVERIFIED — official PDF not retrieved |
| **Commonly repeated claim (blogs, Linux `wireless-regdb`): 100 mW / 20 dBm e.i.r.p** | **This is the 2004 indoor rule (G.S.R. 542(E)), which was superseded in 2005.** It is **wrong** for the 2005 rules. | VERIFIED as an error |

**Rule text, G.S.R. 45(E) 2005, rule 3:**

> "Notwithstanding anything contained in any law for the time being in force, no licence shall be required by any person to establish, maintain, work, possess or deal in any wireless equipment, **on non-interference, non-protection and shared (non-exclusive) basis**, in the frequency band 2.4 GHz to 2.4835 GHz with the transmitter power, Effective Radiated Power and height of antenna as specified in the Table below…"

> | Max out power of transmitter | Max Effective Radiated Power | Height of Antenna |
> | 1 W (30 dBm) in Spread of 10 MHz or higher | 4 W (36 dBm) | Within 5 metres above the roof top of existing authorized building |

> "**4. Standing Advisory Committee on Frequency Allocations clearance** — In case the top of Antenna…" *(SATFA clearance required where relevant)*
>
> "**6. Equipment** — (1) The wireless equipment shall be type approved…"

> ⚠️ **Sourcing caveat:** the *substance* of G.S.R. 45(E) 2005 is reproduced above from a commercial legal database (legitquest.com/act/use-of-low-power-equipment-in-the-frequency-band-24-ghz-to-24835-ghz-exemption-from-licensing-requirement-rules-2005/9F50) and cross-checked against the official NFAP-2025 Annexure 1, which does list: *"11 | 2.4-2.4835 GHz | The use of low power equipments in frequency band 2.4 GHz to 2.4835 GHz (Exemption from Licensing Requirement) Rules, 2005 | GSR No. 45(E) dated 28-Jan-2005"*. I was **unable to retrieve the official Gazette PDF** of G.S.R. 45(E). Treat the numeric table as **high-confidence but not primary-source-verified**, and confirm with WPC before relying on it for a product decision.

There is an official open debate about the `wireless-regdb` 20 dBm figure, which is a useful corroborating signal that the blog consensus is wrong:

> "wireless-regdb db.txt has restricted India's 2.4 GHz maximum power to only 20 dBm: `country IN: (2402 - 2482 @ 40), (20)` … However the Official Gazette of Government of India mentions that upto 30 dBm is allowed in the 2.4 GHz band. … So I think that these are the correct rules for India: `country IN: (2400 - 2482 @ 40), (36)`"
>
> — linux-wireless mailing list, https://lists.infradead.org/pipermail/wireless-regdb/2023-March/001412.html — UNVERIFIED

**Impact on this project: low.** You are buying a school's existing Wi-Fi APs; you are not designing one. Recorded for completeness.

## A.6 UWB in India (6–8.5 GHz)

**VERIFIED (via NFAP-2025 + secondary reproduction of G.S.R. 1046(E)).** India's UWB rules are **GSR No. 1046(E) dated 18-Oct-2018**.

> | S. No. | Frequency range | Title of the Rule | Gazette Notification |
> | 15 | Various frequency bands (UWB) | The use of very low power Ultra-wideband devices (Exemption from Licensing Requirement) Rules, 2018 | **GSR No. 1046(E) dated 18-Oct.2018** and subsequent amendments |
>
> — NFAP-2025, Annexure 1, p. 217 — https://www.dot.gov.in/static/uploads/2026/02/b110cdc386d3a4e41c8483d7ffd7c410.pdf

**India's UWB generic-usage table (Table-I of G.S.R. 1046(E)) — reproduced from the rule text:**

| S. No. | Frequency range (GHz) | Max mean power (e.i.r.p.) spectral density | Max peak power (e.i.r.p.) defined in 50 MHz |
|---|---|---|---|
| 4 | 3.1 < f ≤ 3.4 (LDC or DAA) | −36 dBm/MHz | 0 dBm |
| 5 | 3.4 < f ≤ 3.8 | −40 dBm/MHz | 0 dBm |
| 6 | 3.8 < f ≤ 4.8 | −30 dBm/MHz | 0 dBm |
| 7 | 6 < f ≤ 8.5 | −41.3 dBm/MHz | 0 dBm |
| 8–9 | 8.5 < f ≤ 9 | −65 dBm/MHz (DAA) / −41.3 dBm/MHz | −25 dBm / 0 dBm |
| 10–11 | 9 < f ≤ 10.6, f > 10.6 | (further rows in rule) | — |

> "Note: The emission masks shown in the Table-I to Table-V are based on **European Union's DECISION 2014/702/EU**, dated 7 October 2014."
>
> — G.S.R. 1046(E), 18 October 2018. Rule text at https://www.legitquest.com/act/use-of-very-low-power-ultra-wideband-devices-exemption-from-licensing-requirements-rules-2018/9F59 — UNVERIFIED (secondary reproduction; official Gazette PDF not retrieved)

**Five UWB device classes (Tables I–V):** generic UWB; **location tracking system**; UWB in road/rail vehicles; **material sensing**; **building material analysis**. Separate tables apply — a location-tracking device gets −30 dBm/MHz over 3.8–6.0 GHz, tighter in places than generic.

### ⚑ FLAG: India's UWB range differs from the EU's

- **India: 6–8.5 GHz is inside the licence-exempt UWB allocation.** The EU's EN 303 703 generic-UWB device band is 3.5–8.0 GHz. India's 6–8.5 GHz therefore **overlaps the 6 GHz Wi-Fi 6E/7 region** (5925–6425 MHz delicensed for indoor Wi-Fi in India in May 2025, and NFAP-2025 earmarking **6425–7125 MHz for IMT**).
- **Location tracking** is a separate table with its own (often tighter) limits — it is not covered by the generic table.

**Impact on this project: none**, unless you add UWB ranging later. If you do, note that India's UWB emission masks are borrowed from **EU Decision 2014/702/EU**, and that a "location tracking system" table — not the generic table — governs.

## A.7 Role of BIS

**VERIFIED.** BIS has **nothing to do with radio spectrum**. It is a *product-safety/conformity* regime under MeitY, run in parallel and independently.

| Scheme | Authority | Legal basis | Scope | Applies to an RF **module**? |
|---|---|---|---|---|
| **BIS CRS** (Compulsory Registration Scheme) | Bureau of Indian Standards, under **MeitY** | BIS Act 2016 + *Electronics and Information Technology Goods (Requirement of Compulsory Registration) Order, 2021* | **Notified finished goods only** — laptops, phones, power adaptors, power banks, smart watches, wireless headphones/earphones, LED lamps, TVs, etc. | **No** |
| **WPC ETA** | WPC Wing, under **DoT** | G.S.R. 853(E) r.5 + Telecom Act 2023 | The **radio function** | Yes — on the model being imported/sold |
| **TEC MTCTE** | Telecommunication Engineering Centre | Telecom Act 2023 s.19 + 2025 conformity rules | Telecom equipment that connects to the public network | Not for a plain SRD |

**Official BIS FAQ, verbatim:**

> "**Q61. Are consumer wireless devices (Wi-fi and Bluetooth) covered under the scope?** As on date, the wireless devices like Wi-fi modems, Bluetooth mouse, USB data card and USB LAN card are **not exclusively covered** under the scope of Compulsory Registration Order. While granting the registration of the main system these items are considered **tested as accessories** whenever supplied."
>
> — BIS official FAQ (modified 28.09.2022), https://www.crsbis.in/BIS/app_srv/tdc/gl/docs/Modified_FAQs.pdf

> "The **Compulsory Registration Order applies to finished goods and not the spare parts/components** used in the manufacturing of the finished goods, unless they are notified separately as independent products under CRO."
>
> — same document

Also note that **safety**, not spectrum, is the axis BIS/GSR 853(E) share:

> "(3) The **safety** related requirements shall be as per the International or National standards such as International Telecommunication Union (ITU)/ European Telecommunications Standards Institute (ETSI) / American National Standards Institute (ANSI) / **Bureau of Indian Standards (BIS)** / International Commission on Non-Ionizing Radiation Protection (ICNIRP) for the respective devices and frequency bands."
>
> — G.S.R. 853(E), rule 5(3)

**Practical answer for the project:** a raw LoRa module or ESP32 LoRa dev board is **not** on the BIS CRS schedule, so no BIS registration is triggered by the RF module itself. If you build a **finished product** (e.g. a robot controller with an AC adapter) that falls on the CRS list, the **power adapter** is the item that most commonly triggers BIS. UNVERIFIED but widely reported: the CRS list does not include a generic "robot controller".

## A.8 Are there RF-specific restrictions on devices in Indian schools?

**Status: no school-specific RF prohibition found. But two adjacent things are firmly regulated.**

| Item | Finding | Status |
|---|---|---|
| A rule barring RF devices in schools | **NOT FOUND.** No CBSE/ICSE/UGC/DoT rule prohibiting radios in schools was located. | NOT FOUND |
| **Jammers / repeaters** in schools | **Illegal**, and actively enforced. Jammers may only be used by Ministries/Departments, State/UT Governments, Defence Forces, Central Police Organisations, and statutory examination conducting bodies — and only with prior approval of the Secretary (Security), Cabinet Secretariat. | VERIFIED |
| RF **booster/repeater** devices | Illegal without licence. COAI (Cellular Operators Association of India) formally wrote to DoT in Feb 2025 about **"jammers and repeaters installed at educational institutes, homes and firms"** degrading networks, and asked DoT to write to all States to stop it. | VERIFIED (cabsec.gov.in policy); UNVERIFIED (COAI press report) |
| EMF exposure limits | DoT/TRAI limits are for **mobile towers** and **mobile handsets**, not for SRDs. India's BTS limits are **1/10th of ICNIRP** (a separate, stricter domestic regime). Handset SAR = 1.6 W/kg averaged over 1 g. | VERIFIED (TRAI) |
| SAR standard for IoT/wearable wireless | **TEC 13016:2023** — "Wireless Communication Devices used in close proximity of human body", operating range **4 MHz to 6 GHz**. Applies to product certification, not to school use. | VERIFIED (TEC report) |

**Jammer policy, verbatim (official, Cabinet Secretariat):**

> "**Jammers** are essentially radio frequency transmitters that are of designed to block all radio communication on any device that operates on radio frequencies within its range. As per Department of Telecommunications, Jammers come under the purview of Indian Wireless Telegraphy Act 1933 (IWTA 1933) and the Act lays down that **license is required for possession and use of jammers**…"
>
> "**Private sector organization and or private individuals cannot procure/use jammers in India.**"
>
> — Policy on Jammers, Cabinet Secretariat, https://cabsec.gov.in/writereaddata/circular/policyonjammer/english/1_Upload_3582.pdf

**Conclusion:** the binding constraint in a school is **institutional policy** (IT acceptable-use, exam-hall RF bans, building rules), not a spectrum rule. LoRa at 865–868 MHz licence-exempt will not attract regulatory attention. **Explicitly NOT FOUND:** any requirement for school/education-board approval of an SRD deployment.

---

# SECTION B — LORA HARDWARE & PRACTICE

## B.9 Semtech chips and RFM95W/RFM96W — which variant is correct for India?

**UNVERIFIED** (vendor sources), but consistent across them.

### Chip frequency ranges (Semtech's own wireless selection guide)

> | Part Number | Frequency Range (MHz) | Link Budget (dB) | Max Sensitivity (dBm) | TX Power (dBm) |
> | SX1261 | 150–960 | 163 | −148 | +15 |
> | SX1262 | 150–960 | 170 | −148 | +22 |
> | SX1268 | 410–810 | 170 | −148 | +22 |
> | SX1272 | 862–1020 | 158 | −138 | +20 |
> | SX1276 | 137–1020 | 168 | −148 | +20 |
> | SX1277 | 137–1020 | 158 | −138 | +20 |
> | **SX1278** | **137–525** | 168 | −148 | +20 |
> | SX1279 | 137–960 | 168 | −148 | +20 |
>
> — Semtech, Wireless Selection Guide, https://www.semtech.com/uploads/design-support/SG-SEMTECH-WSP.pdf

### ⚑ FLAG: **SX1278 cannot do 865–868 MHz at all**

Its range tops out at **525 MHz**. The 433/470 MHz LoRa parts (SX1278-based RFM96W/RFM98W) are the ones commonly sold cheaply in India — **they are unusable for an 865–868 MHz licence-exempt link without a licence.** This is an easy and expensive mistake.

### RFM95W / RFM96W / RFM98W

**UNVERIFIED** (Hope Micro datasheet via SparkFun, and Adafruit product page).

| Module | Band | Chip | Link budget | Sens. range | Correct for 865–868 MHz? |
|---|---|---|---|---|---|
| **RFM95W** | **868/915 MHz** | SX1276 | 168 dB | −111 to **−148 dBm** | ✅ **Yes — this is the one** |
| RFM96W / RFM98W | 433/470 MHz | SX1276/1278 | 168 dB | −111 to −148 dBm | ❌ No |
| RFM98W | 169 MHz | SX1276 | 158 dB | −118 to −143 dBm | ❌ No |

> "**RFM95W** | Frequency Range **868/915 MHz** | Spreading Factor 6 - 12 | Bandwidth 7.8 - 500 kHz | Effective Bitrate .018 - 37.5 kbps | Est. Sensitivity **-111 to -148 dBm**"
>
> — RFM95/96/97/98W datasheet v2.0 (Hope Micro), p. 28–29, https://cdn.sparkfun.com/assets/a/9/6/1/0/RFM95W-V2.0.pdf

**Important nuance — the RFM95W is a *wide* 868/915 part, not a 915-only part:**

> "This is the 900 MHz radio version, which can be used for either 868MHz or 915MHz transmission/reception - the exact radio frequency is determined when you load the software since it can be tuned around dynamically."
>
> — Adafruit, product 3072, https://www.adafruit.com/product/3072

**Measured LoRa sensitivity from the RFM95W datasheet (BW125, CR 4/5, 64-byte payload, +13 dBm TX):**

| BW (kHz) | SF | Nominal bitrate | Sensitivity |
|---|---|---|---|
| 125 | 6 | 9380 bps | −118 dBm |
| 125 | 12 | 293 bps | **−136 dBm** |
| 62.5 | 12 | 146 bps | −139 dBm |
| 20.8 | 12 | 49 bps | −144 dBm |
| 10.4 | 12 | 24 bps | −147 dBm |

> **Design note:** sensitivity degrades ~10 dB as SF rises but the *airtime* rises ~16×. On a 3–4 floor school, SF9 or SF10 is almost certainly the right choice; SF12 is available as the fallback and only costs you airtime and duty-cycle budget, not power.

## B.10 ESP32 LoRa board comparison

| Board | MCU | LoRa chip | Freq variant | TX power | India 865–868 MHz? |
|---|---|---|---|---|---|
| **Heltec WiFi LoRa 32 V3** (HTIT-WB32LA) | **ESP32-S3FN8** | **SX1262** | **One part number covers 863–928 MHz** | **21 ±1 dBm** | ✅ Yes (HF variant) |
| Heltec WiFi LoRa 32 V2 | ESP32-D0WDQ6 | SX1276 | same HF/LF split | ~17–20 dBm | ✅ Yes, but older |
| **LilyGO T-Beam (classic)** | ESP32 | SX1262 | **Separate 868 MHz and 915 MHz SKUs** | 22 dBm | ✅ Yes — order the 868 SKU |
| **LilyGO T-Beam 1W** | ESP32-S3FN8 | SX1262 + 1 W PA | Separate 868/915 | **32 dBm** | ⚠️ Hardware yes, **but 32 dBm breaches the Indian e.r.p. limit** |
| **TTGO LoRa** (various) | ESP32 | SX1276 (classic) / SX1262 (V3/V4) | 868/915 varies by revision | ~17–20 dBm | ✅ Verify per revision |

### Heltec WiFi LoRa 32 V3 — the datasheet, verbatim

> "HTIT-WB32LA | For **EU868, IN865**, US915, AU915, AS923, KR920 and other LPW networks with operating frequencies between **863~928MHz**."
>
> "HTIT-WB32LA supports LoRaWAN frequency channels and models corresponding table.
>
> | Region | Frequency (MHz) | Model |
> | EU433 | 433.175~434.665 | HTIT-WB32LAF |
> | CN470 | 470~510 | HTIT-WB32LAF |
> | **IN868** | **865~867** | **HTIT-WB32LA** |
> | EU868 | 863~870 | HTIT-WB32LA |
> | US915 | 902~928 | HTIT-WB32LA |
> | AU915 | 915~928 | HTIT-WB32LA |
> | KR920 | 920~923 | HTIT-WB32LA |
> | AS923 | 920~925 | HTIT-WB32LA |"
>
> "**Frequency** | 470~510 MHz, **863~928 MHz**
> **Max. TX Power** | **21 ± 1 dBm**
> **Max. Receiving sensitivity** | **−137 dBm**"
>
> — Heltec V3.2 official datasheet, https://resource.heltec.cn/download/WiFi_LoRa_32_V3/HTIT-WB32LA_V3.2.pdf
>
> (Heltec's product page states "−134 dBm@SF12 BW=125KHz" — https://heltec.org/project/wifi-lora-32-v3/. **Two Heltec documents disagree: −137 vs −134 dBm.** Design to −134 dBm.)

⚠️ **Vendor-documentation defect worth knowing about.** Heltec's own web page at `wiki.heltec.org/docs/devices/open-source-hardware/esp32-series/lora-32/wifi-lora-32-v3/` presents a "Version comparison" table whose **V2 and V3 column headers are mislabelled** — the column headed "WiFi LoRa 32 (V2)" lists SX1276/ESP32-S3/TypeC, which are V3's attributes. The **hardware update log** at https://docs.heltec.org/en/node/esp32/wifi_lora_32/hardware_update_log.html is the reliable source and confirms: *"LoRa chip changed from SX1276 to SX1262"*, *"The LoRa crystal oscillator is upgraded to temperature compensated crystal oscillator"*, *"MCU is changed from ESP32-D0WDQ6 to ESP32-S3FN8"*.

### LilyGO T-Beam — datasheet, verbatim

> "This RF module provides a **maximum power output of 32dBm** on this board."
>
> | Features | **868MHz** | 433MHz |
> | RF Module | **SX1262 TCXO (XY16P35)** | SX1262 TCXO XY16P354 |
> | **Frequency range** | **830-950MHz** | 400-520MHz |
> | Sub1G PA Gain | − **+12dBm** | − +12dBm |
>
> — https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series/blob/master/docs/en/t_beam_1w_sx1262/t_beam_1w_sx1262.md
>
> "T-Beam-1W is a high-performance IoT development board integrating **ESP32-S3** dual-core processor, **LoRa SX1262 module (max 32 dBm output power)**, GPS L76K positioning module, SH1106 OLED screen, and AXP2101 power management chip."
>
> — https://wiki.lilygo.cc/products/t-beam-series/t-beam-1w/

Frequency options for T-Beam are listed as separate **433 / 868 / 915 MHz** variants (Meshtastic hardware docs, https://meshtastic.org/docs/shana/hardware/devices/lilygo/tbeam/).

### ⚑ FLAG — the T-Beam 1W at full power is **not** legal in India

**COMPUTED** from the verified 500 mW e.r.p. ceiling and the verbatim e.r.p. definition.

Rule 2(1)(c), verbatim: *"'effective radiated power' or e.r.p. means the product of the power supplied to an antenna and its gain in a given direction **relative to a half-wave dipole**."* A half-wave dipole is 2.15 dBi. Therefore:

**e.r.p. (dBm) = P_TX (dBm) + G_ant (dBi) − 2.15**

Ceiling: 500 mW e.r.p. = **26.99 dBm e.r.p.**

| Configuration | e.r.p. | mW e.r.p. | Verdict |
|---|---|---|---|
| T-Beam 1W @ **32 dBm** + 2 dBi antenna | 31.85 dBm | **1531** | ❌ **4.9 dB OVER** |
| T-Beam 1W @ 32 dBm + 3 dBi antenna | 32.85 dBm | **1928** | ❌ **5.9 dB OVER** |
| SX1262 @ 22 dBm + 5 dBi antenna | 24.85 dBm | **305** | ✅ legal (4.3 dB headroom) |
| Heltec V3 @ 21 dBm + 2.5 dBi antenna | 21.35 dBm | **136** | ✅ legal (12 dB headroom) |

**Maximum legal conducted TX power by antenna gain:**

| Antenna gain | Max legal P_TX | Headroom @21 dBm | Headroom @22 dBm |
|---|---|---|---|
| 1.5 dBi | 27.6 dBm | 6.6 dB | 5.6 dB |
| 2 dBi | 27.1 dBm | 6.1 dB | 5.1 dB |
| 2.5 dBi | 26.6 dBm | 5.6 dB | 4.6 dB |
| 3 dBi | 26.1 dBm | 5.1 dB | 4.1 dB |
| 5 dBi | 24.1 dBm | 3.1 dB | 2.1 dB |
| 8 dBi | 21.1 dBm | **0.1 dB** | ✗ over |

**Recommendation:** use a **Heltec WiFi LoRa 32 V3 (HTIT-WB32LA HF) at 21 dBm** for the gateway, and either cap the T-Beam to ~22 dBm in software or prefer the 22 dBm non-1W T-Beam. 21 dBm is ample for a 4-storey building (Section B.12). And remember **APC (Adaptive Power Control) is mandatory** at 500 mW — the SX1262/SX1276 don't do it automatically; you implement it in firmware (reduce TX power as RSSI improves).

## B.11 Practical indoor range — evidence, not a single number

### (a) Manufacturer claims — **MARKETING, treat as fiction**

| Source | Claim | Reality |
|---|---|---|
| Adafruit (RFM95W product page) | "They can easily go **2 Km line of sight** using simple wire antennas, or **up to 20Km** with directional antennas and settings tweakings" / "Range of approx. 2Km, depending on obstructions" | LoS outdoor, different antennas, different regulatory regime |
| MeshCore Ninja (T-Beam listing) | "**Typical Range: 18 km**" | Marketing aggregator. No test conditions given. |
| LilyGO T-Beam | (no range figure published) | — |
| Heltec | (no range figure published) | — |

**None of these is a 4-storey reinforced-concrete school.** Line-of-sight outdoor figures are meaningless indoors — free space path loss has a **−40 dB/decade** slope outdoors, but indoor multipath makes the effective slope far worse (−20 to −25 dB/decade), and every floor adds a fixed attenuation on top. **Do not let any of these numbers into a school project decision.**

### (b) Published field measurements — indoor, multi-floor, LoRa

This is the evidence that actually matters. Four independent studies:

| # | Study | Building | Frequency | Key measured results |
|---|---|---|---|---|
| **1** | **Xu, Kim, Huang, Kanhere, Jha, Hu** — "Measurement, Characterization and Modeling of LoRa Technology in Multi-floor Buildings", *IEEE Internet of Things Journal* 7(1):298–310, 2019. arXiv:1909.03900 | Office, **6 floors + basement, reinforced concrete**; also residential, car park, warehouse | 868 MHz class | **Measured floor attenuation factors: 1 floor = 21.7 dB; 2 floors = 25.9 dB; 3 floors = 27.2 dB; 4 floors = 50.8 dB.** Concrete wall 2.2 dB; wooden door 2.11 dB; glass 2.04 dB; soft partition 2.5 dB. Path-loss exponent n = 1.54–2.43. Shadowing σ up to 5.84 dB. |
| **2** | **Lira, Almeida, Sarmento, Ciraulo, Carvalho, Lopes** — "Measurement-Based Modeling of Large-Scale and Time-Varying Small-Scale Fading for LoRa in Indoor Multi-Floor Environments", *Sensors* 26(4):1152, Feb 2026 | 4-storey university building; **concrete walls, concrete ceilings with polystyrene panels**; 69 × 33 × 20 m | **433 MHz** | **FAF = 5.52 dB/floor, n = 2.53, σ = 6.93 dB.** Measured RSSI: same floor −98…−95 dBm; 1 floor −105…−106; 2 floors −107…−109; 3 floors −107…−112 dBm. Conclusion: *"LoRa can provide reliable, building-wide wireless sensor coverage."* |
| **3** | **Phokharatkul, Phaiboon, Pasomkusolsil** — "A Multifloor Path-Loss Model for 433 MHz LoRa-WAN in Concrete Buildings", 2023 | Two concrete buildings, 5 storeys (lab + parking) | 433 MHz | **FAF for 1–5 floors: 6.0, 10.8, 13.9, 17.0 dB (building 1); 9.8, 14.0, 17.2, 21.5 dB (building 2).** n = 2.03 and 1.99 (LoS). |
| **4** | **Bobkov, Rolich, Denisova, Leonov** — "Study of LoRa Performance at 433 MHz and 868 MHz Bands Inside a Multistory Building", *MWENT 2020* | **9-storey** HSE building (concrete floors) | **433 vs 868 MHz** | At 5th floor, PDR at SF7: **433 MHz = 37.2 %, 868 MHz = 79.4 %**. At 8th floor: **433 MHz = 32.2 %, 868 MHz = 97.6 %**. **Conclusion: *"for nine-story building with concrete floors it is better to deploy 868 MHz LoRa network at 10th spreading factor."*** |
| **5** | **"Performance Evaluations of LoRa Wireless Communication in Building Environments"** (PMC7412434) | **16-storey reinforced concrete**, Dalian University of Technology, China | sub-GHz | Same floor: PDR > 99.5 %, works through **nine cement walls**. **Across floors: "The PDRs in the far zone (B2, 15F, 16F) were close to 0 and even in the near zone (3F, 6F, 8F, 12F), there were still many PDRs less than 80%."** |

**URLs:** [1] https://arxiv.org/abs/1909.03900 (HTML: https://ar5iv.labs.arxiv.org/html/1909.03900) · [2] https://www.mdpi.com/1424-8220/26/4/1152 · [3] https://exa.ai/library/publication/yq3z2js2z5x · [4] https://publications.hse.ru/pubs/share/direct/362876361.pdf · [5] https://pmc.ncbi.nlm.nih.gov/articles/PMC7412434/

### ⚑ The single most important number in this report

**Measured floor attenuation factor for one concrete floor varies from ~5.5 dB to ~21.7 dB across these studies — a 4× spread.**

That is why "LoRa covers 4 floors" is not a fact anyone can state for your building. Study [2] used a *lightweight* concrete ceiling with polystyrene panels; study [1] used full reinforced concrete on a campus building. Your school is one of these, not the other, and you cannot tell which without measuring.

### (c) What the link budget actually depends on

**COMPUTED** from verified chip specifications (Semtech selection guide; Heltec/LilyGO datasheets; RFM95W datasheet) and the published FAFs above.

Design point: **gateway 21 dBm TX, 2.5 dBi antenna → EIRP 23.5 dBm → e.r.p. 21.35 dBm (136 mW e.r.p., comfortably legal)**. Receiver design sensitivity: **−134 dBm** (worst of Heltec's own two published figures; SX1262 silicon is specified to −148 dBm, module-level reality is −134 to −137).

**Free-space path loss at 866 MHz** (COMPUTED):
| Distance | FSPL |
|---|---|
| 10 m | 51.2 dB |
| 20 m | 57.2 dB |
| 30 m | 60.7 dB |
| 50 m | 65.2 dB |
| 100 m | 71.2 dB |

**Predicted RSSI and margin, applying both published indoor models to a 4-storey school** (COMPUTED):

| Distance | Floors | Lira model RSSI | Margin vs −134 dBm | Xu model RSSI | Margin |
|---|---|---|---|---|---|
| 30 m | 0 | −83.7 | 50.3 dB | −65.4 | 68.6 dB |
| 30 m | 1 | −89.3 | 44.7 dB | −87.1 | 46.9 dB |
| 30 m | 2 | −94.8 | 39.2 dB | −91.3 | 42.7 dB |
| 30 m | **3** | **−100.3** | **33.7 dB** | **−92.6** | **41.4 dB** |
| 30 m | 4 | −105.8 | 28.2 dB | −116.2 | 17.8 dB |
| 50 m | **3** | −105.9 | 28.1 dB | −97.0 | 37.0 dB |
| 50 m | 4 | −111.4 | 22.6 dB | −120.6 | 13.4 dB |

**Interpretation:** at 30 m and 3 floors separation, both models predict **33–41 dB of margin**. With σ ≈ 6.9 dB log-normal shadowing, a 33 dB margin is **≈4.8σ** — comfortable. Even the 4-floor worst case at 50 m retains 13–23 dB (≈2–3.4σ), which is workable but tight.

**So the physics says 3–4 floors at 21 dBm should work.** The things that will actually kill it are not floors:

| Real risk | Why it beats concrete floors |
|---|---|
| **Steel lift shaft / metal cupboard housing the gateway** | Conductor, not dielectric. Put the gateway antenna **outside** any metal enclosure, near a stairwell void or high on a wall. |
| **The robot's own chassis** | A robot with a steel frame and 2× 18650 cells + motor drivers can attenuate badly. **Mount the antenna clear of the chassis.** |
| **Reinforced-concrete roof / water tanks above** | Concentrates shadowing at the top floor. |
| **People, doors, movable partitions** | Study [2] measured time-varying small-scale fading from moving people/objects, characterised with a Markov model — i.e. the channel is genuinely time-varying indoors, unlike a static floor-loss number. |
| **Wi-Fi repeaters / boosters in the building** | See Section B.13. |

## B.12 Is there evidence for 3–4 floor indoor coverage specifically?

**Partially. Direct evidence exists for 4–6 floor buildings; nothing Indian.**

| Question | Answer | Status |
|---|---|---|
| Published 4-floor indoor LoRa measurement | **YES** — Lira et al. 2026, *Sensors* 26(4):1152, 4-storey building, building-wide coverage validated, FAF 5.52 dB/floor. | VERIFIED (peer-reviewed) |
| Published 6-floor + basement measurement | **YES** — Xu et al. 2019, IEEE IoT-J, reinforced concrete, with per-floor attenuation table. | VERIFIED (peer-reviewed) |
| Published 9-floor 868 MHz measurement | **YES** — Bobkov et al. 2020; recommends SF10 in a 9-storey concrete building. | VERIFIED (peer-reviewed) |
| Published measurement of a **school** building specifically | **NOT FOUND** | NOT FOUND |
| Any published LoRa measurement **in India** | **NOT FOUND** | NOT FOUND |
| A universal "LoRa reaches N floors indoors" figure | **NOT FOUND, and NOT FOUND is the correct answer.** Study [1] states explicitly: *"the FAF varies greatly in different buildings with varied frequencies. This result indicates that **a site-specific path loss model is required**."* | VERIFIED (as a negative result) |

### What a site survey must check — the physics basis

Since no direct evidence exists for your building, here is exactly what to measure and why. **COMPUTED thresholds shown.**

| # | Check | Method | Pass criterion |
|---|---|---|---|
| 1 | **Link margin at worst point** | Gateway fixed, robot walks/parks at the 4 worst spots (top floor far corner, bottom floor far corner, inside lift lobby, beside the boiler/water tank). Log RSSI + SNR per SF. | **RSSI ≥ −115 dBm** (≈19 dB above the −134 dBm SF12 sensitivity) at the worst point. 19 dB ≈ 2.7σ of shadowing. |
| 2 | **Airtime per packet** | Read the actual airtime your stack reports (`airtime_tx` in RadioLib, or the LoRa modem's own timer). Do not use a datasheet number. | **≤ 1.0 s** at the SF you will actually use for a ~25-byte payload. If SF12 gives >1.5 s, drop to SF10. |
| 3 | **Duty cycle actually consumed** | airtime ÷ reporting interval. | **≤ 2.5 %** measured on air (end device). See the table below. |
| 4 | **Packet delivery ratio** | 500+ packets at each of the 4 worst points. | **≥ 99 %** for unconfirmed; **≥ 95 %** for confirmed-with-retries. Study [5] shows same-floor PDR > 99.5 % is achievable indoors even through 9 walls. |
| 5 | **Gateway antenna siting** | Try 2–3 positions. | Pick the position that maximises the *worst-point* RSSI, not the average. |
| 6 | **APC behaviour** | Confirm firmware reduces TX power as RSSI improves. | Required at 500 mW e.r.p. under Table-II. |

### Duty-cycle budget — this will constrain your design more than range will

**COMPUTED.** Airtime computed from the LoRa time-on-air formula documented in the RFM95W/96W/98W datasheet §4.1.1.7 (formulae are images in the PDF; values below are my computation, shown as a range because header mode / coding rate / the Low-Data-Rate-Optimise flag shift the result by up to ~2×).

| SF / BW125 | Airtime, ~25 B payload | Duty @ 1 pkt/min | Duty @ 1 pkt/2 min | Verdict vs 2.5 % |
|---|---|---|---|---|
| SF7 | 38–78 ms | 0.06–0.13 % | 0.03–0.07 % | ✅ ample |
| **SF9** | **152–303 ms** | **0.25–0.51 %** | **0.13–0.25 %** | ✅ **comfortable — recommended** |
| SF10 | 303–598 ms | 0.50–1.00 % | 0.25–0.50 % | ✅ fine |
| **SF12** | **1180–1573 ms** | **1.97–2.62 %** | **0.98–1.31 %** | ⚠️ **1 pkt/min breaches 2.5 %** |

**LoRaWAN confirmed uplink = uplink + downlink**, so double the figures. **Consequences:**

- **At SF12: cap your robot's reporting rate at 1 packet per 2–5 minutes.** A "heartbeat every 60 s at SF12, confirmed" is a regulatory breach, not just an engineering choice.
- **At SF9 or SF10, 1 packet/minute confirmed is comfortably legal** and you keep near-maximum link margin.
- Airtime also **collides**: two robots on the same channel and SF with unsynchronised transmissions can both be lost. Study [2] characterised exactly this as time-varying small-scale fading; study on LoRaWAN capacity literature attributes it to ALOHA collisions. Use **distinct channels or SF per robot** if you have more than a handful of robots.

## B.13 Coexistence with Wi-Fi 2.4 GHz

### ⚑ FLAG: at 865–868 MHz, this question is mostly moot

**LoRa at 865–868 MHz and Wi-Fi at 2.4–2.4835 GHz are separated by ~1.5 GHz — roughly 5.7× the LoRa bandwidth, and over 1000× in frequency.** There is no shared band, therefore **no in-band interference**. The only coupling is via **spurious and harmonic emissions**:

| Interference path | Magnitude | Assessment |
|---|---|---|
| LoRa fundamental vs Wi-Fi | none | Bands disjoint |
| LoRa 2nd harmonic (1.73–1.74 GHz) | spurious-suppressed by the module's transmit filter/matching | Irrelevant to 2.4 GHz Wi-Fi |
| LoRa 3rd/4th harmonic → 2.6–3.5 GHz | spurious-suppressed | Irrelevant |
| **Wi-Fi transmitter spur → 865 MHz** | Wi-Fi occupied bandwidth is 2402–2472 MHz; its harmonics land at 4.8–4.9 GHz and 7.2–7.4 GHz. Its **spurious** emissions at 865 MHz are limited by EN 300 328 / FCC Part 15 to ≪ −30 dBm/mHz. | **Negligible.** |
| **A 2.4 GHz LoRa (SX1280) module, if you used one** | **This IS a real problem** | **Avoid.** |

### ⚑ FLAG: you will find a large literature on "LoRa and Wi-Fi coexistence" that does not apply to you

The academic literature is almost entirely about **2.4 GHz LoRa (SX1280/SX1281)** — a Semtech part for 2.4 GHz CSS — sharing the band with Wi-Fi. Examples:

> "…our results indicate that a LoRa packet can be successfully received if the **WiFi received signal power is not higher than 25 dB than the LoRa one**, on average… A **WiFi transmission occupies 20 MHz and can arrive 25–40 dB stronger than the overlapping LoRa signal.**"
>
> — Saduakhas et al., "LoRa and WiFi at 2.4 GHz: a Cross-Technology Interference Evaluation", 2025

> "…(3) the deployment of the **no**des in the overlapping case… the Wi-Fi FDR can undergo **huge losses** depending on the modulation of the IEEE 802.11 standard used…"
>
> — Hochet & Derevianckine thesis, https://hal.science/tel-04858023v1/file/Manuscrit_these_HOCHET_DEREVIANCKINE_Gwendoline.pdf

> "…**SF12 holds PRR at 1.0 regardless of WiFi density; SF5 drops below 0.5 with five nearby access points.**"
>
> — Saduakhas, MSc thesis, "Cross-technology interference between LoRa 2.4 GHz and WIFI"

**If you ever switch to a 2.4 GHz LoRa module to avoid the sub-GHz band, you would inherit all of these problems** — a 20 MHz Wi-Fi channel arriving 25–40 dB stronger will destroy your LoRa packets at anything below SF12. **This is the strongest technical argument in the whole report for staying on sub-GHz LoRa in India**, which is also the band where the Indian rules are clearest.

**Practical answer for the school:** put Wi-Fi wherever you like. It cannot interfere with 865–868 MHz LoRa in any meaningful way. The reverse is also effectively true.

---

# SECTION C — REGULATORY / PRACTICAL SUMMARY

## C.14 What a school project actually needs

### Split: regulatory vs institutional

| # | Item | Owner | Type | Trigger | Cost / effort | Status |
|---|---|---|---|---|---|---|
| 1 | **Confirm the module operates in 865–868 MHz** | Project lead | **Regulatory** (self-check) | Always | Free — check datasheet + set channels ≤867.9 MHz | VERIFIED rules |
| 2 | **Cap TX power so e.r.p. ≤ 500 mW** (= P_TX + G_ant ≤ 29.15 dBm for the 2.5 %/10 % row) and **implement APC** | Firmware | **Regulatory** | Always | Free | VERIFIED rules |
| 3 | **Keep end-device duty cycle ≤ 2.5 %, gateway ≤ 10 %** | Firmware + reporting rate | **Regulatory** | Always | Free | VERIFIED rules |
| 4 | **ETA held by the supplier for the exact model you buy** | Supplier/importer | **Regulatory** | On import/sale | **₹10,000 per model, held by supplier**, self-declared, lifetime | VERIFIED |
| 5 | **Import clearance** (if importing yourself) | Importer | **Regulatory** | Only if you import directly | DGFT + WPC import route | VERIFIED |
| 6 | **Confirm you need no possession authorisation** | School administration | **Regulatory** (new 2026 regime) | Rule 10 exemption | See below | VERIFIED rules, **interpretation UNVERIFIED** |
| 7 | **Site survey** (RSSI/PDR/airtime/duty at 4 worst points) | Project lead | Engineering | Always | 1–2 days | See B.12 |
| 8 | **Building / rooftop / antenna works** | School + contractor | **Institutional** | If mounting anything permanent | Local | — |
| 9 | **IT acceptable-use, exam-hall RF policy** | School IT/Principal | **Institutional** | Always | Local | — |
| 10 | **Parent/student notification** | School | **Institutional** | Judgment | Local | — |

### ⚑ Item 6 is the one genuinely new, genuinely uncertain item

Since **06.08.2026**, possession of radio equipment requires authorisation under s.3(1)(c) of the Telecommunications Act 2023, governed by the **Telecommunications (Radio Equipment Possession Authorisation) Rules, 2026** (G.S.R. 592(E), 8 July 2026). The rules, verbatim:

> "**10. Possession by persons not requiring authorisation.** — An authorisation under these rules shall not be required if a person—
> (a) holds a licence, registration, permission, by whatever name called, under the Indian Telegraph Act, 1885 (13 of 1885) or the Indian Wireless Telegraphy Act, 1933 (17 of 1933) and continues to operate under the terms and conditions of such licence, registration or permission under sub-section (6) of section 3 of the Act;
> …
> (c) holds an **exemption** under sub-section (3) of section 3 of the Act **and an exemption from the requirement of assignment** under sub-section (6) of section 4 of the Act;
> …
> (g) is in possession of radio equipment that can accommodate not more than four subscriber identity modules; or
> (h) is a user who holds proof of subscription to a telecommunication service provided by an authorised entity.
>
> **Explanation.**—For the removal of doubts, it is clarified that a person as referred to in this rule **shall not manufacture, sell or let for hire** the radio equipment in its possession."
>
> — https://eservices.dot.gov.in/sites/default/files/circular-notifications/telecommunications-radio-equipment-possession-authorisation-rules-2026.pdf

**How this likely applies to a school — flagged as interpretation, not fact:**

| Route | Likely outcome |
|---|---|
| **Rule 10(c)** — a licence-exempt SRD is, by definition, exempt from the requirement of frequency assignment under s.4(6). A school running only 865–868 MHz equipment should sit inside this exemption. | **Most likely the applicable route.** But rule 10(c) requires holding "an exemption under sub-section (3) of section 3" — i.e. a formal exemption notification, which the G.S.R. 853(E) exemption arguably is. **I did not find a DoT clarification stating this explicitly. NOT VERIFIED.** |
| **Rule 10(h)** — the school is a "user who holds proof of subscription to a telecommunication service provided by an authorised entity". A school with an internet connection from an ISP would qualify. | **Plausible alternative route.** Plausible, not confirmed by guidance. |
| Rule 4(1) — the rules apply if you intend to "**deal** with such radio equipment through manufacture, purchase or import for sale, let for hire, repair or testing or demonstration" | **Does not apply** to a school that simply buys and operates. |
| Rule 4(2)(c) — "the Central Government or a State Government or any agency designated…" | A government school may qualify; a private school is not clearly an "agency". |

**Action:** raise a ticket to the DoT WPC Helpdesk (`https://eservices.dot.gov.in/`) and get the answer in writing before import. This is a **low-cost, high-value action** — the answer is one email, and getting it wrong on a 2026-vintage rule is the kind of mistake that is expensive to unwind.

**Fee context, if you did need it** (UNVERIFIED, from secondary reporting of G.S.R. 592(E)): ₹1,000 non-refundable application fee; ₹10,000/year for dealing (1–5 years); **₹2,000/year, minimum ₹500, for testing/demonstration only, valid up to 12 months**. If any of this applies to the project, the testing-only route is cheap.

## C.15 What a low-bandwidth backup channel is actually worth

### Concrete payload budget: 100–1000 bytes/minute is roughly 1–3 reports/minute of ~350 bytes

**COMPUTED** airtime for a ~350-byte LoRaWAN payload (SF9/BW125):

| Payload | SF7 | SF9 | SF10 | SF12 |
|---|---|---|---|---|
| 100 B | 77 ms | 307 ms | 614 ms | 4063 ms |
| 250 B | ~215 ms | ~850 ms | ~1700 ms | ~11 s ⚠️ |
| 350 B | ~295 ms | ~1180 ms | ~2350 ms | ~16 s ⚠️ |
| 1000 B | ~830 ms | ~3300 ms | ~6600 ms | ~34 s ❌ |

**At SF9/BW125 a 350-byte report is ~1.2 s of airtime → 1 report/minute = ~2 % duty cycle.** That is inside the 2.5 % cap but with almost no margin. **At SF12, 350 bytes takes ~16 s of airtime — a single report is 0.27 % and you get one report every 2 minutes max.**

### What fits in ~350 bytes — and it is genuinely useful

| Field | Bytes (packed) | Example |
|---|---|---|
| Device ID | 4 | `RBT-0042` as uint32 |
| Sequence / uptime | 4 | monotonic counter |
| Battery % + voltage mV | 2 | `83` , `4012` |
| Charging state | 1 | bool |
| Robot state enum | 1 | IDLE / MOVING / CHARGING / FAULT / E-STOP |
| Fault code | 2 | enumerated |
| Fault detail | 4 | e.g. motor current, which wheel |
| Position X, Y (cm) | 8 | int32 × 2 |
| Heading (deg) | 2 | uint16 |
| Pose confidence / fix type | 1 | dead-reckoned vs last-known |
| Wi-Fi RSSI | 1 | int8 |
| LoRa SNR | 1 | int8 |
| Task/zone ID | 2 | enum |
| Payload CRC | 2 | |
| **Total** | **~35 bytes** | — |

**With LoRaWAN's 13-byte overhead and a 350-byte budget you have ~320 bytes of application data — roughly 9× what the field list above needs.** You could carry a short text log line, a mission ID, or a compressed diagnostic block.

### ⚑ The honest limits — what this channel is NOT

| Not possible | Why, numerically |
|---|---|
| **Camera stills** | ESP32-CAM JPEG ≈ 20–60 kB. At SF9 that is 20–60 seconds of airtime **per frame**, and duty cycle >100 %. Illegal and impossible. |
| **Video / streaming** | Not a closeable question. |
| **Lidar scan** | Even a 1/10th-resolution occupancy grid of a small room is kilobytes. A single LiDAR point pack (e.g. 64 points × 3 coords) ≈ 384 bytes at best — one frame, no rate. |
| **Motor control loop / closed-loop control** | LoRaWAN round-trip at SF9 is dominated by airtime and the duty-cycle-governed retry schedule, giving **hundreds of ms to seconds**, non-deterministic, with no QoS. Closed-loop control needs a local, deterministic link. **Never put a control loop on LoRa.** |
| **Reliable high-rate position** | A stale "last known position" is fine as a fallback; it is useless for navigation. |
| **Data you need in <1 s** | Sub-GHz LoRa's minimum airtime at SF7 is still ~38 ms, and most deployments will run SF9–SF12 for building penetration. |

### ⚑ So what *is* it worth? — the actual use case

LoRa earns its place **only** as a **cold-start / heartbeat / fault channel** that survives the failure of the primary link:

| Scenario | Primary (Wi-Fi) | LoRa backup |
|---|---|---|
| Robot on floor 4, AP rebooted or cable cut | Total blackout. Robot stops, nobody knows why or where. | Heartbeat continues. Console shows robot ID, last known position, battery, state. **This is the whole value.** |
| Robot drives out of Wi-Fi range (stairwell, sports hall, courtyard, gate) | Lost. | Still reported. |
| AP/POE switch failure or power cut | Fleet-wide loss of visibility | Each robot independently reports "I am here, at 42 % battery, in FAULT-BEARING-JAM" |
| Primary link congested during a demo | Latency spike, telemetry stalls | Independent low-rate channel unaffected |
| Battery health trending down over days | Invisible until it fails | Trend is visible days in advance from 1-byte reports |

**This is a genuinely valuable, low-cost add-on — provided the primary path (Wi-Fi) is the real system and LoRa is the watchdog.** It is *not* a replacement for Wi-Fi and should not be built as one.

---

# SECTION D — ALTERNATIVES

## D.16 Comparison for this specific use case

| | **LoRa 865–868 MHz** | **ESP-NOW (2.4 GHz)** | **Plain Wi-Fi / MQTT** | **BLE** |
|---|---|---|---|---|
| **Multi-floor reliability (4-floor RC school)** | **Good.** 33–41 dB predicted margin at 30 m / 3 floors; validated in 4–6 storey concrete buildings (studies [1][2][4]). | Good within one AP's coverage; degrades at floor transitions like any 802.11 link. No published multi-floor ESP-NOW evidence found. | **Depends entirely on AP placement.** With ≥1 AP per floor, excellent. | Marginal. Studies report ESP-NOW reaches **~15× further than BLE** at ~2× the power. |
| **Throughput** | 0.3–37.5 kbps (BW125 SF12→SF7). Airtime 38 ms – 16 s. | ~10 Mbps claimed. ESP-NOW max 250 B/frame. | 10s–100s Mbps. | ~2 Mbps theoretical, much less in practice. |
| **Latency determinism** | Poor. SF12 = 1–16 s airtime. Not for control. | Good (~220 ms round trip measured). | Good. | Fair. |
| **Power draw** | **Best for low-rate.** SX1262 RX 4.2 mA, TX ~118 mA @22 dBm. **50 mA @10 dBm, 130 mA @20 dBm** (Heltec V2 measured table, below). | **Worst of the four for anything non-trivial.** 129 mA TX / 121 mA RX measured. | **High.** ESP32 Wi-Fi AP mode 135 mA; scan mode 115 mA; TX spikes to 400–950 mW peaks. Connection association is the killer: **~3.9 s** just to wake+DHCP+TLS+MQTT. | **Lowest.** 74–76 mA measured. |
| **Battery life implication** | ~10 h on 2000 mAh at default settings; but at **10 dBm** TX LoRa beats Wi-Fi at 30 s intervals. | Comparable to Wi-Fi at best. | **Worst.** An ESP-NOW node ran **6.4× longer** than the same node doing Wi-Fi+MQTT (3.7 years vs 6.9 months, 2500 mAh). | Best absolute, but range-limited. |
| **Cost** (India, approximate) | ₹350–900 / board (Heltec V3 HF) | ₹200–400 (plain ESP32) | ₹0 marginal (AP already exists) | ₹150–300 |
| **Indian regulatory burden** | **ETA required** (₹10,000, self-declared, held by supplier). 865–868 MHz, 2.5 %/10 % duty, ≤200 kHz, 500 mW e.r.p. + APC. | ETA required (2.4 GHz SRD, G.S.R. 45(E)). Much lighter in practice — consumer grade. | Already covered by the school's existing AP purchase. | ETA required. Much lighter. |
| **Survives Wi-Fi outage?** | ✅ **Yes — that is the point.** | ❌ No | ❌ No | ❌ No |
| **Coexists with Wi-Fi** | ✅ Bands disjoint, no interference | ⚠️ Shares 2.4 GHz; ESP-NOW radio cannot run concurrently with Wi-Fi on the same ESP32 | n/a | ⚠️ Shares 2.4 GHz, but channelised |

**Power figures (UNVERIFIED) — Heltec WiFi LoRa 32 v2, same board doing both radios:**

| Mode | Consumption |
|---|---|
| LoRa TX 10 dBm | 50 mA |
| LoRa TX 12 dBm | 60 mA |
| LoRa TX 15 dBm | 110 mA |
| LoRa TX 20 dBm | 130 mA |
| **Wi-Fi AP mode** | **135 mA** |
| **Wi-Fi scan mode** | **115 mA** |

> "As the manufacturer stated, the power consumption for **both WiFi and LoRa transmissions is practically the same** with transmission powers of 17 dBm or 18 dBm. It is surprising as LoRa is branded as a low-power consumption communication protocol. Therefore, **the transmission power of LoRa has to be decreased so as to improve its battery consumption, which leads to less coverage.**" … "WiFi **outperformed LoRa** with a 30 second transmission interval."
>
> — "WiFi and LoRa Energy Consumption Comparison in IoT", https://hal.science/hal-04083109v1/document

**⚑ Honest caveat:** this HAL paper used **SX1278** — a 433/470 MHz part that **cannot legally operate at 865–868 MHz in India** — and it measured *active-mode* current on a board with an OLED. Treat as directional, not absolute.

**Wi-Fi association cost, UNVERIFIED but decisive:**

> "It takes several seconds for the ESP32 to wake up, connect to WiFi, get an IP address from the DHCP server and to connect and send a message over MQTT. **3.877 seconds**, to be precise." … "Now let's use the … ESP-NOW … the node is active **564ms**! The total runtime increases from 6.9 months to **3.7 years**!!"
>
> — ThingPulse, https://thingpulse.com/esp32-ultra-long-battery-life-with-espnow/

**⚠️ Important caveat on ESP-NOW:** *"WiFi and ESP-NOW cannot run the same time"* on a single ESP32's radio (ThingPulse, same source). A gateway bridging ESP-NOW→Wi-Fi must time-slice, dropping messages during the Wi-Fi window. ESP-NOW is also hard-limited to **10 encrypted peers / 20 unencrypted** and **250 bytes** per frame.

## D.17 Would a school with Wi-Fi across all blocks not need LoRa at all?

### **Status: PARTIALLY VERIFIED — and the answer is nuanced.**

| Evidence | Direction | Status |
|---|---|---|
| Wi-Fi multi-floor works *if* APs are placed per floor. Study [5] (16-storey RC building) shows even single-floor LoRa penetrating nine cement walls at PDR > 99.5 % — i.e. **2.4 GHz is not the weak link in a reinforced-concrete building, the AP placement is.** | Favours Wi-Fi | VERIFIED (peer-reviewed) |
| **BUT** the same study found that across floors, "**the PDRs in the far zone were close to 0** and even in the near zone there were still many PDRs less than 80%." **LoRa — the technology with 25× the link budget — still failed** where 2.4 GHz Wi-Fi would fail much sooner. | Favours LoRa | VERIFIED (peer-reviewed) |
| ESP32 Wi-Fi + TLS + MQTT association costs **~3.9 s per message** vs ESP-NOW 564 ms — a 6.9× battery-life penalty. | Favours the low-power path for battery nodes | UNVERIFIED |
| Study [4] (9-storey concrete) concluded **868 MHz at SF10** was the right choice — *not* a Wi-Fi solution. | Favours LoRa | VERIFIED (peer-reviewed) |

### The decisive question: **what is LoRa actually buying you?**

It is **not** coverage. Study [5] is decisive on this: LoRa still lost the link across a 16-storey building even at 2 W-adjacent power levels. **Wi-Fi with one AP per floor will beat LoRa for normal coverage in a 4-storey school.** Anyone claiming otherwise is selling something.

LoRa buys you **two specific things**:

| # | Benefit | Real? | Quantified |
|---|---|---|---|
| **1** | **Independence from the Wi-Fi infrastructure.** A separate radio, separate antenna, separate power. Survives AP reboot, PoE switch failure, cable cut, config error, credential expiry, or DDoS/misconfiguration on the WLAN. | ✅ **Yes — this is the real and complete answer.** | An outage of the primary link is a *binary* event. LoRa turns a total blackout into a 1-byte-per-minute trickle. |
| **2** | **Battery node efficiency**, because Wi-Fi association dominates energy cost. | ⚠️ Only if the robot runs on battery and is expected to run for days. If the robot is docked and mains-powered, **this benefit is worth nothing.** | 6.9× battery life, ESP-NOW not LoRa. |

### ⚑ Decision rule

> **If the robot is mains-powered or docked, and the school already has ≥1 working Wi-Fi AP per floor: a local Wi-Fi mesh makes LoRa largely redundant.**
> 
> **If any of these are true, LoRa earns its keep:**
> - The robot runs on battery for more than a few hours.
> - There are stairwells, lifts, a courtyard, a sports hall, or a gate where no AP reaches.
> - You need to distinguish "robot is fine and Wi-Fi is down" from "robot is lost" during a demo or an exam-day failure.
> - You want an independent channel that cannot be knocked over by the same change that broke everything else.

**And note the cost of getting this wrong in either direction:**
- **Over-building (LoRa when Wi-Fi sufficed):** ~₹3,000–8,000 of hardware, a second radio and antenna on a robot that already has Wi-Fi, a second failure domain to maintain, one more thing for the school IT team to support. Real but modest.
- **Under-building (no fallback):** during the one demo where the AP is down, the robot is invisible and the project "failed". Also modest.

**This is a judgement call about failure tolerance, not a technical one.** My read: for a **school** project where the robot is almost certainly docked/mains-powered between runs, and the building already has Wi-Fi, the honest recommendation is **build the Wi-Fi path first and make it good; add the LoRa heartbeat only if (a) you have battery-life budget pressure, or (b) you can name a specific room where Wi-Fi does not reach.** A ~350-byte LoRa heartbeat at SF9 every 60 s is cheap insurance and will not change your architecture — which is exactly why it is a reasonable thing to add late rather than early.

---

# APPENDIX 1 — Complete source list

## Official Indian Government sources (primary)

| Source | URL | Used for |
|---|---|---|
| **NFAP-2025** (National Frequency Allocation Plan, in force 30 Dec 2025; 230 pp.) | https://www.dot.gov.in/static/uploads/2026/02/b110cdc386d3a4e41c8483d7ffd7c410.pdf | A.1–A.6; Annexure 1 exemption table |
| DoT press release on NFAP-2025 | https://www.dot.gov.in/static/uploads/2026/02/52e468605d5b63c6dfedcf538f1d7d36.pdf | Context |
| **G.S.R. 853(E), 10 Dec 2021** — 865–868 MHz SRD Rules (full gazette text) | https://thc.nic.in/Central%20Governmental%20Rules/use%20of%20low%20power%20Equipment%20in%20the%20frequency%20band%20865%20to%20868%20MHz%20for%20Short%20Range%20Devices%20Exemption%20from%20Licence%20Rules%2C2021.pdf | **A.2, A.3, A.4** — the core document |
| DoT page for G.S.R. 853(E) | https://www.dot.gov.in/spectrummanagement/use-low-power-equipment-frequency-band-865-868-mhz-short-range-devicesexemption | A.2 |
| DoT WPC & SACFA description | https://www.dot.gov.in/spectrum-management/2457?page=1 | A.1 |
| **ETA service page** (fee ₹10,000, validity, exclusions) | https://eservices.dot.gov.in/equipment-type-approval-eta | A.4 |
| **O.M. R-11017/01/2018-PP(part-1), 09.09.2024** (ETA self-declaration) | https://eservices.dot.gov.in/sites/default/files/2024-11/Issuance%20of%20Equipment%20Type%20Approval%20%28ETA%29%20forLicense-Exempt.pdf | A.4 |
| O.M. ETA-WPC/Policy/2018-19, 26.02.2019 — **scanned, no text layer** | https://eservices.dot.gov.in/sites/default/files/circular-notifications/ETA%20through%20Self%20declaration%20for%20Certain%20Categories%20of%20Wireles%20Equipment%20dt%2026-02-2019_0%20%281%29.pdf | A.4 (UNVERIFIED) |
| ETA Self-Declaration Applicant Manual (13.06.2023) | https://eservices.dot.gov.in/sites/default/files/user-mannual/ETA%20Self%20Declaration%20Applicant%20%20Manual.pdf | A.4 |
| SaralSanchar ETA report / portal | https://www.saralsanchar.gov.in/wpc_eta_report.php | A.1, A.4 |
| **Telecommunications (Radio Equipment Possession Authorisation) Rules, 2026** (G.S.R. 592(E)) | https://eservices.dot.gov.in/sites/default/files/circular-notifications/telecommunications-radio-equipment-possession-authorisation-rules-2026.pdf | **C.14 item 6** |
| DoT Acts, Rules & Notifications index | https://eservices.dot.gov.in/act-and-rules | Document inventory |
| Authorisation Portal (Telecom Act 2023 regime) | https://eservices.dot.gov.in/authorisation-portal | C.14 |
| **Telecommunications Act, 2023** (44 of 2023), full text | https://egazette.gov.in/WriteReadData/2023/250880.pdf | A.1, C.14 |
| India Code entry for the Act | https://www.indiacode.nic.in/handle/123456789/20101 | A.1 |
| SRD Amendment Rules 2024 (G.S.R. 271(E)) — confirms **no** change to 865–868 | https://eservices.dot.gov.in/sites/default/files/circular-notifications/use-of-low-power-and-very-low-power-short-range-radio-frequency-devices-exemption-from-licensing-requirement-amendment-rules-2024.pdf | A.2 |
| WPC Import License page | https://www.eservices.dot.gov.in/import-license | A.4 |
| **BIS official FAQ** (modified 28.09.2022) | https://www.crsbis.in/BIS/app_srv/tdc/gl/docs/Modified_FAQs.pdf | **A.7** |
| **Policy on Jammers**, Cabinet Secretariat | https://cabsec.gov.in/writereaddata/circular/policyonjammer/english/1_Upload_3582.pdf | **A.8** |
| TRAI EMF Information Paper (30.07.2014) | https://www.trai.gov.in/sites/default/files/2024-10/EMF_Information_Paper_30.07.2014.pdf | A.8 |
| TEC — EMF radiation from IoT devices / TEC 13016:2023 | https://www.tec.gov.in/pdf/M2M/Technical%20Report_EMF%20radiation%20from%20IoT%20devices.pdf | A.8 |
| DoT Circular 1 of 2026 (interim spectrum assignment) | https://www.dot.gov.in/static/uploads/2026/04/284f2d11a06a1c7936cb42a2811d9638.pdf | Context |

## Vendor datasheets

| Source | URL |
|---|---|
| Heltec WiFi LoRa 32 V3.2 official datasheet | https://resource.heltec.cn/download/WiFi_LoRa_32_V3/HTIT-WB32LA_V3.2.pdf |
| Heltec WiFi LoRa 32 V3 product page | https://heltec.org/project/wifi-lora-32-v3/ |
| Heltec hardware update log (authoritative on V2→V3) | https://docs.heltec.org/en/node/esp32/wifi_lora_32/hardware_update_log.html |
| LilyGO T-Beam 1W (GitHub docs) | https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series/blob/master/docs/en/t_beam_1w_sx1262/t_beam_1w_sx1262.md |
| LilyGO T-Beam 1W wiki | https://wiki.lilygo.cc/products/t-beam-series/t-beam-1w/ |
| Meshtastic T-Beam hardware docs | https://meshtastic.org/docs/shana/hardware/devices/lilygo/tbeam/ |
| **Semtech Wireless Selection Guide** (chip ranges/sensitivity) | https://www.semtech.com/uploads/design-support/SG-SEMTECH-WSP.pdf |
| **RFM95W/96W/98W datasheet v2.0** (Hope Micro) | https://cdn.sparkfun.com/assets/a/9/6/1/0/RFM95W-V2.0.pdf |
| Adafruit RFM95W product page | https://www.adafruit.com/product/3072 |

## Peer-reviewed literature

| Paper | URL |
|---|---|
| Xu et al., "Measurement, Characterization and Modeling of LoRa Technology in Multi-floor Buildings", IEEE IoT-J 7(1):298–310, 2019 | https://arxiv.org/abs/1909.03900 |
| Lira et al., "Measurement-Based Modeling of Large-Scale and Time-Varying Small-Scale Fading for LoRa in Indoor Multi-Floor Environments", Sensors 26(4):1152, 2026 | https://www.mdpi.com/1424-8220/26/4/1152 |
| Bobkov et al., "Study of LoRa Performance at 433 MHz and 868 MHz Bands Inside a Multistory Building", MWENT 2020 | https://publications.hse.ru/pubs/share/direct/362876361.pdf |
| "Performance Evaluations of LoRa Wireless Communication in Building Environments" (16-storey RC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC7412434/ |
| Phokharatkul et al., "A Multifloor Path-Loss Model for 433 MHz LoRa-WAN in Concrete Buildings", 2023 | https://exa.ai/library/publication/yq3z2js2z5x |
| Polák & Miloš, "Performance analysis of LoRa in the 2.4 GHz ISM band: coexistence issues with Wi-Fi", Telecommunication Systems 74(3), 2020 | https://link.springer.com/article/10.1007/s11235-020-00658-w |
| Saduakhas, Kadirzhanov, Zorbas, "LoRa and WiFi at 2.4 GHz: a Cross-Technology Interference Evaluation", 2025 | https://exa.ai/library/publication/s9vcwsv6rk5 |
| Saduakhas, "Cross-technology interference between LoRa 2.4 GHz and WIFI", MSc thesis, 2026 | https://exa.ai/library/publication/dd6wt5j2gcj |
| Hochet & Derevianckine, "Feasibility and Performance of a LoRa 2.4 GHz Network" (thesis) | https://hal.science/tel-04858023v1/file/Manuscrit_these_HOCHET_DEREVIANCKINE_Gwendoline.pdf |
| Chen et al., "LoFi: Enabling 2.4GHz LoRa and WiFi Coexistence", INFOCOM 2021 | https://desword.github.io/paper/infocom21-lofi.pdf |
| Gawlowicz et al., "Wi-Lo: Emulating LoRa using COTS WiFi", INFOCOM 2021 | https://www.tkn.tu-berlin.de/bib/gawlowicz2021wi-lo/gawlowicz2021wi-lo.pdf |
| InMAC: Interference-Aware MAC for 2.4 GHz LoRaWAN | https://discovery.ucl.ac.uk/id/eprint/10219850/1/InMAC_An_Interference-Aware_MAC_Protocol_for_2.4_GHz_LoRaWAN.pdf |
| ESP-NOW indoor performance evaluation | https://www.researchgate.net/publication/369626626_Indoor_Performance_Evaluation_of_ESP-NOW |

## Secondary / industry (UNVERIFIED)

| Source | URL | Used for |
|---|---|---|
| G.S.R. 45(E) 2005 (2.4 GHz) text | https://www.legitquest.com/act/use-of-low-power-equipment-in-the-frequency-band-24-ghz-to-24835-ghz-exemption-from-licensing-requirement-rules-2005/9F50 | A.5 |
| G.S.R. 1046(E) 2018 (UWB) text | https://www.legitquest.com/act/use-of-very-low-power-ultra-wideband-devices-exemption-from-licensing-requirements-rules-2018/9F59 | A.6 |
| "Framework for Wireless Local Area Network" 2024 | https://globalvalidity.com/wp-content/uploads/2024/10/2024-09-23-WLAN-Framework-Final-05-09-2024.pdf | A.5 |
| linux-wireless thread on India's 2.4 GHz rules | https://lists.infradead.org/pipermail/wireless-regdb/2023-March/001412.html | A.5 |
| TEC Final Revised Wi-Fi Essential Requirements | https://www.tec.gov.in/pdf/consultations/Final%20Revised%20Wi-Fi%20ER%20(1).pdf | A.5 |
| Khaitan & Co. analysis of REPA Rules 2026 | https://www.khaitanco.com/sites/default/files/2026-07/ERGO_Radio_14072026.pdf | C.14 |
| CBIC Instruction 16/2022-Customs (import licensing) | https://lexcomply.com/rsjadmin/news/202207253844Instruc%2016-2022-Customs%20Compendium%20in%20regard%20to%20Import%20licens%20requirement%20from%20WPC%20wing%20for%20import%20of%20wireless%20equipment.pdf | A.4 |
| COAI on illegal jammers at educational institutes (ET, 2025) | https://economictimes.indiatimes.com/industry/telecom/telecom-news/illegal-jammers-at-educational-institutes-boosters-at-home-firms-hampering-mobile-services-coai/articleshow/117920396.cms | A.8 |
| Wi-Fi vs LoRa energy on Heltec V2 (HAL) | https://hal.science/hal-04083109v1/document | D.16 |
| ThingPulse — ESP32 ultra-long battery life with ESP-NOW | https://thingpulse.com/esp32-ultra-long-battery-life-with-espnow/ | D.16 |
| marending — ESP protocol comparison | https://www.marending.dev/notes/esp-protocol/ | D.16 |

---

# APPENDIX 2 — Explicit NOT FOUND list

Recorded so the next reader does not repeat the search.

| Item searched for | Result |
|---|---|
| ETSI EN 303 708 in any Indian rule | **NOT FOUND.** India cites EN 300 220 and EN 302 208 instead. |
| Any published LoRa indoor measurement **in India** | **NOT FOUND.** |
| Any published LoRa measurement in a **school** building | **NOT FOUND.** |
| A universal "LoRa covers N floors indoors" figure | **NOT FOUND** — and Xu et al. explicitly state a site-specific model is required. |
| An Indian rule specifically restricting RF devices in schools | **NOT FOUND.** (Jammers/repeaters in schools are separately illegal.) |
| Official Gazette PDF of G.S.R. 45(E) 2005 | **NOT FOUND retrievable.** Substance verified via NFAP-2025 Annexure 1 + secondary reproduction. |
| Official Gazette PDF of G.S.R. 1046(E) 2018 | **NOT FOUND retrievable.** Substance via secondary reproduction. |
| Official PDF of the 2024 "Framework for Wireless Local Area Network" | **NOT FOUND retrievable.** |
| Semtech AN1200.22 (LoRa airtime app note) | **NOT FOUND retrievable** — semtech.com returns HTML. Airtime values in B.12 are **COMPUTED** from the formula documented in the RFM95W datasheet §4.1.1.7. |
| DoT guidance explicitly stating a school needs no possession authorisation under REPA Rules 2026 | **NOT FOUND.** — see C.14, raise a Helpdesk ticket. |
| A reachable copy of wpc.dot.gov.in | **NOT FOUND** (host resolves; no response from this network). Use `eservices.dot.gov.in` instead. |
| `wpc.gov.in` | **DOES NOT EXIST** (NXDOMAIN). Frequently cited in error by secondary sources. |

---

# APPENDIX 3 — Pre-flight checklist

```
[ ] Buy HTIT-WB32LA (the HF / 863-928 MHz Heltec V3). NOT the LF (470-510) variant.
[ ] Confirm with the seller that the module has a valid WPC ETA (self-declaration),
    and get the ETA number. Should be on saralsanchar.gov.in.
[ ] Set LoRa channels to ≤ 867.9 MHz. DO NOT use the EU868 default 868.1 MHz.
    (Heltec's own region table says IN868 = 865-867 MHz — go with 865.1-867.0 to be safe.)
[ ] Set bandwidth 125 kHz (Table-II requires ≤ 200 kHz; Table-III's >600 kHz row
    would not cover BW250, so stay at 125 kHz).
[ ] Cap TX power: P_TX(dBm) + antenna gain(dBi) ≤ 29.15 dBm.
    At 21 dBm TX + 2.5 dBi antenna you are legal with 12 dB headroom.
[ ] DO NOT run a T-Beam 1W at 32 dBm. Cap it to ≤ 22 dBm.
[ ] Implement Adaptive Power Control. It is mandatory at 500 mW e.r.p.
[ ] Start at SF9/BW125, unconfirmed uplinks, 1 report/60 s. Duty ≈ 0.25-0.5 %.
    If you need SF12, drop to 1 report per 2-5 min, or you breach the 2.5 % cap
    once the acknowledged downlink is counted.
[ ] Site survey: log RSSI + SNR at 4 worst points. Pass = RSSI ≥ -115 dBm,
    PDR ≥ 99 % (unconfirmed).
[ ] Read your stack's actual airtime report. Do not trust this document's
    airtime table - the exact figure depends on header mode, coding rate and DE.
[ ] Get the gateway antenna out of any metal enclosure (lift shaft, cupboard).
[ ] Mount the robot antenna clear of the chassis, motors and battery pack.
[ ] Ask the DoT WPC Helpdesk (https://eservices.dot.gov.in/) in writing whether
    the school needs a Radio Equipment Possession Authorisation under the
    2026 Rules. One email. Do this before ordering hardware.
[ ] Re-check every channel setting against G.S.R. 853(E) before the demo.
```