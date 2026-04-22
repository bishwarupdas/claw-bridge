---
name: claw-verify
description: Run the Visual Verification Protocol against the current working tree or a specified commit. Uses chrome-devtools-mcp for actual browser-based checks.
argument-hint: [commit-sha | HEAD (default)]
---

Activate the claw-bridge-vvp skill.

Target: ${ARGUMENTS:-HEAD}

If a commit SHA was provided, checkout that commit (use `git stash` first if needed, restore after).

Then:
1. Identify the URLs/routes to test (ask user or infer from README/package.json)
2. Run all 6 VVP checks using chrome-devtools-mcp tools
3. Report in the VVP reporting format defined in the skill
4. Return a PASS or FAIL verdict

If a check fails, immediately invoke claw-bridge-repair before proceeding.
Do not skip checks. If a check can't be run (e.g., no dev server running), report it as BLOCKED with the reason.
