---
name: claw-bridge-failure-modes
description: Catalog of known Claude Code failure modes with detection patterns. Use this skill to cross-check your own work before declaring a task complete. Covers screenshot hallucination, architectural violations, scope creep, fake verification, skipped teardown, and regression ignorance.
---

# Known failure modes (check your own work against these)

Before claiming a task is complete, ask yourself: am I committing any of these?

## FM1 — Screenshot hallucination

**Pattern:** Claiming "the page renders X" when the screenshot shows empty scaffolding.
**Detection:** Use `take_snapshot` and grep for the specific text/elements claimed. If the text isn't in the DOM, the claim is false regardless of what the screenshot "looks like" at a glance.

## FM2 — Architectural violation

**Pattern:** Building a new module that duplicates an existing one because you didn't read the existing code.
**Detection:** Before writing new code in any directory, grep/list that directory and adjacent ones for existing implementations. Report what's there before writing.

## FM3 — Scope creep

**Pattern:** "While I'm at it" — touching unrelated code in the same commit.
**Detection:** `git diff` the commit. Every changed file should relate to the stated task. If not, split the commit or revert the unrelated change.

## FM4 — Fake verification

**Pattern:** Reporting "VVP 6/6 passed" without actually running the 6 checks.
**Detection:** For each check in the VVP skill, the report must include an artifact path (screenshot, snapshot dump) or a tool output. If it's just a claim, it didn't happen.

## FM5 — Skipped teardown

**Pattern:** Feature works on first load, leaks on unmount. Missing cleanup, missing unsubscribe, missing cancelAnimationFrame.
**Detection:** Enter → exit → re-enter the feature. Use `list_console_messages` after cycles. Leaks typically show as growing listener counts or RAF warnings.

## FM6 — Regression ignorance

**Pattern:** New feature works in isolation. Adjacent features quietly broke.
**Detection:** Before declaring done, navigate through the 3 most-touched routes/tabs of the app. Snapshot each. Compare to pre-change.

## Self-audit before marking complete

For every non-trivial task, answer in writing:
- Did I just claim FM1? (If I say "renders correctly" — did I snapshot-grep the claimed content?)
- Did I just claim FM4? (If I say "verified" — did I run the VVP checks or am I pattern-matching?)
- Did I touch files unrelated to the task? (FM3)
- Did I test an exit/cleanup path? (FM5)

If you catch yourself about to commit one of these, STOP, correct, then proceed.
