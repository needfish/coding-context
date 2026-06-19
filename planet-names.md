# ------------------------------------------------------------------------------
#  planet-names.md
#  Hierarchical name pool for pattern modules.
# ------------------------------------------------------------------------------

# Planet Name Pool

Hierarchical list of planet names and their moon names. During the **Model**
phase, assign a planet name to each subproblem module  --  the name acts as a
foreign key back to the pattern.

**Naming convention (per AGENTS.md):** Planets are top-level module names;
moons are sub-module names. The hierarchy suggests decomposition  --  a planet
and its moons form a family of related problem patterns. Module directories
and files use kebab-case.

```
mercury
venus
earth
  moon
mars
  phobos
  deimos
jupiter
  metis, adrastea, amalthea, thebe
  io, europa, ganymede, callisto
  leda, himalia, elara
  ananke, carme, pasiphae, sinope
saturn
  pan, atlas, prometheus, pandora
  epimetheus, janus
  mimas, enceladus, tethys, telesto, calypso
  dione, helene
  rhea, titan, hyperion, iapetus, phoebe
uranus
  cordelia, ophelia, bianca, cressida, desdemona
  juliet, portia, rosalind, belinda, puck
  miranda, ariel, umbriel, titania, oberon
  caliban, sycorax, prospero, setebos, stephano, trinculo
neptune
  naiad, thalassa, despina, galatea, larissa
  proteus, triton, nereid
pluto
  charon
eris
  dysnomia
```

## Flat List (alphabetical for quick lookup)

adrastea, amalthea, ananke, ariel, atlas, belinda, bianca, caliban, callisto, calypso, carme, charon, cordelia, cressida, deimos, desdemona, despina, dione, dysnomia, earth, elara, enceladus, epimetheus, eris, europa, galatea, ganymede, haumea, helene, himalia, hydra, hyperion, iapetus, io, janus, juliet, jupiter, kerberos, larissa, leda, makemake, mars, mercury, metis, mimas, miranda, moon, naiad, neptune, nereid, nereld, nyx, oberon, ophelia, pan, pandora, pasiphae, phobos, phoebe, pluto, portia, prometheus, prospero, proteus, puck, rhea, rosalind, saturn, setebos, sinope, stephano, styx, sycorax, telesto, thalassa, thebe, tethys, titan, titania, trinculo, triton, umbriel, uranus, venus
