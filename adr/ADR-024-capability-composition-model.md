# ADR-024: Capability Composition Model — `@Provides` / `@Requires` / Build-Time Validation

| Attribute       | Value                                                                                                                |
|:----------------|:---------------------------------------------------------------------------------------------------------------------|
| **Status**      | **ACCEPTED**                                                                                                         |
| **Deciders**    | Arkadiusz Przychocki                                                                                                 |
| **Date**        | 2026-05-13                                                                                                           |
| **Scope**       | platform (binds every `exeris-caps-*` repository, every `exeris-sku-*` repository, the `exeris-tooling` codegen pipeline, and the kernel bootstrap lifecycle contract) |
| **Owning Repo** | `exeris-docs`                                                                                                        |
| **Driven By**   | 2026-05-12 whitepaper / HLA restructure that introduced the three-tier architecture; Tier 2 Capability Ecosystem requires a formal contract for how caps compose into Tier 3 SKUs |
| **Compliance**  | [HLA §4 Capability Composition Model](../high-level-architecture.md), [Whitepaper §3.2 Tier 2 — Capability Ecosystem](../b2b-technical-whitepaper.md), [ADR-006 The Wall](ADR-006-spring-free-kernel-boundary.md), [ADR-015 Codegen Emission Strategy](../../exeris-tooling/docs/adr/ADR-015-codegen-emission-strategy.md) |

## Context and Problem Statement

The 2026-05-12 whitepaper / HLA restructure established a three-tier architecture: Tier 1 Substrate (kernel + drivers + Spring Runtime as independent product) — Tier 2 Capability Ecosystem (`exeris-caps-*`) — Tier 3 Vertical SaaS SKUs (`exeris-sku-*`). HLA §4 sketches the Capability Composition Model that binds Tier 2 caps into Tier 3 SKU manifests. Whitepaper §3.2 mirrors the same model for buyer-facing communication. Both documents forward-reference "a forthcoming ADR" for the formal contract. This ADR is that contract.

Without a formal model, three classes of failure recur as the cap layer scales beyond a handful of repositories:

1. **Unresolved dependencies discovered at runtime.** A cap declares `@Requires(SomeOther)` but the SKU manifest doesn't include `SomeOther`. With no build-time validation, the failure surfaces at kernel bootstrap or — worse — at the first request that exercises the missing dependency. The cap layer must fail at build time, not at runtime.
2. **Cycles between caps.** Cap A `@Requires` cap B; cap B `@Requires` cap A directly or transitively. Without DAG enforcement, the bootstrap order is ill-defined and the developer experience degrades fast — the same class of bug Spring Boot's `@DependsOn` chains and OSGi service-tracker dependencies have produced for two decades.
3. **Cap-tier Wall violations.** A cap reaches into Spring internals, into a sibling cap's private classes, or into a kernel private package. The substrate-tier Wall (ADR-006) is enforced by ArchUnit-style tests in `exeris-kernel` and `exeris-spring-runtime`; the cap tier needs its own extension of the same discipline because caps are the layer where third-party glue code is most tempted to short-cut the SPIs.

A fourth concern is **detachability** (whitepaper §5.4 Code Detachment Fee + §6 Sovereignty). A cap composition is only truly portable as IP if a customer can lift the composition into a customer-owned fork without dragging hidden classpath dependencies. The model must structurally guarantee that detachment is mechanical, not a multi-week rewrite.

This ADR answers: **what is the formal contract surface of an `exeris-caps-*` repository, how does the build-time pipeline validate compositions of such caps, and what is the relationship between the capability composition model and the kernel bootstrap lifecycle?**

## 🏁 The Decision

**A capability is a named module declaring three contract surfaces — `@Provides`, `@Requires`, and a lifecycle — through annotations consumed by the `exeris-tooling` codegen pipeline (ADR-015). A composition is a directed acyclic graph of capabilities with no unresolved `@Requires`. The codegen pipeline validates compositions at build time; the kernel refuses to start any composition that fails validation. This contract is enforced uniformly across every Tier 2 `exeris-caps-*` repository and every Tier 3 `exeris-sku-*` SKU manifest.**

### Capability contract surface

Every `exeris-caps-*` repository declares its contract surface through three annotation families and one lifecycle interface:

1. **`@Provides(Service service, [version = "M.m.p"])`** — a service interface this cap exposes to other caps and to user / SKU code. Multiple `@Provides` declarations per cap are permitted (a "substrate-aggregate" cap like `exeris-caps-gateway-core` provides `RouteRegistry`, `UpstreamPool`, `PolicyChain`, `BackendHealthMonitor`, `AdminControlPlane`). The annotation lives on a `@CapabilityModule` class in the cap's API package.
2. **`@Requires(Service service, [versionRange = "[M.m.p,N.x.x)"], [optional = false])`** — a service this cap depends on. Resolution at build time matches a `@Requires` edge against a `@Provides` declaration in another cap on the same composition. Optional `@Requires` (with `optional = true`) is permitted for cross-cutting concerns where the cap degrades gracefully — typically observability and telemetry hooks.
3. **`@CapabilityLifecycle`** — a marker on the class implementing the lifecycle hooks (see lifecycle subsection below). A cap may have at most one `@CapabilityLifecycle` class.

Capabilities consume **kernel SPIs** (Transport, HTTP, Crypto, Persistence, Graph, Events, Flow, Security, Telemetry, Memory) through the same `@Requires` mechanism, with kernel SPIs declared as well-known service identifiers (`KERNEL_TRANSPORT`, `KERNEL_HTTP`, etc.). This unifies "depends on another cap" and "depends on a kernel subsystem" into one contract surface, which keeps the build-time validator simple.

### Lifecycle

Every capability participates in a four-phase lifecycle bound to the kernel bootstrap state machine (`exeris-kernel/docs/subsystems/bootstrap.md` — canonical bootstrap DAG: `FOUNDATION: Memory → SERVICES: Crypto & Persistence & Graph & Transport (parallel) → RUNTIME: Events & Flow & HTTP (parallel) → KERNEL READY`):

| Phase | Trigger | Cap responsibility |
|:---|:---|:---|
| `initialize` | After kernel `READY`, before first request | Acquire owned resources (off-heap buffers, native handles, registry slots). Idempotent — must support replay during composition validation dry-runs. |
| `ready` | After every cap in the composition completes `initialize` | Cap announces readiness to accept work. The composition emits a "composition ready" event only when every cap has signalled. |
| `drain` | On orderly shutdown, before `terminate` | Cap stops accepting new work, completes in-flight units, flushes telemetry. Must complete within the configured drain deadline (default 30s; configurable per SKU). |
| `terminate` | After `drain` completes or its deadline expires | Cap releases owned resources unconditionally. Re-entry-safe — a `terminate` call on an already-terminated cap is a no-op. |

The lifecycle ordering between caps is **derived mechanically** from the `@Requires` DAG: a cap's `initialize` runs after the `initialize` of every cap it transitively requires; a cap's `terminate` runs before the `terminate` of every cap that transitively requires it. There is no `@CapabilityOrder` annotation or hand-written priority — declared dependencies are the only ordering source.

### Composition

A **composition** is a manifest file (`composition.yaml` or equivalent, format TBD by the codegen team in coordination with `exeris-tooling`) that names:

- A set of cap coordinates (group / artefact / version per cap).
- Per-cap configuration overrides (typed property maps).
- A signature (per ADR-020 the manifest is itself a versioned artefact; signing detail is delegated to the SKU repository convention).

A composition is **valid** when all four predicates hold:

1. **Every `@Requires` edge resolves.** For every `@Requires(S)` in every cap, there exists a cap in the composition with `@Provides(S)` whose version satisfies the `versionRange`. Optional `@Requires` (`optional = true`) need not resolve; the consuming cap is responsible for `null`-safe handling.
2. **No cycles.** The `@Requires` graph across all caps in the composition is a DAG.
3. **No version conflicts.** When multiple caps declare `@Provides(S)`, the composition resolves to exactly one — either by version-range intersection (when all consumers' ranges overlap) or by an explicit `prefers:` directive in the composition. Ambiguity is a build failure.
4. **No Wall violations.** Per the cap-tier Wall extension below, no cap class imports across forbidden boundaries.

A composition that fails any predicate **fails the build**. The kernel refuses to start any composition lacking a "validated" stamp from the codegen pipeline.

### The Wall, extended to capabilities

ADR-006 establishes the substrate-tier Wall: `exeris-kernel-spi` and `exeris-kernel-core` are Spring-free; `exeris-kernel-enterprise` depends only on SPI and Core. This ADR extends the same architectural discipline to the cap tier as **the cap-tier Wall**:

- **No cap imports `org.springframework.*`, `io.netty.*`, `reactor.*`, `jakarta.servlet.*`, or any host-runtime-specific package.** Caps consume kernel SPIs and other caps via `@Requires`; host-runtime selection happens at SKU manifest time at Tier 1, never inside a cap. This is what keeps `exeris-spring-runtime` an independent Tier 1 product whose absence does not break any cap (per ADR-021 amendment 2026-05-13 and the `exeris-spring-runtime/CLAUDE.md` "Who consumes this repo" section).
- **No cap reaches into another cap's private classes.** Only types annotated as `@Provides`d services are visible across the cap boundary. The convention is the same `eu.exeris.caps.<cap-name>.api.*` (visible) vs `eu.exeris.caps.<cap-name>.internal.*` (private) split that the kernel uses for SPI vs Core.
- **No cap reaches into kernel private packages.** `eu.exeris.kernel.*.internal.*` and `eu.exeris.kernel.core.internal.*` are off-limits — only the SPI surface (`eu.exeris.kernel.spi.*`) is callable from cap code.

The cap-tier Wall is validated by the same `exeris-tooling` pipeline that performs `@Requires` resolution. A cap repository whose ArchUnit-style guard fails is rejected before its artefact is published; an SKU manifest that contains a cap with disabled or stale Wall guards is rejected by the kernel at boot.

**Concrete obligations:**

1. **Every `exeris-caps-*` repository declares its contract surface through `@Provides` and `@Requires` annotations on a `@CapabilityModule` class.** Caps without these annotations are not loadable. The annotations are processed by the `exeris-tooling` annotation processor (ADR-015) at build time and emit a `cap-manifest.json` artefact alongside the cap's JAR.
2. **The kernel refuses to start any composition lacking a "validated" stamp.** The codegen pipeline emits the validation stamp into the composition manifest only when all four predicates of the validation algorithm pass. The kernel bootstrap checks for the stamp during the FOUNDATION phase, before any subsystem initialises.
3. **The cap-tier Wall is enforced by build-time ArchUnit-style guards in every cap repository.** A new `exeris-caps-*` repository scaffold ships with the standard guard set; modifying or disabling the guards is a registry violation reported through periodic audits until automated cross-repo CI lands.
4. **Lifecycle ordering is derived mechanically from `@Requires`; no manual priorities.** A cap that needs to run before another cap declares the dependency explicitly through a (potentially empty-payload) `@Requires` service marker. Priority hacks (`@Order`, integer priorities, alphabetic ordering) are not permitted and fail the build.
5. **Composition manifests are version-pinned and signed.** An SKU manifest pins every cap to an exact version (no `LATEST`, no version range broader than a single release). Signing detail is delegated to the SKU repository convention (`exeris-sku-*`) — this ADR's obligation is the pinning discipline, not the signature algorithm.
6. **Code Detachment artefact set is defined by composition validation output.** When a customer pays the Code Detachment Fee (whitepaper §5.4), the artefacts they receive are exactly the cap source repositories named in the composition manifest, plus the build-time codegen output (cap manifests, validation stamp, signed composition manifest), plus the perpetual-use grant per ADR-023 obligation 4. Detachment is mechanical because the model guarantees no hidden classpath dependencies — every dependency is declared in `@Requires` and every dependency is in the manifest.

## Consequences

### ✅ Positive Outcomes

- **[+] Composition becomes a build-time concern.** Missing dependencies, cycles, version conflicts, and Wall violations all fail the build, before any deployment. The class of bug where "the SKU starts but crashes on the first request" is structurally impossible.
- **[+] Tier 2 cap layer is uniformly enforceable.** ~50 caps across seven layers all participate in the same contract surface. Reviewers across cap repositories can apply the same rubric without per-cap special cases.
- **[+] Lifecycle ordering is derived, not declared.** Caps state their dependencies; the codegen pipeline computes the topological order; no human ranks caps with magic numbers. This is the same architectural move that made the kernel bootstrap DAG safe (subsystem ordering derived from contracts, not from `@Priority`).
- **[+] Cap-tier Wall protects detachability.** A `commercial`-licensed cap (per ADR-023) that a customer detaches under the Code Detachment Fee can be lifted into a customer-owned fork with no hidden classpath. The Wall guards that the cap doesn't secretly import Spring or sibling-cap internals are what make this guarantee mechanical.
- **[+] Cap composition model is host-runtime-independent.** A composition is byte-identical whether the SKU manifest layers in Spring Runtime (the brownfield path through ADR-021 amendment) or runs kernel-direct (the default Tier 3 Platform SKU shape per HLA §5). This is the structural foundation of "Spring Runtime is an independent Tier 1 product, not part of the platform stack" — caps don't know whether Spring is on the classpath.

### ⚠️ Trade-offs

- **[-] Annotation-driven contract has a learning curve.** Cap authors must internalise `@Provides` / `@Requires` semantics and the four-phase lifecycle before contributing. The platform absorbs this cost via `exeris-tooling` scaffolding (a `mvn archetype:generate` for new cap repos, planned) and via cap-author documentation that ships alongside the first reference cap.
- **[-] Build-time validation requires the codegen pipeline.** A cap repository builds with the `exeris-tooling` annotation processor on the classpath. A cap that needs to build outside the platform pipeline (third-party fork, bisected investigation) has to either include the annotation processor or accept that the validation stamp won't be emitted. Mitigation: the processor is a public Maven artefact under the kernel's open-core licence.
- **[-] Mechanical lifecycle ordering occasionally costs an extra empty-marker `@Requires`.** When cap A must initialise before cap B but doesn't naturally depend on a service B provides, the only way to express the ordering is for A to declare `@Requires(B_INIT_MARKER)` and B to declare `@Provides(B_INIT_MARKER)`. This is verbose but explicit; it preserves the "no manual priority" discipline at the cost of marker-service noise. Empirically rare based on the cap inventory in HLA §3.2 — the natural service dependencies cover the bulk of the cases.

### 📋 What is NOT in scope

- **The annotation library implementation (`exeris-sdk` or a new `exeris-caps-spi`).** This ADR specifies the contract; the concrete package, class, and processor implementation is a separate `exeris-sdk` ADR or implementation task — drafted alongside the first `exeris-caps-*` repository creation.
- **The composition manifest format (YAML / JSON / pkl / etc.).** Delegated to the SKU repository convention (`exeris-sku-*`) in coordination with `exeris-tooling`. The codegen pipeline owns the canonical reader; the format choice is a separate implementation decision.
- **Hot reload, dynamic cap swapping, and runtime composition change.** This ADR specifies a build-time validated, boot-time activated composition. Dynamic cap reload is a future concern (potentially after Kernel 1.0 GA) and is not in scope.
- **Per-cap performance contracts.** Whether a specific cap meets a latency or allocation budget is a per-cap concern; the model enforces composition correctness, not per-cap performance.
- **Cross-SKU cap sharing semantics.** Whether two SKUs deployed in the same JVM can share a single instance of a cap (e.g. one `exeris-caps-observability-bridge` serving both an API Gateway SKU and an IDP SKU co-located on the same node) is delegated to a future deployment-topology ADR. The default this ADR locks in is one cap instance per SKU.
- **Telemetry of the composition itself.** Capability lifecycle events emit JFR via the `exeris-caps-observability-bridge` (per ADR-018 and HLA §8). Compositional metadata (which caps loaded, manifest version, validation stamp) is exposed through the same surface, but the wire-format details belong to ADR-018, not here.

## Cross-references

- ADR-006 (Spring-Free Kernel Boundary — The Wall) — the substrate-tier Wall that this ADR extends to the cap tier.
- ADR-015 (Codegen Emission Strategy, `exeris-tooling`) — the annotation-processor pipeline that consumes `@Provides` / `@Requires` declarations and emits the cap manifest + composition validation stamp.
- ADR-020 (Open-Core Documentation Boundary) — defines the documentation visibility taxonomy this ADR's caps live under.
- ADR-021 (Gateway-Class Workloads Out of Compatibility Scope) — its 2026-05-13 amendment cites this ADR as the binding model for Tier 3 Gateway SKU compositions.
- ADR-023 (Capability Licensing Taxonomy) — companion ADR; this ADR governs the contract, ADR-023 governs the licence under which the contract operates.
- HLA §4 "Capability Composition Model" — the document this ADR formalises.
- Whitepaper §3.2 "Tier 2 — Capability Ecosystem" — buyer-facing summary.
- `exeris-kernel/docs/subsystems/bootstrap.md` — the canonical bootstrap DAG that the capability lifecycle binds to.
- Whitepaper §5.4 "Vertical SKU Subscription" + §6 "Sovereignty and IP Ownership" — Code Detachment Fee that this ADR's obligation 6 supports structurally.

## Engineering Protocol

This ADR is **descriptive at acceptance**: it codifies the composition model already sketched in HLA §4 and whitepaper §3.2 (2026-05-12). It becomes prescriptive when:

1. The first `exeris-caps-*` repository materialises (target: H1 2027 per whitepaper §7 Track B, "Q1 2027 Capability composition language formal release").
2. The annotation-processor extension in `exeris-tooling` ships (depends on ADR-015 deliverables; planned alongside the first cap repository).
3. The kernel bootstrap acquires the "validation stamp" check (target: bundled with the first cap-aware kernel release).

Open follow-ups (tracked separately):

1. **`exeris-sdk` ADR (or amendment) defining the concrete annotation classes.** Specifies the `@CapabilityModule`, `@Provides`, `@Requires`, `@CapabilityLifecycle` Java types, their retention policy, and their package location. Drafted alongside the SDK's first cap-aware release.
2. **`exeris-tooling` extension for cap manifest emission + composition validation.** Implements the four-predicate validator and the manifest emitter. Coordinated with ADR-015 deliverables.
3. **Composition manifest format specification.** YAML / JSON / pkl decision plus the canonical reader implementation. Delegated to the SKU repository convention; first SKU manifest shipped alongside `exeris-sku-api-gateway` (target: Q2 2027 per whitepaper §7 Track B).
4. **Cap-author documentation set.** A `cap-author-guide.md` in `exeris-docs` covering: declaring `@Provides` / `@Requires` correctly, lifecycle hook patterns, Wall-compliant import hygiene, version-range conventions, optional-dependency patterns. Drafted alongside the first reference cap.
5. **Cross-repo CI gate for composition validation.** A workflow in `exeris-docs` (or distributed across cap repositories) that pulls cap manifests and verifies a sample SKU composition validates. Lands when the first two cap repositories ship.

Until the codegen pipeline lands, the binding source-of-truth is the contract specified in this ADR plus the HLA §4 walked example (API Gateway SKU composition diagram). New cap repositories scaffolded before the pipeline ships must include hand-written `@Provides` / `@Requires` declarations matching this ADR's annotation grammar so that the migration to processor-validated builds is a re-compile, not a refactor.
