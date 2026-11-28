# Project AGENTS.md

## Writing style

Apply these rules to all prose you write, including documentation, commit messages, pull request descriptions, reports, and replies.

Use ASD-STE100 Simplified Technical English for technical text:

- Use approved words only. Each word has one meaning.
- Use one term for each idea. Do not use different terms for the same thing.
- Write short sentences. Keep instructions to 20 words or fewer.
- Use active voice. Write "Turn the switch", not "The switch must be turned".
- Write short paragraphs. Keep one topic in each paragraph.

## Automated tests

Never create, add, or restore automated tests of any kind in this repo (unit, integration, e2e, snapshot).
Do not add test files, test runners, test scripts, or test CI steps.
Verify changes with typecheck, lint, build, and manual or device checks instead.

## Branch

Check the current branch before changing files:

```bash
git branch --show-current
```

- If on `main`, create an appropriate working branch:

  ```bash
  git checkout -b feat/<descriptive-name>
  ```

- Never push directly to `main`.
- Allowed working branches:
  - `feat/*`
  - `fix/*`
  - `docs/*`
  - `chore/*`

## Commit

After implementation is complete and relevant checks pass, commit the changes.
Do not wait for a separate user instruction to commit.

- Message format: `type: subject`
- Max 72 chars, lowercase, no final period
- Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `build`, `ci`, `chore`, `revert`
- Use `git commit -m "type: subject"` only
- Never add co-authors (`Co-authored-by:`)
- Use the logged-in GitHub account name and email as the git commit author name and email
- One file per commit is mandatory:
  - Each commit must include changes to one file only
  - Every changed file must be committed separately
  - Never combine multiple files into one commit
  - Do not make exceptions for related files, coupled files, or shared features unless explicitly instructed
  - When in doubt, split further, not less
  - Commit messages must describe the single-file change only
- Do not amend unless explicitly asked

## Push and PR

After commits are ready and relevant checks pass, push the working branch.
Open a pull request that is ready for review.
Do not wait for a separate user instruction to push or to open a PR.

- Never create a pull request as a draft.
- Never convert an open pull request to a draft.
- Always open pull requests as ready for review.
- Do not pass `--draft` to `gh pr create`.
  Draft pull requests do not request CODEOWNERS. The review watch does not fire on `ready_for_review`.

## GitHub review

- Publish the pull request review immediately without requesting approval.
- If the pull request author is the authenticated GitHub user, publish the review as `COMMENT`, not `APPROVE`.

```bash
git status
git diff --stat
git log origin/main..HEAD --oneline
git push -u origin HEAD
```

- Push allowed working branches only
- Open a PR for every branch
- Merge with a merge commit only. No squash, no rebase merge
- When creating PRs or issues with `gh`, write the body to a temporary file first, then pass the file path with `--body-file`.
  Heredoc stdin redirection can fail in non-interactive shell sessions.
  Always delete the temporary file after the PR is created.

PR title format:

```text
<type>: <summary>
```

PR title rules:

- Imperative present tense
- No trailing period
- No ticket IDs
- Add `(no-changelog)` only when changelog exclusion is intended

PR body must include:

```markdown
Fixes #<issue-number>

## Summary

Brief description of what this PR does and why.

## Changes

- Specific change 1
- Specific change 2
```

- If the PR is not tied to an issue, omit `Fixes #...`
- If the PR closes multiple issues, put each closing keyword on its own line in the main body
- Do not create a `Checklist` section in the PR body

## Issues

- Issue titles should not use type prefixes (`perf:`, `chore:`, `test:`) or bracket prefixes (`[bug]`, `[feat]`, `[docs]`).
  Labels handle categorization, so titles should be clean, imperative, and scannable.

## Project-specific technical guidance

### Repository structure

- `skills/`: reusable Agent Skills (`issue-review`, `pr-review`, `review-feedback`), each with a `SKILL.md` file.
- `plugins/`: reusable Devin plugins (such as `plugins/paper-remote`) with `.devin-plugin/plugin.json` and `.mcp.json`.

### Validation

- Parse all changed `.json` manifests, MCP declarations, and `.devin/config.json` files as JSON before you commit.
- When writing or editing long Markdown files, put each full sentence on its own line.
