# XMLUI Jumpstart

Get a running XMLUI app, an AI assistant that knows the XMLUI docs, and a built-in Inspector for debugging — in under 5 minutes.

## Prerequisites

[Claude Code](https://code.claude.com/docs/en/quickstart).

## Install

```
/plugin marketplace add xmlui-org/xmlui-claude
/plugin install xmlui@xmlui-claude
```

When prompted for install scope, choose the default: **Install for you (user scope)**.

## Set up

Navigate to the directory where you want your project, then:

```
/xmlui:xmlui-setup
```

This installs the XMLUI CLI, downloads the `xmlui-weather` app, and starts a dev server. Open a browser to the indicated port (usually 8080).

## Explore the MCP tools

The plugin gives Claude access to the XMLUI documentation via MCP tools. It can ask questions like:

- "How do I paginate a list or table?"
- "How do I handle errors in a DataSource?"
- "What layout components are available?"

If you're writing the XMLUI code yourself, you'll search the documentation to find the answers. The MCP server helps Claude do that.

## Use the Inspector

The weather app includes the Inspector (magnifying glass icon at top right). It records traces of everything your app does, so you and Claude can see what's going on.

### Run the app

When it loads, the app makes an API call to fetch weather for Santa Rosa, CA, and displays the data. Click the Inspector and expand the Startup phase to see what happened. This is your distilled view of a much more detailed log. Try switching to another city. Then click Export, and say to Claude: "Read the trace and evaluate what happened."

## Modify the app

You have a running app, an AI that knows about XMLUI, and way for you and the AI to observe the app's behavior. Try making changes. The layout isn't great, ask Claude to vertically center the elements. It will use the MCP tools to find the right way, and preview changes for you to approve. Approve and refresh the browser. Did it work? Great! If not, export a trace and ask Claude to fix whatever went wrong.


