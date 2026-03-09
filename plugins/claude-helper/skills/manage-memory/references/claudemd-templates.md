# CLAUDE.md Templates by Project Type

## Universal Template (All Projects)

```markdown
# ProjectName
One-line description: framework, purpose, key integrations.

## Tech Stack
[there can be multiple languages, frameworks, etc..]
- Languages: [language] [version]
- Frameworks: [framework] [version]
- Runtimes: [runtime] [version], [package-manager] [version]
- Databases: [db] [version], [ORM] [version]
- Testing: [test-framework] [version]

## Commands
- Dev server: `[command]` (port [N])
- Run all tests: `[command]`
- Run single test: `[command] -- path/to/test`
- Lint: `[command]`
- Type check: `[command]`
- Build: `[command]`

## Architecture
[annotated directory tree — key dirs only, one-line purpose each]

## Architecture Principles
- [3-7 core principles that, if violated, would cause real damage]

## Code Conventions
[only top-level critical conventions which should not live in rules or sub-CLAUDE.md]
- [naming, file organization, import patterns]
- [only what Claude can't infer from reading code]

## Testing
- [high-level philosophy, not detailed patterns]
- [verification command to run before commits]

## Git Workflow
- Branch naming: [pattern]
- Commit format: [convention]

## Critical Rules
- NEVER [critical prohibition] — [alternative]
- IMPORTANT: [critical requirement]

## Verification
- After code changes: `[type-check] && [test]`
- [domain-specific verification steps]
```

## Next.js / React Template Additions

```markdown
## Tech Stack
- Frontend: Next.js [ver], TypeScript [ver], Tailwind CSS [ver]
- State: React Query (server), Zustand (client)

## Architecture Principles
- Server components by default; add "use client" ONLY for interactivity
- Business logic in `server/services/`, NEVER in route handlers or components
- All DB access through `lib/db/` — no direct ORM calls elsewhere

## Code Conventions
- Functional React components with hooks only
- Named exports only (no default exports)
- Components: PascalCase; files: kebab-case
```

## Python / FastAPI Template Additions

```markdown
## Tech Stack
- Language: Python [ver]
- Framework: FastAPI [ver]
- ORM: SQLAlchemy [ver] / Prisma [ver]
- Package manager: uv [ver] / poetry [ver]

## Commands
- Dev server: `uvicorn app.main:app --reload`
- Tests: `pytest`
- Type check: `mypy .`
- Lint: `ruff check .`

## Architecture Principles
- Dependency injection via FastAPI's Depends()
- All routes in `app/routers/`, business logic in `app/services/`
- Pydantic models for ALL request/response schemas
```

## Go Template Additions

```markdown
## Tech Stack
- Language: Go [ver]
- Framework: [gin/echo/chi] [ver]

## Commands
- Build: `go build ./...`
- Test: `go test ./...`
- Lint: `golangci-lint run`

## Code Conventions
- Error handling: always check returned errors, never use `_`
- Package naming: short, lowercase, no underscores
- Interfaces: define where consumed, not where implemented
```

## Java / Spring Template Additions

```markdown
## Tech Stack
- Language: Java [ver]
- Framework: Spring Boot [ver]
- Build: Maven [ver] / Gradle [ver]

## Commands
- Build: `./mvnw clean package` / `./gradlew build`
- Test: `./mvnw test` / `./gradlew test`
- Run: `./mvnw spring-boot:run`

## Architecture Principles
- Constructor injection only (never field injection)
- Service layer for business logic, controllers are thin
- Repository pattern for data access
```

## Rust Template Additions

```markdown
## Tech Stack
- Language: Rust [ver]
- Framework: [actix-web/axum/rocket] [ver]

## Commands
- Build: `cargo build`
- Test: `cargo test`
- Lint: `cargo clippy`
- Format: `cargo fmt`

## Code Conventions
- Use `thiserror` for library errors, `anyhow` for application errors
- Prefer `impl Trait` over `dyn Trait` where possible
- All public APIs must have doc comments
```

## Monorepo Template

```markdown
# MonorepoName
Monorepo with [N] services/packages managed by [turbo/nx/lerna/pnpm workspaces].

## Tech Stack
- Monorepo tool: [tool] [ver]
- Shared: TypeScript [ver], [package-manager] [ver]

## Commands
- Install all: `[command]`
- Build all: `[command]`
- Test all: `[command]`
- Test single package: `[command] --filter=[package]`
- Lint: `[command]`

## Architecture
packages/
├── [package-a]/   # [one-line purpose]
├── [package-b]/   # [one-line purpose]
└── shared/        # Shared utilities, types
apps/
├── [app-a]/       # [one-line purpose]
└── [app-b]/       # [one-line purpose]

## Architecture Principles
- Each package exports ONLY through its index.ts barrel
- Cross-package imports go through the barrel, never reach into internals
- Shared code for 2+ packages goes to `packages/shared/`
- NEVER create circular dependencies between packages

## Per-Package Instructions
Each package has its own CLAUDE.md with specific conventions.
```

## AGENTS.md Template

Same structure as CLAUDE.md but without Claude-specific features:
- No `@imports`
- No references to `.claude/` directory
- No references to skills, subagents, MCP, hooks
- Must be valid CommonMark, UTF-8
- Focus on universal instructions any AI coding tool can follow