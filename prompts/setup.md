# Setup Prompt

Copy this entire prompt and give it to your AI agent (or paste it into ChatGPT, Claude, etc.). It walks you through setup in phases, with quality gates between each phase to ensure nothing is skipped.

---

```
I want to set up an autonomous code quality system based on the methodology
described at https://github.com/Rodin-AI/how-i-work

This is a PHASED setup. Each phase has a quality gate — a check that must
pass before moving to the next phase. Do NOT skip phases or rush past gates.
If a gate fails, work with me to fix it before proceeding.

Walk me through one phase at a time. Show me the gate check result at the end
of each phase. Only proceed when I confirm the gate passes.

---

## Phase 1: Discovery

Ask me these questions one at a time:

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

### 🚧 GATE 1: Discovery complete

Before proceeding, confirm:
- [ ] At least two model providers available (for twin review independence)
- [ ] At least one cheap model + one expensive model identified
- [ ] Agent runtime can: run on a schedule, execute shell commands, call APIs
- [ ] Repo access confirmed (can clone, can push branches, can open PRs)
- [ ] Issue tracker access confirmed (can create issues, can label)

If any check fails, help me resolve it before continuing.
Show me the gate status. Proceed only when all boxes are checked.

---

## Phase 2: Documentation Foundation

Read the existing documentation in the repo (if any). Then run the sufficiency
check — answer EVERY question below using ONLY what's documented:

### Sufficiency check questions:

**Domain understanding:**
1. What are the 10 most important concepts and how do they relate?
2. What's the difference between [two easily confused concepts]?
3. What happens when [common operation] fails halfway through?
4. Which parts own which responsibilities?
5. What invariants must ALWAYS hold?

**Conventions:**
6. Show the correct way to add a new [most common unit of work].
7. How does error handling work? What gets rescued vs propagated vs logged?
8. What's the testing strategy? Unit vs integration vs e2e boundaries?
9. Where does validation live? Input boundary? Domain layer? Both?
10. What naming conventions exist?

**Architecture:**
11. Why is the system structured this way and not [obvious alternative]?
12. What are the trust boundaries?
13. What's the integration pattern for cross-subsystem features?
14. What's shared and what's private?

**Process:**
15. What does "done" mean for a PR?
16. What are the non-negotiable quality checks before merge?
17. What's in scope for the agent vs what requires human decision?

For each question, respond:
- ANSWERED: [answer + source file]
- PARTIAL: [what you can answer + what's missing]
- UNDOCUMENTED: [best guess + why you're not confident]

### 🚧 GATE 2: Documentation sufficient for loops

Count the results:
- Conventions (questions 6-10): ALL must be ANSWERED or PARTIAL
- Domain (questions 1-5): At least 3 must be ANSWERED or PARTIAL
- Process (questions 15-17): ALL must be ANSWERED

**If gate FAILS (most first-time setups):**

DO NOT SKIP THIS. Create issues in the tracker for every gap:

For UNDOCUMENTED conventions/process answers → P1 (blocks everything):
- Issue title: "[docs] Document <topic>"
- Label: documentation, bootstrap, priority:critical
- Body: what question can't be answered, suggested approach

For UNDOCUMENTED domain answers → P2 (blocks quality):
- Issue title: "[docs] Define <concept/relationship>"
- Label: documentation, bootstrap, priority:high

For PARTIAL answers → P3 (improve later):
- Issue title: "[docs] Expand <topic> — <what's missing>"
- Label: documentation, bootstrap

Group related gaps (all glossary terms → one issue, all convention gaps → one issue).

Then: WORK THE P1 ISSUES FIRST. For each one:
1. Have a conversation with the human about the answer
2. Write it down in the appropriate file (CONVENTIONS.md, docs/domain/glossary.md, etc.)
3. Commit to the repo

After P1 issues are resolved, re-run the sufficiency check.
Repeat until gate passes. Only then proceed to Phase 3.

---

## Phase 3: Configuration

Generate the system configuration:

1. **Config file** — repos, models, permissions, schedules
2. **Cron prompts** — one for each loop:
   - Triage (cheap model, every 30 min)
   - Dev loop (expensive model, triggered by triage)
   - Self-review (expensive model, clean context, different model than dev)
   - Twin review (two different providers, specialized prompts)
   - Post-merge audit (every 4 hours)
   - Lookback (every 3 days)
3. **Documentation templates** — feature design + validation checklist in docs/
4. **Review prompts** — specialized per-reviewer (structural vs semantic focus)

### 🚧 GATE 3: Configuration valid

Verify:
- [ ] Dev model and self-review model are different (or at minimum: clean context)
- [ ] Twin reviewers use different providers
- [ ] Each twin reviewer has explicit "Your Domain" AND "NOT Your Domain" sections
- [ ] Triage uses cheap model (not the expensive one)
- [ ] can_merge is false
- [ ] Post-merge audit files issues back to tracker (not just logs)
- [ ] Lookback recommends only (never auto-applies changes)
- [ ] All templates committed to docs/ in the repo

Show gate status. Proceed only when all checks pass.

---

## Phase 4: Smoke Test

Before enabling the full system, run ONE cycle manually:

1. Pick one open issue (or create a test issue)
2. Run the dev loop prompt against it — does it produce reasonable code?
3. Run self-review on the result — does it find anything the dev session missed?
4. Run both twin reviewers — do they produce non-overlapping findings?
5. Check: did the triage prompt correctly identify the issue as "ready for work"?

### 🚧 GATE 4: Smoke test passes

- [ ] Dev loop produced code that builds/compiles/passes existing tests
- [ ] Self-review found at least one thing (or explicitly said "no findings" with reasoning)
- [ ] Twin reviewers produced NON-OVERLAPPING findings (verify the "NOT Your Domain" partition works)
- [ ] Triage correctly assessed work state (didn't miss stuck items, didn't flag non-stuck items)
- [ ] No reviewer used the wrong token/account (verify bot identities)

If twin reviewers overlapped significantly → fix the prompts before proceeding.
If dev loop produced broken code → check conventions doc completeness.
If self-review found nothing AND twins found things → self-review prompt needs work.

---

## Phase 5: Go Live

Enable the cron jobs in this order (not all at once):

**Week 1:** Triage + Dev loop only
- Verify triage doesn't false-positive (flag things that aren't stuck)
- Verify dev loop produces mergeable PRs
- Human reviews and merges everything

**Week 2:** Add Self-review + Twin review
- Verify self-review catches things before twins see them
- Verify twins don't produce noise (findings that get ignored)
- Track: are twin findings actionable? If >50% get dismissed, fix prompts.

**Week 3:** Add Post-merge audit
- Verify audits don't re-file issues that were intentionally deferred
- Verify gap issues are actionable (not vague "this could be better")

**Week 4:** Add Lookback
- Verify lookback produces useful self-improvement suggestions
- Human reviews and approves any prompt changes

### 🚧 GATE 5: System healthy

After 2 weeks of full operation:
- [ ] Triage false-positive rate < 10% (most ticks correctly find nothing)
- [ ] Twin review findings acted on > 50% (not noise)
- [ ] Self-review catches things before twins > 30% of the time
- [ ] Post-merge audit gap issues are legitimate (not re-filed known deferrals)
- [ ] No PR merged with active REQUEST_CHANGES from reviewers
- [ ] Human time is spent on decisions and review, NOT on chasing status

If any metric fails, diagnose and fix before considering the system "operational."

---

## Reference docs:

Read these to understand the system before generating config:
- Philosophy and loops: https://github.com/Rodin-AI/how-i-work/blob/main/README.md
- Why documentation matters: https://github.com/Rodin-AI/how-i-work/blob/main/the-secret-sauce.md
- Building reference docs: https://github.com/Rodin-AI/how-i-work/blob/main/building-reference-docs.md
- Step-by-step setup: https://github.com/Rodin-AI/how-i-work/blob/main/adoption-guide.md
- What we learned: https://github.com/Rodin-AI/how-i-work/blob/main/what-we-learned.md
- Multi-repo: https://github.com/Rodin-AI/how-i-work/blob/main/scaling-multiple-repos.md

## Rules (non-negotiable):

- Never skip a gate. If it fails, fix it.
- Start with can_merge: false. Always.
- Twin reviewers MUST have different providers AND different mandates.
- Self-review MUST be a clean session (no context from dev work).
- Documentation issues are P1 work — they come before feature work.
- Lookback recommends changes. A human approves them. No exceptions.
- Post-merge audit files to the tracker, not just logs.
- WIP ≤ 1 per repo.
```
