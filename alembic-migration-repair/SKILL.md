---
name: alembic-migration-repair
description: Fix migration-numbering collisions after a git merge — two branches added alembic migrations with the same revision IDs, so `alembic upgrade head` dies with "relation ... does not exist" or duplicate-revision errors, and the local alembic_version row no longer means what alembic thinks. Trigger on "duplicate revision", "migration mismatch", "upgrade head fails after pull", "renamed the migration files", "relation does not exist" during upgrade, or stamping questions. Covers the renumber rule (mine move behind main's), the local DB repair with alembic stamp, and the pass-stub trap to never use.
---

# Alembic migration repair after a merge collision

## The failure shape

Two branches each add a migration numbered `128`/`129`. Git keeps both files
(no content conflict — different filenames), so `revision = "128"` exists
twice. The local DB's `alembic_version` row reads `128`, but it was written
by OUR old migration; alembic now reads it as MAIN's `128`. Consequence:
alembic skips main's 128 (never creates its tables) and the next migration
dies on a missing FK target or table — `relation "..." does not exist`.

Key fact: alembic tracks the `revision` ID string, never the filename. A
file rename without an ID change repairs nothing.

## Rule one: main's sequence is the spine

Every other machine replays the chain that main publishes. OUR migrations
move behind main's newest; main's numbers are never renumbered. End state:
one linear chain, one head (`alembic heads`).

Renaming a migration touches three things in one edit, DDL unchanged:

1. filename (`131_<name>.py`)
2. `revision = "131"` and `down_revision = "130"` inside
3. the docstring's `Revision ID:` / `Revises:` lines

Git then records a pure rename (high similarity %). Check no test pins the
old numbers before committing (`grep 'revision = "' tests/`).

## Rule two: repair the local DB with bookkeeping only

Never edit migration bodies to dodge a local error (see the trap below).
Instead:

1. **Verify physically first.** Query `information_schema` / `pg_indexes` /
   `pg_constraint` for every object each migration creates. Record which
   revisions' effects are truly present, which are missing, which partial.
   A stamp believes; it never checks — only stamp what you verified.
2. `alembic stamp <last revision whose effects are all present>`
3. `alembic upgrade <up to the last missing one>` — alembic then emits the
   exact DDL every other machine gets (RLS policies, partial indexes,
   named constraints — hand-written psql drifts from these).
4. `alembic stamp head` for the tail whose effects are already present.
5. `alembic upgrade head` must now be a silent no-op; `alembic current`
   reads `NNN (head)`.

## The trap: pass stubs in migration bodies

Do not comment out or `pass`-stub the operations "just locally, remove
before push". One forgotten restore and every other machine's
`upgrade head` silently creates nothing — and the e2e harness migrates a
fresh template DB from these files, so it breaks there first. Real DDL in
the files, always; the local skip lives in `alembic_version` (stamp), not
in the source.

## Checklist

- [ ] Chain linear, `alembic heads` = single head, main's files untouched
- [ ] Renamed files: filename + `revision`/`down_revision` + docstring IDs
- [ ] DDL byte-identical to before the rename (git shows rename, not rewrite)
- [ ] Local schema verified object-by-object BEFORE any stamp
- [ ] `alembic upgrade head` = no-op locally; files contain real DDL
