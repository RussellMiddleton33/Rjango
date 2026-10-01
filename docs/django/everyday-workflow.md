# Everyday workflow: coming from Django

[Guide index](README.md) · [Operations](../specification/22-application-operations-and-services.md) · [Developer experience](../specification/24-developer-experience-and-diagnostics.md)

**Status:** REVIEW CANDIDATE 2 guide, grounded in the agreed direction. **Evidence:** VALIDATION REQUIRED. Examples are illustrative; commands and APIs are not asserted to exist.

This page maps what a Django developer does every day onto Rjango's Tier 0 path, and states where each analogy stops.

| Django | Rjango (Tier 0) | Where the analogy stops |
| --- | --- | --- |
| View function / APIView | Annotated handler, which is an implicit operation | Every handler declares `policy = ...` or `public`. Django views are open by default; Rjango refuses to start without a declaration. |
| `permission_classes` | Operation policy with `scope` + `check` forms | Policies receive `ctx`, not `request`, so the same policy protects HTTP, Admin, Jobs and MCP. |
| `request.user` | `ctx.actor()`; `CurrentUser` extractor | The actor is a typed enum (anonymous, user, service, API key, agent), and the tenant comes from `ctx`. |
| `Model.objects.filter(...)` | `Venue::objects(&db).filter(...)` with `db = ctx.db()` | The handle is **scoped**: tenant and policy filters apply automatically. Unscoped access is `ctx.system_db(reason)`, which is declared and audited. |
| `get_object_or_404(Venue, pk=id)` | `Venue::objects(&db).get(id).await?` | `NotFound` becomes 404 at the HTTP boundary, including for rows hidden by policy, so existence does not leak. A missing *referenced* ID in input is a 422. |
| `ModelSerializer` | `#[rjango::schema(from = Venue, output, fields(...))]` | Field lists are mandatory and compile-checked. There is no `fields = "__all__"`. |
| `ModelViewSet` | `#[rjango::resource(...)]` | Each action is a scoped operation with a required policy. |
| `transaction.atomic()` | `let mut tx = db.begin().await?; ... tx.commit().await?;` or `db.atomic(async \|tx\| ...)` | **Not ambient.** Only queries that use `tx` are in the transaction. The outer `db` inside the block runs *outside* it (warning `RJG-DB-TX-OUTSIDE`). |
| `transaction.on_commit(lambda: task.delay())` | `tx.dispatch(job).await?` / `tx.emit(event).await?` | Durable: written in the same commit, so it is not lost if the process dies after commit. |
| `post_save` signal | Synchronous model hook (normalization only), or `tx.emit` with a durable listener | Hooks get no database, cache or email handle, and `bulk_update` does not run them. |
| `Model.full_clean()` | Schema validators (sync) plus operation business checks | I/O-based validation lives in the operation, which has `ctx`. |
| `queryset.update(...)` | `bulk_update(...)` | Applies scopes, `auto_now`, versions and audit; no per-row hooks. |
| `manage.py runserver` | `rjango dev` | Rebuilds after each save. The old process keeps serving until the new build is ready, and rebuilds take seconds, not milliseconds. |
| `manage.py shell` | `rjango db shell`, `rjango run scripts/x.rs`, `rjango inspect` | Compiled Rust has no practical REPL, so exploration uses SQL, compiled scripts and metadata queries. |
| `manage.py makemigrations` / `migrate` | `rjango migration make` / `rjango migration run` | Plans show expand/contract deploy ordering, and production never auto-applies. |
| `DEBUG = True` | `RJANGO_ENV=development` (set by `rjango dev`) | An unset environment means **production**. |
| `AUTH_USER_MODEL` | `.auth_user::<User>()` with `UserId` primary key | Reusable apps reference users through the `UserId` contract, not the host's type. |
| Celery task | `#[rjango::job] async fn ...(ctx: JobCtx, input: T)` | The default queue is a PostgreSQL table in the same database, and jobs are at least once. |

## A first endpoint

See the conforming Tier 0 example in [cross-system invariants](../specification/20-cross-system-invariants.md). It introduces six concepts: model, schema, handler, policy, `Result`/`?`, `.await`. The Tier 1 example after it adds a transaction, a durable event and a listener. The next guide, [Rust concepts](rust-concepts.md), explains each Rust idea as it appears.
