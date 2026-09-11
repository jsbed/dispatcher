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
bin/task-worktree handoff  # swap in a fresh-context executor (same worktree/branch)
bin/task-worktree report   # persist an executor's report; auto-close if investigative
bin/task-worktree list     # task status from the ledger
bin/task-worktree remove   # close a task and clean up (keeps the branch)
bin/task-worktree prune    # reconcile dangling symlinks
```

Two kinds of task:
- **`--mode code`** — changes files; stays open for review, closed later.
- **`--mode investigate`** — read-only "find me X"; reports and cleans itself up.

### Handing off to a fresh context

When an executor has mostly finished but its context is large or spent, hand the
work to a brand-new, empty-context agent on the **same worktree/branch** — the
baton pass:

```
bin/task-worktree handoff <task-id> [--prompt "..."] [--note-from <path>] [--agent <kind>]
```

It closes the old executor workspace (discarding its context), starts a fresh
agent rooted at the same task dir, and re-briefs it — keeping the branch, the
worktree, any uncommitted changes, and the PR. Pass `--note-from` to hand the
new agent a handoff note (the old agent's knowledge lives in the tree/notes, not
in the discarded context). The task id, the existing re-arming watcher, and the
ledger lineage (`created → handoff`) all carry across.

Every task starts from the freshly-fetched latest `main` tip unless you say
otherwise.

## What's tracked here

Only the harness tooling — `AGENTS.md` (Dispatch's contract),
`.dispatch/task-executor.AGENTS.md` (the executor's contract), and
`bin/task-worktree`. All the actual task state (worktrees, reports, the ledger,
repo symlinks) is scratch and git-ignored.
