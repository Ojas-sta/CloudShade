# Platform Selection Study (prior phase)

This directory contains the **platform selection study** that preceded the SHANA
feasibility work. It compared the **Unitree G1 humanoid** against the **Unitree
Go2 quadruped** and recommended the Go2 EDU.

## Why it is retained

Several findings are load-bearing for the current phase and are cited throughout
`docs/`:

| Finding | Where it is used |
|---|---|
| Go2 EDU scores **63.5 / 100** against G1 EDU **40.5** on feasibility, autonomy, reliability and safety | Justifies the platform choice |
| G1's only advantage — manipulation — is measured at **20–30% pick-and-place success** and **13.5 N** unbraced force | Explains why SHANA carries rather than picks up |
| G1 requires a **mandatory protective frame and rope** | Would have made a school demonstration impractical |
| The Go2's **built-in obstacle avoidance is on by default**, but `FreeAvoid` may produce an **uncommanded lateral dodge when stationary** | Safety analysis |
| Official Unitree licensing, community stack maturity, and repositories to avoid | Ongoing reference |

## Files

| File | Contents |
|---|---|
| `executive_summary.md` | The recommendation and its evidence |
| `robot_comparison.md` | Full engineering comparison |
| `hardware_configuration_matrix.md` | Variant-by-variant specifications and ambiguities |
| `sdk_and_software_research.md` → moved to `../hardware/sdk_and_interfaces.md` | SDK, ROS 2, and interface inventory |
| `safety_and_failure_analysis.md` → moved to `../safety/hazard_analysis.md` | Failure scenarios and mitigations |
| `implementation_roadmap.md` → moved to `../testing/future_pilot_plan.md` | Phased implementation plan |
| `proposed_architecture.md` | Initial architecture proposal |
| `exhibition_project_ideas.md` | Nine ranked project concepts |
| `weighted_decision_matrix.md` | Scoring with sensitivity analysis |
| `source_register.md` | Source bibliography |

⚠️ **Some documents predate the current privacy and permissions constraints.**
Where they conflict with the current SHANA documents, **the current documents
take precedence.**
