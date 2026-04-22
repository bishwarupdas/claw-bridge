---
name: claw-bridge-discipline
description: Enforces a 4-step discipline (UNDERSTAND, DECOMPOSE, SOLVE, VERIFY) on any non-trivial coding task. Use this skill whenever the user requests code changes that involve more than a single-line edit, any new feature, any UI work, or any task with visual output. Apply proactively — do not wait for the user to ask for discipline.
---

# claw-bridge discipline

For any coding task beyond a trivial edit, you must follow this 4-step protocol. Do not skip steps. Do not combine them.

## Step 1: UNDERSTAND

Before writing code, report:
- What files/modules are involved (paths, not summaries)
- What the current behavior is (run it, read it, don't infer)
- What the spec or user request actually requires (quote it)
- What's unclear or ambiguous (list the open questions)

Do not proceed until you have checked — not assumed — each of these.

## Step 2: DECOMPOSE

Break the work into atomic steps. Each step:
- Does ONE thing
- Can be committed independently
- Leaves the repo in a working state

List the steps. Wait for user confirmation on the decomposition before writing code.

No "while I'm at it" expansions. If you find an adjacent issue, surface it, don't silently fix it.

## Step 3: SOLVE

Execute one step at a time. Commit per step with a clear message.

After each commit, invoke the vvp skill (see claw-bridge-vvp) to verify the step before moving on.

Do not run ahead multiple steps hoping it'll all work. Commit, verify, next.

## Step 4: VERIFY

Before telling the user a task is done:
- Run the vvp skill
- Provide evidence, not claims ("screenshot shows X", "test passes: paste output", NOT "looks good")
- If you used chrome-devtools-mcp tools, include the actual tool outputs

If any verification fails, STOP. Invoke the claw-bridge-repair skill and report the failure with specifics. Do not try to fix and re-claim without user approval.

## When this skill applies

This skill should be active for any:
- Multi-file change
- New feature, new component, new route
- Any UI work (any visual output)
- Any change to routing, state management, or data flow
- Refactors spanning more than one file

This skill does NOT apply to:
- Single-line typo fixes
- Pure documentation edits
- Trivial config tweaks (changing a port number etc.)

Use your judgment for the grey zone — when in doubt, follow the discipline.
