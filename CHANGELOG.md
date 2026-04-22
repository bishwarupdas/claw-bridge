# Changelog

## [0.1.0] — 2026-04-23

### Added
- `claw-bridge-discipline` skill: 4-step UNDERSTAND/DECOMPOSE/SOLVE/VERIFY protocol
- `claw-bridge-vvp` skill: Visual Verification Protocol with 6 checks via chrome-devtools-mcp
- `claw-bridge-failure-modes` skill: catalog of 6 known Claude Code failure patterns
- `claw-bridge-repair` skill: ISOLATE/HYPOTHESIZE/FIX/RE-VERIFY/REGRESS loop for VVP failures
- `/claw-plan` command: generate a discipline-gated kickoff plan
- `/claw-verify` command: run VVP against working tree or a commit
- `/claw-review` command: audit a commit diff for scope creep and missing verification
- `/claw-status` command: show plugin status and chrome-devtools-mcp connectivity
- `/claw-failure-check` command: self-audit recent work against the 6 failure modes
- `chrome-devtools-mcp` declared as plugin MCP server (starts automatically)
