# Prompts: Building Reference Documentation

Copy-pasteable prompts for constructing each type of documentation. Give these to your agent with access to the codebase.

Each prompt has two parts:
1. **Construction** — builds the doc from scratch
2. **Validation** — tests whether the doc is actually useful (run this AFTER construction, ideally in a clean session that hasn't seen the source code)

---

## Domain Glossary

### Build it

```
I need you to build a domain glossary for this project.

1. Scan all struct/type names, database table names, API resource names,
   and package/module names in the codebase.
2. List every noun that represents a concept (not a utility — skip things
   like "handler", "service", "config").
3. For each term, write:
   - A 1-2 sentence definition of what it IS (not what it does)
   - Relationships to other terms ("belongs to", "contains", "references")
   - Constraints (valid values, uniqueness rules, immutability)
4. Group related terms together.
5. Flag any terms where the code uses the same word to mean different
   things in different places — those need disambiguation.

Output format: markdown file at docs/domain/glossary.md
Each term gets an H2 heading, definition paragraph, and relationship notes.

Quality check: for each term, ask "if I deleted the code and only had this
definition, could I reconstruct what the struct/table should contain?" If
no — the definition is too vague.
```

### Validate it

```
Using ONLY the glossary at docs/domain/glossary.md (do not read source code),
answer these questions:

1. What's the difference between [TERM A] and [TERM B]?
   (Pick two terms that are easily confused in this domain)
2. If I create a new [TERM], what other entities must already exist?
3. Can a [TERM] exist without a [RELATED TERM]? Why or why not?
4. What are the valid values for [CONSTRAINED FIELD]?
5. Draw the relationship between [TERM A], [TERM B], and [TERM C].

For each answer:
- If you could answer confidently from the glossary alone: PASS
- If you had to guess or assume: FAIL — note what's missing

Then verify against the source code: were your glossary-only answers correct?
Any wrong answer = the glossary is misleading (worse than incomplete).
```

---

## Conventions

### Build it

```
I need you to document the conventions used in this repo.

1. Read the 10 most recently merged PRs (diffs + full files touched).
2. For each of these categories, identify the CONSISTENT patterns:
   - File/package structure (where do new files go? naming?)
   - Error handling (how are errors created, wrapped, propagated, logged?)
   - Testing (where do tests live? naming? helpers? assertions library?)
   - Naming (functions, types, variables, constants)
   - Dependency injection (constructors? interfaces? global state?)
   - Validation (where does it happen? input boundary? domain layer?)
3. For each pattern you find:
   - State the convention as a rule ("Always X", "Never Y")
   - Show one real example from the codebase (file path + snippet)
   - Note any exceptions you found and whether they look intentional or accidental
4. For anything INCONSISTENT across the PRs:
   - Flag it explicitly: "INCONSISTENT: some files do X, others do Y"
   - Recommend which approach to standardize on (based on which is more common)

Output format: CONVENTIONS.md in the repo root.
Sections: Package Structure, Error Handling, Testing, Naming, plus any
other categories that emerged from the PRs.

Quality check: could a new contributor read this and write code that
passes review without asking questions? If any section requires "you
just have to know" tribal knowledge — it's incomplete.
```

### Validate it

```
Using ONLY CONVENTIONS.md (do not look at existing code), write the
following from scratch:

1. A new function that fetches a [DOMAIN OBJECT] by ID and handles
   the "not found" case. Include the error handling.
2. The test file for that function. Name it, place it in the right
   directory, use the right assertion library and style.
3. A new package/module for a feature called "notifications."
   What files do you create? What do you name them? Where do they go?

Then compare your output to how the codebase ACTUALLY does these things.

Scoring:
- Matches existing patterns exactly: PASS
- Close but minor differences: PARTIAL (note what the doc didn't specify)
- Significantly different from reality: FAIL (doc is misleading or incomplete)

For every PARTIAL or FAIL: what specific sentence or rule would fix the
conventions doc so this mistake can't happen again?
```

---

## Reference Pattern

### Build it

```
I need you to document a reference pattern for the most common unit
of work in this repo.

1. Look at the last 20 merged PRs. What type of change appears most often?
   (new endpoint, new worker, new command, new model, new integration)
2. Find the BEST existing example of that type — the one that's cleanest,
   most complete, and follows all conventions from CONVENTIONS.md.
3. Document it as a step-by-step blueprint:
   - Title: "Pattern: Adding a New <thing>"
   - One paragraph explaining WHAT this pattern is and WHEN to use it
   - "Files to create" — ordered list of every file, with a one-line purpose
   - For each file: complete, real, copy-pasteable code (not pseudocode)
   - "Tests" — what test cases are required (happy path, validation, errors)
   - "Why this structure?" — explain every non-obvious design decision

Output format: docs/domain/<pattern-name>.md

Quality check: if you deleted the example from the codebase and asked
a new developer to implement it using ONLY this document, would they
produce something that passes review? If any step requires guessing —
it's incomplete.
```

### Validate it

```
Using ONLY the reference pattern doc (do not look at existing code),
implement a new <THING> called "<NAME>".

Follow the pattern document exactly:
- Create every file it specifies
- Use the structure it shows
- Write the tests it requires
- Follow the conventions it references

Then review your implementation:
1. Does it compile/build without errors?
2. Do the tests pass?
3. Compare to a real example in the codebase — what's different?

For each difference:
- Was the pattern doc ambiguous? (could be read multiple ways)
- Was the pattern doc incomplete? (left out a step you had to infer)
- Was the pattern doc wrong? (told you to do X but real code does Y)

For every issue found: propose the exact text to add to the pattern doc
that would prevent this mistake.
```

---

## Ecosystem Patterns

### Build it

```
I need you to build an ecosystem pattern file for: <CONCERN>
(e.g., error handling, testing, concurrency, module structure)

Language: <LANGUAGE>
Source projects to analyze:
- <PROJECT 1> (latest stable release)
- <PROJECT 2> (latest stable release)
- <PROJECT 3> (latest stable release)

For each source project:
1. Find 5-10 representative files where <CONCERN> is handled
2. Extract the pattern: what do they consistently do?
3. Note the exact version/commit you analyzed

Then synthesize:
1. "The pattern" — what ALL sources agree on (with code examples from source)
2. "Where they disagree" — different approaches with context on why
3. "Recommendation for our repos" — which approach to follow and why

Output format: <concern>.md in the patterns repo.
Sections: Sources analyzed (with versions), The pattern, Where they
disagree, Recommendation.

Quality check: every claim must cite a specific file in a specific project.
"Projects generally do X" is not acceptable — "kubernetes/pkg/controller/foo.go
does X" is. If you can't cite it, don't claim it.
```

### Validate it

```
For each claim in <concern>.md:

1. Go to the cited file in the cited project at the cited version.
2. Confirm the code actually does what the doc says it does.
3. Check: has this file changed significantly since the cited version?
   (Look at current main branch — does the pattern still hold?)

Scoring:
- Claim matches source code: VERIFIED
- Claim is outdated (source changed since): STALE — note current behavior
- Claim doesn't match source (was wrong): WRONG — remove or correct
- Claim has no citation: UNVERIFIABLE — add citation or delete

Also test the recommendation:
- Write a small piece of code following the "Recommendation" section.
- Does it feel natural in the language? Would it pass review on any of
  the source projects? If it would look out of place in kubernetes or
  etcd, the recommendation might be aspirational rather than grounded.
```

---

## Implementation Docs

### Build it

```
I need you to document how <SUBSYSTEM> works internally.

1. Trace the flow from entry point to exit:
   - What triggers this subsystem? (HTTP request, queue message, timer, etc.)
   - What are the steps in order?
   - What state is read/written at each step?
   - What can fail at each step and what happens when it does?
2. Document the data flow:
   - What goes in (shape, source)
   - What transformations happen
   - What comes out (shape, destination)
3. Document operational concerns:
   - How do you know it's healthy? (metrics, logs, symptoms of failure)
   - What are the scaling limits?
   - What are the dependencies? (if X is down, what breaks?)
4. Include a diagram (mermaid) showing the flow.

Output format: docs/impl/<subsystem>.md

Quality check: "Would this fact survive a complete rewrite?" — if yes,
it belongs in domain docs, not here. Implementation docs should describe
HOW things work today, knowing that the HOW may change. If you catch
yourself writing "an Order is..." that's a glossary entry, not impl docs.
```

### Validate it

```
Simulate this scenario: <SUBSYSTEM> has stopped producing output.
Using ONLY docs/impl/<subsystem>.md:

1. What are the possible failure points? (List them from the doc)
2. For each failure point, what would you check first? (metrics, logs, state)
3. What are the dependencies — what upstream failures could cause this?
4. If the problem is at step N, what's the blast radius downstream?

Scoring:
- Could identify all plausible failure points from the doc: PASS
- Had to guess about failure modes: FAIL (doc doesn't cover what can go wrong)
- Could trace dependencies: PASS
- Had to guess about dependencies: FAIL (doc doesn't cover integration points)
- Could assess blast radius: PASS
- Had to guess what's downstream: FAIL (doc doesn't show the full flow)

Then compare your answers against the actual code. Did the doc lead you
to the right places? Would on-call be able to use this doc at 3am to
diagnose the issue without reading source code?
```

---

## Feature Design

### Build it

```
I need to write a feature design doc for: <FEATURE>

Walk me through these questions (ask one at a time):

1. What problem does this solve? (not "what does it do" — what PROBLEM)
2. Who experiences the problem? (user, operator, developer, system)
3. What does success look like? (measurable outcome, not feature description)
4. What are the constraints? (must work with X, can't break Y, budget of Z)
5. What approaches did you consider? (at least 2 — if only 1, think harder)
6. Why did you pick this approach over the others?
7. What are the risks? (what could go wrong, what are you not sure about)
8. What's explicitly OUT OF SCOPE? (what this feature will NOT do)

After gathering answers, generate a design doc with:
- Problem statement (from #1-3)
- Constraints (from #4)
- Decision: chosen approach with rationale (from #5-6)
- Risks and mitigations (from #7)
- Scope boundary (from #8)
- Acceptance criteria: how to verify the feature is DONE (testable statements)

Output format: docs/designs/<feature>.md

Quality check: could someone implement this feature WITHOUT talking to
you further? If any acceptance criterion is ambiguous ("works well",
"handles errors properly") — it's not specific enough.
```

### Validate it

```
Using ONLY docs/designs/<feature>.md, answer:

1. For each acceptance criterion: write the test that proves it's met.
   (If you can't write a concrete test — the criterion is too vague.)
2. Are there any implicit requirements NOT stated in the doc that you'd
   need to know? (e.g., "it should be fast" — how fast? "handle errors" — which ones?)
3. What would you build FIRST? Can you determine implementation order
   from the doc alone, or do you need to ask?
4. Read the "out of scope" section. Now: is there anything in the
   acceptance criteria that contradicts the scope boundary?

Scoring:
- Every criterion has a writable test: PASS
- Any criterion requires clarification: FAIL (criterion is ambiguous)
- Implementation order is determinable: PASS
- Order requires discussion: PARTIAL (add sequencing guidance)
- Scope and criteria are consistent: PASS
- Contradiction found: FAIL (scope and criteria disagree)

For each FAIL: propose the specific text change that resolves it.
```
