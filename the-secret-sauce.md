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

## Summary

The secret sauce isn't the loops. It's what the loops operate on.

Conversations produce understanding. Understanding gets documented. Documentation creates the constraints that make autonomous work possible. Clear constraints make reviews checkable, audits meaningful, and the quality ratchet functional.

Without documentation, you have an agent that writes code. With documentation, you have an agent that writes code *that belongs* — and a system that ensures it keeps belonging, forever.

The loops are the engine. The documentation is the fuel. Invest in both.
