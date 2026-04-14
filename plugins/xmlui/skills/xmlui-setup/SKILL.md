---
name: xmlui-setup
description: Set up a complete XMLUI development environment. Use when the user wants to start XMLUI development, install the XMLUI CLI, configure the MCP server, or create a new XMLUI project.
disable-model-invocation: true
allowed-tools: Bash
---

# XMLUI Development Environment Setup

Your goal is to set up a complete XMLUI development environment for the user. Work through the steps below in order. At each step, run the relevant script yourself — do not ask the user to copy-paste commands unless a step explicitly requires their input.

Steps 1 and 2 use scripts at `${CLAUDE_SKILL_DIR}/scripts/`.

---

## Step 1: Preflight

Run:

```bash
"${CLAUDE_SKILL_DIR}/scripts/preflight.sh"
```

If it fails, diagnose the missing dependency and tell the user what to install. Common issues:

- `curl` missing: install via system package manager
- `claude` missing: Claude Code CLI is not installed or not on PATH
- `tar`/`unzip` missing: install via system package manager

Do not proceed until preflight passes.

---

## Step 2: Install the XMLUI CLI

The CLI binary is managed by the plugin and installed to the plugin's data directory. It does not need to be on the user's PATH.

Check if already installed:

```bash
test -x "${CLAUDE_PLUGIN_DATA}/bin/xmlui" && echo "found" || echo "not found"    # macOS / Linux
test -x "${CLAUDE_PLUGIN_DATA}/bin/xmlui.exe" && echo "found" || echo "not found" # Windows
```

If found, skip to Step 3.

If not found, run:

```bash
CLAUDE_PLUGIN_DATA="${CLAUDE_PLUGIN_DATA}" "${CLAUDE_SKILL_DIR}/scripts/install-cli.sh"
```

Verify with `"${CLAUDE_PLUGIN_DATA}/bin/xmlui" --help` before continuing.

---

## Step 3: MCP server (automatic)

The MCP server is configured automatically via the plugin's `.mcp.json` — no manual setup needed. Skip to Step 4.

If the user reports that XMLUI MCP tools are not available, verify the plugin is installed:

```bash
claude plugin list
```

If `xmlui@xmlui-claude` is listed, the MCP server should be active after a Claude Code restart.

If `claude` is not found, the Claude Code CLI is not on PATH. Ask the user to verify `claude` is accessible and try again.

---

## Step 4: Create a project

Ask the user if they want to create a new XMLUI project. If not, skip to the Final message.

First, list the available templates:

```bash
xmlui list-demos
```

Show the output to the user. Recommend **`xmlui-hello-world-trace`** — it comes with the Inspector and XS tracing already wired up, which makes debugging and AI-assisted development much easier. The other templates are available if they prefer a different starting point.

Ask the user which template they'd like to use (default: `xmlui-hello-world-trace`).

Ask the user what they'd like to name their project directory (default: `xmlui-hello-world-trace`).

Once you have both answers, run:

```bash
xmlui new <template> --output <project-name>
```

For example:

```bash
xmlui new xmlui-hello-world-trace --output my-project
```

If `xmlui new` is not recognized, there's a problem with the CLI installation. Tell the user to re-run Step 2 to get the latest version.

---

## Final message

Once all steps have completed successfully, tell the user the final message, which has 3 parts — the last 2 are optional.

> Setup is complete.

If the MCP server was added during the setup process, tell the user:

> Please restart Claude Code so the **XMLUI MCP** server becomes active to help you with XMLUI development.

If a project was created, tell the user (substituting the actual project name):

> You may want to restart Claude Code in the `project-name` directory.
