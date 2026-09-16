---
name: solve-issue
description: "Solve GitHub issues directly through implementation and $pr publication. Use `$solve-issue #<issue>` for one exact issue in the current task, or `$solve-issue <amount>` to dispatch that many independent issue-solving tasks in parallel."
---

## Select

- `$solve-issue #<positive-issue-number>`: solve that issue in the current task; never create a worker or substitute another issue.
- `$solve-issue <positive-amount>`: select that many distinct eligible issues in ascending order and immediately create one sidebar-visible task per issue in parallel. If fewer qualify, dispatch all remaining eligible issues and report the shortage.
- `$solve-issue`: solve one eligible issue in the current task.

Eligible: open, with no open or draft PR covering it through a GitHub Development link or closing keyword. Ignore incidental mentions. Recheck once before dispatch.

Exact-target failure:

```text
ISSUE_SKIPPED issue=<number> reason=<not_found|not_open>
ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>
```

## Dispatch

- Keep exact and default runs in the invoking task. For amount runs, use only the selected issue tasks; do not create separate developer, reviewer, or publication tasks.
- Give every issue an isolated worktree and branch named `<type>/<short-kebab-description>`, using the narrowest of `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`. Never namespace it.
- Call `set_thread_title` once per task and never change it:
  - Exact/default task: `Solve #<issue>`
  - Amount orchestrator: `Solve <amount> issues`
  - Amount worker: `#<issue>: <short-description>`
- Give each amount worker the repository, issue + URL, base, branch, absolute worktree, and optional media path. The orchestrator only dispatches, waits, reports terminal results, and resumes existing workers without duplicating them.

## Solve

Each issue-owning task completes the whole workflow:

1. Implement the issue, run relevant checks, and commit the complete solution.
2. For UI changes, keep `.codex-pr-media/` untracked through `.git/info/exclude` and capture matched, reproducible before/after media: video for interaction, motion, or multiple steps; images otherwise.
3. Require a clean worktree except ignored PR media, then run `$pr`.
4. If `$pr` finds actionable problems, fix and commit them, rerun checks, and invoke `$pr` again. Repeat in the same task until it publishes a draft PR.
5. Report the issue, published head, PR URL, and checks. Never merge or mark a PR ready.

On continuation, find and resume the existing issue-owning task, branch, worktree, and draft. Never create replacements for work that still exists.
