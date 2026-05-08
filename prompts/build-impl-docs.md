# Build: Implementation Docs

Give this entire file to your agent. Fill in the placeholder first.

---

## Placeholder to fill

- `<SUBSYSTEM>`: the subsystem to document (e.g., order pipeline, notification worker, auth flow)

---

## Step 1: Build

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

RULE: "Would this fact survive a complete rewrite?" — if yes, it belongs
in domain docs, not here. Implementation docs describe HOW things work
today, knowing that the HOW may change. If you catch yourself writing
"an Order is..." that's a glossary entry, not impl docs.
```

---

## Step 2: Validate

After building the impl doc, run this validation. The point is to test
whether someone can USE this doc to operate and debug the system.

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

## Done when

- [ ] Entry point, steps, and exit are all documented
- [ ] Every step has "what can fail" and "what happens when it does"
- [ ] Dependencies are listed explicitly
- [ ] Mermaid diagram included
- [ ] Validation: all failure points identifiable from doc alone
- [ ] Validation: dependencies traceable without guessing
