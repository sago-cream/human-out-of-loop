# human-out-of-loop

Agent skills for turning completed work—or a fresh GitHub issue—into a reviewed, maintainer-ready draft PR.

## Install

`pr` works with skill-compatible coding agents that can run shell commands, Git, and the authenticated GitHub CLI (`gh`). UI changes also require tools for capturing screenshots or videos.

`solve-issue` and `solve-issue-review` currently require the Codex desktop app because they use its task-management tools.

Install with the [Skills CLI](https://github.com/vercel-labs/skills), then select your agent and skills:

```bash
npx skills add sago-cream/human-out-of-loop -g
```

Add `--skill pr` to install only the PR skill, or `-a codex` to target Codex. You can also ask Codex:

> Install the skills from `sago-cream/human-out-of-loop`.

### Update

Check for updates and update installed skills:

```bash
npx skills check
npx skills update
```

The Skills CLI installs directly from GitHub; no release archive or manual download is needed.

## What it catches

Before opening the PR, the agent checks the complete change for:

- Incorrect or incomplete behavior
- Unintended changes and scope creep
- Branch and commit hygiene
- Repository conventions and PR templates
- Clear, project-appropriate titles and descriptions
- Correct issue linking
- Matched before-and-after screenshots or videos for UI changes
- Correct base branch, head branch, commits, and target repository
- Broken PR formatting or missing media

If `$pr` finds an actionable problem, it stops before publication so you can fix and rerun it. The issue workflows handle that fix-and-review loop for you.

## Use it

| Goal | Prompt |
| --- | --- |
| Review committed work and open a draft PR | `$pr` |
| Solve issue #65 | `$solve-issue #65` |
| Solve five issues in parallel | `$solve-issue 5` |
| Solve and independently review issue #65 | `$solve-issue-review 65` |
| Solve and independently review five issues | `$solve-issue-review pick 5` |

`$pr` reviews the complete diff and publishes it to the current branch's writable remote. For an external contribution, make the branch track your fork; the skill never follows the parent repository.

`$solve-issue` starts from an open issue, implements it in an isolated branch and worktree, runs the relevant checks, fixes review findings, and invokes `$pr`.

`$solve-issue-review` adds a separate, independent reviewer before `$pr` publication. Batch runs process eligible issues in parallel.

The skills may create branches, worktrees, commits, and comparison media.

### Follow up on a PR

Reply in the original **Solve #…** or **Solve N issues** task—not the developer or reviewer task. It reuses the existing tasks, branch, worktree, and draft PR instead of duplicating the run.

> PR #123 has an issue with the empty state. Fix it, review the new commit, and update the draft PR.

## UI comparisons

For UI changes, `$pr` adds matched before-and-after images or videos. Install the [`gh-image`](https://github.com/drogers0/gh-image) extension once to publish GitHub-owned attachments directly:

```bash
gh extension install drogers0/gh-image
```

The skill requires `gh-image` v1.3.0 or later. It uses your existing GitHub CLI token when possible and may fall back to your local GitHub browser session. Images render from returned Markdown; videos render inline from the returned GitHub attachment URL. If `gh-image` cannot upload a file, the skill falls back to pasting it through the GitHub editor.

## Automation

Ask Codex to create an automation that runs `$solve-issue [amount]` for each Git repository in a workspace, then choose the schedule and workspace folder.
