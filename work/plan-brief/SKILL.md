---
name: plan-brief
description: Interview the user into a written PRD and put it on the ticket, before any planning happens. Walks the PRD template one question at a time, refuses vague answers, then writes the ticket description (creating the ticket if there isn't one). Use when the user has an idea, a complaint, a request from someone else, or an empty ticket, and says "write the ticket", "turn this into a PRD", "spec this out", "interview me on this", or runs /plan-brief. The step before /plan-handover.
---

# Plan Brief

You are running an interview, not writing a spec. The user does the thinking; you
hold them to answering the question that was actually asked.

The output is one PRD, on the ticket, that someone else could plan from without
talking to the user. `/plan-handover` starts where this stops.

Input: an idea in one sentence, and optionally an existing ticket key. Neither
needs to be tidy.

## 0. Config and template

Read `.claude/plan-config.md` for the tracker site and project key. Missing → run
`/plan-setup` first.

**The template drives the interview.** `templates/prd.md` in this skill defines the
sections and their order; each section carries the question it answers in italics.
Walk them in order. If the repo has its own PRD template — check the config, then
`docs/`, `.github/ISSUE_TEMPLATE/` — that one wins, and the interview follows its
headings instead.

Do not invent sections, and do not skip one because the answer seems obvious. A
section the user genuinely has no answer for gets recorded as an open question,
not quietly dropped.

## 1. Before the first question

Spend a few minutes finding out what the user is about to tell you, so you can
push back with facts rather than vibes:

- **Search the tracker** for the same idea. A duplicate, or a closed ticket that
  already rejected this, changes the conversation completely. Say so immediately if
  you find one.
- **Look in the codebase** for whether some of this already exists — a half-built
  flag, an endpoint nobody surfaced, a table with the column already on it.
- **If a ticket key was given**, read it. Whatever is there — a one-liner, a pasted
  Slack thread — is the user's first answer, not something to ignore.

Report what you found in a few lines, then start. Don't turn this into a research
project; the interview is the point.

## 2. Run the interview

**One question per turn. Wait for the answer. Never batch questions.**

Name the section in one line before asking, so the user knows where they are.
Reflect the answer back in one sentence, sharpened. Then either press once, or
move to the next section.

Press when an answer is:

- **A solution, not a problem.** "We need a bulk-revoke endpoint" is a solution.
  Ask what breaks without it, and who notices.
- **A category, not a person.** "Users find it confusing" → which user, doing
  what, at which step.
- **Unfalsifiable.** Aimed at **Outcome** and **Done when**: "it'll be much
  easier" → what would two people look at to agree it worked? We have no metrics
  section to hide a vague answer in, so this is where the PRD gets its teeth.
- **A number with no source.** Ask where it comes from. "It feels like a lot" is
  an honest answer — it becomes `**Assumption:**` or an open question, not a fact.
- **Scope with no edge.** If everything is in scope, ask what they'd cut to ship
  a month sooner. Then ask what people will request the week after launch —
  that's the Out of scope list, and an empty one means it hasn't been thought about.

**Press at most twice on any one point.** A stalled interview is worse than an
imperfect answer — record what they gave you, note the weakness in Open
questions, move on.

**Ask about the date once**, alongside Constraints: is there a date this has to
be true by, and what makes it real? "No" is a perfectly good answer and drops the
`Needed by` line entirely. A date with no consequence attached gets one push —
"what happens if it's a month later?" — and if nothing happens, it comes out.

Aim for 8–12 exchanges total. If the user answers three sections in one message,
take all three and skip ahead; don't re-ask what they already told you.

### What you don't do here

- **Don't design the solution.** No approaches, no file names, no architecture. If
  the user starts designing, take a one-line note of it for the handover and pull
  them back: *"That's a plan question — `/plan-handover` will settle it. Stay on
  what's broken."*
- **Don't answer for them.** Offering two options to react to is fine; filling in
  their outcome is not.
- **Don't soften the problem statement.** A PRD whose problem section flatters
  everyone involved is the one nobody can plan from.

## 3. Self-check before writing

Run these against the draft in your head and report the result in a few lines,
quoting the offending text. This catches what the interview let through — it is
not a second interview.

- **Problem stated without naming the solution.** If the problem section only
  makes sense once you know the feature, it's a feature request.
- **Every evidence line has a source**, and anything unsourced is labelled
  `**Assumption:**`.
- **"Who it's for" names a person or role doing something**, not "users" and not
  "the business".
- **Outcome is in their terms** and doesn't just restate the scope list back.
- **"Done when" checks are observable** — two readers reach the same pass/fail.
- **Out of scope is non-empty** and names what people will actually ask for.
- **Every open question has someone who can answer it and something it blocks.**
- **If a `Needed by` date is set**, something in the PRD says what makes it real.
  Otherwise it's a wish, and the line comes out.

Each failure gets one of two outcomes, the user's choice: fix it now, or write it
into the PRD as what it is — an assumption, or an open question. **Never pass a
failure silently**, and never fix it by inventing the missing content.

## 4. Write the PRD

Fill the template. Rules:

- Every section answers its question and then stops. No restating the problem in
  four places.
- The user's own words where they were sharp. Don't launder a good sentence into
  corporate.
- Nothing in the PRD that the user didn't say or you didn't verify. If you inferred
  something to make it read, mark it and ask.
- Delete the italic prompt lines and the `Example:` blocks from the template.
  Neither belongs in a finished PRD.
- Empty sections come out as *"Not established — see Open questions"*, never as
  invented content. The one section that can be absent entirely is the
  `**Needed by:**` line, when there is no real date.
- Unsourced claims keep their `**Assumption:**` label. Don't quietly promote one
  to a fact because it reads better without the label.

### The approval gate

**Print the PRD in full and stop. Do not write it anywhere in the same turn.**

That is the gate. This skill runs outside plan mode on purpose — an interview
needs plain questions in prose, not a form — so nothing in the harness will stop
you writing to the tracker before the user has agreed. The stop is yours to keep.

Ask for corrections, take them, print it again if the changes were material. Only
once the user says it's right does anything leave this conversation. This is the
one document the rest of the chain trusts, and it is about to become a ticket
other people plan and build from.

## 5. Put it on the ticket

Set the summary too — a short, plain title someone can recognise on a board.

With an existing key:

```
getJiraIssue(cloudId: "<tracker site>", issueIdOrKey: "KEY-123", fields: ["summary", "description"])
editJiraIssue(cloudId: "<tracker site>", issueIdOrKey: "KEY-123",
              contentFormat: "markdown",
              fields: { summary: "<title>", description: "<the PRD>" })
```

**Never overwrite a description that already has content.** Read it first; if
there's anything there, show the user what's there and ask before replacing it.

With no key, create the ticket:

```
getJiraProjectIssueTypesMetadata(cloudId: "<tracker site>", projectIdOrKey: "<project key>")
createJiraIssue(cloudId: "<tracker site>", projectKey: "<project key>",
                issueTypeName: "<Story | Task>", summary: "<title>",
                description: "<the PRD>")
```

Ask which issue type if the project has more than the obvious one, and report the
key it created.

Then keep a local copy alongside the plans — `<plans dir>/KEY-123-prd.md` — so
`/plan-handover` can read it without a tracker round-trip. That directory is
gitignored.

If the tracker connector isn't authenticated, say so and ask the user to run `/mcp`
and connect it. Print the PRD either way, so an hour of interview isn't lost to a
missing token.

Jira markdown: headings, lists, tables and fenced code work. **Don't wrap inline
code inside bold** — the markdown→ADF converter breaks the bold at the backtick.

## 6. Hand off

Close with:

- the ticket key and link
- the open questions, listed plainly — these are what the user owes someone before
  planning is worth starting
- anything the user said while designing, saved for the next step
- the next command, ready to paste:

```
/plan-handover KEY-123 — <the outcome in one line>
```

## Rules

- **No hypothesis, no metrics table, no alternatives, no risks, no rollout.**
  That is a deliberate choice, not an omission: we have no instrumentation to make
  baseline→target rows honest, and alternatives and trade-offs are the decision
  note's job in `/plan-handover`. If the interview produces any of it, keep it as
  a note for the handover — don't grow a section for it here.
- **The PRD says what and why, never how.** The moment it names a file or an
  approach, it has stopped being a PRD and started pre-empting the plan.
- **One question at a time.** Batched questions get batched, shallow answers.
- **Two presses maximum**, then record the gap as an open question.
- **Nothing invented.** An open question is a real answer; a plausible fabrication
  is a lie the whole chain then trusts.
- **Never overwrite an existing description silently.**
- **Never show the PRD and write it in the same turn.** The user's "yes" is a
  separate turn, and it is the only thing that ends step 4.
- Stop at the ticket. Don't start planning in the same breath — the user reads the
  PRD, then decides whether to plan it at all.
