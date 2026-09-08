# Task executor context

You were spawned by **Dispatch** to carry out one task. Your working directory is
an isolated `tasks/<id>/` dir containing symlinks to your git worktree(s) and this
file. Stay inside your lane.

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
5. **Finish cleanly.** When done, make sure your changes are committed and pushed,
   then stop (go idle) with a short summary of what you did and how to verify it.
   Dispatch watches your state and will flip you to `awaiting-review`.
6. **Report blockers early.** If you're blocked (missing info, failing setup,
   ambiguous requirements, needing a decision), stop and state the blocker clearly
   rather than guessing or expanding scope.

## Scope discipline

Do exactly the task you were briefed on. If you discover adjacent work that seems
worth doing, **note it in your summary** for Dispatch to triage — do not silently
expand scope. If the task turns out to need a different or additional repo, say so;
Dispatch will spin the right worktree.
