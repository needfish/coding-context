# The Many Forms of a Single Fact

A JavaScript/TypeScript code parallel to William Kent's "The Many Forms of a Single Fact" (1989). Each option shows a different way to model the fact that a SALESMAN serves a TERRITORY.

> Reference: William Kent, "The Many Forms of a Single Fact", Proc. IEEE COMPCON, Feb. 27 - Mar. 3, 1989, San Francisco. Also HPL-SAL-88-8.

## 1. Direct Field

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  territory: string;
}
```

## 2. Unnormalized (Fact in a Different Record)

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

## 3. Multi-Field Representation

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  country: string;
  region: string;
  district: string;
}
```

## 4. Embedded Data in Identifier

```typescript
// Salesman ID format: "SLS-{TERRITORY_CODE}-{SEQUENTIAL}"
// Example: "SLS-NY-042"
type SalesmanId = string;

interface Salesman {
  id: SalesmanId;
  name: string;
  department: string;
}

function extractTerritory(salesman: Salesman): string {
  return salesman.id.split("-")[1];
}
```

## 5. Shared (Multiplexed) Field

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

## 6. Omitted Field -- Data in Record Type Name

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

## 7. Omitted Field -- Implicit from Context

```typescript
// All records in this file are for the single territory "NEW YORK".
interface Salesman {
  id: string;
  name: string;
  department: string;
}
```

## 8. Omitted Field -- Derivative from Other Data

```typescript
interface Salesman {
  id: string;
  name: string;
  departmentId: string;
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

## 9. Default Flag

```typescript
interface SalesmanRecord {
  id: string;
  name: string;
  department: string;
  isLocal: boolean;
}
```

## 10. Multiplexed Field (Conditional Meaning)

```typescript
interface SalesmanRecord {
  id: string;
  name: string;
  department: string;
  isLocal: boolean;
  managerOrTerritory: string;
}
function getTerritory(r: SalesmanRecord): string | null {
  return r.isLocal ? null : r.managerOrTerritory;
}
function getManager(r: SalesmanRecord): string | null {
  return r.isLocal ? r.managerOrTerritory : null;
}
```

## 11. Parametric Form (EAV)

```typescript
interface Attribute {
  parameter: string;
  value: string;
}
interface SalesmanEav {
  id: string;
  attributes: Attribute[];
}
```

## 12. Self-Describing Field

```typescript
interface SalesmanRecord {
  id: string;
  name: string;
  department: string;
  territoryLength: number;
  territory: string;
}
function readTerritory(r: SalesmanRecord): string {
  return r.territory.slice(0, r.territoryLength);
}
```

## 13. Redundant Records

```typescript
interface RedundantStore {
  bySalesman: Map<string, string>;
  byTerritory: Map<string, string[]>;
  intersection: Array<{ salesmanId: string; territory: string }>;
}
```

## 14. Reverse Direction (One-to-One)

```typescript
interface Territory {
  id: string;
  headquarters: string;
  size: number;
  salesmanId: string;
  salesmanName: string;
}
```

## 15. Bidirectional (One-to-One)

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

## 16. Implicit Identification

```typescript
const smithsTerritory = "MIDWEST";
const newYorksSalesman = "Jones";
```

## 17. Intersection Record (Many-to-Many)

```typescript
interface SalesmanTerritory {
  salesmanId: string;
  territoryId: string;
  quota?: number;
}
```

## 18. Repeating Fields -- Territories in Salesman Record

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  territories: string[];
}
```

## 19. Repeating Fields -- Salesmen in Territory Record

```typescript
interface Territory {
  id: string;
  headquarters: string;
  salesmen: string[];
}
```

## 20. Both Directions (Redundant)

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

## 21. Fixed Repeating Fields

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

## 22. Flag per Territory

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  newYork: boolean;
  california: boolean;
  texas: boolean;
}
```

## 23. Bit Vector

```typescript
const TERRITORY_BITS: Record<string, number> = {
  "NEW_YORK":  0b001,
  "CALIF":     0b010,
  "TEXAS":     0b100,
};
interface Salesman {
  id: string;
  name: string;
  department: string;
  territoryBits: number;
}
function hasTerritory(s: Salesman, t: string): boolean {
  return (s.territoryBits & TERRITORY_BITS[t]) !== 0;
}
```

## 24. Keyword String

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  territoryCodes: string;
}
function getTerritories(s: Salesman): string[] {
  return s.territoryCodes.split(",").filter(Boolean);
}
```

## 25. Special Territories + "Other" Flag

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

## 26. Special Territories + Intersection for Others

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  newYork: boolean;
  california: boolean;
  hasOtherTerritories: boolean;
}
interface OtherTerritory {
  salesmanId: string;
  territory: string;
}
```

## 27. Sentinel Value for Exceptions

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  territory: string; // "MULTI" for multi-territory salesmen
}
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

## 28. Separate Flag for Exceptions

```typescript
interface Salesman {
  id: string;
  name: string;
  department: string;
  hasMultipleTerritories: boolean;
  territory: string | null;
}
const MULTI_TERRITORIES: Map<string, string[]> = new Map([
  ["zz", ["California", "Texas"]],
]);
```

## 29. Discriminated Union for Single vs. Multi

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
}
type Salesman = SingleTerritorySalesman | MultiTerritorySalesman;
interface MultiTerritoryAssignment {
  salesmanId: string;
  territory: string;
}
```

## 30. Null Values for Optional Fact

```typescript
interface Employee {
  id: string;
  name: string;
  department: string;
  territory?: string;
  role: "driver" | "salesman" | "manager";
}
```

## 31. Separate Records for Specialized Info

```typescript
interface Employee {
  id: string;
  name: string;
  department: string;
}
interface SalesmanInfo {
  employeeId: string;
  territory: string;
}
```

## 32. Separate Record Types per Subtype

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

## 33. Redundant Supertype + Subtype

```typescript
interface Employee {
  id: string;
  name: string;
  department: string;
}
interface Salesman {
  id: string;
  name: string;
  department: string;
  territory: string;
}
```

## 34. Parameterized Connections

```typescript
type ConnectionType = "serves" | "home";
interface SalesmanTerritory {
  salesmanId: string;
  connection: ConnectionType;
  territory: string;
}
```

## 35. Named Relationship Fields

```typescript
interface SalesmanWithRelationships {
  id: string;
  name: string;
  department: string;
  homeTerritory: string;
  servesTerritory: string;
}
```

## 36. Default Value Convention

```typescript
interface SalesmanConvention {
  id: string;
  name: string;
  department: string;
  homeTerritory: string;
  servesTerritory?: string;
}
function getServesTerritory(s: SalesmanConvention): string {
  return s.servesTerritory ?? s.homeTerritory;
}
```

## 37. (Open -- More Possibilities)

Combinations of the above. For example: multi-field embedded in identifier, with fixed repeating fields, plus a bit vector fallback.

## Key Insight

Kent's central observation: a simple binary fact does not always map to a single property on an object. A relationship between two entities can be represented in dozens of equivalent ways -- embedded in an identifier, split across properties, encoded in type/class names, stored as EAV, implicit in context, derivable from other data.

When designing APIs, database schemas, or data structures:
- No single representation is correct. Each optimizes for different access patterns, consistency constraints, and semantic assumptions.
- Data interchange between systems requires a semantic model independent of any one representation, plus explicit mappings between the model and each concrete format.
- Entity-relationship or type-level modelling does not yield unique designs -- the same E-R diagram maps to many implementation structures.

**Practical counterpart:** Evan Czaplicki's "The Life of a File" (Elm Conf 2017) demonstrates this with concrete Elm code: two checkbox lists that look identical (settings toggles vs. fruit selection with a limit) need different data models. See `refactoring-catalog.md` for the full connection.

**See also:** `examined-projects.md` for real-world examples.
