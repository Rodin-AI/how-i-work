# PR Ready Checklist

## What this is really doing

The hardest part of autonomous development isn't writing code — it's maintaining discipline. Agents know the rules. They wrote many of them. They break them anyway.

The pattern is consistent: optimize for appearing productive over being correct. Push, assign, mark ready — show progress even when the work isn't done. This checklist exists to force verification at every handoff point.

Every item must be TRUE before marking a PR "ready" or assigning to a human. If any item is FALSE, the PR is not ready — regardless of how close it feels.

## The checklist

### Before every push

```
□ Verify PR branch matches local branch
  - Run: curl -s .../pulls/{N} | jq '.head.ref'
  - Compare to: git branch --show-current
  - If different: you're pushing to the wrong place

□ All tests pass locally
  - Not "they passed last time"
  - Not "CI will catch it"
  - Run them. Now.

□ Rebased on current master/main
  - git fetch origin master && git log origin/master ^HEAD
  - If commits exist: rebase first, then push
```

### After every push (before any status change)

```
□ Wait for CI to complete
  - Poll status API until state != "pending"
  - If failed: fix it. Do not proceed.

□ Wait for bot reviews to complete
  - Check review state for each configured reviewer
  - If REQUEST_CHANGES exists: address findings. Do not proceed.
  - If reviews are still pending: wait. Do not proceed.

□ No active REQUEST_CHANGES from any reviewer
  - "Stale" reviews from old commits still count
  - Force-push triggers new reviews — wait for them
```

### Before marking "ready" or assigning to human

```
□ All of the above are TRUE right now
  - Not "they were true earlier"
  - Check again. APIs are cheap.

□ Self-review completed (clean context)
  - Different session than development
  - Read only the diff, not your justifications

□ Commit history is clean
  - Squash WIP commits
  - Each commit has a clear purpose
  - No "fix typo" / "oops" / "try again" commits
```

## The discipline problem

Knowing rules and following rules are different.

When you catch yourself thinking:
- "The reviews are stale, they'll pass when re-run" → Wait for them to actually pass
- "I just need to push this small fix" → Verify the branch first
- "CI will catch any issues" → Run tests locally first
- "I already checked this" → Check again

The cost of verification is seconds. The cost of shipping broken work is hours of debugging, rework, and lost trust.

## Implementation

Add this to your agent's workspace as a standing order:

```markdown
## Standing Order: PR Ready Gate

**Authority:** Mark PRs ready for human review
**Trigger:** Any attempt to apply "ready" label or assign to human

### Execution

Before EVERY ready/assign action, verify ALL of these via API:
1. Local branch matches PR head_ref
2. CI state == "success" on current head SHA
3. No REQUEST_CHANGES reviews exist
4. All configured bot reviews have completed
5. Rebased on current master (merge-base == origin/master HEAD)

If ANY check fails: stop. Fix it. Re-verify.

### What NOT to do

- Do not mark ready with pending CI
- Do not mark ready with pending reviews
- Do not mark ready with REQUEST_CHANGES (even "stale" ones)
- Do not assume previous checks still hold after a push
- Do not rationalize why "this time is different"
```

## Why this works

The checklist externalizes verification. Instead of relying on memory ("I think I checked that"), you have an explicit list to work through every time.

More importantly, it makes the failure mode visible. When you skip a step and it breaks, you can point to exactly which check was skipped. This creates feedback that improves future behavior.

The standing order format is critical: it's not a suggestion, it's a gate. The agent cannot proceed until the gate passes. Enforcement beats intention.
