# Creating Plugins for Claude Code

This guide walks through creating plugins for this marketplace.

## Quick Start

1. **Copy the template**:
   ```bash
   cp -r template plugins/my-plugin-name
   cd plugins/my-plugin-name
   ```

2. **Configure plugin metadata**:
   Edit `.claude-plugin/plugin.json`:
   ```json
   {
     "name": "my-plugin-name",
     "description": "What your plugin does",
     "version": "1.0.0",
     "author": {
       "name": "Your Name",
       "email": "your@email.com"
     },
     "keywords": ["relevant", "keywords"]
   }
   ```

3. **Build your plugin**:
   - Add skills to `skills/` directory
   - Add agents to `agents/` directory
   - Configure hooks in `hooks/`
   - Set up MCP servers in `.mcp.json`
   - Set up LSP servers in `.lsp.json`

4. **Add to marketplace**:
   Edit `/.claude-plugin/marketplace.json`:
   ```json
   {
     "plugins": [
       {
         "name": "my-plugin-name",
         "source": "./plugins/my-plugin-name",
         "description": "What your plugin does",
         "version": "1.0.0",
         "author": {"name": "Your Name"},
         "license": "MIT"
       }
     ]
   }
   ```

## Plugin Structure

A complete plugin can include:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json       # Required: Plugin metadata
├── skills/               # Optional: Skill definitions
│   └── skill-name/
│       └── SKILL.md
├── agents/               # Optional: Specialized agents
│   └── agent-name.md
├── hooks/                # Optional: Event hooks
│   ├── hooks.json
│   └── scripts/
│       └── hook.sh
├── .mcp.json            # Optional: MCP server config
├── .lsp.json            # Optional: LSP server config
└── README.md            # Recommended: Documentation
```

## Component Details

### Skills (`skills/`)

Skills extend Claude's capabilities with specialized workflows and knowledge.

**Structure**:
```
skills/
└── skill-name/
    └── SKILL.md
```

**Example SKILL.md**:
```markdown
---
description: When and how to use this skill
---

# Skill Instructions

Detailed instructions for Claude on executing this skill.

Access user arguments via $ARGUMENTS.
```

**Best practices**:
- Use clear, specific skill names (kebab-case)
- Provide detailed instructions
- Include examples of expected inputs/outputs
- Document when the skill should be used

### Agents (`agents/`)

Agents are specialized versions of Claude for specific domains.

**Example**:
```markdown
---
description: Specialized for data analysis tasks
---

# Data Analysis Agent

You are a data analysis specialist.

Your role:
- Analyze datasets
- Generate insights
- Create visualizations

When given data, you should...
```

### Hooks (`hooks/`)

Hooks execute scripts in response to events.

**hooks.json**:
```json
{
  "pre-submit": "./hooks/scripts/pre-submit.sh",
  "post-submit": "./hooks/scripts/post-submit.sh"
}
```

**Hook script**:
```bash
#!/usr/bin/env bash
# Access plugin root: $CLAUDE_PLUGIN_ROOT
# Exit non-zero to block (pre-submit only)

echo "Hook executing..."
exit 0
```

**Available events**:
- `pre-submit`: Before user message submission (can block)
- `post-submit`: After message submission

### MCP Servers (`.mcp.json`)

Model Context Protocol servers provide external tool integration.

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-package"],
      "env": {
        "PLUGIN_ROOT": "${CLAUDE_PLUGIN_ROOT}",
        "API_KEY": "${YOUR_API_KEY}"
      }
    }
  }
}
```

### LSP Servers (`.lsp.json`)

Language Server Protocol integration for code intelligence.

```json
{
  "lspServers": {
    "python": {
      "command": "pyright-langserver",
      "args": ["--stdio"],
      "filetypes": ["python"]
    }
  }
}
```

## Testing Locally

**Test single plugin**:
```bash
claude --plugin-dir ./plugins/my-plugin-name
```

**Test marketplace**:
```bash
claude
/plugin marketplace add .
/plugin install my-plugin-name@filipowm-plugins
```

## Versioning

Use semantic versioning (MAJOR.MINOR.PATCH):
- **MAJOR**: Breaking changes
- **MINOR**: New features, backward compatible
- **PATCH**: Bug fixes

Set version in marketplace.json for relative-path plugins (preferred):
```json
{
  "plugins": [
    {"name": "my-plugin", "version": "1.2.3", ...}
  ]
}
```

Or in plugin's plugin.json (for strict mode):
```json
{
  "name": "my-plugin",
  "version": "1.2.3"
}
```

## Best Practices

1. **Keep it focused**: One plugin should do one thing well
2. **Document thoroughly**: README, CHANGELOG, inline comments
3. **Test extensively**: Local testing before adding to marketplace
4. **Version properly**: Follow semantic versioning
5. **Use paths correctly**: Always use `${CLAUDE_PLUGIN_ROOT}` in configs
6. **Make scripts executable**: `chmod +x hooks/scripts/*.sh`

## Publishing Checklist

Before adding to marketplace:

- [ ] Plugin name is unique and kebab-case
- [ ] plugin.json has all required fields
- [ ] README documents usage clearly
- [ ] CHANGELOG tracks version history
- [ ] Tested locally with `--plugin-dir`
- [ ] Hook scripts are executable
- [ ] Added entry to marketplace.json
- [ ] Committed and pushed changes

## Getting Help

- Review the [template](../template/) for examples
- Check [plugin template documentation](./plugin-template.md)
- See [using plugins guide](./using-plugins.md) for user perspective
