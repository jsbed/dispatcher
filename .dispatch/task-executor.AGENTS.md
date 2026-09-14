# Task executor context

You were spawned by **Dispatch** to carry out one task. Your working directory is
an isolated `tasks/<id>/` dir containing symlinks to your git worktree(s), this
file, and `task.env` (your identity: task id, branch, compare URL, and the path
to the blessed `task-worktree` command). Stay inside your lane.

## You report to Dispatch yourself

Dispatch does **not** poll you. When you finish, **you tell it** — that is the
primary completion path, and it is a hard requirement of this contract:

```
"$TASK_WORKTREE_BIN" done --note "<one-line summary>" \
    [--pr <url>] [--commit <sha>] [--body-file <path-to-your-report.md>]
```

- Run it **from your task dir** (it reads `./task.env`), or pass your task id
  explicitly: `task-worktree done <task-id> --note "..."`.
- It records the durable ledger event, notifies the human, and wakes Dispatch —
  **exactly once**. A background watcher is only a backstop for the case where
  you die before calling it; the two never double-report.
- **Only `done` is trusted to mean "finished".** Going idle is not a report: if
  the human stops you mid-turn (pi `/stop`), the watcher now classifies that as
  a human interruption, records `paused-by-human` and deliberately does **not**
  wake Dispatch. Nobody will hear about your work until you call `done`.
- **Never run `herdr agent prompt` (or any other `herdr` command) by hand.**
  `task-worktree` is the only sanctioned way to reach Dispatch.
- Stopping blocked instead of finished? Same command with `--blocked`.
- Calling it again after Dispatch steers you and you do more work is correct and
  expected — report every time you finish a round of work.

## Rules

1. **Work only in your worktree(s).** Everything you need is symlinked into this
   dir (`./<repo>` points at your dedicated worktree/branch). Do your work through
   these symlinks.
2. **Stay isolated.** Do **not** touch other tasks' directories, other worktrees,
   the source repos under `~/Repositories`, or anything outside your task dir.
3. **One task, one branch.** Your branch is already checked out. Commit there.
   Do **not** merge into or rebase onto shared branches unless your task brief
   explicitly tells you to.
4. **Commit and push freely; never open PRs.** Committing to your branch and
   pushing it to the remote are the default — push when your work is ready so the
   branch is available for review. **Do not open pull requests** unless your task
   brief explicitly asks you to.
5. **Finish cleanly, and say so.** Your **final action**, in this order:
   1. commit and push your branch (and open a PR only if the brief asked for one);
   2. run `task-worktree done --note "..."` (see above) with your final report —
      include the **PR URL** via `--pr` when you opened one, and your full
      markdown report via `--body-file` so it survives teardown;
   3. then stop (go idle) with the same summary in your terminal.
   **Always include a GitHub compare link** for your branch against its base
   (`$TASK_COMPARE_URL` in `task.env`, or
   `https://github.com/<org>/<repo>/compare/<base>...<your-branch>`) in your
   summary so Dispatch can hand it straight to the human for review.
6. **Report blockers early.** If you're blocked (missing info, failing setup,
   ambiguous requirements, needing a decision), stop and say so clearly rather
   than guessing or expanding scope — and report it with
   `task-worktree done --blocked --note "<what you need>"` so Dispatch hears it
   immediately instead of waiting on a watcher.

## If you are a BRAINSTORM session (`TASK_MODE=brainstorm` in `task.env`)

Your opening brief — not this file — is your contract, and it overrides rules 4
and 5 above. In short:

- You are in a **live dialogue with a human** in your own pane. Going idle
  between turns is normal and reports nothing to anyone; nobody is waiting.
- You **do not commit, push or open PRs**, and you do not implement. Your
  worktree exists so you can read real code before asserting anything about it.
- The session ends only when the **human** ends it — they say so directly, or a
  wrap-up request arrives from Dispatch. Then you write the plan to a markdown
  file and deliver it with
  `task-worktree done --note "..." --body-file <that file>`.
- That plan is the **only** thing that survives: the task auto-closes afterwards
  and the worktree and branch are deleted.

## Scope discipline

Do exactly the task you were briefed on. If you discover adjacent work that seems
worth doing, **note it in your summary** for Dispatch to triage — do not silently
expand scope. If the task turns out to need a different or additional repo, say so;
Dispatch will spin the right worktree.
