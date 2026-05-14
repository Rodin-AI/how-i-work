# Project Config File

Every loop reads from a shared project config file. This separates *what a loop operates on* (the config) from *how it works* (the skill file). When you add a second project, you add a config file and a new cron job — you don't touch the skill.

## Location

```
~/.openclaw/workspace/memory/projects/<project>.yaml
```

One file per project. All loops for that project read the same file.

## Full field reference

```yaml
# Required by all loops
repo: myorg/myproject           # owner/repo on your VCS host
token_path: ~/.mycreds/gitea-bot  # path to bot auth token

# Required if using Gitea (omit for GitHub)
gitea_url: https://gitea.example.com

# Required by dev-loop and post-merge-review
exec_node: my-dev-node          # node where tests run

# Optional — defaults shown
branch_default: main
wip_limit: 1                    # max open PRs from bot before blocking new work

# Optional — triage thresholds
stale_review_hours: 24          # flag reviews older than this
stale_ci_hours: 2               # flag CI pending/failed longer than this

# Optional — post-merge audit
audit_lookback_hours: 4         # how far back to look for merged PRs
audit_state_file: memory/projects/<project>-audit-state.json
issue_label_gap: post-merge-gap # label applied to gap issues

# Optional — label IDs (required if your VCS uses numeric label IDs)
labels:
  ready: 42
  wip: 40
```

## Which loops use which fields

| Field | dev-loop | triage | post-merge | free-time |
|-------|----------|--------|------------|-----------|
| `repo` | ✓ | ✓ | ✓ | ✓ |
| `token_path` | ✓ | ✓ | ✓ | ✓ |
| `gitea_url` | ✓ | ✓ | ✓ | — |
| `exec_node` | ✓ | — | ✓ | — |
| `branch_default` | ✓ | — | — | — |
| `wip_limit` | ✓ | ✓ | — | ✓ |
| `stale_review_hours` | — | ✓ | — | — |
| `stale_ci_hours` | — | ✓ | — | — |
| `audit_lookback_hours` | — | — | ✓ | — |
| `audit_state_file` | — | — | ✓ | — |
| `issue_label_gap` | — | — | ✓ | — |
| `labels` | ✓ | ✓ | — | — |

## Minimal example (GitHub, dev-loop + triage only)

```yaml
repo: myorg/myproject
token_path: ~/.mycreds/github-bot
exec_node: my-dev-node
```

## Adding a second project

Copy the file, change the values. The skill files are unchanged.

```
memory/projects/myproject.yaml
memory/projects/myproject2.yaml
```
