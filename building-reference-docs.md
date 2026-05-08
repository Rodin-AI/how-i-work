# Building Reference Documentation

This is the work that makes everything else possible. Without reference docs, an agent writes code that works. With reference docs, an agent writes code that *belongs*.

This page covers how to build each type of documentation from scratch, and — critically — how to keep it alive as the codebase evolves.

---

## The types and what they're for

| Type | Answers the question | Example |
|------|---------------------|---------|
| Domain glossary | "What IS this thing?" | "An Order is a request to buy or sell..." |
| Conventions | "How do we do things HERE?" | "Error handling uses `{:ok, result}` tuples..." |
| Reference patterns | "Show me how to add a new X" | A complete command handler with tests |
| Ecosystem patterns | "How does the ECOSYSTEM do this?" | Error handling patterns from kubernetes, etcd |
| Implementation docs | "How does this subsystem work internally?" | "The order pipeline uses a GenStage flow..." |
| Feature designs | "What did we decide and why?" | Design doc for the escalation system |

Each type lives in a specific place:

| Type | Location | Scope |
|------|----------|-------|
| Domain glossary | `your-project/docs/domain/glossary.md` | This project |
| Conventions | `your-project/CONVENTIONS.md` | This project |
| Reference patterns | `your-project/docs/domain/<pattern>.md` | This project |
| Ecosystem patterns | `your-org/<lang>-patterns/` (separate repo) | All projects in that language |
| Implementation docs | `your-project/docs/impl/<topic>.md` | This project |
| Feature designs | `your-project/docs/designs/<feature>.md` | This project |

**The stability test:** "Would this fact survive a complete rewrite of the internals?"
- Yes → domain docs
- No → implementation docs
- Maybe → conventions (the WHAT survives, the HOW might change)

---

## Domain glossary

**What it is:** A list of terms your system uses, with precise definitions. Not "what the word means in English" — what it means in YOUR system.

**How to write it:**

1. Open a new file: `docs/domain/glossary.md`
2. Go through your codebase and list every noun that appears in module names, database tables, API endpoints, or config keys
3. For each term, write a 1-2 sentence definition that answers: "If a new developer asked 'what's a [term]?', what would you say?"
4. Note relationships: "An Order belongs to exactly one Account and references one Instrument"
5. Note constraints: "An Instrument ID is always uppercase, 3-8 characters, alphanumeric only"

**Example:**

```markdown
# Domain Glossary

## Order
A request to buy or sell a specific quantity of an Instrument at a given price.
An Order always belongs to exactly one Account. An Order is immutable once filled.

## Instrument
A tradeable asset identified by a unique symbol (uppercase, 3-8 alphanumeric chars).
Examples: BTC, ETH, AAPL. An Instrument exists independently of any Order.

## Account
A container for Orders and Positions belonging to a single user.
Accounts have a balance (denominated in a single settlement currency)
and zero or more open Positions.

## Position
The net exposure to an Instrument within an Account.
Created when an Order fills. Closed when exposure reaches zero.
A Position can be long (positive quantity) or short (negative quantity).
```

**Common mistakes:**
- Too vague: "Order — something a user creates" (what IS it?)
- Too implementation-specific: "Order — a row in the orders table with columns..." (that's impl docs)
- Missing relationships: defines terms in isolation without saying how they connect
- Aspirational: describing what you WANT the system to be, not what it IS

**Time to write:** 1-2 hours for an initial 15-30 term glossary.

---

## Conventions doc

**What it is:** How code is written in THIS repo. Not aspirational rules — a description of the patterns that already exist, plus any explicit decisions about how new code should look.

**How to write it:**

1. Open your 5 most recently merged PRs
2. For each one, note: file naming, module structure, error handling patterns, test file locations, dependency injection approach
3. Look for the patterns — what's consistent across all 5?
4. Write those patterns down as "this is how we do X"
5. For anything inconsistent, make a decision NOW and write it down

**Structure:**

```markdown
# Conventions

## File & Module Structure
- One module per file
- File path mirrors module name: `lib/myapp/orders/validator.ex` → `MyApp.Orders.Validator`
- Context modules (top-level API) go in `lib/myapp/<context>.ex`
- Internal modules go in `lib/myapp/<context>/<module>.ex`

## Error Handling
- Public functions return `{:ok, result}` or `{:error, reason}`
- Internal functions may raise (caller handles at boundary)
- Never rescue in the middle of a pipeline — let it fail to the boundary
- Log at the boundary, not at the call site

## Testing
- Unit tests: `test/<context>/<module>_test.exs`
- Integration tests: `test/integration/<feature>_test.exs`
- All tests async:true unless they touch shared state (DB, PubSub)
- Use factories (ExMachina) for test data, never raw inserts

## Naming
- Boolean functions: `is_<thing>?` or `has_<thing>?`
- Conversion functions: `to_<target>` (e.g., `to_map`, `to_string`)
- Modules: nouns (`OrderValidator`), not verbs (`ValidateOrder`)
```

**Common mistakes:**
- Describing what you WISH the code looked like (fix the code first, then document reality)
- Being too abstract: "follow good practices" (which practices? show me)
- Not covering error handling (the #1 source of inconsistency in every codebase)
- Forgetting tests (where do they go? what's the naming? what helpers exist?)

**Time to write:** 2-3 hours. Most of it is reading existing code, not writing.

---

## Reference patterns

**What it is:** A documented example of the most common unit of work in your repo. When the agent needs to "add a new X", this is the blueprint.

**How to write it:**

1. Pick the thing you add most often (endpoint, handler, worker, module, command)
2. Find the BEST existing example in your codebase — the one you'd point a new teammate to
3. Document it step by step: what files get created, what gets registered, what tests are needed
4. Include a complete code example (real code from your repo, not pseudocode)
5. Note the WHY for non-obvious decisions

**Example:**

```markdown
# Pattern: Adding a New Command

Commands are the write path in the CQRS system. Every state mutation
goes through a command.

## Files to create

1. `lib/myapp/commands/do_thing.ex` — the command struct
2. `lib/myapp/commands/do_thing_handler.ex` — the handler
3. `test/myapp/commands/do_thing_handler_test.exs` — tests

## 1. The Command struct

```elixir
defmodule MyApp.Commands.DoThing do
  @moduledoc "Describes the intent to do the thing."

  @enforce_keys [:account_id, :instrument_id]
  defstruct [:account_id, :instrument_id, :quantity, :metadata]

  @type t :: %__MODULE__{
    account_id: String.t(),
    instrument_id: String.t(),
    quantity: Decimal.t() | nil,
    metadata: map()
  }
end
```

## 2. The Handler

```elixir
defmodule MyApp.Commands.DoThingHandler do
  @behaviour MyApp.CommandHandler

  @impl true
  def handle(%DoThing{} = cmd, context) do
    with {:ok, account} <- fetch_account(cmd.account_id, context),
         :ok <- validate_permissions(account, :do_thing),
         {:ok, result} <- execute(cmd, account) do
      {:ok, result}
    end
  end
end
```

## 3. Tests

- Happy path: command succeeds with valid inputs
- Validation: rejects invalid account_id, missing required fields
- Permissions: returns error when account lacks permission
- Idempotency: same command twice produces same result (if applicable)

## Why this structure?

- Commands are data (structs), not behavior. Handlers are behavior.
  This separation lets us serialize commands, replay them, and test handlers in isolation.
- `@behaviour` ensures every handler implements the same interface.
  The router dispatches by pattern matching on the command struct.
- `with` chains for the happy path. Each clause can fail independently
  and the error propagates without nesting.
```

**Common mistakes:**
- Too abstract: showing pseudocode instead of real code from your repo
- No WHY: showing what to do without explaining why it's done that way
- Only the happy path: not showing error cases, tests, or registration
- Stale: the pattern doc says one thing but recent PRs do another (update it!)

**Time to write:** 1-2 hours per pattern. Start with your most common one.

---

## Ecosystem pattern repos

**What it is:** Curated knowledge about how your language/framework ecosystem does things — extracted from authoritative sources (major open-source projects, official guides, standard library source code). NOT your opinions about best practices. Reality, documented.

**How to build one:**

1. **Pick 3-5 authoritative projects** in your ecosystem. For Go: kubernetes, etcd, cockroachdb. For Elixir: phoenix, ecto, oban. For Rust: tokio, serde, axum.

2. **Pick a concern** (error handling, testing, concurrency, module structure).

3. **Read their source code** for that concern. Not their docs — their code. How do they ACTUALLY handle errors? What patterns repeat across files?

4. **Extract the pattern.** Describe what you see, with real examples from the source. Note where projects disagree — that's valuable information too.

5. **Organize by question.** Each file answers one question a developer would have:
   - `error-handling.md` → "How should I handle errors in this ecosystem?"
   - `concurrency.md` → "What are the concurrency patterns and when do I use each?"
   - `testing.md` → "How do I structure tests and what do I test?"

**Repo structure:**

```
your-org/go-patterns/
├── README.md              # what this repo is, how to use it
├── error-handling.md      # from kubernetes, etcd, cockroachdb
├── concurrency.md         # goroutine patterns, channel usage
├── testing.md             # table-driven, test helpers, mocking
├── interfaces.md          # accept interfaces, return structs
└── module-structure.md    # package layout, internal packages
```

**What a pattern file looks like:**

```markdown
# Error Handling in Go

## Sources analyzed
- kubernetes/kubernetes (v1.29)
- etcd-io/etcd (v3.5)
- cockroachdb/cockroach (v23.2)

## The pattern

All three projects wrap errors with context at each level:

```go
if err != nil {
    return fmt.Errorf("fetching pod %s: %w", name, err)
}
```

They never:
- Return bare `err` without wrapping
- Use `errors.New` for dynamic messages (that's for sentinel errors)
- Log AND return (pick one — the caller decides)

## Where they disagree

Kubernetes uses `utilruntime.HandleError()` for non-fatal errors in
controllers. Etcd propagates everything. CockroachDB has a custom
`errors` package with stack traces.

## Recommendation for our repos

Follow the kubernetes pattern: wrap at every level, log at the boundary,
propagate in the middle. Use `fmt.Errorf("context: %w", err)` everywhere.
```

**The key insight:** These aren't YOUR opinions. They're documented reality from projects that have been in production for years with hundreds of contributors. The agent can trust them because they're grounded in verifiable sources.

**Time to build:** 4-8 hours for initial 5-file repo. But it compounds — applies to every project in that language forever.

---

## Keeping documentation alive

Building docs is the easy part. The hard part is keeping them current as the codebase evolves. Stale docs are worse than no docs — they cause the agent to confidently write wrong code.

### The staleness problem

Documentation goes stale in predictable ways:

| What rots | Why | How fast |
|-----------|-----|----------|
| Implementation docs | Code changes, docs don't get updated | Weeks |
| Conventions | Team evolves practices without updating the doc | Months |
| Reference patterns | New patterns emerge, old examples get outdated | Months |
| Domain glossary | New concepts added without glossary entries | Slow (terms are stable) |
| Ecosystem patterns | Upstream projects evolve | 6-12 months |

### Automated staleness detection

The post-merge audit loop catches staleness naturally: if a merged PR contradicts what docs say, it files an issue. But you can also detect it proactively.

**Monthly doc health check (give this to your agent):**

```
Review all documentation in docs/ and CONVENTIONS.md.
For each documented pattern, convention, or claim:
1. Search the codebase for counter-examples (code that contradicts the doc)
2. Count conforming vs non-conforming instances
3. If non-conforming > 30%, flag as potentially stale

Output: list of potentially stale docs with evidence (file paths of contradictions)
```

**Signals that docs need updating:**
- Triage flags increasing → agent can't find answers → docs have gaps
- Self-review findings that cite docs but the code "disagrees" → docs or code is wrong
- New modules that don't follow documented patterns → pattern evolved, doc didn't
- Lookback finds: "agent kept doing X despite docs saying Y" → docs are being ignored (maybe wrong)

### Who updates what

| Type | Who updates | When |
|------|------------|------|
| Domain glossary | Human (after conversations) | When new concepts emerge |
| Conventions | Agent proposes, human approves | When patterns drift is detected |
| Reference patterns | Agent proposes, human approves | When a better example exists |
| Ecosystem patterns | Scheduled refresh (quarterly) | When upstream releases major versions |
| Implementation docs | Agent (after code changes) | Every PR that changes architecture |
| Feature designs | Human (before implementation) | Before starting a feature |

### The documentation PR pattern

When the agent detects stale docs, it shouldn't silently fix them. Documentation changes affect how ALL future code is written. They need human review.

The pattern:

1. Agent detects contradiction between docs and code
2. Agent opens a PR that ONLY updates docs (no code changes)
3. PR description explains: "Code does X. Docs say Y. Evidence: [files]. Proposed update: align docs to code."
4. Human reviews: is the code right (update docs) or are the docs right (file a bug)?

This keeps the human as the authority on "what should be true" while the agent handles the grunt work of finding contradictions and proposing updates.

### Ecosystem pattern refresh

Upstream projects evolve. The patterns you extracted 6 months ago might be outdated. Schedule a quarterly refresh:

```
Review our <language>-patterns repo against current main branches of:
- [source project 1]
- [source project 2]
- [source project 3]

For each pattern file:
1. Check if the sources still follow the documented pattern
2. Note any new patterns that emerged since last review
3. Check if any pattern was deprecated or replaced
4. Flag files where the source reality has diverged from our docs

Output: list of files needing updates, with evidence from current source
```

This can be a scheduled task (monthly or quarterly) that runs cheaply — it's just reading code and comparing against docs.

### The compound benefit of freshness

Fresh docs don't just prevent errors — they accelerate everything:

- **Triage is faster** because the gate can answer "can I do this?" immediately
- **Dev produces fewer iterations** because the agent writes correct code on the first try
- **Self-review is more precise** because it's checking against current truth, not stale aspirations
- **Twin review is more actionable** because findings cite living docs, not dead ones
- **Post-merge audit is cheaper** because there's less drift to catch

Stale docs create a doom loop: docs are wrong → agent writes wrong code → reviews catch it → agent gets confused → more iterations → more cost → docs get more stale because nobody has time to update them.

Fresh docs create a flywheel: docs are right → agent writes correct code → reviews find nothing → everyone saves time → time gets invested in keeping docs fresh → docs stay right.

---

## Getting started: the 4-hour bootstrap

If you're starting from zero, here's the minimum viable documentation investment:

**Hour 1: Domain glossary** (15-20 terms)
- Scan module names and DB tables
- Write a 1-2 sentence definition for each
- Note the most important relationships

**Hour 2: Conventions** (error handling + testing + naming)
- Read your 5 most recent PRs
- Document the patterns you see repeating
- Make decisions where things are inconsistent

**Hour 3: One reference pattern** (your most common unit of work)
- Find the best existing example
- Document it step by step
- Include the WHY

**Hour 4: Gap-finding run**
- Give the agent the [sufficiency checklist](adoption-guide.md) and your new docs
- See what it can and can't answer
- Note the gaps for future sessions

That's it. Four hours gets you from "the agent guesses at everything" to "the agent has a foundation to build on." It won't be complete — but it'll be enough to start the loops, and the loops will surface what's missing.

---

## Further reading

- [The Secret Sauce](the-secret-sauce.md) — Why documentation is the fuel, how it compounds, the triage gate
- [Adoption Guide](adoption-guide.md) — How to set up the loops that USE these docs
- [Scaling to Multiple Repos](scaling-multiple-repos.md) — How ecosystem pattern repos amortize across projects
