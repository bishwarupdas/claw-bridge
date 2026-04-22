---
name: claw-bridge-vvp
description: Visual Verification Protocol — runs concrete checks against work with visual output using chrome-devtools-mcp tools. Use this skill before declaring any UI or frontend task complete. Provides 6 checks covering content visibility, readability, state transitions, viewport bounds, content fit, and cross-tab regression. If any check fails, invoke claw-bridge-repair immediately.
---

# Visual Verification Protocol (VVP)

Six checks. ALL must pass before a task with visual output is complete. Use chrome-devtools-mcp to run them — do not hand-wave.

## Check 1: Content visibility

Claim "the page renders correctly" requires evidence. Use chrome-devtools-mcp:
1. `navigate_page` to the target URL
2. `take_screenshot` — the actual rendered page
3. `take_snapshot` — the DOM tree

Then verify:
- Every element named in the spec appears in the snapshot (grep the snapshot for expected text/IDs)
- The screenshot shows content, not empty scaffolding

Report: screenshot path + specific elements found in snapshot.

## Check 2: Human readability

Using the DOM snapshot + computed styles:
- Font sizes: no text below 11px unless explicitly decorative
- Contrast: text against background meets WCAG AA (4.5:1 for body, 3:1 for large text)
- Touch targets: ≥48px on mobile, ≥32px on desktop

Use `evaluate_script` to query `window.getComputedStyle` for key elements.

## Check 3: State switching

Test at least 3 states:
- Default / happy path
- One transition (click, form submit, navigation)
- One edge case (empty data, error state, loading state)

Use `click`, `fill`, or `evaluate_script` to trigger transitions. Screenshot each state.

## Check 4: Viewport bounds

Use `resize_page` to check at 4 widths: 360, 768, 1024, 1440.
At each width: `take_snapshot`. Every named element must have bounding rects inside the viewport (no horizontal scroll unless intentional).

## Check 5: Content fits

For each primary scrollable container:

```javascript
const el = document.querySelector('<selector>');
return { scrollHeight: el.scrollHeight, clientHeight: el.clientHeight };
```

Run via `evaluate_script`. scrollHeight ≤ clientHeight, OR vertical scroll is explicitly intentional design.

## Check 6: Cross-tab / cross-route regression

If the project has multiple routes or tabs: navigate through each, snapshot each, confirm no console errors (`list_console_messages`), and that previously-working content still renders.

## Reporting format

After running VVP, report in this format:

```
VVP Report for <commit-sha | "working tree">
  Check 1 (content visibility): PASS — screenshot at /tmp/vvp-1.png, elements found: [...]
  Check 2 (readability):        PASS
  Check 3 (state switching):    PASS — screenshots at /tmp/vvp-3-{default,transition,edge}.png
  Check 4 (viewport bounds):    FAIL — submit button out of viewport at 768px (x=820, viewport=768)
  Check 5 (content fits):       PASS
  Check 6 (regression):         PASS — 3 routes verified, 0 console errors

Verdict: FAIL (Check 4)
```

If a check fails: STOP. Invoke claw-bridge-repair. Do not proceed to the next check or the next task.

Do not pass checks you did not run. Do not report PASS without evidence.
If a check cannot be run (no dev server, no Chrome), report it as BLOCKED with the reason.
