---
name: issue-review
description: |
  Analyze and review GitHub issues for bugs or feature requests. Use when asked to
  review, analyze, triage, or debug a GitHub issue. Triggers on issue URLs, issue
  numbers (#123), or explicit requests to analyze GitHub issues.
---

# Issue Review

Analyze GitHub issues deeply. Treat the issue author's analysis as a hypothesis —
consider it, then independently verify by reading the code and tracing execution paths.

## Trigger Conditions

Use this skill when:
- User provides a GitHub issue URL
- User mentions an issue number (`#123`) or `owner/repo#123`
- User asks to "analyze", "review", "triage", or "debug" a GitHub issue

## Step 1: Resolve the Issue

Accept these formats:
- Full URL: `https://github.com/owner/repo/issues/123`
- Scoped: `owner/repo#123`
- Number only: `#123` (infer from current repo via `git remote` or `gh repo view`)

If ambiguous, ask the user for the full URL before proceeding.

## Step 2: Fetch the Issue

Use the GitHub CLI to retrieve the full issue:
```bash
gh issue view <url> --json number,title,body,author,labels,comments,state,url
```

If `gh` is unavailable or unauthenticated, inform the user and stop.

## Step 3: Read Everything

Read the issue body, **all** comments, and any linked issues/PRs mentioned in the thread.
Do not stop at the first comment.

## Step 4: Classify

Determine if the issue is a **bug report** or a **feature request** based on the content,
not the labels.

### For Bugs

1. **Consider the reporter's analysis** — note their stated symptoms, reproduction steps,
   and any root cause they propose. Treat this as a working hypothesis.
2. **Identify related code** — find all files mentioned or implied in the issue.
3. **Read every related file in full** — do not truncate.
4. **Trace the actual code execution path** — reproduce the behavior mentally or by
   reviewing the call chain.
5. **Form your own root cause** — independently derive the actual cause from the code.
6. **Correlate** — compare your findings with the reporter's hypothesis. If they match,
   note it. If they differ, explain why the reporter's analysis is incomplete or incorrect.
7. **Propose a fix** — based on your verified root cause, not the issue text.

### For Feature Requests

1. **Consider the reporter's proposal** — note their desired behavior and any implementation
   ideas they suggest. Treat this as a working hypothesis.
2. **Read all related code files** — understand the current architecture, existing patterns,
   and where the feature would fit.
3. **Evaluate feasibility** — assess whether the proposal aligns with the codebase and
   whether a simpler approach exists.
4. **Propose the most concise implementation** — optimize for minimal change, consistency
   with existing patterns, and maintainability.
5. **List affected files and specific changes**.
6. **Correlate** — compare your proposal with the reporter's. If they match, note it.
   If your approach differs, explain why and what the trade-offs are.

## Step 5: Propose Only

**Do NOT implement the fix or feature unless the user explicitly asks.**

Present a structured analysis:

1. **Issue Summary** — concise restatement of the problem or request
2. **Classification** — bug / feature request
3. **Reporter's Hypothesis** — summary of what the issue author claimed or proposed
4. **Independent Verification** — your findings from reading the code
5. **Correlation** — where the reporter's analysis aligns or diverges from your findings
6. **Proposed Fix / Implementation** — based on your verified analysis
7. **Affected Files** — specific files and changes needed

## Optional: Workflow Integration

If the user wants to mark the issue as in-progress:
```bash
gh issue edit <url> --add-label inprogress --add-assignee @me
```
If this fails, report the error and continue analysis.
