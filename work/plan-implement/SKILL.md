---
name: plan-implement
description: Build out a tracker ticket that already has an implementation plan on it — read the parent and its sub-tickets, order them by what blocks what, and take each one to a green PR in this session. Use when the user gives an issue key prepared by /plan-handover and wants it implemented, or runs /plan-implement.
---

# Plan Implement

The other half of `/plan-handover`. That skill wrote the decision note and the
per-sub-ticket implementation plans; this one builds them.

Input: an issue key. Usually the parent `KEY-123` that has sub-tickets, but a
single sub-ticket key works too.

One session, one sub-ticket at a time, in this checkout. You build it, you test
it, you review it, you take it to a green PR, then you start the next one. The
user's only job after starting is reviewing the PRs.

Written for Jira; the tracker calls are Jira MCP calls. Everything else is
tracker-agnostic — for another tracker, adapt **From the tracker** in step 2.

## 0. Project config, and keeping the session alive

Read `.claude/plan-config.md` in the repo — tracker site, integration branch,
plans directory, whether the plans are committed or gitignored, PR title formats,
check commands, project quirks. If it's missing, run `/plan-setup` first;
everything below reads from it.

**Whether this can run unattended depends on your setup.** The skill is designed
for a session that keeps re-running turns until a condition holds. In Claude Code,
that's `/goal` — a skill cannot set one, it has to be typed, so print this for the
user to paste and fill in:

```
/goal Every sub-ticket of KEY-123 has an open PR against the correct base, titled with its own [KEY-124] prefix, with `gh pr checks` showing every required check passing and a filled-in "## Implementation decisions" section in the body. Track progress in <plans dir>/KEY-123-progress.md and paste the full `gh pr checks` output each turn. Stop after 60 turns and report exactly what is unfinished and why.
```

Setting a goal starts a turn immediately, so pasting it is the go signal — this
skill is already loaded in the conversation and stays in force for every goal
turn. A user who knows they want this can skip the two-step and open with the
goal line directly, adding "Follow the /plan-implement skill." to the front of
the condition.

The condition re-runs the turn until an evaluator agrees it holds, which is what
carries this across five sub-tickets without the user prompting between them. The
evaluator does not run tools — it judges only what is in the transcript, which is
why the condition asks for `gh pr checks` output every turn. The turn cap is not
optional.

Keep the condition close to the template above — it is ~400 characters against a
4,000-character ceiling, and that headroom is there for the key substitutions,
not for content. The plans, the acceptance criteria and the file lists belong in
the plans directory, which you read anyway; a condition that inlines them is both
over the limit and harder for the evaluator to judge.

**If your setup has no equivalent**, everything below still works — the user types
"next ticket" between sub-tickets and babysits the run. Say so up front rather
than letting them walk away from a session that will stop after one turn.

Either way, keep `<plans dir>/KEY-123-progress.md` current: one line per
sub-ticket with its branch, PR number and state. A long run gets compacted, and
that file is what survives it. `templates/progress.md` is a filled example.

## 1. Preflight

This checkout is the user's live working directory.

```bash
git status --short        # must be empty
gh auth status
git checkout <integration branch> && git pull
```

If the tree is dirty, stop and say so. Don't stash the user's work.

## 2. Gather the plan

### The plans are files, not tracker content

The ticket description is a short summary that deliberately omits the file lists,
and a tracker attachment is not readable by an agent at all — the Jira connector
has no attachment tool, its `fetch` takes an ARI for an issue or page only, and the
attachment content URL returns 403 without a bearer token. So the plan you follow
is always a file:

```
<plans dir>/<KEY>-<slug>.md
```

**When plans are committed** (the config's default), they are on the same
unmerged `[Plan - KEY-123]` branch as the decision note. Normally they are already
on disk from the planning session; if this is a different machine, fetch them:

```bash
gh pr list --search "Plan - KEY-123 in:title" --state all --json number,headRefName,title
git fetch origin <headRefName>
git checkout origin/<headRefName> -- <plans dir>
```

That is the whole reason to commit them — nothing has to be copied between
machines by hand.

**When plans are gitignored**, the local directory is the only copy that exists.
If it's empty, they are on another machine and you cannot recover them here; say
so and ask the user to copy the directory across. Don't reconstruct a plan from
the ticket description.

Either way: parent plans made during plan mode may sit in
`~/.claude/plans/<scratch-name>.md` under a generated name — check there before
concluding a plan is missing, and copy it to `<plans dir>/<KEY>-<slug>.md` so the
set is in one place. Match files to tickets by key.

### From the tracker

The tracker gives you the ordering and the framing, not the plan:

```
getJiraIssue(cloudId: "<tracker site>", issueIdOrKey: "KEY-123",
             fields: ["summary", "description", "subtasks", "issuelinks", "attachment"])
```

- `subtasks` — the tickets to build.
- `issuelinks` — the blocking edges. This is the authoritative order; the plans
  restate it but the board is what the team sees.
- `description` — what the ticket is for and what "done" looks like. Useful
  context, and the acceptance criteria to check yourself against.
- `attachment` — confirms which plan file belongs to this ticket, by filename.

If the connector isn't authenticated, say so and ask the user to run `/mcp` and
connect it. Without it you can still work from the plans directory, but you lose
the blocking links — ask the user for the order rather than guessing it.

### The decision note

The note explains *why*, and it is what you follow when a plan is ambiguous. It
usually lives on an unmerged `[Plan - KEY-123]` draft PR, so it is not on the
integration branch:

```bash
gh pr list --search "Plan - KEY-123 in:title" --state all --json number,headRefName,title
git fetch origin <headRefName>
git show origin/<headRefName>:<decision notes dir>/<file>.md > <plans dir>/KEY-123-decision-note.md
```

If there is no plan PR, check the decision notes directory on the integration
branch.

Keep all of it in the plans directory. Where plans are gitignored that also
guarantees nothing leaks into a commit; where they're committed, the decision-note
copy is a working file — don't commit it back onto a feature branch.

## 3. Order the work

Build the dependency graph from the sub-ticket *blocked by* lines and the
parent's PR breakdown table, and print it before starting:

```
1  KEY-124  Index batch_reference + read model      base: <integration branch>
2  KEY-125  Job + endpoint      (blocked by KEY-124) base: KEY-124 branch
3  KEY-126  Batch detail UI     (independent)        base: KEY-125 branch
```

Mark which sub-tickets are independent of everything unstarted, so the user can
take one to a second machine. Independent still means **it gets a position in the
chain** — see below.

### The stack is one linear chain

Your end state is "PR green", not "PR merged" — merging stays with the user. So
every ticket after the first branches off the previous one **in the plan's PR
breakdown order**, not off the integration branch and not off a shared parent:

- branch from the previous PR's branch
- open the PR with `--base <previous-branch>`
- note "stacked on #<n>" in the PR body

**This holds even for sub-tickets with no real dependency between them.** GitHub's
stacked-pull-request view models a stack as a linear chain, so if two PRs share a
base, the second cannot join the stack — it renders separately and drops out of
the review flow. Two independent tickets still get an order: take the one the plan
lists first, and base the other on it.

If the plan's breakdown has no order — an older plan, or one written before this
rule — pick one yourself, keep it consistent, and say in both PR bodies that the
dependency is presentational so nobody hunts for a coupling that is not there.

Retro-fitting linearity costs a merge commit and a CI run on every PR downstream
of the fork, so get it right when the branch is created.

GitHub retargets to the integration branch automatically once the parent merges.

## 4. Build one sub-ticket

Branch `KEY-124-<slug>` from the integration branch, or from the blocking branch
if stacked.

Follow the sub-ticket's plan. It names the files, the tests it owes and how to
verify it in isolation; the decision note explains why it is shaped that way.
Stay inside the ticket's scope — what its plan assigns to a sibling ticket is
not yours to build, or it gets built twice.

### Decisions the plan doesn't cover

Make the call and keep going. Do not stop to ask, and do not stall — the point
of running this unattended is that a gap in the plan costs minutes, not a day.

Record every one in the PR body under `## Implementation decisions`: what you
decided, the alternative, why the alternative lost. That section is the deal —
autonomy in exchange for the user being able to audit the calls in one place.
`templates/pr-body-implementation.md` shows what that section looks like filled
in properly.

The exception is a decision that would waste the entire PR if wrong and that
the decision note is silent on. Raise those in step 3, before starting, along
with any other question — batched, not one at a time.

### Prove it

Run the repo's own checks, in the order the config lists them — formatter, static
analysis, architecture rules, the relevant tests, and the frontend tests if the
diff touches the frontend. Scope them to what changed where the tool allows it;
run the whole suite before opening the PR.

If the config names a wrapper — a task runner, a `make` target, a command that
enters the project environment — **every check goes through it**. Never run the
bare binary directly, or you are testing a different environment from CI.

Check the config's **project quirks** section before debugging a tool failure by
hand. Cache permissions, container ownership and dirty state are the usual
culprits and are usually already written down there. If you hit one that isn't,
add it.

## 5. Review your own diff before the PR exists

You wrote it, so you are the worst reviewer of it. Do both passes.

**Fresh eyes first.** Run `/code-review high` on the working diff and act on
what it finds. It reviews with context you have stopped seeing.

**Then the repo's own checklist** — the config names the files (a `REVIEW.md`,
an `AGENTS.md`, a PR template's checklist). Read them and go through every line
honestly.

Whether or not the repo has one, cover the classes of failure that CI cannot
catch, because this is where reviewers actually push back:

- **Authorisation scoping on the base query**, not delegated to an optional filter
  argument — and a negative test proving another tenant/user/org gets nothing
- **Forbidden-path coverage**: the request that should be rejected, tested
- **Silent behaviour changes** — fallback order, defaults, field precedence,
  error codes — shipped inside a "refactor" without being named as changes
- **Input validation kept narrow**, and mappers kept as mappers: no business rules
  smuggled into a serializer
- **Docs, validation, tests and response shape agreeing** about the same endpoint
- **Migrations that account for existing rows**, rollback, and deployment order
- The repo's own code style and testing conventions, as written down, not as
  guessed from the nearest file

Fix what both passes find before pushing. Do not open the PR and then review it.

## 6. Open the PR and take it to green

```bash
gh pr create --base <base branch> --title "[KEY-124] <title>" \
  --body-file <scratchpad>/pr-body.md
gh pr checks <n>
```

Write the body file outside the repo, or it ends up in the commit. Fill in the
repo's PR template — including the ticket link and whatever labels it demands —
and add the `## Implementation decisions` section.

If `gh pr edit` fails when editing a body afterwards — repos with Projects
(classic) enabled return a deprecation error — use the REST API:

```bash
gh api -X PATCH repos/<owner>/<repo>/pulls/<n> -F body=@<scratchpad>/pr-body.md
```

Then watch the checks and fix what fails. Green means every required check, not
just the tests.

Two traps that are common enough to expect anywhere:

- **A lint workflow that auto-commits style fixes to your branch.** After any CI
  run, `git pull` before pushing again or the push is rejected.
- **A duplication gate on new code** (SonarCloud and friends), where lines
  duplicating *pre-existing* code count against you. Copying a sibling
  controller or request class will fail it — extract a shared helper instead of
  rewording.

Anything else CI does here is in the config's project quirks.

Once it is green, update `<plans dir>/KEY-123-progress.md`, return to the
integration branch, and start the next sub-ticket. Don't stop to report progress
in between — the summary comes at the end.

```bash
git checkout <integration branch> && git pull
```

## 7. Report

When every sub-ticket has a green PR:

- one line per PR: key, number, base, and whether it is stacked
- the merge order, since stacked PRs must go in bottom-up
- every decision made along the way, collected from the PR bodies — the user
  needs to see these in one place, not spread across five PRs
- anything deferred, and why the PRs are still safe without it

## Rules

- **Never merge.** Green CI is the end state. Merging is the user's call.
- **Every autonomous decision lands in the PR body.** That is the whole reason
  you are allowed to decide without asking.
- **Green CI is not a passing review.** The tools don't check authorisation
  scoping, negative-path coverage, or silent behaviour changes. That pass is
  yours, and skipping it is skipping the point.
- **Front-load every question.** A dirty tree, an unauthenticated connector,
  sub-tickets that don't map onto the PR breakdown, a plan ambiguity the
  decision note doesn't settle — raise them together before starting the first
  sub-ticket. The user walks away after starting this; a question that surfaces
  on ticket four costs them the whole gap.
- **One sub-ticket at a time in this checkout**, unless the config says the
  toolchain tolerates more. If tests bind to the repo root, parallelism comes
  from another machine, not another branch here.
- **Keep the progress file current.** A long run gets compacted; it is what
  survives.
