# Prompts

Copy-pasteable prompts for setting up and building documentation. Each file is self-contained — give it to your agent as-is.

## Setup

| Prompt | What it does |
|--------|-------------|
| [setup.md](setup.md) | 5-phase guided setup with quality gates — takes you from zero to running system |

## Building Documentation

Each doc type has its own prompt with two parts: **Build** (construct the doc) and **Validate** (test that it's actually useful). Always run both.

| Prompt | What it builds |
|--------|---------------|
| [build-glossary.md](build-glossary.md) | Domain glossary — precise definitions of every concept in your system |
| [build-conventions.md](build-conventions.md) | Conventions doc — how code is written in THIS repo |
| [build-reference-pattern.md](build-reference-pattern.md) | Reference pattern — step-by-step blueprint for the most common unit of work |
| [build-ecosystem-patterns.md](build-ecosystem-patterns.md) | Ecosystem patterns — how authoritative open-source projects solve a concern |
| [build-impl-docs.md](build-impl-docs.md) | Implementation docs — how a subsystem works internally |
| [build-feature-design.md](build-feature-design.md) | Feature design — what you decided, why, and how to verify it's done |

## How to use

1. Pick the doc type you need
2. Open that file
3. Fill in the `<PLACEHOLDERS>` with your specifics
4. Give the entire file to your agent (it contains both build and validate instructions)
5. Agent builds the doc, then validates it
6. Fix any FAIL/PARTIAL results before considering the doc complete
