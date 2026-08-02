# Global AGENTS.md

## Writing style

Apply these rules to all prose you write, including documentation, commit messages, pull request descriptions, reports, and replies.

Use ASD-STE100 Simplified Technical English for technical text:

- Use approved words only. Each word has one meaning.
- Use one term for each idea. Do not use different terms for the same thing.
- Write short sentences. Keep instructions to 20 words or fewer.
- Use active voice. Write "Turn the switch", not "The switch must be turned".
- Write short paragraphs. Keep one topic in each paragraph.

## Tool Usage

Do not run Playwright, chrome-devtools, browser automation, or similar browser-control/testing features in Codex unless the user explicitly asks for them in the prompt.

## General Guidelines

- Never use the em dash "—". Use plain dash "-" instead.
- When writing commit messages, NEVER auto-add your agent name as co-author.
- Never manually modify CHANGELOG.md files or any files that are marked as auto-generated.
- When writing or substantially editing long Markdown files, put each full sentence on its own line.
  Preserve normal Markdown structure, but avoid wrapping multiple sentences onto one physical line.
- When making technical decisions, do not give much weight to development cost.
  Instead, prefer quality, simplicity, robustness, scalability, and long term maintainability.
- When doing bug fixes, always start with reproducing the bug in an E2E setting as closely aligned with how an end user experiences it as possible.
  This makes sure you find the real problem so your fix will actually solve it.
- When end-to-end testing a product, be picky about the UI you see and be obsessed with pixel perfection.
  If something clearly looks off, even if it is not directly related to what you are doing, try to get it fixed alongside the main task.
- Apply that same high standard to engineering excellence: lint, test failures, and test flakiness.
  If you see one, even if it is not caused by what you are working on right now, still get it fixed.
