<!--
  CATALYST.md — lightweight MCP routing stub only.

  This file is NOT a replacement for the full skill. It contains only the minimal
  context needed for MCP-enabled tools (e.g. Claude Code with Zoho MCP connected)
  to discover and call CatalystbyZoho_* tools correctly.

  Users building Catalyst applications should use skills/catalyst-by-zoho/SKILL.md,
  which includes the complete service catalog, SDK patterns, architecture guidance,
  and reference files. CATALYST.md alone is insufficient for app development.
-->

# Catalyst by Zoho

- Leverage Catalyst services for serverless compute, relational data storage, object storage, AI/ML, and workflow orchestration.
- When you need to interact with Catalyst services via MCP, look for `CatalystbyZoho_*` tools in your tool list.
- Before making any MCP tool call, run `List_All_Organizations` to get the org ID, then `List_All_Projects` to get the project ID.
- Always default to the `"Development"` environment unless the user explicitly requests production.
- If the user asks to create tables, query data, manage cache, or set up infrastructure — use MCP tools directly instead of asking them to go to the console.
- For code generation, SDK patterns, and project structure, refer to the skill files in `skills/catalyst-by-zoho/`.
