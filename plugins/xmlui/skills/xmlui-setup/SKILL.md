---
name: xmlui-setup
description: Set up a complete XMLUI development environment. Use when the user wants to start XMLUI development, install the XMLUI CLI, configure the MCP server, or create a new XMLUI project.
disable-model-invocation: true
allowed-tools: Bash, Read, Edit, Write
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

---

## Step 4: Create or choose a project

Ask the user: **Do you want to create a new XMLUI project, or set up tracing on an existing project?**

### Option A: Create a new project

List the available templates:

```bash
"${CLAUDE_PLUGIN_DATA}/bin/xmlui" list-demos
```

Recommend **`xmlui-weather`** — it's a real app with API calls, state variables, and user input, which gives you something meaningful to trace and work with. The other templates are available if they prefer a different starting point.

Ask the user which template they'd like (default: `xmlui-weather`) and what to name the project directory.

Run:

```bash
"${CLAUDE_PLUGIN_DATA}/bin/xmlui" new <template> --output <project-name>
```

Then proceed to **Step 5** to add tracing.

### Option B: Existing project

Ask the user for the project directory path. Verify it contains a `Main.xmlui` file. Then proceed to **Step 5**.

---

## Step 5: Add tracing

Before making any changes, explain to the user what tracing is and what it involves:

> **Tracing lets you (and Claude) see what your app is doing at runtime — API calls, state changes, handler timing.** It requires four things:
>
> 1. **`xsVerbose: true` in config.json** — turns on trace collection in the XMLUI engine
> 2. **`xs-diff.html`** — the trace viewer UI, displayed when you click the Inspector icon
> 3. **`xmlui-parser.es.js`** — the trace parser, used by the viewer to process raw trace data (must be in the same directory as xs-diff.html)
> 4. **`<Inspector />`** — an XMLUI component that adds a magnifying glass icon to your app header; clicking it opens the trace viewer
>
> **Optional:** `xs-trace.js` lets you add custom app-level tracing — timing wrappers (`xsTrace`) and semantic events (`xsTraceEvent`). Not needed to get started but useful as your app grows. Want me to include it?
>
> All trace-tools files come from the `xmlui-org/trace-tools` repo on GitHub. Shall I set this up?

Wait for the user to confirm before proceeding. Note whether they want `xs-trace.js`.

### 5a: Enable tracing in config.json

Read the project's `config.json`.

**If it doesn't exist**, create it with the Write tool:

```json
{
  "appGlobals": {
    "xsVerbose": true,
    "xsVerboseLogMax": 200
  }
}
```

**If it exists**, read it and check for `xsVerbose` in `appGlobals`:
- If already present, tell the user and skip.
- If `appGlobals` exists but has no `xsVerbose`, use the Edit tool to add `"xsVerbose": true` and `"xsVerboseLogMax": 200` to the existing `appGlobals`. Do not overwrite other settings.
- If no `appGlobals` key exists, use the Edit tool to add it.

Tell the user: **Tracing is now enabled. The XMLUI engine will collect detailed logs of every interaction.**

### 5b: Download trace-tools files

Check which files already exist in the project's `xmlui/` directory before downloading.

**Required** — both must be in the same directory:

```bash
mkdir -p <project-dir>/xmlui
curl -fsSL https://raw.githubusercontent.com/xmlui-org/trace-tools/main/xs-diff.html -o <project-dir>/xmlui/xs-diff.html
curl -fsSL https://raw.githubusercontent.com/xmlui-org/trace-tools/main/xmlui-parser.es.js -o <project-dir>/xmlui/xmlui-parser.es.js
```

**Optional** — only if the user said yes:

```bash
curl -fsSL https://raw.githubusercontent.com/xmlui-org/trace-tools/main/xs-trace.js -o <project-dir>/xmlui/xs-trace.js
```

Tell the user what was downloaded:
- **xs-diff.html**: the trace viewer — renders interactive timelines of your app's behavior
- **xmlui-parser.es.js**: the trace parser — processes raw trace data for the viewer
- **xs-trace.js** (if included): app-level tracing helpers — `xsTrace()` for timing, `xsTraceEvent()` for semantic events

### 5c: Add the Inspector component to Main.xmlui

Read `Main.xmlui` in the project directory.

**If it already contains `<Inspector`**, tell the user and skip.

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

Tell the user: **Added the Inspector component. You'll see a magnifying glass icon in the top right of your app — click it to view traces.**

---

## Step 6: Start the dev server

```bash
cd <project-dir> && "${CLAUDE_PLUGIN_DATA}/bin/xmlui" run
```

Tell the user:

> Your app is running at the URL shown above. Open it in your browser. You should see a magnifying glass icon in the top right — that's the Inspector. Click it to view traces of your app's behavior.

If port 8080 is already in use, `xmlui run` will pick a random port automatically.

---

## Final message

Once all steps have completed, tell the user:

> **Setup is complete.** You have:
> - The XMLUI CLI (managed by the plugin)
> - The XMLUI MCP server (gives Claude access to component docs, examples, and how-tos)
> - Tracing enabled with the Inspector (click the magnifying glass icon)
> - A running dev server
>
> Try interacting with your app in the browser, then open the Inspector to see what happened. You can also ask Claude to look at traces to help diagnose issues.

If the CLI was just installed in this session (Step 2), add:

> **Important:** Restart Claude Code so the XMLUI MCP server becomes active, then come back to this project directory.
