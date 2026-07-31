---
name: review-feedback
description: |
  Assess code review feedback before acting on it. Use when asked to inspect the latest
  PR feedback, validate reviewer comments, triage requested changes, or determine whether
  feedback is correct, outdated, already addressed, unclear, or inapplicable.
---

# Review Feedback

Treat every review comment as a hypothesis to verify against the current codebase.
Do not accept or reject feedback based on tone, authority, or confidence alone.

## When to Use

Use this skill when the user asks to:
- Assess the latest feedback on a pull request
- Validate code review comments or requested changes
- Determine which review items should be implemented
- Reassess feedback after new commits
- Prepare a technical response to a reviewer

## Step 1: Preserve the Current Git State

Feedback assessment is a read-only workflow unless the user explicitly asks for implementation.
Do not create a branch, switch branches, create a worktree, commit, stash, or alter the working tree solely to assess feedback.

Inspect the current branch and working tree with `git branch --show-current` and `git status --short`.
Respect the current checkout and any uncommitted changes.
If another branch or commit is required, ask before switching and never discard or overwrite local work.

## Step 2: Resolve the Target and Latest Feedback

Accept a PR URL, PR number, current branch with an associated PR, or feedback pasted by the user.
If the target is ambiguous, ask for the PR or feedback before proceeding.

For GitHub pull requests:

1. Retrieve the PR metadata, latest head SHA, reviews, and conversation comments with `gh pr view <pr> --json number,title,url,state,baseRefName,headRefName,headRefOid,reviews,comments`.
2. Retrieve inline review comments and their timestamps with `gh api repos/{owner}/{repo}/pulls/{number}/comments --paginate`.
3. Inspect complete review threads, including author replies, resolution state, and whether comments refer to outdated code.
4. Fetch refs when needed, but do not check them out automatically.

"Latest feedback" includes the newest review and any earlier unresolved feedback that remains applicable.
Do not silently omit unresolved items because a newer comment exists.
Record the head SHA used as the assessment baseline.

If `gh` is unavailable or unauthenticated, inform the user and ask them to provide the feedback directly.

## Step 3: Understand Each Feedback Item

Read every feedback item and its full thread before evaluating it.
Split reviews containing multiple independent requests into separate items.
For each item, identify:

- Reviewer, timestamp, and review state
- File, line, commit, and relevant diff
- The specific technical claim or requested behavior
- Any assumptions, constraints, or questions in the thread
- Whether later commits or replies supersede the feedback

If a request is unclear, mark it as unclear instead of guessing what the reviewer intended.

## Step 4: Verify Independently

Read the full affected files at the selected head SHA, not only the diff or comment snippet.
Trace relevant call sites, types, tests, configuration, error paths, and platform or version constraints.
Check repository instructions and established patterns before judging style or architecture.
Use current primary documentation when validity depends on an external API, framework, or tool.
Run the smallest relevant test, type check, lint command, or reproduction when it materially increases confidence.

Evaluate each item independently and assign one verdict:

- **Valid**: the reported problem exists and the requested direction is technically appropriate.
- **Partially valid**: the concern is real, but the proposed fix, scope, or reasoning is incomplete.
- **Invalid**: the claim does not hold for the current code and supported scenarios.
- **Already addressed**: the current head already resolves the concern.
- **Outdated or superseded**: the comment applies only to older code or was replaced by later feedback.
- **Unclear**: the intended behavior or acceptance criteria cannot be determined safely.
- **Not verifiable**: required evidence, environment, or access is unavailable.

Do not call feedback invalid merely because the suggested implementation is poor.
Separate the underlying concern from the reviewer's proposed solution.
Support every verdict with concrete code, test, documentation, or repository evidence.

## Step 5: Recommend an Action

For each item, recommend one of:

- Implement as requested
- Implement with a corrected or narrower approach
- Ask a specific clarifying question
- Respond with evidence-based technical pushback
- Take no action because it is addressed, outdated, or inapplicable

Prefer the smallest change that resolves a verified concern without introducing unrelated behavior.
If feedback conflicts with repository requirements or an explicit user decision, surface the conflict instead of choosing silently.

## Step 6: Assess Only Unless Asked

Do not implement changes, reply on GitHub, resolve threads, or submit a review unless the user explicitly asks.

Present the assessment in this structure:

1. **Target**: PR, branch, and head SHA assessed
2. **Summary**: counts by verdict and the highest-priority valid concern
3. **Feedback Items**: source, location, claim, verdict, evidence, and recommended action for each item
4. **Verification Performed**: files, tests, commands, and documentation checked
5. **Open Questions or Limitations**: anything unclear or not verifiable

Order valid blocking concerns first, followed by partially valid, unclear, and non-actionable items.
Keep technical disagreement direct, specific, and evidence-based.
