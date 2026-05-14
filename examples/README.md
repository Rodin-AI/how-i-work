# Examples

Real, working prompt structures for each loop. These aren't minimal sketches — they're the actual patterns that make autonomous dev reliable in practice.

## Before you read these

Each example references two things you need to set up first:

1. **A project config file** (`memory/projects/<project>.yaml`) — repo URL, token path, node target, label IDs. One file per project, shared across all loops.

2. **A skill file** (`~/.openclaw/workspace/skills/<skill>/SKILL.md`) — the logic for each loop. The cron prompt tells the agent to read the skill file; the skill file drives everything.

This separation is intentional. Your cron prompt stays short and readable. Your logic lives in a file you can edit and version. Your project specifics live in a config file you can copy when adding a second project.

## The loops

| Loop | File | How often | What it does |
|------|------|-----------|-------------|
| [Triage](triage.md) | `project-triage` skill | Every 30 min | Surface stuck work — CI failures, stale reviews, merge conflicts |
| [Dev Loop](dev-loop.md) | `dev-loop` skill | Every 10 min | Dispatch work to a worker when there's something to do |
| [Post-Merge Audit](post-merge-review.md) | `post-merge-review` skill | Every hour | Verify merged PRs delivered what the issue asked for |
| [Free Time](free-time-work.md) | _(inline)_ | Every 20 min | Work on maintenance tasks when nothing else is in flight |
| [PR Ready Checklist](pr-ready-checklist.md) | _(standing order)_ | Before every handoff | Gate: verify CI, reviews, and rebase before marking ready |

## The contracts that matter

Every loop must follow these contracts or the system breaks down:

**`NO_REPLY`** — When there's nothing to report, the model responds with exactly `NO_REPLY`. Your runtime must treat this as "don't deliver." Without this contract, every run notifies you, you learn to ignore the channel, and you miss real problems.

**WIP ≤ 1** — No loop starts new work if there's already an open PR. The dev loop and free-time loop both check this explicitly. Parallel in-flight work creates merge conflicts, context fragmentation, and unfinished PRs.

**Explicit skill path** — Cron prompts name the exact skill file to read: `Read ~/.openclaw/workspace/skills/<skill>/SKILL.md and follow it exactly.` Not "check my PRs." The agent doesn't hunt for instructions.

**State files for deduplication** — Jobs that should run once per event (post-merge audit) track state in a JSON file. Without this, the same PR gets audited on every run.
