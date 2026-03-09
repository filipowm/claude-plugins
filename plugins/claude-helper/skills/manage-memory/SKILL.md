---
name: manage-memory
description: 'Manage Claude Code memory — CLAUDE.md files, AGENTS.md, and .claude/rules/. Use when user wants to create, update, review, or optimize project memory files. Triggers: "manage memory", "create CLAUDE.md", "update rules", "init memory", "optimize CLAUDE.md".'
disable-model-invocation: true
argument-hint: '[topic-or-action]'
---

# Manage Claude Code Memory

Manages the full lifecycle of Claude Code memory: CLAUDE.md files, AGENTS.md, and `.claude/rules/`. Performs deep codebase research to generate accurate, project-specific memory files.

IMPORTANT: This skill must be highly interactive. Challenge user decisions, propose alternatives, and surface non-obvious patterns. Do NOT rubber-stamp — push back when you see weak patterns or missing conventions.

## Routing

Determine the execution path based on `$ARGUMENTS`:

- **No arguments provided** → Go to [Phase 1: Assessment](#phase-1-assessment)
- **Arguments provided** (e.g., `react`, `security`, `testing`, `architecture`) → Go to [Phase 4: Targeted Update](#phase-4-targeted-update)

---

## Phase 1: Assessment

Scan the project to determine current memory state.

### 1.1 Check existing memory files

Use Glob and Read to check for:
- `./CLAUDE.md` or `./.claude/CLAUDE.md` (project CLAUDE.md)
- `./AGENTS.md` (cross-tool agent instructions)
- `./.claude/rules/*.md` (modular rules)
- Subdirectory `CLAUDE.md` or `AGENTS.md` files (use `**/CLAUDE.md` and `**/AGENTS.md` glob)
- `./.claude/settings.json` or `./.claude/settings.local.json`

### 1.2 Determine state and route

| State | Action |
|---|---|
| No CLAUDE.md, no rules | Full generation → Phase 2 |
| CLAUDE.md exists, no rules | Propose adding rules → Phase 3 (targeted) |
| Rules exist, no CLAUDE.md | Propose creating CLAUDE.md → Phase 3 |
| Both exist | Propose improvements → Phase 5 |

### 1.3 Fetch official documentation

Before proceeding, fetch the latest official Claude Code memory documentation:
1. Fetch `https://code.claude.com/docs/en/memory.md` using WebFetch — extract the full memory system documentation
2. Fetch `https://code.claude.com/docs/en/sub-agents.md` using WebFetch — extract subagent memory patterns

Use fetched docs as authoritative reference throughout all phases. Cross-reference with best practices in [memory-best-practices.md](references/memory-best-practices.md).

Present findings to user using AskUserQuestion:
- What exists currently (with brief quality assessment if files exist)
- What's missing
- Recommended action path

**Gatekeeping**: User must approve the action path before proceeding.

---

## Phase 2: Strategy Decision

### 2.1 AGENTS.md vs CLAUDE.md

Use AskUserQuestion to ask the user (provide sufficient context):

**Question**: "Should this project use AGENTS.md alongside CLAUDE.md?"

Present these options with clear trade-offs:
- **CLAUDE.md only** — Simpler. Best when team uses only Claude Code. All instructions in one system.
- **CLAUDE.md + AGENTS.md** — Cross-tool compatibility. Universal instructions go in AGENTS.md, Claude-specific extensions (skills, subagents, MCP, rules references) go in CLAUDE.md. Every CLAUDE.md MUST contain `@AGENTS.md` as first line.

IMPORTANT: If AGENTS.md approach is chosen:
- Every directory that gets a CLAUDE.md MUST also get an AGENTS.md
- Every CLAUDE.md MUST start with `@AGENTS.md`
- AGENTS.md contains vendor-neutral instructions only (no `@imports`, no `.claude/` references)
- CLAUDE.md contains Claude-specific extensions below the `@AGENTS.md` import

### 2.2 Scope determination

Use AskUserQuestion to determine:
- Which subdirectories should get their own CLAUDE.md? (Propose candidates based on project structure). It can be any number of subdirectories
- What rule categories are needed? (e.g., code-style, testing, security, architecture)
- Are there compliance/regulatory requirements? (HIPAA, PCI-DSS, GDPR)

Challenge the user's choices:
- If they want CLAUDE.md in every directory: "That's likely overkill. Only directories with distinct conventions need their own file. Which directories have genuinely different patterns?"
- If they skip security rules: "Are you sure? Even simple projects benefit from basic security guidance."
- If they want everything in root CLAUDE.md: "That risks exceeding the 200-line sweet spot. Consider splitting domain-specific content into .claude/rules/ with path filters and subdirectory-specific CLAUDE.md."

**Gatekeeping**: Confirm final strategy with user before spawning research agents.

---

## Phase 3: Codebase Research

### 3.1 Spawn research team

IMPORTANT: Prefer using TeamCreate for parallel research. If teams are unavailable or fail, fall back to sequential Agent tool calls.

**Team approach** (preferred):

Create a team with TeamCreate. Then create tasks and spawn agents for parallel research:

1. **`stack-analyst`** — Discovers tech stack, frameworks, languages, package managers, build tools. Reads package.json, Cargo.toml, go.mod, pom.xml, requirements.txt, etc. Reports exact versions.
   - Agent type: `Explore`

2. **`architecture-mapper`** — Maps project structure, identifies key directories, module boundaries, entry points, data flow patterns. Produces an architecture map.
   - Agent type: `Explore`

3. **`conventions-researcher`** — Analyzes existing code for naming conventions, file organization, import patterns, error handling patterns, logging patterns. Identifies both followed AND violated conventions.
   - Agent type: `Explore`

4. **`commands-researcher`** — Discovers build commands, test commands, lint commands, deploy commands from package.json scripts, Makefile, CI configs, README. Verifies commands actually work.
   - Agent type: `Explore`

After initial research completes, review findings and dynamically spawn additional domain experts based on discoveries:

| Discovery | Spawn Agent | Focus |
|---|---|---|
| React/Vue/Svelte | `frontend-expert` | Component patterns, state management, styling approach |
| Java/Kotlin/Spring | `java-expert` | Spring conventions, Maven/Gradle patterns, package structure |
| Python/Django/FastAPI | `python-expert` | Python conventions, framework patterns, virtual env |
| Go | `go-expert` | Go conventions, module structure, error handling |
| Rust | `rust-expert` | Cargo workspace, trait patterns, error handling |
| Database/ORM | `data-expert` | Schema patterns, migration conventions, query patterns |
| API routes | `api-expert` | Route patterns, auth, validation, error handling |
| Tests found | `testing-expert` | Test framework, patterns, fixtures, coverage |
| CI/CD configs | `devops-expert` | Pipeline patterns, deployment, environment management |
| Monorepo structure | `monorepo-expert` | Package boundaries, shared code, workspace patterns |

Consider other domain experts based on findings, but follow patterns above (e.g. `product-manager`, `dotnet-expert` etc..).

Each spawned agent should:
- Use `subagent_type: "Explore"` or `subagent_type: "general-purpose"` as appropriate
- Be given clear instructions on what to research
- Report findings in a structured format
- The team lead (you) coordinates, assigns tasks, and aggregates findings

**Subagent fallback** (if teams unavailable):

Spawn sequential Agent calls with `subagent_type: "Explore"`:
1. First agent: stack + architecture + commands
2. Second agent: conventions + patterns + testing
3. Third agent: domain-specific deep dive based on findings

### 3.2 Aggregate and analyze findings

After all research completes:
1. Aggregate all findings into a coherent picture
2. Identify conflicts or inconsistencies in findings
3. Determine what goes where using the placement decision framework from [memory-best-practices.md](references/memory-best-practices.md).
4. Identify which subdirectories need their own CLAUDE.md

Present research summary to user using AskUserQuestion:
- Key discoveries (stack, patterns, conventions)
- Proposed memory file structure
- Any surprising findings or inconsistencies

Challenge the user:
- "I found you're using both X and Y for the same purpose. Which is the canonical approach?"
- "Your code follows pattern A in most places but pattern B in [directory]. Is this intentional?"
- "I didn't find any testing patterns. Should I create testing rules anyway?"

**Gatekeeping**: User must approve the proposed structure before content generation.

---

## Phase 4: Content Generation

### 4.1 Generate root CLAUDE.md

Follow the template structure from [claudemd-templates.md](references/claudemd-templates.md).

CRITICAL constraints:
- Target under 150 lines (hard max 200)
- Use imperative form ("Use...", "Run...", "Never...")
- Pin exact versions (not "latest")
- Include only what Claude can't infer from code
- Reserve IMPORTANT/NEVER/CRITICAL for 2-3 truly critical rules
- Include architecture map (annotated tree, key dirs only)
- Include verification commands
- Reference `.claude/rules/` and subdirectory docs for depth

If AGENTS.md strategy was chosen:
- Generate AGENTS.md with vendor-neutral content
- Generate CLAUDE.md with `@AGENTS.md` import + Claude-specific extensions

### 4.2 Generate .claude/rules/ files

Create rule files based on research findings. Use path-scoped rules (`paths:` frontmatter) where applicable.

Common rule files to consider:
- `code-style.md` — naming, formatting, import patterns (if not handled by linter)
- `architecture.md` — layer dependencies, module boundaries, data flow (global, no paths)
- `testing.md` — test patterns, mocking, fixtures (scoped to `**/*.test.*`)
- `security.md` — security patterns, input validation, secrets handling (global)
- Domain-specific rules based on research discoveries

Each rule file: target under 50 lines. Move detailed content to separate files.

### 4.3 Generate subdirectory CLAUDE.md files

Only for directories that have genuinely distinct conventions. Target 30-80 lines each.

### 4.4 Targeted update mode

If `$ARGUMENTS` was provided (e.g., `react`, `security`, `testing`):
1. Read existing CLAUDE.md and relevant rules
2. Spawn a focused research agent for the specific topic
3. Generate or update the specific rule file
4. Update root CLAUDE.md references if needed
5. Present changes for review

---

## Phase 5: Review & Challenge

IMPORTANT: This phase is mandatory. Never skip it.

### 5.1 Present all generated content

Show user:
- Brief summary and layout of generated memory -- what goes where, why.
- Complete root CLAUDE.md content
- Each rule file content
- Each subdirectory CLAUDE.md content
- File structure overview

### 5.2 Quality review

Review generated content against best practices:
- Is root CLAUDE.md under 200 lines?
- Are instructions imperative, not descriptive?
- Are there contradictions across files?
- Are prohibitions paired with alternatives?
- Are versions pinned exactly?
- Is emphasis used sparingly (2-3 critical rules max)?
- Are path patterns correct in rule frontmatter?
- Is the architecture map accurate but concise?

### 5.3 Challenge and improve

Proactively identify weaknesses:
- "This CLAUDE.md is X lines — consider moving [section] to a rule file"
- "You have no security rules — even basic projects benefit from input validation guidance"
- "These two rules contradict each other: [rule A] vs [rule B]"
- "This instruction is too vague: [instruction]. Consider: [specific alternative]"

Use AskUserQuestion to present findings and proposed improvements.

### 5.4 Iterate

Repeat review cycle until user approves all content.

**Gatekeeping**: User MUST explicitly approve before saving.

---

## Phase 6: Save & Validate

### 6.1 Write files

Save all approved content to their target locations:
- Root CLAUDE.md → `./CLAUDE.md` (or `./.claude/CLAUDE.md`)
- AGENTS.md → `./AGENTS.md` (if chosen)
- Rules → `./.claude/rules/*.md`
- Subdirectory files → `./[subdir]/CLAUDE.md` (or `./[subdir]/AGENTS.md` if chosen)

### 6.2 Validate

After saving:
1. Verify all files exist and are readable
2. Check `@` imports resolve correctly
3. Verify path patterns in rule frontmatter match actual file paths
4. Count total lines across all always-loaded files (should be under 250)
5. If AGENTS.md approach: verify every CLAUDE.md has `@AGENTS.md` reference

### 6.3 Report

Present final summary:
- Files created/modified (with line counts)
- Total always-loaded context size
- Recommendations for next steps (e.g., "consider adding hooks for X", "review auto-memory in 10 sessions")

---

## Improvement Mode (Both CLAUDE.md and Rules Exist)

When existing memory is detected, analyze and propose specific improvements:

1. Read all existing memory files
2. Check against best practices from [memory-best-practices.md](references/memory-best-practices.md)
3. Run a focused codebase scan to detect drift (conventions in code that aren't in memory)
4. Propose specific changes grouped by priority:
   - **Critical**: contradictions, outdated versions, missing critical rules
   - **Important**: missing conventions detected in code, oversized files
   - **Nice-to-have**: reorganization, path-scoping opportunities, progressive disclosure improvements

Use AskUserQuestion to present each group and let user cherry-pick which to apply.

---

## Reference Material

For detailed guidance, read these files from the skill directory:
- [memory-best-practices.md](references/memory-best-practices.md) — comprehensive best practices guide
- [claudemd-templates.md](references/claudemd-templates.md) — CLAUDE.md templates by project type
- [rules-templates.md](references/rules-templates.md) — example rule files by domain