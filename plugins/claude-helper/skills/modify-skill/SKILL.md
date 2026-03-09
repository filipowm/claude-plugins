---
name: modify-skill
description: 'Modify an existing Claude Code skill. Use when user wants to edit, update, refactor, or change a skill at any scope (project, user, plugin).'
argument-hint: [skill-name] [modification-description]
disable-model-invocation: true
---

# Modify Existing Claude Code Skill

You are modifying an existing Claude Code skill. Follow all phases sequentially.

For skill structure reference, validation criteria, and best practices, refer to `../create-skill/skill-guide.md`.

---

## Phase 1: Discover Skills

If `$1` is provided, search for a skill matching that name. Otherwise, discover all available skills.

Scan these locations using Glob for `**/SKILL.md`:

1. **Project-level**: `.claude/skills/` in current working directory
2. **User-level**: `~/.claude/skills/`
3. **Plugin-level**: Check for `.claude-plugin/marketplace.json` or `.claude-plugin/plugin.json` in CWD and parents, then scan `<plugin-root>/skills/`

For each discovered skill, extract from frontmatter:
- `name`, `description`, scope (project/user/plugin), directory path

If `$1` matches exactly one skill, auto-select it. If multiple matches or no argument, use AskUserQuestion to present the list and let user choose.

---

## Phase 2: Analyze Selected Skill

Thoroughly analyze the target skill before any modification:

1. **Read SKILL.md** — parse frontmatter fields and full prompt content
2. **Inventory supporting files** — Glob for files in the skill directory:
   - `scripts/` — executable scripts
   - `references/` — documentation, examples
   - `assets/` — templates, configs
3. **Read all supporting files** — understand their content and how SKILL.md references them
4. **Identify structure** — skill type (knowledge, task/workflow, background, generator), phase count, gatekeeping points, engagement style
5. **Check for issues** — validate against criteria in `../create-skill/skill-guide.md`

Present a summary to user:
- Skill name, scope, type
- Frontmatter fields
- Supporting files list
- Current workflow phases (if any)
- Any issues or improvement opportunities found

---

## Phase 3: Determine Modifications

If `$2` (or remaining arguments after skill name) describes the modification, use that as the starting point. Otherwise, use extensively AskUserQuestion to ask what the user wants to change in multiple iterations.

Common modification types:
- **Frontmatter changes** — name, description, invocation control, tools, context/agent
- **Workflow changes** — add/remove/reorder phases, adjust gatekeeping
- **Content changes** — rewrite instructions, update output formats, add engagement points
- **Supporting files** — add/edit/remove scripts, references, assets
- **Scope migration** — move skill between project/user/plugin levels
- **Refactoring** — extract content to references, split/merge phases, improve prompt quality

For each requested change, assess risk:

**Challenge the user when:**
- Removing workflow phases or gatekeeping points
- Changing scope (may affect availability)
- Removing or renaming supporting files that SKILL.md references
- Changing invocation control (e.g., enabling auto-invocation)
- Changes that could break argument handling (`$1`, `$ARGUMENTS`, etc.)
- Significantly reducing skill content without moving to references

For risky changes, explain the impact and propose alternatives before proceeding.

---

## Phase 4: Plan Changes

Design the specific modifications:

1. List every file that will be changed, created, or deleted
2. For each file change, describe what will be modified and why
3. For frontmatter changes, show old → new values
4. For content changes, describe the structural impact
5. For new supporting files, outline their purpose and content

**IMPORTANT**: If modifications require domain knowledge (library APIs, framework patterns), use Context7 MCP to fetch current documentation.

Present the change plan to user using AskUserQuestion. Include:
- Summary of all planned changes
- Risk assessment for each change
- Recommendation if applicable

Do NOT proceed until user approves the plan.

---

## Phase 5: Apply Changes

Execute the approved modifications:

1. **Edit existing files** using the Edit tool (prefer targeted edits over full rewrites)
2. **Create new files** using Write tool if adding supporting files
3. **Delete files** only with user confirmation (use Bash `rm` after confirming)
4. **For scope migration**:
   - Copy skill directory to new location
   - Update any absolute path references
   - Confirm deletion of old location with user
5. **Make scripts executable** if new scripts were added (`chmod +x`)

---

## Phase 6: Validate & Review

After applying changes:

1. **Read the modified SKILL.md** and all changed files
2. **Validate against criteria** from `../create-skill/skill-guide.md`:
   - Frontmatter fields correct and complete
   - Description follows WHAT + WHEN + trigger phrases pattern
   - SKILL.md under 500 lines
   - File and assets references use markdown links (`[text](file.md)`)
   - Script references use `${CLAUDE_SKILL_DIR}` notation
   - Plugin-level skills use `${CLAUDE_PLUGIN_ROOT}` where appropriate
   - Workflow has review step (for task/workflow skills)
   - Critical instructions use emphasis
3. **Check for broken references** — ensure all file paths in SKILL.md point to existing files
4. **Present validation results** to user

If issues found, fix them. If clean, confirm completion.

Remind user to:
- Test with `/help` to verify skill still appears correctly
- Run the skill to verify modified behavior
- For plugin-level skills: test with `claude --plugin-dir <plugin-root>`
