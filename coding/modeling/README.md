# modeling/

Guides for the Model phase of the Model -> Build -> Evaluate -> Reflect
workflow. Covers decomposition, type design, invariants, and representation
choices.

| File | Purpose | Read first |
|------|---------|------------|
| [process.md](process.md) | The modeling workflow: decomposition, types, operations, entities | Yes |
| [invariants.md](invariants.md) | Categories of invariants and encoding strategy | Yes |
| [type-precision.md](type-precision.md) | Choosing precise types (ADTs, branded types, newtypes) over primitives | When defining types |
| [optionality.md](optionality.md) | Modeling absent/optional values (Option, null, sentinel, etc.) | When modeling optional fields |
| [state-machine.md](state-machine.md) | Modeling state transitions as sum types | When entities have state |
| [operation-invariants.md](operation-invariants.md) | Declaring invariants for each operation (empty, absent, boundary, partial failure) | When defining operations |
| [many-forms.md](many-forms.md) | Kent's catalog of 37+ representations for a single fact (JS/TS examples) | Reference for data representation alternatives |
| [modeling-communication.md](modeling-communication.md) | How to communicate modeling decisions | Always |
