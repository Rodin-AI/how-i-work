# Adoption Guide

You've read `how-i-work.md` and want to run the same loops on your own repos. This guide covers the concrete setup.

Before you configure loops, read [The Secret Sauce](the-secret-sauce.md). The loops are the engine; documentation is the fuel. Without the fuel, the loops produce noise instead of quality.

---

## Step 0: Document Before You Automate

The most common adoption failure: setting up all the loops on day one without any documentation in place. The triage gate fires constantly, the agent guesses at conventions, reviews are subjective rather than checkable.

**Before enabling any loop, create at minimum:**

1. **A domain glossary** — What are the core concepts? What do you call them? (Even 20 terms is enough to start.)
2. **A conventions doc** — How is code structured in this repo? Error handling, naming, module layout. Not aspirational — descriptive of what exists today.
3. **One reference pattern** — Pick the most common thing the agent will write (e.g., "add a new API endpoint" or "add a new GenServer") and document how it should look.

This takes 2-4 hours. It pays back immediately because:
- The triage gate has something to check against ("can I implement this using documented facts?")
- Self-review has conventions to verify ("does this match the documented patterns?")
- Twin reviews have criteria beyond subjective taste ("the convention says X, this does Y")

**The test:** If the agent asked "how should I handle errors in this repo?" — could it answer that question by reading your docs? If not, you're not ready for automation.

See [The Secret Sauce: how documentation compounds](the-secret-sauce.md#how-documentation-compounds) for why this investment grows over time rather than being a one-time cost.

---

## Prerequisites

- An AI agent runtime with cron/scheduling (OpenClaw, Hermes, or equivalent)
- Access to your repo (GitHub, GitHub Enterprise, Gitea, etc.)
- Access to your issue tracker (GitHub Issues, Jira, Linear, etc.)
- At least two model providers (for twin review independence)

---

## Step 1: Define Your Parameters

Fill these in before anything else:

```yaml
# config.yaml (conceptual — adapt to your runtime)
repos:
  - org: Rodin-AI
    name: ops-gateway
    host: github.com           # or GitHub Enterprise, Gitea, etc.
    default_branch: main

issue_tracker:
  type: github_issues          # or jira, linear, etc.
  # If Jira: project key + URL
  # project: MYPROJ
  # url: https://jira.yourcompany.com

models:
  dev: claude-sonnet           # cheap, fast, for writing code
  self_review: claude-opus     # expensive, for catching your own mistakes
  twin_a: claude-sonnet        # reviewer 1
  twin_b: gpt-5               # reviewer 2 (different provider = different blind spots)
  dispatch: gpt-4.1-mini      # ultra-cheap, for "is there work?" checks

permissions:
  can_merge: false             # start here. always.
  can_open_prs: true
  can_comment: true
  can_create_issues: true
```

**Start with `can_merge: false`.** The agent opens PRs and hands them to humans. Trust is earned by a track record of clean PRs, not by configuration.

**When to flip `can_merge: true`:**
- The agent has opened 20+ PRs that were merged without significant rework
- Twin reviews consistently find only NITs, not WARNINGs or CRITICALs
- Post-merge review rarely finds gaps (< 1 gap issue per 10 merges)
- You trust the test suite to catch regressions (high coverage on critical paths)
- The human has reviewed enough output to trust the agent's judgment

This is a human decision, not a metric threshold. Some teams never flip it — and that's fine. The system works either way.

---

## Step 2: Implement the Loops

Each loop is an independent cron job. Start with Triage and Dev, add the rest once those are stable.

### Loop 1: Triage (every 20-30 min)

```
Check:
  1. My open PRs — CI status, unaddressed review comments, merge conflicts
  2. Open issues assigned to me — anything new or unblocked?
  3. WIP count — do I already have an open PR?

Output:
  - If WIP > 0 and PR needs attention → fix it (enter Dev loop)
  - If WIP == 0 and unblocked issue exists → start it (enter Dev loop)
  - Otherwise → skip (or enter Free Time)
```

**The critical rule:** WIP ≤ 1. If you already have an open PR, you don't start new work. You finish what's in flight.

### Loop 2: Dev (triggered by Triage or human request)

```
1. Read the issue — understand the PROBLEM, not the title
2. Read surrounding code — understand context, patterns, conventions
3. Plan (for non-trivial work, have a second model critique the plan)
4. Write code + tests
5. Run the full local suite: build, lint, test
6. Commit (conventional format: feat/fix/refactor)
7. Push + open PR
8. Assign to self (WIP signal)
```

**For Go specifically:**
```bash
go build ./...
go vet ./...
golangci-lint run
go test ./... -race -count=1
```

Don't skip `-race`. Don't skip `-count=1` (disables test caching).

### Loop 3: Self-Review (immediately after PR push)

```
1. Switch to a DIFFERENT model than you used for dev
2. Pull your own diff
3. Review for:
   - Error handling gaps (Go: unchecked errors, bare returns)
   - Concurrency issues (goroutine leaks, unprotected shared state)
   - Interface compliance (accept interfaces, return structs)
   - Test coverage (behavior, not implementation)
   - Idiomatic patterns (does it look like the rest of the codebase?)
4. Fix everything found, push again
5. Goal: twin reviewers should find NOTHING
```

### Loop 4: Twin Review (after CI green)

```
1. Two independent models review the diff
2. Each posts findings with severity: CRITICAL / WARNING / MINOR / NIT
3. Triage findings as a batch:
   - CRITICAL/WARNING → fix
   - Theoretical → push back (reply explaining why)
   - NIT → fix if cheap
4. One push with all fixes
5. Wait for CI green
```

**Why two models?** They have different blind spots. GPT-5 catches self-contradictions and attack chains. Sonnet/Opus catch cross-document consistency and pattern deviations. Neither alone is sufficient.

**Bot accounts for reviews:** Ideally, each reviewer posts as its own account (e.g., `sonnet-review`, `gpt-review`). This makes it clear who said what and allows re-requesting reviews independently.

**⚠️ Enterprise GitHub note:** On GitHub Enterprise, creating bot accounts (machine users) often requires IT approval, security review, or a service account request process that takes days-to-weeks. Plan for this early — it's a common adoption blocker. Options if blocked:
- Run both reviewers as sub-agents posting from your own account (prefix findings with the model name)
- Use a single service account for both, with clearly labeled sections
- Request one "CI bot" account and multiplex both reviewers through it

Don't let the bot account problem block your entire setup. Start with sub-agents, migrate to dedicated accounts when IT approves them.

### Loop 5: Handoff

```
When: CI green + twins done + all findings addressed
Do: Reassign PR to human (or apply a "ready" label)
Signal: No chat message. The assignment IS the notification.
```

### Loop 6: Post-Merge Review (every 4 hours)

```
For each PR merged since last check:
  1. Find the linked issue
  2. Read acceptance criteria
  3. Diff: did the code actually deliver each criterion?
  4. If gaps exist → file a new issue describing what's missing
```

This is the quality ratchet. It catches "good enough" that wasn't actually good enough.

### Loop 7: Lookback (every 3 days)

```
For PRs you reviewed that have since been merged:
  1. Pull the final diff (merged code vs code at review time)
  2. What changed because of your review? (impact)
  3. What did humans catch that you missed? (gaps)
  4. What did you flag that nobody acted on? (noise)
  5. Score yourself honestly
  6. If a pattern emerges → update review prompts
```

**This is the most important loop.** Without it, the system drifts toward noise. Reviews that don't cause code to improve are just wasted tokens.

**⚠️ Critical constraint:** Lookback *recommends* prompt and pattern changes — it never applies them autonomously. Self-modification of review criteria requires human approval. See [the scaling doc's failure modes section](scaling-multiple-repos.md#lookback-self-modification-risk) for why.

---

## Step 3: The Dispatch Pattern

Every cron loop should use a cheap model to CHECK whether there's work, and only invoke an expensive model to DO the work.

```
Dispatch (gpt-4.1-mini, ~0.1s, ~$0.001):
  "Are there open PRs with failing CI from me?"
  → Yes/No

If yes → spawn expensive worker (opus/gpt-5, ~30s, ~$0.10):
  "Here's the CI failure log. Fix it."
```

This keeps costs sane. Most cron ticks find nothing to do — don't burn $0.10 per tick discovering that.

---

## Step 4: Jira Integration (if applicable)

For teams using Jira instead of GitHub Issues:

```
Triage reads from:
  - Jira (JQL): project = MYPROJ AND assignee = currentUser() AND status = "To Do"
  - GitHub: open PRs from your bot account

Post-merge review files issues to:
  - Jira (create issue via API): type = Bug, project = MYPROJ

Link PRs to issues:
  - Commit message: "fix(MYPROJ-123): description"
  - Or PR description: "Closes MYPROJ-123"
```

---

## Step 5: Safety Rails

Non-negotiable regardless of setup:

| Rule | Why |
|------|-----|
| Never merge without human approval | Trust is earned |
| Never push to default branch | Always branch + PR |
| Never skip tests | "It's just a small change" is how bugs ship |
| Never dismiss flaky tests | Flaky = real problem, file an issue |
| Never workaround CI in feature branches | CI issues get their own fix |
| Fail toward escalation | If confused → ask human, don't guess |

---

## Step 6: Adapting for Your Language

The loops are language-agnostic. The tooling inside them isn't.

### Go
```bash
# Dev loop local checks
go build ./...
go vet ./...
golangci-lint run
go test ./... -race -count=1

# Review focus areas
- Unchecked errors (the #1 Go bug class)
- Goroutine leaks (context cancellation)
- Interface satisfaction (compile-time, but check for unnecessary interfaces)
- Package boundaries (internal/ for private, avoid wide exports)
```

### Elixir
```bash
mix format --check-formatted
mix compile --warnings-as-errors
mix test
mix credo --strict

# Review focus areas
- GenServer state design (what's in state vs what's derived?)
- Supervision tree structure (restart strategies)
- Pattern match completeness (especially in handle_info)
- Typespec coverage on public functions
```

### Rust
```bash
cargo build
cargo clippy -- -D warnings
cargo test
cargo fmt --check

# Review focus areas
- Lifetime annotations (unnecessary complexity?)
- Error type design (thiserror vs anyhow at boundaries)
- Unsafe blocks (justify every single one)
- Send/Sync bounds (accidental !Send in async code)
```

---

## Common Pitfalls

**"I'll add lookback later."**
No. Start it from day one, even if you have zero data. The habit of asking "did my review matter?" is more important than the scoring math.

**"I'll review everything with the best model."**
You'll burn your budget in a week. Cheap dispatch + expensive work. Most ticks are idle.

**"Twin reviews agree on everything."**
That's a signal something is broken. If two models always agree, one of them isn't adding value. Check that they're actually different providers/architectures, and that the prompts aren't accidentally leading them to the same conclusions.

**"The bot opened 15 PRs while I was at lunch."**
WIP limit exists for a reason. One PR at a time. The agent waits for handoff before starting new work. If you're tempted to raise the limit — don't. Parallel PRs = exponential rebases.

**"Post-merge review files too many issues."**
Calibrate what counts as a "gap." Missing a NIT from acceptance criteria isn't worth an issue. Missing core functionality is. The threshold should be: "would a user notice this is missing?"

---

## Minimal Viable Setup

If you want to start with the absolute minimum and expand:

1. **Week 1:** Triage + Dev loops only. Agent picks up issues, opens PRs, assigns to human.
2. **Week 2:** Add Self-Review. Agent reviews its own PR before marking ready.
3. **Week 3:** Add Twin Review. Two models review every PR.
4. **Week 4:** Add Post-Merge Review. Audit merged work for completeness.
5. **Ongoing:** Add Lookback once you have 5+ reviewed PRs to analyze.

Each step builds on the last. Don't try to run all 7 loops on day one.

---

## The One Thing That Matters Most

The system works because every loop feeds the next one. Work creates reviews, reviews create issues, issues create work. Quality ratchets up over time because:

1. Nothing ships without review (twin review)
2. Nothing stays shipped without audit (post-merge review)
3. Nothing stays broken without a ticket (issue filing)
4. Nothing stays noisy without pruning (lookback)

If you only implement one thing beyond basic dev work, implement **post-merge review**. It's the loop that catches "good enough" before it becomes technical debt.

---

## Further reading

| Document | What it covers |
|----------|----------------|
| [The Secret Sauce](the-secret-sauce.md) | Why documentation drives the system, reference docs, the cascade discipline, failure tensions |
| [Scaling to Multiple Repos](scaling-multiple-repos.md) | Global dispatcher, WIP rules, failure modes and mitigations |
