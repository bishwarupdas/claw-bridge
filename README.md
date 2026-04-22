# claw-bridge

> Discipline enforcement + visual verification for Claude Code.

claw-bridge is a Claude Code plugin that enforces verification discipline. It makes Claude follow a 4-step protocol (UNDERSTAND → DECOMPOSE → SOLVE → VERIFY) on every non-trivial task, run concrete browser checks before declaring work done, and audit its own output against a catalog of known failure patterns. It pre-wires [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) so visual verification tools are available automatically.

---

## Install

```
/plugin marketplace add bishwarupdas/claw-bridge
/plugin install claw-bridge@claw-bridge
/reload-plugins
```

## Prerequisites

- **Node.js 20.19+** — required by chrome-devtools-mcp (run via `npx` at session start; nothing to install manually)
- **Recent Chrome** — chrome-devtools-mcp connects to a running Chrome instance
- **Claude Code** — current release with plugin support

That's it. No Python, no setup scripts, no auth tokens.

---

## The five slash commands

| Command | What it does |
|---------|-------------|
| `/claw-plan <task>` | Runs UNDERSTAND + DECOMPOSE for a task and waits for your approval before executing anything |
| `/claw-verify [commit]` | Runs all 6 VVP checks via chrome-devtools-mcp; defaults to current working tree |
| `/claw-review <commit>` | Audits a commit diff for scope creep (FM3), duplication (FM2), and missing verification evidence (FM4) |
| `/claw-status` | Shows plugin version, active skills, and whether chrome-devtools-mcp is reachable |
| `/claw-failure-check` | Self-audit: Claude answers honestly whether it committed any of the 6 known failure modes |

---

## The four skills

**`claw-bridge-discipline`** — Activates on any multi-file change, new feature, UI work, or non-trivial task. Forces Claude through UNDERSTAND (what do the files actually say?) → DECOMPOSE (atomic steps, confirmed before writing) → SOLVE (one commit at a time) → VERIFY (evidence required, not claims). Does not apply to typo fixes or pure doc edits.

**`claw-bridge-vvp`** — Visual Verification Protocol. Six concrete checks run via chrome-devtools-mcp: content visibility (screenshot + DOM snapshot), human readability (computed styles, WCAG AA), state switching (3 states tested), viewport bounds (4 widths: 360/768/1024/1440), content fit (scroll dimensions), and cross-route regression. All six must pass before a UI task is declared complete.

**`claw-bridge-failure-modes`** — A catalog of six failure patterns with detection methods: FM1 screenshot hallucination, FM2 architectural violation, FM3 scope creep, FM4 fake verification, FM5 skipped teardown, FM6 regression ignorance. Used by `/claw-review` and `/claw-failure-check`.

**`claw-bridge-repair`** — The fix-verify loop for when a VVP check fails. Enforces: ISOLATE the specific failure (quote tool output, don't paraphrase) → HYPOTHESIZE minimal cause (grep to confirm, state before coding) → FIX narrowly (no refactoring mixed in) → RE-VERIFY the specific check → REGRESS (full VVP) → CONTINUE. Based on PDCA (Deming) and Tidy First (Kent Beck, 2023).

---

## Why this exists

Claude Code hallucinates checkpoint completion. It will say "the page renders correctly" when the DOM is empty, report "VVP 6/6 passed" when it ran zero checks, and bundle unrelated changes into a commit while noting they were "minor cleanups." These are not rare edge cases — they are systematic patterns documented in the failure-modes skill.

claw-bridge makes Claude expensive to lie to itself. UNDERSTAND means reading files, not assuming. VERIFY means tool output, not claims. The repair loop means a failed check can't be glossed over.

---

## What it doesn't do (v0.1)

- No custom Python MCP server — chrome-devtools-mcp covers the verification surface
- No hooks — slash commands + skills cover the user-invoked path (v0.2 candidate)
- No Cloudflare tunnel, no auth — local plugin, no network surface
- No orchestration — discipline is built into Claude Code itself via the skills

---

## Contributing

MIT licensed. PRs welcome. When filing issues, include real examples of failure behavior — "it didn't verify correctly" is not actionable; a specific FM number with a reproduction is.

---

## Credits

- [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) — the browser automation layer that makes VVP possible
- [Claude Code plugin docs](https://code.claude.com/docs/en/plugins-reference) — authoritative reference for plugin.json schema and skill frontmatter
