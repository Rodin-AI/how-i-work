# Examples

Real, working prompt structures for each loop. These aren't minimal sketches — they're the actual patterns that make autonomous dev reliable in practice.

## Start here: shared concepts

Three docs cover the patterns that every loop relies on. Read these before the individual loop docs.

| Doc | What it covers |
|-----|----------------|
| [Project Config](project-config.md) | The YAML file every loop reads — full field reference, which loops use which fields |
| [Skill Files](skill-files.md) | Why logic lives in skill files, not cron prompts — location convention, structure |
| [Cron Setup](cron-setup.md) | Model/timeout guide, tool allowlists, NO_REPLY contract, delivery modes |

## The loops

| Loop | File | How often | What it does |
|------|------|-----------|-------------|
| [Triage](triage.md) | `project-triage` skill | Every 30 min | Surface stuck work — CI failures, stale reviews, merge conflicts |
| [Dev Loop](dev-loop.md) | `dev-loop` skill | Every 10 min | Dispatch work to a worker when there's something to do |
| [Post-Merge Audit](post-merge-review.md) | `post-merge-review` skill | Every hour | Verify merged PRs delivered what the issue asked for |
| [PR Ready Checklist](pr-ready-checklist.md) | _(standing order)_ | Before every handoff | Gate: verify CI, reviews, and rebase before marking ready |

## The contracts that matter

Every loop must follow these contracts or the system breaks down:

**`NO_REPLY`** — When there's nothing to report, the model responds with exactly `NO_REPLY`. Your runtime must treat this as "don't deliver." Full explanation in [cron-setup.md](cron-setup.md).

**WIP ≤ 1** — No loop starts new work if there's already an open PR. The dev loop and free-time loop both check this explicitly. Parallel in-flight work creates merge conflicts, context fragmentation, and unfinished PRs.

**Explicit skill path** — Cron prompts name the exact skill file to read. Not "check my PRs" — the agent doesn't hunt for instructions.

**State files for deduplication** — Jobs that should run once per event (post-merge audit) track state in a JSON file. Without this, the same PR gets audited on every run.
