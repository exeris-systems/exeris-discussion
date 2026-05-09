# BUS-002: R&D Infrastructure Model (Virtual Lab)

| Attribute      | Value                                                                                                                                                            |
|:---------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Status**     | **ACCEPTED**                                                                                                                                                       |
| **Deciders**   | Arkadiusz Przychocki                                                                                                                                               |
| **Date**       | 2025-12-11                                                                                                                                                         |
| **Side**       | business (financial / procurement / capex)                                                                                                                         |
| **Visibility** | internal                                                                                                                                                           |
| **Owning Repo**| `exeris-docs`                                                                                                                                                      |
| **Driven By**  | [RFC-2025-12-11: Infrastructure Model](https://exeris.atlassian.net/wiki/spaces/ENG/pages/13369408/2025-12-11+-+RFC+-+Infrastructure+Model+Virtual+Biotech)        |
| **Compliance** | [Strategic Pillar: Risk-Managed Structure](https://exeris.atlassian.net/wiki/spaces/HUB/pages/6094854/Strategic+Pillars+Architecture+Principles#Strategic-Pillars) |

## Context and Problem Statement
Validating "Hyper-Density" (1M RPS) requires enterprise-grade hardware (NVMe, 100GbE). Building an on-premise Data Center requires massive CAPEX (>€500k), creating unnecessary financial risk. Public Cloud (AWS EC2) suffers from the "Noisy Neighbor" effect, making scientific benchmarking unreliable.

## 🏁 The Decision
We adopt a **"Virtual Lab" Model** utilizing rented High-Performance Computing (HPC).

**Strategy:**
* **Resource Source:** Rent HPC power from Cyfronet Helios (or similar) only for the duration of experiments (Batch Jobs).
* **Financial Model:** Convert CAPEX to OPEX. Pay only for compute time used.
* **Environment:** Bare metal access via HPC scheduling to ensure consistent benchmark results.

## Positive Outcomes
* **Valid Benchmarks:** Access to scientific-grade hardware eliminates public cloud variance.
* **Cost Efficiency:** We can simulate massive scale (1M users) for a fraction of the cost of owning hardware.
* **TRL Validation:** Enables credible proof of performance claims for FENG/Investors.

## Trade-offs / Risks
* **Availability:** HPC queues (Slurm) might delay experiments during high-demand periods.
* **Complexity:** Requires adapting deployment scripts to HPC batch schedulers (Slurm) instead of standard K8s.

## Engineering Protocol
Once this decision is ACCEPTED, it must be committed to the repository to maintain the Single Source of Truth.
