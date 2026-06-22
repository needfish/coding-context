# ------------------------------------------------------------------------------
#  jupiter-validation.md
#  Rules variation -- rules combine, grow, and change independently.
# ------------------------------------------------------------------------------

# - Problem -

Data comes into your system from the outside. Before you process it, you need to check that it meets your requirements -- right shape, right types, right ranges, right values.

# - When You'll See This -

Any boundary where external data enters -- user input, API requests, file imports, network messages, config files. Validation is a gate: it either passes data through or rejects it with a reason. Valid data then flows to transformation, storage, or routing.

# - How It Works -

- **Input**: the raw data to check -- could be JSON, form fields, a text file, or bytes from the network.
- **Rule set**: the constraints the input must satisfy -- a schema, a list of predicates, a type definition.
- **Result**: the validated data (if it passes) or a rejection report listing what failed.

# - What Must Always Be True -

- Accepted data satisfies every rule in the rule set.
- Rejected data comes with a report saying which rule failed and on which field.
- The same input and rules always produce the same result (deterministic).
- Validation never mutates the input or any system state -- it's a pure check.

# - What Goes Wrong -

- **Silent coercion**: the validator changes "123" (string) to 123 (int) without telling anyone, and downstream code gets a type the author didn't expect.
- **Contradictory rules**: two rules can never both be true, so nothing ever passes.
- **Too loose**: the rule is too broad and lets bad data through.
- **Too strict**: the rule rejects valid data it should accept.
- **Slow validation**: rules query external systems or run expensive checks, blocking the caller.
- **Vague errors**: "invalid input" with no indication of which field or rule failed.
- **Version drift**: the rules changed but old data was checked against an older version.

# - Design Trade-Offs -

- **Strict vs. lenient**: strict catches errors early but might reject borderline data; lenient accepts more but pushes detection downstream.
- **Report all vs. report first**: showing all errors at once helps the caller fix everything; showing only the first avoids wasted work.
- **Declarative vs. imperative**: schemas are easier to audit but less expressive; code is more flexible but harder to verify.
- **Parsing vs. validation**: parsing checks structure (is this valid JSON?), validation checks meaning (is this value in range?). Keep them separate.

# - Related Problems -

- **neptune/transformation**: validation runs before transformation -- clean data first, then reshape it.
- **mars/persistence**: data gets validated before writing.
- **saturn/routing**: incoming requests get validated before routing.
- **mercury/display**: user input from a form needs validation before action.
- **venus/filtering**: filter predicates might need validation.

# - Getting Started -

Inline `if` checks at the point of use. Python: `if not isinstance(value, int): raise ValueError(...)`. Rust: `if value > MAX { return Err(...) }`. No schema library, no validator trait.

# - When to Split Into Its Own Module -

- The same validation rules appear in multiple places (you're copy-pasting).
- The rule set grows beyond what fits on one screen (10+ rules).
- Rules need composition (AND/OR across fields) or conditionality (field A requires field B).
- Validation needs to produce structured error reports (field-level error codes).

# - Signs You're in This Pattern -

- Your functions start with guard clauses that check inputs and return errors.
- The same validation checks appear in multiple functions with slight variations.
- You need to reject bad input but the checks are mixed in with processing logic.
- Users submit data that causes crashes because boundary conditions weren't checked.

# - Common Mistakes -

- Adopting a schema library (JSON Schema, Zod, Pydantic) before you have 5+ fields to validate.
- Building a declarative rule engine before `if` checks are proven to be the bottleneck.
- Validating data that comes from a trusted internal source.

# - The Core Challenge -

The validator can only enforce the rules it's given. It can't know that a missing rule would have caught a problem. The real challenge isn't checking rules -- it's knowing you have enough rules. What the rule set doesn't say is as important as what it says.
