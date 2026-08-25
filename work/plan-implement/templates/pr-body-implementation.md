<!--
The implementation PR body. Fill the repo's own PR template first (labels,
checkboxes, ticket link), then add the "Implementation decisions" section — that
section is what buys the agent the right to decide without asking.
-->

## Summary

Adds `POST /batches/{reference}/revoke` and the queued `RevokeBatchJob` behind it.
Revocation goes through the existing `RevokeCredential` action row by row, so
notifications, audit entries and registry writes are unchanged.

Stacked on #813 (PROJ-102) — review that first. Base is `proj-102-batch-index`,
not `develop`; GitHub retargets automatically once #813 merges.

Ticket: [PROJ-103](https://acme.atlassian.net/browse/PROJ-103)
Plan: `docs/plans/PROJ-103-batch-revocation-job.md`
Decision note: #812

## Implementation decisions

Decisions the plan didn't cover, made without stopping to ask:

- **Job returns counts in its final state rather than firing an event.**
  Alternative was a `BatchRevocationCompleted` event; nothing listens for it yet
  and PROJ-104 polls the job state anyway. Add the event when a second consumer
  exists.
- **Iterated with a keyset cursor rather than nested chunk callbacks.** Both work
  at 200; the cursor keeps the failure-and-continue logic in one loop instead of a
  closure per chunk. No behavioural difference.
- **Malformed reference returns 422 before touching the repository.** The plan
  said "validates format only" but not what happens first; validating early keeps
  a bad string out of a 2.1M-row query.
- **Skipped rows are logged at `warning`, not `error`.** An already-revoked row is
  an expected outcome of a retry, not a fault. Errors here would page someone.

## Testing

- `RevokeBatchJobTest` — happy path, failing row, already-revoked row, empty batch
- `BatchRevocationTest` — 202 with job id, 404 unknown, 404 cross-organisation,
  422 malformed
- Full suite green; format, static analysis and architecture checks clean
