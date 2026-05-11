# BUS-001: R&D Cooperation Model (IP Sovereignty)

| Attribute      | Value                                                                                                                                                           |
|:---------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Status**     | **ACCEPTED**                                                                                                                                                      |
| **Deciders**   | Arkadiusz Przychocki                                                                                                                                              |
| **Date**       | 2025-12-10                                                                                                                                                        |
| **Side**       | business (legal / IP / contracting)                                                                                                                               |
| **Visibility** | internal (do not externalize without legal review)                                                                                                                |
| **Owning Repo**| `exeris-docs`                                                                                                                                                     |
| **Compliance** | [Strategic Pillar: Clean IP & Detachment](../../exeris-kernel/docs/architecture.md)                                                                               |

## Context and Problem Statement
Deep Tech R&D (e.g., Scheduler optimization, HTTP/3 tuning) requires scientific expertise (HPC/AI). However, typical grant-based "Consortia" with Universities lead to Shared IP Ownership. If a University owns even 10% of the Kernel, we legally **cannot** sell "Code Detachment" to clients, effectively killing our business model.

## 🏁 The Decision
We adopt the **"Subcontracting Model" (B2B)**. We explicitly reject Consortium models for Exeris Kernel development.

**Implementation:**
* We hire Research Units (e.g., Cyfronet) as B2B vendors for specific, well-defined tasks (benchmarking, algorithm validation).
* Contracts must explicitly assign **100% of Intellectual Property Rights (IPR)** to Exeris upon payment.
* Researchers work in sandboxed environments on anonymized datasets, not on the full Kernel repository.

## Positive Outcomes
* **Clean IP:** Exeris retains full legal ability to license and "detach" the code.
* **Agility:** Vendor relationship allows for faster pivoting than rigid grant consortia agreements.

## Trade-offs / Risks
* **Cost:** We pay market/B2B rates for research rather than leveraging subsidized academic grants.
* **Recruitment:** Top academics might prefer co-authorship/ownership over B2B work (mitigation: allow publication of non-commercial theoretical findings).

## Engineering Protocol
Once this decision is ACCEPTED, it must be committed to the repository to maintain the Single Source of Truth.
