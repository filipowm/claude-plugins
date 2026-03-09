# Memory Best Practices — Comprehensive Guide

## Core Mental Model

CLAUDE.md goes into every conversation as system prompt context. The critical constraint: LLMs can reliably follow ~150-200 total instructions. Every line must earn its place. As instruction count increases, instruction-following quality degrades uniformly across ALL instructions.

## The Memory Hierarchy

| Priority | Scope | Location | When Loaded |
|---|---|---|---|
| 1 (highest) | Enterprise | `/etc/claude-code/CLAUDE.md` | Always |
| 2 | Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Always at launch |
| 3 | User | `~/.claude/CLAUDE.md` | Always at launch |
| 4 | Local | `./CLAUDE.local.md` | Always at launch |
| 5 | Subdirectory | `./src/api/CLAUDE.md` | On-demand |
| 6 | Modular rules | `.claude/rules/*.md` | At launch (no paths) or on-demand (with paths) |
| 7 | Auto-memory | `~/.claude/projects/<hash>/memory/MEMORY.md` | First 200 lines |

Files above the working directory: loaded in full at launch. Files below (children): loaded on-demand when Claude reads files there.

## What Goes Where

| File | What Belongs There |
|---|---|
| Root CLAUDE.md | Universal project standards, commands, stack versions, architecture map, critical rules |
| ~/.claude/CLAUDE.md | Personal preferences (response language, editor quirks). Never project-specific |
| CLAUDE.local.md | Personal sandbox URLs, test credentials, local overrides. Auto-gitignored |
| Subdirectory CLAUDE.md | Domain-specific conventions that only matter in that module |
| .claude/rules/*.md | Deep domain knowledge, path-filtered via paths: frontmatter |
| MEMORY.md | Auto-accumulated learnings. Organize by theme, not chronologically |

## Placement Decision Framework

| Question | Yes → | No → |
|---|---|---|
| Does every session need this? | Root CLAUDE.md | .claude/rules/ or subdirectory |
| Is it >5 lines of domain detail? | .claude/rules/ | Inline in CLAUDE.md |
| Does it apply to specific file types? | .claude/rules/ with glob paths | Root CLAUDE.md or subdirectory |
| Does it apply to a whole folder? | Subdirectory CLAUDE.md | .claude/rules/ |
| Would removing it cause mistakes in ANY task? | Root CLAUDE.md | .claude/rules/ or subdirectory |
| Must NEVER be skipped? | Hook (deterministic) | CLAUDE.md (advisory) |

## Writing Effective Instructions

### DO

- **Imperatives, not descriptions**: "Use functional components" not "The project uses functional components" (~25% better compliance)
- **Bullet points, not paragraphs**: ~35% better compliance
- **Exact versions**: "Next.js 15, TypeScript 5.7" not "latest Next.js" (~50% fewer compatibility errors)
- **Verifiable commands**: Including runnable commands reduces build errors by ~60%
- **Code examples for non-obvious patterns**: ~40% fewer correction requests
- **Alternatives with prohibitions**: "Never use console.log; use structuredLog() from lib/logger"
- **Sparse emphasis**: Reserve IMPORTANT/NEVER/CRITICAL for 2-3 truly critical rules

### DON'T

- Exceed 200 lines per file (target under 150)
- Include what Claude can infer from code
- Use CLAUDE.md as a linter (use Biome/ESLint + hooks)
- Auto-generate without editing (/init is a starting point only)
- Mark everything as IMPORTANT (if everything is critical, nothing is)
- Use bare prohibitions without alternatives
- `@`-import large docs (embeds entire file every session)

## Progressive Disclosure Architecture

```
Layer 1: CLAUDE.md (root)          <- Every conversation, ~60-150 lines
Layer 2: On-demand context         <- Loaded when Claude enters a directory
Layer 3: Deep reference            <- Claude reads only when needed
```

## AGENTS.md Integration Rules

When AGENTS.md approach is chosen:
- AGENTS.md: vendor-neutral content. No @imports, no .claude/ references
- CLAUDE.md: starts with `@AGENTS.md`, then Claude-specific extensions
- Every directory with CLAUDE.md also gets AGENTS.md
- AGENTS.md follows CommonMark, UTF-8
- Same content structure as CLAUDE.md but without Claude-specific features

## Architecture Guardrails Placement

| Guardrail Type | Where | Why |
|---|---|---|
| Critical never-violate boundaries | Hook + root CLAUDE.md | Deterministic + advisory |
| Architecture principles (3-5) | Root CLAUDE.md | Every session |
| Detailed architecture rules | .claude/rules/architecture.md (global) | Too long for root |
| Domain-specific architecture | .claude/rules/ with paths or subdirectory CLAUDE.md | On-demand |
| Security boundaries | .claude/rules/security.md (global) + Hook | Never optional |

## Context Budget

| Component | Typical Consumption |
|---|---|
| System prompt + CLAUDE.md + rules | 1,800-4,500 tokens (1-2%) |
| Auto-read files | 10,000-50,000 tokens (5-25%) |
| Conversation history | 30,000-120,000 tokens (15-60%) |

Total across all always-loaded sources should stay under 250 lines.

## Rules: .claude/rules/

Rules without `paths` field: loaded at launch (global). Rules with `paths`: loaded on-demand.

### Path Pattern Examples

| Pattern | Matches |
|---|---|
| `**/*.ts` | All TypeScript files |
| `src/**/*` | All files under src/ |
| `*.md` | Markdown files in project root only |
| `src/components/*.tsx` | React components in specific dir |
| `packages/*/src/**/*.ts` | Source files across monorepo packages |

### When to Use Rules vs Subdirectory CLAUDE.md

| Use .claude/rules/ | Use Subdirectory CLAUDE.md |
|---|---|
| Rule applies across scattered file types | Module has its own identity with multiple conventions |
| Centralized management of cross-cutting concerns | Each package in monorepo has distinct standards |
| Path matching doesn't align with folder structure | Folder structure IS the scoping mechanism |

## Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| Over-specified (>200 lines) | Prune; move depth to .claude/rules/ |
| Passive descriptions | Rewrite as imperatives |
| Contradictory instructions across levels | Audit all levels; remove conflicts |
| Kitchen sink (everything in one file) | Progressive disclosure: root -> subdirectory -> reference |
| Vague versions ("latest") | Pin exact versions |
| Using CLAUDE.md as linter | Use Biome/ESLint + hooks |
| Everything marked IMPORTANT | Reserve emphasis for 2-3 critical rules |
| @-importing large docs | Use path references; let Claude read on-demand |
| Bare prohibitions without alternatives | Always provide the preferred alternative |

## Security & Compliance Rules Placement

| Concern | Where |
|---|---|
| Core security principles (2-4 bullets) | Root CLAUDE.md |
| Detailed security patterns | .claude/rules/security.md (global) |
| Regulatory compliance (HIPAA, PCI, GDPR) | .claude/rules/[domain]-compliance.md (global) |
| Pre-commit secrets scan | Hook (deterministic) |
| Framework-specific security | .claude/rules/ with paths |