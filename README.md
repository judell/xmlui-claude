# xmlui-claude

A Claude Code plugin marketplace for XMLUI development.

## Install

```
/plugin marketplace add xmlui-org/xmlui-claude
/plugin install xmlui@xmlui-claude
```

## What it does

The `xmlui` plugin provides:

- **`/xmlui-setup` skill** — Automates XMLUI development environment setup: preflight checks, CLI installation, MCP server configuration, and project scaffolding
- **XMLUI MCP server** — Registered automatically on install, giving Claude access to XMLUI documentation and tooling

## For internal testers

1. Run the two install commands above in Claude Code
2. Restart Claude Code so the MCP server activates
3. Use `/xmlui-setup` to set up your environment, or just start asking Claude about XMLUI
