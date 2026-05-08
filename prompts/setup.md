# Setup Prompt

Copy this entire prompt and give it to your AI agent (or paste it into ChatGPT, Claude, etc.). It will walk you through setting up the system for your repos.

---

```
I want to set up an autonomous code quality system based on the methodology
described at https://github.com/Rodin-AI/how-i-work

Walk me through the setup by asking me questions one at a time. After you have
enough information, generate my complete configuration: cron prompts, review
templates, and documentation scaffolding.

## Questions to ask me:

### Basics
1. How many repos do you want to manage? (1, or multiple?)
2. For each repo: what's the URL and default branch?
3. What language(s) are used in each repo?
4. What git host? (GitHub, GitHub Enterprise, GitLab, Gitea, etc.)
5. What issue tracker? (GitHub Issues, Jira, Linear, etc.)

### Models and access
6. What AI model providers do you have access to? (OpenAI, Anthropic, Google, etc.)
7. Which specific models? (e.g., GPT-5, Claude Opus, Claude Sonnet)
8. Do you have separate bot accounts for reviews, or just your main account?

### Runtime
9. What agent runtime are you using? (OpenClaw, Hermes, GitHub Actions, custom cron, other)
10. Where does it run? (local machine, VPS, cloud instance)
11. Does the agent have shell access (can it run git, build tools, linters)?

### Preferences
12. Can the agent open PRs autonomously, or should it draft them for you?
13. Should the agent ever merge without your approval? (recommend: no)
14. What's your timezone? (for scheduling)
15. How do you want to be notified? (Slack, Telegram, email, just PR assignments)

### Documentation state
16. Do you have existing documentation in your repo? (domain docs, conventions, etc.)
17. If not — do you want me to help scaffold it first?

## After gathering answers, generate:

1. **Config file** — repos, models, permissions, schedules
2. **Cron prompts** — one for each loop (triage, dev, self-review, twin review, post-merge audit, lookback)
3. **Documentation templates** — feature design template + validation checklist, placed in docs/
4. **Conventions scaffold** — a starter CONVENTIONS.md based on the language(s)
5. **Gap-finding prompt** — the documentation sufficiency check, pre-filled with their domain
6. **Bootstrap issues** — for every gap found, create an issue in the tracker (see below)

## After generating config: bootstrap the documentation backlog

Once setup is complete, IMMEDIATELY run the gap-finding prompt against the repo.
For every question answered UNDOCUMENTED or PARTIAL, create an issue in the tracker:

```
Issue title: "[docs] <what's missing>"
Issue body:
- What question can't be answered: <the question>
- What the agent found (or couldn't find): <the UNDOCUMENTED/PARTIAL response>
- Suggested approach: <conversation topic, file to create, or existing knowledge to write down>
- Label: documentation, bootstrap
```

Group related gaps into single issues where it makes sense:
- All missing glossary terms → one "Create domain glossary" issue
- All missing convention answers → one "Document conventions" issue  
- Each missing architecture decision → its own issue (these need individual conversations)

Prioritize issues by what BLOCKS the loops from working:
1. **Conventions** (P1) — without these, every PR drifts differently
2. **Domain glossary** (P1) — without this, the agent uses inconsistent terminology
3. **One reference pattern** (P2) — without this, the agent has no blueprint
4. **Architecture decisions** (P3) — triage gate will flag these as they come up
5. **Implementation docs** (P4) — nice to have, not blocking

The result: the system's first work items are "build the fuel that makes me work."
Triage picks up the P1 issues. The agent proposes drafts. The human reviews and
refines. Within a week, the loops have enough documentation to operate on real
feature work — because the system bootstrapped its own foundation.

## Reference docs:

Read these to understand the system before generating config:
- Philosophy and loops: https://github.com/Rodin-AI/how-i-work/blob/main/README.md
- Why documentation matters: https://github.com/Rodin-AI/how-i-work/blob/main/the-secret-sauce.md
- Step-by-step setup: https://github.com/Rodin-AI/how-i-work/blob/main/adoption-guide.md
- Multi-repo architecture: https://github.com/Rodin-AI/how-i-work/blob/main/scaling-multiple-repos.md

## Rules for generated config:

- Start with can_merge: false (always)
- Use cheap models for dispatch (triage), expensive for work (dev, self-review)
- Twin review must use two different providers
- Self-review must spawn a clean session (context isolation) with a different model
- WIP ≤ 1 per repo
- Post-merge audit files issues back to the tracker, not just logs
- Lookback recommends changes but never auto-applies them
- All documentation templates go in docs/ within the repo
```
