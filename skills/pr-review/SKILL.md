---
name: pr-review
description: |
  Review code changes as a senior engineer. Use when asked to review a pull request,
  branch comparison, commit, or uncommitted changes. Assesses correctness, security,
  maintainability, and test coverage. Produces severity-ranked findings with merge readiness.
---

# PR Review

## When to Use

Use this skill when the user asks to:
- Review a pull request (URL or number)
- Review a branch, commit, or uncommitted changes
- Re-review a PR after fixes
- Assess whether a PR is safe to merge

## Step 1: Resolve the Target

Determine the review target without guessing:

- **Local changes**: when the user asks for a local review without mentioning a PR, review all uncommitted changes with `git diff`, `git diff --cached`, and `git status --short`.
- **Commit hash**: use `git show <sha>`.
- **Branch name**: use `git diff <branch>...HEAD`.
- **PR URL or number**: use the specified pull request.
- **PR review without a URL or number**: list open pull requests with `gh pr list --state open --json number,title,headRefName,author,isDraft,updatedAt,url`, then ask the user which one to review.

For PR reviews, confirm the PR number, base branch and SHA, head branch and latest SHA, author, draft state, and URL.
Fetch the latest refs before reviewing, especially for re-reviews, but do not switch branches or alter the working tree solely for review.

## Step 2: Gather Complete PR Context

Retrieve structured metadata and the changeset separately so truncation in one does not discard the other:

```bash
gh pr view <pr> --json number,title,body,author,url,state,isDraft,baseRefName,baseRefOid,headRefName,headRefOid,additions,deletions,changedFiles,commits,files,mergeable,mergeStateStatus,reviewDecision,statusCheckRollup
gh pr diff <pr> --color=never
```

Read the title, body, commits, changed-file list, CI state, and complete diff.
Compare `changedFiles` with `gh pr diff <pr> --name-only` to confirm that every changed file is accounted for.
Do not rely on a combined metadata-and-diff command when output may be truncated.
If output is truncated, rerun the metadata and diff separately, list changed paths with `gh pr diff <pr> --name-only`, and inspect each path from fetched Git refs with `git diff <base-sha>...<head-sha> -- <path>`.
The current `gh pr diff` command does not accept a pathspec after `--`.

If `gh` is unavailable or unauthenticated, inform the user and ask for the PR metadata and patch.

## Step 3: Read the Full Files

Diffs alone are not enough.
Read the **full contents at the reviewed head SHA** of every modified source, test, and configuration file to understand surrounding logic, contracts, error handling, and project patterns.
For deleted files, inspect the base version.
Do not accidentally read a different local branch when its contents differ from the reviewed head SHA.

Check for repository conventions before judging:
- `AGENTS.md`, `CONVENTIONS.md`, `.editorconfig`
- Formatter/linter configs and local style guides
- Relevant locally installed skills such as Next.js, React, TypeScript, or security skills

Inspect the harness-provided active skill list before reviewing.
Load and apply installed skills materially relevant to the changed technologies or risks.
Follow the harness's skill-loading rules.
Do not install skills or search external registries during review.

## Step 4: Review

Prioritize issues that affect users, data, security, operations, or future maintenance.

Review only the **changes and the behavior they affect**. Do not flag unrelated pre-existing code unless the change makes that existing risk newly reachable or worse.

### Intent and Scope
Determine what the PR is trying to change, whether the implementation matches that intent, and whether unrelated changes or accidental behavior shifts are included.

### Correctness
Bugs, logical errors, off-by-one mistakes, incorrect conditionals, missing guards, unreachable branches, regressions, race conditions, stale state, broken UX flows.
Does the code achieve its stated purpose?

### Error Handling
Swallowed failures, unexpected throws, uncaught error return types, partial failure paths, broken exceptional flows.

### Security & Sensitive Data
Authorization, authentication, access control, tenant/role boundaries, unsafe client input trust, injection risks, DoS paths. Exposure of sensitive data in API responses, logs, serialized props, errors, snapshots, or client state.

### Maintainability
Clarity, modularity, ownership boundaries, duplicated logic, misleading names, weak type contracts, excessive nesting. Flag style preferences only when they create real risk or violate established conventions.

### Efficiency
Only flag obvious problems: N+1 queries, O(n²) on unbounded data, unbounded queries, expensive filters, unstable pagination, blocking I/O on hot paths, unnecessary client work, avoidable memory use.

### Edge Cases & Boundaries
Validation gaps, unbounded inputs, unsafe parsing, empty states, retries, partial failure, boundary values.

### Testability
Weak, missing, or misleading tests where changed behavior crosses service, action, database, or UI boundaries. Suggest additional test cases if they would materially improve confidence.

### Behavior Changes
Call out changed behavior that appears possibly unintentional.

## Step 5: Form the Review

Start with a concise overview of what the PR changes and its user-visible or architectural impact.
Then present actionable findings ordered by severity.
If there are no findings, state that explicitly instead of inventing suggestions.

For each issue, include:

- **Severity**: `Critical`, `High`, `Medium`, `Low`, or `Suggestion`
- **Location**: file, function/component, and line if available
- **Problem**: what is wrong
- **Why it matters**: realistic scenario, environment, or inputs needed for the issue to occur
- **Recommended fix**: concise implementation guidance for the developer or model who will make the change

Be certain before calling something a bug. Investigate first, explain the realistic scenario, and avoid invented hypotheticals. If uncertain after reasonable investigation, describe the uncertainty instead of presenting it as a definite finding.

When a locally installed skill materially informed a finding, cite it.
Do not cite skills for generic issues.

After the findings, include a compact quality and readiness table with a `1-5` score and one-line rationale for:

- Correctness
- Project conventions and maintainability
- Test coverage
- Security
- Performance
- Overall merge readiness

Use `1` for fundamentally unsafe or incomplete work, `3` for work requiring material fixes, and `5` for merge-ready work with no meaningful concerns.
Keep scores consistent with the findings and merge recommendation.
An unresolved Critical or High finding caps overall readiness at `2/5`; an unresolved Medium finding or PR-caused failing check caps it at `3/5`.

End the written review with the checks run or skipped and any limitations caused by unavailable context, tooling, or environment.
Keep sections and bullets concise while preserving the evidence needed to act.

## Step 6: Submit the Review

### For PR Reviews

When GitHub review tooling is available, submit actionable findings as a formal PR review with inline comments. Use tight file and line references. Do not post duplicate comments for the same issue.

- Compare the authenticated GitHub user with the PR author. If they are the same, GitHub rejects self-approval. Submit a formal `COMMENT` review with the body `Approved - Safe to merge` instead of `APPROVE`.
- Verify the submitted feedback appears in the PR's formal review list, not only as general PR comments. If no review object is returned, report that as a tooling caveat locally.
- Keep the local final response concise. Do not duplicate detailed findings that were posted to the PR. Report only the review outcome, whether a formal PR review was submitted, checks run or skipped, and any tooling/permission caveat.
- If a formal PR review could not be submitted, provide the findings locally in review-ready form.

### For Non-PR Reviews

Present findings directly in the local response. Keep the tone matter-of-fact, direct, and helpful. Avoid flattery, accusatory phrasing, and filler.

## Step 7: Merge Recommendation

End every review with one of:

- **`Approved - Safe to merge`**: no blocking or meaningful unresolved issues; note any external CI caveat.
- **`Request Changes - Safe after minor changes`**: only low/medium issues that are small and low risk to fix.
- **`Request Changes - Block until fixed`**: critical/high issue, security/data exposure, broken core behavior, or tests/build failing for PR-caused reasons.

## Investigation Commands

Use these as needed:
- `git diff`, `git diff --cached`, `git status --short` for local changes
- `git show <sha>` for a commit
- `git diff <branch>...HEAD` for branch comparison
- `gh pr list --state open` when a PR review has no identifier
- `gh pr view <number> --json <fields>` for structured PR metadata
- `gh pr diff <number> --color=never` for the complete PR diff
- `gh pr diff <number> --name-only` to inventory changed paths
- `git diff <base-sha>...<head-sha> -- <path>` for a single file when the full diff is too large
- `git show <head-sha>:<path>` for full file contents at the reviewed revision
- `git fetch` to update refs without switching the working tree
- `rg` for call sites, related types, tests, existing patterns
- `nl -ba <file> | sed -n '<start>,<end>p'` for precise line references
- `gh pr review <number> --approve|--request-changes|--comment --body ...` for final review state
- `gh api repos/{owner}/{repo}/pulls/{number}/reviews` for formal review creation/verification
- Avoid `gh pr comment` and issue-style comments for code review findings

## Severity Guide

- **`Critical`**: exploitable security issue, data loss/corruption, production outage, or merge would immediately break core flows.
- **`High`**: sensitive data exposure, authorization bypass, serious correctness regression, or likely operational failure.
- **`Medium`**: user-visible bug, important edge case, misleading behavior, unstable pagination/state, or meaningful maintainability risk.
- **`Low`**: minor bug, localized maintainability issue, weak test gap for low-risk behavior.
- **`Suggestion`**: non-blocking improvement with clear value.
