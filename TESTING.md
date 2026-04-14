# xmlui-claude plugin testing

## Try it yourself

Everything below documents how we built and tested the XMLUI Claude Code plugin. We simulated the `/xmlui-setup` skill by running its steps manually. **The next step is to test the skill for real** — have Claude run it, not a human pretending to be Claude.

### Quick start (~5 minutes)

```
/plugin marketplace add judell/xmlui-claude
/plugin install xmlui@xmlui-claude
```

Restart Claude Code, then:

```
/xmlui-setup
```

The skill should walk you through:
1. Preflight checks
2. CLI installation (to the plugin data dir — no PATH conflicts)
3. MCP server confirmation (automatic via .mcp.json)
4. Create a project — choose **xmlui-weather** (has API calls worth tracing)
5. Add tracing — explains each file, asks permission, downloads from trace-tools repo
6. Start the dev server with `xmlui run`
7. Write a project CLAUDE.md that steers Claude toward the MCP tools

At the end you should have: a running weather app, a working Inspector, and an MCP-connected Claude that consults real docs instead of guessing.

### What to watch for

- **Does Claude follow the SKILL.md steps in order?** Or does it skip, improvise, or get creative?
- **Does the Inspector injection work?** The weather template has no `<AppHeader>` (Case 3 in the SKILL.md). Does Claude produce valid XML?
- **Does the trace show API calls?** After PR #3265 lands, startup DataSource fetches will be traced. Without it, only interaction events appear.
- **Does Claude explain what it's doing?** The skill asks Claude to describe each tracing file before downloading. Does it?
- **How long does it take?** Chuck's manual setup took 40 minutes. This should be under 5.
- **Does Claude use the MCP tools?** After setup, ask Claude an XMLUI question (e.g. "how does DataSource work?"). Does it call `xmlui_component_docs` or guess?

### If you already have the plugin installed

```
/plugin marketplace update xmlui-claude
/plugin update xmlui@xmlui-claude
```

Restart Claude Code and run `/xmlui-setup`.

### Report what happens

Note what Claude did well, what it got wrong, and what was confusing. Update this doc or file issues on https://github.com/judell/xmlui-claude.

---

## What we did

### 1. Evaluated commit 1c00f394b in xmlui repo

PR #3264 added a Claude Code plugin at `tools/xmlui-claude/` with:
- `plugin.json` — plugin metadata
- `SKILL.md` — `/xmlui-setup` skill with 4 steps: preflight, CLI install, MCP config, project creation
- Shell scripts — `preflight.sh`, `install-cli.sh`, `common.sh` (cross-platform: macOS, Linux, Windows)

### 2. Identified distribution gap

The original commit only supports `claude --plugin-dir ./tools/xmlui-claude` which requires having the xmlui repo cloned. No marketplace structure, so no way for users to `plugin install` it.

### 3. Learned how Claude Code plugin distribution works

- Plugins are distributed via **marketplaces** — repos (or URLs) containing `.claude-plugin/marketplace.json`
- Users add a marketplace, then install plugins from it
- Installed plugins are cached at `~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`
- Plugins can declare MCP servers via `.mcp.json` (auto-registered on install, no manual `claude mcp add` needed)

### 4. Created github.com/judell/xmlui-claude

Standalone marketplace repo as a testing proxy. Structure:

```
.claude-plugin/marketplace.json    # marketplace catalog
plugins/xmlui/
  .claude-plugin/plugin.json       # plugin metadata (now includes mcpServers reference)
  .mcp.json                        # declarative MCP server config (replaces manual Step 3)
  skills/xmlui-setup/
    SKILL.md                       # the /xmlui-setup skill
    scripts/
      common.sh
      preflight.sh
      install-cli.sh
  dev-docs/
    about-skills.md
    create-plugins.md
    install-problems.md
```

Key improvements over original commit:
- **Marketplace-distributable** — has `.claude-plugin/marketplace.json` at root
- **Declarative MCP server** — `.mcp.json` replaces the manual `claude mcp add` in SKILL.md Step 3

### 5. Installed successfully

```
claude plugin marketplace add judell/xmlui-claude   # ✔ added
claude plugin install xmlui@xmlui-claude             # ✔ installed (scope: user)
```

Files confirmed at `~/.claude/plugins/cache/xmlui-claude/xmlui/1.0.0/`.

## After restart (2026-04-14)

### MCP server: auto-registered ✔

After restart, the plugin's MCP server appeared as `mcp__plugin_xmlui_xmlui__*` tools — 12 tools total (component_docs, examples, find_trace, get_prompt, get_session_context, inject_prompt, list_components, list_howto, list_prompts, read_file, search, search_howto). The `.mcp.json` declarative approach works. SKILL.md Step 3 (manual `claude mcp add`) is now redundant and should be removed or made into a fallback check.

### MCP tools: callable ✔

Called `xmlui_list_components` successfully — returned full component list (100+ components). Also surfaced a useful warning: CLI at v0.0.1, latest is v0.0.9. This is exactly what `/xmlui-setup` Step 2 should handle.

### Note: local (not-yet-published) xmlui-cli

We're testing with a local build of `xmlui-cli` (from `~/xmlui-cli`). The CLI embeds the MCP server (`xmlui-mcp`). The version is set at build time via `-ldflags "-X main.version=$VERSION"` in `build.sh`.

### How update alerts currently work

The MCP server checks for updates on startup:

1. **Version defined** in `xmlui-cli/main.go` line 21: `var version = "dev"` (overridden at build time)
2. **Check**: `xmlui-mcp/pkg/xmluimcp/update_check.go` hits `https://api.github.com/repos/xmlui-org/xmlui-cli/releases/latest` and compares semver
3. **Surface**: The notice is prepended to the first MCP tool response (via `analytics.go` lines 169-177) and also registered as a prompt (`xmlui_update_notice`)
4. **Upgrade**: Manual only — message says "Download: https://github.com/xmlui-org/xmlui-cli/releases/latest". No self-update command exists.

### Streamlining CLI detection and upgrade

The current setup has two overlapping mechanisms for CLI installation:

**Plugin's `/xmlui-setup` Step 2** — `install-cli.sh` downloads and installs the CLI binary. This runs once at setup time.

**MCP server's update check** — detects outdated CLI on every session start, but only tells the user to manually download.

These could be unified:

1. **Remove Step 2 from the skill, or make it conditional.** If the MCP server is already running (registered via `.mcp.json`), the CLI is installed. The skill only needs Step 2 if the CLI isn't found at all — but then the MCP server also wouldn't be running.

2. **Add a self-update mechanism.** The `install-cli.sh` script already knows how to download and install the correct platform binary. The MCP server could expose an `xmlui_upgrade` tool that runs equivalent logic, or the skill could offer `/xmlui-upgrade`. This would close the loop: detect outdated → upgrade in place, without sending the user to a browser.

3. **Plugin update hook.** Claude Code plugins support hooks — a post-install or session-start hook could run the version check and auto-upgrade, or at least surface the update notice more prominently than buried in the first tool response.

4. **Chicken-and-egg problem.** If someone has no CLI at all, the MCP server can't start (it runs `xmlui mcp`). The plugin's `/xmlui-setup` skill is the bootstrap path for that case. But once installed, the MCP server's update check handles ongoing upgrades — if we give it an actual upgrade action rather than just a notice.

**Recommended approach:** Keep `install-cli.sh` for bootstrapping (no CLI at all). Add an `xmlui_upgrade` MCP tool that reuses the same download/install logic for ongoing updates. Remove the manual `claude mcp add` step (Step 3) since `.mcp.json` handles it.

### Name collision: `xmlui` npm package vs `xmlui` CLI binary

The xmlui repo installs an npm package that puts a different `xmlui` on PATH (via nvm/node bin). So even after moving `/usr/local/bin/xmlui` out of the way, `command -v xmlui` still finds the npm one. The install script's check (`command -v xmlui`) can't distinguish between the two.

This only affects people developing on the xmlui repo itself — rare, but exactly the audience for early testing.

**Decision:** Install the CLI binary to the plugin's own data directory (`${CLAUDE_PLUGIN_DATA}/bin/xmlui`) rather than `~/.local/bin`. The `.mcp.json` references it by absolute path. This avoids PATH collisions entirely — the npm `xmlui` is undisturbed, and the plugin doesn't need `xmlui` on PATH at all since it's the only consumer of `xmlui mcp`. Users who want the CLI in their shell can alias it themselves.

### v1.1.0: install to plugin data dir, auto-register MCP

Changes pushed to `judell/xmlui-claude`:
- `.mcp.json` now uses `${CLAUDE_PLUGIN_DATA}/bin/xmlui` instead of bare `xmlui`
- `install-cli.sh` installs to `${CLAUDE_PLUGIN_DATA}/bin/` when that env var is set
- SKILL.md Step 3 replaced with a note that MCP is automatic; fallback check only
- `${CLAUDE_PLUGIN_DATA}` interpolation confirmed supported in `.mcp.json` per docs

### Plugin update cycle: works ✔

```
claude plugin marketplace update xmlui-claude          # pulls latest from GitHub
claude plugin update xmlui@xmlui-claude                # 1.0.0 → 1.1.0 ✔
```

Note: must refresh the marketplace first, then update the plugin. Version bump in `plugin.json` required.

Files confirmed at `~/.claude/plugins/cache/xmlui-claude/xmlui/1.1.0/`.

### After second restart: MCP server down (expected), CLI install works ✔

As expected, the MCP server tools disappeared — no binary at `${CLAUDE_PLUGIN_DATA}/bin/xmlui` yet.

**Preflight (Step 1):** ✔ passed (darwin/arm64)

**CLI install (Step 2):** ✔
- `install-cli.sh` downloaded from `https://github.com/xmlui-org/xmlui-cli/releases/latest/download/xmlui-macos-arm64.tar.gz`
- Installed to `~/.claude/plugins/data/xmlui-xmlui-claude/bin/xmlui`
- Quarantine removal warned but non-blocking
- Binary runs: `xmlui --help` works, `xmlui mcp` available

**Note:** `CLAUDE_PLUGIN_DATA` is not available in direct bash calls — it's only set for processes launched by the plugin (MCP servers, hooks, skill scripts via skill runner). For manual testing, we set it explicitly: `CLAUDE_PLUGIN_DATA="$HOME/.claude/plugins/data/xmlui-xmlui-claude"`.

**Note:** The skill has `disable-model-invocation: true`, so it can't be invoked programmatically via the Skill tool — only by the user typing `/xmlui-setup`. For testing we ran the scripts directly.

**Note:** `CLAUDE_PLUGIN_DATA` resolves to `~/.claude/plugins/data/xmlui-xmlui-claude/` (plugin name + marketplace name, special chars replaced with `-`).

Existing `/usr/local/bin/xmlui` has been moved to `xmlui.bak` to simulate a clean start. There is also an npm `xmlui` on PATH (from the xmlui repo dev setup) which we are intentionally ignoring — the plugin data dir approach avoids this collision.

### After third restart: MCP server up from plugin data dir ✔

MCP server came back — 11 tools available, served from `~/.claude/plugins/data/xmlui-xmlui-claude/bin/xmlui`. Called `xmlui_list_components` successfully, returned full component list. No version warning this time (freshly downloaded binary is current).

### Full end-to-end flow verified ✔

| Step | Result |
|------|--------|
| `plugin marketplace add judell/xmlui-claude` | ✔ |
| `plugin install xmlui@xmlui-claude` | ✔ |
| MCP auto-registers via `.mcp.json` | ✔ |
| MCP server fails gracefully when CLI missing | ✔ (tools just don't appear) |
| Preflight script | ✔ |
| CLI install to `${CLAUDE_PLUGIN_DATA}/bin/` | ✔ |
| MCP server starts from plugin data dir after restart | ✔ |
| MCP tools callable | ✔ |
| Plugin update cycle (marketplace update → plugin update → version bump) | ✔ |

### UX issue: first-install requires two restarts

1. Install plugin → restart (MCP server fails, no CLI yet)
2. Run `/xmlui-setup` → installs CLI → restart (MCP server works)

This could be improved if the install script ran automatically as a post-install hook, reducing to one restart.

### Gap analysis: Chuck's 40-minute setup vs the skill

Reference: https://gist.github.com/judell/69e7c4cd2c6f895ffa5753120f49bf49

Chuck's setup pain points mapped to skill coverage:

| Chuck's step (~time) | Original skill | Expanded skill |
|---|---|---|
| 1. Install CLI (~10 min) | ✔ Step 2 | ✔ Step 2 |
| 2. Configure MCP server (~15 min) | ✔ Step 3 (automatic) | ✔ Step 3 (automatic) |
| 3. Install trace viewer (~10 min) | not covered | ✔ Step 5 |
| 4. Start dev server (~5 min) | not covered | ✔ Step 6 |
| 5. Side-by-side layout | not covered | out of scope (IDE) |

### Tracing: what's required and what's optional

From `xmlui-org/trace-tools` README:

**Required:**
1. `xsVerbose: true` + `xsVerboseLogMax: 200` in config.json `appGlobals`
2. `xs-diff.html` — trace viewer UI (from trace-tools repo root)
3. `xmlui-parser.es.js` — trace parser (must be in same directory as xs-diff.html)
4. `<Inspector />` — component that opens the viewer in a modal

**Optional:**
5. `xs-trace.js` — app-level tracing helpers: `xsTrace()` for timing wrappers, `xsTraceEvent()` for semantic events, `xsTraceWith()` for combined timing+data. Useful as app grows, not needed to get started.

### Reverted CDN auto-download (commit af9661ee0)

Previously landed and reverted: Inspector auto-fetching xs-diff.html, xs-trace.js, and xmlui-parser.es.js from jsDelivr CDN when not found locally. Reverted (`6a0230765`) because it was too automatic — files appeared silently.

The skill approach solves this differently: same files, same sources, but the user is walked through each one interactively. Claude explains what each file is for and asks permission before downloading. The transparency that was missing from the automatic approach is built into the skill.

### Template choice: xmlui-weather over xmlui-hello-world

`xmlui-hello-world` has almost nothing to trace — just static text and a button. `xmlui-weather` has API calls (DataSource hitting wttr.in), state variables, user input, error handling. It produces meaningful traces that demonstrate the value of the Inspector. Skill now recommends `xmlui-weather` as the default template.

### Expanded SKILL.md (v1.2.0)

New steps added:
- **Step 4** now offers create-new vs existing-project
- **Step 5** adds tracing interactively (config.json, xs-diff.html, xmlui-parser.es.js, Inspector component, optional xs-trace.js)
- **Step 6** starts the dev server with `xmlui run`
- `allowed-tools` expanded to `Bash, Read, Edit, Write` (needed for Inspector injection into Main.xmlui)

### Expanded flow tested end-to-end ✔

Simulated the full skill on `xmlui-weather` template:

| Step | Result |
|------|--------|
| Scaffold `xmlui-weather` | ✔ |
| Create config.json with `xsVerbose` | ✔ (weather template has none) |
| Download xs-diff.html from trace-tools | ✔ |
| Download xmlui-parser.es.js from trace-tools | ✔ |
| Add `<AppHeader>` + `<Inspector />` (Case 3) | ✔ |
| `xmlui run` on port 8080 | ✔ |
| Trace files served at /xmlui/ | ✔ |

Pushed as v1.2.0 to `judell/xmlui-claude`.

### Trace analysis: DataSource API calls missing from trace

Exported trace from the running weather app (`xs-trace-20260414T162824.json`). The trace captured button clicks, handler execution, and state changes — but **zero `api:start` / `api:complete` events**, even though the DataSource fires a request to `https://wttr.in/{searchCity}?format=j1` (confirmed reachable, returns 200).

Evidence: the first click's `state:changes` shows `errorMessage` clearing from `"Could not find weather data for Santa Rosa, CA."` — so the DataSource *did* fire on initial load and errored. But neither the initial (failed) nor subsequent API calls appear in the trace.

Both engine versions (0.12.14 in trace template, 0.12.15 in weather template) contain `api:start` references in the JS bundle, so the instrumentation exists in the code.

**This is a tracing gap, not a setup problem.** The skill correctly configured tracing (xsVerbose, xs-diff.html, xmlui-parser.es.js, Inspector). The engine traces interactions, handlers, and state changes but appears to not trace DataSource fetch calls.

This is exactly the kind of issue tracing is meant to surface — and exactly the kind of thing Claude can help diagnose when given a trace export.

### Root cause found and fixed

**Bug:** `window._xsLogs` (the trace event array) was only initialized inside the interaction listener (AppContent.tsx line 853) — on the first user click/keydown. But DataSource fetches fire during startup, before any interaction. `pushXsLog()` checks `Array.isArray(w._xsLogs)` and noops if false. So all startup API events were silently dropped.

The irony: line 524 already had a comment saying *"Initialize startup trace early during render, BEFORE children mount. This ensures DataLoader can capture the trace when useQuery triggers fetches."* — it initialized the startup trace ID, but forgot to initialize `_xsLogs`.

**Fix:** 3-line addition at AppContent.tsx line 526, in the same early-render block:
```typescript
if (!Array.isArray(w._xsLogs)) {
  w._xsLogs = [];
}
```

**After fix:** trace now shows the full startup sequence:
```
0 state:changes         DataSource:weatherData
1 api:start             GET https://wttr.in/Santa Rosa, CA
2 api:complete          GET https://wttr.in/Santa Rosa, CA (keys: current_condition, nearest_area, request, weather)
3 state:changes         DataSource:weatherData loaded
4 component:vars:init   Main
```

Built with `cd xmlui && npm run build:xmlui-standalone`, copied bundle to weather-test app, confirmed working.

**PR:** https://github.com/xmlui-org/xmlui/pull/3265

**Note:** The initial API failure (error message about Santa Rosa) was unrelated — wttr.in was transiently flaky. The fix only changed trace initialization, not fetch behavior.

### Boosting MCP tool usage (v1.3.0)

Claude has XMLUI MCP tools available but doesn't reliably reach for them. Three mechanisms added:

1. **`agents/xmlui-expert.md`** — a plugin agent with a system prompt that tells Claude to prefer MCP tools over guessing/web search. Lists each tool and when to use it.
2. **`settings.json`** — sets `xmlui-expert` as the default agent when the plugin is active.
3. **SKILL.md Step 7** — writes a project-level `.claude/CLAUDE.md` with MCP tool guidance. This persists across sessions and is scoped to the project.

These are layered: the agent provides always-on context when the plugin is enabled, and the project CLAUDE.md reinforces it for anyone working in that directory even without the plugin.

## Open questions

- Should we add an `xmlui_upgrade` MCP tool to close the detect → upgrade loop?
- Can a post-install plugin hook run `install-cli.sh` automatically to avoid the two-restart problem?
- Restore `/usr/local/bin/xmlui` from `xmlui.bak` when done testing
- Does the `settings.json` `"agent": "xmlui-expert"` actually activate? Need to test after plugin update + restart.
