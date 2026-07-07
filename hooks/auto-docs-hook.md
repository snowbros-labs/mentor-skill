# Optional: auto-docs Stop hook

Opt-in automation for the incremental-documentation workflow (`references/orchestration.md` §6). **Not active by default** — a skill cannot self-trigger, so true "update docs after every task" needs a harness hook that *you* install.

## Honest caveats (read before enabling)

- A `Stop` hook fires after **every** task turn, in **every** project that uses the config it lives in. It is blunt by nature — expect it on trivial tasks too.
- A hook runs a **shell command**; it cannot itself "run the mentor workflow." The realistic pattern is to inject a **reminder** back into context so the model runs the workflow on the next turn. That is a nudge, not a guarantee.
- Global `~/.claude/settings.json` affects all work. Prefer **project-scoped** `.claude/settings.json` in the repo you actually want documented.
- This trades tokens and noise for currency of docs. Most users are better served running the workflow deliberately on Medium+ tasks (the default).

## Project-scoped install (recommended)

Add to the target project's `.claude/settings.json` (not the global one):

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'REMINDER: if this task materially changed the codebase, run the mentor incremental-documentation workflow (orchestration.md §6) — patch only what changed in docs/, append to changelog/decisions.'"
          }
        ]
      }
    ]
  }
}
```

The hook's stdout is surfaced as context, reminding the model to run §6 before ending. Remove the block to disable.

## Making it less noisy

Gate the reminder on real changes so it stays quiet on trivial turns — e.g. a small script that emits the reminder only when `git status --porcelain` shows source changes, and stays silent otherwise. Point the `command` at that script instead of the raw `echo`.

## Why this isn't wired in for you

Per the system's honest-limits stance (`orchestration.md` §9): global auto-triggers are a config change with side effects across every project, so they are left as a deliberate opt-in rather than shipped active.
