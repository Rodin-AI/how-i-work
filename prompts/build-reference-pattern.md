# Build: Reference Pattern

Give this entire file to your agent. It will build a reference pattern doc, then validate it.

---

## Step 1: Build

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
```

---

## Step 2: Validate

After building the pattern doc, run this validation in a CLEAN SESSION
that has not seen the source code. The point is to test whether someone
can build a correct implementation from the doc alone.

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

## Done when

- [ ] Pattern doc identifies the most common unit of work
- [ ] Every file to create is listed with complete code
- [ ] "Why this structure?" explains all non-obvious decisions
- [ ] Validation implementation compiles and tests pass
- [ ] No differences found between validation output and real codebase patterns
