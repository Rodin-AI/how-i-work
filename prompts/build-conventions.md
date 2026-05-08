# Build: Conventions Doc

Give this entire file to your agent. It will build the conventions doc, then validate it.

---

## Step 1: Build

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
```

---

## Step 2: Validate

After building conventions, run this validation in a CLEAN SESSION that
has not seen the source code. The point is to test whether the conventions
doc alone produces correct code.

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

## Done when

- [ ] Every category has at least one explicit rule with a real example
- [ ] No category is marked INCONSISTENT without a recommendation
- [ ] Validation produces code that matches codebase patterns exactly
- [ ] No FAIL results remain unaddressed
