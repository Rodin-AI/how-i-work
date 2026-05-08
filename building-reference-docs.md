# Building Reference Documentation

Without reference docs, an agent writes code that compiles. With reference docs, it writes code that *belongs in your codebase*. This page tells you what to build, how to build it, and how to keep it alive.

## Contents

1. [What to Build](#part-1-what-to-build) — the six types, where they live, how to decide
2. [How to Write Each One](#part-2-how-to-write-each-one)
   - [Domain glossary](#domain-glossary)
   - [Conventions](#conventions)
   - [Reference patterns](#reference-patterns)
   - [Ecosystem patterns](#ecosystem-patterns)
3. [Keeping Documentation Alive](#part-3-keeping-documentation-alive) — rot rates, detection, ownership
4. [Getting Started: The 4-Hour Bootstrap](#getting-started-the-4-hour-bootstrap)

---

## Part 1: What to Build

You need six types of documentation. Each answers a different question an agent will ask while working on your code.

| Type | Question it answers | Example answer |
|------|-------------------|----------------|
| Domain glossary | "What IS this thing?" | "An Order is a request to buy/sell..." |
| Conventions | "How do we do things HERE?" | "Errors are always wrapped with context" |
| Reference patterns | "Show me how to add a new X" | Complete handler + service + tests |
| Ecosystem patterns | "How does the ecosystem do this?" | Error handling from kubernetes, etcd |
| Implementation docs | "How does this subsystem work internally?" | "The pipeline uses streaming..." |
| Feature designs | "What did we decide and why?" | Design doc for escalation system |

### Where each type lives

| Type | Location |
|------|----------|
| Domain glossary | `your-project/docs/domain/glossary.md` |
| Conventions | `your-project/CONVENTIONS.md` |
| Reference patterns | `your-project/docs/domain/<pattern>.md` |
| Ecosystem patterns | `your-org/<lang>-patterns/` (separate repo) |
| Implementation docs | `your-project/docs/impl/<topic>.md` |
| Feature designs | `your-project/docs/designs/<feature>.md` |

### How to decide where something goes

Ask: "Would this fact survive a complete rewrite of the internals?"

- **Yes** → domain docs (glossary, reference patterns)
- **No** → implementation docs
- **The what survives but the how might change** → conventions

---

## Part 2: How to Write Each One

Each section below covers one type: what it is, the process, pitfalls, and a link to a complete example. For each type, there's also a **copy-pasteable prompt** you can give to an agent to do the construction work: [Building docs prompts](prompts/building-docs.md).

### Domain glossary

Every noun in your system — module names, database tables, API resources — gets a precise definition. Not what the word means in English. What it means in YOUR system, including relationships to other terms and any constraints.

**Process:** Scan your codebase for nouns (struct names, table names, endpoint resources). For each one, write what it IS, what it relates to, and what constraints it has.

→ **[See complete example](examples/domain-glossary.md)** · **[Construction prompt](prompts/building-docs.md#domain-glossary)**

**Pitfalls:**
- Too vague ("Order — something a user creates")
- Too implementation-specific ("Order — a row in the orders table")
- Missing relationships (defining terms in isolation)

**Time:** 1-2 hours for 15-30 terms.

---

### Conventions

How code is written in THIS repo. Not aspirational rules — a description of the patterns that already exist. Read your 5 most recent PRs, identify what's consistent, write it down. Where things are inconsistent, make a decision and document it.

**Process:** Open recent PRs. Note file naming, error handling, test locations, dependency injection. Write down what repeats. Decide where things conflict.

→ **[See complete example](examples/conventions.md)** · **[Construction prompt](prompts/building-docs.md#conventions)**

**Pitfalls:**
- Describing what you WISH the code looked like (document reality, then fix reality)
- Too abstract ("follow good practices" — which ones?)
- Not covering error handling (the #1 source of inconsistency)

**Time:** 2-3 hours. Mostly reading existing code.

---

### Reference patterns

A step-by-step blueprint for the most common unit of work in your repo. When the agent needs to "add a new endpoint" or "add a new worker," this is the document it follows. Think: what you'd hand a new teammate on day one.

**Process:** Pick the thing you add most often. Find the best existing example. Document it: files to create, complete code, tests to write, and — critically — WHY it's structured this way.

→ **[See complete example](examples/reference-pattern.md)** · **[Construction prompt](prompts/building-docs.md#reference-pattern)**

**Pitfalls:**
- Pseudocode instead of real, copy-pasteable code
- No WHY section (showing what without explaining why)
- Only the happy path (missing error cases and tests)

**Time:** 1-2 hours per pattern. Start with your most common one.

---

### Ecosystem patterns

Curated knowledge about how authoritative open-source projects solve common problems in your language. NOT opinions or "best practices" blog posts — reality extracted from source code of projects with hundreds of contributors.

**Key difference from the types above:** Everything else is per-project. Ecosystem patterns are per-language — build them once, reference them from every project.

**Process:** Pick 3-5 authoritative projects (Go: kubernetes, etcd, cockroachdb. TypeScript: next.js, prisma, trpc). Pick a concern (error handling, testing, concurrency). Read their source code — not docs, source. Document what you find, including where they disagree.

→ **[See complete example](examples/ecosystem-pattern.md)** · **[Construction prompt](prompts/building-docs.md#ecosystem-patterns)**

**Pitfalls:**
- Documenting what you THINK the community does (verify against source)
- No version pinning (cite which version you analyzed)
- Ignoring disagreements (where projects diverge is where the interesting judgment lives)

**Time:** 4-8 hours for an initial 5-file repo. Compounds forever.

---

## Part 3: Keeping Documentation Alive

Stale docs are worse than no docs. They cause the agent to confidently write wrong code. This section is about making freshness sustainable.

### Rot rates

| Type | How fast it goes stale | Why |
|------|----------------------|-----|
| Implementation docs | Weeks | Code changes, docs don't follow |
| Conventions | Months | Practices evolve silently |
| Reference patterns | Months | New patterns emerge |
| Domain glossary | Slowly | Terms are inherently stable |
| Ecosystem patterns | 6-12 months | Upstream evolves |

### Detection

The post-merge audit loop catches staleness automatically (merged PR contradicts docs → files an issue). Proactively, run a monthly check: for each documented claim, search the codebase for counter-examples. If >30% of instances don't follow the doc, it's stale.

**Signals something is off:**
- Triage flags increasing (agent can't find answers)
- Self-review cites docs but code "disagrees"
- New modules don't follow documented patterns
- Lookback finds the agent keeps doing X despite docs saying Y

### Who fixes what

| Type | Owner | Trigger |
|------|-------|---------|
| Domain glossary | Human | New concepts emerge |
| Conventions | Agent proposes, human approves | Pattern drift detected |
| Reference patterns | Agent proposes, human approves | Better example exists |
| Ecosystem patterns | Quarterly scheduled refresh | Upstream major release |
| Implementation docs | Agent (autonomous) | PR changes architecture |
| Feature designs | Human | Before starting a feature |

### The documentation PR

When the agent finds stale docs, it doesn't silently fix them. It opens a PR that ONLY updates docs (no code), explaining: "Code does X, docs say Y, evidence: [files]." The human decides: is the code right (update docs) or are the docs right (file a bug)?

### Why this matters

**The doom loop:** docs wrong → agent writes wrong code → reviews catch it → more iterations → no time to fix docs → docs get worse.

**The flywheel:** docs right → agent writes correct code → reviews find nothing → time invested in docs → docs stay right.

---

## Getting Started: The 4-Hour Bootstrap

| Hour | What to do | Output |
|------|-----------|--------|
| 1 | Scan module names and tables, define 15-20 terms | `docs/domain/glossary.md` |
| 2 | Read 5 recent PRs, document repeating patterns | `CONVENTIONS.md` |
| 3 | Find best existing example, document step-by-step | `docs/domain/<your-pattern>.md` |
| 4 | Run gap-finding check, note what's still missing | Issue backlog |

Four hours gets you from "agent guesses at everything" to "agent has a foundation." The loops will surface what's still missing.

---

## Further reading

- [The Secret Sauce](the-secret-sauce.md) — Why documentation is the fuel that makes loops work
- [Adoption Guide](adoption-guide.md) — How to set up the loops that USE these docs
- [Scaling to Multiple Repos](scaling-multiple-repos.md) — How ecosystem pattern repos amortize across projects
- [Examples directory](examples/) — Complete worked examples of every doc type
