---
name: distill-trace
description: Analyze or distill an XMLUI Inspector trace. Use when the user says "analyze the trace", "distill the trace", "what happened in the trace", or wants to understand an exported trace file.
allowed-tools: Read, Bash, mcp__plugin_xmlui_xmlui__xmlui_find_trace
---

# Distill an XMLUI Inspector Trace

The user has exported a trace JSON file from the XMLUI Inspector. Your job is to distill the raw log events into a concise step-by-step summary of what happened.

First, call the `xmlui_find_trace` MCP tool to locate the most recent trace export. Then read that file and follow the algorithm below.

---

## Input format

The trace is a JSON array of log event objects. Each event has:

- `kind` — event type: `interaction`, `api:start`, `api:complete`, `state:changes`, `value:change`, `handler:start`, `handler:complete`, `toast`, `modal:show`, `modal:confirm`, `modal:cancel`, `navigate`, `app:trace`, `focus:change`, `component:vars:init`
- `traceId` — groups events that belong to the same user action (e.g. `startup-abc123`, `i-xyz-456`)
- `perfTs` — high-resolution timestamp (milliseconds since page load), use for ordering
- `ts` — wall-clock timestamp (fallback if `perfTs` missing)

### Key event kinds

**`interaction`** — a user action (click, keydown, dblclick, contextmenu). Fields:
- `interaction` or `eventName` — the action type
- `componentType`, `componentLabel` — the XMLUI component
- `ariaRole`, `ariaName` — accessible role/name (e.g. `button`, `textbox`)
- `uid` — component test ID
- `detail` — object with `targetTag`, `text`, `ariaRole`, `ariaName`, `key` (for keydown), modifier keys (`ctrlKey`, `shiftKey`, `metaKey`, `altKey`)

**`api:start` / `api:complete`** — HTTP requests made by DataSources. Fields:
- `method` — HTTP method (GET, POST, PUT, DELETE)
- `url` or `endpoint` — the URL
- `result` — response data (on `api:complete`)
- `body` — request body (on `api:start`, for POST/PUT/PATCH)
- `status` — HTTP status code

**`value:change`** — a form control's value changed. Fields:
- `component` — component type (TextBox, Select, etc.)
- `displayLabel` — the new display value
- `ariaName` — accessible name of the control
- `componentLabel` — component ID/label

**`state:changes`** — reactive state mutations. Fields:
- `diffJson` — array of `{ path, before, after }` diffs

**`toast`** — notification shown to user. Fields:
- `toastType` — success, error, warning, info
- `message` — toast text

**`navigate`** — page navigation. Fields:
- `from`, `to` — URLs

**`app:trace`** — custom trace points emitted by the app. Fields:
- `label` — trace point name
- `data` — arbitrary data object

**`handler:start`** — event handler invoked. Fields:
- `eventName` — handler event (click, submit, didChange, etc.)
- `args` or `eventArgs` — handler arguments (form data, selected item, etc.)

---

## Distillation algorithm

### Step 1: Group by traceId

Group all events by their `traceId`. Sort groups by the earliest `perfTs` in each group.

### Step 2: Classify each group

For each trace group:

**Startup group** (traceId starts with `startup-`):
- Extract all `api:complete` events → list of `{ method, endpoint }`
- Extract `app:trace` events → group by label
- Output as: `{ action: "startup", await: { api: [...] } }`

**Interaction group** (contains an `interaction` event):
- The interaction event defines the step's `action` and `target`
- Skip interactions on the Inspector itself (componentLabel = "XMLUI Inspector" or componentType = "XSInspector")

**No interaction, no startup prefix** → skip (internal framework events)

### Step 3: Extract each interaction step

From the interaction event, build:

```json
{
  "action": "<click|keydown|dblclick|contextmenu>",
  "target": {
    "component": "<componentType or componentLabel>",
    "ariaRole": "<ariaRole>",
    "ariaName": "<ariaName>",
    "label": "<human-readable label>"
  }
}
```

Then scan the other events in the same trace group for:

- **API calls**: `api:complete` events → add `await.api` array of `{ method, endpoint }`
- **Navigation**: `navigate` event → add `await.navigate: { from, to }`
- **Value changes**: `value:change` events → add `valueChanges` array of `{ component, value, ariaName }`
- **Toasts**: `toast` events → add `toasts` array of `{ type, message }`
- **Form data**: from `handler:start` with eventName "submit", or from `api:start` with POST/PUT/PATCH body → add to `target.formData`
- **App traces**: `app:trace` events → add `appTraces` grouped by label

### Step 4: Post-process

**Collapse consecutive textbox keydowns into fill steps:**
If you see multiple consecutive keydown steps on the same textbox (same ariaName or componentId), collapse them into a single step:
```json
{
  "action": "fill",
  "target": { "ariaRole": "textbox", "ariaName": "..." },
  "fillValue": "<final value from the last value:change on that textbox>"
}
```

**Deduplicate double-clicks:**
If you see click + click + dblclick on the same target in sequence, keep only the dblclick.

---

## Output format

Present the distilled trace as a numbered list of steps. For each step, describe:

1. **What the user did** — the action and target (e.g. "Clicked the 'Santa Rosa Now' button")
2. **What happened** — API calls made, data loaded, values changed, toasts shown
3. **Notable details** — form data submitted, navigation, errors

### Example output

For a weather app trace, the output might look like:

> **Step 1: Startup**
> - API call: GET /api/weather?city=santa-rosa → returned current conditions
> - App loaded weather data for Santa Rosa
>
> **Step 2: Clicked the "San Francisco" button**
> - API call: GET /api/weather?city=san-francisco → returned current conditions
> - Value changed: city selector updated to "San Francisco"
> - Weather display refreshed with San Francisco data

---

## Important notes

- Focus on the **user journey** — what did the user do and what was the result?
- Ignore `component:vars:init` events (framework internals)
- Ignore events with traceId `unknown` (orphaned framework events)
- For API endpoints, show the path portion — strip long query parameters unless they're meaningful
- If API results are present, summarize them briefly (row counts, key fields) rather than dumping raw data
- If you see errors (failed API calls, validation errors, error toasts), highlight them prominently
