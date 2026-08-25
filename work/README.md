# work

My team's feature workflow, written down. Four skills: one configures, then a
chain of three.

```
/plan-setup        →  once per repo. Writes .claude/plan-config.md

/plan-brief        →  interviews you into a PRD and writes the ticket
/plan-handover     →  plan it, hand it over as a decision note + sub-tickets
/plan-implement    →  build each sub-ticket to a green PR
```

[**WALKTHROUGH.md**](WALKTHROUGH.md) traces one feature through all four, end to
end, with a diagram of where a human is needed.

Each link answers one question and refuses the next one: `/plan-brief` says what
and why and won't design; `/plan-handover` decides how and won't write code;
`/plan-implement` writes the code and won't merge. Keeping those apart is the
whole trick — a document that tries to do two of them does neither well.

The chain is ordered, not a menu: **`/plan-handover` refuses to run on a ticket
with no PRD** and sends you to `/plan-brief` — planning against a one-line ticket
is the failure it exists to prevent. A ticket that already carries a brief, written
by hand or by anyone else, goes straight to `/plan-handover`; you can also override
the gate by saying "this is the brief, go".

The skills themselves are generic. Everything repo-specific — tracker site,
integration branch, paths, PR title formats, check commands, the CI quirks that
have bitten before — lives in `.claude/plan-config.md` **in your repo**, which
`/plan-setup` writes. Nothing here needs editing to fit your stack.

## Prerequisites

- **Jira**, reached through the Atlassian MCP connector (`/mcp` to connect). These
  skills are Jira-shaped — sub-tickets, `Blocks` links. Another tracker works if
  you adapt the two "posting to the tracker" sections; the rest is tracker-agnostic.
- **GitHub CLI**, authenticated (`gh auth status`).
- **Plan mode** — `/plan-handover` runs inside it.
- For unattended runs, a way to keep re-running a turn until a condition holds. In
  Claude Code that's `/goal`; `/plan-implement` prints the line to paste. Without
  it the skill still works, you just type "next ticket" between sub-tickets.

## Install

```bash
cp -R plan-setup plan-brief plan-handover plan-implement ~/.claude/skills/
```

Then, in the repo you'll be working in:

```
/plan-setup
```

It detects the branch, paths, PR template, CI and check commands, asks for the
handful it can't (tracker site, project key, and what your reviewers push back on
that CI doesn't catch), and writes the config. Re-run it when conventions change.

## [`plan-setup`](plan-setup)

Detect-then-confirm, not a twelve-question form. Reads `CLAUDE.md`, `AGENTS.md`,
whatever defines your tasks (`Makefile`, package scripts, the language's runner),
`.github/workflows/`, and recent PR titles — then asks only about the gaps.

It also asks one question with real consequences: **are the implementation plans
committed or gitignored?** Committed (the default) puts them on the same draft PR
as the decision note — reviewers can read them, and `/plan-implement` can fetch
them on any machine. Gitignored keeps line numbers and internal paths out of the
repo, at the cost of the plans existing only where they were written.

The section worth your attention is **project quirks** — the things that have cost
a run before (a lint job that auto-commits to your branch, a duplication gate, a
root-owned cache in the container). It exists nowhere in the repo, and it's what
stops an agent re-debugging the same failure every week. Add to it as you hit
things.

## [`plan-brief`](plan-brief)

An interview, one question per turn, that ends with a PRD on the ticket — creating
the ticket if there isn't one. It refuses the answers that make a PRD useless: a
solution where a problem belongs, "users" where a person belongs, a metric with no
source, a scope with no edge. It presses twice, then records the gap as an open
question and moves on.

It searches the tracker and the codebase before asking anything, so "we already
have a half-built version of this" surfaces in minute one rather than during
planning.

```
/plan-brief we need some way to undo a bulk issuance
```

[`templates/prd.md`](plan-brief/templates/prd.md) **is the interview.** The skill
walks that file's headings in order and asks the italic question under each one —
so swapping in your own PRD template changes the interview, with no edit to the
skill.

Ours is a deliberately lean one-pager, built from a survey of Atlassian, Amazon
(Working Backwards), Google, Netflix, Duolingo and OGP/FormSG practice. What it
borrows: problem-before-solution, auditable evidence, explicit non-goals,
requirements as observable capabilities, and owned open questions. What it
deliberately drops, and why:

- **Hypothesis and a baseline→target metrics table.** The strongest recommendation
  in the literature, and useless without instrumentation — a made-up baseline is
  worse than none. `## Done when` carries falsifiability instead: observable
  checks two people would agree on.
- **Alternatives, trade-offs, risks, rollout, NFRs, decision log.** These have a
  home already — the decision note `/plan-handover` writes, and the implementation
  plans. No section appears in both documents.
- **The metadata block.** Status, owner, links and dates are Jira fields;
  duplicating them into a description just gives them somewhere to go stale. The
  one exception is an optional `**Needed by:**` line, which carries the date *and*
  what makes it real.

## [`plan-handover`](plan-handover)

Explore, plan, then produce the two artefacts that make a handover actually work:

- a **decision note** on a draft `[Plan - KEY-123]` PR — the *why*, the
  alternatives, what we found out along the way, so the implementer's questions
  are answered before they ask
- **implementation plans** — one per ticket, committed onto that same PR, with
  blocking links showing the real order

The plan you approve is deliberately plain English — no file lists, no method
signatures. You're deciding *whether this is the right thing to build*. The
technical detail gets written after, once the direction is settled.

```
/plan-handover PROJ-101 — let orgs bulk-revoke credentials by batch reference
```

It creates the sub-tickets itself — one per PR in the breakdown, parented to the
main ticket — after showing you the list. On a re-run it matches what's already
there instead of duplicating.

**To get value out of it:**

- Run `/plan-brief` first. This skill refuses a ticket with no PRD on it.
- Argue with the plan while it's still prose. That's the cheap moment.
- Check the PR breakdown before approving — the stack must be linear, and a wrong
  split is expensive to unwind later.

[`templates/`](plan-handover/templates) has a worked decision note and the matching
PR body, filled in with a realistic example rather than placeholders. If your repo
already has a decision-notes directory, that convention wins and the templates are
just reference.

## [`plan-implement`](plan-implement)

The other half. Reads the plans `/plan-handover` wrote, orders the sub-tickets by
what blocks what, and takes each one to a green PR — build, test, self-review,
open, fix CI, next.

```
/plan-implement PROJ-101
```

**To get value out of it:**

- Paste the `/goal` line it prints. Without it the session stops after one turn
  and you're back to babysitting.
- Keep the goal condition close to the template. It asks for `gh pr checks` output
  every turn because the evaluator can't run tools — it only sees the transcript.
- The plans ride the plan PR, so a second machine just fetches the branch. If your
  repo gitignores the plans directory instead, they're machine-local and you copy
  them across — `/plan-setup` records which.
- Your job after starting is reviewing PRs. That's the deal — in exchange, every
  decision the agent made without asking is in the PR body under
  `## Implementation decisions`.

[`templates/`](plan-implement/templates) has a worked implementation PR body and a
progress file.

## Why I use these

I'm the one exploring and deciding; the team is the one building — so the
bottleneck was never the code, it was everything I knew and hadn't written down.
These force the *why* onto a PR and the *what to do* onto the ticket, in the two
different registers those two audiences need, instead of one document that serves
neither. `/plan-implement` then exists because a plan good enough to hand to a
person is also good enough to hand to an agent, and reviewing five PRs is a better
use of my afternoon than writing them.
