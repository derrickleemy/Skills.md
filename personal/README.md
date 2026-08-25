# personal

Thinking tools. No repo, no Jira, no accounts — these work for anyone the moment
you copy them in.

## [`dalio-5-step`](dalio-5-step)

Runs Ray Dalio's 5-Step Process on you as an interview: clear goals → identify
problems → diagnose root causes → design a plan → push through to completion.
One question per turn, strictly in order, and it stops you when you answer a
problem with a solution. Ends in a one-page worksheet.

**Use it when** you're stuck and going in circles, something has failed more than
once, or a retro needs more rigour than "what went wrong".

```
/dalio-5-step
I keep missing my own deadlines on side projects.
```

**To get value out of it:**

- Give it a real goal, not a tidy one. The skill is built to find the
  uncomfortable root cause; a flattering answer wastes the exercise.
- Expect ~10–15 exchanges. It's an interview, not a prompt.
- Ask for one step if that's all you want — "just diagnose this" runs step 3
  alone.
- Save the worksheet. It asks. Say yes: re-reading last month's diagnosis is
  where the pattern shows up.

**Why I use it:** my instinct is to skip to the plan, which is how I end up
planning around a symptom. Enforcing the order is the whole value, and I won't
enforce it on myself.

## [`people-intelligence`](people-intelligence)

Reads your accumulated notes on one person — 1:1s, transcripts, chat logs, your
own observations — and answers a single question: *given all this, how do I
handle this particular conversation?* Uses Erin Meyer's Culture Map as one lens
among several, and tells you when the evidence is too thin to say.

**Setup:** it needs notes to read. Point it at a `People/` folder next to your
notes, either layout:

```
People/
  Sam Tan/
    Notes.md
    2026-06-14 - Promotion Discussion.md
    Meeting Transcript.md
  Priya.md
```

It never moves or restructures your existing notes — safe to run on an Obsidian
vault with `[[wiki links]]`.

**Use it when** a conversation is going to be hard: bad news, feedback, a
decision they'll dislike, persuading someone who's said no before.

```
/people-intelligence I need to tell Sam his project is being deprioritised — how do I do it
```

**To get value out of it:**

- Name the actual situation, not the person. The default output is advice for
  *that* conversation; ask for a profile only when you want one.
- Write notes as they happen, including the things that went badly. The output
  quality tracks the corpus, and disagreements are the highest-signal entries.
- Read the confidence section. "Insufficient evidence" is a real answer and
  worth more than a confident guess.

**Why I use it:** I remember what people said and forget how they prefer to be
told things. The notes already held the answer — this makes them queryable at
the moment I need it, which is twenty minutes before the meeting.

**One boundary, deliberately:** it won't tell you whether to hire, fire, promote,
pay or discipline someone. It only helps you communicate a decision you've
already made.

## [`interactive-thinking`](interactive-thinking)

Refuses to hand you an analysis. Instead it works out what you're actually
deciding, names what's at stake, makes you give your instinct *before* it gives
its own, simulates the people who'd really be in the room — CEO, principal
engineer, customer, finance — and then attacks the strongest version of your
reasoning until one recommendation survives. Ends in a decision with its
reasoning, rejected alternatives, assumptions, risks and a reversal condition.

**Use it when** you're about to think alone about something consequential: a
decision, a proposal or PRD to approve or reject, a meeting to prepare for, an
idea you like too much to trust.

```
/interactive-thinking Here's the architecture proposal for the new sync layer — I need to decide whether to approve it
```

**To get value out of it:**

- Answer the "what's your gut reaction" question badly rather than not at all.
  Half-formed is fine — the point is to have something to attack.
- Expect to be disagreed with, including on the parts you're most confident
  about. It's built to challenge your strongest reasoning, not your weakest.
- When you notice yourself typing "actually…", follow it. The skill is watching
  for exactly that and will drop its own agenda to chase it.
- Don't ask it to summarise. Summaries are how the session turns back into
  reading.

**Why I use it:** I think noticeably better in a room with other people — when
someone's going to question the reasoning and I have to hold a position out loud
— and noticeably worse alone with a document. This manufactures the room.

**One boundary, deliberately:** it makes thinking sharper, not correct. Fluent
synthesis under pressure still produces confident wrong assumptions, so the
skill separates generating from verifying, and the factual claims still need
checking afterwards.
