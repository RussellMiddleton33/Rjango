# Migrations and schema management

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III, ORM specification.

Desired model state, historical migration state, and actual database state are distinct. Production changes are explicit, reviewable migrations; model declaration never silently changes production schema.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Phase/step execution, recovery and stable IR

Migration-wide atomic=false is superseded as the complete execution model. A versioned stable migration IR records historical schema types, structured operations, dependencies, explicit renames, pre/post database fingerprints and backend capabilities. Replay must not depend on current application models. Custom SQL/data callbacks declare compatibility/reversal/recovery limits; arbitrary Rust code has no automatic historical compatibility.

Each phase/step declares transaction Required, Forbidden or Allowed; Allowed prefers a transaction where supported. Planning rejects incompatible grouping. Mixed transactional/nontransactional phases do not promise whole-migration rollback. Large backfills use bounded batches and durable checkpoints.

The ledger records migration/phase/step identity, artifact/IR checksum, started/completed/failed/unknown outcome, checkpoints and sanitized diagnostics. Record transactional completion atomically with its effects when possible. A crash between external DDL and ledger update requires postcondition introspection, never blind replay. Partial/invalid objects need a reviewed cleanup/resume path. Dependencies advance only after required steps are verified. Resume distinguishes retry-safe, reconciliation-required and operator-required states; destructive repair is separately authorized.

One logical migrator holds a database/graph-scoped lock across all phases, including nontransactional steps. Advisory locking is a candidate; exact fencing/session-loss behavior needs validation. Lock loss stops progress; the successor reconciles unknown steps. Acquisition has bounded wait. Applied checksum mismatches fail closed; edited history never silently replaces applied history. Repair records actor/reason/actual state. IR upgrades preserve meaning with explicit checksum-version handling; unsupported IR fails before mutation.


## Open decisions and interpretation

Migration file representation, exact commands, opaque field identities, rename UX, and detailed backend support remain open. A safety label is a review aid, not proof an operation is harmless.



<!-- Source: iii section 1. -->
## Migrations & Schema Management

### Goal

Rjango migrations must make database evolution:

- deterministic
- reviewable
- safe
- reproducible
- source-controlled
- branch-friendly
- AI-readable
- human-readable
- recoverable when possible

The fundamental rule is:

> Models describe desired state. Migrations describe how state changes over time.

The model definition is **not** itself a production migration.

---

### Three Database Representations

Rjango distinguishes:

```
Desired Schema
     │
     │ generated from current AMG
     ▼
Current Model State

Historical Schema
     │
     │ reconstructed from migrations
     ▼
Migration State

Actual Schema
     │
     │ introspected from PostgreSQL
     ▼
Database State
```

Normally:

```
Model State
     =
Migration State
     =
Database State
```

If not, Rjango can identify exactly what differs.

---

### Migration Graph

Migrations form a dependency DAG.

Not merely:

```
0001
 ↓
0002
 ↓
0003
```

But potentially:

```
        ┌── 0042_accounts
0041 ───┤
        └── 0042_venues
                │
                ▼
            0043_merge
```

Each migration has:

```
pub struct MigrationDescriptor {
    pub id: MigrationId,
    pub app: AppId,
    pub dependencies: Vec<MigrationId>,
    pub operations: Vec<MigrationOperation>,
    pub before_fingerprint: SchemaFingerprint,
    pub after_fingerprint: SchemaFingerprint,
}
```

Migration identity must not depend solely on sequential numbers.

---

### Migration Operations

Rjango has structured operations rather than treating migration files as arbitrary SQL.

Examples:

```
CreateTable
DropTable

RenameTable

AddColumn
DropColumn
RenameColumn
AlterColumn

CreateIndex
DropIndex

AddConstraint
DropConstraint

AddForeignKey
DropForeignKey

CreateEnum
AlterEnum
DropEnum

RunData
RunSql
Custom
```

Structured operations enable:

- safety classification
- automatic reversal
- SQL generation
- database portability
- documentation
- MCP understanding
- migration optimization
- impact analysis

---

<!-- Source: iii section 2. -->
## Migration Safety Classification

Every operation receives a safety classification.

### SAFE

Examples:

```
Add nullable column
Add index concurrently
Increase varchar length
Create table
```

### REVIEW

Examples:

```
Make nullable column required
Change numeric precision
Add unique constraint to existing data
Change enum
Rename objects
```

### DESTRUCTIVE

Examples:

```
Drop column
Drop table
Reduce maximum length
Potentially lossy type conversion
```

### DATA-SENSITIVE

Example:

```
Add NOT NULL column to existing table
```

whether safe depends on existing data/default strategy.

Rjango should never pretend binary safe/unsafe classification is sufficient.

---

<!-- Source: iii section 3. -->
## Migration Plan

Before applying migrations:

```
rjango migration plan
```

conceptually presents:

```
Migration: venues.0042_add_description

SAFE
+ Add nullable TEXT column venues.description

REVIEW
~ Add index venues.slug_idx

Estimated table:
3.8 million rows

Potential concern:
index creation may block writes unless created concurrently
```

Later, database statistics can enrich this.

---

<!-- Source: iii section 4. -->
## Rename Detection

Rjango must never blindly infer destructive renames.

Changing:

```
pub name: String
```

to:

```
pub title: String
```

could mean:

```
rename name → title
```

or:

```
drop name
add title
```

Those are radically different.

Rjango should require explicit intent:

```
#[renamed_from = "name"]
pub title: String,
```

Then migration metadata records the identity transition.

---

<!-- Source: iii section 5. -->
## Stable Field Identity

Longer term, we should consider an internal stable identifier independent from the current display name.

Conceptually:

```
field UUID:
f_729ac...

historical name:
name

current name:
title
```

That would make schema evolution dramatically safer.

However:

**STATUS: OPEN**

We should not expose UUID-style metadata identifiers publicly until we prove they provide enough value to justify the complexity.

Human-readable AMG IDs remain the initial design.

---

<!-- Source: iii section 6. -->
## Atomicity

PostgreSQL migrations should default to transactional execution whenever PostgreSQL permits it.

Conceptually:

```
BEGIN

operation
operation
operation

COMMIT
```

Failure:

```
ROLLBACK
```

Explicit non-atomic migration:

```
#[rjango::migration(atomic = false)]
```

should be required for operations incompatible with transaction boundaries or intentionally batched large-data migrations.

---

<!-- Source: iii section 7. -->
## Data Migrations

Schema and data migration are separate concepts.

Example:

```
#[rjango::data_migration]
async fn populate_slugs(ctx: MigrationContext) -> Result<()> {
    ...
}
```

Historical data migrations must operate against a **historical schema representation**, not accidentally assume today's model.

This is a crucial lesson from mature migration systems.

---

<!-- Source: iii section 8. -->
## Historical Models

A migration created in 2027 must still function in 2032 even if:

```
Venue
```

has changed enormously.

Therefore migration code cannot simply import the current application model and assume its shape matches historical data.

Rjango needs versioned migration-facing schema information.

Conceptually:

```
let venues = ctx.model("venues.Venue")?;
```

where `ctx` understands the historical schema at that migration boundary.

---

<!-- Source: iii section 9. -->
## Raw SQL

Must be possible:

```
MigrationOperation::sql(
    "...",
    Some("reverse SQL...")
)
```

But raw SQL receives:

```
portability = unknown
automatic_reverse = false/explicit
optimizer_visibility = limited
AI_safety_analysis = limited
```

Rjango should warn, not prohibit.

---

<!-- Source: iii section 10. -->
## Reversibility

Migration status:

```
Reversible
ConditionallyReversible
Irreversible
```

Example:

```
ADD COLUMN
→ generally reversible

DROP COLUMN
→ schema operation reversible,
   lost data is not

arbitrary RunSql
→ depends on explicit reverse
```

`rjango migration rollback` must refuse pretending an irreversible migration can restore destroyed information.

---

<!-- Source: iii section 11. -->
## Migration Squashing

Long-lived projects may eventually have hundreds or thousands of migrations.

Rjango should support migration squashing.

Concept:

```
0001 create venue
0002 add slug
0003 add description
0004 rename x
0005 add index
```

can potentially become:

```
0001_squashed_0005
```

while preserving compatibility for installations that already ran the originals.

The squasher operates on structured migration operations.

Example optimization:

```
CreateTable
+
AddColumn

→

CreateTable including column
```

---

<!-- Source: iii section 12. -->
## Migration Branching

Parallel development will create sibling migrations.

Rjango should detect:

```
migration graph has two heads
```

and distinguish:

```
independent compatible changes
```

from:

```
conflicting changes
```

Potential resolution is a merge migration:

```
depends_on:
    venues.0042_a
    venues.0042_b
```

No arbitrary renumbering should be necessary.

---

<!-- Source: iii section 13. -->
## Drift Detection

Three-way verification:

```
AMG model state
     │
     ├───────────┐
     ▼           ▼
migration      actual
state          database
```

Command concept:

```
rjango schema verify
```

Possible output:

```
✓ Models match migration state
✓ Migration history matches database

Database drift detected:

+ unexpected index:
  venues_manual_idx

~ column:
  users.email
  expected: VARCHAR(254)
  actual:   TEXT
```

This is especially valuable in production.

---

<!-- Source: iii section 14. -->
## Development Schema Sync

Rjango may provide:

```
rjango db sync --development
```

for experimentation.

It must be loudly classified as:

```
DEVELOPMENT TOOL
```

and not become the recommended production workflow.

---

<!-- Source: iii section 15. -->
## Database Lock Awareness

Migration planning should eventually understand PostgreSQL lock consequences.

Example:

```
ALTER TABLE
```

may have radically different production implications depending on operation and table size.

Rjango's long-term migration analyzer should report:

```
possible ACCESS EXCLUSIVE lock
```

when deterministically knowable.

---

<!-- Source: iii section 16. -->
## Migration Metadata + MCP

MCP:

```
inspect_migrations
inspect_pending_changes
explain_migration
analyze_migration_risk
```

Generation permission:

```
migration.generate
```

Execution permission:

```
migration.apply
```

must be completely separate.

An agent allowed to inspect or generate migrations is **not automatically authorized to alter a database**.

---

<!-- Source: iii section 17. -->
## Migration Testing Standard

Mandatory coverage includes:

- every schema operation
- forward migration
- reverse migration
- transactional failure
- non-atomic failure
- graph branching
- graph merging
- rename handling
- drift
- squashing
- historical models
- data migrations
- raw SQL
- enum evolution
- constraint failures
- huge graph behavior
- corrupted history
- interrupted migrations

Property tests should create random schema graphs and test:

```
A → B → C
```

for deterministic diffs.

---

<!-- Source: iii section 18. -->
## Django Mapping — Migrations

```
Django                    Rjango

makemigrations            rjango migration make

migrate                   rjango migration run

showmigrations             rjango migration list

squashmigrations           rjango migration squash

RunPython                  data migration

RunSQL                     raw SQL operation

migration dependencies     migration DAG
```

Rjango adds first-class AMG fingerprints, drift analysis, structured safety classification and agent-facing introspection.

---

<!-- Source: iii section 19. -->
## Migration Decisions

### LOCK

- migrations source-controlled
- dependency DAG
- explicit production migration files
- PostgreSQL transactional by default
- explicit data migrations
- explicit rename intent
- structured operations
- safety classification
- schema drift detection
- AMG fingerprints

### OPEN

- exact migration source format
- whether migrations compile as Rust
- stable opaque field identity
- migration optimizer sophistication
- online migration automation

<!-- Source: orm section 40. -->
## Migrations

Rjango's migration design should be based on:

```
Previous known schema
       VS
Current AMG model graph
       ↓
Graph Diff
       ↓
Migration Operations
       ↓
Explicit Migration Artifact
```

Not:

```
server starts
↓
database magically changes
```

---

<!-- Source: orm section 41. -->
## Migration Command

```
rjango migration make
```

Output:

```
Detected model changes:

+ Venue.description
+ Venue.active
~ Venue.name max_length 100 → 200

Migration:
0042_update_venue

Created:
migrations/0042_update_venue.rs
```

---

<!-- Source: orm section 42. -->
## Migration Plan

```
rjango migration plan
```

Example:

```
0042_update_venue

SAFE
 + Add nullable column venue.description

SAFE
 + Add venue.active with default true

REVIEW
 ~ Expand venue.name VARCHAR(100) → VARCHAR(200)
```

---

<!-- Source: orm section 43. -->
## Destructive Migration Safety

Example:

```
DESTRUCTIVE

- venues.legacy_id
```

CLI should refuse accidental unattended execution under configured production policy.

AI generation never bypasses this classification.

---

<!-- Source: orm section 44. -->
## Migration Files

Migration artifacts are source-controlled.

Each migration contains:

```
ID
parent migration(s)
AMG fingerprint before
AMG fingerprint after
operations
reverse operations where available
creation metadata
```

This allows Rjango to detect schema drift.

---

<!-- Source: orm section 45. -->
## Migration Determinism

Given:

```
same previous AMG
same current AMG
```

Rjango must generate:

```
same semantic migration plan
```

every time.

This requires canonical graph sorting and stable identifiers.

---

<!-- Source: orm section 46. -->
## Rename Detection

Renames are dangerous.

If:

```
name
```

disappears and:

```
display_name
```

appears, Rjango must **not** automatically assume a rename.

CLI:

```
Possible rename detected:

venues.Venue.name
    →
venues.Venue.display_name

[R] rename
[A] add + remove
[C] cancel
```

For CI/noninteractive generation, explicit source annotation should be required:

```
#[renamed_from = "name"]
pub display_name: String,
```

---

<!-- Source: orm section 47. -->
## Schema Drift

Command:

```
rjango migration verify
```

Compare:

```
migration state
database schema
AMG
```

Possible:

```
✓ migrations match application
✓ database migration history matches

⚠ database schema differs from expected migration state
```

This should become another signature Rjango feature.

---

<!-- Source: orm section 48. -->
## Development Schema Sync

Automatic synchronization can potentially be offered only as explicit development tooling:

```
rjango db sync --dev
```

But it must never be mistaken for the production migration workflow.

Default recommended flow remains:

```
change model
migration make
review
migration run
```
