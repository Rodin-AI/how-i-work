# Build: Ecosystem Patterns

Give this entire file to your agent. Fill in the placeholders first.

---

## What this is

An ecosystem pattern file documents how authoritative open-source projects solve a specific concern — extracted from their source code, not their docs. The result is a citable reference your agent checks before writing code.

**Real example:** [Rodin-AI/go-patterns](https://github.com/Rodin-AI/go-patterns) — clone it, read it, make yours look like that.

---

## Placeholders to fill

- `<CONCERN>`: the topic (e.g., error handling, testing, concurrency, interfaces)
- `<LANGUAGE>`: the language
- `<PROJECT 1/2/3>`: 3 authoritative open-source projects in that language (large, mature, many contributors — pick ones you'd trust to get it right)

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

Then synthesize into a single file with these sections:
1. "Sources analyzed" — project names + exact versions/commits
2. "The pattern" — what ALL sources agree on (with code from source)
3. "Where they disagree" — different approaches with context on why
4. "Anti-patterns" — what they explicitly avoid (with evidence)
5. "Recommendation" — which approach to follow and why

RULE: every claim must cite a specific file in a specific project.
"Projects generally do X" is not acceptable — show the file path.

Use https://github.com/Rodin-AI/go-patterns as a structural reference
for what the output should look like.
```

---

## Step 2: Validate

```
For each claim in the pattern file:

1. Go to the cited file at the cited version
2. Confirm the code does what the doc says
3. Check if the file has changed significantly on current main

Scoring:
- Matches source: VERIFIED
- Source changed since: STALE — note current behavior
- Doesn't match (was wrong): WRONG — correct or remove
- No citation: UNVERIFIABLE — add citation or delete

Then test the recommendation: write code following it. Does it look
natural? Would it pass review on the source projects?
```

---

## Done when

- [ ] Every claim cites a specific file and version
- [ ] All citations verified against source
- [ ] Zero WRONG claims remain
- [ ] Recommendation produces natural-looking code
