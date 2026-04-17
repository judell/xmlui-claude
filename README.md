# xmlui-claude

A Claude Code plugin marketplace for XMLUI development.

## Prerequisites

Claude Code. See the [quickstart](https://code.claude.com/docs/en/quickstart).


## Add a marketplace

```
/plugin marketplace add xmlui-org/xmlui-claude
⎿  Successfully added marketplace: xmlui-claude
```

## Install the xmlui plugin

```
/plugin install xmlui@xmlui-claude
```

When prompted for install scope, choose the default: Install for you (user scope). This makes the plugin available across all your projects.

```
❯ /plugin install xmlui@xmlui-claude
  ⎿  ✓ Installed xmlui. Run /reload-plugins to apply.
```

## Run /xmlui-setup

The `xmlui-setup` skill installs and configures tools to help you develop XMLUI apps effectively. The XMLUI CLI lists, fetches, and runs demo apps. The XMLUI MCP server helps Claude use the XMLUI documentation.

```/xmlui-setup

 When prompted:
  - Choose **create a new project**
  - Pick the **xmlui-weather** template
  - Say **yes** to tracing

## What it does

The `xmlui` plugin provides:

- **`/xmlui-setup` skill** — Automates XMLUI development environment setup: preflight checks, CLI installation, MCP server configuration, and project scaffolding
- **XMLUI MCP server** — Registered automatically on install, giving Claude access to XMLUI documentation and tooling

## For internal testers

1. Run the two install commands above in Claude Code
2. Restart Claude Code so the MCP server activates
3. Use `/xmlui-setup` to set up your environment, or just start asking Claude about XMLUI
