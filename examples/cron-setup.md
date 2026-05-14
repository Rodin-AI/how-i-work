# Cron Job Setup

Every loop follows the same setup pattern. This doc covers the parts that are the same across all loops — model choice, timeouts, tool allowlists, delivery, and the NO_REPLY contract. Each loop's own doc covers what's unique to it.

## The NO_REPLY contract

This is the most important configuration decision for any loop.

**Without it:** every run notifies you. 95% say "nothing to report." You learn to ignore the channel. The one time something is actually wrong, you ignore that too.

**With it:** silence means "all good." A message means "something needs your attention." That's the right information density.

Every cron prompt must end with one of these (pick the one that fits):
```
If nothing to report, respond with exactly NO_REPLY.
If no new PRs were audited, respond with exactly NO_REPLY.
If no work was done, respond with exactly NO_REPLY.
```

And your runtime must be configured to suppress delivery when the response is `NO_REPLY`. In OpenClaw: set `delivery.mode` to `none` for jobs where you handle delivery selectively, or `announce` + `bestEffort: true` for jobs that should deliver only when there's output.

## Model and timeout guide

| Loop | Model | Thinking | Timeout | Why |
|------|-------|----------|---------|-----|
| Dev dispatcher | Fast (Haiku, GPT-4.1 Mini) | off | 300s | API reads + spawn decision only. No reasoning needed. |
| Dev worker | Strong (Sonnet, Opus) | medium | 600s | Real implementation work. Needs capability and time. |
| Triage | Sonnet | medium | 120s | Pattern matching on API data. Fast, structured. |
| Post-merge audit | Sonnet | medium | 300s | Reading diffs + comparing to acceptance criteria. |
| Free time | Strong (Sonnet, Opus) | medium | 600s | Real work. Most runs bail immediately (WIP check), so cost is low. |

**Never use a strong model for a dispatcher.** Dispatchers check APIs and spawn workers. They do not reason. A fast model costs fractions of a cent per run. An expensive model costs dollars per run. At 48 runs/day that difference matters.

## Tool allowlists

Restrict dispatcher tools explicitly. Without this, the dispatcher will reach for `exec` and start writing code — a fast cheap model doing "quick fixes" misses edge cases.

Dispatcher tool allowlist (OpenClaw `toolsAllow`):
```
exec, sessions_spawn, sessions_yield, subagents
```

Workers get full tool access. Only dispatchers need restriction.

## Cron prompt template

```
<one-line trigger for the skill>

Read ~/.openclaw/workspace/skills/<skill-name>/SKILL.md and follow it exactly.
Load project config from ~/.openclaw/workspace/memory/projects/<project>.yaml.

If nothing to report, respond with exactly NO_REPLY.
```

For dispatchers that don't use a skill file (e.g. simple cleanup jobs), inline the logic — but keep it short. If it's more than 10 lines, it belongs in a skill file.

## OpenClaw setup

In your OpenClaw chat, paste:

> Set up a cron job called "\<project\>-\<loop\>" that runs every \<interval\>. Use \<model\> with \<thinking\> thinking and a \<N\> second timeout. \<toolsAllow if dispatcher\> The prompt should be: "\<cron prompt from above\>". Deliver results to this chat \<only when something is found / always\>.

Then check the cron job was created:
```
/cron list
```

## Delivery modes

| Situation | Delivery config |
|-----------|----------------|
| Loop reports to you (triage, post-merge) | `announce` + `bestEffort: true` → delivers only when model produces output |
| Loop is silent by design (dispatcher) | `none` → never delivers, dispatcher output is noise |
| Loop always has useful output | `announce` without `bestEffort` |

The combination of `NO_REPLY` in the prompt + `bestEffort: true` in delivery is the standard pattern for "notify me only when something happened."
