# claw-bridge v0.1 — Build Specification

**For:** Claude Code, building this project end-to-end
**Author:** Varun (design), drafted via Claude (Opus 4.7)
**License target:** MIT
**Status:** Ready to build — this is the spec, not a draft

---

## 0. Kickoff prompt (paste this to Claude Code)

Put this file in an empty folder. Then run Claude Code in that folder and paste:

> Read `CLAW_BRIDGE_V0.1_BUILD.md` in full before doing anything else. This is the complete build specification for a project called **claw-bridge** — a Claude Code plugin that enforces verification discipline.
>
> **You must follow Karpathy's 4-step discipline for this build itself:**
>
> 1. **UNDERSTAND** — Before writing any file, read the spec end-to-end. Then read the linked external references (plugin.json schema, chrome-devtools-mcp README, Claude Code plugins docs) to confirm the current format. Do NOT assume from training data — the plugin system shipped in late 2025 and format details may have evolved. Report back what you learned and what uncertainties remain. Wait for my confirmation before proceeding.
> 2. **DECOMPOSE** — Break the build into the atomic commits listed in Section 9. One commit per logical unit. Show me the commit plan before you start writing.
> 3. **SOLVE** — Implement one commit at a time. Each commit must be self-contained and leave the repo in a working state.
> 4. **VERIFY** — After each commit, verify using chrome-devtools-mcp and filesystem checks per Section 10. Do not claim something works without evidence. If a check fails, STOP and report — do not "fix and move on" without my approval.
>
> Start with UNDERSTAND. Do not write any files yet. Report your findings.

---

## 1. What we're building

A **Claude Code plugin** called `claw-bridge` that:

1. Enforces 4-step discipline (UNDERSTAND / DECOMPOSE / SOLVE / VERIFY) on coding tasks via a SKILL.md skill.
2. Provides a Visual Verification Protocol (VVP) as a second skill — concrete checks Claude must run before declaring a task complete.
3. Exposes slash commands for the most common verification workflows.
4. Declares `chrome-devtools-mcp` as an MCP dependency so verification tools come pre-wired.
5. Optionally, adds a PostToolUse hook that reminds Claude to run verification before marking work as done. (v0.1 scope flexible; see Section 5.)

The plugin is installed with **one command** by end users:

```
/plugin install github.com/bishwarupdas/claw-bridge
```

No Python, no tunnel, no setup script, no bearer tokens in v0.1.

---

## 2. What we are explicitly NOT building in v0.1

These were in the original design doc and are **deferred or dropped**:

- ❌ Custom Python MCP server (chrome-devtools-mcp covers the verification surface)
- ❌ tmux-based Claude Code orchestration (brittle, platform-dependent, unnecessary — the plugin runs *inside* Claude Code)
- ❌ Cloudflare tunnel / remote HTTP MCP transport (local plugin, no network surface)
- ❌ Bearer token auth, security model, audit logs (no network = nothing to secure beyond what Claude Code already handles)
- ❌ The "chat Claude orchestrates Claude Code" two-agent pattern (replaced: discipline is built into Claude Code itself via the skill)
- ❌ Playwright / custom screenshot analysis (chrome-devtools-mcp has `take_screenshot`, `take_snapshot`, `list_console_messages`, etc.)
- ❌ PyPI publish, Homebrew formula, Docker image, VS Code extension

These can come in v0.2+ if and only if real-world use shows they're needed.

---

## 3. Target user experience

### First-time install

```
$ claude
> /plugin marketplace add github.com/bishwarupdas/claw-bridge
  ✓ Added marketplace

> /plugin install claw-bridge@claw-bridge
  ✓ Installed claw-bridge
  ✓ Installed dependency: chrome-devtools-mcp
  ✓ 3 skills loaded, 5 slash commands registered
  
> /reload-plugins
  ✓ Reloaded
```

Prerequisites: Node.js 22+ (for chrome-devtools-mcp), recent Chrome. That's it.

### Daily use

```
> /claw-plan Build the user authentication page per docs/auth-spec.md
  [plugin generates a 4-step kickoff, asks for confirmation, then runs]

> /claw-verify HEAD
  [plugin runs VVP checks on latest commit, reports pass/fail with evidence]

> /claw-review abc123
  [plugin reviews diff + screenshots of commit abc123]
```

---

## 4. Repository layout (target)

```
claw-bridge/
├── .claude-plugin/
│   └── plugin.json                    # manifest — see Section 5
├── skills/
│   ├── discipline/
│   │   └── SKILL.md                   # 4-step discipline — see Section 6
│   ├── vvp/
│   │   └── SKILL.md                   # Visual Verification Protocol — Section 6
│   └── failure-modes/
│       └── SKILL.md                   # Known Claude Code failure patterns — Section 6
├── commands/
│   ├── claw-plan.md                   # /claw-plan <task> — Section 7
│   ├── claw-verify.md                 # /claw-verify [commit] — Section 7
│   ├── claw-review.md                 # /claw-review <commit> — Section 7
│   ├── claw-status.md                 # /claw-status — Section 7
│   └── claw-failure-check.md          # /claw-failure-check — Section 7
├── hooks/                             # v0.1 scope: optional, see Section 8
│   └── post-tool-use.json             # (optional) remind to verify before "done"
├── README.md                          # public-facing, see Section 11
├── LICENSE                            # MIT
├── CHANGELOG.md                       # start with 0.1.0
└── .gitignore                         # standard Node / macOS / editor ignores
```

No `src/`, no `pyproject.toml`, no `package.json`, no build step. Just markdown + JSON.

---

## 5. plugin.json manifest

Authoritative source for current schema: **Claude Code plugin docs** at `https://code.claude.com/docs/en/discover-plugins` and the sibling "Create and distribute a plugin marketplace" page. Read those first; schema below is the current shape but verify before finalizing.

```json
{
  "name": "claw-bridge",
  "version": "0.1.0",
  "description": "Discipline enforcement + visual verification for Claude Code. Built-in 4-step protocol, pre-wired chrome-devtools-mcp for verification.",
  "author": {
    "name": "bishwarupdas",
    "url": "https://github.com/bishwarupdas/claw-bridge"
  },
  "license": "MIT",
  "homepage": "https://github.com/bishwarupdas/claw-bridge",
  "keywords": ["verification", "discipline", "karpathy", "vvp", "visual-verification"],
  "skills": [
    "./skills/discipline/SKILL.md",
    "./skills/vvp/SKILL.md",
    "./skills/failure-modes/SKILL.md"
  ],
  "commands": [
    "./commands/claw-plan.md",
    "./commands/claw-verify.md",
    "./commands/claw-review.md",
    "./commands/claw-status.md",
    "./commands/claw-failure-check.md"
  ],
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

**UNDERSTAND step for Claude Code:** Before writing this file, verify:
- Exact field names (`skills` vs `skillPaths`, `commands` vs `commandPaths`)
- Whether MCP server declaration is `mcpServers` in plugin.json or a separate `.mcp.json`
- Whether `dependencies` is a supported field or if MCP servers are the dependency mechanism
- Node version requirement for chrome-devtools-mcp (22+ per current docs)

If the schema differs, update this section with what you find and continue.

---

## 6. The three skills

Each skill is a SKILL.md file. Claude Code loads skills on demand; the YAML frontmatter's `description` field is what Claude uses to decide when to apply the skill, so these must be precise.

### 6.1 `skills/discipline/SKILL.md`

```markdown
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

If any verification fails, STOP. Report the failure with specifics. Do not try to fix and re-claim without user approval.

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
```

### 6.2 `skills/vvp/SKILL.md`

```markdown
---
name: claw-bridge-vvp
description: Visual Verification Protocol — runs concrete checks against work with visual output using chrome-devtools-mcp tools. Use this skill before declaring any UI or frontend task complete. Provides 6 checks covering content visibility, readability, state transitions, viewport bounds, content fit, and cross-tab regression.
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
evaluate_script: 
  const el = document.querySelector('<selector>');
  return { scrollHeight: el.scrollHeight, clientHeight: el.clientHeight };
```
scrollHeight ≤ clientHeight, OR vertical scroll is explicitly intentional design.

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

Do not pass checks you did not run. Do not report PASS without evidence.
```

### 6.3 `skills/failure-modes/SKILL.md`

```markdown
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
```

---

## 7. Slash commands (5 commands, ~50-80 lines of markdown each)

Each slash command is a markdown file under `commands/`. The filename (without `.md`) becomes the command name. The file body is the prompt that executes when the user invokes the command.

### 7.1 `commands/claw-plan.md`

```markdown
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
```

### 7.2 `commands/claw-verify.md`

```markdown
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

Do not skip checks. If a check can't be run (e.g., no dev server running), report it as BLOCKED with the reason.
```

### 7.3 `commands/claw-review.md`

```markdown
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
```

### 7.4 `commands/claw-status.md`

```markdown
---
name: claw-status
description: Show claw-bridge status — skills active, commands available, chrome-devtools-mcp connectivity, recent VVP runs.
---

Report:
1. claw-bridge version (read from .claude-plugin/plugin.json)
2. Active skills (list all three claw-bridge-* skills)
3. Available commands (list all five)
4. chrome-devtools-mcp status: attempt a `list_pages` call. If it succeeds, report OK; if it fails, report the error.
5. If there are any files named `vvp-*.png` or `claw-report-*.md` in `/tmp`, list them (recent verification artifacts).
```

### 7.5 `commands/claw-failure-check.md`

```markdown
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
```

---

## 8. Hooks (v0.1 decision: SKIP initially)

Hooks would fire on Claude Code lifecycle events (PostToolUse, Stop, UserPromptSubmit). A natural hook would be: on Stop, if the session edited files, remind Claude to run `/claw-verify`.

**Decision for v0.1: skip.** Reasons:
- Hooks introduce behavioral side effects that are hard to debug
- The slash commands + skills already cover the "user-invoked" path
- A hook that fires on every Stop event is annoying if the user was doing something trivial
- Getting hooks wrong creates negative first impressions

**v0.2 candidate:** Stop-event hook that checks `git diff --stat` and, if the diff contains `.tsx/.jsx/.html/.vue/.svelte` files, reminds Claude to run VVP. Only fires when visual files changed.

Do not build the hook in v0.1. Reserve the `hooks/` directory in the layout for v0.2.

---

## 9. Atomic commit plan (DECOMPOSE step for Claude Code)

Before writing code, confirm this plan with the user. Adjust if needed.

1. **commit: scaffold repo** — `README.md` (stub), `LICENSE` (MIT), `.gitignore`, `CHANGELOG.md` (0.1.0 entry)
2. **commit: plugin manifest** — `.claude-plugin/plugin.json` with verified schema
3. **commit: discipline skill** — `skills/discipline/SKILL.md`
4. **commit: VVP skill** — `skills/vvp/SKILL.md`
5. **commit: failure-modes skill** — `skills/failure-modes/SKILL.md`
6. **commit: /claw-plan command** — `commands/claw-plan.md`
7. **commit: /claw-verify command** — `commands/claw-verify.md`
8. **commit: /claw-review command** — `commands/claw-review.md`
9. **commit: /claw-status command** — `commands/claw-status.md`
10. **commit: /claw-failure-check command** — `commands/claw-failure-check.md`
11. **commit: README full content** — public-facing docs (see Section 11)
12. **commit: verify install end-to-end** — this is the VERIFY step; see Section 10. Don't commit anything; instead, produce a verification report.

12 commits. Each one small, reviewable, revertable.

---

## 10. Verification protocol for the build itself (VERIFY step)

Before declaring v0.1 done, run this checklist. Use filesystem + chrome-devtools-mcp where applicable.

### 10.1 Static checks
- [ ] `plugin.json` parses as valid JSON (`python -m json.tool .claude-plugin/plugin.json` or `jq . .claude-plugin/plugin.json`)
- [ ] Every file path referenced in `plugin.json` exists on disk
- [ ] Every SKILL.md has valid YAML frontmatter with `name` and `description` fields
- [ ] Every command file has valid frontmatter with `name` and `description`
- [ ] README mentions: install command, prerequisites, the five slash commands, link to chrome-devtools-mcp

### 10.2 Install smoke test

Create a temporary empty directory outside the repo. In a fresh Claude Code session:
```
/plugin marketplace add <local-path-to-this-repo>
/plugin install claw-bridge@claw-bridge
/reload-plugins
/claw-status
```

Expected:
- All three skills load without error
- All five commands are registered
- `/claw-status` reports OK and confirms chrome-devtools-mcp is reachable

If any step fails: STOP. Do not declare v0.1 done. Report the failure and its cause.

### 10.3 Functional test

On any simple web project (Vite starter, Next.js starter, or point at localhost:3000 of anything):
```
/claw-verify
```

Expected: VVP skill activates, uses chrome-devtools-mcp to navigate + screenshot + snapshot, produces a VVP report in the defined format. At least one check should actually run (even if others are BLOCKED).

### 10.4 Honest report

Produce a file `BUILD_VERIFICATION.md` (not committed — this is a build artifact) containing:
- What you tested
- What passed with evidence (screenshot paths, command outputs)
- What failed and why
- What you could not test and why

Do NOT produce a clean "all green" report unless it's actually true. A FAIL report with specifics is more valuable than a dishonest PASS.

---

## 11. README structure (for commit 11)

Public-facing. Must cover:

1. **What this is** — one paragraph. Plugin for Claude Code that enforces verification discipline.
2. **Install** — one code block, the `/plugin install` line.
3. **Prerequisites** — Node 22+, recent Chrome. That's it.
4. **The five slash commands** — one-line description each.
5. **The three skills** — one paragraph each, what triggers them.
6. **Why this exists** — short version of "Claude Code hallucinates checkpoint completion; this makes it honest."
7. **What it doesn't do** — honest about v0.1 scope; no custom MCP server, no orchestration.
8. **Contributing** — MIT, PRs welcome, file issues with real examples.
9. **Credits** — mention chrome-devtools-mcp, link to the Claude Code plugin docs.

Keep it under 300 lines. A README that's too long is a README nobody reads.

---

## 12. Open questions Claude Code must resolve in UNDERSTAND step

Do not guess on these. Search the current docs.

1. **plugin.json field names** — is it `skills` or `skillPaths` or something else? Is `mcpServers` inside `plugin.json` or in a separate `.mcp.json`?
2. **Command frontmatter** — what fields are required? Is it `argument-hint` or `argumentHint` or `arguments`?
3. **Marketplace registration** — does `github.com/bishwarupdas/<REPO>` work directly as a marketplace source, or does it need a `marketplace.json` in the repo root?
4. **chrome-devtools-mcp installation via plugin manifest** — does Claude Code auto-install declared MCP servers, or does the user still need to run `claude mcp add` separately? This is the biggest UX question. If it's not auto, the README must reflect that.

Reference URLs (verify these still work; update if the docs have moved):
- `https://code.claude.com/docs/en/discover-plugins`
- `https://github.com/ChromeDevTools/chrome-devtools-mcp`
- `https://developer.chrome.com/docs/devtools/agents`

Report findings before writing any files.

---

## 13. What done looks like

v0.1 is done when:
- Repo contains all files in Section 4
- All 12 commits in Section 9 are made with clean messages
- Section 10 verification passes with real evidence
- `BUILD_VERIFICATION.md` reports honest results
- A fresh Claude Code session in an empty directory can `/plugin install` the repo and get a working `/claw-status` output

Publishing to GitHub and announcing is a post-v0.1 task — not part of this build.

---

## 14. Rules for Claude Code during this build

- Do not skip UNDERSTAND. Ever.
- Do not batch commits. One logical change per commit.
- Do not claim verification you didn't run. Section 10 is not optional.
- If you find the plugin.json schema is different from Section 5, update Section 5 of this doc with the real schema before proceeding — make it a documentation fix commit.
- If something is ambiguous, ask. Don't guess.
- If you discover a better way to structure something, surface it — don't silently deviate.
- This document is the spec. If reality contradicts the spec, the spec gets updated, not ignored.

Good luck.
