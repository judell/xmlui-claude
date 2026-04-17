---
name: xmlui-setup
description: Set up a complete XMLUI development environment. Use when the user wants to start XMLUI development, install the XMLUI CLI, configure the MCP server, or create a new XMLUI project.
disable-model-invocation: true
allowed-tools: Bash, Read, Edit, Write
---

# XMLUI Development Environment Setup

`xmlui-setup` installs the XMLUI CLI, configures the MCP server, downloads the `xmlui-weather` app, configures the app to use the XMLUI Inspector for debugging, and starts a local webserver to run the app.

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

If the CLI was just installed in this session (Step 2), note that a Claude Code restart will be needed at the end for the MCP server to become active.

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

## Step 5: Configure the Inspector

The Inspector requires: `xsVerbose: true` in config.json, the trace viewer files (`xs-diff.html` and `xmlui-parser.es.js`), and the `<Inspector />` component in Main.xmlui.

### 5a: Enable tracing in config.json

Read the project's `config.json`.

**If it doesn't exist**, create it:

```json
{
  "appGlobals": {
    "xsVerbose": true,
    "xsVerboseLogMax": 200
  }
}
```

**If it exists**, add `"xsVerbose": true` and `"xsVerboseLogMax": 200` to `appGlobals` (creating the key if needed). Do not overwrite other settings.

### 5b: Download trace-tools files

```bash
mkdir -p <project-dir>/xmlui
curl -fsSL https://raw.githubusercontent.com/xmlui-org/trace-tools/main/xs-diff.html -o <project-dir>/xmlui/xs-diff.html
curl -fsSL https://raw.githubusercontent.com/xmlui-org/trace-tools/main/xmlui-parser.es.js -o <project-dir>/xmlui/xmlui-parser.es.js
```

### 5c: Add the Inspector component to Main.xmlui

Read `Main.xmlui` in the project directory.

**If it already contains `<Inspector`**, skip.

Otherwise, determine which case applies and use the Edit tool:

**Case 1: `<AppHeader>` exists and has a `profileMenuTemplate` property**

Add `<Inspector />` inside the existing `profileMenuTemplate` block.

**Case 2: `<AppHeader>` exists but has no `profileMenuTemplate`**

Add a `profileMenuTemplate` property with Inspector inside the `<AppHeader>`:

```xml
<AppHeader ...existing-attrs...>
  <property name="profileMenuTemplate">
    <Inspector />
  </property>
  ...existing content...
</AppHeader>
```

**Case 3: No `<AppHeader>` at all**

Add an `<AppHeader>` with Inspector right after the `<App>` opening tag:

```xml
<App ...>
  <AppHeader>
    <property name="profileMenuTemplate">
      <Inspector />
    </property>
  </AppHeader>
  ...existing content...
```

---

## Step 6: Start the dev server

```bash
cd <project-dir> && "${CLAUDE_PLUGIN_DATA}/bin/xmlui" run
```

Tell the user: **Open a browser to the indicated port (usually 8080).** You should see a magnifying glass icon in the top right — that's the Inspector. Click it to view traces.

If the CLI was just installed in this session (Step 2), also tell the user:

> **Important:** Restart Claude Code so the XMLUI MCP server becomes active, then come back to this project directory.
