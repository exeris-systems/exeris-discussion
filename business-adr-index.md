# Business ADR Index — Exeris

Business-side decisions (legal / IP / financial / procurement / partner agreements / regional compliance). Distinct namespace `BUS-NNN` — never collides with the tech ADR namespace.

## Rules

1. **Number reservation.** A number is reserved by a PR to this file before the BUS-content PR merges.
2. **Visibility.** Default `internal` (do not externalize without legal review). `public` only for intentionally externalized decisions.
3. **Cross-namespace boundary.** Tech ADRs (`ADR-NNN`) and business ADRs (`BUS-NNN`) are distinct. Mixed decisions are filed in the namespace of the *deciding* concern, with a cross-reference to the other.

## Index

| #       | Title                                          | Owning repo  | Side               | Visibility | Status     | Date       | Link                                                                                                          |
|---------|------------------------------------------------|--------------|--------------------|------------|------------|------------|---------------------------------------------------------------------------------------------------------------|
| BUS-001 | R&D Cooperation Model (IP Sovereignty)         | exeris-docs  | legal / IP         | internal   | accepted   | 2025-12-10 | [business-adr/BUS-001 …](business-adr/BUS-001-rd-cooperation-model.md)                                        |
| BUS-002 | R&D Infrastructure Model (Virtual Lab)         | exeris-docs  | financial / capex  | internal   | accepted   | 2025-12-11 | [business-adr/BUS-002 …](business-adr/BUS-002-rd-infrastructure-model.md)                                     |

## Cross-references with the tech registry

| Business decision  | Tech implication / cross-ref                                                                                |
|--------------------|-------------------------------------------------------------------------------------------------------------|
| BUS-001 (Clean IP) | Underpins ADR-008 (Open-Core Strategy) — the IP model defined here is what makes "Code Detachment" sellable |
