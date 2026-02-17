# Claude Code Plugins Marketplace

A custom marketplace for Claude Code plugins by Mateusz Filipowicz.

[![Plugins](https://img.shields.io/badge/plugins-0-blue)](.claude-plugin/marketplace.json)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

## Quick Start

### Adding This Marketplace

```bash
claude
/plugin marketplace add filipowm/claude-plugins
```

### Installing Plugins

```bash
/plugin install <plugin-name>@filipowm-plugins
```

## Available Plugins

| Plugin | Version | Description |
|--------|---------|-------------|
| _No plugins yet_ | - | Check back soon! |

## Documentation

- **[Using Plugins](docs/using-plugins.md)** - How to install and use plugins from this marketplace
- **[Creating Plugins](docs/creating-plugins.md)** - Guide for creating new plugins
- **[Plugin Template](docs/plugin-template.md)** - Detailed template documentation

## Creating Your Own Plugin

This repository includes a complete plugin template demonstrating all capabilities:

1. **Copy the template**:
   ```bash
   cp -r template plugins/my-plugin-name
   ```

2. **Customize the plugin**:
   ```bash
   cd plugins/my-plugin-name
   # Edit .claude-plugin/plugin.json
   # Add your skills, agents, hooks, etc.
   ```

3. **Test locally**:
   ```bash
   claude --plugin-dir ./plugins/my-plugin-name
   ```

4. **Add to marketplace**:
   - Add entry to `.claude-plugin/marketplace.json`
   - Update this README's plugin table
   - Create a pull request

See [Creating Plugins](docs/creating-plugins.md) for detailed instructions.

## Plugin Components

The template demonstrates all available plugin capabilities:

- **Skills** - Extend Claude with specialized workflows
- **Agents** - Specialized versions of Claude for specific domains
- **Hooks** - Execute scripts on events (pre-submit, post-submit)
- **MCP Servers** - Integrate external tools via Model Context Protocol
- **LSP Servers** - Add language server protocol support
- **Commands** - Static markdown content (legacy)

## Repository Structure

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace catalog
├── plugins/                  # Individual plugins
│   └── (plugins added here)
├── template/                 # Complete plugin template
│   ├── .claude-plugin/
│   ├── skills/
│   ├── agents/
│   ├── hooks/
│   └── ...
├── docs/                     # Documentation
│   ├── creating-plugins.md
│   ├── using-plugins.md
│   └── plugin-template.md
└── README.md
```

## Local Development

Test the entire marketplace locally:

```bash
claude
/plugin marketplace add .
/plugin install <plugin-name>@filipowm-plugins
```

## Contributing

Contributions welcome! To add a plugin:

1. Fork this repository
2. Create your plugin using the template
3. Add it to `.claude-plugin/marketplace.json`
4. Update the Available Plugins table above
5. Submit a pull request

See [Creating Plugins](docs/creating-plugins.md) for guidelines.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Resources

- [Claude Code Documentation](https://docs.claude.com/claude-code)
- [Plugin Development Guide](docs/creating-plugins.md)
- [Model Context Protocol](https://modelcontextprotocol.io)