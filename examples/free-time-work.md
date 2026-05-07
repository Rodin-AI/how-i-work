# Free Time

## What this is really doing

Left to their own devices, agents (and humans) always pick the interesting work. Features over bug fixes. Experiments over infrastructure. New code over maintenance. This is natural — creative work is more rewarding than janitorial work.

But the janitorial work is what keeps a project healthy. Security updates get applied. Flaky tests get fixed. Documentation stays current. Dependencies stay fresh. The boring stuff is boring precisely because it's unglamorous maintenance — and that's exactly why it needs to be forced.

Strict category rotation solves this. The agent can't choose — it just takes the next category in sequence. Some runs produce exciting new features. Others apply a security patch and exit. Both are equally valuable to the project's long-term health.

The WIP check is the other critical piece. If there's already an open PR, free time does nothing. This prevents the failure mode where the agent starts five things in parallel and finishes none of them. Finish what's in flight, *then* start something new.

## Set it up

Paste this into your OpenClaw chat:

> Set up a cron job called "free-time" on the schedule "0,20,40 * * * *" (every 20 minutes). Use Opus with medium thinking and a 600 second timeout.
>
> The prompt should be:
>
> "Free time. Read memory/free-time-last.md for the last category. Pick the NEXT one in rotation: A → B → C → D → E → A. Write your choice to memory/free-time-last.md immediately.
>
> Categories: A=Bugs, B=Tooling, C=Experiments, D=Features, E=Infrastructure.
>
> Do ONE task from that category, then exit. If I already have an open PR (WIP > 0), respond with NO_REPLY. Log one line to memory/YYYY-MM-DD.md."
>
> Deliver results to this chat.

Then customize the categories:

> Categories: A=Security, B=Documentation, C=Tests, D=Refactoring, E=Dependencies

## Why Opus for idle work

Most free-time runs bail immediately because of the WIP check — costing almost nothing. The few runs that actually do work need to implement features, fix subtle bugs, or run experiments. These are creative tasks that benefit from the strongest reasoning model. The economics work out: cheap exits (most of the time) fund expensive execution (when it matters).
