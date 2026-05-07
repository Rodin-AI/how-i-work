# Dev Loop

## What this is really doing

The insight here isn't "automate coding." It's **separation of assessment from execution.**

Assessment is cheap. "Are there open PRs? What's their CI status? Is there unaddressed feedback?" — this takes 5 seconds of API calls and costs fractions of a cent. You can afford to do it every 30 minutes forever.

Execution is expensive. Actually reading code, understanding context, writing a fix, running tests — this takes minutes and costs real money. You only want to do it when there's confirmed work to do.

So the dev loop is two agents: a cheap dispatcher that checks state constantly, and an expensive worker that only wakes up when the dispatcher finds something. Most runs (nothing to do), the expensive model never fires. The few runs where there's real work, it gets full power and time.

This is also about **discipline.** The dispatcher can't touch code. It can only look at APIs and spawn a worker. This prevents the failure mode where the dispatcher "just quickly fixes" something and does it badly because it's a small model running with minimal context. Assessment and execution stay cleanly separated.

## Set it up

Paste this into your OpenClaw chat:

> Set up a cron job called "dev-loop" that runs every 30 minutes. Use GPT-4.1 Mini as the dispatcher model with a 660 second timeout. Only allow the tools: exec, sessions_spawn, sessions_yield, subagents.
>
> The prompt should be:
>
> "Dev loop dispatcher. Assess project state via API, spawn ONE worker, exit.
>
> Priority order (first match wins):
> 1. Merge conflicts on open PRs
> 2. CI failures on open PRs
> 3. Unaddressed review feedback
> 4. PRs needing review
> 5. New implementation (only if no open PRs from me)
>
> Steps: Check open PRs via API. Check CI and review state for each. If nothing open, check unassigned issues. Spawn ONE Opus worker (timeout 600s, lightContext) for the highest priority item. Output a one-line summary and exit.
>
> Rules: Never touch code yourself. Spawn exactly one worker. Total time under 60 seconds."
>
> Deliver results to this chat.

Then tell it about your project:

> ...check PRs on github.com/myorg/myproject using `gh pr list`. The worker should clone the repo and run tests with `npm test`.

## Why the priority order matters

The order isn't arbitrary — it's a triage of urgency:

1. **Merge conflicts** block everything downstream. Nothing else matters if the PR can't merge.
2. **CI failures** mean the code is broken right now. Fix it before doing anything else.
3. **Unaddressed feedback** means someone took time to review and is waiting. Respect that.
4. **Needs review** means work is done but stalled — unblock it.
5. **New work** only happens when nothing else is in flight. This enforces WIP=1 naturally.
