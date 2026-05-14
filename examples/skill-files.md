# Skill Files

Loop logic lives in skill files, not inline in cron prompts. This is the pattern that keeps the system maintainable.

## Why not inline prompts?

A cron prompt with 200 words of inline logic is:
- Hard to read
- Hard to update (you have to edit the cron job config)
- Impossible to version without versioning the cron job
- Copy-pasted differently across loops, diverging over time

A skill file is a plain markdown file in your workspace. Edit it, commit it, improve it — the cron job never changes.

## Location convention

```
~/.openclaw/workspace/skills/<skill-name>/SKILL.md
```

## How the cron prompt references it

Every cron prompt that delegates to a skill uses this exact pattern:

```
Read ~/.openclaw/workspace/skills/<skill-name>/SKILL.md and follow it exactly.
Load project config from ~/.openclaw/workspace/memory/projects/<project>.yaml.
```

The phrase "and follow it exactly" matters. Without it, the model treats the skill file as a suggestion.

## Skill file structure

A skill file should contain:

```markdown
# <skill name>

## Trigger
What message or condition activates this skill.

## Steps
Numbered, sequential. Each step is a concrete action.
No ambiguity. No "check if needed." Either do it or skip it with a stated condition.

## Rules
Hard constraints. Things the skill must never do.
Each rule on its own line, stated as a prohibition.

## Output
What the skill produces: a message, a file, a PR, NO_REPLY.
State the NO_REPLY condition explicitly.
```

## The loops and their skill files

| Loop | Skill file location |
|------|-------------------|
| Dev Loop | `skills/dev-loop/SKILL.md` |
| Triage | `skills/project-triage/SKILL.md` |
| Post-Merge Audit | `skills/post-merge-review/SKILL.md` |
| Self-Review | `skills/self-review/SKILL.md` |

## Example skill file (minimal)

```markdown
# project-triage

## Trigger
Message: "Run the project-triage skill" with a config path.

## Steps
1. Load project config
2. Fetch open PRs via API
3. For each PR: check CI, review state, merge conflicts
4. Report stuck items (see thresholds in config)
5. If nothing stuck: NO_REPLY

## Rules
- Do not attempt to fix anything. Observe and report only.
- Do not report items that are not stuck.

## Output
Bullet list of stuck items, one per line. Or NO_REPLY.
```
