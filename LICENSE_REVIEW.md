# Licence Review — Open Decision

**Status: NOT YET DECIDED. No licence has been chosen.**

This file records the licence questions that must be answered before any code or
documentation is published outside the school.

---

## Why this is open

The project has a specific and unusual constraint profile:

1. It sits in a school, involving a school-owned robot, school networks, school
   CCTV, and student safety obligations.
2. It may draw on third-party robotics libraries with **materially different
   licence terms**.
3. It has not yet written code, so there is nothing to licence today — but that
   will change.

---

## Questions to resolve

### Q1 — Ownership

Who owns the intellectual property produced by this project?

- The school, as employer of any student or staff contributors?
- Individual contributors?
- A joint arrangement?

⚠️ **This must be settled before any external release.** It is not a decision
the project team can make alone.

### Q2 — Repository hosting

Where will the repository be hosted, and does school policy permit external
hosting at all? See `SH-GIT-002`, `SH-GIT-003`.

### Q3 — Documentation licence

Should documentation be openly licensed (for example CC BY-SA), kept internal,
or licensed to the school?

### Q4 — Code licence, if code is ever written

Candidate licences, and what each implies:

| Licence | Implication |
|---|---|
| **Proprietary / all rights reserved** | Simple. Nobody may reuse without permission. Common for school projects. |
| **MIT / BSD / Apache-2.0** | Permissive. Allows commercial reuse. Requires attribution. |
| **GPL-3.0** | Strong copyleft. Any derivative must also be GPL-3.0. |
| **AGPL-3.0** | Extends copyleft to network use. |

⚠️ **Recommendation: do not choose a copyleft licence without deliberate
consideration.** GPL would require that any derived work — including a school's
own modifications — be released under GPL.

---

## Third-party dependency licence obligations

This project's likely dependencies include robotics SDKs and frameworks with
**very different terms.** These were identified during feasibility research and
must be re-checked before any code is written.

| Component | Licence | Obligation |
|---|---|---|
| `unitree_sdk2` | **BSD-3-Clause** | Permissive. Attribution + no-endorsement clause. ✅ Safe. |
| `unitree_sdk2_python` | **BSD-3-Clause** | Permissive. ✅ Safe. |
| `unitree_ros2` | **BSD-3-Clause** | Permissive. ✅ Safe. |
| `unitree_mujoco` | **BSD-3-Clause** | Permissive. ✅ Safe. |
| `unitree_rl_lab`, `unitree_rl_mjlab` | Apache-2.0 | Permissive with patent grant. ✅ Safe. |
| `unilidar_sdk`, `unilidar_sdk2` | BSD-3-Clause | Permissive. ✅ Safe. |
| **`unitree_slam`** | **NO LICENCE FILE** | ⛔ **No legal grant of any right.** May not be copied, modified, or redistributed. **Do not use.** |
| **`point_lio_unilidar`** | **GPL-2.0** | ⛔ **Strong copyleft.** Linking forces release of the entire combined work. Also ROS 1 Noetic. **Avoid.** |
| **`unitree_ros_to_real`** | **NO LICENCE FILE** | ⛔ **No legal grant.** **Do not use.** |
| **`unitree_pybullet`** | **NO LICENCE FILE** | ⛔ **No legal grant.** Also dead since 2020. **Do not use.** |
| `unitree_lerobot`, `avp_teleoperate` | `NOASSERTION` | ⚠️ Undetermined. **Read the licence text before use.** |
| `brainco_hand_service`, `linker_hand_service` | **NO LICENCE FILE** | ⛔ **No legal grant.** **Do not use.** |

### The closed-binary problem

⚠️ `unitree_sdk2` is BSD-3-Clause, but its **implementation is a closed ~28 MB
prebuilt static library**. The licence permits redistribution, but the code
**cannot be audited or patched.** For a project touching student safety, this
matters and should be recorded as an accepted risk rather than overlooked.

### Simulation and Isaac

- `unitree_mujoco` (BSD-3) and `unitree_rl_mjlab` (Apache-2.0) avoid licensing
  complications entirely.
- ⚠️ **Isaac Sim / Isaac Lab is distributed under the NVIDIA Omniverse License
  Agreement, which is not OSI-approved.** It permits research and education use
  but carries redistribution and commercial-use restrictions. **This was not
  verified** and must be read directly before any use.

---

## Compliance checklist

Before any external release:

- [ ] Q1 ownership settled
- [ ] Q2 hosting permitted
- [ ] Q3 documentation licence chosen
- [ ] Q4 code licence chosen, if applicable
- [ ] Every dependency's licence re-checked from source, not from a package index
- [ ] No GPL or unlicensed component has entered the dependency tree
- [ ] Isaac/NVIDIA licence reviewed if any Isaac component is used
- [ ] The closed-binary SDK has been recorded as an accepted risk
- [ ] No school-confidential information is included
- [ ] No student personal data is included

---

## Current position

**No licence has been chosen and no release has been made.** This file exists to
prevent an implicit default — the absence of a licence is itself a decision, and
an unhelpful one.

Question IDs: `SH-GIT-017`, `SH-GIT-018`.