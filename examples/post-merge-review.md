# Post-Merge Review

## What this is really doing

There's a gap between "passed review" and "actually complete." Every developer knows this gap exists — it's the acceptance criterion you half-addressed, the edge case mentioned in a comment that you meant to come back to, the documentation you planned to update "after merge."

Most processes treat merge as the end of the quality story. This job treats it as a checkpoint where the *real* question gets asked: **"Did the code actually deliver what the issue asked for?"**

This is the quality ratchet. Without it, small gaps accumulate into debt that nobody tracks because nobody ever formally noticed it. With it, every gap becomes a tracked issue that flows back into triage — and eventually gets fixed.

The subtle power: this job makes it psychologically safe to merge imperfect PRs. If something slips through, the system catches it and creates a follow-up. Perfection isn't required at merge time — completeness is eventually guaranteed by the cycle.

## The skill file

The audit logic lives in a skill file. The cron prompt should be explicit:

```
Execute the post-merge-review skill for the <project> project. Read ~/.openclaw/workspace/skills/post-merge-review/SKILL.md and follow it exactly. Load project config from ~/.openclaw/workspace/memory/projects/<project>.yaml. If no new PRs were audited, respond with exactly NO_REPLY.
```

## The project config file

```yaml
# memory/projects/myproject.yaml
repo: myorg/myproject
gitea_url: https://gitea.example.com
token_path: ~/.mycreds/gitea-bot
audit_lookback_hours: 4
audit_state_file: memory/projects/myproject-audit-state.json
issue_label_gap: post-merge-gap
```

The `audit_state_file` tracks which PRs have already been audited, so each PR is audited exactly once regardless of how many times the job runs.

## The skill file pattern

```markdown
# post-merge-review skill

## Trigger
Message matches: `Execute the post-merge-review skill` with config path

## Steps

1. Load project config
2. Load audit state from `audit_state_file` (create if missing: `{"audited": []}`)
3. Fetch PRs merged in the last `audit_lookback_hours` hours
4. Filter out any PR numbers already in `audited`
5. For each unaudited PR:
   a. Fetch the PR body and linked issue number (look for "Closes #N", "Fixes #N", "Resolves #N")
   b. Fetch the linked issue body (acceptance criteria, requirements)
   c. Fetch the PR diff
   d. Compare: does the diff address every item in the issue?
   e. If gaps exist: file a new issue with label `issue_label_gap`
   f. Add PR number to `audited` list
6. Save updated audit state
7. If no PRs were audited this run: NO_REPLY
8. If PRs were audited but no gaps found: brief summary, no new issues filed
9. If gaps found: report what was filed

## Gap detection — what to look for

A "gap" is any item in the issue that the PR diff does not address:

- Acceptance criteria listed in the issue not implemented in the diff
- Error handling mentioned in the issue not present in the diff
- Test coverage requested in the issue not visible in the diff
- Documentation updates mentioned but not committed

What is NOT a gap:
- Stylistic differences from how you'd personally implement it
- Optimizations the issue didn't ask for
- Things the issue mentioned as "future work" or "out of scope"

When in doubt, file the issue. A false positive creates a tracked conversation. A missed gap creates hidden debt.

## New issue format

Title: `post-merge gap: #<original-pr> — <short description>`

Body:
```
## Gap Found

**Source PR:** #<N> — <PR title>
**Source Issue:** #<M> — <issue title>

## What Was Missed

<specific description of the gap — quote from the original issue>

## Evidence

<quote from the original issue requirement>

The PR diff does not address this because: <explanation>

## Suggested Fix

<brief description of what would close this gap>
```
```

## Set it up

**Step 1: Create your project config** (same file used by dev-loop and triage)

**Step 2: Create your skill file** at `~/.openclaw/workspace/skills/post-merge-review/SKILL.md`

**Step 3: Create the cron job**

Paste into your OpenClaw chat:

> Set up a cron job called "myproject-pr-audit" that runs every hour. Use Sonnet with medium thinking. The prompt should be: "Execute the post-merge-review skill for the myproject project. Read ~/.openclaw/workspace/skills/post-merge-review/SKILL.md and follow it exactly. Load project config from ~/.openclaw/workspace/memory/projects/myproject.yaml. If no new PRs were audited, respond with exactly NO_REPLY." Deliver results to this chat.

**Why these constraints matter:**

- **Sonnet with medium thinking** — Finding gaps requires reading an issue, reading a diff, and comparing them. This is light reasoning work. Medium thinking handles it without the cost of full Opus reasoning.
- **Every hour, not every 4 hours** — Merges happen throughout the day. Catching a gap the same hour it's introduced keeps context fresh for filing a useful issue. Waiting 4 hours means the context is stale.
- **Explicit skill path** — The prompt names the exact skill file. The agent doesn't hunt for it.
- **State file is mandatory** — Without tracking which PRs were audited, the job re-audits the same PRs every run and files duplicate issues. The state file prevents this.
- **`NO_REPLY` contract** — Same as triage: silence means "nothing new." A message means "gaps were found."

## Why every hour

Merges happen a few times a day at most. Running every hour means:
- Each merge is audited within an hour of landing
- The PR author and context are still fresh when the gap issue is filed
- The gap issue flows into triage promptly instead of sitting for hours

If merges are rare, the job is cheap — it runs, finds nothing new to audit, and exits in seconds.

## The state file pattern

The audit state file prevents duplicate issues. It's a simple JSON file:

```json
{
  "audited": [42, 43, 51],
  "last_run": "2026-05-14T04:00:00Z"
}
```

The skill reads this at the start of each run, adds newly audited PR numbers, and writes it back. PR numbers in `audited` are skipped even if they're still within the lookback window.

This means: each PR is audited exactly once. The lookback window just determines which recent PRs to *consider* — the state file determines which ones still need attention.
