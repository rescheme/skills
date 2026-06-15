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

Determine the review target:

- **No arguments**: review all local uncommitted changes. Run `git diff` (unstaged), `git diff --cached` (staged), and `git status --short` (untracked).
- **Commit hash**: `git show <sha>`
- **Branch name**: `git diff <branch>...HEAD`
- **PR URL or number**: `gh pr view <pr> --json` for metadata, then `gh pr diff <pr>` or `git diff base...head` for the diff.

For PR reviews, confirm: target PR number, base branch, head branch, latest head SHA, and author. Fetch the latest refs before reviewing, especially for re-reviews.

## Step 2: Read the Full Files

Diffs alone are not enough. Identify changed files from the diff, then read the **full contents** of each modified or untracked file to understand surrounding logic, contracts, error handling, and project patterns.

Check for repository conventions before judging:
- `AGENTS.md`, `CONVENTIONS.md`, `.editorconfig`
- Formatter/linter configs, local style guides
- Relevant locally installed skills (e.g., Next.js, React, TypeScript, security)

Use the active skill list when provided, or inspect `~/.agents/skills`, `.pi/agent/skills`, or run `npx skills list --json` / `npx skills list -g --json`. Apply matching skills as review guidance for their domain. Do not discover or install new skills during a review.

## Step 3: Review

Prioritize issues that affect users, data, security, operations, or future maintenance.

Review only the **changes and the behavior they affect**. Do not flag unrelated pre-existing code unless the change makes that existing risk newly reachable or worse.

### Correctness
Bugs, logical errors, off-by-one mistakes, incorrect conditionals, missing guards, unreachable branches, regressions, race conditions, stale state, broken UX flows. Does the code achieve its stated purpose?

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

## Step 4: Form Findings

For each issue, include:

- **Severity**: `Critical`, `High`, `Medium`, `Low`, or `Suggestion`
- **Location**: file, function/component, and line if available
- **Problem**: what is wrong
- **Why it matters**: realistic scenario, environment, or inputs needed for the issue to occur
- **Recommended fix**: concise implementation guidance for the developer or model who will make the change

Be certain before calling something a bug. Investigate first, explain the realistic scenario, and avoid invented hypotheticals. If uncertain after reasonable investigation, describe the uncertainty instead of presenting it as a definite finding.

When a locally installed skill materially informed a finding, cite it. Do not cite skills for generic issues.

## Step 5: Submit the Review

### For PR Reviews

When GitHub review tooling is available, submit actionable findings as a formal PR review with inline comments. Use tight file and line references. Do not post duplicate comments for the same issue.

- Compare the authenticated GitHub user with the PR author. If they are the same, GitHub rejects self-approval. Submit a formal `COMMENT` review with the body `Approved - Safe to merge` instead of `APPROVE`.
- Verify the submitted feedback appears in the PR's formal review list, not only as general PR comments. If no review object is returned, report that as a tooling caveat locally.
- Keep the local final response concise. Do not duplicate detailed findings that were posted to the PR. Report only the review outcome, whether a formal PR review was submitted, checks run or skipped, and any tooling/permission caveat.
- If a formal PR review could not be submitted, provide the findings locally in review-ready form.

### For Non-PR Reviews

Present findings directly in the local response. Keep the tone matter-of-fact, direct, and helpful. Avoid flattery, accusatory phrasing, and filler.

## Step 6: Merge Recommendation

End every review with one of:

- **`Approved - Safe to merge`**: no blocking or meaningful unresolved issues; note any external CI caveat.
- **`Request Changes - Safe after minor changes`**: only low/medium issues that are small and low risk to fix.
- **`Request Changes - Block until fixed`**: critical/high issue, security/data exposure, broken core behavior, or tests/build failing for PR-caused reasons.

## Investigation Commands

Use these as needed:
- `git diff`, `git diff --cached`, `git status --short` for local changes
- `git show <sha>` for a commit
- `git diff <branch>...HEAD` for branch comparison
- `gh pr view <number> --json` for PR metadata
- `gh pr diff <number>` for PR diff
- `git fetch` to update refs
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
