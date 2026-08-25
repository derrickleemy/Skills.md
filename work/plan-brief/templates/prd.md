<!--
  The house PRD template. /plan-brief walks these headings in order and asks the
  italic question under each one, so this file *is* the interview — change a
  section here and the interview changes with it.

  Each section carries two things that do NOT appear in a finished PRD:
    - the *italic question* the section answers — this is what gets asked
    - an `Example:` block showing an answer that would pass the self-check
  Delete both when writing the real thing.

  Three conventions:

  1. `**Needed by:**` is the only line above `## Problem`, and it is optional.
     Most features have no real date; omit the line rather than guess one. When
     there is one it carries the *why*, because that is the half Jira's due-date
     field can't hold. A soft date is not a date — "would be nice before the
     holidays" belongs in Constraints.

  2. `## Done when` replaces the metrics table on purpose. We don't have the
     instrumentation to make baseline→target rows honest, and an invented
     baseline is worse than none. These are observable checks instead: two people
     reading them reach the same pass/fail conclusion. A number that genuinely
     exists belongs here as a check, not as a target.

  3. Anything not sourced is labelled `**Assumption:**` inline, wherever it
     appears. No separate assumptions section.

  What this template deliberately does NOT carry: hypothesis, metrics,
  alternatives, trade-offs, risks, rollout, NFRs, decision log. Alternatives and
  trade-offs live in the decision note that /plan-handover writes; the rest live
  in the implementation plans. Nothing appears in both documents.
-->

# <Feature, in the words someone would use on a board>

**Needed by:** <date or quarter> — <what makes that date real>

## Problem

*What can't be done today, who hits it, and why does it matter now?*

> **Example:** An admin who issues credentials in bulk — a school uploading a
> 400-row CSV at the end of term — has no way to undo it. Revocation is one
> credential at a time through the detail page. It matters now because term-end
> issuance is the heaviest window of the year and the next one is in October.

## Evidence

*How do we know — tickets, support threads, a conversation, a number? What has
been tried or worked around already?*

> **Example:** Four support escalations since March (SUP-2201, 2288, 2410, 2455),
> all resolved by running a database script by hand. The last one revoked 12 rows
> belonging to a different school. Two schools have asked for it directly in
> renewal calls (notes in PROJ-88). **Assumption:** the other high-volume
> issuers hit this too and work around it silently — nobody has asked them.

## Who it's for

*Which person or role, doing what. Secondary users only where materially different.*

> **Example:** A school administrator who issues credentials in batches at term
> end and occasionally has to undo one batch — wrong template, wrong cohort, a
> re-upload after a correction. Support handles the same job today on their
> behalf, which is the workaround being replaced.

## Outcome

*What is true for them once this ships, stated as their experience, not ours.*

> **Example:** The administrator who uploaded the wrong cohort can undo it
> themselves, in one action, and see that every recipient was told — without
> filing a support ticket or waiting for someone to run a script.

## Done when

*The observable checks. Two people reading these should reach the same pass/fail
conclusion.*

> **Example:**
> - An admin can revoke every credential in a batch from its detail page, in one
>   action, without leaving the page open.
> - Every revocation in a batch produces the same recipient notification, audit
>   entry and registry write as revoking one by hand.
> - Revoking a batch never touches a credential outside it — including batches
>   belonging to another school.
> - A credential issued before batches existed says so, rather than showing an
>   empty batch.

## In scope

*Each bullet an observable thing a person can do. Everything here is required —
if it's optional, it belongs in Out of scope or a later ticket.*

> **Example:**
> - Revoke every credential in a batch from the batch detail page
> - See progress while it runs, and a final count of what was revoked, skipped
>   and already revoked
> - See an explicit empty state for credentials that have no batch

## Out of scope

*The adjacent things people will ask for, and why it's safe to leave them out.*

> **Example:**
> - **Partial revocation** (revoke 300 of 400) — needs a row-selection UI and
>   nobody has asked; the four escalations were all whole batches.
> - **Undo** — revocation is already terminal for a single credential; making it
>   reversible for batches only would be a worse inconsistency than not having it.
> - **Scheduled or recurring revocation** — no request, and it changes the shape
>   of the feature entirely.

## Constraints and dependencies

*Deadlines, contractual or compliance obligations, other teams, anything that has
to exist first.*

> **Example:** Revocation must keep writing to the external registry within the
> existing 24-hour obligation — that isn't negotiable per batch size. Nothing
> depends on another team. **Assumption:** the registry tolerates 400 writes in
> quick succession; unverified.

## Open questions

*Question · who can answer · what it blocks. A PRD with none of these is usually
hiding them.*

| Question | Who | Blocks |
| --- | --- | --- |
| Do we tell recipients individually, or once per batch? | Product + Support | Design |
| What does an admin see for a batch that is half revoked already? | Product | Build |
| Is 400 the largest real batch, or have we seen bigger? | Data | Nothing — sizing only |
