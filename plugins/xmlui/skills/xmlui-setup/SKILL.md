---
name: xmlui-setup
description: Set up a complete XMLUI development environment. Use when the user wants to start XMLUI development, install the XMLUI CLI, configure the MCP server, or create a new XMLUI project.
disable-model-invocation: true
allowed-tools: Bash, Read, Edit, Write
---

# XMLUI Development Environment Setup

`xmlui-setup` installs the XMLUI CLI, downloads the `xmlui-weather` app (which includes the Inspector for debugging), and starts a local webserver to run the app.

Run every step automatically — do not ask the user for confirmation between steps.

---

## Step 1: Preflight

Run:

```bash
"${CLAUDE_SKILL_DIR}/scripts/preflight.sh"
```

If it fails, tell the user what to install and stop. Do not proceed until preflight passes.

---

## Step 2: Install the XMLUI CLI

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

## Step 3: MCP server

The MCP server is configured automatically via the plugin's `.mcp.json` — no manual setup needed.

---

## Step 4: Download the weather app

Check if `xmlui-weather` already exists in the current directory:

```bash
test -d xmlui-weather && echo "exists" || echo "ok"
```

If it exists, tell the user and ask what they'd like to name the project directory instead. Otherwise, proceed.

Run:

```bash
"${CLAUDE_PLUGIN_DATA}/bin/xmlui" new xmlui-weather --output xmlui-weather
```

---

## Step 5: Start the dev server

```bash
cd <project-dir> && "${CLAUDE_PLUGIN_DATA}/bin/xmlui" run
```

Tell the user: **Open a browser to the indicated port (usually 8080).** You should see the Weather Dashboard with a magnifying glass icon in the top right — that's the Inspector.

Then tell the user:

> **Your XMLUI environment is ready.** See the [README](https://github.com/xmlui-org/xmlui-claude#readme) for a guided tour of the XMLUI MCP tools and the Inspector.
