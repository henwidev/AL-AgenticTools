---
description: "Connection defaults for bc-dev MCP tool calls in this workspace."
applyTo: "**"
---

# BC Dev Connection Defaults

When calling bc-dev MCP tools that accept connection parameters, pass these
defaults unless the user explicitly specifies another target:

- `server`: `"http://BC28"`
- `serverInstance`: `"BC"`
- `environmentType`: `"OnPrem"`

Only include parameters supported by the tool's input schema. Session-scoped
tools reuse their existing connection and do not need these parameters.

Do not create or modify `launch.json` to configure this connection.