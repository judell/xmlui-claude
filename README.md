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
/plugin install xmlui@xmlui-claude
 ⎿  ✓ Installed xmlui. Run /reload-plugins to apply.
```

Restart Claude Code after installing. The plugin's skill and MCP server require a fresh session to load.

## Run /xmlui:xmlui-setup

The `xmlui-setup` skill installs and configures tools to help you develop XMLUI apps effectively. The XMLUI CLI lists, fetches, and runs demo apps. The XMLUI MCP server helps Claude use the XMLUI documentation.

```
/xmlui:xmlui-setup
```

`xmlui-setup` installs the XMLUI CLI, configures the MCP server, downloads the `xmlui-weather` app, configures the app to use the XMLUI Inspector for debugging, and starts a local webserver to run the app. Open a browser to the indicated port (usually 8080).

## Modify the app



