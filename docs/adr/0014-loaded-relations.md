# ADR 0014: Loaded relations as explicit values with runtime-checked accessors

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from Review Candidate 2.

[ADR index](README.md) · [ORM](../specification/03-models-and-orm.md) · Complements [ADR 0008](0008-no-hidden-network-io.md)

## Context

Candidate 1 removed relation fields from rows but did not say what `.with(...)` returns or how loaded data is read. This is the most direct Django mapping (`select_related`/`prefetch_related`), and the representation decides both safety and compiler-error quality.

## Decision

`.with(...)` changes a query's item type to `Loaded<M>`. It dereferences read-only to the row and exposes generated accessors:

| Relation | Accessor returns |
| --- | --- |
| collection | `Result<&[Loaded<T>], NotLoaded>` |
| single | `Result<&Loaded<T>, NotLoaded>` |
| nullable single | `Result<Option<&Loaded<T>>, NotLoaded>` |

The type does not encode which relations were loaded. `NotLoaded` errors name the missing `.with(...)`. Typed projections (`select_as`) provide compile-time guarantees when needed.

## Consequences and boundaries

There is no hidden I/O, and no nested generic type-state appears in user code or compiler errors. A missing `.with` is a runtime `Result`, which teaches error handling, not a compile error. Loader strategy is deterministic and inspectable.

## Alternatives and rationale

- **Type-state loading:** rejected, because nested generics surface in error messages and are hard for newcomers.
- **Lazy loading on access:** rejected, because it is hidden I/O (ADR 0008).
- **Mutable relation fields on rows:** rejected, because it conflates row state with loaded state.

## Evidence still required

Accessor cost; error quality for nested loads; deterministic loader strategy over SeaORM; query-count assertions.
