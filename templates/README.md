# Document Templates

Canonical templates for **decision documents** in the Exeris ecosystem. Three templates for three question shapes.

## Templates

| Template | File | Use when |
|:---|:---|:---|
| **ADR** | [`ADR-TEMPLATE.md`](ADR-TEMPLATE.md) | A decision is made. The document records what, why, and how it's enforced. |
| **RFC** | [`RFC-TEMPLATE.md`](RFC-TEMPLATE.md) | A multi-option strategic/policy question is open. The document enumerates options and recommends one. |
| **Research** | [`RESEARCH-TEMPLATE.md`](RESEARCH-TEMPLATE.md) | A falsifiable hypothesis needs measurement (JMH, JFR, profiling, prototypes). Lab-notebook shape; produces data and a falsifiable conclusion. |

All three live here at `exeris-docs/templates/` so they're available to every repo.

## Picking the right template

The three are not interchangeable. Pick by the *shape of the question*:

| | Research | RFC | ADR |
|:---|:---|:---|:---|
| **Question shape** | "Does X work as predicted?" | "Should we choose A, B, or C?" | "We've decided X." |
| **Measurable hypothesis?** | Yes — falsifiable | No — multiple options compared | N/A — already decided |
| **Output** | Data tables + conclusion | Recommendation + reasoning against alternatives | Concrete obligations + enforcement |
| **Drives** | Often → ADR, sometimes → feature | → ADR | (terminal) |

**Rule of thumb:** if you can write a measurable hypothesis ("X will reduce Y by Z% under workload W"), use Research. If your decision is "which option do we pick?", use RFC. If the decision has already been made and you just need to record it, go straight to ADR.

## Lifecycle

```
              ┌────────────────────────────────────────┐
              │  Question forms                         │
              └───┬────────────────────────────────┬────┘
                  │ measurable hypothesis           │ multi-option / strategic
                  ▼                                 ▼
           ┌─────────────┐                  ┌─────────────┐
           │  Research   │                  │ RFC (DRAFT) │
           │  (active)   │                  └──────┬──────┘
           └──────┬──────┘                         │ review / iteration
                  │ experiments + data             ▼
                  ▼                         ┌─────────────────┐
           ┌──────────────────┐             │  RFC (ACCEPTED) │
           │ Research.Decision│             └────────┬────────┘
           │ (concluded)      │                      │
           └──────┬───────────┘                      │
                  │                                  │
                  └──────────────┬───────────────────┘
                                 ▼
                          ┌─────────────┐
                          │  ADR(s)     │  ← Decision recorded, enforceable
                          └─────────────┘
```

A single Research effort or RFC may produce multiple ADRs.

**You may skip the upstream stage** when:
- The decision is descriptive (codifies existing reality), not prescriptive — go straight to ADR.
- The decision is a small per-repo convention with no cross-repo blast radius.
- You've already done the analysis informally and there's no losing alternative whose proponents will ask "why didn't we do X?" later.

**You should NOT skip the upstream stage** when:
- The decision needs measurable data to justify (use Research).
- The decision affects multiple repos and has clear losing alternatives (use RFC).
- The decision will be questioned later (use whichever upstream form fits — capture the reasoning).

## Where the documents live

- **ADR location** is governed by [ADR-020](../adr/ADR-020%20Open-Core%20Documentation%20Mirror%20Policy.md): platform → `exeris-docs/adr/`; per-repo → `<repo>/docs/adr/`; cross-repo → owning repo with `.link.md` stubs in consumers; enterprise-private → `<enterprise-repo>/docs/adr/`.
- **RFC location** may be this repo, the relevant code repo, or an external system (Confluence, Notion). The accepted RFC's URL is referenced in the resulting ADR's `Driven By` field.
- **Research location** is the relevant code repo's `docs/research/` directory, on a `research/<slug>` branch — see the kernel's `docs/research/RESEARCH.md` framework doc for the established branch workflow. Research can happen in any repo where measurement-driven decisions arise.

## Conventions

- **Filenames.**
  - ADR: `ADR-NNN <Short Title>.md` (3-digit zero-padded).
  - RFC: `RFC-YYYY-MM-DD <Short Title>.md` (date prefix).
  - Research: `<short-slug>.md` on a `research/<slug>` branch.
- **Numbering.** ADR numbers are reserved in [`../adr-index.md`](../adr-index.md) before content is written. RFC IDs are the date the RFC opens — no central registry. Research has no central registry — it's branch-scoped.
- **Status discipline.** ADRs in PROPOSED and RFCs in DRAFT should accept, reject, or withdraw within a couple of weeks. Research with `Status: active` for more than one milestone should conclude, park, or abandon.
- **Language.** Body content of registry-tracked documents (ADRs, RFCs that drive ADRs, the index, this README) is in English. Frontmatter table labels (`Atrybut` / `Wartość`) are preserved as a historical convention from the early ADRs; new ADRs may use them or English equivalents (`Attribute` / `Value`) — both forms are recognised. Research lab-notebook content may be informal. Repo-local working notes (refactor archaeology, internal R&D) may be in any language but never enter the ADR namespace.

## Cross-references

- [`../adr-index.md`](../adr-index.md) — central tech ADR registry.
- [`../business-adr-index.md`](../business-adr-index.md) — separate `BUS-NNN` namespace for business / legal / financial decisions.
- [ADR-020](../adr/ADR-020%20Open-Core%20Documentation%20Mirror%20Policy.md) — where ADRs live across the open-core / enterprise repo split.
- `exeris-kernel/docs/research/RESEARCH.md` — kernel research framework (when, how, branch lifecycle, current portfolio).
