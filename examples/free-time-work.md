# Free Time

## What this is really doing

Left to their own devices, agents (and humans) always pick the interesting work. Features over bug fixes. Experiments over infrastructure. New code over maintenance. This is natural — creative work is more rewarding than janitorial work.

But the janitorial work is what keeps a project healthy. Security updates get applied. Flaky tests get fixed. Documentation stays current. Dependencies stay fresh. The boring stuff is boring precisely because it's unglamorous maintenance — and that's exactly why it needs to be forced.

Strict category rotation solves this. The agent can't choose — it just takes the next category in sequence. Some runs produce exciting new features. Others apply a security patch and exit. Both are equally valuable to the project's long-term health.

The WIP check is the other critical piece. If there's already an open PR, free time does nothing. This prevents the failure mode where the agent starts five things in parallel and finishes none of them. Finish what's in flight, *then* start something new.

## Before you start

Read [cron-setup.md](cron-setup.md) and [project-config.md](project-config.md) first. Free time doesn't use a skill file — the logic is short enough to keep inline — but it does read the project config for the `repo`, `token_path`, and `wip_limit` fields.

## The state file

Free-time needs persistent state to rotate categories. Without it, it picks the same category every run.

```json
// memory/free-time-last.json
{
  "last_category": "B",
  "last_run": "2026-05-14T03:00:00Z",
  "last_task": "Updated README with missing setup step"
}
```

The prompt reads this file, picks the next category in sequence, writes the choice back, then does the work.

## Customize your categories

The default rotation is a starting point. Replace it with what your project actually needs:

**Software project:**
```
A = Bugs (filed issues, test failures, error reports)
B = Tests (coverage gaps, flaky tests, missing edge cases)
C = Documentation (stale docs, missing examples, unclear explanations)
D = Dependencies (outdated packages, security advisories)
E = Refactoring (code smells, duplication, complexity)
```

**Infrastructure / ops project:**
```
A = Security (CVEs, certificate expiry, access audit)
B = Reliability (alerting gaps, runbook gaps, single points of failure)
C = Observability (missing metrics, incomplete dashboards)
D = Cost (idle resources, oversized instances, unused storage)
E = Documentation (runbooks, architecture docs, onboarding)
```

Pick categories that map to real neglected work in your project. If you never need to update dependencies, remove that category. If security is critical, make it two slots in the rotation.

## Set it up

**Step 1: Create the state file**

```json
// memory/free-time-last.json
{"last_category": "E", "last_run": null, "last_task": null}
```

Starting at E means the first run picks A.

**Step 2: Write your category definitions** — replace the placeholders in the cron prompt below with real descriptions specific to your project. Vague categories produce vague work.

**Step 3: Create the cron job** — see [cron-setup.md](cron-setup.md) for the full guide. For this loop:

> Set up a cron job called "myproject-free-time" on the schedule "0,20,40 * * * *" (every 20 minutes). Use your strongest model with medium thinking and a 600 second timeout.
>
> The prompt should be:
>
> "Free time for myproject. Read memory/free-time-last.json for the last category. Pick the NEXT one in rotation: A → B → C → D → E → A. Write your choice to memory/free-time-last.json immediately (before doing any work).
>
> Categories:
> A = <your category A>
> B = <your category B>
> C = <your category C>
> D = <your category D>
> E = <your category E>
>
> FIRST: Check open PRs in <repo> using token at <token_path>. If any open PRs exist from me, respond with NO_REPLY immediately — do not start new work.
>
> Do ONE task from the selected category. Create a PR if code changes are needed. Log one line to memory/YYYY-MM-DD.md: what category, what you did."
>
> Deliver results to this chat.

**Why these constraints matter:**

- **Write state BEFORE doing work** — If the job times out or crashes mid-task, you still want the category rotation to have advanced. Writing state first prevents the same category from being stuck forever when runs fail.
- **Strong model with medium thinking** — Most runs bail immediately because of the WIP check, costing almost nothing. The few runs that actually do work need to implement features, fix subtle bugs, or apply security patches. These require real capability. The economics work: cheap exits (most of the time) fund capable execution (when it matters).
- **600s timeout** — Free-time tasks are real work. Give it enough time.
- **ONE task** — The constraint is important. "Do some cleanup" turns into a 3-hour refactor. "Do one task" means: find the highest-priority item in the category, do that, stop.
- **Log to daily file** — Over time, this creates an audit trail of what free time actually produced. Useful for the lookback loop.

## The failure modes this prevents

**Parallel work explosion** — Without the WIP check, free-time kicks off a new task every 20 minutes regardless of what's in flight. After a few hours you have 9 open PRs. The WIP check prevents this.

**Category starvation** — Without forced rotation, the agent would pick the most interesting category every time. Documentation never gets touched. Security updates never happen. Rotation ensures every category gets attention.

**Scope creep** — Without the "ONE task" constraint, free-time runs turn into open-ended improvement sessions. The single-task constraint keeps each run bounded and predictable.

**State corruption after crash** — Without writing state first, a crash during execution leaves state pointing to the previous category. Next run picks the same category again. Writing state before work prevents this.
