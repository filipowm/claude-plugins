# Rule File Templates by Domain

## Code Style (global — no paths)

```markdown
# Code Style

## Naming
- Files: kebab-case (e.g., user-profile.tsx)
- Components: PascalCase (e.g., UserProfile)
- Functions/variables: camelCase
- Constants: UPPER_SNAKE_CASE
- Types/interfaces: PascalCase with no I- prefix

## Imports
- Group imports: external → internal → relative
- Use absolute imports from `@/` prefix
- Named exports only (no default exports)

## Functions
- Prefer pure functions where possible
- Max function length: ~50 lines (split if longer)
- Early returns over nested conditionals
```

## Architecture (global — no paths)

```markdown
# Architecture Rules

## Layer Dependencies
- UI components -> hooks -> services -> repositories -> database
- NEVER skip layers (e.g., component directly calling repository)
- NEVER create circular dependencies between packages

## Module Boundaries
- Each feature module exports ONLY through its index barrel file
- Cross-feature imports must go through the barrel
- Shared code for 2+ features goes to lib/shared/

## State Management
- Server state: [React Query / SWR / etc.]
- Local UI state: useState/useReducer in components
- Global client state: [Zustand / Redux / etc.] stores
- NEVER mix server and client state in the same store

## Data Flow
- API responses validated before reaching the app
- Error boundaries wrap every route segment
- Logging through structured logger only — never console.log in production
```

## Testing (path-scoped)

```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
  - "**/*.spec.ts"
  - "tests/**/*"
---

# Testing Conventions

## Structure
- Use describe(ComponentOrFunction, ...) grouping
- Test names: should [expected behavior] when [condition]
- One assertion per test when possible

## Mocking
- Mock network calls with [msw / nock / etc.], NOT manual fetch mocks
- Fixtures in tests/fixtures/ — share across tests, never duplicate
- NEVER mock what you don't own — wrap third-party code, mock the wrapper

## Anti-Patterns (DO NOT)
- Testing implementation details (internal state, private methods)
- Tests that depend on execution order
- Snapshot tests for dynamic content
- any casts in test files — type your mocks properly
- Thread.sleep/setTimeout for async — use waitFor() or findBy
```

## Security (global — no paths)

```markdown
# Security Rules

## Input Validation
- Validate on the server even if client validates
- Use allowlists, not denylists for input patterns
- Parameterized queries ONLY — never string concatenation for SQL
- Sanitize HTML output to prevent XSS

## Secrets
- NEVER hardcode secrets, API keys, or credentials
- Use environment variables for all secrets
- NEVER log secrets or PII

## Authentication
- Check authorization on EVERY endpoint
- Use established auth libraries, never roll your own crypto
- Session tokens: HttpOnly, Secure, SameSite=Strict

## Dependencies
- Pin exact dependency versions in lockfiles
- Audit before adding new dependencies
```

## API Development (path-scoped)

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "src/routes/**/*.ts"
  - "app/api/**/*.ts"
---

# API Development Rules

## Patterns
- Validate all inputs with [Zod / Joi / class-validator]
- Use standard error response format
- Include rate limiting on public endpoints
- Document with OpenAPI comments

## Auth
- Public routes: [publicProcedure / no middleware]
- Authenticated routes: [protectedProcedure / auth middleware]
- Admin-only routes: [adminProcedure / admin middleware]

## Conventions
- One router/controller per domain entity
- Business logic in services, NOT in route handlers
- Always log mutations with structured logger
```

## Frontend Components (path-scoped)

```markdown
---
paths:
  - "src/components/**/*.tsx"
  - "src/components/**/*.ts"
  - "app/components/**/*.tsx"
---

# Component Development

## Architecture
- [Atomic design / feature-based / flat] organization
- Every component: ComponentName/index.tsx + ComponentName.test.tsx

## Conventions
- [Tailwind / CSS Modules / styled-components] for styling
- All components must support keyboard navigation
- Use forwardRef for any component that accepts ref
- Props interface named [ComponentName]Props
```

## Database & Migrations (path-scoped)

```markdown
---
paths:
  - "prisma/**/*"
  - "src/db/**/*"
  - "migrations/**/*"
  - "drizzle/**/*"
---

# Database Rules

## Migrations
- NEVER edit existing migration files
- Use the ORM's migration tool to generate migrations
- Always test migrations with rollback

## Queries
- All DB access goes through the data access layer
- Use transactions for multi-step mutations
- Index frequently queried columns

## Schema
- Use explicit column types (never rely on defaults)
- Add NOT NULL constraints where appropriate
- Foreign keys for all relationships
```

## E2E Testing (path-scoped)

```markdown
---
paths:
  - "tests/e2e/**/*"
  - "e2e/**/*"
  - "cypress/**/*"
  - "playwright/**/*"
---

# E2E Test Rules

## Selectors
- Use data-testid attributes, NEVER CSS classes or text content
- Page objects in tests/e2e/pages/ — one per page/view

## Execution
- Each test creates fresh state, cleans up after
- Wait for network idle before assertions
- Default timeout: 30s; use test.slow() for complex workflows

## Data
- Seed test data programmatically, not via UI
- Use synthetic data only — never real user data
```

## Compliance: HIPAA (global — no paths)

```markdown
# HIPAA Compliance Rules

## PHI Protection
- NEVER include real patient data in code, comments, tests, or logs
- Use de-identified placeholder data in ALL fixtures
- All PHI fields encrypted at rest (AES-256)
- NEVER log PHI — log only opaque record IDs

## Access Controls
- Every PHI endpoint requires authorized procedure
- Session timeout: 15 minutes for PHI-accessing sessions
- Break-the-glass audit logging for emergency access

## Audit
- All PHI access generates audit log: who, what, when, why
- Audit logs are append-only, NEVER modified or deleted
```

## Compliance: PCI-DSS (global — no paths)

```markdown
# PCI-DSS Compliance Rules

## Payment Data
- NEVER store raw credit card numbers — use payment processor tokens
- NEVER log card numbers, CVVs, or full account numbers
- Mask card numbers in UI: show only last 4 digits
- All monetary calculations use decimal arithmetic, NEVER floating point

## Isolation
- Payment processing code isolated in dedicated module
- No payment logic in route handlers or UI components
- All payment API calls use TLS 1.3

## Audit Trail
- Every financial transaction has immutable audit entry
- Records retained for minimum 7 years
```