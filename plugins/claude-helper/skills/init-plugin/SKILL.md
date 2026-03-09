---
name: init-plugin
description: 'Initialize a new Claude Code plugin with interactive wizard. Use when creating a new plugin or setting up a plugin marketplace.'
disable-model-invocation: true
argument-hint: '[plugin-name]'
---

You are a Claude Code Plugin building expert. You know every detail of the plugin system: structure, metadata, capabilities, configuration, and distribution.

Your task: initialize a new Claude Code plugin using an interactive wizard approach.

If plugin name is provided via arguments, use it: "$ARGUMENTS"

Read (plugin-structure.md)[plugin-structure.md] before starting. It contains all templates and structure details you need.

## Workflow

Strictly follow workflow phases. Do not skip phases.

### Phase 1: Detect Context

Check if current directory is a plugin marketplace repository:
1. Look for `.claude-plugin/marketplace.json` in the current working directory
2. If found → read it, extract `pluginRoot` path. This is an **existing marketplace**
3. If not found → proceed to Phase 1a

### Phase 1a: Environment Setup

Use AskUserQuestion to ask:
- "No marketplace detected. What would you like to do?"
  - **Set up marketplace first** - Create marketplace structure, then create plugin inside it
  - **Standalone plugin** - Create plugin in current directory without marketplace

If user chooses marketplace setup:
1. Collect marketplace info via AskUserQuestion:
   - Marketplace name (kebab-case, e.g., "my-plugins")
   - Owner name and email
   - Description
   - Plugins directory path (default: `./plugins`) - where plugins will be stored
2. Create `.claude-plugin/marketplace.json` using template from [plugin-structure.md](plugin-structure.md), setting `pluginRoot` to the chosen directory
3. Create the plugins directory
4. Continue to Phase 2 with marketplace context

### Phase 2: Collect Plugin Metadata

Use AskUserQuestion to collect (skip fields already provided via arguments):

**Required:**
- Plugin name (must be kebab-case, max 64 chars)
- Description (concise, what the plugin does)

**Optional (ask in second question):**
- Author name, email, URL
- License (default: MIT)
- Keywords (comma-separated)
- Homepage URL
- Repository URL

Validate:
- Name is kebab-case (lowercase letters, numbers, hyphens only)
- Name is unique within marketplace (if in marketplace context - check marketplace.json)
- Description is not empty

### Phase 3: Select Capabilities

Use AskUserQuestion with multiSelect to ask which capabilities to scaffold:

| Capability | Directory | Description |
|-----------|-----------|-------------|
| Skills | `skills/` | Specialized workflows and multi-step processes |
| Agents | `agents/` | Domain-specific agent personas |
| Hooks | `hooks/` | Event-triggered scripts (pre-submit, post-submit) |
| MCP Servers | `.mcp.json` | External tool integration via Model Context Protocol |
| LSP Servers | `.lsp.json` | Language server protocol for code intelligence |
| Commands | `commands/` | Legacy static markdown commands |

Present each option with its description. Default recommendation: Skills + Hooks.

### Phase 4: Generate Plugin

Determine the target directory:
- Marketplace context → `{pluginRoot}/{plugin-name}/`
- Standalone → current directory (or `./{plugin-name}/` if current dir is not empty)

Create the following structure:

**Always created:**
- `.claude-plugin/plugin.json` - Plugin manifest with collected metadata
- `README.md` - Plugin documentation with installation and usage sections
- `CHANGELOG.md` - Version history starting at 1.0.0

**Created based on Phase 3 selections:**
- Skills → `skills/{plugin-name}/SKILL.md` with template content
- Agents → `agents/example-agent.md` with template content
- Hooks → `hooks/hooks.json` + `hooks/scripts/pre-submit.sh` + `hooks/scripts/post-submit.sh` (make scripts executable with `chmod +x`)
- MCP Servers → `.mcp.json` with example server configuration
- LSP Servers → `.lsp.json` with example server configuration
- Commands → `commands/example.md` with template content

Also create:
- `scripts/` directory with a `README.md` explaining utility scripts purpose

Use templates from [plugin-structure.md](plugin-structure.md) for all generated files. Replace placeholder values with actual collected metadata.

IMPORTANT: After creating hook scripts, run `chmod +x` on them.

### Phase 5: Marketplace Registration

If in marketplace context:
1. Read current `marketplace.json`
2. Add new plugin entry to the `plugins` array:
   ```json
   {
     "name": "{plugin-name}",
     "source": "./{pluginRoot}/{plugin-name}",
     "description": "{description}",
     "version": "1.0.0",
     "author": {"name": "{author-name}"},
     "license": "{license}"
   }
   ```
3. Write updated `marketplace.json`
4. Validate JSON is well-formed

If standalone → skip this phase.

### Phase 6: Summary & Next Steps

Display a summary:

1. **Created structure** - show the directory tree of what was generated
2. **Plugin metadata** - show name, version, description
3. **Next steps** - suggest:
   - Test locally: `claude --plugin-dir /path/to/plugin`
   - Customize skills: edit `skills/` directory
   - Configure hooks: edit `hooks/scripts/`
   - Set up MCP servers: edit `.mcp.json`
   - If marketplace: commit and push to publish

IMPORTANT: Remind user that hook scripts must remain executable (`chmod +x`).
