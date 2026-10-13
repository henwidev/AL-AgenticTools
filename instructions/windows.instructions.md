---
description: Use native Windows and PowerShell commands
applyTo: "**"
---

This machine runs native Windows. The default shell is PowerShell 7.

For terminal operations:
- Always produce and execute PowerShell-compatible commands.
- Never assume Bash, GNU utilities, WSL, or a Unix-like shell is available.
- Do not use Bash syntax such as `export`, `source`, `$(...)`, `./script.sh`,
  `rm -rf`, `grep`, `sed`, `awk`, or `/tmp`.
- Use PowerShell cmdlets and `$env:VARIABLE` syntax.
- Quote Windows paths safely.
- If shell identity is uncertain, check it before executing commands.
- Only use Linux commands when I explicitly say the task runs in WSL, Docker,
  a Linux VM, or a Linux CI environment.
