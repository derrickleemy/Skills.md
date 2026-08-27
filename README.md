# Skills.md

The Claude Code skills I actually use, kept in one place so other people can steal them.

A skill is a folder with a `SKILL.md` in it. The frontmatter `description` is the only
part Claude reads up front — when your request matches it, the rest of the file loads
and Claude follows it instead of improvising. So a skill is really a written-down
procedure: the thing you'd otherwise re-explain every time.

I write one when I notice I've explained the same process to Claude three times, or
when the default behaviour is *nearly* right but skips the step that matters.

## What's here

| Skill | Folder | What it does |
|---|---|---|
| [`dalio-5-step`](personal/dalio-5-step) | personal | Interviews you through Ray Dalio's 5-Step Process, one step at a time, and won't let you skip to solutions |
| [`people-intelligence`](personal/people-intelligence) | personal | Turns your notes about a person into advice for one specific upcoming conversation |
| [`interactive-thinking`](personal/interactive-thinking) | personal | Makes you argue for a position instead of reading an analysis — stakes, competing perspectives, then attacks your strongest reasoning |
| [`plan-setup`](work/plan-setup) | work | Points the three skills below at your repo — writes `.claude/plan-config.md`. Run once per repo |
| [`plan-brief`](work/plan-brief) | work | Interviews you into a PRD and writes the ticket — refuses vague answers, one question at a time |
| [`plan-handover`](work/plan-handover) | work | Plan a feature with Claude, then hand it to the team as a decision-note PR + per-ticket implementation plans in Jira |
| [`plan-implement`](work/plan-implement) | work | Take a ticket that already has those plans and build each sub-ticket to a green PR |

**personal/** works for anyone out of the box — no repo, no tooling, no accounts.
**work/** is my team's feature workflow — see
[the end-to-end walkthrough](work/WALKTHROUGH.md) — and it's generic: `/plan-setup` writes a
`.claude/plan-config.md` in your repo with your branch, paths, commands and tracker,
and the other two skills read it. It does assume Jira and the GitHub CLI. Each
folder's README says more.

## Install

Skills go in `~/.claude/skills/` (available everywhere) or `.claude/skills/` inside a
project (that project only). The skill folder must be a *direct* child — the
`personal/` and `work/` split here is for browsing, not for nesting.

```bash
git clone git@github.com:derrickleemy/Skills.md.git
cd Skills.md

# one skill, user-wide
cp -R personal/dalio-5-step ~/.claude/skills/

# or symlink, so a git pull updates it
ln -s "$PWD/personal/dalio-5-step" ~/.claude/skills/dalio-5-step
```

Run `/doctor` to confirm it was picked up. (On another agent, see
**Written for Claude Code** below.)

## Written for Claude Code

That's what I use, so that's what these assume: a `SKILL.md` in `~/.claude/skills/`,
picked up by description, invoked as `/name`. Nothing here is deep magic though —
a skill is a procedure written down, and any agent that can be handed a procedure
can follow it. If you're on something else, the substance carries over and you swap
the mechanics.

The Claude-specific pieces, and what they are underneath:

| What the skills use | What it's doing | Swap for |
|---|---|---|
| `~/.claude/skills/<name>/SKILL.md` | how a skill gets discovered and loaded | wherever your harness keeps reusable instructions — a prompt file, a custom mode, a rules directory |
| `/name` invocation | running one deliberately | however you trigger a saved prompt |
| `/doctor` | confirming a skill was picked up | your harness's equivalent, or just ask it to list what it loaded |
| **plan mode** + `ExitPlanMode` | read-only exploration, then one approval gate before anything is written (`/plan-handover` §1–2) | any explicit "propose, then wait for my yes" step. The gate matters more than the mechanism — without it the skill writes files before you've agreed to the direction |
| `/goal` | re-running a turn until a condition holds, which is what carries `/plan-implement` across five sub-tickets unattended | a loop or eval harness if you have one; otherwise type "next ticket" between them and babysit. The skill says this and degrades cleanly |
| `/code-review high` | a second pass over the diff with fresh context (`/plan-implement` §5) | any review command, or a subagent told to review the working diff |
| `/mcp` | connecting the Atlassian connector | however you attach tools; the Jira calls themselves are MCP, not Claude |
| `~/.claude/plans/<generated-name>.md` | where a plan-mode plan gets parked before the skill copies it somewhere permanent | wherever your harness writes its scratch plans — or delete the step and write the plan directly |
| the scratchpad path for PR bodies | keeping a body file *outside* the repo so it can't land in a commit | any temp directory. Don't skip it — this one bites |

The Jira tool calls (`getJiraIssue`, `editJiraIssue`, `createJiraIssue`, …) come
from the Atlassian MCP server, so they work with any MCP-capable agent. `gh` and
`git` are just CLIs.

`work/` skills also carry Jira and GitHub assumptions beyond the harness —
`work/README.md` covers those.

## Tools I use alongside these

Skills cover the *procedure*. These cover the parts of the loop Claude Code isn't:

| Tool | What it does | Where it fits |
|---|---|---|
| [crit](https://github.com/tomasz-tomczyk/crit) | Review plans, diffs and live pages in a browser UI, leave inline comments, send them straight back to the agent | The human review gate — instead of reading a plan in the terminal and typing paragraphs of feedback, comment on the line you mean. Pairs with `/plan-handover`'s approval step |
| [NotebookLM](https://notebooklm.google.com) | Upload sources, ask questions grounded only in them, audio overviews | Reading-in before a plan exists — dump the RFCs, vendor docs and meeting transcripts in, interrogate them there, then bring the conclusions into `/plan-brief` |

## Use

Claude picks a skill up on its own when your request matches the description — that's
the point, and it's why the descriptions are long and full of the phrases I actually
type. You can also invoke one directly:

```
/dalio-5-step
/people-intelligence how should I approach Sam about the deadline slipping
```

Three things that make skills work better, learned the hard way:

- **Be specific in the request.** "Prep me for the 1:1 with Sam about the promotion
  he didn't get" beats "help with Sam" — the skill has more to work with, and a
  vague ask is what makes Claude guess.
- **Answer the questions properly.** `dalio-5-step` and anything else
  interview-shaped is only as good as your answers. One-word replies get a
  one-word-quality worksheet.
- **Edit the skill when it's wrong.** These are text files, not a product. If the
  procedure doesn't fit, change the file — that's cheaper than arguing with Claude
  about it every session.

## Why I bother

Claude is good at the work and bad at *my* order of operations. Left alone it jumps
to a plan before the problem is named, writes a technical doc when a colleague needed
prose, or gives me a personality profile when I asked how to open a hard conversation.
A skill fixes that once instead of every time — and because it's a file, the process
is now reviewable, diffable and shareable, which the version living in my head never was.

## License

MIT — take them, fork them, make them yours.
