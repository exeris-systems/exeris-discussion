# ADR-020: Open-Core Documentation Boundary & Cross-Repo Mirror Policy

| Atrybut         | Wartość                                                                                                       |
|:----------------|:--------------------------------------------------------------------------------------------------------------|
| **Status**      | **PROPOSED** (drafted 2026-05-08; decision date 2026-05-05)                                                   |
| **Deciders**    | Arkadiusz Przychocki                                                                                          |
| **Date**        | 2026-05-05                                                                                                    |
| **Scope**       | platform (binds every Exeris repository — public open-core and private enterprise alike)                      |
| **Owning Repo** | `exeris-docs`                                                                                                 |
| **Driven By**   | ADR-008 (Open-Core Strategy); cleanup pass on 2026-05-07 that consolidated platform-scope ADRs                |
| **Compliance**  | [`adr-index.md`](../adr-index.md) §Rules §Visibility                                                          |

## Context and Problem Statement

Exeris is distributed as **open-core**: a set of public repositories that any consumer can clone, build, and deploy, plus a set of private enterprise repositories that ship the high-density runtime extensions.

**Public open-core repos** (consumers see these): `exeris-kernel`, `exeris-kernel-community`, `exeris-sdk`, `exeris-spring-runtime`, `exeris-tooling`, `exeris-platform`, `exeris-benchmarks`, `exeris-telemetry-spec`, `exeris-docs`.

**Private enterprise repos** (commercial overlay): `exeris-kernel-enterprise`, `exeris-benchmarks-enterprise`, `exeris-enterprise-observability`.

ADRs and architectural documents move freely across both sides. Some examples:
- ADR-001 (Cloud Native) lives in `exeris-docs` and applies to every repo, public and private.
- ADR-007 (Next-Gen Runtime) lives in `exeris-kernel` (public) and binds `exeris-kernel-enterprise` (private).
- ADR-018 (Observability Tooling Repo Split) lives in `exeris-kernel-enterprise` (private), is enterprise-private content, but its number is publicly registered in this index and consumed by `exeris-telemetry-spec` (public) via a `.link.md` stub.

Without a clear policy, three classes of bug recur:

1. **Duplicate full-content copies.** The same document gets full-content copies in multiple repos, each editable, each eventually drifting. (Discovered during the 2026-05-07 cleanup: ADR-001 had two full copies — in `exeris-kernel/docs/adr/` and in `exeris-kernel-enterprise/docs/adr/` — that had already drifted in line endings and were on a path to substantive divergence.)
2. **Refactor leakage.** Internal refactor working notes — sometimes Polish-language R&D archaeology — get filed as ADRs and assigned global numbers. (Discovered: local refactor docs at numbers 010/011/012 inside `exeris-kernel-enterprise/docs/adr/` that used the global ADR-NNN namespace despite being internal-only.)
3. **Number collisions.** When local-only refactor docs use the global ADR-NNN numbering, they collide with the registry's authoritative assignments. (Discovered: kernel-enterprise local ADR-010 (Persistence SPI Refactor) collided with global ADR-010 (Host Runtime Model, owned by `exeris-spring-runtime`).)

The 2026-05-07 cleanup fixed the immediate state. This ADR locks in the policy that prevents these failure modes from recurring.

## 🏁 The Decision

**The open-core boundary is repo-level. Each document has exactly one authoritative location. Cross-repo navigation uses `.link.md` stubs. Drift between authoritative copies and stubs is build-detectable.**

### 1. Authoritative-location rule

Every ADR and every architecturally-load-bearing document (whitepaper, architecture, performance-contract, subsystem docs, module docs) has **one and only one** authoritative full-content location. That location is determined by ownership:

| Document type                                              | Authoritative location                                  |
|:-----------------------------------------------------------|:--------------------------------------------------------|
| Platform-scope ADR                                         | `exeris-docs/adr/`                                      |
| Per-repo ADR (kernel, sdk, spring-runtime, tooling, ...)   | `<repo>/docs/adr/`                                      |
| Cross-repo ADR                                             | Owning repo's `docs/adr/`; other repos hold link stubs  |
| Enterprise-private ADR                                     | `<enterprise-repo>/docs/adr/`                           |
| Subsystem doc (transport, persistence, crypto, ...)        | `<repo>/docs/subsystems/<name>.md` in the owning repo   |
| Whitepaper / architecture / performance-contract           | `exeris-kernel/docs/` is the canonical home for kernel-platform documents; an enterprise-private extension may live at `exeris-kernel-enterprise/docs/<doc>.md` only when it materially extends the public document — and the extension must clearly cite the canonical version it overlays |

If a document is intended for public consumption, its authoritative copy lives in a public repo. If it is enterprise-private, its authoritative copy lives in an enterprise repo. There is no third option.

### 2. Cross-repo navigation via `.link.md` stubs

When a document owned by repo A is referenced by repos B, C, ..., each consuming repo MAY hold a `<docname>.link.md` (or `ADR-NNN.link.md`) stub that:

- Names the authoritative path (relative or absolute, but unambiguous).
- States in one short paragraph why the consuming repo cares.
- Lists "when to consult" triggers relevant to the consuming repo.

Stubs are **navigation aids**, not content. They MUST NOT replicate the authoritative content. A stub that drifts into a half-summary is a policy violation.

### 3. Visibility taxonomy (canonical)

The `adr-index.md` Visibility column uses exactly two values:

- **`public`** — content lives in a public repo. Anyone can read it.
- **`enterprise-private`** — content lives only in a private repo. Number is publicly registered in `adr-index.md`; content is not.

The legacy value **`public-staged`** is **deprecated by this ADR**. Existing rows that carry it (if any) are reclassified as `public` (when authoritative content already lives in a public repo) or `enterprise-private` (when it does not). On acceptance of this ADR, `adr-index.md` Rules §3 must be updated to drop `public-staged` from the canonical taxonomy.

### 4. Refactor working notes are NOT ADRs

Documents whose primary purpose is to track the mechanics of a refactor — file moves, package renames, migration phases, internal R&D archaeology, Polish-language working notes — live in `<repo>/docs/refactor-notes/` (or in PR descriptions / commit messages). They are NEVER assigned ADR numbers and NEVER mirrored across repos. The 2026-05-07 cleanup relocated the historical kernel-enterprise refactor docs (Persistence SPI Refactor, Transport Provider SPI Surface, Graph Engine SPI Refactor) to `exeris-kernel-enterprise/docs/refactor-notes/` accordingly.

### 5. Drift detection (mandatory)

A CI job verifies cross-repo consistency:

- Every `<docname>.link.md` stub names a path that resolves to an existing file in the named repo.
- For every ADR row in `exeris-docs/adr-index.md`, the linked authoritative file exists at the named path.
- No two repos contain full-content copies of the same logically-identical document. (Detection heuristic: identical `# Title` headers in `docs/adr/` files across repos trigger a review flag; identical content under non-overlay filenames triggers a hard failure.)
- Every ADR file under `<repo>/docs/adr/` matches a row in `exeris-docs/adr-index.md` (either as authoritative copy or as `.link.md` stub) — orphan ADR files in any repo are a policy violation.

The drift-detection job runs in CI for `exeris-docs` and aggregates results across repos. Until the job lands, periodic manual audits (like the 2026-05-07 pass) substitute.

## Consequences

### ✅ Positive Outcomes

- **[+] Single source of truth.** Each ADR has exactly one editable location across the entire ecosystem.
- **[+] Boundary clarity.** "Is this doc public or enterprise-private?" reduces to "which repo does it live in?". No `public-staged` ambiguity.
- **[+] Refactor notes stay private.** Internal mechanics (Polish R&D, migration phases) never accidentally cross the boundary.
- **[+] Drift becomes a build outcome.** "Did this PR break a stub link?" is a CI failure, not a review judgment call.

### ⚠️ Trade-offs

- **[-] Visibility migration touches every row.** ADRs currently marked `public-staged` (if any remain after the 2026-05-07 pass) need to be re-classified. The migration is mechanical but every entry must be touched.
- **[-] Cross-repo CI complexity.** Drift-detection requires checkouts of multiple repos; a heavier build configuration than a typical single-repo CI step. Acceptable for a release-gate job, not a per-PR step.
- **[-] Authors must classify intent at write time.** When adding a new ADR or doc, the author must answer: "Is this for public release? If yes, in which public repo?" — and act accordingly. Repo-level CLAUDE.md updates should call this out.

### 📋 What is NOT in this ADR

- This ADR does not redefine what `open-core` means strategically. ADR-008 owns that.
- This ADR does not retroactively reclassify existing ADRs. Index updates happen as separate PRs guided by this policy.
- This ADR does not specify any tooling for promoting an ADR from `enterprise-private` to `public`. Promotion is a manual move-file-across-repos action when the decision is ready for public release.

## Cross-references

- ADR-008 (Open-Core Strategy) — the strategic framing this policy serves.
- ADR-018 (Observability Tooling Repo Split) — established the `.link.md` stub pattern that this ADR generalises.
- ADR-001 (Cloud Native) — first ADR migrated under this policy (PR on 2026-05-07 moved the authoritative copy to `exeris-docs/adr/` with link stubs in consuming repos).
- `adr-index.md` Rules §Visibility — to be updated to remove `public-staged` and reflect this ADR's two-value taxonomy.

## Engineering Protocol

Once this decision is ACCEPTED:

1. Update `adr-index.md` Rules §3 (Visibility) to remove the `public-staged` value and document the two-value canonical taxonomy.
2. Implement the cross-repo drift-detection CI job in `exeris-docs` (or as a release-gate workflow). Until it lands, document a periodic-audit cadence.
3. Each repo's CLAUDE.md (or CONTRIBUTING.md) should reference this ADR when discussing where new documentation belongs.
