# Triage

## What this is really doing

Work doesn't stall dramatically. It stalls silently. A test failed at 2am and nobody noticed. A reviewer left findings six hours ago and the PR is just sitting there. A merge conflict appeared when another branch landed and now the PR can't merge even though everything else is green.

This job's purpose is to make silence impossible. Every 30 minutes, it asks: **"Is anything stuck that shouldn't be?"** Not "what should I work on" — that's the dev loop's job. Triage just shines a light on things that have gone quiet when they should be moving.

The distinction matters. A triage job that tries to *fix* things is doing too much. Its only job is to *see* — to turn invisible stalls into visible status that other loops can act on.

## Set it up

Paste this into your OpenClaw chat:

> Set up a cron job called "triage" that runs every 30 minutes. It should check my project for open PRs (CI status, review state, merge conflicts) and new issues. If anything needs attention, tell me. If everything is clean, respond with NO_REPLY. Use Sonnet. Deliver results to this chat.

Then add your project specifics:

> ...check the repo at github.com/myorg/myproject using the GitHub CLI. "Needs attention" means CI failing, reviews pending over 24 hours, or merge conflicts.

## Why Sonnet

Triage is pattern-matching, not reasoning. "Is CI green? Are reviews addressed? Are there conflicts?" — these are yes/no checks. A cheap, fast model is all you need. Save expensive reasoning for loops that actually need to think.
