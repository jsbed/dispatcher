# Dispatch

A tiny harness for running coding work as **background agent tasks**.

You talk to one orchestrator agent — **Dispatch** — and say *"do X in repo Y."*
Dispatch doesn't do the work itself. It spins up an isolated git worktree, boots
an **executor** agent in it, and tracks the task while you carry on. When the
executor finishes it reports back, and Dispatch surfaces the result to you.

## The idea

- **Dispatch** = orchestrator. Listens to you, spawns and tracks tasks. Never
  blocks — tasks run in the background and wake it when they settle.
- **Executor** = worker. One task, one branch, one isolated worktree. Commits and
  pushes its own branch; never touches anything outside its lane.
- **Ledger** = an append-only log of *why* each task exists and its status history.

## One command runs everything

`bin/task-worktree` is the blessed path — never call `git worktree` by hand:

```
bin/task-worktree create   # spin a task: worktree + executor agent
bin/task-worktree report   # persist an executor's report; auto-close if investigative
bin/task-worktree list     # task status from the ledger
bin/task-worktree remove   # close a task and clean up (keeps the branch)
bin/task-worktree prune    # reconcile dangling symlinks
```

Two kinds of task:
- **`--mode code`** — changes files; stays open for review, closed later.
- **`--mode investigate`** — read-only "find me X"; reports and cleans itself up.

Every task starts from the freshly-fetched latest `main` tip unless you say
otherwise.

## What's tracked here

Only the harness tooling — `AGENTS.md` (Dispatch's contract),
`.dispatch/task-executor.AGENTS.md` (the executor's contract), and
`bin/task-worktree`. All the actual task state (worktrees, reports, the ledger,
repo symlinks) is scratch and git-ignored.
