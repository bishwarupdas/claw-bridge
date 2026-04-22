---
name: claw-status
description: Show claw-bridge status — skills active, commands available, chrome-devtools-mcp connectivity, recent VVP runs.
---

Report:
1. claw-bridge version (read from .claude-plugin/plugin.json)
2. Active skills (list all four claw-bridge-* skills)
3. Available commands (list all five)
4. chrome-devtools-mcp status: attempt a `list_pages` call. If it succeeds, report OK with the page count; if it fails, report the error verbatim.
5. If there are any files named `vvp-*.png` or `claw-report-*.md` in `/tmp`, list them (recent verification artifacts).
