---
name: claw-failure-check
description: Audit your own recent work against the 6 known failure modes. Run this before declaring a task complete.
---

Activate the claw-bridge-failure-modes skill.

For the current working session:
1. What task did the user request? Restate it.
2. What files did you change? (`git diff --stat` if uncommitted, or the last commit if committed)
3. For each of FM1–FM6, answer honestly: did I commit this failure mode? Provide evidence for each answer.
4. If any failure mode was committed: what are you going to do about it before declaring done?

This is a self-audit. Be honest. A clean audit only counts if you actually ran the checks.
