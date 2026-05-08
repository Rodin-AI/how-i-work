# What We Learned: Why Multi-Model Systems Work

This system wasn't designed from theory. It was built through experimentation — running different models on the same tasks, measuring what each catches, and tracking what gets missed. This page documents the insights that shaped the architecture.

---

## The core discovery: different models see differently

We started with a simple hypothesis: run two independent reviewers on every PR and you'll catch more bugs. That turned out to be true, but the *why* was more interesting than expected.

Given the same code, the same context, and the same instructions:

- **One model consistently catches structural issues** — broken references, missing imports, dead code, pattern violations, formatting problems
- **Another consistently catches semantic issues** — logical contradictions, cross-component assumption breaks, race conditions, domain-specific risks

These aren't random differences. They're consistent across dozens of reviews. Each model gravitates toward what it's architecturally better at, even with identical prompts.

**What this means for the system:** Twin review isn't just "two sets of eyes." It's two fundamentally different analytical modes applied to the same code. One checks "does this follow the rules?" The other checks "does this actually make sense?"

---

## Signal-to-noise ratio matters more than model capability

Early in our testing, we found something counterintuitive:

> A cheap, small model with one precise question outperformed an expensive, large model doing a broad review.

The small model was given just the relevant text and asked: "Do any of these lead toward a predetermined conclusion?" It found every issue.

The expensive models were given the same text buried inside a full PR review (diff, file content, issue description, acceptance criteria, project conventions) and asked "review this PR." They missed the same issues entirely.

**We thought this meant small models were better at focused analysis.** We were wrong.

When we re-ran the experiment giving all models ONLY the relevant text (removing the surrounding noise), every model — including the cheapest — caught every issue regardless of whether the question was narrow or broad.

**The real finding:** It's not about model size or question precision. It's about **signal-to-noise ratio**. When the thing you're looking for is buried in layers of other context, models lose it. When it's isolated, any model finds it.

**What this means for the system:**
- Triage uses a cheap model because the question is simple: "Is anything stuck?" — low noise, clear signal
- Dev uses an expensive model because implementation requires holding complex context
- Self-review works because the reviewer gets ONLY the diff — not the conversation that produced it. The noise (justifications, trade-offs, reasoning) is gone
- The post-merge audit isolates one question: "Did this PR deliver what the issue asked?" — focused comparison, not broad review

---

## Specialization beats identical mandates

We ran both twin reviewers with the same prompt: "Review this PR for correctness, idiomatic code, potential bugs, and design issues." They found overlapping things and missed the same categories.

When we gave each model a focused mandate aligned with its strengths:

**Reviewer 1 (structural focus):**
- Pattern compliance: does the code match documented conventions?
- Specification completeness: are all edge cases handled?
- Cross-references: do imports, types, and interfaces align?

**Reviewer 2 (semantic focus):**
- Does the implementation actually do what the PR claims?
- Are there cross-component interaction risks?
- Could concurrent execution produce surprising behavior?

The overlap dropped dramatically. Each reviewer found things the other couldn't — not because they were told to look for different things, but because focusing on one domain freed their attention for deeper analysis in that domain.

**What this means for the system:** "Use two reviewers" isn't enough. The reviewers need different *jobs*, not just different models. Identical mandates produce redundancy, not coverage.

---

## Architecture documents need different review than code

Code review and architecture review are fundamentally different analytical tasks:

| | Code review | Architecture review |
|---|---|---|
| Question | "Does this implementation follow patterns?" | "Does this design hold up under real-world conditions?" |
| Analytical mode | Within-frame: "given these rules, does this comply?" | Cross-boundary: "what must be true about the world for this to work?" |
| What gets missed | Subtle bugs, inconsistencies, drift from conventions | Hidden assumptions, failure modes, design tensions |
| Best model trait | Pattern matching, structural consistency | Reasoning about interactions across boundaries |

When we tested gap-finding on architecture documents:

- Non-reasoning models found risks **within the document's own frame** — "what could this mechanism fail at?"
- Reasoning models found risks **the document can't see** — "what must be true about the world for this mechanism to work?"

The difference isn't quantity. It's a qualitatively different kind of analysis. Reasoning models identify multi-component interactions, semantic mismatches between systems, and second-order effects that require thinking about things the document doesn't mention.

**Example findings only reasoning models produced:**
- "The reconciliation process assumes broker positions use the same date semantics as internal positions — but brokers often report trade-date while internal state uses settlement-date"
- "Queued messages during recovery will grow unbounded at market open when all users reconnect simultaneously"
- "The gate marked 'ready' before warmup completes — a race window where the system accepts work it can't process"

These aren't things you'd find by checking code against patterns. They require reasoning about the *relationship between the system and its environment*.

**What this means for the system:** Post-merge audit and self-review can use different analytical depths depending on what changed. Code-only PRs get pattern-focused review. PRs touching architecture or domain boundaries get reasoning-focused review.

---

## Context isolation removes rationalization

The most surprising finding wasn't about models at all — it was about context.

The session that writes code has already justified every decision. It "knows" why it left out that error handler (it decided it wasn't needed), why it chose that pattern (it reasoned through alternatives), why it didn't test that edge case (it determined it was unlikely).

A fresh session has none of this. It sees the diff and asks: "Why isn't this handled?" It doesn't have the built-in answer. The rationalization is gone.

**This is why self-review works even with the same model.** The model didn't change. The context did. Without the development conversation's justifications, the same model applies its full analytical capability without bias toward the existing implementation.

Combined with a different model (which brings genuinely different pattern recognition), you get the intersection: no rationalization AND different strengths. Either alone helps. Both together is where the system catches things that would otherwise ship.

---

## What these findings shaped

These aren't academic observations. They directly shaped the system architecture:

| Finding | System design decision |
|---------|----------------------|
| Models see differently | Twin review uses two different providers |
| Signal-to-noise ratio | Self-review gets only the diff, no dev conversation |
| Specialization > identical mandates | Each reviewer has a focused job description |
| Architecture needs reasoning | Post-merge audit uses deeper analysis on design changes |
| Context isolation removes rationalization | Self-review spawns a completely fresh session |
| Cheap + focused > expensive + broad | Triage uses cheap model with one question |

---

## Open questions we're still investigating

- **Does model placement matter?** (Model A investigates → Model B judges, vs. reversed.) We're A/B testing this with alternating PR assignments.
- **How often should review prompts evolve?** The lookback loop finds noise, but changing prompts too often prevents measuring improvement. Current answer: human-approved changes only, batched every 3 days.
- **Can the system bootstrap its own reference docs?** Models can extract patterns from source code, but a human must validate that extracted patterns are universal, not project-specific historical accidents.
- **What's the ceiling?** At some point, the reviews will find nothing because the code quality is genuinely high. Is that the goal, or does it mean the system has become blind to a new category of issue?

---

## Further reading

- [The Secret Sauce](the-secret-sauce.md) — How documentation makes the review system meaningful (reviews need something to check against)
- [Building Reference Docs](building-reference-docs.md) — How to create the ground truth that reviews verify against
- [Adoption Guide](adoption-guide.md) — How to set up the twin review system with specialized mandates
