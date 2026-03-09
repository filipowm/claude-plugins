# GitHub Ways of Working - Reference Guidelines

## Roles and Responsibilities

### DevOps Team
- Define CI/CD architecture
- Maintain documentation and onboarding
- Define teams and permissions
- Admin/maintain organization settings, roles, and rulesets
- Grant initial access through CIDM
- Maintain self-hosted runners
- Write and maintain catalog of reusable GitHub Actions
- Review and implement action requests from developers
- Maintain list of approved community actions
- Provide guidance for workflow contributions

### Development Teams
- Contribute to workflows and actions following DevOps guidelines
- Adapt workflows to code changes (upgrades, EOL changes)
- Collaborate with DevOps on workflow changes (DevOps owns `.github/` folder)
- Create PRs for workflow changes, reviewed by DevOps
- Identify and propose workflow improvements
- Manage team access and per-project repository controls

## Recommended Roles

### Organization Roles
- **Base role**: Read permissions over entire organization (enforced by default)

### Repository Roles

**write-role** (Team Write):
- Base Role: Write
- For developers to write to team repositories

**tech-lead-role** (Team Tech Lead):
- Base Role: Write
- Additional: bypass certain rules (e.g., signed commits), create release branches

## Naming Conventions

### Actions & Workflows
- Action repos: suffix `-action`
- Workflow repos: suffix `-workflows`

### Branches
- `feature/<description>` (e.g., `feature/add-user-authentication`)
- `bugfix/<description>` (e.g., `bugfix/fix-login-error`)
- `release/<version>` (e.g., `release/1.0.0`)
- `hotfix/<description>` (e.g., `hotfix/critical-security-patch`)

### Tags
- Version: `v<major>.<minor>.<patch>` (e.g., `v1.0.0`)
- Pre-release: `v<version>-<label>` (e.g., `v1.0.0-beta`)

### Commit Messages
- Follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
- Format: `<type>: <description>`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### Pull Request Titles
- Format: `[Type] Short description` (e.g., `[Feature] Implement user authentication`)

## Required Repository Files
1. `README.md` — project overview, setup, usage, contribution guidelines
2. `CONTRIBUTING.md` — code standards, PR process, issue reporting
3. `CODE_OF_CONDUCT.md` — community standards (Contributor Covenant recommended)
4. `CHANGELOG.md` — notable changes per version (Keep a Changelog format)
5. `CODEOWNERS` — file/directory ownership for code reviews
6. `catalog-info.yaml` — DevHub metadata

## Security Requirements

### RBAC
- Use GitHub Teams and Organizations for permission management
- Grant least privilege for each role
- Map groups/users to CIDM
- DevOps grants initial org access, dev teams manage team/repo assignments

### Rulesets (Branch Protection)
- **Restrict Deletions**: only bypass users can delete matching refs
- **Require PR Before Merging**: all commits via PR
- **Require Status Checks**: linting + minimum test set must pass
- **Block Force Pushes**: prevent force push to protected refs

### Dependency Management
- Enable **Dependabot** for monitoring and managing dependencies
- Regularly update dependencies

### Secret Detection
- Secret scanning **must be enabled**
- Enable push protection to block secrets before they reach the repo
