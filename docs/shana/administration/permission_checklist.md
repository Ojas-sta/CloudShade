# SHANA — Permission Checklist

**Status: NOTHING HAS BEEN GRANTED. Every item below is `NOT REQUESTED`.**

This checklist exists to make permission-seeking explicit and auditable. A
permission is only recorded as granted when a **named person has given written
authorisation**, and the evidence is attached.

---

## How to use this checklist

1. Each row has a unique ID matching the questionnaire where applicable.
2. Move a row to `GRANTED` **only** when written evidence exists.
3. Record who granted it and when. Verbal agreement is **not** sufficient.
4. A `GRANTED` permission that is later withdrawn reverts to `WITHDRAWN`, not to
   the previous state.
5. ⚠️ **No item may be marked granted on the basis of a conversation.** If it is
   not in writing, it is not granted.

---

## Status values

`NOT REQUESTED` · `REQUESTED` · `GRANTED` · `DENIED` · `CONDITIONAL` ·
`WITHDRAWN` · `NOT APPLICABLE`

---

## 1. Principal / Authorised School Administration

| ID | Permission requested | Evidence required | Status | Granted by | Date |
|---|---|---|---|---|---|
| ADM-A1 | Authorise the SHANA project and name a project owner | Written authorisation naming the owner | NOT REQUESTED | | |
| ADM-A2 | Authorise use of the school-owned robot for this project | Written authorisation | NOT REQUESTED | | |
| ADM-A3 | Approve the server host location (proposed: AI Lab, CP-C3) | Room allocation record | NOT REQUESTED | | |
| ADM-A4 | Approve installation of access points, if required | Written approval | NOT REQUESTED | | |
| ADM-A5 | Approve wall-mounted ArUco markers or other route markers | Written approval | NOT REQUESTED | | |
| ADM-A6 | Approve cabling for the robot network segment | Written approval | NOT REQUESTED | | |
| ADM-A7 | Issue a robot operating policy for occupied corridors | Written policy | NOT REQUESTED | | |
| ADM-A8 | Approve controlled testing, out of student hours | Written approval | NOT REQUESTED | | |
| ADM-A9 | Approve exhibition demonstration scope | Written approval | NOT REQUESTED | | |
| ADM-A10 | **Separately authorise any student-facing demonstration** | Written approval, distinct from A9 | NOT REQUESTED | | |
| ADM-A11 | Confirm acceptance gates and who may declare them passed | Written confirmation | NOT REQUESTED | | |
| ADM-A12 | Confirm the explicit stop criteria for the project | Written criteria | NOT REQUESTED | | |
| ADM-A13 | Confirm project ownership of robot, server and all data | Ownership record | NOT REQUESTED | | |
| ADM-A14 | Confirm the decision-making model for architecture gates | Written decision | NOT REQUESTED | | |

## 2. Robot Owner / Custodian

| ID | Permission requested | Evidence required | Status | Granted by | Date |
|---|---|---|---|---|---|
| ROB-P1 | **Authorise use of the Unitree secondary development interface** | Written authorisation; purchase or support record | NOT REQUESTED | | |
| ROB-P2 | Provide model, revision, and firmware details | Photos and screenshots | NOT REQUESTED | | |
| ROB-P3 | Provide installed options (dock, depth camera, foot sensors) | Photos | NOT REQUESTED | | |
| ROB-P4 | Provide battery identification | Photo of battery label | NOT REQUESTED | | |
| ROB-P5 | Provide warranty and service arrangements | Contract reference | NOT REQUESTED | | |
| ROB-P6 | Provide repair and fault history | Service records | NOT REQUESTED | | |
| ROB-P7 | Authorise a dedicated Ubuntu development host | Written confirmation | NOT REQUESTED | | |
| ROB-P8 | Nominate the trained custodian for physical handling | Named person | NOT REQUESTED | | |

## 3. IT / Network Administrator

| ID | Permission requested | Evidence required | Status | Granted by | Date |
|---|---|---|---|---|---|
| IT-P1 | Approve a dedicated robot network segment, `192.168.123.0/24` | Written approval and VLAN/port plan | NOT REQUESTED | | |
| IT-P2 | Assign a static IP for the server on the robot subnet | Assignment record | NOT REQUESTED | | |
| IT-P3 | Confirm UDP multicast is permitted on that subnet | Configuration confirmation | NOT REQUESTED | | |
| IT-P4 | Confirm the server is reachable by teacher devices on the LAN | Firewall rule review | NOT REQUESTED | | |
| IT-P5 | **Confirm the server is not exposed to any public network** | Written confirmation | NOT REQUESTED | | |
| IT-P6 | Provide the existing wired LAN topology | Topology diagram | NOT REQUESTED | | |
| IT-P7 | Confirm VLAN and client-isolation settings | Configuration excerpt | NOT REQUESTED | | |
| IT-P8 | Provide Wi-Fi AP inventory and floor coverage | Inventory and floor plan | NOT REQUESTED | | |
| IT-P9 | Approve additional access points, if required | Written approval | NOT REQUESTED | | |
| IT-P10 | Approve a private SHANA wireless network, if required | Written approval | NOT REQUESTED | | |
| IT-P11 | Provide the school network acceptable-use and RF policies | Policy documents | NOT REQUESTED | | |
| IT-P12 | Confirm credential storage and rotation policy | Written policy | NOT REQUESTED | | |
| IT-P13 | Provide scheduled outage and load-shedding windows | Outage schedule | NOT REQUESTED | | |
| IT-P14 | Approve a temporary isolated exhibition network, if required | Written approval | NOT REQUESTED | | |

## 4. CCTV System Owner and Privacy / Policy Authority

| ID | Permission requested | Evidence required | Status | Granted by | Date |
|---|---|---|---|---|---|
| CCTV-P1 | Provide CCTV brand, model and management system details | System documentation | NOT REQUESTED | | |
| CCTV-P2 | Provide ONVIF or RTSP interface documentation, if any | Vendor documentation | NOT REQUESTED | | |
| CCTV-P3 | **Authorise any analytics access to CCTV streams** | Written authorisation | NOT REQUESTED | | |
| CCTV-P4 | Confirm camera coverage of candidate routes | Coverage map | NOT REQUESTED | | |
| CCTV-P5 | Confirm whether cameras carry audio and how it is handled | Specification and confirmation | NOT REQUESTED | | |
| CCTV-P6 | Confirm whether any people-counting analytics already exists | Product documentation | NOT REQUESTED | | |
| PRIV-P1 | **Confirm the Data Fiduciary for student data** | Named role in writing | NOT REQUESTED | | |
| PRIV-P2 | **Authorise the project in principle as Data Fiduciary** | Written approval | NOT REQUESTED | | |
| PRIV-P3 | Provide existing privacy, retention and camera policies | Policy documents | NOT REQUESTED | | |
| PRIV-P4 | Determine whether a privacy impact assessment is required | Written determination | NOT REQUESTED | | |
| PRIV-P5 | **Provide a legal determination on whether any exemption covers corridor-routing analytics** | Written opinion from a qualified adviser | NOT REQUESTED | | |
| PRIV-P6 | Confirm derived occupancy data retention policy | Written policy | NOT REQUESTED | | |
| PRIV-P7 | Approve the robot's own camera policy | Written policy | NOT REQUESTED | | |
| PRIV-P8 | Provide the breach-notification process | Written procedure | NOT REQUESTED | | |

## 5. Facilities and Housekeeping

| ID | Permission requested | Evidence required | Status | Granted by | Date |
|---|---|---|---|---|---|
| FAC-P1 | **Measure and provide all stair riser heights, tread depths and widths** | Measurements and photographs | NOT REQUESTED | | |
| FAC-P2 | Provide the B↔A block route survey (currently unrecorded) | Annotated floor plan | NOT REQUESTED | | |
| FAC-P3 | Provide corridor width and floor surface survey | Survey table and photographs | NOT REQUESTED | | |
| FAC-P4 | Confirm which corridors are authorised for robot access | Authorised segment list | NOT REQUESTED | | |
| FAC-P5 | Identify fire and evacuation routes to exclude | Fire plan | NOT REQUESTED | | |
| FAC-P6 | Nominate who may declare a corridor closed or hazardous | Named authority list | NOT REQUESTED | | |
| FAC-P7 | Provide a cleaning and wet-floor schedule in a usable format | Schedule file | NOT REQUESTED | | |
| FAC-P8 | Provide permanent and recurring corridor obstructions | Survey | NOT REQUESTED | | |
| FAC-P9 | Confirm checkpoint locations, security and power | Confirmation and photographs | NOT REQUESTED | | |
| FAC-P10 | Provide the battery handling and charging procedure | Written procedure | NOT REQUESTED | | |
| FAC-P11 | Provide the damaged-battery quarantine procedure | Written procedure | NOT REQUESTED | | |
| FAC-P12 | Provide the emergency stop procedure | Written procedure | NOT REQUESTED | | |
| FAC-P13 | Identify a physically secured exclusion and test area | Site plan | NOT REQUESTED | | |
| FAC-P14 | Provide the first-aid contact and injury procedure | Written procedure | NOT REQUESTED | | |

## 6. Lift Manufacturer / Authorised Service Provider

> ⚠️ **Route all questions through the school's authorised service provider.**
> Do not approach the manufacturer directly unless the provider directs you to.
> **Do not request, propose, or build any physical modification to a lift.**

| ID | Permission requested / information sought | Evidence required | Status | Granted by | Date |
|---|---|---|---|---|---|
| LIFT-P1 | Confirm lift manufacturer, model and standard | Nameplate and type approval | NOT REQUESTED | | |
| LIFT-P2 | Provide car dimensions, door clear opening and sill gap | Drawings and measurements | NOT REQUESTED | | |
| LIFT-P3 | Provide rated load and installation date | Specification | NOT REQUESTED | | |
| LIFT-P4 | Confirm whether the maintenance contract permits third-party equipment | Written confirmation | NOT REQUESTED | | |
| LIFT-P5 | **Ask whether any sanctioned third-party or robot call interface exists** | Written response | NOT REQUESTED | | |
| LIFT-P6 | **Ask whether EyeRIS is retrofittable, and whether its QR is passive or active** | Written response | NOT REQUESTED | | |
| LIFT-P7 | **Ask whether EyeRIS can be provisioned to a non-human client** | Written response | NOT REQUESTED | | |
| LIFT-P8 | **Ask whether any physical external button actuator would be permitted** | Written statement — **do not assume permission** | NOT REQUESTED | | |
| LIFT-P9 | Ask the written position on attaching devices to the COP or LOP | Written policy | NOT REQUESTED | | |
| LIFT-P10 | Confirm fire recall / Phase I service presence | Specification | NOT REQUESTED | | |
| LIFT-P11 | Confirm whether an escorted robot trial is permissible | Written consent | NOT REQUESTED | | |
| LIFT-P12 | Confirm whether integration affects insurance, warranty or certification | Written confirmation | NOT REQUESTED | | |

## 7. Makerspace and Laboratory Supervisors

| ID | Permission requested | Evidence required | Status | Granted by | Date |
|---|---|---|---|---|---|
| LAB-P1 | Approve the AI Lab as the proposed primary server and base | Written approval | NOT REQUESTED | | |
| LAB-P2 | Provide host machine specification and available ports | Specification output | NOT REQUESTED | | |
| LAB-P3 | Confirm reliable mains power and UPS availability | Confirmation | NOT REQUESTED | | |
| LAB-P4 | Provide secure robot storage arrangements | Confirmation | NOT REQUESTED | | |
| LAB-P5 | Nominate trained manual operators and backups | Named persons | NOT REQUESTED | | |
| LAB-P6 | Provide permitted testing windows | Schedule | NOT REQUESTED | | |
| LAB-P7 | **Approve the permitted and prohibited delivery item list** | Written item policy | NOT REQUESTED | | |
| LAB-P8 | Approve approved container types and weight limits | Written specification | NOT REQUESTED | | |

## 8. Exhibition Organisers

| ID | Permission requested | Evidence required | Status | Granted by | Date |
|---|---|---|---|---|---|
| EXPO-P1 | Approve the demonstration scope and what is excluded | Written scope | NOT REQUESTED | | |
| EXPO-P2 | Approve the venue and confirm it is verified safe | Site survey | NOT REQUESTED | | |
| EXPO-P3 | Provide power and network at the venue | Site services | NOT REQUESTED | | |
| EXPO-P4 | Approve crowd control arrangements | Written plan | NOT REQUESTED | | |
| EXPO-P5 | Nominate the event supervisor | Named person | NOT REQUESTED | | |
| EXPO-P6 | Approve abort criteria | Written criteria | NOT REQUESTED | | |
| EXPO-P7 | Approve a fallback demonstration plan | Written plan | NOT REQUESTED | | |
| EXPO-P8 | Approve the incident response plan | Written plan | NOT REQUESTED | | |
| EXPO-P9 | Confirm photography and publication policy | Policy | NOT REQUESTED | | |

## 9. Safety, Security and Insurance

| ID | Permission requested | Evidence required | Status | Granted by | Date |
|---|---|---|---|---|---|
| SAFE-P1 | Confirm insurance covers robot operation and transport | Written confirmation from insurer | NOT REQUESTED | | |
| SAFE-P2 | Confirm the spotter role and training requirement | Written confirmation | NOT REQUESTED | | |
| SAFE-P3 | Confirm spotter unilateral stop authority | Written confirmation | NOT REQUESTED | | |
| SAFE-P4 | Approve the approved maximum operating speed | Written speed policy | NOT REQUESTED | | |
| SAFE-P5 | Approve the exclusion zone definition | Written procedure | NOT REQUESTED | | |
| SAFE-P6 | Approve the fire or evacuation procedure for the robot | Written procedure | NOT REQUESTED | | |
| SAFE-P7 | Approve the incident and near-miss reporting process | Written procedure | NOT REQUESTED | | |
| SEC-P1 | Approve repository hosting location and access | Written approval | NOT REQUESTED | | |
| SEC-P2 | Confirm the rule prohibiting credentials in Git | Written rule | NOT REQUESTED | | |

---

## Absolute prohibitions — no permission will be sought

These are recorded so that under time pressure there is a pre-decided answer.

| # | Action | Status |
|---|---|---|
| 1 | ⛔ Any modification to a lift control system, panel, wiring, or safety circuit | **NEVER PERMITTED** |
| 2 | ⛔ Attaching a servo, actuator, or any device to a lift COP or LOP | **NEVER PERMITTED** |
| 3 | ⛔ Bypassing, blocking, or disabling lift interlocks or door sensors | **NEVER PERMITTED** |
| 4 | ⛔ Entering a lift shaft, pit, or car top | **NEVER PERMITTED** |
| 5 | ⛔ Facial recognition or any biometric identification of students | **NEVER PERMITTED** |
| 6 | ⛔ Automatic misconduct, discipline, attendance, or behaviour reporting | **NEVER PERMITTED** |
| 7 | ⛔ Unsupervised robot operation | **NEVER PERMITTED** |
| 8 | ⛔ Public network exposure of any SHANA component | **NEVER PERMITTED** |
| 9 | ⛔ Teacher access to motor control | **NEVER PERMITTED** |
| 10 | ⛔ Credentials, keys, or private network details committed to Git | **NEVER PERMITTED** |

---

## Summary

| Role | Items requested | Granted |
|---|---|---|
| School administration | 14 | 0 |
| Robot owner / custodian | 8 | 0 |
| IT / network | 14 | 0 |
| CCTV owner and privacy authority | 14 | 0 |
| Facilities and housekeeping | 14 | 0 |
| Lift service provider | 12 | 0 |
| Lab / Makerspace supervisors | 8 | 0 |
| Exhibition organisers | 9 | 0 |
| Safety, security, insurance | 9 | 0 |
| **Total** | **102** | **0** |

⚠️ **The zero in the final column is the accurate current state, not an
oversight.**