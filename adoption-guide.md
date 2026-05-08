# Adoption Guide

You've read the [README](README.md) and want to run the same loops on your own repos. This guide covers the concrete setup.

Before you configure loops, read [The Secret Sauce](the-secret-sauce.md). The loops are the engine; documentation is the fuel. Without the fuel, the loops produce noise instead of quality.

---

## Before you start: what you need and what it costs

If you're new to AI-assisted development, here's what the pieces are and how to get them.

### What is an AI model?

An AI model (like GPT-5 or Claude) is a service you call over the internet. You send it text ("review this code"), it sends back text ("here are three issues I found"). You access it through an API — which means you need an account with the provider and an API key (a password-like string that identifies you).

The major providers:

| Provider | Models | Sign up | Pricing (per million tokens) |
|----------|--------|---------|--------|
| [OpenAI](https://platform.openai.com) | GPT-4.1, GPT-5 | platform.openai.com | Input $1.25–$2, Output $8–$10 |
| [Anthropic](https://console.anthropic.com) | Claude Sonnet, Claude Opus | console.anthropic.com | Input $3–$5, Output $15–$25 |
| [Google](https://ai.google.dev) | Gemini 2.5 Pro, Gemini Flash | ai.google.dev | Input $0.15–$1.25, Output $0.60–$10 |

*A "token" is roughly 3/4 of a word. A typical code review of a 200-line PR uses 10,000–50,000 input tokens and 1,000–5,000 output tokens. A review with an expensive model (Opus at $5/$25) costs roughly $0.10–$0.40. With a cheaper model (GPT-4.1 at $2/$8), it's $0.03–$0.06.*

**Why two providers?** The twin review system uses two different models so they have different blind spots. If both came from the same provider, they'd share the same weaknesses and miss the same things.

### What is an agent runtime?

An agent runtime is software that lets an AI model DO things — not just talk. It gives the model access to tools: running shell commands (git, compilers, linters), reading/writing files, calling APIs (GitHub, Jira). Without a runtime, the model can only generate text. With one, it can actually write code, run tests, and open PRs.

The runtime also handles **scheduling** — running the model on a timer ("check for CI failures every 30 minutes"). This is what "cron" means throughout these docs: a scheduled task that runs automatically at an interval.

Options:

| Runtime | What it is | Get started |
|---------|-----------|-------------|
| [OpenClaw](https://openclaw.ai) | Self-hosted agent platform. Built-in scheduling, tool access, persistent workspace. What this system was built on. | Install on a Linux machine. Free, open source. |
| [Hermes](https://github.com/socrates-ai/hermes) | Temporal-based agent runtime. Scheduling + tool use with workflow orchestration. | Self-hosted. Requires Temporal cluster. |
| GitHub Actions (self-hosted runner) | Use GitHub's `schedule:` triggers to run prompts on a timer. The runner machine is the workspace. | Free if you have a machine. Cheapest starting point. |
| Any agent framework with timers | LangGraph, CrewAI, AutoGen, etc. | Varies. You build the scheduling + tool glue yourself. |

**You don't need to pick the "right" one to start.** The loops are prompt patterns + scheduling. If your system can run a prompt on a timer with access to git and a GitHub API token, it works.

### What does this cost to run?

Rough monthly costs for a single active repo:

| Component | Cost | Notes |
|-----------|------|-------|
| Triage (cheap model, every 30 min) | ~$3–8 | Most ticks find nothing — GPT-4.1 Mini at $0.40/$1.60 per MTok |
| Dev loop (expensive model, triggered) | ~$15–40 | Opus-class model, depends on PR volume |
| Twin reviews (2 models per PR) | ~$5–20 | ~$0.20–$0.80 per review × 2 models |
| Self-review (expensive model) | ~$5–15 | One review per PR, Opus-class |
| Post-merge audit | ~$3–8 | Every 4 hours, usually finds nothing |
| **Total** | **~$30–90/month** | For an active repo with 5–10 PRs/week |

For comparison: one senior engineer costs $15,000–25,000/month. This system doesn't replace them — it removes the grunt work so they can focus on design and decisions.

### Minimum viable setup (cheapest possible)

1. **One model provider account** (OpenAI or Anthropic, ~$20 prepaid credits)
2. **A Linux machine** (your laptop, a $5/month VPS, or a spare Raspberry Pi)
3. **An agent runtime** (OpenClaw is free, or just use `cron` + a CLI agent)
4. **A GitHub/Gitea/GitLab account** with API access to your repo

You can start with just triage + dev loop (two cron jobs, one cheap model, one expensive model) for under $25/month.

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

### The documentation sufficiency checklist

Before enabling loops, run through these prompts. For each one, try to answer using ONLY what's written in your repo's docs. If you can't — that's a gap to fill.

**Domain understanding:**
- [ ] "What are the 10 most important concepts in this system and how do they relate?"
- [ ] "What's the difference between [concept A] and [concept B]?" (pick two that are easy to confuse)
- [ ] "What happens when [common operation] fails halfway through?"
- [ ] "Which parts of the system own which responsibilities?" (bounded contexts)
- [ ] "What invariants must ALWAYS hold, regardless of implementation?"

**Conventions:**
- [ ] "Show me the correct way to add a new [most common unit of work — endpoint, module, handler, etc.]"
- [ ] "How does error handling work here? What gets rescued vs propagated vs logged?"
- [ ] "What's the testing strategy? Unit vs integration vs e2e boundaries?"
- [ ] "Where does validation live? Input boundary? Domain layer? Both?"
- [ ] "What naming conventions exist? (files, modules, functions, variables, DB tables)"

**Architecture:**
- [ ] "Why is the system structured this way and not [obvious alternative]?"
- [ ] "What are the trust boundaries? What talks to what, and who validates at each boundary?"
- [ ] "If I need to add a feature that crosses two subsystems, what's the integration pattern?"
- [ ] "What's shared and what's private? Where are the fences?"

**Process:**
- [ ] "What does 'done' mean for a PR in this repo?"
- [ ] "What are the non-negotiable quality checks before merge?"
- [ ] "What's in scope for the agent vs what requires human decision?"

**Scoring:**
- ✅ Can answer from docs alone → Ready for automation in that area
- ⚠️ Can answer but it's in someone's head, not written down → Document it first
- ❌ Can't answer at all → Need a conversation to figure it out, then document

You don't need 100% coverage to start. But you need the **conventions** and **process** sections mostly green. Domain gaps are okay early — the triage gate will flag them as you go. Convention gaps are not okay — they cause every PR to drift in a different direction.

### Using these prompts as a gap-finding tool

These aren't just a one-time checklist. Run them periodically (monthly, or when triage flags increase). Copy-paste this entire prompt to your agent:

```
Read all documentation in this repo. Then answer the following
questions using ONLY what's documented. For each question, respond
with one of:
  - ANSWERED: [your answer, citing the source file]
  - PARTIAL: [what you can answer + what's missing]
  - UNDOCUMENTED: [your best guess + why you're not confident]

Domain understanding:
1. What are the 10 most important concepts in this system and how do they relate?
2. What's the difference between [concept A] and [concept B]? (pick two that are easy to confuse)
3. What happens when [common operation] fails halfway through?
4. Which parts of the system own which responsibilities?
5. What invariants must ALWAYS hold, regardless of implementation?

Conventions:
6. Show me the correct way to add a new [most common unit of work].
7. How does error handling work here? What gets rescued vs propagated vs logged?
8. What's the testing strategy? Unit vs integration vs e2e boundaries?
9. Where does validation live? Input boundary? Domain layer? Both?
10. What naming conventions exist? (files, modules, functions, variables, DB tables)

Architecture:
11. Why is the system structured this way and not [obvious alternative]?
12. What are the trust boundaries? What talks to what, and who validates at each boundary?
13. If I need to add a feature that crosses two subsystems, what's the integration pattern?
14. What's shared and what's private? Where are the fences?

Process:
15. What does "done" mean for a PR in this repo?
16. What are the non-negotiable quality checks before merge?
17. What's in scope for the agent vs what requires human decision?
```

Every UNDOCUMENTED response is a conversation waiting to happen. Every PARTIAL is a doc that needs expansion. The agent identifying its own knowledge gaps is cheaper than discovering them mid-implementation when it guesses wrong.

See [The Secret Sauce: how documentation compounds](the-secret-sauce.md#how-documentation-compounds) for why this investment grows over time rather than being a one-time cost.

---

## Prerequisites

- An AI agent runtime with cron/scheduling (see "What counts as a runtime" below)
- Access to your repo (GitHub, GitHub Enterprise, Gitea, etc.)
- Access to your issue tracker (GitHub Issues, Jira, Linear, etc.)
- At least two model providers (for twin review independence)

### What counts as a runtime

The system needs three capabilities:

1. **Scheduled execution** — Run a prompt on a timer (every 30 min for triage, every 4h for audits)
2. **Tool access** — The agent can run shell commands (git, build tools, linters) and call APIs (GitHub, Jira)
3. **Persistence between runs** — The agent's workspace (cloned repo, config files, state) survives between cron ticks

Concrete options:

| Runtime | How it works |
|---------|-------------|
| [OpenClaw](https://openclaw.ai) | Built-in cron scheduler, tool access, persistent workspace. What this system was built on. |
| [Hermes](https://github.com/socrates-ai/hermes) | Temporal-based agent runtime with scheduling and tool use. |
| GitHub Actions + self-hosted runner | Use `schedule:` triggers. The runner IS the persistent workspace. Cheapest to start but hardest to scale. |
| Custom cron + CLI agent | A cron job that invokes your agent CLI (`claude`, `aider`, etc.) with a prompt file. Works but you build the glue yourself. |
| Any agent framework with timers | LangGraph, CrewAI, AutoGen — if it can schedule, run tools, and persist state, it works. |

The loops described here are **runtime-agnostic**. They're prompt patterns + scheduling. If your system can "run this prompt every 30 minutes with access to git and the GitHub API" — you're set.

### Where documentation lives and how the agent reads it

The agent reads documentation by cloning repos and reading files. That's it — no vector databases, no RAG pipelines, no special tooling. Files in a git repo.

**Project docs** live in the project repo itself:
```
your-project/
├── docs/
│   ├── domain/           # What + why (survives a rewrite)
│   │   ├── glossary.md
│   │   ├── architecture.md
│   │   └── trading-pipeline.md
│   ├── impl/             # How (implementation-specific)
│   │   ├── supervision-tree.md
│   │   └── telemetry.md
│   ├── TEMPLATE-FEATURE-DESIGN.md
│   └── TEMPLATE-FEATURE-VALIDATION.md
├── CONVENTIONS.md    # or docs/conventions.md
└── ...
```

**Reference/pattern docs** live in a separate repo per ecosystem:
```
your-org/go-patterns/          # or elixir-patterns, rust-patterns, etc.
├── README.md
├── error-handling.md
├── concurrency.md
├── testing.md
├── module-structure.md
└── api-design.md
```

How the agent accesses them at runtime:
- **Project docs:** Already there — the agent clones the project repo to work on it, docs are included.
- **Pattern repos:** Referenced in config (`patterns_repo: your-org/go-patterns`). The agent clones them into its workspace on first use. They're just git repos with markdown files.
- **No special format required.** Markdown files with clear headings. The agent reads them like a human would — by opening the file and reading it. Structured headings and anchor links help for citability in reviews.

The key insight: documentation that lives in git is documentation that gets versioned, reviewed, and stays in sync with the code. No external knowledge bases that drift.

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
  dev: claude-opus             # expensive — code quality matters
  self_review: claude-sonnet   # different model, clean context
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
- Post-merge audit rarely finds gaps (< 1 gap issue per 10 merges)
- You trust the test suite to catch regressions (high coverage on critical paths)
- The human has reviewed enough output to trust the agent's judgment

This is a human decision, not a metric threshold. Some teams never flip it — and that's fine. The system works either way.

---

## Step 2: Implement the Loops

Each loop is an independent cron job. Start with Triage and Dev, add the rest once those are stable.

### Loop 1: Triage (every 30 min)

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
1. Spawn a CLEAN session — no context carry-over from dev work AND use a different model (both matter: isolation removes rationalizations, different model brings different strengths)
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

### Loop 6: Post-Merge Audit (every 4 hours)

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

Post-merge audit files issues to:
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

**"Post-merge audit files too many issues."**
Calibrate what counts as a "gap." Missing a NIT from acceptance criteria isn't worth an issue. Missing core functionality is. The threshold should be: "would a user notice this is missing?"

---

## Minimal Viable Setup

If you want to start with the absolute minimum and expand:

1. **Week 1:** Triage + Dev loops only. Agent picks up issues, opens PRs, assigns to human.
2. **Week 2:** Add Self-Review. Agent reviews its own PR before marking ready.
3. **Week 3:** Add Twin Review. Two models review every PR.
4. **Week 4:** Add Post-Merge Audit. Audit merged work for completeness.
5. **Ongoing:** Add Lookback once you have 5+ reviewed PRs to analyze.

Each step builds on the last. Don't try to run all 7 loops on day one.

---

## The One Thing That Matters Most

The system works because every loop feeds the next one. Work creates reviews, reviews create issues, issues create work. Quality ratchets up over time because:

1. Nothing ships without review (twin review)
2. Nothing stays shipped without audit (post-merge audit)
3. Nothing stays broken without a ticket (issue filing)
4. Nothing stays noisy without pruning (lookback)

If you only implement one thing beyond basic dev work, implement **post-merge audit**. It's the loop that catches "good enough" before it becomes technical debt.

---

## Further reading

| Document | What it covers |
|----------|----------------|
| [The Secret Sauce](the-secret-sauce.md) | Why documentation drives the system, reference docs, the cascade discipline, failure tensions |
| [Scaling to Multiple Repos](scaling-multiple-repos.md) | Global dispatcher, WIP rules, failure modes and mitigations |

---

## Appendix: Documentation Templates

These templates are starting points — adapt them to your language and domain. The important thing isn't the exact sections; it's that every feature goes through the same structure, and that structure is checkable by automation.

### Feature Design Template

Every feature that introduces new behavior gets a design doc before implementation begins. Not after. Not "when we have time." Before.

```markdown
# [Feature Title]

[Opening paragraph: what problem this solves, what happens WITHOUT it.
Reference the prior doc or issue that establishes the need.]

---

## Context ownership

**Owner:** [which subsystem/module/team owns this]
**Collaborators:** [what other parts of the system are affected and how]

---

## Mechanism

[How it works. Present tense, prescriptive. Not goals — behavior.]

{Mermaid diagram: architecture placement, data flow, or state machine}

### State tracking (if stateful)

| State | Meaning | Transitions to |
|-------|---------|----------------|
|       |         |                |

### Events (produced or consumed)

| Event | Trigger | Action |
|-------|---------|--------|
|       |         |        |

---

## Core logic

[The algorithm. What is evaluated, when, what the outcomes are.]

### Why this approach (not the obvious one)

[Name the failure scenario that motivates this design over the simpler alternative.]

### Edge cases

**[Case name].** Scenario, behavior, why it's correct.

---

## Configuration

[For each tunable parameter:]

| Parameter | Too low means... | Too high means... | Target |
|-----------|-----------------|-------------------|--------|
|           |                 |                   |        |

---

## Failure modes

| Scenario | Outcome | Recovery |
|----------|---------|----------|
|          |         |          |

---

## Cross-references

- [Related doc](link) — what aspect it covers
```

**Why this template works for agents:**
- "What happens WITHOUT it" forces problem clarity (agent can evaluate if the issue even needs solving)
- Mermaid diagrams are parseable — the agent can verify implementation matches the diagram
- The failure modes table becomes testable acceptance criteria
- Edge cases become test cases
- "Why this approach" prevents the agent from refactoring toward the "obvious" alternative

### Design Validation Checklist

Run Phase 1 before implementation starts. Run Phase 2 before marking the PR ready for review. Items that don't apply get marked N/A with a one-line justification — blank is not the same as N/A.

#### Phase 1 — Design Review (before writing code)

**Problem framing:**
- [ ] Opening paragraph states what happens WITHOUT this feature
- [ ] References the specific prior doc or issue that establishes the need
- [ ] Scope is clear from reading the document alone

**Ownership:**
- [ ] Names exactly one owning subsystem/module/context
- [ ] If it crosses boundaries, relationship patterns are named for each crossing
- [ ] New terms checked against existing glossary — no synonyms for existing concepts

**Mechanism clarity:**
- [ ] A newcomer can understand the design from this doc alone (plus cross-refs)
- [ ] At least one diagram shows architecture placement or data flow
- [ ] Process/component placement is justified — why here, what breaks if moved
- [ ] States (if any) have an explicit table with meanings

**Edge cases and failures:**
- [ ] Every edge case that could cause silent failure is named
- [ ] Failure mode table covers: crash, dependency down, data inconsistency, timing race
- [ ] Each failure states recovery path (automatic, manual, or alert)
- [ ] Default behavior is fail-closed for safety-critical paths

**Configuration:**
- [ ] Every tunable parameter has "too low" and "too high" failure scenarios
- [ ] No magic numbers without justification
- [ ] Decision framework exists even if exact value isn't known yet

**Integration:**
- [ ] Events documented with shape, producer, consumers
- [ ] Dependencies on other features are cross-references, not inline duplications
- [ ] No new vocabulary that duplicates existing terms

**Documentation quality:**
- [ ] Present tense, prescriptive (how it behaves, not how it might)
- [ ] No deliberation in the doc — investigation belongs in comments/ADRs
- [ ] Every non-obvious choice explains why the alternative was rejected

#### Phase 2 — Implementation Review (before marking PR ready)

**Architecture fit:**
- [ ] Feature lives in exactly one namespace/module/package
- [ ] No business logic in data layer (schemas, models, DTOs)
- [ ] External side effects go through a defined boundary (context module, service layer, port)
- [ ] No direct DB/IO calls from controllers, handlers, or UI layer

**Testing:**
- [ ] Unit tests cover core logic in isolation
- [ ] Integration tests cover boundary crossings
- [ ] Edge cases from the design doc have corresponding test cases
- [ ] No time-dependent tests (no `sleep` — use deterministic synchronization)
- [ ] Tests are isolated — can run in parallel without interference

**Observability:**
- [ ] Key operations emit metrics or structured log events
- [ ] Error paths log with enough context to diagnose without reproducing
- [ ] Metric tags are low-cardinality (< 100 distinct values per tag)

**Error handling:**
- [ ] Every external call has explicit error handling (no bare unwrap/raise/panic)
- [ ] Errors at boundaries are translated to the local domain's vocabulary
- [ ] User-facing errors are distinct from internal errors

### How agents use these templates

The templates aren't just for humans writing docs. They're machine-checkable gates:

1. **Triage reads the design doc** and checks: does it pass Phase 1? If not, the issue isn't ready for implementation — flag it.
2. **Self-review runs Phase 2** against the PR diff. Each checkbox becomes a concrete question: "did the code add error handling for external calls?"
3. **Twin review cross-references the design** — does the implementation match the mechanism section? Do the test cases cover the edge cases listed?
4. **Post-merge audit checks completeness** — every failure mode in the table should have a corresponding test or handler in the code.

The templates turn subjective review ("does this look good?") into objective verification ("does this satisfy the checklist?"). That's what makes autonomous review possible — the standard is documented, not vibes.
