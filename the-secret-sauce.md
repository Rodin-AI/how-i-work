# The Secret Sauce: Why Documentation Drives Everything

The loops described in `how-i-work.md` are the engine. This document is about the fuel.

People who try to replicate this system focus on the automation — the cron jobs, the dispatchers, the twin reviews, the post-merge audits. They miss the thing that makes all of it work: **the documentation investment that happens before any code gets written.**

Without it, you have an expensive noise generator. With it, you have an autonomous quality ratchet.

---

## The pipeline most people skip

Most teams go: idea → issue → code → review → merge.

This system goes: **conversation → understanding → documentation → issue → code → review → merge → audit.**

The first three steps are where 80% of the value is created. They're also the steps most people skip because they feel like "not real work."

---

## Stage 1: Conversation

Before anything gets documented, there's a conversation. Not "write me a spec" — an actual back-and-forth where two minds (human and agent, or two humans) wrestle with the problem:

- **What IS this thing?** Not what does it do — what is it? A trading system is a state machine where the state is money. Name the essence.
- **What are the forces?** Every design decision exists in tension with others. Find the tensions before you resolve them.
- **What assumptions are we making?** Say them out loud. Write them down. Half of them are wrong — better to discover that now.
- **What would make this wrong?** Try to kill the idea before investing in it. If it survives, it's stronger.

This phase has no deliverable. No document gets written. No issue gets filed. It's pure thinking — and it's the highest-leverage time in the entire process.

**Why it matters for agents:** An agent that skips this phase and jumps to implementation is working from its training data's average understanding of the problem. An agent that went through this phase is working from a *specific, shared, tested* understanding. The difference shows up in every line of code.

---

## Stage 2: Understanding → Documentation

Once the conversation produces clarity, that clarity gets written down. Not as a one-time spec that rots — as living documents that the entire system reads:

**Domain docs** — What the system IS. Bounded contexts, entities, relationships, invariants. "An order cannot exist without a valid instrument." These are facts about the problem domain that survive any rewrite.

**Pattern repos** — How to write code idiomatically. Not style guides (tabs vs spaces) — structural patterns. "GenServer state should be minimal; derived data doesn't belong in state." "Accept interfaces, return structs." These encode architectural taste.

**Conventions files** — How THIS project works specifically. File structure, naming, testing approach, module boundaries. The thing that makes code "fit" vs "bolted on."

**Decision records** — Why we chose X over Y. Six months from now, nobody remembers why. Written down, the reasoning survives and prevents relitigating settled decisions.

Each of these is a **compressed judgment call.** The human thought hard about it once, in full context, with real tradeoffs weighed. The documentation encodes that thinking so it can be applied consistently, forever, without re-derivation.

---

## Stage 3: Issues with teeth

Only after documentation exists do issues get filed. And they're different from typical issues because they have:

- **Clear acceptance criteria** — not "add retry logic" but "retry 3 times with exponential backoff, starting at 100ms, capping at 5s, only on 503/429, with jitter"
- **Domain context** — which bounded context this touches, which invariants must hold
- **What "done" looks like** — specific enough that a machine can audit it post-merge

Vague issues produce vague PRs that can't be reviewed meaningfully. Specific issues produce specific PRs that can be checked mechanically.

---

## The triage gate: don't guess, escalate

Here's a critical piece that makes the autonomous loop safe:

**When triage encounters an issue with ambiguity that can't be resolved from existing documentation, it doesn't guess. It flags the issue for human review.**

The agent's job is to determine: "Can I implement this using only facts already established in the repo?" If the answer is yes — proceed. If the answer is "I'd have to make a judgment call about something not yet decided" — stop and ask.

Examples:

| Situation | Agent decision |
|-----------|---------------|
| Issue says "add retry logic" and patterns repo defines retry conventions | ✅ Proceed — conventions exist |
| Issue says "add caching" but no caching pattern is documented | ⛔ Flag — need human input on caching strategy |
| Issue references a bounded context that's fully specified in domain docs | ✅ Proceed — domain is clear |
| Issue touches two bounded contexts and the boundary isn't defined | ⛔ Flag — boundary decision needed |
| Acceptance criteria are explicit and testable | ✅ Proceed |
| Acceptance criteria say "should feel responsive" | ⛔ Flag — not measurable |

This gate is what makes autonomy safe. The agent doesn't operate on vibes or best guesses. It operates on established facts. When facts don't exist, it says so and asks for them.

**The result:** Every piece of work the agent touches has been pre-validated as "clear enough to implement without judgment calls." The human's time goes to making decisions (high leverage), not reviewing guesses (low leverage).

---

## How documentation compounds

The documentation investment isn't linear — it compounds:

**Session 1:** You spend an hour discussing how orders work. Write it down. Now every future session involving orders starts from shared understanding instead of re-derivation.

**Session 10:** You've documented 5 bounded contexts. New issues that touch those contexts can be implemented autonomously because the rules are clear.

**Session 50:** The patterns repo has 20+ documented patterns. Code review becomes mechanical — "does this follow Pattern 12?" — instead of subjective.

**Session 100:** Post-merge review can audit against real acceptance criteria from real domain docs. The quality ratchet works because there's something to ratchet against.

Without documentation, session 100 looks like session 1. The agent is still re-deriving the same decisions, still guessing at the same conventions, still producing inconsistent output. There's no compounding because there's nothing to compound on.

---

## What "documentation" actually means here

It's not:
- Javadoc comments on every function
- A 200-page architecture document nobody reads
- Confluence pages that rot after creation
- README files that describe how to run the project

It IS:
- **Domain facts** that survive a rewrite ("an order must have a valid instrument")
- **Structural patterns** that encode taste ("errors propagate up; logging happens at boundaries")
- **Decision records** that prevent relitigating ("we chose event sourcing because X, not Y")
- **Conventions** that make code belong ("modules are namespaced by bounded context")
- **Clear boundaries** that prevent scope creep ("the Ledger context never knows about market data")

The test: "Would this fact be useful to a new team member on day 1?" If yes, it belongs in documentation. If it's only useful for the current sprint, it belongs in the issue.

---

## The feedback loop nobody talks about

Documentation isn't write-once. The loops improve it:

**Post-merge review finds a gap** → "The issue didn't specify behavior for timeout errors" → Update the domain docs to specify timeout semantics → Future issues in that area are more specific → Future PRs handle timeouts correctly from the start.

**Lookback finds noise** → "Reviews keep flagging X but nobody acts on it" → Either X doesn't matter (remove it from patterns) or X matters but isn't understood (improve the documentation of why it matters) → Future reviews are more precise.

**Triage flags ambiguity** → Human makes a decision → Decision gets documented → Next time the same question comes up, the agent doesn't need to ask.

Every cycle through the loops produces documentation improvements. Documentation improvements make future cycles faster and more accurate. This is the compound interest of the system.

---

## Why most AI agent implementations fail

They skip all of this. They point an agent at a repo with:
- No documented conventions (agent guesses at style)
- No domain docs (agent guesses at business logic)
- No patterns (agent uses training-data-average patterns)
- Vague issues (agent interprets creatively)
- No triage gate (agent implements best guesses autonomously)

Then they're surprised when the output is inconsistent, requires heavy review, and doesn't "fit" the codebase. The agent isn't broken. It just has no fuel.

**The uncomfortable truth:** The documentation investment is the hard part. The automation is easy. Anyone can set up cron jobs and API calls. Not everyone can articulate their domain clearly enough that a machine can implement against it without guessing.

---

## What this means for adoption

If you're adopting this system, the loops are step 2. Step 1 is:

1. **Document your domain.** What are the bounded contexts? What are the invariants? What are the entities and their relationships?
2. **Document your patterns.** How do you handle errors? How do you structure modules? What does idiomatic code look like in your stack?
3. **Document your conventions.** How is this specific project organized? What goes where? How are things named?
4. **Write issues with acceptance criteria.** Not "add feature X" but "when Y happens, Z should occur, verified by test W."
5. **Implement the triage gate.** Agent proceeds only when facts support it. Ambiguity goes to the human.

Then turn on the loops. They'll work because they have something to work against.

---

## The ratio

In practice, the time split looks roughly like:

| Activity | Time | Leverage |
|----------|------|----------|
| Conversation + thinking | 20% | Highest — shapes everything downstream |
| Documentation | 20% | High — amortizes across all future work |
| Issue creation | 10% | Medium — precision here prevents waste later |
| Implementation | 30% | Medium — this is where agents shine |
| Review + audit | 20% | High — closes the loop |

Most teams spend 80% on implementation and 5% on documentation. Then they wonder why automation doesn't help much. The automation amplifies whatever understanding exists. If the understanding is shallow, the automation produces shallow work at scale.

---

## Reference docs: grounding in reality

There's another layer beyond project documentation that most people miss entirely: **reference documents that ground the agent's knowledge in external reality.**

An LLM has training data. That training data includes millions of opinions about how to write code, how to design systems, how to structure domains. The problem: it's averaged. Ask an agent "write idiomatic Elixir" and you get a blend of blog posts, tutorials, Stack Overflow answers, and production code — weighted by whatever was popular in the training set.

That's not good enough. "Average understanding" produces average code.

### What we actually did

Before writing application code, we built **reference repositories** — curated knowledge extracted from authoritative sources:

**Pattern repos (extracted from top open-source projects):**
- Analyzed the top 10 Elixir projects by sustained engineering (elixir-lang/elixir, phoenix, ecto, etc.)
- Analyzed the top 10 Go projects (kubernetes, etcd, prometheus, etc.)
- Extracted *actual patterns* — not what blogs say is idiomatic, but what battle-tested production code actually does
- Organized by category: error handling, concurrency, testing, module structure, API design
- Compared against official style guides and documented where real practice diverges from aspirational docs

**Theory references (language-agnostic, grounded in literature):**
- Canonical DDD reference — what the terms actually mean per Evans, not "DDD as interpreted by tutorials"
- CQRS/Event Sourcing reference — precise definitions, tradeoffs, when-to-use-what
- Cross-language analysis — what DDD concepts survive contact with specific runtimes (e.g., Elixir/OTP changes how you think about aggregates)

### Why this changes everything

**Without reference docs:** "Write idiomatic Elixir" means "write what the model thinks Elixir looks like based on training data." The agent uses patterns from tutorials, outdated practices, and averaged opinions.

**With reference docs:** "Write idiomatic Elixir" means "write what Phoenix, Ecto, and LiveView actually do in production." The agent is grounded in the same reality that experienced developers know from years of reading source code.

The difference is concrete:

| Without reference docs | With reference docs |
|----------------------|--------------------|
| GenServer with bloated state | GenServer state is minimal — derived data computed on demand (Pattern: phoenix_live_view's assign_new) |
| Generic error handling | Errors propagate up via tuples; logging happens at boundaries only (Pattern: ecto's Repo module) |
| "Accept interfaces" as abstract advice | Specific: io.Reader for input, concrete struct returns, never interface-to-interface (Pattern: kubernetes/client-go) |
| DDD terms used loosely | Aggregate = consistency boundary, not "big object." Bounded context = linguistic boundary, not "microservice" |

### The grounding effect on each loop

**Dev loop:** When the agent writes code, it checks against patterns extracted from *real projects*, not training-data averages. The code comes out looking like it was written by someone who's read the Phoenix source — because in a meaningful sense, it was.

**Self-review:** The reviewer checks against the same reference. "Does this follow the pattern Phoenix uses for channel authentication?" is a checkable question. "Is this idiomatic?" is subjective.

**Twin review:** Reviewers cite specific patterns by name and URL. "This violates the error-handling pattern from ecto (see: patterns/error-handling.md#let-it-crash)." Actionable, verifiable, not a matter of opinion.

**Post-merge audit:** When checking if a PR delivered what it should, the reference docs define what "correct" implementation of a DDD concept looks like. Without that definition, the audit is just guessing.

### Why training data isn't enough

People assume the model "already knows" how to write good code. It does — in the same way a library contains every book. Having the information exist somewhere in weights is not the same as having it actively grounding every decision.

The reference docs do three things training data can't:

1. **Specificity.** Training data says "handle errors." Reference docs say "use tagged tuples `{:ok, result} | {:error, reason}`, never raise for expected failures, pattern match at boundaries, and return the error unchanged through intermediate layers."

2. **Authority.** Training data is popularity-weighted. Reference docs are curated from authoritative sources. The agent follows what *kubernetes actually does*, not what a blog post about kubernetes recommended five years ago.

3. **Consistency.** Training data gives different answers depending on prompt wording. Reference docs give the same answer every time. The agent can't drift because the reference doesn't drift.

### How to build reference docs

You don't write them from scratch. You extract them:

1. **Identify authoritative sources.** For your language/stack, which projects have the longest sustained engineering, most contributors, and best reputation? Those are your sources.

2. **Extract patterns from source code.** Don't read their docs — read their code. Docs describe aspirations. Code describes reality. Where they diverge, trust the code.

3. **Compare against official guidelines.** Some divergences are intentional (the guideline is aspirational). Some are accidental (the guideline is outdated). Note both.

4. **Organize by concern.** Error handling, concurrency, testing, module structure, API design. Not by project — by the *question* the developer has when they're writing code.

5. **Make them citable.** Each pattern gets a URL, a name, a clear example. Review bots should be able to say "see Pattern X" with a link, not just "consider doing it differently."

### The compound effect

Reference docs compound differently from project docs:

- **Project docs** make THIS codebase better.
- **Reference docs** make EVERY codebase better.

Once you've extracted Go patterns from kubernetes, those patterns apply to every Go project you work on. The investment amortizes across repos, across teams, across years. Build them once, use them everywhere.

---

## Summary

The secret sauce isn't the loops. It's what the loops operate on.

Conversations produce understanding. Understanding gets documented. Documentation creates the constraints that make autonomous work possible. Clear constraints make reviews checkable, audits meaningful, and the quality ratchet functional.

Without documentation, you have an agent that writes code. With documentation, you have an agent that writes code *that belongs* — and a system that ensures it keeps belonging, forever.

The loops are the engine. The documentation is the fuel. Invest in both.
