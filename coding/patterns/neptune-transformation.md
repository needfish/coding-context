# ------------------------------------------------------------------------------
#  neptune-transformation.md
#  Behavior variation -- the mapping algorithm changes per source/target pair.
# ------------------------------------------------------------------------------

# - Problem -

You have data in one shape and you need it in a different shape. Same information, different structure -- like converting a database row to a JSON response, or a Celsius reading to Fahrenheit.

# - When You'll See This -

Any time two parts of your system use different representations for the same data. Serialisation (internal objects to bytes and back), DTO mapping (database models to API responses), version migration (v1 format to v2 format). Transformation connects two subproblems that would otherwise be incompatible.

# - How It Works -

- **Source representation**: the input data in its original form.
- **Target representation**: the output data in the desired form.
- **Mapping**: the rules that translate source fields to target fields.

# - What Must Always Be True -

- If the mapping is bidirectional, transforming there and back gives you the original (round-trip).
- Source fields with no target equivalent are handled explicitly -- dropped, defaulted, or flagged as error.
- The transformation adds no information that wasn't in the source, unless the mapping explicitly defines it.
- Transformation has no side effects on the source or external systems.

# - What Goes Wrong -

- **Information loss**: the target format can't express something the source has, and nobody notices.
- **Silent coercion**: converting a float to an int truncates the decimal without warning.
- **Incomplete mapping**: some source fields have no target defined, producing broken output.
- **Cyclic structures**: the source has circular references and the transformer loops forever.
- **Version mismatch**: the source data is from a different schema version than the mapping expects.
- **Identity loss**: two values that mean different things ("123" and 123) become identical in the target.

# - Design Trade-Offs -

- **Explicit vs. convention-based**: listing every field correspondence is clear but verbose; using conventions and defaults is compact but surprising.
- **Strict vs. forgiving**: strict mode fails on unexpected input; forgiving mode skips unknown fields.
- **In-place vs. copy**: transforming in place saves memory but risks aliasing; copying is safe but costs time and space.
- **Declarative vs. procedural**: declarative mappings are easier to verify; procedural transformations are more powerful.

# - Related Problems -

- **mars/persistence**: saving data to disk is a transformation (serialisation).
- **mercury/display**: data often needs reshaping before display.
- **janus/reduction**: transformed elements often get reduced (map then reduce).
- **jupiter/validation**: validation runs before transformation.
- **venus/filtering**: filtered items might need transformation.
- **earth/sorting**: sort keys might need transformation before comparison.

# - Getting Started -

Write a function that takes the source type and returns the target type. Map fields one at a time. Python: a dict comprehension or a simple loop. Rust: `From` / `Into` trait implementations or a `match` on fields.

# - When to Split Into Its Own Module -

- The same transformation logic is needed in multiple places.
- The mapping has special cases, conditional logic, or error handling.
- You need to test the transformation independently from the data sources.
- The transformation becomes bidirectional and you need to keep both directions in sync.

# - Signs You're in This Pattern -

- You're writing code that converts one data structure into another.
- The conversion logic has field renames, type casts, or default values.
- You see the same field mapping in multiple places with slight differences.
- Adding a field means updating the conversion code in several spots.

# - Common Mistakes -

- Building a generic mapping framework before you have three different transformations.
- Using implicit conventions (field names must match!) without documenting them.
- Forgetting to handle null/missing fields in one direction.

# - The Core Challenge -

Transformation sits between two representations that evolve independently. When the source adds a field, the target might not need it -- but you need to decide explicitly whether to pass it through, drop it, or flag it. The challenge is keeping the mapping complete and correct as both sides change over time.
