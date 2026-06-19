# ------------------------------------------------------------------------------
#  README.md
#  Software problem patterns  --  recurring problem structures in code.
# ------------------------------------------------------------------------------

# Software Problem Patterns

A repository of recurring problem structures in software construction. Each pattern describes a problem type  --  its domain structure, invariants, failure modes, and frame concern  --  without prescribing a solution.

## Where This Fits in the Workflow

Every design problem cycles through four phases: **Model  ->  Build  ->  Evaluate  ->  Reflect**.

| Phase | How this repository helps |
|-------|--------------------------|
| **Model** | Decompose your system into subproblems. Open a pattern when you recognise its problem type. Use the planet name as a module name  --  it says "this module instantiates this problem type." Define types and invariants from the pattern's Domains and Invariants sections. |
| **Build** | Start with the pattern's "Start with (minimal form)"  --  the simplest correct implementation. Stub and iterate. |
| **Evaluate** | Check the pattern's "Known Failure Modes"  --  are you hitting any? Compare against "Real project examples" in [examined-projects.md](examined-projects.md). |
| **Reflect** | Check "Split into its own module when" triggers  --  have they fired? Run 5 whys analysis on any issues found. If patterns no longer match, rename the module. |

See [how-to-use.md](how-to-use.md) for a full walk-through and project-type-to-patterns reference.

## Principles

- **Patterns are problem descriptions, not solution templates.** The solution emerges from instantiating the problem structure in a specific context.
- **The name is a foreign key.** Each pattern has a canonical planet name. Assigning that name to a module during construction says "this module instantiates this problem type."
- **Patterns compose.** Every real system decomposes into multiple patterns connected by shared domains and phenomena.

## Structure

```
patterns/                      --  27 problem patterns (one per planet name)
  mercury-display.md
  venus-filtering.md
  earth-sorting.md
  mars-persistence.md
  ...
index.md                       --  master pattern table, common clusters, project mappings
how-to-use.md                  --  entry point: combinations table, walk-through, mindset shifts
examined-projects.md           --  62+ open-source projects mapped to patterns
template.md                    --  pattern authoring template
planet-names.md                --  name pool (planets: top-level, moons: sub-modules)
```

## Key Files

| File | Purpose |
|------|---------|
| [how-to-use.md](how-to-use.md) | Start here. Combinations table by project type, walk-through, navigation. |
| [index.md](index.md) | Master pattern table, common clusters, per-module project mappings. |
| [examined-projects.md](examined-projects.md) | Registry of analysed projects  --  use during Evaluate to compare instantiations. |
| [template.md](template.md) | Pattern authoring template with required sections. |
| [planet-names.md](planet-names.md) | Hierarchical name pool for naming subproblems during Model. |

## Pattern Template

See [template.md](./template.md).

## Planet Name Pool

Patterns are indexed by planet name. Unused names available for new patterns:

`ceres`, `makemake`, `haumea`, `titania`, `oberon`, `miranda`, `ariel`, `umbriel`, `tethys`, `enceladus`, `mimas`, `phobos`, `deimos`, `proteus`, `triton`, `nereid`, `styx`, `nyx`, `kerberos`, `hydra`

See [planet-names.md](planet-names.md) for the full hierarchical pool.
