# Software Problem Patterns

A repository of recurring problem structures in software construction. Each pattern describes a problem type — its domain structure, invariants, failure modes, and frame concern — without prescribing a solution.

## Principles

- **Patterns are problem descriptions, not solution templates.** The solution emerges from instantiating the problem structure in a specific context.
- **The name is a foreign key.** Each pattern has a canonical planet name. Assigning that name to a module during construction says "this module instantiates this problem type."
- **Patterns compose.** Every real system decomposes into multiple patterns connected by shared domains and phenomena.

## Structure

```
patterns/
  mercury-display.md
  venus-filter.md
  earth-sort.md
  mars-persist.md
  jupiter-validate.md
  saturn-route.md
  uranus-authenticate.md
  neptune-transform.md
  ...
```

## Pattern Template

See [template.md](./template.md).

## Planet Name Pool

Patterns are indexed by planet name. Unused names available for new patterns:

`pluto`, `ceres`, `makemake`, `haumea`, `titania`, `oberon`, `miranda`, `ariel`, `umbriel`, `callisto`, `ganymede`, `europa`, `io`, `titan`, `rhea`, `iapetus`, `dione`, `tethys`, `enceladus`, `mimas`, `phobos`, `deimos`, `proteus`, `triton`, `nereld`, `charon`, `styx`, `nyx`, `kerberos`, `hydra`
