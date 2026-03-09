# Agent Guide — Frontmatter Reference, Description Patterns & Prompt Template

## Frontmatter Reference

All available frontmatter fields:

| Field | Required | Type | Default | Notes |
|-------|----------|------|---------|-------|
| `name` | Yes | string | — | Unique ID, lowercase + hyphens |
| `description` | Yes | string | — | CRITICAL: action-oriented, when to invoke |
| `tools` | No | string/string[] | all inherited | Allowlist. Supports `Agent(type1, type2)` syntax |
| `disallowedTools` | No | string/string[] | — | Denylist, removed from inherited set |
| `model` | No | string | `inherit` | `sonnet`, `opus`, `haiku`, `inherit` |
| `permissionMode` | No | string | `default` | `default`, `acceptEdits`, `dontAsk`, `bypassPermissions`, `plan` |
| `maxTurns` | No | number | — | Max agentic turns |
| `skills` | No | string[] | — | Skill file paths preloaded into context |
| `mcpServers` | No | object/string[] | — | MCP servers by name ref or inline config |
| `hooks` | No | object | — | Lifecycle: PreToolUse, PostToolUse, Stop |
| `memory` | No | string | — | `user`, `project`, or `local` |
| `background` | No | boolean | false | Always run as background task |
| `isolation` | No | string | — | `worktree` for isolated git worktree |
| `color` | No | string | — | Agent label color: red, blue, green, yellow, purple, orange, pink, cyan |

### Key rules:
- Only `name` and `description` are required
- Subagents **cannot** spawn other subagents
- `Agent(type)` in tools restricts which subagents can be spawned (only for main-thread agents via `claude --agent`)
- Hook events: PreToolUse, PostToolUse, Stop (Stop becomes SubagentStop at runtime)
- Memory scopes: user (`~/.claude/agent-memory/<name>/`), project (`.claude/agent-memory/<name>/`), local (`.claude/agent-memory-local/<name>/`)

### Hook format:
```yaml
hooks:
  PreToolUse:
    - matcher: "Write"
      command: "npm run lint --fix $FILE"
  PostToolUse:
    - matcher: "Edit"
      command: "prettier --write $FILE"
  Stop:
    - command: "echo 'Agent finished'"
```

---

## Description Writing Guide

The `description` field is the most important field after `name`. It controls when Claude auto-invokes the agent.

### Patterns for proactive invocation:
- Include `PROACTIVELY` or `ALWAYS` to trigger auto-invocation
- Be specific about the task type and context

### Good examples:
- `"Use PROACTIVELY to run tests and fix failures after code changes"`
- `"Expert code reviewer. Use ALWAYS after implementing features to catch bugs and suggest improvements."`
- `"Use ALWAYS for security audits on authentication-related code changes"`
- `"Use PROACTIVELY when user asks about database schema or migration questions"`

### Bad examples:
- `"A helpful assistant"` — too vague, never auto-invoked
- `"Does code stuff"` — not action-oriented
- `"Reviews code"` — missing trigger context

---

## System Prompt Template

Use this structure for the system prompt body (below the frontmatter `---`):

```markdown
<role>
You are a [specific role] specializing in [domain].
Your expertise includes [key skills].
You are [behavioral traits — e.g. "critical and thorough", "concise and fast"].
</role>

<responsibilities>
1. [Primary responsibility — be specific about HOW]
2. [Secondary responsibility]
3. [Additional responsibilities as needed]
</responsibilities>

<workflow>
1. [Step 1 — specific action with details on HOW to do it]
2. [Step 2 — include decision criteria: "If X then do Y, otherwise do Z"]
3. [Step 3 — define what outputs to produce]
</workflow>

<constraints>
- [What NOT to do — be explicit]
- [Boundaries and limitations]
- [Quality requirements]
</constraints>

<output-format>
[Define exact structure of what agent should return]
[Include example if format is non-obvious]
</output-format>
```

### Prompt writing principles:
- **Be obsessively specific** — say HOW, not just WHAT
- **Single responsibility** — one clear goal per agent
- **Use XML tags** for structure (`<role>`, `<workflow>`, `<constraints>`, etc.)
- **Include decision criteria** — when to do X vs Y
- **Define handoff rules** — how to return results to the user
- **Remember statelessness** — agents start fresh each invocation, include all needed context
- **Specify behavior** — explicitly state personality and communication style

---

## Agent File Locations

| Scope | Path |
|-------|------|
| Project-level | `.claude/agents/<name>.md` |
| User-level | `~/.claude/agents/<name>.md` |
| Plugin-level | `<plugin-root>/agents/<name>.md` |
