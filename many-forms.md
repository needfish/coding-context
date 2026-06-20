# ------------------------------------------------------------------------------
#  many-forms.md
#  A JavaScript/TypeScript code parallel to William Kent's "The Many Forms
#  of a Single Fact" (1989). Each option shows a different way to model the
#  fact that a SALESMAN serves a TERRITORY.
# ------------------------------------------------------------------------------

> **Reference:** William Kent, "The Many Forms of a Single Fact", Proc. IEEE
> COMPCON, Feb. 27 - Mar. 3, 1989, San Francisco. Also HPL-SAL-88-8.
>
> This file translates Kent's 37+ field-design options into JavaScript and
> TypeScript representations. The original paper uses flat-file records; here
> each option is shown as an object shape, a type definition, or a data
> pattern in common JS/TS idioms.

# 1. Direct Field

The simplest representation: a territory property on the salesman object.

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  territory: string;
}
```

# 2. Unnormalized (Fact in a Different Record)

The fact appears in a record where neither salesman nor territory is the key.

```typescript
interface Order {
  orderId: string;
  item: string;
  quantity: number;
  price: number;
  salesman: string;
  territory: string;
}
```

# 3. Multi-Field Representation

A single conceptual entity (territory) is represented by multiple fields.

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  country: string;
  region: string;
  district: string;
}
// "territory" no longer appears as a property name.
```

# 4. Embedded Data in Identifier

The territory code is embedded inside the salesman identifier.

```typescript
// Salesman ID format:  "SLS-{TERRITORY_CODE}-{SEQUENTIAL}"
// Example: "SLS-NY-042"
type SalesmanId = string; // contains territory code at position 5-6

interface Salesman {
  id: SalesmanId;   // territory derivable from id
  name: string;
  department: string;
}

function extractTerritory(salesman: Salesman): string {
  return salesman.id.split("-")[1]; // "NY"
}
```

# 5. Shared (Multiplexed) Field

Several facts encoded into one field via a composite value.

```typescript
// Assignment format: "{TERRITORY}:{PRODUCT}"
// Example: "NY:WIDGETS"
type Assignment = string;

interface Salesman {
  id: string;
  name: string;
  department: string;
  assignment: Assignment;
}

function parseAssignment(a: Assignment): { territory: string; product: string } {
  const [territory, product] = a.split(":");
  return { territory, product };
}
```

# 6. Omitted Field -- Data in Record Type Name

Record types (classes) are partitioned by territory; the territory identifier is
part of the type/constructor name.

```typescript
class WesternSalesman {
  constructor(public id: string, public name: string, public department: string) {}
  territory = "WEST";
}

class EasternSalesman {
  constructor(public id: string, public name: string, public department: string) {}
  territory = "EAST";
}

type AnySalesman = WesternSalesman | EasternSalesman;
```

# 7. Omitted Field -- Implicit from Context

Territory is a constant for all salesmen in this context. Not recorded in data.

```typescript
// All records in this file are for the SINGLE territory "NEW YORK".
interface Salesman {
  id: string;
  name: string;
  department: string;
  // no territory field -- it is "NEW YORK" for everyone
}
```

# 8. Omitted Field -- Derivative from Other Data

Territory is derivable from the department, stored in a separate record.

```typescript
interface Salesman {
  id: string;
  name: string;
  departmentId: string; // foreign key
}

interface Department {
  id: string;
  name: string;
  manager: string;
  territory: string;
}

function getTerritory(salesman: Salesman, departments: Map<string, Department>): string {
  const dept = departments.get(salesman.departmentId);
  return dept?.territory ?? "UNKNOWN";
}
```

# 9. Default Flag

A boolean flag distinguishes local vs. remote salesmen. Local salesmen serve
the territory of the file they are in.

```typescript
interface SalesmanRecord {
  id: string;
  name: string;
  department: string;
  isLocal: boolean; // true = territory is this office's territory
}
```

# 10. Multiplexed Field (Conditional Meaning)

A field's meaning depends on another field's value.

```typescript
interface SalesmanRecord {
  id: string;
  name: string;
  department: string;
  isLocal: boolean;
  managerOrTerritory: string; // manager name if isLocal, else territory name
}

function getTerritory(r: SalesmanRecord): string | null {
  return r.isLocal ? null : r.managerOrTerritory;
}

function getManager(r: SalesmanRecord): string | null {
  return r.isLocal ? r.managerOrTerritory : null;
}
```

# 11. Parametric Form (EAV)

Entity-Attribute-Value: the property name appears as data.

```typescript
interface Attribute {
  parameter: string; // e.g., "TERRITORY", "manager", "product"
  value: string;
}

interface SalesmanEav {
  id: string;
  attributes: Attribute[];
}

// Data:
//   { id: "xx", attributes: [
//     { parameter: "TERRITORY", value: "Texas" },
//     { parameter: "manager",   value: "yy"    },
//     { parameter: "product",   value: "toys"  },
//   ]}
```

# 12. Self-Describing Field

A field carries metadata describing its own encoding (length, unit, scale).

```typescript
interface SalesmanRecord {
  id: string;
  name: string;
  department: string;
  territoryLength: number; // byte length of the following field
  territory: string;       // variable-length
}

function readTerritory(r: SalesmanRecord): string {
  return r.territory.slice(0, r.territoryLength);
}
```

# 13. Redundant Records

Multiple forms maintained simultaneously, each optimized for different access
patterns.

```typescript
interface RedundantStore {
  bySalesman: Map<string, string>;       // salesmanId -> territory
  byTerritory: Map<string, string[]>;    // territory -> salesmanId[]
  intersection: Array<{ salesmanId: string; territory: string }>;
}
```

# 14. Reverse Direction (One-to-One)

With one salesman per territory, put the salesman field in the territory record.

```typescript
interface Territory {
  id: string;
  headquarters: string;
  size: number;
  salesmanId: string;
  salesmanName: string;
}
```

# 15. Bidirectional (One-to-One)

Both records carry the foreign key.

```typescript
interface Salesman {
  id: string;
  territoryId: string;
}

interface Territory {
  id: string;
  salesmanId: string;
}
```

# 16. Implicit Identification

The relationship itself serves as the identity (e.g., "Smith's territory").

```typescript
// No explicit field for territory. A salesman /is/ identified by his
// territory, or vice versa.
const smithsTerritory = "MIDWEST";
const newYorksSalesman = "Jones";
```

# 17. Intersection Record (Many-to-Many)

A separate record type for the relationship.

```typescript
interface SalesmanTerritory {
  salesmanId: string;
  territoryId: string;
  quota?: number; // attribute of the relationship
}
```

# 18. Repeating Fields -- Territories in Salesman Record

An array (variable-length) of territories on the salesman.

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  territories: string[]; // variable-length list
}
```

# 19. Repeating Fields -- Salesmen in Territory Record

An array of salesmen on the territory.

```typescript
interface Territory {
  id: string;
  headquarters: string;
  salesmen: string[]; // variable-length list
}
```

# 20. Both Directions (Redundant)

Both sides carry arrays.

```typescript
interface Salesman {
  id: string;
  territoryIds: string[];
}

interface Territory {
  id: string;
  salesmanIds: string[];
}
```

# 21. Fixed Repeating Fields

Known limit (e.g., 3 territories per salesman), fixed slots.

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  territory1: string | null;
  territory2: string | null;
  territory3: string | null;
}
```

# 22. Flag per Territory

One boolean field per known territory value.

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  newYork: boolean;
  california: boolean;
  texas: boolean;
}
// The property names /are/ the data values.
```

# 23. Bit Vector

A single field encodes all territories as positional bits.

```typescript
// Predefined territory index:
const TERRITORY_BITS: Record<string, number> = {
  "NEW_YORK":  0b001,
  "CALIF":     0b010,
  "TEXAS":     0b100,
};

interface Salesman {
  id: string;
  name: string;
  department: string;
  territoryBits: number; // e.g., 0b101 = NEW_YORK + TEXAS
}

function hasTerritory(s: Salesman, t: string): boolean {
  return (s.territoryBits & TERRITORY_BITS[t]) !== 0;
}
```

# 24. Keyword String

A single field contains a concatenation of territory codes.

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  territoryCodes: string; // e.g., "C,T"  (comma-separated codes)
}

function getTerritories(s: Salesman): string[] {
  return s.territoryCodes.split(",").filter(Boolean);
}
```

# 25. Special Territories + "Other" Flag

Separate flags for important territories, plus a generic "other" indicator.

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  newYork: boolean;
  california: boolean;
  hasOtherTerritories: boolean;
}
```

# 26. Special Territories + Intersection for Others

Flags for special territories; a separate table details the "others".

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  newYork: boolean;
  california: boolean;
  hasOtherTerritories: boolean;
}

// Separate record for the "other" territories:
interface OtherTerritory {
  salesmanId: string;
  territory: string;
}
```

# 27. Sentinel Value for Exceptions

A special string value in the territory field marks multi-territory salesmen.

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  territory: string; // "MULTI" for salesmen with more than one
}

// Separate lookup for the actual multiple assignments:
const MULTI_TERRITORIES: Map<string, string[]> = new Map([
  ["zz", ["California", "Texas"]],
]);

function getTerritories(s: Salesman): string[] {
  if (s.territory === "MULTI") {
    return MULTI_TERRITORIES.get(s.id) ?? [];
  }
  return [s.territory];
}
```

# 28. Separate Flag for Exceptions

A boolean field flags multi-territory salesmen, keeping the territory field clean.

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  hasMultipleTerritories: boolean;
  territory: string | null; // null when hasMultipleTerritories is true
}

const MULTI_TERRITORIES: Map<string, string[]> = new Map([
  ["zz", ["California", "Texas"]],
]);
```

# 29. Discriminated Union for Single vs. Multi

Separate record types for salesmen with one vs. many territories.

```typescript
interface SingleTerritorySalesman {
  kind: "single";
  id: string;
  name: string;
  department: string;
  territory: string;
}

interface MultiTerritorySalesman {
  kind: "multi";
  id: string;
  name: string;
  department: string;
  // territories in a separate table
}

type Salesman = SingleTerritorySalesman | MultiTerritorySalesman;

// Separate table only for multi-territory salesmen:
interface MultiTerritoryAssignment {
  salesmanId: string;
  territory: string;
}
```

# 30. Null Values for Optional Fact

Non-salesmen employees have null/undefined territory.

```typescript
interface Employee {
  id: string;
  name: string;
  department: string;
  territory?: string; // undefined for non-salesmen
  role: "driver" | "salesman" | "manager";
}
```

# 31. Separate Records for Specialized Info

Common employee fields are in one record; salesman-specific data in another.

```typescript
interface Employee {
  id: string;
  name: string;
  department: string;
}

interface SalesmanInfo {
  employeeId: string; // foreign key
  territory: string;
}
```

# 32. Separate Record Types per Subtype

Each subtype (driver, salesman) has its own record structure.

```typescript
interface Driver {
  id: string;
  name: string;
  department: string;
  licenseNumber: string;
}

interface Salesman {
  id: string;
  name: string;
  department: string;
  territory: string;
}
```

# 33. Redundant Supertype + Subtype

Both the general employee record and the specialized salesman record exist.

```typescript
interface Employee {
  id: string;
  name: string;
  department: string;
}

interface Salesman {
  id: string; // same id as Employee
  name: string;
  department: string;
  territory: string;
}
```

# 34. Parameterized Connections

Multiple connection types between salesman and territory, stored as tuples.

```typescript
type ConnectionType = "serves" | "home";

interface SalesmanTerritory {
  salesmanId: string;
  connection: ConnectionType;
  territory: string;
}

// Data:
//   { salesmanId: "xx", connection: "serves", territory: "New York" }
//   { salesmanId: "yy", connection: "home",   territory: "California" }
```

# 35. Named Relationship Fields

Separate fields for each kind of connection.

```typescript
interface SalesmanWithRelationships {
  id: string;
  name: string;
  department: string;
  homeTerritory: string;
  servesTerritory: string;
}
// Note: "territory" does not appear as a direct property name.
```

# 36. Default Value Convention

When two values are usually the same, one field is omitted for the common case.

```typescript
interface SalesmanConvention {
  id: string;
  name: string;
  department: string;
  homeTerritory: string;         // always set
  servesTerritory?: string;      // omitted when same as homeTerritory
}

function getServesTerritory(s: SalesmanConvention): string {
  return s.servesTerritory ?? s.homeTerritory;
}
```

# 37. (Open -- More Possibilities)

Combinations of the above. For example: multi-field embedded in identifier,
with fixed repeating fields, plus a bit vector fallback.

```typescript
// Exercise: pick any three options and compose them.
// Kent's original point: the number of possible designs is astronomical
// since each option can be combined with others for every fact.
```

# Key Insight

Kent's central observation, translated to programming:

> The assumption that a simple binary fact always maps to a single property
> on an object is false. A relationship between two entities can be
> represented in dozens of equivalent ways -- embedded in an identifier,
> split across properties, encoded in type/class names, stored as EAV,
> implicit in context, derivable from other data, etc.

When designing APIs, database schemas, or data structures:

- No single representation is "correct" -- each optimizes for different access
  patterns, consistency constraints, and semantic assumptions.
- Dictionary/documentation systems that index by property name will miss facts
  encoded in class names, identifiers, or context.
- Data interchange between systems requires a semantic model independent of
  any one representation, plus explicit mappings between the model and each
  concrete format.
- Entity-relationship or type-level modelling does not yield unique designs --
  the same E-R diagram maps to many implementation structures.

**Practical counterpart:** Evan Czaplicki's "The Life of a File" (Elm Conf 2017)
demonstrates this with concrete Elm code: two checkbox lists that look identical
(settings toggles vs. fruit selection with a limit) need different data models.
The talk walks through multiple Elm type representations for the same UI pattern
-- records, dictionaries, custom types with invariants -- as a hands-on catalog
of design alternatives. See `refactoring-catalog.md` for the full connection.

**See also:** `examined-projects.md` for real-world examples of representation
diversity in open-source projects.
