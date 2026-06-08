---
name: catalyst-zoho-mcp
description: "Catalyst Zoho MCP — manage Catalyst infrastructure (tables, buckets, cache) via CatalystbyZoho_* MCP tools using natural language. Trigger on 'Zoho MCP', 'MCP tools', 'catalyst MCP', 'CatalystbyZoho', 'create table with AI', 'MCP setup', 'MCP config', or 'infrastructure as conversation'."
compatibility: "Requires an MCP-capable AI host: Claude Desktop, VS Code with GitHub Copilot, or Cursor."
metadata:
  version: "2.0.0"
---

## How It Works

1. **Verify MCP is connected** — Look for `CatalystbyZoho_*` in the tool list. If not present, guide the user to configure the MCP server (see `references/zoho-mcp.md` setup section).
2. **Pre-flight sequence** — Always run `CatalystbyZoho_List_All_Organizations` → `CatalystbyZoho_List_All_Projects` first to set project context before any other MCP tool call.
3. **Load `references/zoho-mcp.md`** — for the full tool catalog, execution flow, and common error fixes.
4. **Answer** — Call the appropriate `CatalystbyZoho_*` tool directly. Show the user what tool was called and what it returned.

## Triggers

Use this skill for: "Zoho MCP", "MCP tools", "catalyst MCP", "create table with AI", "MCP DataStore", "MCP Cache", "MCP Stratus", `zoho-mcp-server`, "MCP setup", "MCP config", "infrastructure as conversation", "Claude MCP Catalyst", "MCP tool error", "MCP not connecting", "use AI to create database table", or `CatalystbyZoho_` tool.

## References

| Reference | Load when the query is about… |
|-----------|-------------------------------|
| `references/zoho-mcp.md` | MCP server setup (Claude Desktop + VS Code), all available CatalystbyZoho_* tools, execution flow, org→project pre-flight sequence, common MCP errors |
