# PROJ-101 — progress

One line per sub-ticket. Keep this current: a long run gets compacted, and this
file is what survives it.

| # | Ticket | Branch | Base | PR | State |
| --- | --- | --- | --- | --- | --- |
| 1 | PROJ-102 | `proj-102-batch-index` | `develop` | #813 | green, awaiting review |
| 2 | PROJ-103 | `proj-103-revocation-job` | `proj-102-batch-index` | #814 | green, awaiting review |
| 3 | PROJ-104 | `proj-104-batch-detail-ui` | `proj-103-revocation-job` | — | in progress: polling spec failing |

**Merge order:** bottom-up — #813, then #814, then #815.

## Notes to carry across a compaction

- PROJ-102: index migration needs the concurrent path on the target engine; called
  out in the PR body for the reviewer to confirm against the deployment runbook.
- PROJ-104: `BatchDetail.spec.ts` fake timers don't advance the 2s poll — switching
  to `vi.advanceTimersByTimeAsync`.
