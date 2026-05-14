# Triage

## What this is really doing

Work doesn't stall dramatically. It stalls silently. A test failed at 2am and nobody noticed. A reviewer left findings six hours ago and the PR is just sitting there. A merge conflict appeared when another branch landed and now the PR can't merge even though everything else is green.

This job's purpose is to make silence impossible. Every 30 minutes, it asks: **"Is anything stuck that shouldn't be?"** Not "what should I work on" — that's the dev loop's job. Triage just shines a light on things that have gone quiet when they should be moving.

The distinction matters. A triage job that tries to *fix* things is doing too much. Its only job is to *see* — to turn invisible stalls into visible status that other loops can act on.

## The skill file

Triage logic lives in a skill file. The cron prompt should be one line:

```
Run the project-triage skill. Read ~/.openclaw/workspace/skills/project-triage/SKILL.md and follow it exactly. Load project config from ~/.openclaw/workspace/memory/projects/<project>.yaml. If nothing to report, respond with exactly NO_REPLY.
```

The skill file drives everything. This separation means you can improve your triage logic without editing cron jobs.

## The project config file

```yaml
# memory/projects/myproject.yaml
repo: myorg/myproject
gitea_url: https://gitea.example.com
token_path: ~/.mycreds/gitea-bot
wip_limit: 1
stale_review_hours: 24
stale_ci_hours: 2
labels:
  ready: 42
  wip: 40
```

## The skill file pattern

```markdown
# project-triage skill

## Trigger
Message matches: `Run the project-triage skill` with a config path

## Steps

1. Load project config
2. Fetch all open PRs via API
3. For each open PR, check:
   - CI status (pending/failed/missing)
   - Review state (REQUEST_CHANGES present? Reviews stale?)
   - Merge conflict status
   - Age since last update
4. Check issue queue: any new unassigned implementation issues?
5. Check WIP: more than wip_limit open PRs from bot account?
6. Report anything that qualifies as "stuck" (see thresholds below)
7. If nothing stuck: NO_REPLY

## Stuck thresholds (use config values, fall back to these defaults)

- CI pending or failed for > 2 hours
- REQUEST_CHANGES review not addressed for > 24 hours
- Merge conflict present (any age)
- PR has `ready` label but CI is not green
- More open PRs than wip_limit

## Output format (when there IS something to report)

One bullet per item. Be specific:
- "PR #42: CI failing for 3h (last commit: abc1234)"
- "PR #38: REQUEST_CHANGES from @reviewer not addressed (18h ago)"
- "PR #51: merge conflict present"

Do NOT say "everything looks good" before the list. If there's nothing, NO_REPLY.
Do NOT include items that are fine — only report problems.
```

## Set it up

**Step 1: Create your project config** (same file used by dev-loop — one config per project)

**Step 2: Create your skill file** at `~/.openclaw/workspace/skills/project-triage/SKILL.md`

**Step 3: Create the cron job**

Paste into your OpenClaw chat:

> Set up a cron job called "myproject-triage" that runs every 30 minutes. Use Sonnet with medium thinking and a 120 second timeout. The prompt should be: "Run the project-triage skill. Read ~/.openclaw/workspace/skills/project-triage/SKILL.md and follow it exactly. Load project config from ~/.openclaw/workspace/memory/projects/myproject.yaml. If nothing to report, respond with exactly NO_REPLY." Deliver results to this chat.

**Why these constraints matter:**

- **Sonnet, not Opus** — Triage is pattern-matching: is CI green, is there a review, is there a conflict. These are yes/no checks. You don't need deep reasoning. Sonnet is fast and cheap and plenty capable.
- **120s timeout** — Triage should complete in seconds. If it's taking longer, it's doing too much.
- **`NO_REPLY` contract is mandatory** — Without it, you get a message every 30 minutes saying "everything looks good." That's noise that trains you to ignore the channel. The model must know to respond with exactly `NO_REPLY` when there's nothing to report, and your runtime must handle that as "no delivery."
- **Explicit skill path** — Don't say "check my PRs." Say exactly which skill file to read and which config to load. Vague instructions get interpreted differently on each run.

## Why Sonnet, not a cheaper model

Triage reads API responses and makes boolean judgments. The failure modes of cheaper models here are:

- Missing a finding that's clearly present in the API data
- Misreading a timestamp (is 3 hours ago > 2 hours? Yes, but some models fumble this)
- Hallucinating a status that isn't in the API response

Sonnet is reliable enough on structured data that false negatives are rare. Go cheaper and you'll start missing real problems.

## The NO_REPLY contract

This is the most important implementation detail.

Without it: you get a notification every 30 minutes. Most say "nothing to report." You learn to ignore the channel. The one time something is actually wrong, you ignore that too.

With it: the channel is silent until there's a real problem. Silence means "all good." A message means "something needs attention." That's the right information density.

Every triage job must have this contract. Configure your runtime to treat `NO_REPLY` as "don't deliver this." Then tell the model: "If nothing to report, respond with exactly NO_REPLY."
