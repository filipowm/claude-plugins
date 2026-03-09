---
name: configure-github
description: 'Configure GitHub organization or repository settings: rulesets, roles, security, branch protection, Dependabot, secret scanning, merge settings. Use when user wants to set up, configure, or audit GitHub settings.'
argument-hint: '[org|repo] [name]'
---

# GitHub Configuration

Guide user through comprehensive GitHub configuration for an organization or repository using the `gh` CLI. Challenge risky choices, recommend best practices from Ways of Working guidelines, and document everything.

Read the reference guidelines first, see [ways-of-working.md](references/ways-of-working.md).

IMPORTANT: Always challenge decisions that weaken security or deviate from best practices. Explain trade-offs clearly.

## Workflow

Strictly follow workflow phases. Do not skip phases.

---

### Phase 1: Discovery

Determine scope and assess current state.

1. Verify `gh` is authenticated: `gh auth status`
   - If not authenticated, tell user to run `/github:setup` first and stop.

2. Determine scope from `$ARGUMENTS` or ask:
   - **Organization**: configure org-wide settings (rulesets, roles, security policies)
   - **Repository**: configure a specific repo (rulesets, security, merge settings, structure)

3. If **organization** scope:
   ```bash
   gh api /orgs/{org} --jq '{login,name,default_repository_permission,members_can_create_repositories,two_factor_requirement_enabled}'
   gh api /orgs/{org}/rulesets --jq '.[] | {id,name,target,enforcement}'
   gh api /orgs/{org}/teams --jq '.[] | {name,slug,permission,privacy}' 2>/dev/null || echo "No teams access"
   gh api /orgs/{org}/custom-repository-roles --jq '.custom_roles[] | {name,base_role}' 2>/dev/null || echo "No custom roles access"
   ```

4. If **repository** scope:
   ```bash
   gh repo view {owner}/{repo} --json name,defaultBranchRef,isPrivate,hasIssuesEnabled,hasWikiEnabled,hasProjectsEnabled,mergeCommitAllowed,squashMergeAllowed,rebaseMergeAllowed,deleteBranchOnMerge,autoMergeAllowed
   gh api /repos/{owner}/{repo}/rulesets --jq '.[] | {id,name,target,enforcement}' 2>/dev/null || echo "No rulesets"
   gh api /repos/{owner}/{repo} --jq '{security_and_analysis}'
   ```

5. Present current state summary to user. Highlight gaps vs. recommended configuration.

Use AskUserQuestion to confirm which areas to configure. Present all phases as options with current status (configured/not configured/needs update).

---

### Phase 2: Roles & Access Control

IMPORTANT: This phase applies primarily to organization scope. For repo scope, focus on team access and CODEOWNERS.

**For organizations:**

1. **Review base permissions**. Recommend: `read` as base role.
   ```bash
   gh api /orgs/{org} --jq '.default_repository_permission'
   ```
   If not `read`, warn and recommend changing:
   ```bash
   gh api /orgs/{org} -X PATCH -f default_repository_permission=read
   ```

2. **Custom repository roles**. Recommend creating per Ways of Working:
   - `write-role` (base: write)
   - `tech-lead-role` (base: write + bypass rules + create release branches)

   Use AskUserQuestion to explore: What teams exist? What access levels do they need? Are there tech leads who need elevated permissions?

   ```bash
   gh api /orgs/{org}/custom-repository-roles -X POST \
     -f name="write-role" -f base_role="write" -f description="Standard developer write access"
   gh api /orgs/{org}/custom-repository-roles -X POST \
     -f name="tech-lead-role" -f base_role="write" -f description="Tech lead with bypass and release permissions" \
     --input - <<< '{"permissions":["bypass_branch_protections","create_tag"]}'
   ```

3. **Teams setup**. Ask about team structure and create/verify teams:
   ```bash
   gh api /orgs/{org}/teams -X POST -f name="{team}" -f privacy="closed" -f description="{desc}"
   ```

**For repositories:**

1. **Team access**: assign teams to repo with appropriate roles:
   ```bash
   gh api /orgs/{org}/teams/{team}/repos/{owner}/{repo} -X PUT -f permission="push"
   ```

2. **CODEOWNERS**: discuss ownership structure and create `.github/CODEOWNERS`:
   - Ask: Who owns what parts of the codebase? Should DevOps own `.github/`?
   - Recommend: DevOps team owns `.github/` directory

---

### Phase 3: Rulesets

Configure branch and tag rulesets. This is the primary mechanism for branch protection.

IMPORTANT: Rulesets replace legacy branch protection rules. Always prefer rulesets. If legacy rules exist, recommend migrating.

1. **Explore requirements** with probing questions:
   - Which branches need protection? (at minimum: default branch, release branches)
   - Who should be able to bypass rules? (tech leads? CI bots?)
   - What status checks are required before merging?
   - Should commits be signed?
   - What branch naming patterns to enforce?

2. **Default branch protection ruleset** (CRITICAL — always recommend):

   For **repository** scope:
   ```bash
   gh api /repos/{owner}/{repo}/rulesets -X POST --input - << 'EOF'
   {
     "name": "default-branch-protection",
     "target": "branch",
     "enforcement": "active",
     "conditions": {
       "ref_name": { "include": ["~DEFAULT_BRANCH"], "exclude": [] }
     },
     "rules": [
       { "type": "deletion" },
       { "type": "non_fast_forward" },
       { "type": "pull_request", "parameters": {
         "required_approving_review_count": 1,
         "dismiss_stale_reviews_on_push": true,
         "require_code_owner_review": true,
         "require_last_push_approval": false,
         "required_review_thread_resolution": true
       }},
       { "type": "required_status_checks", "parameters": {
         "strict_required_status_checks_policy": true,
         "required_status_checks": []
       }}
     ],
     "bypass_actors": []
   }
   EOF
   ```

   For **organization** scope:
   ```bash
   gh api /orgs/{org}/rulesets -X POST --input - << 'EOF'
   {
     "name": "default-branch-protection",
     "target": "branch",
     "enforcement": "active",
     "conditions": {
       "ref_name": { "include": ["~DEFAULT_BRANCH"], "exclude": [] },
       "repository_name": { "include": ["~ALL"], "exclude": [] }
     },
     "rules": [
       { "type": "deletion" },
       { "type": "non_fast_forward" },
       { "type": "pull_request", "parameters": {
         "required_approving_review_count": 1,
         "dismiss_stale_reviews_on_push": true,
         "require_code_owner_review": true,
         "required_review_thread_resolution": true
       }}
     ]
   }
   EOF
   ```

3. **Release branch protection** — protect `release/*` branches with similar rules.

4. **Branch naming convention ruleset** — enforce naming patterns:
   ```json
   {
     "name": "branch-naming-convention",
     "target": "branch",
     "enforcement": "active",
     "rules": [
       { "type": "creation", "parameters": {
         "name_pattern": {
           "operator": "regex",
           "pattern": "^(main|master|develop|feature/.*|bugfix/.*|hotfix/.*|release/.*)$",
           "negate": false,
           "name": "Branch naming"
         }
       }}
     ]
   }
   ```

5. **Tag protection ruleset** — enforce `v*` tag format and restrict who can create tags.

Use AskUserQuestion after presenting each ruleset to discuss: Are the rules too strict? Too lenient? Need bypass actors?

IMPORTANT: If user wants to skip branch protection, actively challenge this. Explain the risks of unprotected branches.

---

### Phase 4: Security

Configure security features. All items in this phase are STRONGLY RECOMMENDED.

1. **Secret scanning + push protection**:
   ```bash
   # Enable secret scanning
   gh api /repos/{owner}/{repo} -X PATCH --input - << 'EOF'
   {
     "security_and_analysis": {
       "secret_scanning": { "status": "enabled" },
       "secret_scanning_push_protection": { "status": "enabled" }
     }
   }
   EOF
   ```
   For org-level: configure via org security settings.

   IMPORTANT: Secret scanning MUST be enabled per security requirements. Challenge any attempt to skip this.

2. **Dependabot**:
   - Enable Dependabot alerts:
     ```bash
     gh api /repos/{owner}/{repo}/vulnerability-alerts -X PUT
     ```
   - Enable Dependabot security updates:
     ```bash
     gh api /repos/{owner}/{repo}/automated-security-fixes -X PUT
     ```
   - Ask if user wants Dependabot version updates. If yes, create `.github/dependabot.yml`:
     ```yaml
     version: 2
     updates:
       - package-ecosystem: "<detect-from-repo>"
         directory: "/"
         schedule:
           interval: "weekly"
         open-pull-requests-limit: 10
     ```
   Auto-detect ecosystems from repo contents (package.json, go.mod, pom.xml, requirements.txt, Gemfile, etc.).

3. **Code scanning** (if available):
   Ask if user wants to enable CodeQL or other code scanning tools. Recommend for production repositories.

4. **Two-factor authentication** (org scope):
   ```bash
   gh api /orgs/{org} --jq '.two_factor_requirement_enabled'
   ```
   If not enabled, strongly recommend enabling.

---

### Phase 5: Merge & CI Settings

Configure merge strategies, merge queue, and CI integration.

1. **Merge strategies** — ask which strategies to allow:
   ```bash
   gh api /repos/{owner}/{repo} -X PATCH \
     -F allow_merge_commit=true \
     -F allow_squash_merge=true \
     -F allow_rebase_merge=true \
     -F delete_branch_on_merge=true \
     -F allow_auto_merge=false \
     -F squash_merge_commit_title="PR_TITLE" \
     -F squash_merge_commit_message="PR_BODY"
   ```
   Discuss trade-offs: squash for clean history vs. merge for full context.

2. **Auto-merge**: recommend enabling for repos with good CI coverage. Challenge enabling it without required status checks.

3. **Merge queue**: ask if high-traffic repo needs merge queue to prevent broken builds.

4. **Required status checks**: work with user to define which CI checks must pass. Ask about their CI pipeline (linting, tests, builds, security scans).

---

### Phase 6: Repository Structure

Verify and create required repository files.

1. Check for existing files:
   ```bash
   gh api /repos/{owner}/{repo}/contents/.github 2>/dev/null
   ls -la README.md CONTRIBUTING.md CODE_OF_CONDUCT.md CHANGELOG.md .github/CODEOWNERS 2>/dev/null
   ```

2. For each missing recommended file, ask user if they want to create it:
   - `README.md` — project overview
   - `CONTRIBUTING.md` — contribution guidelines
   - `CODE_OF_CONDUCT.md` — Contributor Covenant template
   - `CHANGELOG.md` — Keep a Changelog format
   - `CODEOWNERS` — code ownership
   - `.github/PULL_REQUEST_TEMPLATE.md` — PR template
   - `.github/ISSUE_TEMPLATE/` — issue templates (bug report, feature request)

3. Do NOT auto-generate all files. Discuss what's relevant for this project. Challenge if user skips CODEOWNERS or PR templates — these significantly improve code review quality.

---

### Phase 7: Generate SETUP.md

Create `.github/SETUP.md` documenting all configuration applied.

Structure:
```markdown
# GitHub Configuration

> Last updated: {date}
> Configured by: {user}
> Scope: {org|repo} — {name}

## Roles & Access Control
{summary of roles, teams, permissions}

## Rulesets
{list of rulesets with their rules and enforcement level}

## Security
{secret scanning, Dependabot, code scanning status}

## Merge Settings
{allowed strategies, auto-merge, merge queue}

## Repository Structure
{list of configured files: CODEOWNERS, templates, etc.}

## Naming Conventions
{branch, tag, commit message conventions}

## Audit Log
{chronological list of changes made during this configuration session}
```

Include actual values — not placeholders. Document what was configured and what was intentionally skipped (with reasons).

---

### Phase 8: Review & Verify

Verify the configuration is complete and consistent.

1. **Re-fetch current state** using the same commands from Phase 1.
2. **Compare** against recommended configuration from Ways of Working.
3. **Generate audit summary**:

   | Area | Status | Details |
   |------|--------|---------|
   | Branch protection | Configured/Missing | ... |
   | Secret scanning | Enabled/Disabled | ... |
   | Dependabot | Enabled/Disabled | ... |
   | CODEOWNERS | Present/Missing | ... |
   | ... | ... | ... |

4. **Flag gaps**: clearly highlight anything that deviates from recommendations and why it matters.

5. Present final summary to user. If there are critical gaps (no branch protection, no secret scanning), strongly recommend addressing them before finishing.

---

## Guidelines

1. **Always use `gh` CLI and `gh api`** — never construct raw HTTP requests
2. **Challenge risky choices** — if user wants to skip security features, explain consequences
3. **Explain trade-offs** — don't just recommend, explain why
4. **Idempotent operations** — check before creating (don't duplicate rulesets, roles, etc.)
5. **Error handling** — if a `gh api` call fails due to permissions, explain what access is needed
6. **Organization vs. repo** — some features are org-only (custom roles, org rulesets). Guide appropriately.
7. **Document everything** — every configuration change goes into SETUP.md
8. **Conventional commits** — remind user about commit message conventions when relevant
