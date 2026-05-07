# Post-Merge Review

## What this is really doing

There's a gap between "passed review" and "actually complete." Every developer knows this gap exists — it's the acceptance criterion you half-addressed, the edge case mentioned in a comment that you meant to come back to, the documentation you planned to update "after merge."

Most processes treat merge as the end of the quality story. This job treats it as a checkpoint where the *real* question gets asked: **"Did the code actually deliver what the issue asked for?"**

This is the quality ratchet. Without it, small gaps accumulate into debt that nobody tracks because nobody ever formally noticed it. With it, every gap becomes a tracked issue that flows back into triage — and eventually gets fixed.

The subtle power: this job makes it psychologically safe to merge imperfect PRs. If something slips through, the system catches it and creates a follow-up. Perfection isn't required at merge time — completeness is eventually guaranteed by the cycle.

## Set it up

Paste this into your OpenClaw chat:

> Set up a cron job called "post-merge-review" that runs every 4 hours. Use Sonnet with medium thinking. It should check recently merged PRs, find each one's linked issue, and verify all acceptance criteria were addressed. If anything is incomplete, file a new bug issue. If nothing was missed, respond with NO_REPLY. Deliver results to this chat.

Then add your project specifics:

> ...check merged PRs on github.com/myorg/myproject from the last 4 hours. Issues are linked via "Closes #123" in PR bodies. File new issues with the label "post-merge-gap".

## Why every 4 hours

Merges happen a few times a day at most. Running this more frequently burns compute checking empty state. Every 4 hours is frequent enough to catch gaps the same day they're introduced — before the context fades from everyone's memory.
