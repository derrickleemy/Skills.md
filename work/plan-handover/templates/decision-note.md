# Bulk revocation by batch reference

**Status:** planned, awaiting implementation. Tracked in [PROJ-101](https://acme.atlassian.net/browse/PROJ-101).

## Context

An admin who issues credentials in bulk — a school uploading a 400-row CSV at the
end of term — has no way to undo it. Revocation is one credential at a time
through the detail page. Support has handled four of these by running a database
script, and the last one revoked 12 rows belonging to a different school.

Issuance already records a `batch_reference` on every row it creates. Nothing
reads it.

## Decision

Revoke by batch reference, through the existing single-credential revocation path,
one row at a time inside a queued job. The API takes a batch reference and returns
a job id; the UI polls it.

We are not adding a bulk-revocation code path. The single-credential path already
handles the recipient notification, the audit entry and the registry write, and
those three are the parts that must not diverge.

## Alternatives

**One `UPDATE ... WHERE batch_reference = ?`.** Fast, and wrong: it skips the
notification, the audit entry and the registry write. The support script that
caused the cross-school incident was exactly this.

**Synchronous revocation in the request.** Fine at 50 rows, times out at 400. The
largest batch in production is 1,340.

**A new `Batch` model owning its rows.** The right long-term shape, and about
three weeks of backfill for 2.1M existing rows. Deferred — `batch_reference` as a
plain indexed column carries this feature, and the migration path to a real model
stays open.

## Deliberately not doing

- **Partial revocation** (revoke 300 of 400). Needs a row-selection UI; no one has
  asked. If it lands later it reuses the same job with an id list.
- **Undo.** Revocation is already terminal for single credentials; making it
  reversible for batches only would be a worse inconsistency than not having it.

## What we found out

- `batch_reference` is nullable and **not indexed**. Everything upstream assumed it
  was. The query plan on 2.1M rows is a full scan — the index is phase 1, and it is
  the reason this splits into three PRs rather than one.
- Credentials issued before v4.2 have no batch reference at all (~340k rows). The
  UI must say "this credential was not issued as part of a batch" rather than
  showing an empty batch.
- The registry write is already idempotent, so a retried job can't double-revoke.
  This removes the need for the job-level locking the first sketch of this had.

## Related

- [PROJ-101](https://acme.atlassian.net/browse/PROJ-101)
- Builds on `2026-02-11-credential-revocation-audit-trail.md` — the audit entry
  shape this reuses.
- Codebase facts were verified by direct read on `develop` at commit `a3f91c2`;
  line numbers will drift.
