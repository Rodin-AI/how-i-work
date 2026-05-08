# What We Learned: Why Multi-Model Systems Work

This is an automated code review pipeline that uses multiple AI models in different roles — writing, self-reviewing, and twin-reviewing every PR. This page documents what we discovered through 50+ experiments about why it works and what makes it break.

The full experiment data, methodology, and per-finding details are published in our [model-research repo](https://gitea.weiker.me/rodin/model-research).

---

## How to read this

The summary table tells you what we found and what we did about it. The sections below explain the evidence.

| Finding | System design decision |
|---------|----------------------|
| Claude Sonnet catches structural issues; GPT-5 catches semantic issues | Twin review uses two different providers with specialized prompts |
| Signal buried in noise gets missed regardless of model | Self-review gets only the diff, no dev conversation |
| Same prompt to both reviewers = redundancy, not coverage | Each reviewer has a focused job description (NOT "review this PR") |
| Architecture docs need reasoning about the world, not pattern matching | Design-touching PRs get a third reviewer focused on contradictions |
| A fresh session catches what the author session can't | Self-review spawns a completely fresh session (context isolation) |
| GPT-4.1 Mini + one precise question > GPT-5 + broad mandate | Triage uses cheap model with one question; expensive models for deep work |

---

## Different models see differently

We ran Claude Sonnet and GPT-5 on the same PRs with the same prompt ("review this PR for correctness, idiomatic code, potential bugs, and design issues"). Across 50+ reviews:

**Claude Sonnet consistently found:**
- Broken cross-references between files
- Missing type specifications on public functions
- Pattern deviations from documented conventions
- Dead code and unused imports

**GPT-5 consistently found:**
- Logic that contradicted what the PR title claimed to implement
- Race conditions with specific interleaving sequences described
- Cross-module assumption violations ("module A assumes X, but module B doesn't guarantee X")
- Financial calculation edge cases that produce wrong values silently

These aren't random — they're repeatable. Given the same PR, Sonnet gravitates toward form and GPT-5 gravitates toward meaning. Neither is "better." They cover different failure modes.

**Real example:** On a PR adding a new order validation handler:
- Sonnet found: missing `@spec` on 2 public functions, one unused alias, test file not following naming convention
- GPT-5 found: the validation bypasses the position limit check when the order quantity is exactly equal to the remaining capacity (off-by-one in `>=` vs `>`)

Both findings were legitimate. The off-by-one would have shipped without GPT-5. The missing specs would have shipped without Sonnet. Neither model found both.

---

## Signal-to-noise ratio matters more than model capability

We ran an experiment: 12 research hypotheses contained directional bias (words like "inevitably," "must," "requires" pushing toward predetermined conclusions).

**Condition A:** GPT-4.1 Mini given ONLY the 12 hypotheses + one question: "Do any lead toward a predetermined conclusion?"
→ Found all 12 biased. Cost: ~$0.003.

**Condition B:** Claude Sonnet and GPT-5 given the same hypotheses buried inside a full PR review (diff, file content, issue description, acceptance criteria, project conventions) and asked "review this PR."
→ Both approved. Neither flagged the bias.

**Condition C (the reveal):** Same Sonnet and GPT-5 given ONLY the hypotheses (noise removed), asked the same broad "review quality, clarity, and issues" question.
→ Both found all 12 biased.

**The insight isn't "small models are better."** It's that when the signal is buried in noise (a 50,000-token PR review context), even powerful models lose it. Isolate the signal, and any model finds it.

This directly shaped the system:
- **Self-review** strips everything except the diff — no development conversation, no trade-off reasoning, no justifications. The diff IS the signal.
- **Triage** asks one question ("is anything stuck?") against a clean state summary — not "review the repo"
- **Post-merge audit** compares exactly two things: the issue's acceptance criteria vs the merged code. Nothing else.

Full experiment details: [Finding #2 and #8 in model-research](https://gitea.weiker.me/rodin/model-research)

---

## Specialization beats identical mandates

When both reviewers got "review this PR," they produced overlapping findings and missed entire categories. We split them:

**Claude Sonnet's actual prompt (structural focus):**
```
Your role is STRUCTURAL review — correctness of form, not meaning.
Your Domain: pattern compliance, spec completeness, structural 
correctness, formatting, test coverage.
NOT Your Domain: race conditions, cross-component semantics,
financial domain correctness.
```

**GPT-5's actual prompt (semantic focus):**
```
Your role is SEMANTIC review — correctness of meaning, not form.
Your Domain: semantic correctness, cross-component interactions,
race conditions, domain-specific risks, assumption identification.
NOT Your Domain: missing specs, formatting, pattern compliance,
broken references.
```

The "NOT Your Domain" section is critical. Without it, models default to broad review and duplicate each other's work. With explicit exclusions, they go deeper into their assigned domain because they're not spending attention budget on the other model's territory.

**Result:** Overlap dropped. Unique findings per reviewer increased. Neither reviewer produces findings in the other's domain — the partition is clean.

---

## Architecture documents need different review than code

We tested gap-finding on architecture documents (failure modes, recovery procedures, system design) across GPT-4.1 Mini, GPT-4.1, and GPT-5:

| Model | Time | Findings | Type of finding |
|-------|------|----------|----------------|
| GPT-4.1 Mini | 16s | 10 | Formulaic: "what could this mechanism fail at?" |
| GPT-4.1 | 24s | 15 | Thorough within-frame, some meta-observations |
| GPT-5 | 45s | 14 | Cross-boundary: "what must the world guarantee for this to work?" |

GPT-5 found fewer total items but qualitatively different ones. Its unique findings:

- "The reconciliation process assumes broker positions use trade-date semantics — but the internal ledger uses settlement-date. These diverge over weekends."
- "Queued messages during recovery grow unbounded at market open when all users reconnect simultaneously — the mailbox isn't bounded."
- "The gate is marked 'ready' before worker warmup completes — a 200ms race window where the system accepts work it can't process."
- "Rate-limiting responses (429) don't trigger the 'connection lost' detection — the system thinks it's connected but can't trade."

GPT-4.1 and Mini found risks the document itself describes. GPT-5 found risks that require reasoning about the relationship between the system and things it connects to (brokers, clocks, network topology, concurrent users).

**Reasoning tokens produce a different analytical mode** — not just "more of the same" but thinking about what ISN'T stated. For architecture docs where missing an assumption means months of wrong implementation, this justifies the cost.

Full data: [Findings #9, #10, #11 in model-research](https://gitea.weiker.me/rodin/model-research)

---

## Context isolation removes rationalization

The most surprising finding: **the same model in a fresh session catches things it missed when it wrote the code.**

This isn't about different capabilities. The development session has justified every decision in its context window. It "decided" not to add that error handler. It "reasoned through" alternatives and picked this pattern. Those justifications are in-context and act as rationalization — the model won't question what it already defended.

A fresh session sees only the diff. No justifications. No trade-off reasoning. No "I already decided this is fine." It asks "why isn't this handled?" because it genuinely doesn't know the answer.

**Combined with a different model**, you get the intersection:
- Context isolation removes rationalizations (the model can't dismiss what it never justified)
- Different model brings different pattern recognition (different architectural strengths)

Either alone helps. Both together is where PRs get caught that would otherwise ship.

---

## Open questions we're still investigating

- **Does model placement matter?** (Model A investigates → Model B judges, vs. reversed.) We're A/B testing with alternating PR assignments.
- **How often should review prompts evolve?** The lookback loop finds noise, but changing prompts too often prevents measuring improvement. Current answer: human-approved changes only, every 3 days.
- **Can the system bootstrap its own reference docs?** Models can extract patterns from source code, but a human must validate that extracted patterns are universal vs project-specific accidents.
- **What's the ceiling?** When reviews consistently find nothing, is that success or blindness? We don't know yet.

---

## Further reading

- [Full experiment data and methodology](https://gitea.weiker.me/rodin/model-research) — 50+ experiments with raw results, prompts, and per-model comparisons
- [The Secret Sauce](the-secret-sauce.md) — How documentation makes the review system meaningful (reviews need something to check against)
- [Building Reference Docs](building-reference-docs.md) — How to create the ground truth that reviews verify against
- [Adoption Guide](adoption-guide.md) — How to set up the twin review system with specialized mandates
