# XMLUI Quickstart

Get a running XMLUI app, an AI assistant that knows the XMLUI docs, and a built-in Inspector for debugging — in under 5 minutes.

## Prerequisites

[Claude Code](https://code.claude.com/docs/en/quickstart)

## Add a marketplace

```
/plugin marketplace add xmlui-org/xmlui-claude
```

## Install the XMLUI plugin

```
/plugin install xmlui@xmlui-claude
```

When prompted for install scope, choose the default: **Install for you (user scope)**.

**IMPORTANT! Quit Claude Code and restart so it can load the plugin.**

## Set up

Navigate to the directory where you want your project, then:

```
/xmlui-setup
```

This installs the XMLUI CLI, downloads the `xmlui-weather` app, starts a dev server, and loads the app.

## Explore the MCP tools

The plugin gives Claude access to the XMLUI documentation via MCP tools. 

Ask Claude: "What XMLUI MCP tools are available to you?"

It can use these tools to ask questions like:

- How do I paginate a list or table?
- How do I handle errors in a DataSource?
- What layout components are available?

If you're writing the XMLUI code yourself, you'll search the documentation to find the answers. The MCP server helps Claude do that.

## Use the Inspector

The weather app includes the Inspector (magnifying glass icon at top right). It records traces of everything your app does, so you and Claude can see what's going on.

### Run the app

When it loads, the app makes an API call to fetch weather for Santa Rosa, CA, and displays the data. Click the Inspector and expand the Startup phase to see what happened. This is your distilled view of a much more detailed log.

Try switching to another city. Then click Export, and say to Claude: "distill and analyze the trace".

## Modify the layout

You have a running app, an AI that knows about XMLUI, and way for you and the AI to observe the app's behavior. Try making changes. The layout isn't great, ask Claude to "center the input box and button as group, and center the radio group on a new row". Expect Claude to use the MCP tools to find an answer based on a documented how-to example that provably works. Don't be afraid to challenge Claude to prove its answer and cite evidence. If Claude *can't* find a known working example, it should admit that and cite the best documentation it can find.

When Claude previews a plausible answer, approve it and refresh the browser. Did it work? Great! If not, capture a screenshot of the botched layout, paste into Claude, and tell it to look harder for an evidence-based solution.

## Add a feature

The `xmlui-weather` app includes the Apache ECharts extension. Ask Claude to add a table that reports hourly temperatures for three cities. If things don't work the first time, export a trace so Claude can analyze what went wrong and find a working solution.







