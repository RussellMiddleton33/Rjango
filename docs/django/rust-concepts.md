# Rust concepts through Rjango: coming from Django

[Guide index](README.md) · [Developer experience and diagnostics](../specification/24-developer-experience-and-diagnostics.md)

**Status:** REVIEW CANDIDATE 2 guide outline. **Evidence:** VALIDATION REQUIRED. Curriculum order is PROPOSED.

Rjango teaches Rust in the order a Django developer needs it. Each step pairs a Python habit with the Rust concept, the Rjango API where it first appears, and the compiler error you are most likely to meet.

| Order | Python habit | Rust concept | First appears in | Typical first error and fix |
| --- | --- | --- | --- | --- |
| 1 | Exceptions | `Result`, `?` | Every handler returns `rjango::Result<T>` | "the `?` operator can only be used in a function that returns `Result`": return `rjango::Result<T>`. |
| 2 | `None` checks | `Option` vs `Result` | `find(pk)` vs `get(pk)`; `first()` vs `require()` | "expected `Venue`, found `Option<Venue>`": choose `get` or handle `None`. |
| 3 | `await` (asyncio) | `async`/`.await`, futures are lazy | Every query terminal | "unused implementer of `Future` that must be used": add `.await`. |
| 4 | Mutating objects freely | Moves and consumption | `venue.edit().save(&db)`, `tx.commit()` | "borrow of moved value": use the value returned by `save`, or do work before `commit`. |
| 5 | Sharing a DB connection implicitly | Exclusive borrow `&mut` | `Venue::objects(&mut tx)` | "cannot borrow `tx` as mutable more than once": run the two queries one after the other. One connection does one thing at a time. |
| 6 | Enums as strings | `enum`, exhaustive `match` | `Patch<T>`, `Decision`, commit outcome | "non-exhaustive patterns": handle each variant. |
| 7 | Duck typing | Traits and bounds | Field types, response types | Rjango messages say which trait to derive or implement. |
| 8 | Threads/GIL | `Send`, `Arc`, shared state | Handlers on the multi-thread runtime; `.state(T)` | "future cannot be sent between threads safely": don't hold a `std::sync::MutexGuard` or `Rc` across `.await`. |
| 9 | Dynamic attributes | Generated items and macros | `Venue::F`, `Venue::R`, `NewVenue` | Use `rjango expand Venue` or rustdoc to see generated items. |
| 10 | — | Lifetimes | Tier 2 APIs only | Tier 0 code should never need lifetime annotations. If it does, that is a framework diagnostics bug to report. |

Each row links, when written, to a tutorial step, a `rjango explain` code and a compile-fail snapshot verified in CI.
