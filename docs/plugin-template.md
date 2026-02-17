# Plugin Template Documentation

The `template/` directory provides a complete, ready-to-use plugin structure demonstrating all Claude Code plugin capabilities.

## Template Structure

```
template/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── commands/
│   └── example.md           # Legacy command example
├── skills/
│   └── example-skill/
│       └── SKILL.md         # Skill definition
├── agents/
│   └── example-agent.md     # Specialized agent
├── hooks/
│   ├── hooks.json          # Hook configuration
│   └── scripts/
│       ├── pre-submit.sh   # Pre-submit hook
│       └── post-submit.sh  # Post-submit hook
├── scripts/
│   └── README.md           # Scripts directory
├── .mcp.json               # MCP server config
├── .lsp.json               # LSP server config
├── README.md               # Plugin documentation
└── CHANGELOG.md            # Version history
```

## Component Details

### Plugin Manifest (`.claude-plugin/plugin.json`)

**Required fields**:
```json
{
  "name": "plugin-name",           // Kebab-case, unique
  "description": "What it does",   // Brief description
  "version": "1.0.0"               // Semantic version
}
```

**Optional fields**:
```json
{
  "author": {
    "name": "Author Name",
    "email": "email@example.com",
    "url": "https://github.com/author"
  },
  "homepage": "https://...",
  "repository": "https://...",
  "license": "MIT",
  "keywords": ["tag1", "tag2"]
}
```

### Commands (`commands/`)

**Legacy feature** - prefer skills for new development.

Commands are static markdown files that Claude loads when invoked:

```markdown
This is command content.

Invoked via /plugin-name:command-name
```

**Use cases**:
- Simple text templates
- Static instructions
- Quick reference content

### Skills (`skills/`)

Skills extend Claude's capabilities with specialized workflows.

**Directory structure**:
```
skills/
└── skill-name/
    └── SKILL.md
```

**SKILL.md format**:
```markdown
---
description: When to use this skill
---

# Instructions

Detailed instructions for Claude.

Access user arguments: $ARGUMENTS
```

**Use cases**:
- Multi-step workflows
- Specialized processing
- Domain-specific tasks
- Template generation

**Example - Code Review Skill**:
```markdown
---
description: Perform comprehensive code review
---

# Code Review Instructions

Review the code for "$ARGUMENTS":

1. Check for security vulnerabilities
2. Assess code quality and patterns
3. Suggest improvements
4. Verify best practices

Provide structured feedback with examples.
```

### Agents (`agents/`)

Agents are specialized versions of Claude for specific domains.

**Format**:
```markdown
---
description: Agent purpose and capabilities
---

# Agent Instructions

You are a specialized agent for [domain].

Your role:
- Responsibility 1
- Responsibility 2

When given a task...
```

**Use cases**:
- Domain expertise (data analysis, frontend, testing)
- Specific workflows
- Contextual behavior changes

**Example - Testing Agent**:
```markdown
---
description: Specialized for writing and running tests
---

# Testing Agent

You are a testing specialist.

Focus on:
- Writing comprehensive test cases
- Identifying edge cases
- Ensuring test coverage
- Debugging test failures

When asked to test code, create unit tests covering...
```

### Hooks (`hooks/`)

Hooks execute scripts in response to events.

**Configuration (`hooks.json`)**:
```json
{
  "pre-submit": "./hooks/scripts/pre-submit.sh",
  "post-submit": "./hooks/scripts/post-submit.sh"
}
```

**Hook script structure**:
```bash
#!/usr/bin/env bash
# Description of what this hook does

# Access plugin root
PLUGIN_ROOT="${CLAUDE_PLUGIN_ROOT}"

# Your logic here
echo "Hook executing..."

# For pre-submit: exit non-zero to block
exit 0
```

**Available events**:
- **pre-submit**: Before user message submission
  - Can block submission (exit non-zero)
  - Use for validation, checks, preprocessing
- **post-submit**: After message submission
  - Cannot block
  - Use for logging, notifications, cleanup

**Use cases**:
- Lint/format checking (pre-submit)
- Logging/analytics (post-submit)
- Validation (pre-submit)
- Notifications (post-submit)

**Important**:
- Make scripts executable: `chmod +x hooks/scripts/*.sh`
- Use `${CLAUDE_PLUGIN_ROOT}` for paths
- Test thoroughly before deployment

### MCP Servers (`.mcp.json`)

Model Context Protocol servers provide external tool integration.

**Configuration**:
```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-package"],
      "env": {
        "PLUGIN_ROOT": "${CLAUDE_PLUGIN_ROOT}",
        "API_KEY": "${ENV_VAR_NAME}"
      }
    }
  }
}
```

**Common patterns**:

**Filesystem access**:
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"]
    }
  }
}
```

**Database access**:
```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

**Custom server**:
```json
{
  "mcpServers": {
    "custom": {
      "command": "python",
      "args": ["${CLAUDE_PLUGIN_ROOT}/server.py"]
    }
  }
}
```

### LSP Servers (`.lsp.json`)

Language Server Protocol integration for code intelligence.

**Configuration**:
```json
{
  "lspServers": {
    "language-name": {
      "command": "language-server-binary",
      "args": ["--stdio"],
      "filetypes": ["extension1", "extension2"]
    }
  }
}
```

**Examples**:

**Python (Pyright)**:
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

**TypeScript**:
```json
{
  "lspServers": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "filetypes": ["javascript", "typescript", "javascriptreact", "typescriptreact"]
    }
  }
}
```

## Using the Template

### Creating a New Plugin

1. **Copy template**:
   ```bash
   cp -r template plugins/my-plugin
   ```

2. **Customize plugin.json**:
   ```bash
   cd plugins/my-plugin
   # Edit .claude-plugin/plugin.json
   ```

3. **Remove unused components**:
   ```bash
   # If not using hooks:
   rm -rf hooks/

   # If not using MCP:
   rm .mcp.json

   # If not using LSP:
   rm .lsp.json
   ```

4. **Build your components**:
   - Add skills to `skills/`
   - Add agents to `agents/`
   - Configure remaining components

5. **Update documentation**:
   - Edit README.md
   - Update CHANGELOG.md

### Testing

**Local testing**:
```bash
claude --plugin-dir ./plugins/my-plugin
```

**Test specific components**:
- Skills: Invoke via `/my-plugin:skill-name`
- Hooks: Submit messages to trigger
- MCP: Verify tools are available
- LSP: Open relevant file types

## Best Practices

1. **Start minimal**: Only include components you need
2. **Document everything**: Clear READMEs and comments
3. **Version properly**: Update CHANGELOG.md with each release
4. **Test thoroughly**: Local testing before marketplace addition
5. **Use environment variables**: Never hardcode secrets
6. **Make scripts executable**: `chmod +x` for all hook scripts
7. **Follow conventions**: Kebab-case names, semantic versioning

## Common Patterns

### Skill with MCP Server

Combine a skill for user interaction with MCP server for tool access:

**Skill** (instructions): How to use the tools
**MCP** (tools): What tools are available

### Agent with Hooks

Agent for specialized behavior + hooks for workflow automation:

**Agent**: Domain expertise
**Hooks**: Automated checks/actions

### Multiple Skills

Organize related functionality:

```
skills/
├── analyze/
│   └── SKILL.md
├── generate/
│   └── SKILL.md
└── optimize/
    └── SKILL.md
```

## Getting Help

- Review example components in `template/`
- Check [creating plugins guide](./creating-plugins.md)
- See [using plugins guide](./using-plugins.md)
