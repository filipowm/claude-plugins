# Using Plugins from This Marketplace

This guide explains how to install and use plugins from the filipowm-plugins marketplace.

## Adding the Marketplace

First, add this marketplace to Claude Code:

```bash
claude
/plugin marketplace add filipowm/claude-plugins
```

This makes all plugins in this repository available for installation.

## Browsing Available Plugins

View available plugins:

```bash
/plugin
```

This shows:
- **Installed**: Plugins you've installed
- **Marketplaces**: Available marketplaces (including filipowm-plugins)
- Plugin details and versions

## Installing Plugins

Install a plugin from this marketplace:

```bash
/plugin install <plugin-name>@filipowm-plugins
```

**Example**:
```bash
/plugin install my-plugin@filipowm-plugins
```

**Install specific version**:
```bash
/plugin install my-plugin@1.2.0@filipowm-plugins
```

## Using Installed Plugins

Once installed, plugins extend Claude's capabilities:

### Skills

Skills appear in Claude's available skills list. Invoke them:

```bash
/plugin-name:skill-name
```

**With arguments**:
```bash
/plugin-name:skill-name argument here
```

### Agents

Agents are specialized versions of Claude for specific tasks. They're automatically available based on the plugin's configuration.

### Hooks

Hooks run automatically on configured events (pre-submit, post-submit). No manual invocation needed.

### MCP Servers

MCP servers provide additional tools to Claude. They're loaded automatically when the plugin is installed.

### LSP Servers

LSP servers provide code intelligence features. They activate automatically for configured file types.

## Managing Plugins

**List installed plugins**:
```bash
/plugin
```

**Update a plugin**:
```bash
/plugin update <plugin-name>
```

**Uninstall a plugin**:
```bash
/plugin uninstall <plugin-name>
```

**Refresh marketplace**:
```bash
/plugin marketplace refresh filipowm-plugins
```

## Plugin Permissions

Some plugins may require permissions:
- File system access
- Network access
- External command execution

Claude Code will prompt for permission when needed.

## Troubleshooting

### Plugin not found

Ensure marketplace is added:
```bash
/plugin marketplace add filipowm/claude-plugins
```

### Plugin not working

1. Check if properly installed: `/plugin`
2. Try reinstalling: `/plugin uninstall <name>` then `/plugin install <name>@filipowm-plugins`
3. Check plugin documentation in its README

### Marketplace sync issues

Refresh the marketplace:
```bash
/plugin marketplace refresh filipowm-plugins
```

## Available Plugins

See the main [README](../README.md) for a list of available plugins and their descriptions.

## Local Development

To test plugins locally before they're published:

```bash
claude --plugin-dir /path/to/plugin
```

This loads the plugin directly from a local directory.

## Getting Help

- Check individual plugin READMEs for specific usage
- Review [creating plugins guide](./creating-plugins.md) for technical details
- See [plugin template documentation](./plugin-template.md) for component details
