# ADR-002: Unified Database Strategy (PostgreSQL 18)

> **Scope:** platform-recommended stack for new applications, not a kernel mandate. The Exeris kernel itself remains database-agnostic at the runtime layer (`PersistenceProvider` SPI); apps can override with their own persistence provider.

| Attribute      | Value                                                                                                                                                             |
|:---------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Status**     | **ACCEPTED**                                                                                                                                                        |
| **Deciders**   | Arkadiusz Przychocki                                                                                                                                                |
| **Date**       | 2025-10-15                                                                                                                                                          |
| **Scope**      | platform (stack-level recommendation across all Exeris apps)                                                                                                        |
| **Owning Repo**| `exeris-docs`                                                                                                                                                       |
| **Driven By**  | [RFC-2025-10-15: Unified Database Strategy](https://exeris.atlassian.net/wiki/spaces/ENG/pages/14254124/2025-10-15+-+RFC+-+Unified+Database+Strategy+PostgreSQL+18) |
| **Compliance** | [Strategic Pillar: No-Waste Compute](https://exeris.atlassian.net/wiki/spaces/HUB/pages/6094854/Strategic+Pillars+Architecture+Principles#Strategic-Pillars)        |

## Context and Problem Statement
The "Polyglot Persistence" pattern (e.g., using Neo4j for Graph, Mongo for Docs, SQL for core) introduces massive operational complexity ("Integration Tax"), high licensing costs, and distributed transaction consistency issues.

To support **Hyper-Density** and keep the "Code Detachment" stack manageable for clients, we need to consolidate data models while maintaining high write throughput (>50k writes/sec).

## 🏁 The Decision
We standardize on **PostgreSQL 18** as the single engine for all primary data needs.

**Implementation Details:**
* **Relational:** Standard SQL for core business entities.
* **Documents:** `JSONB` column type for unstructured data (replacing Mongo).
* **Graph:** **SQL/PGQ** standard (read) + Recursive CTEs (write) for graph traversals (replacing Neo4j).
* **Identity:** UUIDv7 for primary keys to ensure index locality and performance.
* **Security:** Mandatory usage of **Row Level Security (RLS)** for multi-tenancy enforcement.

## Positive Outcomes
* **Simplified Stack:** One database to manage, back up, and scale.
* **ACID Guarantees:** Strong consistency across relational, document, and graph data in a single transaction.
* **Cost Reduction:** Elimination of expensive specialized database licenses (e.g., Neo4j Enterprise).

## Trade-offs / Risks
* **No Specialized Time-Series:** For extremely high-frequency telemetry, Postgres might hit limits (mitigation: TimescaleDB extension if needed later).
* **Vertical Scaling Limits:** Scaling a single massive Postgres cluster is harder than sharding NoSQL (mitigation: Read Replicas and future sharding strategies).

## Engineering Protocol
Once this decision is ACCEPTED, it must be committed to the repository to maintain the Single Source of Truth.

## Kernel relationship

The kernel (`exeris-kernel`, `exeris-kernel-enterprise`) does **not** hard-depend on PostgreSQL. The persistence boundary is the `PersistenceProvider` SPI. ADR-002 is a platform-tier recommendation for the *default* stack used by new applications and reference implementations. Applications that need a different persistence backend can ship their own provider via `ServiceLoader` priority — that path is supported by the SPI and is not a deviation from this ADR.
