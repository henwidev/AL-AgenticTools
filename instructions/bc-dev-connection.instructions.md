---
description: "Connection instructions for bc-dev MCP tool calls."
applyTo: "**"
---

# BC Dev Connection Defaults

When calling bc-dev MCP tools that accept connection parameters, search for
a valid launch configuration in the vs code workspace file.

Do not create or modify `launch.json` to configure a connection.

Only include parameters supported by the tool's input schema. Session-scoped
tools reuse their existing connection and do not need these parameters.

If no launch configuration is found, halt and ask the user to provide one, ideally
in the workspace file.