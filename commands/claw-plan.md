---
name: claw-plan
description: Generate a 4-step (UNDERSTAND/DECOMPOSE/SOLVE/VERIFY) kickoff plan for a coding task, then wait for user approval before executing.
argument-hint: <task description, or path to spec file>
---

Activate the claw-bridge-discipline skill.

The user wants to work on: $ARGUMENTS

Step through the discipline:

1. **UNDERSTAND**: List the files you'd need to read and read them. If a spec file was referenced, read it. Report what you found — current state, spec requirements, open questions.

2. **DECOMPOSE**: Propose 3-7 atomic steps to accomplish the task. Each step must:
   - Do one thing
   - Be independently committable
   - Leave the repo working

3. **Wait for user confirmation** before proceeding to SOLVE.

Do not start coding. This command's job is the plan, not the execution.
