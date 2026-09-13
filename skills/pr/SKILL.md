---
name: pr
description: "Review committed changes and publish a maintainer-ready draft PR in the current branch's writable remote repository. Use when the user invokes \"$pr\" or asks to draft, create, or open a PR."
---

## Preflight

1. Treat the current branch's Git remote as the sole publication target. Never infer a repository from GitHub CLI context, follow a parent/upstream repository, create or synchronize a fork, or publish anywhere else. Require authenticated write access; otherwise stop and ask the user to run from a branch whose remote is their writable fork or repository.
2. Run the bundled `bash scripts/pr-preflight` from the target worktree before review and again before publication. Never review or publish while blocked.
3. Inspect its prospective commits and complete diff for task scope.
4. If the branch is default or spent, rebuild from the reported base with only intended commits, then rerun preflight.
5. If base or head changes, repeat the affected review.

## Review

1. Use the preflight base and head; verify worktree HEAD before each review.
2. Review the complete `base...head` diff. Report actionable findings and stop.
   Run repository checks with CI-equivalent options; expected visual changes need scoped assertions, not blanket bypasses.
3. For every UI change, reuse supplied media or capture matched, reproducible before/after media from base and head. Capture `Before` from the exact base commit the PR will merge into and `After` from the reviewed head. On follow-ups, reuse or recapture `Before` from that base; never use an earlier feature-branch revision or a prior `After` as the new `Before`:
   - Use video for interaction, motion, or multiple steps; images otherwise.
     Keep recordings focused and paced for a reviewer, and inspect the exported media before publication.
   - Match viewport, state, data, and action sequence; verify each capture runs the intended revision's build.
   - Frame screenshots with enough surrounding UI to make the changed element's location and purpose clear, including a page or section landmark and relevant adjacent controls. Prefer viewport- or section-level framing; use tight element crops only when context is genuinely unnecessary.
   - Keep media untracked through `.git/info/exclude`.
   - Never publish with either side missing.

## Package

- Use the repository PR template exactly when present.
- Before writing the title and description, inspect recently merged PRs and base-branch Git history for the repository's vocabulary, scope, and level of detail.
- Treat prospective commit subjects as inputs for change scope, not as PR title candidates.
- Title: `<type>(<scope>): <imperative summary>` using the narrowest Conventional Commits type. Omit an unhelpful scope; reserve `style` for formatting-only changes.
- Write a concise, human-readable title around the complete user-visible or system-visible outcome that makes the change valuable. Make it stand alone for a reviewer who has not read the description.
- Add issue-closing syntax when the change resolves an issue.
- Do not invent template sections or include generic command output.
- Without a repository template, use `## Purpose`, `## Solution`, and `## Verification` for substantive changes. Purpose states the problem or user need. Solution explains the approach and relevant trade-offs; add implementation bullets only when they help reviewers. Verification records the evidence that the change works. For a trivial change, one short paragraph may replace these sections.
- Put both UI `Before:` and `After:` media under the best template heading; never publish a comparison with either side missing. Without a template, add `## Comparison` only when visual evidence benefits from its own section.

## Publish

Immediately before publication, verify worktree HEAD equals the reviewed head. If not, restart preflight.

For UI changes, run `bash scripts/pr-media-upload OWNER/REPO FILE`; if it fails, use the GitHub editor. Use each returned reference exactly as emitted.

For an existing draft, snapshot with `bash scripts/pr-publish --snapshot > /tmp/pr-body.old` and preserve its content. Publish with `bash scripts/pr-publish REVIEWED_BASE REVIEWED_HEAD TITLE [/tmp/pr-body.old] < /tmp/pr-body.md`; it blocks stale reviews, unsafe body replacement, invalid titles or media, and mismatched draft metadata or commits.

After pushing, wait for required checks on that head and resolve failures before reporting completion.

Only create or update drafts unless the user explicitly requests a merge. For an explicit merge request, mark the reviewed PR ready if necessary and merge it; otherwise never mark ready or merge. Never request reviewers, enable auto-merge, or post GitHub review activity.
