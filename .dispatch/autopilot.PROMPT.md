# Autopilot: PR #{{PR_NUMBER}} ({{REPO}})

Drive PR #{{PR_NUMBER}} — {{PR_URL}} — to a mergeable, green state on branch
`{{BRANCH}}`. You are babysitting an existing PR, not writing a new feature.

If `~/.cursor/skills-cursor/autopilot/SKILL.md` exists, read it first and follow
it; the loop below is the contract either way.

## The loop

Each pass, **re-read live PR state** (never trust an earlier pass):

```
gh pr view {{PR_NUMBER}} --json state,mergeable,mergeStateStatus,isDraft,reviewDecision,comments,reviews
gh pr checks {{PR_NUMBER}}
```

Then fix, in this order of priority:

1. **Merge conflicts** — rebase/merge as the PR requires and resolve them.
2. **Review comments** — address unresolved reviewer feedback; reply or note in
   your report when you deliberately don't.
3. **Failing CI** — fix the cause, not the symptom.

Commit and push after each meaningful fix. Repeat until conflicts are gone,
feedback is addressed, and checks are green — or until you are genuinely stuck.

While waiting on CI, **watch** the checks rather than tight-polling:
`gh pr checks {{PR_NUMBER}} --watch` (one blocking call), not a `sleep` loop.

## Hard limits

- **Never** merge, enable auto-merge, mark the PR ready for review, close it, or
  force-push someone else's branch.
- Stay on `{{BRANCH}}`. Don't touch other branches or open new PRs.
- Don't expand scope beyond making this PR mergeable and green; note adjacent
  work in your report instead.
- Blocked (needs a human decision, a secret, or a broken external service)? Stop
  and say so with `task-worktree done --blocked`.

## Finish

Push, then report with `task-worktree done --note "..." --pr {{PR_URL}}` and your
full markdown report via `--body-file`.
