# Build: Ecosystem Patterns

Give this entire file to your agent. Fill in the placeholders first.

---

## Placeholders to fill

- `<CONCERN>`: the topic (e.g., error handling, testing, concurrency)
- `<LANGUAGE>`: the language (e.g., Go, TypeScript, Rust)
- `<PROJECT 1/2/3>`: authoritative open-source projects to analyze

---

## Step 1: Build

```
I need you to build an ecosystem pattern file for: <CONCERN>

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

RULE: every claim must cite a specific file in a specific project.
"Projects generally do X" is not acceptable — "kubernetes/pkg/controller/foo.go
does X" is. If you can't cite it, don't claim it.
```

---

## Step 2: Validate

After building the pattern file, run this validation. This CAN use
source code access — the point is to verify claims are real.

```
For each claim in the pattern file:

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

## Done when

- [ ] Every claim cites a specific file and version
- [ ] All citations verified against actual source code
- [ ] Zero WRONG claims remain
- [ ] Recommendation produces natural-looking code
- [ ] "Where they disagree" section covers at least one real divergence
