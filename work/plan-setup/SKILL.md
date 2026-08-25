---
name: plan-setup
description: Personalise /plan-handover and /plan-implement to this repository — detect the branch, paths, PR conventions and check commands, ask only what can't be detected, and write `.claude/plan-config.md`. Use when either of those skills reports a missing config, when setting them up in a new repo, when the repo's conventions change, or when the user runs /plan-setup.
---

# Plan Setup

`/plan-handover` and `/plan-implement` are written generically. Everything
repo-specific — the tracker, the branch, the paths, the commands — lives in one
file they both read:

```
.claude/plan-config.md
```

This skill writes it. Run once per repo, and again whenever the conventions move.

## 1. Detect first, ask second

Nobody wants a twelve-question form. Work out everything you can, then ask only
about the gaps.

```bash
git symbolic-ref refs/remotes/origin/HEAD          # integration branch
git branch -r | head -20                            # sanity-check it
gh repo view --json nameWithOwner                   # repo slug
ls .github/PULL_REQUEST_TEMPLATE.md .github/pull_request_template.md 2>/dev/null
ls -d docs/decision-notes docs/plans docs/adr 2>/dev/null
ls CLAUDE.md AGENTS.md REVIEW.md CONTRIBUTING.md 2>/dev/null
cat .gitignore | grep -n 'docs/'
```

Then read what you found:

- **Check commands** — `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, and wherever
  this repo defines its tasks: a `Makefile`, a `justfile`, a package manifest's
  scripts section, the language's own runner. Note whether the repo wraps them —
  a `make` target, a script, an environment-entering command — because if it does,
  every check goes through the wrapper, and skipping it means testing a different
  environment from CI.
- **Existing conventions** — if there is already a decision-notes or ADR directory,
  read one entry. Its shape wins over the skill's template.
- **CI** — `.github/workflows/*.yml`. What's actually required for green, and
  anything that writes back to the branch (auto-formatting jobs) or gates on a
  quality service.
- **Recent PR titles** — `gh pr list --state all --limit 20 --json title` shows the
  real prefix convention, which is usually not what anyone documents.

## 2. Ask for the rest

Typically only these can't be detected:

- **Tracker site and project key** — e.g. `acme.atlassian.net`, key `PROJ`.
  (These skills are Jira-shaped: sub-tickets and blocking links. Another tracker
  works, but the user adapts the posting sections themselves.)
- **Which branch PRs target**, if the repo has both a `main` and a `develop`.
- **What reviewers here push back on that CI doesn't catch.** Ask directly. This
  is the highest-value line in the file and it exists nowhere in the repo.
- **Where the plans go**, if `docs/plans/` isn't wanted, and whether they are
  committed or ignored — see step 3. Default to committed; a repo that already
  ignores the directory has answered the question.

Confirm the detected values in the same message rather than one at a time. State
what you found, ask what's missing, write the file.

## 3. Decide whether plans are committed or ignored

The implementation plans are the one artefact with a real choice attached. Ask,
and record the answer in the config — the other two skills behave differently
depending on it.

**Committed (default).** The plans go onto the same draft `[Plan - KEY-123]` PR as
the decision note. Reviewers can read them, they travel to any machine, and
`/plan-implement` can pull them off the branch with `git show`. Nobody attaches
anything by hand. The cost: they name line numbers and internal file paths, so
they go stale in the repo the way any doc does, and they're visible to anyone who
can see the PR.

**Gitignored.** They stay local — nothing about the implementation detail lands in
the repo. The cost is that they exist only on the machine that wrote them:
`/plan-implement` needs them on disk, so building elsewhere means copying the
directory across, and the plan has to be attached to the ticket by hand for anyone
else to read it.

Set up whichever they chose:

- **Committed** — make sure the path is *not* ignored. Check with
  `git check-ignore -v <plans dir>`; if a rule matches, show the user the rule and
  let them decide. Never reach for `git add -f` to get past an ignore.
- **Gitignored** — add the line to `.gitignore`, or a `.gitignore` containing `*`
  inside the directory itself. Say which you did, and say plainly that the plans
  are now machine-local.

## 4. Write the file

Fill in what you found. Delete rows that don't apply — an empty row is worse than
a missing one. The values below are shape only — yours will look nothing like them.

```md
# Plan config

Read by `/plan-handover` and `/plan-implement`. Re-run `/plan-setup` to update.

## Tracker

| Key | Value |
| --- | --- |
| Tracker | Jira |
| Site | `acme.atlassian.net` |
| Project key | `PROJ` |
| Blocking link type | `Blocks` |

## Repo

| Key | Value |
| --- | --- |
| Repo slug | `acme-inc/platform` |
| Integration branch | `develop` |
| Decision notes | `docs/decision-notes/` |
| Plans | `docs/plans/` |
| Plans are | **committed** on the plan PR — or `gitignored, local only` |
| PR template | `.github/PULL_REQUEST_TEMPLATE.md` |
| Review checklist | `REVIEW.md` §6, `AGENTS.md` |

## PR titles

| Kind | Format |
| --- | --- |
| Decision-note PR | `[Plan - PROJ-101] <title>` |
| Implementation PR | `[PROJ-102] <title>` |

## Checks

Run in this order. Every command goes through the wrapper — never the bare binary
on the host.

| Step | Command |
| --- | --- |
| Wrapper | `make` — targets run inside the project environment |
| Format | `make fmt` |
| Static analysis | `make lint PATHS=<changed paths>` |
| Architecture rules | `make arch` |
| Tests | `make test FILTER=<relevant>` |
| Frontend tests | `npm test -- <changed spec files>` |

## Project quirks

Things that have cost a run before. Add to this list as you hit them.

- **One checkout at a time.** The test environment binds to the repo root, so a
  second worktree can't run tests. Parallelism means another machine.
- **The lint workflow auto-commits style fixes to your branch.** After any CI run,
  `git pull` before pushing again or the push is rejected.
- **Duplication gate at ≤3% on new code**, and lines duplicating *pre-existing*
  code count. Copying a sibling class fails it — extract a shared helper.
- **The static-analysis cache goes stale after a dependency change** and reports
  errors that aren't real. `make lint-clean` before trusting a failure.

## Review failures CI can't catch

What reviewers here actually push back on.

- Tenant scoping applied as an optional filter instead of on the base query
- Missing forbidden-path and cross-tenant test coverage
- Behaviour changes — fallback order, defaults, precedence — shipped as a
  "refactor" without being named
- Migrations that ignore existing rows, rollback, or deployment order
```

## Rules

- **Don't invent quirks or review failures.** An empty section the user fills in
  later beats a plausible guess they'll trust. Only write what you found in the
  repo or heard from the user.
- **Don't run the checks to see if they pass.** You're recording the commands, not
  validating the build.
- **Never overwrite an existing config silently.** Show the diff you intend and
  ask.
- Re-running is cheap and expected. When a command or a quirk changes, the fix is
  this file, not the skills.
