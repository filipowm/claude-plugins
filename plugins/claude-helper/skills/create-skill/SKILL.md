---
name: create-skill
description: 'Create a new Claude Code skill with best practices. Use when user wants to create a skill, custom command, slash command, or extend Claude capabilities.'
argument-hint: [skill-name] [brief-description]
disable-model-invocation: true
---

# Create Custom Claude Code Skill

You are creating a new Claude Code skill. Follow all phases sequentially — do not skip any phase.

For detailed examples, templates, best practices, and validation criteria, refer to [skill-guide.md](skill-guide.md).

---

## Phase 1: Gather Requirements

Creating skill: **$1**
Purpose: **$2**

If no arguments provided, use AskUserQuestion to ask:
1. Skill name (lowercase with hyphens, e.g., "code-review", "deploy-staging")
2. Brief description of what the skill should do

Then use AskUserQuestion to determine:

**Skill type:**
- **Reference/Knowledge** — "how to" guidance, conventions, standards. May have NO workflow/phases at all — just reference material.
- **Task/Workflow** — performs a specific task with defined steps (can be 1 step or 20+ phases). Includes generator/builder skills that produce structured sets of output files (specs, scaffolds, documentation suites).
- **Background Context** — always-on project knowledge, auto-loaded, hidden from `/` menu. No workflow needed.

**Scope:**
- **Project-level** — `.claude/skills/` in current project
- **User-level** — `~/.claude/skills/` for all projects
- **Plugin-level** — `<plugin-root>/skills/` inside a plugin repository

If Plugin-level scope is selected, run the following detection logic:

1. Check CWD for `.claude-plugin/marketplace.json`
   - If found → read it, extract `pluginRoot`, list plugin subdirectories under `pluginRoot`
   - Use AskUserQuestion to ask which plugin to target
   - Read selected plugin's `.claude-plugin/plugin.json` to get plugin name
2. If no marketplace found → search CWD and parent directories for `.claude-plugin/plugin.json`
   - If found → that directory is the plugin root, read `plugin.json` for plugin name
3. If neither found → use AskUserQuestion to ask for the plugin root path
4. Confirm detected plugin name and root path with user

**Gatekeeping**: Confirm requirements summary with user before proceeding.

---

## Phase 2: Define Engagement Style

Skip this phase for knowledge/reference and background context skills (they don't interact).

For task/workflow skills, have a conversation with the user about HOW the skill should engage. Use AskUserQuestion to explore:

1. **Challenge level** — Should the skill challenge user decisions, propose alternatives, and push back? Or take instructions at face value and just execute?
2. **Exploration depth** — Should the skill dig deep into requirements, ask probing questions, and surface edge cases? Or move fast with reasonable defaults?
3. **Where does user judgment matter?** — Which specific steps or decisions genuinely benefit from user input vs. which can be automated?

The outcome is NOT a template level — it's an understanding of the skill's engagement personality that you'll weave into the prompt at appropriate points. Some steps may need deep exploration, others just execution. Interactiveness happens where it adds value, not at predefined gates.

See engagement design guidance in [skill-guide.md](skill-guide.md).

---

## Phase 3: Design Skill Architecture

Work through these decisions (use AskUserQuestion where user input needed):

**Supporting files:**
- Scripts (`scripts/`) — executable bash/python files
- References (`references/`) — documentation, examples, API specs
- Assets (`assets/`) — templates, config files
- For plugin-level skills, use `${CLAUDE_PLUGIN_ROOT}` to reference shared plugin resources outside the skill directory

**Invocation control:**
- **User-invocable** (`user-invocable: true`, default) — triggered via `/skill-name`
- **Background-only** (`user-invocable: false`) — Claude auto-invokes based on description
- **Manual-only** (`disable-model-invocation: true`) — only user can trigger

**Execution context:**
- **Inline** (default) — runs in main conversation
- **Forked** (`context: fork`) — runs in isolated subagent context
  - Requires `agent` field: `Explore`, `Plan`, or `general-purpose`

**Tool restrictions:**
- Read-only: `allowed-tools: Read, Grep, Glob`
- Git-only: `allowed-tools: Bash(git:*)`
- Full access: omit `allowed-tools`

**Hooks:** Determine if skill needs lifecycle hooks (pre/post tool execution).

**Domain knowledge:** Use Context7 MCP to fetch library/API documentation if the skill's domain requires it.

**Input data** (if skill consumes structured data):
- Does the skill read configuration/manifest files? (YAML, JSON, XML)
- Does it process data files? (CSV, JSON arrays)
- Should input schemas be bundled in `assets/` for validation or as templates?
- Can it use existing project files as input? (package.json, openapi.yaml, etc.)

**Output artifacts** (for generator/builder skills):
- What files does the skill produce? (markdown, YAML, JSON, CSV, XML, or a mix)
- What is the output structure? (flat, nested, convention-based)
- Where do outputs go? (CWD, configurable path, subdirectory)
- Are outputs interdependent? (cross-references, shared schemas, index files)
- Should a manifest/schema drive what gets generated?
- Should outputs be validated as a set after generation?

**Gatekeeping**: Present architecture summary to user for approval before drafting.

---

## Phase 4: Draft Skill

Create the skill files:

1. **Write SKILL.md** with:
   - Correct frontmatter (all fields from Phase 3)
   - Clear, specific description following WHAT + WHEN + trigger phrases pattern
   - Use markdown file reference (`[link text](file.md)`) for any supporting documentation / references / templates
   - Use `${CLAUDE_SKILL_DIR}` for referencing bundled scripts
   - For plugin-level skills, save to `<plugin-root>/skills/<skill-name>/SKILL.md`
   - Apply engagement style from Phase 2 at points where it adds value

2. **Adapt structure to skill type:**
   - **Knowledge/reference**: Write clear reference material. No workflow needed — just well-organized guidance.
   - **Task/workflow**: Define steps/phases (can be 1 or many). For each step, include decision criteria — when to do X vs Y. Place user interaction where user judgment genuinely matters.
   - **Generator/builder**: Define output file structure explicitly — list every file with its format and purpose. Bundle input schemas/templates in `assets/`. Consider schema-driven generation (a manifest file controls what gets produced). Include scaffold → fill → cross-validate phases. Support both structured data (YAML, JSON, CSV) and prose (markdown) outputs as appropriate.
   - **Background context**: Write conventions and patterns. No workflow.

3. **Create supporting files** as determined in Phase 3

4. **Follow constraints:**
   - SKILL.md under 500 lines
   - Move detailed content to `references/`
   - Use progressive disclosure pattern
   - Description in third person ("Analyzes...", "Creates...", "Deploys...")

---

## Phase 5: Review & Refine

MANDATORY: Use AskUserQuestion to show user the complete draft:
1. Full SKILL.md content
2. Supporting file list and key contents
3. File structure overview

Review against validation criteria in [skill-guide.md](skill-guide.md).

Apply prompt engineering best practices:
- Imperative form for instructions
- Emphasis for critical instructions (IMPORTANT, CRITICAL)
- Specific over generic
- Structured output formats

Iterate based on user feedback. Only proceed after user approves.

---

## Phase 6: Save & Test Guidance

1. **Save files** to chosen location:
   - **Project-level:** `.claude/skills/<skill-name>/SKILL.md`
   - **User-level:** `~/.claude/skills/<skill-name>/SKILL.md`
   - **Plugin-level:** `<plugin-root>/skills/<skill-name>/SKILL.md`
2. **Make scripts executable** if any in `scripts/`
3. **Test scripts** by running them if applicable

Remind user to:
- Test with `/help` to verify skill appears
- Run the skill to verify behavior
- For `context: fork` skills: verify subagent execution
- For plugin-level skills: test with `claude --plugin-dir <plugin-root>` and invoke as `/plugin-name:skill-name`
- Iterate on prompt based on results
- Consider version control for team sharing
