# ADR-004: JDK 26 EA + Preview Features Mandate

| Atrybut         | Wartość                                                                                                  |
|:----------------|:---------------------------------------------------------------------------------------------------------|
| **Status**      | **ACCEPTED** (formal record authored 2026-05-08 — see retrospective note below)                          |
| **Deciders**    | Arkadiusz Przychocki                                                                                     |
| **Date**        | 2025-12-26                                                                                               |
| **Scope**       | platform (binds every Exeris repository — kernel, sdk, spring-runtime, tooling, enterprise)              |
| **Owning Repo** | `exeris-docs`                                                                                            |
| **Driven By**   | No-Waste Compute thesis; JDK 26 EA availability (work started 2025-09-16 when EA dropped)                |
| **Compliance**  | [Strategic Pillar: No-Waste Compute](../../exeris-kernel/docs/whitepaper.md)                             |

> **Retrospective record.** The decision was made on 2025-12-26 and operationalized across all repositories well before this ADR was written. The formal record was authored on 2026-05-08 as part of the ADR registry bootstrap. The Date field reflects the actual decision date, not the authoring date.

## Context and Problem Statement

The Exeris platform's "No Waste Compute" thesis depends on JVM features that crystallised across JDK 21–26:

- **Virtual Threads** went stable in JDK 21 — but the `synchronized` carrier-pinning fix did not land until **JEP 491 (JDK 24)**. Before that fix, any `synchronized` method on a VT-mounted carrier would pin the carrier thread, defeating the entire scalability model.
- **Foreign Function & Memory API (Panama FFM)** went stable in **JEP 454 (JDK 22)**. This is the load-bearing API for enterprise tier features: `io_uring` ring management, QUIC OpenSSL BIO pairs, off-heap slab pools, and zero-copy `MemorySegment` paths.
- **ScopedValue (JEP 506)** matured into a real `ThreadLocal` replacement — required for context propagation under VTs without the GC pressure and pinning hazards of TL.
- **StructuredTaskScope (JEP 525)** matured into a real `ExecutorService` replacement for orchestration — required for the structured-concurrency discipline mandated across kernel hot paths.
- **Valhalla previews** continue to evolve toward value classes (JEP 401 family). Records and immutable final classes designed today against current preview semantics will scalarise via C2 escape analysis when Valhalla GAs.

A platform pinned at JDK 21 LTS (or even JDK 25 LTS) cannot honour these contracts. JDK 21 lacks the pinning fix, FFM stability, and mature ScopedValue/StructuredTaskScope. JDK 25 lacks JEP 491. The platform's value proposition is incompatible with LTS.

## 🏁 The Decision

**Mandate JDK 26 (EA → GA when released) with `--enable-preview` across every Exeris repository.** No backport to LTS.

**Concrete obligations:**

1. **Compiler target.** Every `pom.xml` sets `<maven.compiler.release>26</maven.compiler.release>` and passes `--enable-preview` to both `maven-compiler-plugin` and `maven-surefire-plugin`.
2. **Test JVM args.** Test runners use `-XX:+UnlockExperimentalVMOptions -XX:+UseZGC --enable-preview` (consult repo-specific `pom.xml` for additional flags).
3. **Operator deployment.** Production deployments require JDK 26 (EA acceptable until GA). There is no "fallback to 21 LTS" path.
4. **Preview-feature drift.** Preview features may shift between EA versions. Code MUST track the latest EA semantics, not pin to an older preview shape. CI runs against the latest stable EA build.
5. **Enforcement.** Build fails if `release` is set below 26; `PureModeClasspathGuardTest`-style guards check the runtime JVM major version on bootstrap.

**History note:** JDK 26 EA work in this codebase started on **2025-09-16** (the day the EA build dropped). The platform-level mandate was formalised on **2025-12-26**. The 3-month gap covers the period during which compatibility with JDK 25 LTS was still being evaluated and rejected.

## Consequences

### ✅ Positive Outcomes

- **[+] Coherent runtime model.** ScopedValue + StructuredTaskScope + FFM + VTs (with pinning fix) compose into a single design philosophy. No subsystem has to fall back to legacy primitives.
- **[+] Zero pin-pollution.** JEP 491 closes the `synchronized` pinning gap. Slab-pool ABA-safety blocks (per ADR-007 amendment) are now legal without carrier pinning.
- **[+] Native-interop without `Unsafe`.** Panama FFM replaces every legacy `sun.misc.Unsafe` use case in the kernel and enterprise tiers.
- **[+] Valhalla-ready.** Records and immutable final classes designed today scalarise under C2 EA today and become true value types when Valhalla GAs — no migration required.

### ⚠️ Trade-offs

- **[-] Operator JDK requirement.** Every Exeris deployment requires JDK 26+. Operators on RHEL/Ubuntu LTS with vendor-blessed JDK 21 cannot run Exeris without upgrading the JVM. This is a deliberate trade — the platform's performance contract is incompatible with LTS.
- **[-] Preview-feature volatility.** Preview features may rename, change signatures, or be removed across EA builds. The repos accept the maintenance cost of tracking EA semantics in exchange for early access to the runtime model.
- **[-] Tooling lag.** Some IDEs, static analysers, and bytecode tools lag behind preview features. Workarounds may require manual configuration.

### 📋 What is NOT in scope

- Backwards-compatibility shims for JDK 21 / 25. None are provided. Code that targets older JDKs is rejected.
- Multi-release JARs. The platform ships JDK 26 bytecode only; no `META-INF/versions/21/` paths.

## Cross-references

- ADR-007 (Next-Gen Runtime Architecture) — depends on this ADR's preview-features mandate to be possible.
- ADR-005 (JFR-First Telemetry) — depends on `@StackTrace(false)` semantics stable since JDK 21+ and refined in JDK 26.
- `exeris-kernel/CLAUDE.md` §"Hard Constraints" — operationalises the bans that this ADR's feature mandate makes possible.

## Engineering Protocol

Once this decision is ACCEPTED, every repo's `pom.xml` must reflect `<maven.compiler.release>26</maven.compiler.release>` and the `--enable-preview` flags. Existing repos already comply; this ADR codifies the existing reality.
