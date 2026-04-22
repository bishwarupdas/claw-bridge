---
name: claw-review
description: Review a commit diff with discipline — check for scope creep, architectural violations, and missing verification evidence.
argument-hint: <commit-sha>
---

Activate the claw-bridge-failure-modes skill.

Target commit: $ARGUMENTS

1. Run `git show $ARGUMENTS --stat` and `git show $ARGUMENTS` to see the diff
2. For each file changed:
   - Does it relate to the commit message? (FM3 scope creep check)
   - Does it duplicate existing code elsewhere? (FM2 check — grep for similar patterns)
3. If the commit references "verified" or "tested" in its message: where is the evidence? (FM4 check)
4. Report findings.

Output format:
- Commit: <sha> — <message>
- Files changed: <n>
- Scope check: [PASS / FAIL with specifics]
- Duplication check: [PASS / FAIL with paths]
- Verification evidence: [PROVIDED / MISSING]
- Recommendation: [MERGE / REQUEST CHANGES]
