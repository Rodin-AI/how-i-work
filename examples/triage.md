# Triage

## What this is really doing

Work doesn't stall dramatically. It stalls silently. A test failed at 2am and nobody noticed. A reviewer left findings six hours ago and the PR is just sitting there. A merge conflict appeared when another branch landed and now the PR can't merge even though everything else is green.

This job's purpose is to make silence impossible. Every 30 minutes, it asks: **"Is anything stuck that shouldn't be?"** Not "what should I work on" — that's the dev loop's job. Triage just shines a light on things that have gone quiet when they should be moving.

The distinction matters. A triage job that tries to *fix* things is doing too much. Its only job is to *see* — to turn invisible stalls into visible status that other loops can act on.

## Before you start

Read [skill-files.md](skill-files.md) and [project-config.md](project-config.md) first — they explain the two structural patterns this loop relies on. This doc only covers what's unique to triage.

Your cron prompt:
```
Run the project-triage skill. Read ~/.openclaw/workspace/skills/project-triage/SKILL.md and follow it exactly. Load project config from ~/.openclaw/workspace/memory/projects/<project>.yaml. If nothing to report, respond with exactly NO_REPLY.
```

## The skill file pattern

```markdown
# project-triage skill

## Trigger
Message matches: `Run the project-triage skill` with a config path

## Steps

1. Load project config
2. **Load domain docs** (design docs, CLAUDE.md, validation template, regulations) — these are the authority on how the system must behave
3. Fetch all open issues (excluding blocked/needs-split/needs-detail)
4. For each issue — evaluate against domain docs:
   - Body empty or no problem statement → `needs-detail`
   - Conflicts with domain docs/regulations → `needs-detail` + comment with specific conflict
   - Ambiguous and docs don't resolve it → `needs-detail` + flag for human decision
   - Docs resolve the ambiguity → enrich issue with context comment, no label change
   - Clear and consistent → no action
   The dev loop will not pick up a `needs-detail` issue.
5. Fetch all open PRs via API
6. For each open PR, check:
   - CI status (pending/failed/missing)
   - Review state (REQUEST_CHANGES present? Reviews stale?)
   - Merge conflict status
   - Age since last update
7. Check issue queue: any new unassigned implementation issues?
8. Check WIP: more than wip_limit open PRs from bot account?
9. Report anything that qualifies as "stuck" (see thresholds below)
10. If nothing stuck: NO_REPLY

## Stuck thresholds (use config values, fall back to these defaults)

- Issue missing problem statement
- Issue conflicts with domain docs (flag with explanation)
- Issue ambiguous against docs (flag for human)
- CI pending or failed for > 2 hours
- REQUEST_CHANGES review not addressed for > 24 hours
- Merge conflict present (any age)
- PR has `ready` label but CI is not green
- More open PRs than wip_limit

## Output format (when there IS something to report)

One bullet per item. Be specific:
- "Issue #12: needs detail (empty body)"
- "Issue #15: needs human decision — docs don't specify how X should behave when Y"
- "Issue #18: conflicts with trading regulation Z — proposes double-fill which is prohibited"
- "PR #42: CI failing for 3h (last commit: abc1234)"
- "PR #38: REQUEST_CHANGES from @reviewer not addressed (18h ago)"
- "PR #51: merge conflict present"

Do NOT say "everything looks good" before the list. If there's nothing, NO_REPLY.
Do NOT include items that are fine — only report problems.

## Rules

- **Do not attempt to fix anything. Observe and report only.**
  Triage that fixes things is doing two jobs at once — neither reliably. The dev loop is responsible for fixes. If triage starts patching CI configs or applying labels, it will sometimes make things worse and always obscure what actually happened.

- **Do not make any write API calls. No creating issues, labels, comments, or status changes.**
  Write access is the mechanism by which scope creep happens. A read-only triage job cannot accidentally close a PR, apply the wrong label, or spam a thread. Remove the temptation entirely.

- **Do not report the same item twice in one run.**
  A PR can satisfy multiple stuck criteria simultaneously (failing CI *and* stale review). Reporting it twice makes the output harder to read and implies more problems than exist.

- **Do not report items outside the configured repo.**
  Without this constraint, a triage job with broad API access will start pulling in PRs from other repos it can see. The report becomes noise.

- **Do not speculate about why something is stuck. Report facts only: what is stuck and for how long.**
  "CI is probably failing because of the flaky test" is a guess. It may be wrong and it trains the human to dismiss triage output as opinion rather than signal. Stick to what the API returns.

- **If the API returns an error for one PR, skip it and continue. Do not abort the entire run.**
  A single 404 or timeout should not silence the entire triage report. Other PRs may have real problems. Log the error, skip that PR, continue.
```

## Set it up

**Step 1: Create your project config** — see [project-config.md](project-config.md). Fields used by this loop: `repo`, `token_path`, `gitea_url`, `wip_limit`, `stale_review_hours`, `stale_ci_hours`, `labels`.

**Step 2: Create your skill file** at `~/.openclaw/workspace/skills/project-triage/SKILL.md` using the pattern above.

**Step 3: Create the cron job** — see [cron-setup.md](cron-setup.md) for the full guide. For this loop:

> Set up a cron job called "myproject-triage" that runs every 30 minutes. Use Sonnet with medium thinking and a 120 second timeout. The prompt should be: "Run the project-triage skill. Read ~/.openclaw/workspace/skills/project-triage/SKILL.md and follow it exactly. Load project config from ~/.openclaw/workspace/memory/projects/myproject.yaml. If nothing to report, respond with exactly NO_REPLY." Deliver results to this chat.

## Why Opus with high thinking, not a fast model

Triage now does two distinct kinds of work:

**Structured data checks** (CI status, timestamps, label presence) — a fast model handles this fine.

**Domain reasoning** (does this issue conflict with our regulations? is this ambiguous or does the doc resolve it? what question does the human need to answer?) — this requires genuine reasoning. The failure modes of a fast model here are:

- Confidently resolving ambiguity that should be escalated to a human
- Missing a conflict between an issue and a domain invariant
- Treating "I don't know" as "looks fine"
- Hallucinating resolution from training data instead of the actual domain docs

The cost of a wrong call here is high: a bad issue enters the dev loop, produces a design doc built on false assumptions, and the post-merge review files the resulting bugs. Opus with high thinking makes that failure mode rare enough to trust.

