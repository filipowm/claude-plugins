# Claude Code Plugin Structure Reference

## Complete Plugin Directory Structure

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json              # Required: Plugin metadata manifest
├── skills/                       # Specialized workflows
│   └── skill-name/
│       └── SKILL.md
├── agents/                       # Domain-specific agents
│   └── agent-name.md
├── hooks/                        # Event-triggered scripts
│   ├── hooks.json
│   └── scripts/
│       ├── pre-submit.sh
│       └── post-submit.sh
├── commands/                     # Legacy commands
│   └── command-name.md
├── scripts/                      # Utility scripts
│   └── README.md
├── .mcp.json                     # MCP server configuration
├── .lsp.json                     # LSP server configuration
├── README.md                     # Documentation
└── CHANGELOG.md                  # Version history
```

## File Templates

### plugin.json

```json
{
  "name": "PLUGIN_NAME",
  "description": "PLUGIN_DESCRIPTION",
  "version": "1.0.0",
  "author": {
    "name": "AUTHOR_NAME",
    "email": "AUTHOR_EMAIL",
    "url": "AUTHOR_URL"
  },
  "homepage": "HOMEPAGE_URL",
  "repository": "REPOSITORY_URL",
  "license": "LICENSE",
  "keywords": []
}
```

Required fields: `name`, `description`, `version`. All others are optional.
Omit optional fields if user doesn't provide values (don't include empty strings).

### marketplace.json

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "MARKETPLACE_NAME",
  "owner": {
    "name": "OWNER_NAME",
    "email": "OWNER_EMAIL"
  },
  "metadata": {
    "description": "MARKETPLACE_DESCRIPTION",
    "version": "1.0.0",
    "pluginRoot": "./plugins"
  },
  "plugins": []
}
```

### Marketplace Plugin Entry

```json
{
  "name": "PLUGIN_NAME",
  "source": "./plugins/PLUGIN_NAME",
  "description": "PLUGIN_DESCRIPTION",
  "version": "1.0.0",
  "author": {
    "name": "AUTHOR_NAME"
  },
  "license": "LICENSE"
}
```

### SKILL.md

```markdown
---
description: Description of what this skill does and when to use it
---

# Skill Instructions

This is where you provide instructions to Claude on how to execute this skill.

Arguments are available via $ARGUMENTS variable.

Example: Process the user request for "$ARGUMENTS" by...
```

### Agent (agents/example-agent.md)

```markdown
---
description: Specialized agent for specific task domain
---

# Agent Instructions

You are a specialized agent for [specific purpose].

Your capabilities include:
- Capability 1
- Capability 2

When invoked, you should...
```

### hooks.json

```json
{
  "pre-submit": "./hooks/scripts/pre-submit.sh",
  "post-submit": "./hooks/scripts/post-submit.sh"
}
```

### pre-submit.sh

```bash
#!/usr/bin/env bash
# Pre-submit hook
# Exit with non-zero to block submission

PLUGIN_ROOT="${CLAUDE_PLUGIN_ROOT}"

echo "Running pre-submit checks..."
exit 0
```

### post-submit.sh

```bash
#!/usr/bin/env bash
# Post-submit hook

PLUGIN_ROOT="${CLAUDE_PLUGIN_ROOT}"

echo "Post-submit hook executed"
```

### .mcp.json

```json
{
  "mcpServers": {
    "example-server": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-example"],
      "env": {
        "PLUGIN_ROOT": "${CLAUDE_PLUGIN_ROOT}"
      }
    }
  }
}
```

### .lsp.json

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

### commands/example.md

```markdown
This is a simple command example.

Commands are static markdown files that Claude loads when invoked.

Use `/plugin-name:example` to invoke this command.

Note: Commands are legacy - prefer using skills/ for new functionality.
```

### README.md

```markdown
# PLUGIN_NAME

PLUGIN_DESCRIPTION

## Features

- Feature 1
- Feature 2

## Installation

\`\`\`bash
/plugin marketplace add MARKETPLACE_REPO
/plugin install PLUGIN_NAME@MARKETPLACE_NAME
\`\`\`

## Usage

Instructions on how to use the plugin...

## Configuration

Any configuration options...

## Development

Test locally:
\`\`\`bash
claude --plugin-dir /path/to/PLUGIN_NAME
\`\`\`
```

### CHANGELOG.md

```markdown
# Changelog

## [1.0.0] - CURRENT_DATE

### Added
- Initial release
```

### scripts/README.md

```markdown
# Utility Scripts

Place utility scripts used by the plugin here.

Scripts referenced in hooks, skills, or other components should be placed in their respective directories.
This directory is for standalone utility scripts.
```

## Plugin Metadata Fields

| Field | Required | Type | Constraints |
|-------|----------|------|-------------|
| name | Yes | string | kebab-case, max 64 chars, unique in marketplace |
| description | Yes | string | Brief, what the plugin does |
| version | Yes | string | Semantic versioning (MAJOR.MINOR.PATCH) |
| author.name | No | string | Author display name |
| author.email | No | string | Valid email |
| author.url | No | string | Valid URL |
| homepage | No | string | Valid URL |
| repository | No | string | Valid URL |
| license | No | string | SPDX identifier (e.g., MIT, Apache-2.0) |
| keywords | No | array | Array of strings for discoverability |

## Capability Details

### Skills
- Located in `skills/{skill-name}/SKILL.md`
- YAML frontmatter with `description` field
- User invokes via `/plugin-name:skill-name [arguments]`
- Arguments accessible via `$ARGUMENTS`

### Agents
- Located in `agents/{agent-name}.md`
- YAML frontmatter with `description` field
- Specialized Claude behavior for specific domains

### Hooks
- Config in `hooks/hooks.json` mapping events to scripts
- **pre-submit**: Runs before message submission, can block (non-zero exit)
- **post-submit**: Runs after message submission, cannot block
- Scripts must be executable (`chmod +x`)
- Use `${CLAUDE_PLUGIN_ROOT}` for paths

### MCP Servers
- Config in `.mcp.json` at plugin root
- Integrates external tools via Model Context Protocol
- Supports env variable interpolation (`${VAR_NAME}`)

### LSP Servers
- Config in `.lsp.json` at plugin root
- Provides code intelligence for specified file types
- Auto-activates for configured filetypes

### Commands (Legacy)
- Located in `commands/{command-name}.md`
- Static markdown loaded on demand
- Prefer skills for new functionality

## Validation Rules

- Plugin name: `/^[a-z0-9]+(-[a-z0-9]+)*$/` (kebab-case)
- Version: valid semver (`X.Y.Z`)
- Hook scripts: must be executable
- JSON files: must be valid JSON
- marketplace.json source paths: must point to existing directories
