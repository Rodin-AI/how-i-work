# Build: Domain Glossary

Give this entire file to your agent. It will build the glossary, then validate it.

---

## Step 1: Build

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
```

---

## Step 2: Validate

After building the glossary, run this validation in a CLEAN SESSION that
has not seen the source code. The point is to test whether the glossary
alone is sufficient.

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

## Done when

- [ ] Every domain concept has a definition
- [ ] Relationships between terms are explicit
- [ ] Validation passes with zero FAILs
- [ ] No glossary-only answer was WRONG when verified against code
