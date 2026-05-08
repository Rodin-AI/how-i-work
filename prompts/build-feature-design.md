# Build: Feature Design Doc

Give this entire file to your agent. Fill in the placeholder first.

---

## Placeholder to fill

- `<FEATURE>`: the feature to design (e.g., rate limiting, user notifications, audit trail)

---

## Step 1: Build

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
```

---

## Step 2: Validate

After building the design doc, run this validation. The point is to test
whether someone can IMPLEMENT this feature from the doc alone without
asking further questions.

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

---

## Done when

- [ ] Problem is stated as a problem (not a feature description)
- [ ] At least 2 approaches considered with rationale for the chosen one
- [ ] Every acceptance criterion can have a concrete test written for it
- [ ] Out of scope section exists and doesn't contradict acceptance criteria
- [ ] A developer could implement this without asking further questions
