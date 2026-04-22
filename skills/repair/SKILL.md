---
name: claw-bridge-repair
description: Governs the fix-verify loop after a VVP failure. Use this skill when a verification check has failed and work is needed to fix it. Enforces: isolate the specific failure, hypothesize minimal cause, fix narrowly, re-verify the specific check, then re-run full VVP for regressions, before continuing.
---

# Repair loop — what to do when verification fails

## Phase 1: ISOLATE

- Name the specific check that failed (Check 4 of VVP, etc.)
- Name the specific element/selector/viewport/state
- Quote the actual tool output — don't paraphrase

## Phase 2: HYPOTHESIZE (before writing any fix)

- What's the minimal plausible cause?
- What files likely contain it? (grep to confirm, don't assume)
- State the hypothesis. The user may correct it before you code.

## Phase 3: FIX

- Minimal change addressing only the identified cause
- Tidy First principle: do not mix refactoring with the fix
- If you notice an adjacent issue, surface it, don't silently fix it (FM3 scope creep)

## Phase 4: RE-VERIFY specific

- Re-run ONLY the check that failed, first
- If still failing: back to ISOLATE with the new output (hypothesis was wrong)
- If passing: proceed to Phase 5

## Phase 5: REGRESS

- Re-run the FULL VVP
- If new checks fail, you've caused regression: back to ISOLATE for the new failure
- If all pass, move to Phase 6

## Phase 6: CONTINUE

- Report the fix with evidence (before/after screenshots, tool outputs)
- Do not stack multiple fixes before verification — one fix, one verify cycle
- Only then advance to the next task

---

Named methodology this is based on: PDCA (Plan-Do-Check-Act, Deming) +
Tidy First (Kent Beck, 2023 — separate structural from behavioral).
