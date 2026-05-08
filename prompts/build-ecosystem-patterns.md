# Build: Ecosystem Patterns

Give this entire file to your agent. Fill in the placeholders first.

---

## What this is

An ecosystem pattern file documents how authoritative open-source projects
solve a specific concern — extracted from their source code, not their docs.
The result is a citable reference your agent checks before writing code.

**See a real example of the finished output:** A completed ecosystem patterns repo has this structure:
```
<lang>-patterns/
├── patterns/       # extracted patterns with file:line citations
├── smells/         # anti-patterns to avoid (also cited)
└── sources/        # raw analysis notes per project
```
Each pattern file follows: Sources analyzed → The pattern → Where they disagree → Anti-patterns → Recommendation.

---

## Placeholders to fill

- `<CONCERN>`: the topic (e.g., error handling, testing, concurrency, interfaces)
- `<LANGUAGE>`: the language (e.g., Go, TypeScript, Rust, Elixir)
- `<PROJECT 1/2/3>`: authoritative projects (see suggestions below)

### Suggested source projects by language

**Go:**
| Concern | Best sources |
|---------|-------------|
| Error handling | kubernetes/kubernetes, etcd-io/etcd, cockroachdb/cockroach |
| Concurrency | golang/go (stdlib), hashicorp/consul, nats-io/nats-server |
| Testing | golang/go (stdlib tests), prometheus/prometheus, vitessio/vitess |
| Interfaces | kubernetes/kubernetes, hashicorp/terraform, docker/moby |
| HTTP/API design | kubernetes/kubernetes, go-chi/chi, labstack/echo |
| Configuration | hashicorp/consul, spf13/viper, nats-io/nats-server |

**TypeScript:**
| Concern | Best sources |
|---------|-------------|
| Error handling | prisma/prisma, trpc/trpc, vercel/next.js |
| Testing | vitest-dev/vitest, playwright-community, jestjs/jest |
| Type patterns | trpc/trpc, prisma/prisma, tRPC/trpc |
| Module structure | vercel/next.js, remix-run/remix, angular/angular |
| State management | pmndrs/zustand, reduxjs/redux-toolkit, TanStack/query |
| Validation | colinhacks/zod, fakerjs/faker, Effect-TS/effect |

**Rust:**
| Concern | Best sources |
|---------|-------------|
| Error handling | rust-lang/rust (stdlib), dtolnay/anyhow, BurntSushi/ripgrep |
| Concurrency | tokio-rs/tokio, rayon-rs/rayon, crossbeam-rs/crossbeam |
| Testing | rust-lang/rust, tokio-rs/tokio, serde-rs/serde |
| Traits/generics | serde-rs/serde, rust-lang/rust (stdlib), tower-rs/tower |
| CLI patterns | clap-rs/clap, BurntSushi/ripgrep, sharkdp/fd |

**Elixir:**
| Concern | Best sources |
|---------|-------------|
| GenServer | elixir-lang/elixir, phoenixframework/phoenix, dashbitco/broadway |
| Error handling | elixir-lang/elixir, phoenixframework/phoenix, oban-bg/oban |
| Testing | elixir-lang/elixir, phoenixframework/phoenix, ecto (sandbox) |
| Supervision | elixir-lang/elixir (stdlib), nerves-project/nerves, membrane |
| Ecto patterns | elixir-ecto/ecto, phoenixframework/phoenix, ash-project/ash |

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
3. "Anti-patterns" — what they explicitly avoid (with evidence)
4. "Recommendation for our repos" — which approach to follow and why

Output format: <concern>.md in the patterns repo.
Structure:
  ## Sources analyzed (with versions)
  ## The pattern (with cited examples)
  ## Where they disagree
  ## Anti-patterns (what to avoid)
  ## Recommendation

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
- [ ] Anti-patterns section has at least one cited example
- [ ] Recommendation produces natural-looking code
- [ ] "Where they disagree" section covers at least one real divergence
