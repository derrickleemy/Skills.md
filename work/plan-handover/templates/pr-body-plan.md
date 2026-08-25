## What

Records the design for bulk credential revocation by batch reference.
Documentation only, no code changes.

## Why

Admins who issue credentials in bulk can't undo it — revocation is one credential
at a time. Support has done four of these by hand with a database script, and the
last one revoked rows belonging to the wrong school.

## The decision

Revoke by batch reference through the **existing single-credential path**, one row
at a time in a queued job. The obvious alternative — a single bulk `UPDATE` — is
what the support script did, and it skips the recipient notification, the audit
entry and the registry write. Those three must not diverge.

## Worth reviewers' attention

- **`batch_reference` is not indexed.** Everything upstream assumed it was. That's
  why this is three PRs and not one.
- **A real `Batch` model is the right long-term shape and is deferred** — ~3 weeks
  of backfill across 2.1M rows. Argue with this if you disagree; the migration path
  stays open either way.
- **~340k pre-v4.2 credentials have no batch reference.** Handled with an explicit
  empty state, not by backfilling.
- No partial revocation and no undo — reasoning in the note.

## Related

- [PROJ-101](https://acme.atlassian.net/browse/PROJ-101)
- Builds on the credential revocation audit trail note.
