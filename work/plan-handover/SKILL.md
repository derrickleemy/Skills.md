---
name: plan-handover
description: Explore, plan and hand a feature over to the team — plan mode with the user, then a decision note on a draft `[Plan - KEY-123]` PR plus implementation plans posted onto the tracker parent ticket and each sub-ticket. Requires a ticket that already carries a PRD (write one with /plan-brief first). Use when the user gives an issue key with a brief on it and wants it prepared for someone else to build, or runs /plan-handover.
---

# Plan Handover

One person explores and plans; the team implements. This skill produces the two
artefacts that make that handover work:

- a **decision note** on a draft PR — the *why*, the alternatives, the things
  clarified along the way, so the implementer's questions are already answered
- **implementation plans** — one `.md` per ticket, riding the same PR (or attached,
  in a repo that keeps them local), with a short description on each ticket and
  blocking links showing the real order — the *what to do*

Inputs: an issue key (`KEY-123`) and a sentence on what we're trying to achieve.

Written for Jira. The tracker calls are Jira MCP calls and the sub-ticket +
blocking-link model is Jira's; everything else is tracker-agnostic. For another
tracker, adapt **Posting to the tracker** and keep the rest.

## 0. Project config

Read `.claude/plan-config.md` in the repo — tracker site, project key, integration
branch, decision-note and plans paths, PR title formats, check commands, and
**whether the plans are committed or gitignored**. That last one changes how the
plans reach the team: committed, they ride the same draft PR as the decision note;
ignored, they stay local and the user attaches them by hand.

If it isn't there, stop and run `/plan-setup`, which detects what it can, asks the
rest in one pass and writes the file. Don't improvise the values; a wrong branch or
a wrong title prefix is discovered by the team, not by you.

## 1. The ticket must carry a PRD

**Read the ticket before anything else. No PRD, no planning.**

```
getJiraIssue(cloudId: "<tracker site>", issueIdOrKey: "KEY-123", fields: ["summary", "description"])
```

A PRD is not "the description has words in it". It states the problem
independently of any solution, who hits it, what's in and out of scope, and how
you'd know it worked. `/plan-brief`'s template is the shape; a hand-written brief
with different headings counts if it answers those things.

If it doesn't, **stop and say so** — name what's missing, then:

```
/plan-brief KEY-123 — <the goal in one line>
```

Planning against a one-line ticket is the failure this chain exists to prevent.
The decision note you'd write would record decisions made against a problem
nobody wrote down, the sub-tickets would inherit that, and the gap surfaces in
review — where it costs a week instead of twenty minutes.

Two exits, both requiring the user to say it out loud:

- **"This is the brief, go"** — they've read what's on the ticket and it's enough.
  Proceed, and treat that text as the PRD.
- **"Skip the PRD"** — proceed with their sentence as the brief, and say plainly in
  the decision note that the work was planned without one.

Never take either exit on your own inference. An empty ticket plus an
enthusiastic sentence from the user is not consent.

Once there is a PRD: it is the requirement. `<plans dir>/KEY-123-prd.md` is the
local copy. Plan against it, and raise a conflict with it rather than designing
around it silently. Its open questions are yours to either answer during
exploration or carry into the decision note — don't let them evaporate.

## 2. Plan with the user

Enter plan mode. Explore the codebase, propose an approach, argue it out.

### The plan put up for approval is plain English

The reader knows the product cold and wrote a fair amount of this codebase, but
may not have opened these files in months, and is deciding *whether this is the
right thing to build* — not reviewing a diff. Write for that.

So the plan file that `ExitPlanMode` presents is prose, and it answers:

- **What can't be done today**, in terms of what a customer or colleague hits.
- **What changes for them** once this ships.
- **The shape of the approach** — a few sentences, the mental model, not the
  call graph.
- **The calls worth arguing with**, each with the alternative and why it loses.
- **What we're deliberately not doing**, and what that costs later.
- **What we found out that we didn't know going in** — especially anything that
  contradicts what was assumed at the start.
- **Roughly how it splits up** and what has to land first.

Rules for this pass:

- No phase-by-phase file lists, no line numbers, no method signatures, no
  migration DDL. If a specific file or constraint *is* the decision, name it in
  one clause and move on ("the export payload has no room for extra fields") —
  don't cite it.
- Name real things: the customer, the standard, the field. Not "the entity".
- Unavoidable jargon gets one plain sentence before it's used.
- It should be readable end to end in a couple of minutes.

Explore as deeply as the work needs — the technical detail is what makes the
prose trustworthy — but keep it out of this document. It gets written down in
step 4, once the direction is settled.

If the technical detail is piling up faster than you can hold it, keep it in the
scratchpad, not in the plan file.

**Keep a running note of decision material as it happens.** This is the point
of the whole exercise and it cannot be reconstructed afterwards:

- decisions made, and the alternative a future reader would rediscover
- questions the user asked and how they were resolved
- **assumptions that turned out wrong** — these are the most valuable lines in
  any of these notes, and they only exist if captured mid-exploration
- constraints found in the code that pin the design (cite `path/to/file:line`)
- what was deliberately deferred, and what would go wrong if it weren't

Do not write files, branch, or touch git during this step.

## 3. After the plan is approved

Only once the user has approved via `ExitPlanMode`.

### Decision note

Write `<decision notes dir>/YYYY-MM-DD-<slug>.md` (today's date, `date +%F`).

If the directory has a `README.md` or existing notes, that is the convention —
read one and match it. Otherwise follow `templates/decision-note.md` in this
skill. Two things that are easy to skip either way:

- the note must stand alone — a reader arriving from a PR link or blame has
  not read the plan and will not open neighbouring notes
- product language before code language; omit sections that add nothing

Add a status line under the title, because these notes describe work that
hasn't been built yet:

```md
**Status:** planned, awaiting implementation. Tracked in [KEY-123](<ticket url>).
```

If the work can't start yet, say what blocks it instead — "blocked on X, recorded
now so the decisions survive until the work is scheduled."

Under `## Related`, link the ticket, any decision note this builds on or blocks,
and state the commit the codebase facts were verified against:

```md
- Codebase facts were verified by direct read on `<integration branch>` at commit `<sha>`; line numbers will drift.
```

### Branch, commit, draft PR

```bash
git checkout <integration branch> && git pull
git checkout -b docs/<slug>
git add <decision notes dir> && git commit -m "[KEY-123] <what the note records>"
git push -u origin docs/<slug>
gh pr create --draft --base <integration branch> \
  --title "[Plan - KEY-123] <title>" --body-file <scratchpad>/pr-body.md
```

Keep this branch checked out. If plans are committed, they land on it as a second
commit at the end of step 5 — one PR carrying the reasoning and the plans together.

Write the body file outside the repo (scratchpad or `/tmp`) — never inside it,
or it ends up in the commit.

If `gh pr edit` fails when you try to edit the body afterwards — repos with
Projects (classic) enabled return a deprecation error — use the REST API instead:

```bash
gh api -X PATCH repos/<owner>/<repo>/pulls/<n> -F body=@<scratchpad>/pr-body.md
```

The PR body is a summary of the note for reviewers, not the repo's code-change
template — there is no code to check off. `templates/pr-body-plan.md` is a filled
example; the shape is:

```md
## What
One paragraph: what this note records. Say "documentation only, no code changes".

## Why
The problem, in product terms. What can't be done today.

## The decision
The chosen approach and the main alternative, with why the alternative loses.

## Worth reviewers' attention
- The choices most likely to be argued with, or that go against the grain.
- Assumptions corrected during exploration.
- Open questions recorded as follow-ups rather than assumed away.

## Related
- [KEY-123](<ticket url>)
- Any decision note this builds on.
```

## 4. Overall implementation plan → the main ticket

Write the plan to `<plans dir>/KEY-123-<slug>.md`. Whether that file ends up on
the plan PR or stays on this machine is the config's call, not yours — write it
either way and let step 5 do the committing.

**This is where the approved plain-English plan becomes technical.** Same
decisions, no new ones — if writing it out surfaces a decision the prose glossed,
that's a question for the user, not a thing to settle silently.

The plan is the *how*, split so it can be picked up cold: phases, the files
each touches, the order they must land in, what's a hard prerequisite. Line
numbers, signatures, migrations and the exact names of things all belong here.
Keep the reasoning in the decision note — the plan links to it rather than
repeating it.

### Always include a PR breakdown

Directly after the goal, before the phases, state how the work splits into PRs:

```md
## PR breakdown

The stack is **linear** — each PR is based on the one above it, no forks:

| PR | Ticket | Phases | Based on | Scope |
| --- | --- | --- | --- | --- |
| 1 | KEY-124 | Phase 1 | `<integration branch>` | <what lands> |
| 2 | KEY-125 | Phases 2 + 3 | PR 1 | <what lands> |
| 3 | KEY-126 | Phase 4 | PR 2 | <what lands> |

<One line per PR on why it is its own PR — the prerequisite it satisfies, the
risk it isolates, or the point at which the feature turns on.>

<What is deliberately not a PR: data seeding, config, running a command per
environment.>
```

Phases are the build order; PRs are the review units, and they are not the same
thing. Merge phases into one PR when splitting them would make either half
harder to judge — a UI change reviewed apart from the endpoint it calls is worse,
not better. Split when a PR is a hard prerequisite for the rest, or when it
changes a signature existing callers use. If a PR looks too big, say where the
seam is, so the team doesn't split it backend/frontend by reflex.

### The stack must be linear

**Order the PRs into one chain, even when the work does not require it.** Two
sub-tickets that are genuinely independent still get an order: pick one, base the
second on it, and say in both plans that the dependency is presentational.

This is not a preference. GitHub's stacked-pull-request view models a stack as a
linear chain, so a PR whose base already has another child **cannot join the
stack at all** — it renders as a separate stack or as nothing. If the team reviews
the chain top to bottom and merges it as a unit, a forked PR silently drops out of
that flow.

Ordering rules, in priority order:

1. **Real dependencies first.** If B reads a column A adds, A comes first.
2. **Then narrative.** Order the rest so a reviewer reading bottom-up follows the
   feature being built: define the thing, then persist it, then validate it, then
   surface it. The chain is a reading order as much as a merge order.

Retro-fitting linearity later is possible but wasteful — it costs a merge commit
and a CI run per PR downstream of the fork, and re-parenting a branch that
already carries merge commits is worse. Decide the order here, in the plan, where
it is free.

Then add a plan pointer to the parent ticket — see **Posting to the tracker**
below — pointing at the plan file. If `/plan-brief` has been
through this ticket, the description is the PRD: **leave it alone and append**,
don't summarise it back. The PRD is the requirement and you are not its editor.

### Then create the sub-tickets

One per PR in the breakdown, in chain order. **Show the list first and wait for a
go** — creating tickets is a write to a board the whole team reads, and a wrong
breakdown is public:

```
About to create 3 sub-tickets under KEY-123:

  1. Index batch_reference and add the batch read model     (PR 1, base develop)
  2. Batch revocation job and endpoint                      (PR 2, on PR 1)
  3. Batch detail page: revoke, progress, empty state       (PR 3, on PR 2)
```

One word from the user, then create them. Summaries come from the breakdown
table's scope column — the same words, so the board and the plan agree.

**Check for existing sub-tickets first.** Read the parent's `subtasks`; on a re-run
some or all may already exist. Match by summary, create only what's missing, and
say which ones you skipped. Never create a second ticket for a PR that has one.

The sub-task issue type name varies by project (`Sub-task`, `Subtask`, `Sub Task`).
Read it once rather than guessing, and use whatever the config names if it does:

```
getJiraProjectIssueTypesMetadata(cloudId: "<tracker site>", projectIdOrKey: "<project key>")
createJiraIssue(cloudId: "<tracker site>", projectKey: "<project key>",
                issueTypeName: "<the sub-task type>", parent: "KEY-123",
                summary: "<from the breakdown table>",
                contentFormat: "markdown", description: "<see step 5>")
```

`parent` is what makes it a sub-task rather than a loose ticket in the project —
omitting it creates the right issue in the wrong place, and moving it afterwards is
manual. Leave the assignee unset; whoever picks the work up assigns themselves.

Report the keys it created, mapped to their PR. **Those keys are now the mapping
for step 5** — you created them, so no reconciliation is needed. If the connector
fails part way, say exactly which ones exist and which don't before doing anything
else; a half-created breakdown that nobody knows about is the worst outcome here.

## 5. One implementation plan per sub-ticket

You created the sub-tickets in step 4, so you already know which key is which PR.
Use that mapping.

**Only when the sub-tickets weren't created by this run** — a ticket prepared by
hand, or a re-run against an existing breakdown — read them off the parent and
match by summary:

```
getJiraIssue(cloudId: "<tracker site>", issueIdOrKey: "KEY-123",
             fields: ["summary", "description", "subtasks", "issuelinks"])
```

If the count doesn't match the PR breakdown, or a summary is ambiguous, say which
ones you couldn't map and ask — don't guess, and don't write a plan onto the wrong
ticket.

Then write one plan per key to `<plans dir>/KEY-124-<slug>.md`, same directory as
the parent plan.

Each sub-ticket plan is picked up by someone who has opened only that ticket:

- link the decision note and the parent ticket at the top
- name the sub-ticket this one is blocked by, and the one it unblocks
- restate just enough context to act — the goal of *this* PR, not the feature
- the files it touches, the tests it owes, and how to verify it in isolation
- say explicitly what belongs to a sibling ticket, so nobody builds it twice

Keep the parent plan as the map. A sub-ticket plan repeats the file paths it
needs rather than saying "see phase 2 of the parent".

Then, per **Posting to the tracker** below: write a description onto each
sub-ticket, and add blocking links so the board shows the real order.

### Where a sub-ticket's description comes from

Not from the plan, and not written from scratch: **it is a slice of the parent's
PRD.** Each sub-ticket takes the `In scope` bullets and the `Done when` checks
that *this* PR delivers, in the PRD's own words, plus its position in the chain
and pointers to its plan and the decision note. The implementer then reads the
same sentence the brief promised, not a paraphrase of it.

```md
Part of KEY-123. <Which slice of the feature this is, one sentence.>

**Delivers**
- <the In scope bullet(s) this PR owns, lifted from the PRD>

**Done when**
- <the Done when check(s) this PR satisfies, lifted from the PRD>

**Order** — blocked by KEY-124; unblocks KEY-126. Base your branch on KEY-124's.

**Details** — implementation plan: `docs/plans/KEY-125-<slug>.md` on PR #812
(or "attached as `KEY-125-<slug>.md`" when plans are kept local). Approach and
alternatives: PR #812.
```

**Every `In scope` bullet and every `Done when` check must be claimed by exactly
one sub-ticket.** Check the coverage before posting: an unclaimed bullet means the
PR breakdown has a hole, and a bullet claimed twice means two people are about to
build it. Either is a question for the user, not something to smooth over. A check
that no single PR satisfies alone — "the feature is reachable from the batch
page" — belongs to the PR that turns it on, and say so in that ticket.

A sub-ticket that delivers no user-visible slice (an index, a read model) still
names the check it *enables* and whose ticket that check lands on.

### Get the plans to the team

**When plans are committed** (the config's default), put them on the plan branch
you still have checked out:

```bash
git add <plans dir> && git commit -m "[KEY-123] implementation plans"
git push
```

One PR now carries the reasoning and the plans. Nothing to attach, and
`/plan-implement` can read them on any machine straight off the branch. If `git
add` reports the path is ignored, the config and the repo disagree — stop and say
so rather than forcing it with `-f`.

**When plans are gitignored**, they are machine-local and a tracker attachment is
not readable by an agent, so the plans directory is the only copy
`/plan-implement` can open. Finish with the attachment list — one line per ticket,
key and file path — since attaching is the user's step, and say plainly that
building on another machine means copying the directory across.

Either way, don't dump the plan bodies into the reply. Then stop.

## Posting to the tracker

Two different things go on a ticket, and they are not the same document.

**The `.md` file is the plan**, and the ticket only ever points at it.

When plans are committed, the pointer is a path on the plan PR — readable by the
team and by `/plan-implement`, on any machine. When they're kept local, the pointer
is an attachment filename and the user attaches it: the Jira connector has no
attachment tool, so you cannot. One line per ticket, path and key, so it's a
drag-and-drop list and not a puzzle.

Either way, before pointing at anything, make sure every plan is actually on disk
under its `KEY-123-<slug>.md` name — the parent plan from plan mode lives in
`~/.claude/plans/` under a generated name like `foamy-strolling-walrus.md`, so copy
it across rather than leaving it there. And note that nothing can read an
attachment back — not the connector's `fetch` (ARI only), not `WebFetch` (403
without a bearer token) — which is why a local-plans setup leaves the directory as
the only copy `/plan-implement` can open.

**The description is the friendly version** — and what goes there depends on
what's already on the ticket.

**A parent that carries a PRD** (`/plan-brief` ran, or someone wrote one) already
says what this is for, why, and what done looks like. Don't restate any of it.
Read the description, then **append one short block** and change nothing above it:

```md
## Plan

Approach and the alternatives considered: PR #812 — `[Plan - KEY-123]`.
Implementation plan: `docs/plans/KEY-123-<slug>.md` on that PR.
Splits into <n> sub-tickets, in a chain: <one clause each>. Each is blocked by
the one before it.
```

That is the whole parent write. Four lines, no summary of the PRD, nothing that
can contradict it later.

**A ticket with an empty description** — every sub-ticket you just had created,
and a parent nobody briefed — gets the friendly version written fresh. Short,
plain English, readable by someone opening the ticket cold on a phone. It is not
a summary of the plan's file list — it answers:

- what this ticket is for, in one or two sentences
- why it exists / what's broken or missing today
- what "done" looks like
- what it's waiting on and what's waiting on it
- a pointer to the plan file — its path on the plan PR, or its attachment name —
  and to the decision-note PR

Keep it to something you'd read in under a minute. No line numbers, no method
signatures, no file paths except the plan's. Bullets over paragraphs. If a
sentence only makes sense to someone who has the codebase open, it belongs in the
plan file instead.

```
editJiraIssue(cloudId: "<tracker site>", issueIdOrKey: "KEY-123",
              contentFormat: "markdown",
              fields: { description: "<the friendly summary>" })
```

**Link the tickets** so the board shows what blocks what. Read the blocked-by /
unblocks lines out of the plans and create one link per edge:

```
createIssueLink(cloudId: "<tracker site>", type: "Blocks",
                inwardIssue: "<the blocker>", outwardIssue: "<the blocked>")
```

Inward blocks outward — getting these backwards inverts the whole board, so read
them off the plan rather than from memory. Only real edges: if two tickets can run
in parallel, leave them unlinked and say so in both descriptions. Use
`getIssueLinkTypes` if `Blocks` isn't available on the site.

Rules:

- **Never overwrite a description that already has content.** Read it first. A
  PRD gets the appended `## Plan` block and nothing else. Anything else with
  content — someone's acceptance criteria, a pasted thread — gets shown to the
  user with a question, not a replacement.
- If the tracker connector isn't authenticated, say so and ask the user to run
  `/mcp` and connect it. Fall back to printing the summaries so the work isn't
  lost either way.
- Jira markdown handles headings, lists, tables and fenced code. Check one posted
  ticket renders before doing the rest of a batch.
- **Don't wrap inline code inside bold.** The markdown→ADF converter breaks the
  bold at the backtick, so ``**keep X `foo` as a fallback**`` renders bold up to
  `foo` and plain after it. Put the emphasis on a clause without code in it, or
  drop the bold. Same for `**...**` spanning a line break — write the whole
  emphasised phrase on one line.

Re-posting is the same call — it replaces the description, so on a PRD ticket
re-post the PRD **plus** the appended block, never the block alone. Read before
every write; `editJiraIssue` has no append mode. The file in the plans directory
stays the source of truth for the plan itself.

## Rules

- **No PRD on the ticket, no plan.** Step 1 is a gate, not a formality — the
  only way past it is the user saying so out loud.
- **Sub-tickets are created, not requested — but never without showing the list
  first.** And never a second ticket for a PR that already has one.
- **Nothing is written before the plan is approved.** Steps 1 and 2 are read-only.
- **The plan approved in step 2 is prose; the technical version is written in
  step 4.** Never put the file-by-file plan up for approval — the decision being
  made there is "is this the right thing to build", and file lists bury it.
- **Decision note ≠ PR description.** The note is for someone debugging this in
  a year; the PR body is for the reviewer this week.
- One note per feature, on the main ticket — sub-tickets get plans, not notes.
  If it needs a second note to make sense, merge them.
- Base is always the integration branch, PR is always `--draft`, title always
  carries the plan prefix — that prefix is how these are told apart from real work.
- Skip the follow-up sections when there's nothing real in them. Don't pad.
