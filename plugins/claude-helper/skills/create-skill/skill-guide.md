# Skill Guide - Examples, Templates & Best Practices

## SKILL.md Structure Examples

### Simple Skill (SKILL.md only)

```
.claude/skills/code-review/
└── SKILL.md
```

### Skill with scripts

```
.claude/skills/deploy/
├── SKILL.md
└── scripts/
    ├── deploy.sh
    └── rollback.sh
```

### Skill with references

```
.claude/skills/api-client/
├── SKILL.md
└── references/
    ├── api-spec.yaml
    └── examples.md
```

### Skill with assets

```
.claude/skills/component-gen/
├── SKILL.md
└── assets/
    ├── template.tsx
    └── styles.css
```

### Complex Skill (all types)

```
.claude/skills/release/
├── SKILL.md
├── scripts/
│   ├── version-bump.sh
│   └── changelog.sh
├── references/
│   └── release-process.md
└── assets/
    └── release-notes-template.md
```

### Skill inside a Plugin

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── my-skill/
        ├── SKILL.md
        └── references/
            └── guide.md
```

## Frontmatter Fields Reference

| Field                      | Description                                     | Example                                                       |
|----------------------------|-------------------------------------------------|---------------------------------------------------------------|
| `name`                     | Display name (lowercase, hyphens, max 64 chars) | `code-review`                                                 |
| `description`              | What it does + when to use (PRIMARY TRIGGER)    | `Review code for security vulnerabilities and best practices` |
| `argument-hint`            | Autocomplete hint                               | `[filename] [format]`                                         |
| `disable-model-invocation` | Prevent Claude auto-invoke                      | `true`                                                        |
| `user-invocable`           | Show in `/` menu                                | `false`                                                       |
| `allowed-tools`            | Tools allowed without permission                | `Read, Grep, Glob`                                            |
| `model`                    | Model to use                                    | `claude-opus-4-6`                                             |
| `context`                  | Run in forked subagent                          | `fork`                                                        |
| `agent`                    | Subagent type (requires context: fork)          | `Explore` or `Plan`                                           |
| `hooks`                    | Skill-scoped lifecycle hooks                    | See Hooks docs                                                |

### Invocation Control Matrix

| `disable-model-invocation` | `user-invocable` | Behavior                                                            |
|----------------------------|------------------|---------------------------------------------------------------------|
| `false` (default)          | `true` (default) | Claude can auto-invoke, user can invoke via `/`                     |
| `true`                     | `true`           | Only user can invoke via `/` (manual-only)                          |
| `false`                    | `false`          | Claude can auto-invoke, hidden from `/` menu (background knowledge) |
| `true`                     | `false`          | Skill is disabled (not useful)                                      |

## Argument Handling

### All Arguments: `$ARGUMENTS`

```markdown
Fix the issue described: $ARGUMENTS
```

Usage: `/fix-issue Login button not working on mobile`

### Positional Arguments: `$1`, `$2`, `$3`

```markdown
Review PR #$1 with priority: $2
Assign to: $3
```

Usage: `/review-pr 456 high @alice`

## Advanced Patterns

### Dynamic Context Injection

Embed live shell output into your prompt using `!`command``:

```yaml
---
name: pr-summary
description: 'Summarize GitHub pull request with comments'
context: fork
agent: Explore
---

Analyze this pull request:

PR diff:
  \!`gh pr diff`

PR comments:
  \!`gh pr view --comments`

  Provide a comprehensive summary.
```

### Subagent Execution

Run skill in isolated context with specialized subagent:

```yaml
---
name: perf-analysis
description: 'Analyze codebase for performance issues'
context: fork
agent: Explore
---

Analyze $ARGUMENTS for performance issues:

  1. Find hot paths
  2. Identify inefficient algorithms
  3. Check database queries
  4. Review memory usage

  Use thorough codebase exploration.
```

### Progressive Disclosure

Provide workflow in SKILL.md with detailed references:

```
.claude/skills/cloud-deploy/
├── SKILL.md (workflow + cloud selection)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

```markdown
Select cloud provider: $1

## Workflow

1. Read references/$1.md
2. Follow deployment steps
3. Verify deployment
```

### Background Knowledge (auto-loaded)

```yaml
---
name: project-context
description: 'Provides background knowledge about project architecture'
user-invocable: false
---

This project uses:
  - Architecture: Microservices
  - Database: PostgreSQL
  - Message Queue: RabbitMQ

  When suggesting changes, consider these patterns.
```

### Manual-only Deployment

```yaml
---
name: deploy-prod
description: 'Deploy to production (manual trigger only)'
disable-model-invocation: true
---

Deploy to production:

  1. Verify all tests pass
  2. Check staging deployment
  3. Run deployment script
  4. Monitor logs

CRITICAL: Only proceed with explicit user confirmation.
```

## Best Practices

### 1. Description (CRITICAL for discoverability)

The description is the PRIMARY trigger for Claude auto-invocation. Follow WHAT + WHEN + trigger phrases:

- **WHAT**: Action the skill performs (third person: "Analyzes...", "Creates...", "Deploys...")
- **WHEN**: Context that helps Claude match user intent
- **Trigger phrases**: Common ways users might ask for this

Keep it concise but informative. Will appear in `/help` output.

Good:
`'Analyze code for security vulnerabilities and best practices. Use when user asks to review, audit, or check code quality.'`
Good: `'Deploy to production environment. Use when user wants to release, ship, or push to prod. NEVER auto-invoke.'`
Bad: `'Security stuff'`
Bad: `'Does code review'`

Use negative triggers to prevent false matches:
`'Generate React components. Use for UI creation. NOT for styling-only changes.'`

### 2. Be Specific in Prompts

Instead of:

```markdown
Review the code
```

Write:

```markdown
Review the code in $ARGUMENTS focusing on:

1. Security vulnerabilities (SQL injection, XSS, CSRF)
2. Performance bottlenecks
3. Error handling completeness
4. Test coverage gaps

For each issue found, provide:

- Severity (Critical/High/Medium/Low)
- Location (file:line)
- Description
- Suggested fix with code example
```

### 3. Use Emphasis for Critical Instructions

```markdown
IMPORTANT: Always run tests before committing.

YOU MUST follow these steps in order:

1. First step
2. Second step

CRITICAL: Never expose API keys in responses.
```

### 4. Tool Restrictions

Only restrict when necessary:

- `allowed-tools: Read, Grep, Glob` - Read-only analysis
- `allowed-tools: Bash(git:*)` - Only git commands
- Omit for full tool access

### 5. Referencing Bundled Files with

Use markdown references to reference additional documentation and assets (only except scripts):

```markdown
For coding standards, see [Coding Standards](references/coding-standards.md)
```

Use `${CLAUDE_SKILL_DIR}` to reference scripts bundled with your skill — it resolves to the
skill's directory regardless of the user's working directory:

```markdown
Run the analysis script:
!`${CLAUDE_SKILL_DIR}/scripts/analyze.sh`
```

### 6. Structured Output Requests

```markdown
Generate output in this format:

## Summary

[1-2 sentence overview]

## Changes

- [ ] Change 1
- [ ] Change 2

## Risks

| Risk | Severity | Mitigation |
| ---- | -------- | ---------- |
```

### 7. Workflow definition and subagents

In skill, define the workflow and phases (mandatory and optional):

```
## Workflow

<workflow>
1. [Step 1 - be specific about HOW]
2. [Step 2 - include decision criteria]
3. [Step 3 - define outputs]
</workflow>
```

There can be any number of steps. Consider using existing subagents to delegate parts of workflow.
To invoke subagent, use `context: fork` with `agent` field in frontmatter, or instruct the skill to "Use Agent tool with
subagent_type=[type]". You can specify what information should be passed to subagent and expected outputs.

Valid subagent types: `Explore`, `Plan`, `general-purpose`

ALWAYS include review step in workflow, to review work done by skill. Consider delegating this to subagent if possible.
After review, workflow should support fixing.

### 8. Leverage user feedback

In skill and workflow, you can explicitly mention `AskUserQuestion` tool when user feedback should be provided. Do not
hesitate to ask user feedback/questions when flow might be beneficial from it.

### 9. Keep SKILL.md concise

- Under 500 lines for SKILL.md
- Move detailed content to `references/`
- Use scripts for complex logic
- Use assets for templates

### 10. Workflow Gatekeeping

For skills with multi-phase workflows, add explicit gatekeeping to ensure phase completion:

**Bad (no gatekeeping):**

```markdown
## Workflow

1. Analyze requirements
2. Design solution
3. Implement
4. Test
```

**Good (with gatekeeping):**

```markdown
## Workflow

Strictly follow workflow phases. Do not skip phases.

### Phase 1: Analyze requirements

[Detailed instructions]

**Gatekeeping**: Use AskUserQuestion to confirm: "I've completed analysis. Ready to proceed to design?"

### Phase 2: Design solution

[Detailed instructions]

**Gatekeeping**: Present design using AskUserQuestion and ask: "Please review this design. Approve or request changes?"

### Phase 3: Implement

[Detailed instructions]

**Gatekeeping**: Show summary of changes using AskUserQuestion: "Implementation complete. Review before testing?"

### Phase 4: Test

[Detailed instructions]
```

**When to use gatekeeping:**

- Multi-phase workflows (3+ phases)
- Workflows with user decisions
- Workflows where later phases depend on earlier phase quality
- Destructive or irreversible operations

**When to skip gatekeeping:**

- Simple linear tasks
- Fully automated workflows
- Read-only analysis

## Template Patterns

### Analysis Skill

```yaml
---
name: analyze-code
description: 'Analyze code for quality and best practices'
allowed-tools: Read, Grep, Glob
argument-hint: [ file-or-directory ]
---

Analyze $ARGUMENTS for:
  1. Code quality issues
  2. Best practice violations
  3. Potential bugs

Output format:
  ## Findings
    [ Structured findings ]

    ## Recommendations
    [ Actionable recommendations ]
```

### Manual-only Deployment Skill

```yaml
---
name: deploy-prod
description: 'Deploy to production environment (manual trigger only)'
disable-model-invocation: true
allowed-tools: Bash, Read
---

Deploy $ARGUMENTS to production:

CRITICAL:
  This is a production deployment. You MUST:

  1. Verify all tests pass
  2. Check staging deployment successful
  3. Review deployment checklist
  4. Get explicit user confirmation
  5. Run deployment script
  6. Monitor logs for errors

  Never proceed without explicit user confirmation.
```

### Background Knowledge Skill

```yaml
---
name: monorepo-context
description: 'Provides background knowledge about monorepo structure and conventions'
user-invocable: false
---

This monorepo uses:

  **Structure:**
    - `packages/`: Shared packages
    - `apps/`: Applications
    - `tools/`: Build tools

      **Conventions:**
    - Use workspace protocol for internal deps
    - Run tasks from root using turbo
    - Test with `pnpm test`

      When working with code, follow these conventions.
```

### Generator/Builder Skill — Spec Suite Generator

```yaml
---
name: spec-gen
description: 'Generate API specification suite from project analysis. Use when user wants to create specs, API docs, or specification files.'
disable-model-invocation: true
argument-hint: [ output-dir ]
---

  Generate a specification suite for this project.

Output directory:
  $1 (default: `./specs`)

## Phase 1: Scope & Discovery

Analyze the project to determine what to generate:
  1. Read package.json, openapi.yaml, or other project manifests
  2. Identify APIs, models, and integration points
  3. Read [spec-manifest-template.yaml](assets/spec-manifest-template.yaml) for output structure

Use AskUserQuestion to confirm scope: which APIs/modules to include.

## Phase 2: Scaffold Output Structure

Create the output directory structure based on the manifest:

```

specs/
├── manifest.yaml # Generated manifest listing all spec files
├── index.md # Navigation index linking all specs
├── api/
│ ├── endpoints.yaml # OpenAPI-style endpoint definitions
│ └── models.yaml # Data model schemas
├── specs/
│ ├── auth-spec.md # Feature specifications
│ └── data-spec.md
└── validation-report.md # Cross-validation results

```

## Phase 3: Generate Specs

For each item in the manifest:
1. Generate the spec file in its target format (markdown for prose, YAML for structured data)
2. Include cross-references to related specs (e.g., auth-spec.md links to endpoints.yaml)
3. Update index.md with links to each generated file

## Phase 4: Cross-Validate

1. Verify all cross-references resolve to existing files
2. Check manifest.yaml lists every generated file
3. Validate YAML files parse correctly
4. Write validation-report.md with results

**Gatekeeping**: Present validation report to user. Fix any issues before completing.
```

### Generator/Builder Skill — Structured Data Patterns

```yaml
---
name: config-gen
description: 'Generate project configuration files from templates. Use when user wants to scaffold configs, generate boilerplate, or create project structure.'
disable-model-invocation: true
argument-hint: [ config-type ]
---

Generate configuration files for: $1

## Phase 1: Analyze Input

Read input sources to determine what to generate:
  1. Check for [$1.schema.json](assets/schemas/$1.schema.json) — validation schema
  2. Check for [$1.template.yaml](assets/templates/$1.template.yaml) — base template
  3. Read existing project files for context (package.json, tsconfig.json, etc.)

  Use AskUserQuestion to gather any required parameters not inferrable from project.

## Phase 2: Generate Files

For each output file defined in the template:
  1. Start from the template in assets/
  2. Fill in project-specific values from Phase 1 analysis
  3. Validate against the JSON schema if available
  4. Write to the target location

Support these output formats as needed:
  - YAML — for configuration files
  - JSON — for schemas and package configs
  - Markdown — for documentation alongside configs
  - CSV — for matrix/compatibility data

  ## Phase 3: Validate & Link

  1. Validate all generated files against their schemas
  2. Ensure cross-references between files are consistent
  3. Generate an index/summary of what was created

**Gatekeeping** : Show user the list of generated files with a summary of each.
```

### Structured Data in Skills

Skills can consume and produce structured data beyond markdown. Key patterns:

**As input** — structured data that drives generation:

- Manifest/config files (YAML, JSON) that control what the skill produces
- CSV data files to process or transform
- JSON schemas to validate output against
- Existing project files (package.json, openapi.yaml) as context

**As output** — structured files the skill generates:

- YAML configs, OpenAPI specs, Docker Compose files
- JSON schemas, package manifests, data files
- CSV reports, compatibility matrices
- XML documents, HTML reports
- Mixed output: markdown prose alongside structured data files

**As templates** — bundled in `assets/` for the skill to fill in:

- YAML/JSON templates with placeholder values
- Schema files for validating generated output
- Starter manifests the skill copies and customizes

**Cross-referencing** — when outputs reference each other:

- Index/navigation files that link to all generated files
- Manifest files that list every output with its path and purpose
- Specs that reference shared model definitions
- Always include a cross-validation step to verify references resolve

## String Substitutions Reference

| Substitution            | Description                                  | Example                                                    |
|-------------------------|----------------------------------------------|------------------------------------------------------------|
| `$ARGUMENTS`            | All arguments as single string               | `/skill fix the login bug` → `fix the login bug`           |
| `$ARGUMENTS[N]`         | Nth word from arguments (0-indexed)          | `$ARGUMENTS[0]` → first word                               |
| `$1`, `$2`, `$3`        | Positional arguments                         | `/skill arg1 arg2` → `$1`=`arg1`, `$2`=`arg2`              |
| `$N`                    | Nth positional argument                      | Same as above                                              |
| `${CLAUDE_SESSION_ID}`  | Current session identifier                   | Unique per conversation                                    |
| `${CLAUDE_SKILL_DIR}`   | Absolute path to skill's directory           | Use for bundled scripts                                    |
| `${CLAUDE_PLUGIN_ROOT}` | Absolute path to the plugin's root directory | Use for shared plugin resources (plugin-level skills only) |

## Hooks in Skills

Skills can define scoped hooks via the `hooks` frontmatter field:

```yaml
---
name: safe-deploy
description: 'Deploy with pre-flight checks'
hooks:
  PreToolUse:
    - matcher: Bash
      hooks:
        - type: command
          command: echo "Bash command intercepted"
---
```

Hooks run only during skill execution and override global hooks for matched events.

## Visual Output Pattern

Skills can generate interactive HTML for rich output:

```markdown
Generate an interactive HTML report and save it to a temp file.
Use Bash to open it: `open /tmp/report.html`

The HTML should include:

- Summary dashboard
- Interactive charts (use inline CSS/JS, no CDN)
- Collapsible detail sections
```

This is useful for analysis skills, dashboards, and reports.

## Designing Skill Interactiveness

Interactiveness is NOT a level or template — it's a per-step design decision. Each step in a skill should interact with
the user only where user judgment genuinely adds value.

### When to engage deeply

- **High-stakes decisions** — architecture choices, destructive operations, irreversible actions
- **Ambiguous requirements** — multiple valid interpretations, user context needed
- **Trade-off selection** — performance vs readability, speed vs thoroughness
- **Creative/subjective work** — naming, UX decisions, prioritization

In these cases, the skill should:

- Propose alternatives with explicit trade-offs
- Ask probing questions to surface hidden requirements
- Challenge assumptions — "Are you sure X is the right approach? Consider Y because..."
- Present counter-arguments before proceeding

### When to just execute

- **Clear instructions** — user knows exactly what they want
- **Low-risk, reversible** — formatting, linting, simple generation
- **Well-defined scope** — no ambiguity in what to do
- **Automated checks** — tests, validation, analysis

### Embedding engagement in prompts

**Bad** — rubber-stamp gates:

```markdown
**Gatekeeping**: Ask user "Ready to proceed?"
```

**Good** — engagement where it matters:

```markdown
Before implementing, analyze the trade-offs:

- Option A: [pros/cons]
- Option B: [pros/cons]

Use AskUserQuestion to present options with your recommendation and reasoning. Challenge the user's assumptions if their
choice has non-obvious downsides.
```

**Good** — decision criteria with depth:

```markdown
If the codebase uses pattern X, propose approach A.
If the codebase uses pattern Y, propose approach B.
If unclear, use AskUserQuestion to explore: "I found both patterns X and Y in your code. Which direction do you want to
take, and why?"
```

## Validation Criteria

### Frontmatter Validation

- [ ] `name` uses lowercase, hyphens (max 64 chars)
- [ ] `description` explains what AND when to use (not just what)
- [ ] `description` is concise (under 100 chars preferred)
- [ ] Invocation fields (`disable-model-invocation`, `user-invocable`) set correctly
- [ ] Tools restricted if read-only (`allowed-tools`)
- [ ] `argument-hint` provided if arguments expected
- [ ] `context: fork` + `agent` used together if subagent execution needed
- [ ] `model` only set if specific model required

### Content Validation

- [ ] Imperative form ("Analyze...", "Create...", "Review...")
- [ ] SKILL.md under 500 lines (move detailed content to references/)
- [ ] Scripts tested by running (if using scripts/)
- [ ] References cited clearly (if using references/)
- [ ] Workflow defined with clear steps
- [ ] Review step included in workflow
- [ ] User feedback points identified (AskUserQuestion)
- [ ] Output format specified
- [ ] Critical instructions use emphasis (IMPORTANT, CRITICAL)
- [ ] Output file structure defined explicitly (for generator skills)
- [ ] Cross-validation step included (for multi-file output)
- [ ] Navigation/index file included (for 3+ output files)
- [ ] Input schemas/templates bundled in assets/ if applicable

### Structure Validation

- [ ] Directory follows conventions (`.claude/skills/$NAME/`)
- [ ] No extraneous files (README, CHANGELOG, etc.)
- [ ] Scripts in `scripts/` directory
- [ ] References in `references/` directory
- [ ] Assets in `assets/` directory
- [ ] Bundled files referenced with markdown links (`[text](references/file.md)`) notation
- [ ] Bundled script references use `${CLAUDE_SKILL_DIR}` notation
- [ ] Plugin-level skills use `${CLAUDE_PLUGIN_ROOT}` for shared plugin resources
- [ ] Plugin-level skills saved under `<plugin-root>/skills/<skill-name>/`
- [ ] Shell injection uses `!`command`` notation
- [ ] Engagement style clearly defined for interactive skills (where to challenge, where to execute)
