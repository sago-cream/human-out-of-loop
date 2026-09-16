---
name: solve-issue-review
description: "Turn exact or picked GitHub issues into independently reviewed draft PRs using isolated developers and independent reviewers. Use for issue-to-PR runs, picked batches, or continuation; batches default to parallel."
---

## Select

- `$solve-issue-review`: select one eligible issue.
- `$solve-issue-review <positive-issue-number>`: select that issue only; never substitute another.
- `$solve-issue-review pick [positive-amount]`: select distinct issues in ascending order; default to `1`, or use all remaining when fewer qualify.
- Dispatch batches immediately in parallel unless the user requests sequential execution; then finish each issue before dispatching the next.

Eligible: open, with no open or draft PR covering it through a GitHub Development link or closing keyword. Ignore incidental mentions.

Ineligible exact-target output:

```text
ISSUE_SKIPPED issue=<number> reason=<not_found|not_open>
ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>
```

## Invariants

- Let `$pr` publish the draft PR in the current branch's writable remote repository; never select or pass a publication mode.
- Give each issue one sidebar-visible `DEV` in an isolated worktree and one independent, source-read-only `REV`, created only after `DEV` produces a committed head.
- Use no neutral seed, transient subagent, or `REV` forked from `DEV`.
- Keep review feedback inside Codex tasks; never post GitHub review activity.
- Treat active workers as black boxes. ORCH routes lifecycle state; it never repeats or independently interprets worker diagnosis, design, implementation, or validation commentary.
- ORCH reports only selection/dispatch, an accepted ready head, review/fix transitions, publication, skip, or blocker. Worker tasks own detailed progress narration.
- Call `set_thread_title` once per task and never change it:
  - Orchestrator: `Solve #<issue>` or `Solve <amount> issues`
  - Developer: `#<issue>: <short-description>`
  - Reviewer: `Review #<issue>`
- Key each run record by repo + issue. Store DEV/REV task IDs and cursors, absolute worktree, accepted head, PR URL, and lifecycle state.

## Run

1. Query issues and open/draft PRs. Select without replacement; recheck before each sequential dispatch or once per parallel batch.
2. Create branch `<type>/<short-kebab-description>` using the narrowest of `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`. Never namespace it.
3. Create `DEV` with repo, issue + URL, branch, base, absolute worktree, and optional media path. Require `DEV` to:
   - Trust ORCH's coverage check. Recheck only after delay, interruption, or resumption; if covered, stop unchanged with `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`.
   - For UI changes, exclude `.codex-pr-media/` through `.git/info/exclude` and capture matched, reproducible before/after media: video for interaction, motion, or multiple steps; images otherwise.
   - Implement, commit, and check. Before readiness, restore only agent-generated artifacts outside the commit and require a clean worktree except ignored media.
   - Return `{ state: "ready_for_review", branch, headSha, checks, media, cleanWorktree: true }`; never publish.
   - Leave the branch unchanged after readiness. Send replacement results only to REV.
4. Verify DEV's first ready head equals worktree `HEAD` and the worktree is clean except ignored media. If either check fails, send DEV one narrow correction and wait for a replacement result. Otherwise store the result and create `REV` with issue, DEV task ID, repo, worktree, base, head, checks, and media.
5. REV owns the loop:
   - Verify each head, run `$pr`, and send ORCH lifecycle milestones only; never relay intermediate technical commentary.
   - On findings, send the technical details only to DEV, report `fix` to ORCH, and wait for DEV's replacement head.
   - Repeat until publication; report only the published head and PR URL to ORCH.
6. After the first ready head, accept head and lifecycle updates only from REV. Matching publication is terminal. For later user-requested changes, set `fix`, send the request to DEV, and route DEV's replacement head to the same REV.

While a worker is active, preserve its `wait_threads` cursor and wait again. Commentary updates, tool activity, and timeouts are not reasons to inspect the worker with `read_thread`, summarize its progress, or send a status probe. Contact a worker only for its explicit request or blocker, a malformed completed result, a failed readiness invariant, or a user-requested scope change.

## Continue

On “continue,” “keep going,” “finish it,” unfinished status, or interrupted waits, read [references/resume.md](references/resume.md), reconcile existing state, and resume without duplicating tasks or artifacts.
