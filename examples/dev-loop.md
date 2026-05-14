# Dev Loop

## What this is really doing

The insight here isn't "automate coding." It's **separation of assessment from execution.**

Assessment is cheap. "Are there open PRs? What's their CI status? Is there unaddressed feedback?" — this takes 5 seconds of API calls and costs fractions of a cent. You can afford to do it every 30 minutes forever.

Execution is expensive. Actually reading code, understanding context, writing a fix, running tests — this takes minutes and costs real money. You only want to do it when there's confirmed work to do.

So the dev loop is two agents: a cheap dispatcher that checks state constantly, and an expensive worker that only wakes up when the dispatcher finds something. Most runs (nothing to do), the expensive model never fires. The few runs where there's real work, it gets full power and time.

This is also about **discipline.** The dispatcher cannot touch code. It can only look at APIs and spawn a worker. This prevents the failure mode where the dispatcher "just quickly fixes" something and does it badly because it's a small model running with minimal context. Assessment and execution stay cleanly separated.

## The dispatcher / worker split

```
Dispatcher (cheap, always running)
  → reads APIs: PR state, CI status, reviews, issue list
  → decides: is there work? what's highest priority?
  → if yes: spawns ONE worker session, then exits
  → if no: exits immediately (NO_REPLY)

Worker (expensive, runs only when needed)
  → receives: specific task description from dispatcher
  → does full dev loop: understand → plan → code → test → push
  → runs with full context and long timeout
```

Never give the dispatcher tools that allow it to write code or push commits. If it has those tools, it will use them — and a fast cheap model doing "quick fixes" is how you get subtle bugs at 3am.

## Before you start

Read [skill-files.md](skill-files.md) and [project-config.md](project-config.md) first — they explain the two structural patterns this loop relies on. This doc only covers what's unique to the dev loop.

Your dispatcher cron prompt should be exactly this:
```
dev-loop <project-name>
```

And your skill file handles everything. See the [skill pattern](#the-skill-file-pattern) below.

## The skill file pattern

```markdown
# dev-loop skill

## Trigger
Message matches: `dev-loop <project>`

## Steps

1. Load project config from `memory/projects/<project>.yaml`
2. Check open PRs via API — for each, check: CI status, review state, merge conflicts
3. If open PR exists from bot account: follow priority order below for that PR
4. If no open PRs: check for unassigned implementation issues
5. Spawn ONE worker for highest-priority item (see worker prompt below)
6. Exit with one-line summary

## Priority order (first match wins)

1. Merge conflict on open PR
2. CI failure on open PR
3. Unaddressed REQUEST_CHANGES on open PR
4. Open PR needing review
5. New issue implementation (only if WIP = 0)

## Worker prompt

Pass this to the spawned worker session:

"""
You are working on <project>. Your task: <specific task from dispatcher>.

Read AGENTS.md before writing any code.
Read the pre-code skill and write a plan before any implementation.
Run the full test suite before pushing.
Run the self-review skill before marking ready.
Create PR using the bot token at <token_path>.
"""

## Rules

- **Spawn exactly ONE worker per run. Never two.**
  Two workers running simultaneously will both pick the highest-priority item and produce conflicting PRs. The second worker's work is wasted and the merge conflict cleanup costs more than the work saved.

- **If nothing qualifies: NO_REPLY. Do not invent work.**
  A dispatcher that always finds *something* to do is manufacturing work. Idle is correct when there's nothing to act on.

- **Do not write code yourself. You are assessment only.**
  The dispatcher is a fast, cheap model running with restricted context. Code written here won't have the full context needed to be correct. That's what the worker is for.

- **Do not push commits, open PRs, or modify files in any repo.**
  Same reason as above — enforced at the action level so there's no temptation to "just quickly fix" something.

- **Do not make any write API calls. No creating issues, labels, or comments.**
  The dispatcher's job is to read state and decide. Any write action it takes is outside its mandate and may interfere with the worker it's about to spawn.

- **Total dispatcher runtime must be under 60 seconds.**
  If the dispatcher is taking longer than 60 seconds, it's doing too much. The API calls it needs are fast. Long runtimes signal scope creep or a hung request that should be skipped.

## Worker rules

Include these in the worker prompt:

- **Do not start a second task if the first produces a PR. One PR per worker run.**
  Two open PRs from one run means two things to review, rebase, and merge. They will conflict. WIP=1 is the invariant; a single worker run must not break it.

- **Do not push directly to the default branch. All changes go through a PR.**
  Direct pushes bypass CI, bypass review, and cannot be reverted cleanly. A PR is the contract — always.

- **Do not modify files outside the target repo.**
  Scope containment. A worker that edits config files, shared libraries, or workspace files outside its assigned repo is operating outside its mandate and may break unrelated work.

- **Do not mark the PR ready or assign to the human until self-review passes.**
  The human's queue should contain only clean, finished work. An unreviewed PR in their queue wastes their time and erodes trust in the system.

- **If the task is blocked (missing context, ambiguous requirements), open an issue asking for clarification. Do not guess.**
  A guess that ships as a PR costs more to fix than an issue asking a clarifying question. Guessing optimizes for appearing productive over being correct.
```

## Set it up

**Step 1: Create your project config** — see [project-config.md](project-config.md) for the full field reference. Fields used by this loop: `repo`, `token_path`, `gitea_url`, `exec_node`, `branch_default`, `wip_limit`, `labels`.

**Step 2: Create your skill file** at `~/.openclaw/workspace/skills/dev-loop/SKILL.md` using the pattern above. Customize the worker prompt with your project's test command, PR conventions, and standing rules.

**Step 3: Create the cron job** — see [cron-setup.md](cron-setup.md) for the full setup guide. For this loop:

> Set up a cron job called "myproject-dev-loop" that runs every 30 minutes. Use a fast model (GPT-4.1 Mini or Haiku) as the dispatcher with a 300 second timeout. Only allow the tools: exec, sessions_spawn, sessions_yield, subagents. The prompt should be exactly: `dev-loop myproject`. Deliver results to this chat only when a worker is spawned.

## Why the priority order matters

The order isn't arbitrary — it's a triage of urgency:

1. **Merge conflicts** block everything downstream. Nothing else matters if the PR can't merge.
2. **CI failures** mean the code is broken right now. Fix it before doing anything else.
3. **Unaddressed feedback** means someone took time to review and is waiting. Respect that.
4. **Needs review** means work is done but stalled — unblock it.
5. **New work** only happens when nothing else is in flight. This enforces WIP=1 naturally.

## The failure modes this prevents

**"Quick fix" drift** — Dispatchers with code-writing tools start doing fixes themselves. They're fast and cheap, so the fix ships in 20 seconds. It also misses the edge case that a proper worker with full context would have caught. Restricting tools prevents this.

**Parallel work explosion** — Without WIP=1 enforcement, the dispatcher spawns a worker for every open issue and you end up with five in-flight PRs that all conflict. The priority order + WIP check prevents this.

**Expensive idle polling** — Without the dispatcher/worker split, you're running an expensive model every 30 minutes even when there's nothing to do. 95% of those runs find nothing. The dispatcher run costs cents; the worker run costs dollars. Only fire the worker when there's confirmed work.
