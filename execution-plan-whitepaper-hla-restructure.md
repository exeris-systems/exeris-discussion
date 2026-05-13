# Execution Plan: Whitepaper and HLA Restructure

This plan specifies the section-by-section changes required to make `b2b-technical-whitepaper.md` and `high-level-architecture.md` reflect the full Exeris Systems vision. The plan assumes the three-tier architecture established in the previous synthesis (Substrate → Capability Ecosystem → Vertical SaaS SKUs) plus the BudgetHQ clarification: BudgetHQ is an independent SaaS product within the Exeris Systems family that uses Exeris Spring Runtime as a downstream customer, serves as the dogfooding vehicle for new platform capabilities, and is not itself a platform SKU.

The plan is organized as: (1) front-matter and naming conventions to settle first, (2) whitepaper restructure with section-level specifications, (3) HLA restructure with section-level specifications, (4) ADR work, (5) dependency order and acceptance criteria. Estimated total effort is twelve to sixteen focused working days for the documentation pass alone, exclusive of new ADR drafting.

---

## 1. Front Matter and Naming Conventions to Settle First

Three naming conventions must be fixed before either document is touched, because both documents reference them throughout.

**Family hierarchy.** "Exeris Systems" is the umbrella for the founder's portfolio of products. The portfolio contains two categories: **Platform products** (which constitute the open-core platform itself) and **Family products** (which are independent SaaS products built on the platform, of which BudgetHQ is the first). Platform products are listed under "Exeris Platform Components" in the whitepaper §3 table. Family products are listed under a new "Exeris Systems Family Products" section. The distinction is structural: a Platform product is part of what customers buy or fork; a Family product is what Exeris Systems itself ships using the platform.

**SKU vocabulary.** The vertical SaaS products you described (API Gateway, Edge Proxy, Bot Blocker, IDP, PIM, OMS, Headless CMS, Context-Centric CRM) are first-party **Platform SKUs** — they live on the platform, are defined as cap compositions, and are sold or licensed by Exeris Systems. They are not Family products. BudgetHQ, by contrast, is a Family product. This distinction matters in the documents because the commercial story for a Platform SKU (subscribed via Platform tier with cap composition included) is different from the commercial story for a Family product (independent SaaS company, independent brand, independent pricing).

**Dogfooding language.** BudgetHQ's relationship to the platform should be described as "dogfooding the Exeris Spring Runtime in production" rather than "proof-of-concept" or "demo application." The Corelio biznesplan used the language of demo and proof-of-concept; that framing weakens the platform claim because it implies BudgetHQ exists to validate the platform rather than to use it. The dogfooding framing inverts that: BudgetHQ is a real product with real customers; it happens to be built on Exeris Spring Runtime; its production load proves the runtime works under conditions other reference customers cannot disclose.

---

## 2. Whitepaper Restructure

The current whitepaper is approximately 4,500 words. The restructured version targets **6,200–6,800 words**. The section-level plan follows.

### Section 1 — Software Inflation Problem (minor edit, ~50 words added)

Keep the existing problem framing. Add a single closing paragraph that previews the three-tier architecture: the platform addresses the substrate inflation problem; the capability ecosystem makes that substrate composable; the SKU layer ships pre-composed solutions for specific verticals. This preview is the spine the rest of the document will refer back to.

### Section 2 — Runtime-Owned Execution (no structural change)

The current section is correct. The only edit is to ensure the closing paragraph references "Tier 1 substrate" terminology so the three-tier vocabulary is established before §3.

### Section 3 — Strategic Platform Components (major restructure, ~1,800 words total)

Replace the current single table with three subsections, each anchored by its own table.

**§3.1 Tier 1 — Substrate Repositories.** This is the current §3 table renamed and reframed. Rows: `exeris-kernel`, `exeris-kernel-enterprise`, `exeris-spring-runtime`, `exeris-sdk`, `exeris-tooling`, `exeris-telemetry-spec`, `exeris-enterprise-observability`, `exeris-benchmarks`, `exeris-platform` (Studio shell). Each row keeps its current ADR anchors but fixes the dangling references (ADR-002, ADR-003, ADR-016, ADR-019, ADR-020, ADR-022) per the previous audit — either supply the ADR or replace the citation with a prose pointer to the subsystem doc.

**§3.2 Tier 2 — Capability Ecosystem.** New subsection, approximately 600 words. Open with a paragraph defining the capability composition model in plain terms: capabilities are named modules that expose `@Provides` services and declare `@Requires` dependencies; the kernel validates the wiring graph at build time; composition is the unit of product definition. Follow with a table of the planned capability repositories grouped into three types: substrate capabilities (`exeris-caps-gateway-core`, `exeris-caps-service-boundary-core`), policy capabilities reusable across multiple SKUs (`exeris-caps-rate-limiting`, `exeris-caps-jwt-validation`, `exeris-caps-tls-termination`, `exeris-caps-request-routing`, `exeris-caps-observability-bridge`), and SKU-specific capabilities introduced as the relevant SKUs are scoped. Each row should list the repository name, its open-core tier (public versus enterprise-private), its current status (specified, scaffolded, implemented), and the SKUs it participates in. Close with a paragraph on the Wall enforcement at the cap layer: capabilities cannot bypass the Wall to reach Spring or other host runtimes, which is what makes a cap composition portable across host runtimes.

**§3.3 Tier 3 — Vertical SaaS SKUs (Platform SKUs).** New subsection, approximately 700 words. Open with the SKU inventory table from the previous synthesis, scoped to Platform SKUs only (no BudgetHQ in this table). Three subsection paragraphs follow, one per SKU family: Gateway family (API Gateway, Edge Proxy, Bot Blocker), Service Boundary family (IDP, PIM, OMS, Headless CMS), Context-Centric data model (the cross-cutting B2B data primitive that any Service Boundary SKU can compose). Each family paragraph specifies its target customer, its commercial positioning, and its Enterprise-tier engine swap path (Gateway family swaps to `exeris-caps-quic-h3` plus `io_uring` transport at Enterprise tier; Service Boundary family swaps to NUMA-aware compute paths at Enterprise tier).

### Section 4 — Empirical Evidence (additive, ~400 words added)

Keep the current Distributed Saga and TLS benchmark tables. After fixing the TLS engine tier label from the previous audit, add a new subsection **§4.3 "SKU-Level Benchmark Forecast"** that lists the benchmarks Exeris will publish per SKU family. The forecast table should have rows: API Gateway RPS at matched-contract parity versus Envoy and Kong (target: H1 2027); Edge Proxy P99 latency at multi-region failover (target: H2 2027); Bot Blocker JA3/JA4 fingerprinting throughput cost (target: Q4 2027); IDP page-throughput against Azure Document Intelligence native (target: H1 2028). The forecast does not claim numbers; it commits to the matched-contract methodology and the publication dates. Reviewers reward methodological discipline more than premature numbers.

### Section 5 — Deployment Models (additive, ~500 words added)

Keep Greenfield, Brownfield, and Edge as currently described. Apply the previous audit's fix to soften "without rewrite" in Brownfield. Add two new deployment models.

**§5.4 Vertical SKU Subscription.** Approximately 250 words. Customer subscribes to one or more Platform SKUs from the §3.3 inventory. Customer does not interact with cap composition directly; they receive the SKU's pre-wired manifest. Customer can elect to upgrade to platform-tier subscription at any time, at which point they gain Studio access to author and modify cap compositions. The Code Detachment Fee from the Corelio biznesplan is reframed here as the one-time license that transfers cap composition ownership permanently to the customer's repository fork; this is the structural realization of the IP sovereignty promise.

**§5.5 Family Product Hosting.** Approximately 250 words. Independent SaaS products in the Exeris Systems family (BudgetHQ being the first; others may follow) run on the same kernel and Spring Runtime as platform subscribers. They are commercially independent: their pricing, customer base, branding, and roadmap are determined by their own product strategy, not by the platform's. This section establishes that the platform is genuinely usable by independent SaaS products with no special privileges — the dogfooding claim is structural, not theatrical.

### Section 6 — Sovereignty and IP Ownership (modest extension, ~200 words added)

Keep the existing BUS-001 and BUS-002 framing. Add a closing paragraph extending the sovereignty thesis to the SKU layer: when a customer detaches a vertical SKU under the §5.4 model, they receive not just the substrate code but also the cap composition manifest, the SKU's specific policy capabilities (subject to their licensing tier), and the build-time codegen artifacts needed to operate the SKU independently. The Code Detachment Fee is named, sized roughly (a one-time license in the low five-figure euros for SMB-tier SKUs, scaling with deployment size), and positioned as a feature rather than a barrier.

### Section 7 — Roadmap (major rewrite, ~400 words)

Replace the current roadmap with a two-track structure.

**Track A — Platform.** Q3 2026 TRL-5 component validation with first Edge/IoT pilots (revised from the current TRL-6 claim per the previous audit). Q4 2026 Studio MVP private beta (qualified to "read-only inspection plus targeted edit surfaces"). H1 2027 Exeris Spring Runtime 1.0 GA. H2 2027 Exeris Kernel 1.0 GA with SPI/Core/TCK stabilization.

**Track B — SKUs.** Q1 2027 capability composition language formal release. Q2 2027 API Gateway SKU GA. Q3 2027 Edge Proxy SKU GA. Q4 2027 Bot Blocker SKU GA. H1 2028 first Service Boundary SKUs (IDP and Headless CMS chosen first as the simplest cap compositions). H2 2028 PIM and OMS. Context-Centric CRM data model promoted to ADR in 2028, target H2 2029 for first product-form release.

**Family Products.** BudgetHQ enters private beta H2 2026, GA H1 2027, parallel to but independent of the Platform Track. The BudgetHQ date is mentioned but the roadmap does not subordinate it to platform milestones, because BudgetHQ is run as an independent product.

### New Section 8 — Exeris Systems Family Products (~400 words)

This new section introduces BudgetHQ specifically and establishes the family pattern generally.

Open with one paragraph defining what a Family product is and is not. A Family product is built and operated by Exeris Systems using the Exeris Platform. It is not a Platform SKU — meaning it is not sold as a cap composition to platform subscribers. It runs as an independent SaaS product with its own brand, pricing, and customer base. It serves three strategic functions: it dogfoods the platform in production, generating bug reports and performance data that no synthetic benchmark can produce; it diversifies Exeris Systems revenue away from B2B platform sales cycles; and it provides a reference implementation that demonstrates real production use of capabilities under development.

Follow with the BudgetHQ-specific paragraph: independent SaaS in the personal finance management category, positioned in Europe as "holistic net-worth tracking for the mass affluent segment," built on Exeris Spring Runtime in Pure Mode, integrating with Tink and Salt Edge through the platform's bank-aggregator capability (which is itself developed first inside BudgetHQ before being promoted to a reusable Service Boundary capability). Note the integration points without overstating them: BudgetHQ uses the IDP capability for receipt scanning; that same capability will later ship as the IDP Platform SKU.

Close with a paragraph explaining how Family products feed back into the platform. New capabilities are typically prototyped inside a Family product under production load, then promoted to reusable capabilities once they have stabilized. This is the dogfooding feedback loop. BudgetHQ in 2026–2027 will prototype: the bank-aggregator capability (Tink and Salt Edge SPI); the receipt-scan IDP integration; the OAuth and OIDC capability for B2C identity; and the subscription-billing capability (Stripe SPI). Each of these capabilities will land in the platform's capability ecosystem after BudgetHQ has stabilized them in production. This is the technical realization of the "Trojan horse" marketing logic from the Corelio biznesplan, but reframed as capability development pipeline rather than lead generation.

---

## 3. HLA Restructure

The current HLA is approximately 3,800 words. The restructured version targets **6,000–6,500 words**. The HLA changes are more substantial than the whitepaper changes because the HLA is the document technical reviewers will compare against the source repositories.

### Section 1 — Mission and Document Scope (minor edit)

Add one paragraph establishing the three-tier vocabulary and noting that the HLA covers the Substrate and Capability Ecosystem tiers in detail, with SKU-level architecture summarized but not exhaustively specified (those will receive their own SKU-specific architecture documents).

### Section 2 — System Context and Containers (critical fixes plus additive change)

Apply the previous audit's bootstrap-DAG fix in §2.3. The corrected DAG must read: **FOUNDATION: Memory (sequential) → SERVICES: Crypto & Persistence & Graph & Transport (parallel) → RUNTIME: Events & Flow & HTTP (parallel) → READY**. Remove Config and Exceptions from the DAG entirely. Remove Security from the DAG; it is L1 Citadel, not a boot phase. This single fix is the highest-priority change in the entire restructure.

Update the C4 Containers diagram (§2.2) to add a third layer above the current Substrate and Capability stripes: a "Vertical SaaS SKUs" stripe containing Gateway, Service Boundary, and Family product boxes. The Family product box should be visually distinguished (different fill or dashed border) to indicate it is not commercially equivalent to the SKU boxes.

Update the C4 Components diagram (§2.3) to fix the L0/L1 telemetry placement drift flagged in the previous audit. The decision is to follow `architecture.md` and place Telemetry under L0; the telemetry subsystem doc will need a corresponding update in its own pass.

### Section 3 — Platform Capabilities Map (restructure)

Reorganize the current capabilities map by tier rather than by capability type. Three subsections.

**§3.1 Substrate Capabilities.** Kernel-level capabilities currently in `exeris-kernel` and `exeris-kernel-enterprise`: zero-copy hot path, high-density compute, edge sovereignty, physical tenant isolation, compile-time RBAC, distributed saga orchestration, host-Spring integration, glass-box observability, stack-portable graph. These are not user-space capabilities — they are properties of the substrate itself. Each entry carries its ADR anchor (after the previous audit's dangling-reference cleanup).

**§3.2 Composable Capabilities.** User-space capabilities developed in the `exeris-caps-*` repositories. List with `@Provides` and `@Requires` declarations for each. Group by family (Gateway-family caps, Service Boundary-family caps, cross-cutting caps such as observability-bridge and rate-limiting). This is the layer that defines the cap composition vocabulary; reviewers need to see it explicitly.

**§3.3 SKU Compositions.** Named compositions of §3.2 capabilities. One row per SKU listing the capabilities it composes. Treat this as the source-of-truth manifest format that Studio will eventually consume.

### New Section 4 — Capability Composition Model (~700 words)

Insert between current §3 and §4 (renumbering downstream sections).

Open with a precise definition of the composition model. A capability is a named module with three contract surfaces: `@Provides` services exposed to other capabilities, `@Requires` declarations on capabilities it depends on, and a lifecycle (initialize, ready, drain, terminate) tied to the kernel bootstrap subsystem. A composition is a directed acyclic graph of capabilities with no unresolved `@Requires`. The kernel validates compositions at build time via the codegen pipeline (`exeris-tooling`, ADR-015) and refuses to start any composition with unresolved dependencies or cycles. This validation is what makes SKUs structurally trustworthy: a malformed cap composition fails at build time, not at runtime.

Follow with a sample composition walkthrough using the API Gateway SKU as the example: substrate cap `exeris-caps-gateway-core` provides `RouteRegistry`, `UpstreamPool`, `PolicyChain`, `BackendHealthMonitor`, `AdminControlPlane`; policy caps `exeris-caps-rate-limiting`, `exeris-caps-jwt-validation`, `exeris-caps-tls-termination`, `exeris-caps-request-routing`, `exeris-caps-observability-bridge` each require `gateway-core` and contribute to the `PolicyChain`. Include a small graph diagram showing the wiring.

Continue with the Enterprise engine swap pattern. The composition declares its required transport and crypto SPIs abstractly; the swap happens at deployment time by substituting `exeris-caps-quic-h3` for the standard transport cap and `exeris-caps-io-uring-transport` for the standard IO cap. The cap composition manifest does not change; the SPI implementations do. This is the architectural mechanism that makes "Enterprise license unlocks engine swap" structurally clean rather than rhetorical.

Close with the validation pipeline: compositions are validated at build time by the codegen pipeline; cycles, missing dependencies, version mismatches, and Wall violations all fail the build. The Wall (ADR-006) extends to capabilities — a capability cannot reach into Spring, into a sibling capability's internals, or into the kernel's private classes. This is what makes a cap composition reusable across host runtimes and detachable as IP.

### New Section 5 — First-Party Platform SKUs (~600 words)

Insert after the new §4.

Open with the SKU inventory table (mirror the whitepaper §3.3 table but with HLA-level technical detail — each row adds the host-runtime mode, the open-core tier per cap, and the target deployment topology). Follow with one paragraph per SKU family explaining its architecture: Gateway family (kernel-level HTTP path, no Spring dependency, single-process or distributed depending on cap manifest), Service Boundary family (typically Spring Runtime in Pure Mode for the API surface, kernel-level for the AI abstraction layer), Context-Centric CRM data model (cap-layer, consumed by Service Boundary SKUs, not a standalone product).

Close with a paragraph addressing the Spring Runtime relationship per SKU family. Gateway SKUs do not use Spring (consistent with the clarified ADR-021). Service Boundary SKUs use Spring Runtime in Pure Mode for their external API surface, demonstrating that brownfield migration from Spring-based vertical SaaS is structurally possible. Family products such as BudgetHQ use Spring Runtime in Pure Mode for their entire stack — this is the dogfooding case.

### Section 6 — Open-core split table (extension)

Add rows for every new `exeris-caps-*` repository specified in §3.2 of the new HLA. Each row declares its open-core tier (most are public; `exeris-caps-quic-h3`, `exeris-caps-io-uring-transport`, anything depending on the Enterprise SPIs, are enterprise-private). Reorganize the existing rows so substrate repositories appear first, then cap repositories, then SKU repositories (each Platform SKU lives in its own repository — for example `exeris-sku-api-gateway`), then Family product repositories (BudgetHQ in `budgethq-*` repositories under a separate organizational structure since it is an independent product).

The BudgetHQ entries deserve a clarifying note: BudgetHQ repositories are not in the open-core split because BudgetHQ is not open source — it is a proprietary Family product. They appear in the table only to clarify the boundary between platform IP and Family-product IP.

### Section 7 — Spring Adoption Paths (refine with BudgetHQ as case study)

Apply the previous audit's softening of "without rewrite" claims. Add a closing paragraph using BudgetHQ as the worked example: BudgetHQ is built entirely on Spring Runtime Pure Mode, consuming external APIs (Tink, Salt Edge, Stripe) through the platform's capability-level adapters, with the bank-aggregator capability prototyped inside BudgetHQ and later promoted to the platform's reusable capability ecosystem. This paragraph is the structural validation of the platform's brownfield claim: a real product runs on the platform from day one.

### Section 8 — Telemetry Architecture (minor extension)

Add one paragraph noting that capabilities emit JFR events via the `exeris-caps-observability-bridge` cap, which forwards them to Repo B consumers through the Repo C wire spec, identical to substrate-level telemetry. SKUs and Family products inherit this telemetry pipeline by composing the observability bridge cap.

### New Section 9 — Exeris Systems Family Products (~500 words)

Mirror the whitepaper §8 content at HLA technical detail. Open with the structural distinction between Platform SKUs (cap compositions sold to platform subscribers) and Family products (independent SaaS products built by Exeris Systems on the platform). Specify the architectural rules that Family products follow: they run on Exeris Spring Runtime (typically Pure Mode); they consume platform capabilities through the same mechanism platform subscribers do, with no special access; they may prototype new capabilities that later land in the platform's capability ecosystem after stabilization; their proprietary code, branding, and customer data are entirely outside the platform's IP perimeter.

Include a paragraph on BudgetHQ specifically as the HLA case study. Spring Runtime Pure Mode for the entire application stack. Bank-aggregator capability prototyped here (Tink and Salt Edge adapters, with the platform-level SPI abstraction designed first inside this product). Receipt-scan capability that bridges to the IDP capability via the platform's AI abstraction layer (the AI abstraction SPI that the Corelio WNF-03 specified will live in the platform's capability ecosystem, prototyped inside BudgetHQ). Subscription-billing capability (Stripe adapter, prototyped here, promotable to platform). OAuth/OIDC B2C identity capability. Each of these is named with its prototype-to-promotion path so reviewers can see the capability development pipeline.

Close with a paragraph on additional future Family products. The HLA should mention that the Family product pattern is extensible — future Family products may include other B2C SaaS verticals (the Corelio Anti-Account-Centric CRM idea could become a Family product before it becomes a Platform SKU, for example) — but should not list specific future products beyond BudgetHQ. Premature commitment to a long Family product roadmap weakens the focus claim.

---

## 4. ADR Work Required (parallel track)

The ADR work runs in parallel to the document restructure and gates the dangling-reference cleanup. Five ADR-pass items.

**Pass 1 — Cleanup of dangling references.** Resolve ADR-002, ADR-003, ADR-016, ADR-019, ADR-020, ADR-022 from the previous audit. For each, decide whether the ADR genuinely exists (and needs to be added to the documentation set) or whether the citation should be replaced with a prose pointer to the relevant subsystem document. This pass blocks the whitepaper §3 and HLA §3 rewrites because both reference these ADRs.

**Pass 2 — ADR-021 clarifying amendment.** Add a brief amendment to ADR-021 noting that Gateway-class workloads are out of scope for `exeris-spring-runtime` specifically, and pointing at the capability layer as their correct architectural home. This unblocks the Gateway SKU framing in the whitepaper.

**Pass 3 — Capability Composition Model ADR (new).** A formal ADR specifying `@Provides`/`@Requires` semantics, build-time validation, lifecycle, SKU-as-named-composition convention, and the relationship to the Wall (ADR-006) and codegen pipeline (ADR-015). This is the foundational ADR for Tier 2 and must land before the HLA §4 can be authoritative.

**Pass 4 — Gateway Substrate Cap ADR (new).** Establishes `exeris-caps-gateway-core` placement (kernel-level), scope (HTTP routing, upstream pooling, policy chain, health monitoring, admin control plane), `@Provides` contract, and explicit relationship to ADR-021. Unblocks HLA §5 Gateway family content.

**Pass 5 — TLS Fingerprinting Kernel Proposal (new).** Kernel-side proposal documenting the JA3/JA4 fingerprinting modification to `CoreSslHandles`. Cross-repo change touching `exeris-kernel` with downstream implications for `exeris-kernel-enterprise`. This is a smaller scope than a full ADR but should follow the same review path. Unblocks Bot Blocker SKU content.

Three additional ADRs are needed for the full vision but are not blocking: AI Abstraction Layer SPI (currently captured only in Corelio WNF-03), Anti-Account-Centric Context-Graph data model, and a per-Family-product architectural ADR for BudgetHQ if the product warrants one (probably yes, given the Tink and Stripe integration surface area).

---

## 5. Execution Order and Acceptance Criteria

The recommended execution order is:

**Week 1 (foundational fixes).** Complete ADR Pass 1 (dangling-reference cleanup) and Pass 2 (ADR-021 amendment). Apply the HLA §2.3 bootstrap-DAG fix and the whitepaper §4.2 TLS engine tier label fix. These are the highest-leverage corrections from the previous audit and they are prerequisites for everything else.

**Week 2 (capability foundation).** Draft and review ADR Pass 3 (Capability Composition Model). Draft and review ADR Pass 4 (Gateway Substrate Cap). Begin HLA §4 (Capability Composition Model) authoring.

**Week 3 (whitepaper restructure).** Complete the whitepaper §3 three-tier restructure, the §4 SKU benchmark forecast extension, the §5.4 Vertical SKU deployment model, the §5.5 Family product hosting paragraph, the §6 sovereignty extension, and the §7 roadmap rewrite.

**Week 4 (HLA restructure).** Complete the HLA §3 capabilities-map restructure, the new §5 First-Party Platform SKUs section, the §6 open-core split extension, the §7 Spring adoption refinement with BudgetHQ, the §8 telemetry extension, and the new §9 Family Products section.

**Week 5 (consistency review and new sections).** Author the new whitepaper §8 Family Products section. Resolve any cross-document inconsistencies introduced during the parallel restructure. Apply the §4.3 SKU benchmark forecast with measured methodology language. Final pass for terminology consistency (the family/platform/SKU vocabulary in particular).

**Week 6 (review and external sign-off).** Internal review against ADR set. Grants-reviewer pre-read with a trusted technical adviser. Final edits.

**Acceptance criteria** for the restructured documents are four:

1. Every ADR citation must resolve to either a present ADR file in the documentation set or to a prose pointer at a subsystem document.
2. Every Platform SKU named in the whitepaper §3.3 inventory must appear in the HLA §5 with a cap composition manifest.
3. The bootstrap DAG diagram in HLA §2.3 must match the canonical sequence in `exeris-kernel/docs/subsystems/bootstrap.md`.
4. BudgetHQ must appear in both documents framed as Family product dogfooding rather than as proof-of-concept or demo.

---

## 6. Post-Execution Reconciliation (added 2026-05-13)

This plan was executed across two parallel passes (12 May 2026 and 13 May 2026). During execution several plan assumptions turned out to be incorrect when validated against subsystem documentation and the user's architectural intent. This section captures the deviations between the plan-as-written and the documents-as-shipped. Final word counts: whitepaper 5,130 words; HLA 7,087 words.

### 6.1 Tier 2 capability decomposition (plan was too coarse)

**Plan §3.2 specified a flat policy-caps list** anchored to `exeris-caps-gateway-core` and `exeris-caps-service-boundary-core` substrate aggregates, with a handful of policy caps named (`rate-limiting`, `jwt-validation`, `tls-termination`, `request-routing`, `observability-bridge`).

**Shipped state** decomposes Tier 2 into **seven layers** with approximately fifty caps. Service Boundary specifically — which the plan treated as monolithic per-SKU — is fully decomposed in mirror to the Gateway family. The seven layers are: (1) substrate aggregates, (2) Gateway building blocks, (3) Gateway policies, (4) Service Boundary platform caps, (5) Domain primitives, (6) AI Abstraction layer, (7) Cross-cutting. This deviation was driven by the user's correction: "IDP, CMS, PIM, CRM, OMS powinny byc rozbite na bardziej szczegolowe caps i to z nich wlasnie powinno sie moc zbudowac caly CRM/ERP itp."

### 6.2 License taxonomy expanded from two values to three (plan assumed ADR-020 two-tier)

**Plan §3.2 specified** "open-core tier (public versus enterprise-private)" — a two-value scheme directly inherited from ADR-020 visibility taxonomy.

**Shipped state** introduces a three-tier **license** taxonomy distinct from ADR-020's visibility taxonomy: `community` (Apache 2.0), `commercial` (Exeris Commercial License — source-available, BSL-style, monetized), `enterprise-private` (closed source, Enterprise tier only). Distribution across the ~50 caps: three community (`cors-policy`, `i18n`, `observability-bridge`), ~46 commercial, exactly one enterprise-private (`bot-fingerprinting`). This deviation was driven by the user's correction: "CAPS skoro mamy je sprzedawac... to raczej nie powinny byc public zeby kazdy mogl z nich korzystac?". An ADR amendment to ADR-020 (or a new ADR formalizing capability license taxonomy as a separate dimension from documentation visibility) is now on the 2027 roadmap horizon and is **not yet drafted**.

### 6.3 Enterprise engine swap is Tier 1 substrate, not Tier 2 cap (plan was architecturally wrong)

**Plan §3.3 and §4 specified** that the Gateway-family Enterprise engine swap happens by substituting `exeris-caps-quic-h3` for the standard transport cap and `exeris-caps-io-uring-transport` for the standard IO cap — i.e. as a **Tier 2 cap manifest change**.

**Shipped state** reflects the correct mechanism: Enterprise engine swap is a **Tier 1 substrate driver swap**, achieved by replacing the Maven coordinate of the substrate driver JAR. Community substrate (`exeris-kernel`) ships custom NIO H1/H2 plus OpenSSL/Panama FFM TLS over TCP. Enterprise substrate (`exeris-kernel-enterprise`) ships `io_uring`/IOCP transport, `EnterpriseQuicTlsEngine`, HTTP/3 over QUIC, slab pools, and libnuma bindings. **The cap manifest does not change.** The caps `exeris-caps-quic-h3` and `exeris-caps-io-uring-transport` were removed from Tier 2 entirely and never shipped. The C4 Container diagram in HLA §2.2 reflects this with four boxes inside Tier 1 Substrate (Kernel SPI+Core, Community Driver, Enterprise Driver, Spring Runtime); Tier 2 is driver-agnostic. This deviation was driven by the user's correction: "Enterprise engine swap [...] przeciez tutaj to SPI sie nei rozni niczym a H1/H2 Custom Nio jest w open-core exeris-kernel a H1/H2/H3 (z QUIC) ioUring/iocp jest w exeris-kernel-enterprise".

### 6.4 Graph is dual-engine, not Postgres-only (plan inherited ADR-002 over-summary)

**Plan §3.1 implicitly inherited** the surface read of ADR-002 — "PostgreSQL 18 + SQL:2023 PGQ replacing Neo4j" — as the Graph stance.

**Shipped state** reflects the kernel's actual Graph SPI design per `exeris-kernel/docs/subsystems/graph.md`: a unified MATCH DSL that transpiles to **either** SQL:2023 PGQ (on Postgres 18) **or** Cypher (on Neo4j, Memgraph, FalkorDB). Both engine paths ship in Community. Enterprise tier adds a native Postgres v3 wire-protocol driver (off-heap, no JDBC tax) plus a planned FFM Bolt v5 driver for Neo4j (TRL-4, not yet shipping). ADR-002 itself is consistent with this — it scopes itself as a "platform-recommended stack for new applications, not a kernel mandate" — but the plan and an earlier draft of HLA conflated platform recommendation with kernel capability. Fixed in four locations in HLA (§2.1 mermaid, §2.1 paragraph, §3.1 capabilities row, §3.3 CRM data model paragraph). This deviation was driven by the user's correction: "z GRAPH jest blad bo mamy zarowno PGQ jak i NEO4J/CYPHER".

### 6.5 Events: Kafka is first-class Community driver, not optional adjunct

**Plan did not specify** the Events story in detail. The plan-implicit assumption (carried into early drafts) was that Kafka is an optional adjunct alongside the Postgres Outbox default.

**Shipped state** reflects the actual subsystem doc state: Community ships Postgres Outbox + JVM-heap pub/sub as the default, **plus** a first-class `exeris-kernel-community-kafka` Kafka 3.x driver since v0.7 Sprint 5b2 (2026-05-10). Fixed in HLA §2.1 mermaid (broker labeled as first-class Community Events driver) and the surrounding paragraph (explicit module name disclosure).

### 6.6 Roadmap dependency sequencing fixed

**Plan §7 Track A specified:** Q3 2026 TRL-5; Q4 2026 Studio MVP private beta; **H1 2027 Exeris Spring Runtime 1.0 GA; H2 2027 Exeris Kernel 1.0 GA**.

**Shipped state corrects two problems.** (a) Q3 2026 TRL-5 is too aggressive given v0.7.0 just shipped on 2026-05-10 (≈8–10 week sprint cadence) — recalibrated to **Q3 2026 TRL-4** integration-tested with first internal pilots; Q4 2026 absorbs TRL-5 component validation plus Studio MVP. (b) Spring Runtime 1.0 GA H1 2027 cited **before** Kernel 1.0 GA H2 2027 breaks dependency sequencing — Spring Runtime depends on stable Kernel SPI. Corrected to **H1 2027 Spring Runtime 1.0 RC + Kernel SPI lock; H2 2027 Kernel 1.0 GA and Spring Runtime 1.0 GA shipped together.** TRL-6 also moves to H2 2027 with Kernel 1.0.

### 6.7 ADR Pass 1 (dangling references) was moot

**Plan §4 Pass 1 listed** six dangling ADR references (ADR-002, ADR-003, ADR-016, ADR-019, ADR-020, ADR-022) for resolution. **All six exist** in `adr-index.md` and were verified during execution. The auditor that flagged them had an incomplete source set. Pass 1 produced no work.

### 6.8 ADR Pass 5 scope re-cut

**Plan §4 Pass 5 specified** a "TLS Fingerprinting Kernel Proposal" for JA3/JA4 only. **Three additional ADR drafts now sit on the 2027 horizon** as a result of the discoveries in this restructure: (a) Capability Composition Model ADR — formalizes `@Provides`/`@Requires`/build-time validation/lifecycle; (b) Capability License Taxonomy ADR — formalizes the three-tier license dimension introduced in §6.2 above as a separate axis from ADR-020 visibility taxonomy; (c) AI Abstraction Layer SPI ADR — captures content currently only sketched in Corelio WNF-03 and HLA §3.2 Layer 6. All three are referenced from whitepaper §7 Track A as roadmap items; none are drafted yet.

### 6.9 Forward-reference markers added during sweep

A final sweep added "(planned)" markers for documents and tracks that the docs reference but which do not yet exist as concrete artifacts: `edge-rss-baseline.md` in `exeris-benchmarks` (planned for v0.8.0+); ADR-016 H3 track scope document (`exeris-benchmarks-enterprise` README is currently a placeholder).

### 6.10 Plan vs reality summary

| Plan assumption | Shipped state | Source |
|---|---|---|
| Flat policy-cap list | 7-layer / ~50-cap decomposition | §6.1 |
| Two-tier license (`public` / `enterprise-private`) | Three-tier license (`community` / `commercial` / `enterprise-private`) | §6.2 |
| Enterprise swap is Tier 2 cap substitution | Enterprise swap is Tier 1 driver substitution | §6.3 |
| Postgres-only graph (replacing Neo4j) | Dual-engine: PGQ (Postgres) + Cypher (Neo4j/Memgraph/FalkorDB) | §6.4 |
| Kafka as optional adjunct | Kafka as first-class Community driver | §6.5 |
| Spring 1.0 H1 2027 → Kernel 1.0 H2 2027 | Both ship together H2 2027 after H1 2027 RC | §6.6 |
| 6 dangling ADRs to resolve | All 6 exist; pass moot | §6.7 |
| 5-pass ADR work plan | 8 pass items on horizon, 3 new | §6.8 |
| `service-boundary-core` `@Requires: kernel SPI, exeris-spring-runtime` | `@Requires: kernel SPI` only — host runtime is Tier 1 selection, not cap dependency | §6.11 (intermediate) |
| SB-family Platform SKUs are Spring-Runtime-hosted | Platform SKUs are kernel-direct; Spring Runtime is independent Tier 1 product, only customer-brownfield + BudgetHQ use it | §6.12 (final) |
| Family products run on Spring Runtime | Only BudgetHQ runs on Spring Runtime (singular dogfooding case); future Family products pure-Exeris | §6.12 (final) |
| ADR-023 silent on SKU repository source-visibility | Source-available default for all SKU repos; Bot Blocker singular closed-source exception (anti-abuse security principle); Code Detachment scope follows visibility | §6.13 |

### 6.13 SKU repository source-visibility policy (ADR-023 same-day amendment, 2026-05-13)

**ADR-023 as originally drafted (earlier today)** explicitly listed "License changes for Platform SKU repositories themselves" in its "What is NOT in scope" carve-out — meaning the ADR formalised the three-tier license taxonomy at the Tier 2 cap layer but deferred the Tier 3 SKU question entirely.

**Same-day amendment (2026-05-13) resolved the gap.** A strategic analysis surfaced three coherent postures (fully-closed SKUs / source-available SKUs / mixed) and recommended **source-available SKUs as the default with the Bot Blocker family as a single principled exception**. The reasoning compresses to four points: (a) Glass Box auditability must hold at every audit layer including SKU; (b) the competitive moat is the kernel (Enterprise tier especially), not the SKU code; (c) Code Detachment Fee economics are qualitatively stronger when "detach the SKU" includes the SKU source; (d) CNCF / EU public-sector positioning requires source-availability at the SKU layer.

**Shipped state.** ADR-023 received a "SKU Repository Source-Visibility Policy" same-day amendment with: a per-SKU policy table (6 source-available + 1 closed-source Bot Blocker); 4 new obligations (7–10) covering three-source consistency, irreversibility, Code Detachment scope by visibility, and source-available leak-rate mitigations; explicit cross-references to whitepaper §3.3, HLA §3.3, HLA §6.3, whitepaper §5.4. The Bot Blocker exception is framed as "principled inconsistency that strengthens rather than weakens the position" — same posture Cloudflare, Akamai Bot Manager, and every serious bot-protection vendor maintains.

Fixed in: ADR-023 same-day amendment ("SKU Repository Source-Visibility Policy" section + removal of the corresponding bullet from "What is NOT in scope"); whitepaper §3.3 SKU table (added Source-visibility column to all 8 rows + Source-visibility policy paragraph); whitepaper §5.4 Code Detachment paragraph (tightened to distinguish source-available vs closed-source detachment scope per ADR-023 obligation 9); HLA §3.3 SKU compositions table (added Source-visibility column + policy paragraph); HLA §6.3 Tier 3 Platform SKUs table (added Source-visibility column).

**Pricing-model forward reference.** The strategic analysis also sketched a four-tier pricing model (Free / Platform-Community / Platform-Enterprise / Managed cloud) plus the Code Detachment Fee operating orthogonally. ADR-023 amendment captures this as a forward reference only; the canonical pricing-model ADR lives in the BUS-NNN namespace and will be drafted separately. The source-availability decision is independent of the pricing-model decision: source-available B2B infrastructure generates revenue through operational guarantees, Enterprise-tier features, managed hosting, and services — not through source-code gatekeeping (GitLab / Confluent / Elastic / MongoDB / Grafana revenue-model pattern; ~$5B+ combined annual revenue across these five public-source-visible B2B infrastructure companies).

### 6.11 Spring Runtime is Tier 1 host, never a cap dependency (intermediate correction)

**Plan §3.2 specified** that `exeris-caps-service-boundary-core` `@Requires` includes `exeris-spring-runtime`. Plan §3.3 and HLA §5 framed the SB family as "composes Spring Runtime in Pure Mode for the external API surface" — implying SKU = `cap manifest including Spring Runtime`.

**Intermediate shipped state corrected the dependency direction.** No cap `@Requires` `exeris-spring-runtime`. The cap-tier Wall (HLA §4: "a capability cannot reach into Spring internals") forbids this. An initial fix modelled SKU compositions as having "two orthogonal Tier 1 substrate selections" — (a) driver swap, (b) host-runtime selection where SB SKUs select Spring Runtime — keeping caps Spring-free. **This intermediate model was further superseded by §6.12 below**, which removes Spring Runtime from the platform entirely.

This intermediate deviation was driven by the user's correction: "exeris-spring-runtime nie bedzie nigdy CAPS ani nie bedziemy uzywac Springa w CAPS wiec co ma spring-runtime do caps?".

### 6.12 `exeris-spring-runtime` is an INDEPENDENT Tier 1 product; the platform does NOT use it (final correction)

**Plan and §6.11 intermediate state framed** `exeris-spring-runtime` as a host-runtime selection that Service Boundary–family Platform SKUs and BudgetHQ layer in at Tier 3. This positioned Spring Runtime as part of the platform stack.

**Final shipped state is more aggressive.** `exeris-spring-runtime` is an **independent Tier 1 product** sold separately from the kernel and from Platform SKUs. The platform itself — kernel, capability ecosystem, and all Tier 3 first-party Platform SKUs (both Gateway-family and Service Boundary-family) — runs **kernel-direct** on pure Exeris, with the API surface emitted from `@ExerisDomain` types + `@Action` methods through `rest-emission` / `graphql-emission` / `openapi-emission` codegen capabilities (ADR-015). There is no Spring `@RestController` in any first-party SKU. No cap `@Requires` `exeris-spring-runtime`. No SKU manifest layers it in.

`exeris-spring-runtime` has only two consumers, both outside the first-party SKU set:
1. **Customer applications doing brownfield Spring migration** — teams with existing Spring code adopting the Exeris kernel without rewriting business code. This is the structural showcase that Exeris is a **runtime** (not a framework competing with Spring) — strong enough to host Spring code underneath.
2. **BudgetHQ** — the **singular Spring-on-Exeris Family product**, deliberately structured to dogfood the Spring Runtime as a shippable product under real customer load. All future Family products will be pure-Exeris on SDK + tooling; BudgetHQ's dogfooding role for Spring Runtime is filled once and does not need to be repeated.

This deviation was driven by two consecutive user corrections: (i) "No na platformie w ogole nie bedziemy uzywac Spring Runtime, to jest niezalezne repo tier 1 dla zespolow ktore nie chca migrowac w pelni do exeris a maja aplikacje springowe - to jest tylko showcase ze Exeris to RUNTIME a nie framework" and (ii) "ale BHQ to bedzie jedyny produkt na Exeris Spring Runtime XD reszta bedzie pure Exeris i to na SDK+TOOLING (future caps na platforme)".

Fixed in: HLA §1 Executive Summary item 3, HLA §2.1 Person + System definitions, HLA §2.2 C4 Container (separate Spring Runtime into its own Tier 1 boundary; SB SKU box reframed to kernel-direct; Rel(sku_sb, spring_rt) removed), HLA §3.1 row reframed, HLA §3.2 host-runtime note rewritten, HLA §4 "Tier 1 substrate driver swap" paragraph rewritten (single-axis swap, not dual-axis), HLA §5 SKU table all SB rows changed from "Spring Runtime Pure Mode" to "Kernel-direct (@ExerisDomain + rest-emission codegen)", HLA §5 SB family architecture paragraph rewritten, HLA §5 Spring Runtime relationship paragraph rewritten, HLA §7 Spring Adoption Paths reframed as customer-facing brownfield migration path, HLA §9 Family product rules reworked (BudgetHQ as singular exception), HLA §9.2 extensibility paragraph (future Family products pure-Exeris), whitepaper §3.1 row reframed, whitepaper §3.3 SB family description rewritten, whitepaper §5.5 Family Product Hosting paragraph rewritten, whitepaper §8 BudgetHQ + future Family products framing.

The original plan §3.2 cap definition for `service-boundary-core` (with Spring Runtime in `@Requires`), plan §3.3 SB family description, and plan HLA §7 framing ("Spring Adoption Paths" as platform-internal) should all be read as superseded historical-plan-intent only.
