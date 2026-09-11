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
bin/task-worktree create     # spin a task: worktree + executor agent
bin/task-worktree handoff    # swap in a fresh-context executor (same worktree/branch)
bin/task-worktree done       # (executors) "I am finished" — wakes Dispatch
bin/task-worktree report     # persist an executor's report; auto-close if investigative
bin/task-worktree list       # task status from the ledger
bin/task-worktree reconcile  # repair ledger status from live herdr state
bin/task-worktree classify   # why is an executor idle? (stopped by you / done / crashed)
bin/task-worktree remove     # close a task and clean up (keeps the branch)
bin/task-worktree prune      # reconcile dangling symlinks
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

## How a task reports back

The executor reports **itself**. Its final action, after committing and pushing,
is:

```
bin/task-worktree done --note "<summary>" [--pr <url>] [--body-file report.md]
```

which appends a durable `settled` event to the ledger, notifies you on the
desktop, and wakes Dispatch. `create` writes a `task.env` into the task workspace
so the executor knows its own id and which pane to wake — no guessing.

A detached `bin/task-worktree watch <id>` runs as a **backstop** for the case
where an executor dies or forgets. Both paths take the same per-task *settle
claim* before recording anything, so **Dispatch is woken exactly once per
settle** — and the claim is released when the executor resumes work, so a task
that settles, gets steered, and settles again reports every time.

### "Did it finish, or did *you* stop it?"

herdr can only see that an agent went **idle** — and stopping an executor
yourself (pi's `/stop`, Esc) looks exactly like it finishing. That used to wake
Dispatch with a half-done task and a compare link.

So before the backstop wakes anyone it **classifies** the settle by reading pi's
own session log, where the reason is recorded structurally:

| pi `stopReason` / `errorMessage` | cause | wake? |
|---|---|---|
| `stop` | `completed` | ✅ |
| `aborted` / `Operation aborted` | `human-interrupt` | 🚫 suppressed |
| `error` / `This operation was aborted` | `human-interrupt` | 🚫 suppressed |
| `aborted` / `Aborted after N retry attempt` | `agent-error` | ✅ flagged |
| `error` / `Connection error.`, `terminated`, … | `agent-error` | ✅ flagged |
| anything else / no session log | `unknown` | ✅ (bias toward waking) |

A suppressed settle is **not invisible**: it lands in the ledger as
`paused-by-human`, so `list` and `reconcile` show it on your next turn. The
watcher stays armed, so the next genuine settle reports normally, and the
executor's own `done` is never classified or suppressed. `TASK_SETTLE_CLASSIFY=0`
turns the whole thing off (always wake).

If a watcher ever breaks it fails **loud**: it logs to
`.dispatch/logs/watch-<id>.log`, records a `watcher-failed` settle and wakes
Dispatch, rather than spinning silently. An executor that *dies* (crash, or its
workspace closed) is reported the same way, as an `orphaned` settle — silence is
never an acceptable answer. `bin/task-worktree reconcile` repairs
any task whose ledger status drifted from live herdr state.

`bin/task-worktree-selftest` exercises all of this against a scripted fake herdr
(exactly-once, re-arming, the boot-transient guard, the settle classifier and
the fail-loud paths). It runs its sections concurrently — each has its own
sandbox, its own stub state and its own watcher — and finishes in ~10s:

```
bin/task-worktree-selftest             # all 89 checks (default -j6)
bin/task-worktree-selftest -j1         # serial, for debugging; same transcript
bin/task-worktree-selftest --falsify   # prove the timing-sensitive checks still bite
```

The live-watcher cases wait on **conditions** ("until a settle is recorded",
"until the watcher exits") rather than sleeping for a worst case. The few
assertions that are inherently negative ("*no* settle during the boot
transient") cannot be condition-waited, so they dwell for one tunable constant
— and `--falsify` exists to prove that constant is still long enough: it
disables the watcher's boot-transient defences and **requires** the suite to
catch it. If a shortened dwell ever made those checks vacuous, `--falsify`
fails loudly instead of the suite quietly passing for the wrong reason.

## What's tracked here

Only the harness tooling — `AGENTS.md` (Dispatch's contract),
`.dispatch/task-executor.AGENTS.md` (the executor's contract), and
`bin/task-worktree`. All the actual task state (worktrees, reports, the ledger,
repo symlinks) is scratch and git-ignored.
