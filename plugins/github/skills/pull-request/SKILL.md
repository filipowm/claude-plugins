---
name: pull-request
description: 'Manage GitHub pull requests: create, modify, close, merge, summarize status. Use when user wants to work with PRs, check PR status, merge, or manage pull requests. NOT for code review — use code-review skill instead.'
argument-hint: [ action ] [ PR-number ] [ options ]
---

# Pull Request Management

Manage GitHub pull requests using the `gh` CLI. Parse `$ARGUMENTS` to determine the requested operation and execute it.

IMPORTANT: This skill handles PR lifecycle management. For code review (reviewing code changes, leaving review comments,
approving/requesting changes), defer to the `code-review` skill.

## Operation Detection

Parse `$ARGUMENTS` to identify the operation. Match against these patterns:

| Intent         | Trigger words                                         |
|----------------|-------------------------------------------------------|
| Create         | create, new, open, make                               |
| Modify         | edit, update, change, modify, set                     |
| Close          | close                                                 |
| Reopen         | reopen, re-open                                       |
| Merge          | merge, land, ship                                     |
| Status         | status, summary, summarize, info, details, show, view |
| List           | list, ls, show all, my prs                            |
| Diff           | diff, changes, what changed                           |
| Checkout       | checkout, check out, switch to, test locally          |
| Draft          | draft, ready, mark ready, mark draft                  |
| Comment        | comment, note, add comment                            |
| Reply          | reply, respond, answer                                |
| Review Summary | review summary, reviews, review status, feedback      |
| Update branch  | update branch, rebase, sync                           |
| Actions/Checks | checks, actions, ci, pipeline, workflows              |

If no clear match, treat `$ARGUMENTS` as a free-form PR request and use best judgment to determine the appropriate `gh`
commands.

If `$ARGUMENTS` is empty, run `gh pr status` to show current PR context, then ask what the user wants to do.

---

## Operations

### Create PR

1. Detect current branch and verify it has unpushed commits or differs from base
2. Auto-detect base branch (default branch of repo via `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`)
3. Push branch if needed: `git push -u origin HEAD`
4. Generate title and body from commits: `git log $(git merge-base HEAD <base>)..HEAD --oneline`
5. Check for PR template: look for `.github/PULL_REQUEST_TEMPLATE.md` or `.github/pull_request_template.md`
6. Create PR: `gh pr create --base <base> --title "<title>" --body "<body>"`
7. If PR template exists, incorporate it into the body
8. Apply labels, reviewers, assignees if specified in `$ARGUMENTS`

Use sensible defaults — do NOT ask for title/body/branch unless the user explicitly provides them. Infer from commits.

### Modify PR

Parse what to modify from `$ARGUMENTS`. Use appropriate `gh pr edit` flags:

- Title: `gh pr edit <number> --title "<title>"`
- Body: `gh pr edit <number> --body "<body>"`
- Reviewers: `gh pr edit <number> --add-reviewer <user>`
- Labels: `gh pr edit <number> --add-label <label>`
- Assignees: `gh pr edit <number> --add-assignee <user>`
- Base branch: `gh pr edit <number> --base <branch>`
- Milestone: `gh pr edit <number> --milestone "<name>"`

Multiple edits can be combined in a single `gh pr edit` call.

### Close PR

1. Fetch PR details: `gh pr view <number> --json title,state,comments,reviews`
2. **IMPORTANT: Confirm with the user before closing.** Show PR title and any unresolved review comments.
3. Close: `gh pr close <number>`
4. Optionally delete branch if user requests: `gh pr close <number> --delete-branch`

### Reopen PR

Reopen: `gh pr reopen <number>`

### Merge PR

1. Fetch PR status:
   `gh pr view <number> --json title,state,mergeable,mergeStateStatus,statusCheckRollup,reviews,reviewDecision`
2. **Check CI status.** If checks are failing or pending:
    - IMPORTANT: Warn the user explicitly: "CI checks are failing/pending. Are you sure you want to merge?"
    - Show which checks failed
    - Wait for user confirmation before proceeding
3. **Check review status.** If reviews are required and not approved, warn the user.
4. **Confirm merge strategy with user** only if not specified in `$ARGUMENTS`:
    - merge commit (default)
    - squash
    - rebase
5. Merge: `gh pr merge <number> --merge|--squash|--rebase`
6. Optionally delete branch: add `--delete-branch` if user requests

### Status / Summary

Gather comprehensive PR information and present it clearly:

1.
`gh pr view <number> --json number,title,state,author,baseRefName,headRefName,body,createdAt,updatedAt,mergeable,mergeStateStatus,isDraft,labels,assignees,milestone,reviewRequests,reviewDecision,additions,deletions,changedFiles`
2. `gh pr checks <number>` — CI/actions status
3.
`gh api repos/{owner}/{repo}/pulls/<number>/comments --jq '.[] | {user: .user.login, body: .body, created_at: .created_at, path: .path}'` —
review comments
4. `gh pr view <number> --comments` — conversation comments

Present as structured summary:

```
## PR #<number>: <title>
**Status:** <state> | **Draft:** <yes/no> | **Mergeable:** <status>
**Author:** <author> | **Branch:** <head> → <base>
**Created:** <date> | **Updated:** <date>
**Changes:** +<additions> -<deletions> across <files> files
**Labels:** <labels> | **Assignees:** <assignees> | **Milestone:** <milestone>

### Review Status
<review decision and pending reviewers>

### CI/Checks
<table of check names and statuses>

### Comments Summary
<summary of conversation and review comments>
```

### List PRs

List PRs with optional filters from `$ARGUMENTS`:

- Default: `gh pr list`
- By author: `gh pr list --author <user>`
- By state: `gh pr list --state <open|closed|merged|all>`
- By label: `gh pr list --label <label>`
- By assignee: `gh pr list --assignee <user>`
- My PRs: `gh pr list --author @me`
- Search: `gh pr list --search "<query>"`

### Diff

Show PR changes: `gh pr diff <number>`

If no number given, show diff for current branch PR: `gh pr diff`

### Checkout

Checkout PR locally: `gh pr checkout <number>`

### Draft Toggle

- Mark as ready: `gh pr ready <number>`
- Convert to draft: `gh pr ready <number> --undo`

IMPORTANT: Confirm before marking as ready — this notifies reviewers.

### Comment

Add a conversation comment (top-level PR comment):

```
gh pr comment <number> --body "<comment>"
```

Add an inline review comment on a specific file/line using `gh api`:

```
gh api repos/{owner}/{repo}/pulls/<number>/reviews \
  -X POST \
  -f event=COMMENT \
  -f body="" \
  --jq '.id' \
  # Then add individual file comments:
gh api repos/{owner}/{repo}/pulls/<number>/comments \
  -X POST \
  -f body="<comment>" \
  -f path="<file-path>" \
  -F line=<line-number> \
  -f side=RIGHT \
  -f commit_id="$(gh pr view <number> --json headRefOid -q .headRefOid)"
```

If the user specifies a file and line, use the inline comment approach. Otherwise, default to conversation comment.

If comment body not provided in `$ARGUMENTS`, ask the user what to comment.

### Reply to Comment

Reply to an existing PR review comment:

1. First, list existing review comments to find the target:
   ```
   gh api repos/{owner}/{repo}/pulls/<number>/comments \
     --jq '.[] | {id: .id, user: .user.login, body: .body[:80], path: .path, line: .line, created_at: .created_at}'
   ```
2. If user doesn't specify which comment, show the list and use AskUserQuestion to let them pick.
3. Reply to the comment:
   ```
   gh api repos/{owner}/{repo}/pulls/<number>/comments/<comment-id>/replies \
     -X POST \
     -f body="<reply>"
   ```

For replying to a **conversation comment** (non-review, top-level), use:

```
gh pr comment <number> --body "<reply>"
```

since conversation comments are linear — a new comment serves as a reply.

### Review Summary

Summarize all reviews and review feedback on a PR:

1. Fetch reviews:
   ```
   gh api repos/{owner}/{repo}/pulls/<number>/reviews \
     --jq '.[] | {id: .id, user: .user.login, state: .state, body: .body, submitted_at: .submitted_at}'
   ```
2. Fetch review comments (inline code comments):
   ```
   gh api repos/{owner}/{repo}/pulls/<number>/comments \
     --jq '.[] | {id: .id, user: .user.login, body: .body, path: .path, line: .line, in_reply_to_id: .in_reply_to_id, created_at: .created_at}'
   ```
3. Fetch conversation comments:
   ```
   gh pr view <number> --comments
   ```

Present as structured summary:

```
## Review Summary for PR #<number>

### Reviews
| Reviewer | Decision | Summary | Date |
|----------|----------|---------|------|
| @user | APPROVED / CHANGES_REQUESTED / COMMENTED | brief summary | date |

### Inline Code Comments (<count> total)
Group by file, then show:
- **<file-path>** (line <N>): @user — <comment summary>
  - Reply: @user2 — <reply summary>

### Unresolved Threads
List threads that still have open CHANGES_REQUESTED or unanswered questions.

### Conversation Comments (<count>)
Brief chronological summary of top-level PR comments.
```

IMPORTANT: Distinguish between review decisions (APPROVED, CHANGES_REQUESTED) and individual review comments. Group
inline comments by file for readability. Highlight unresolved threads prominently.

### Update Branch

Update PR branch with base:

1. Check current merge status: `gh pr view <number> --json mergeStateStatus`
2. Update: `gh api repos/{owner}/{repo}/pulls/<number>/update-branch -X PUT`
3. If API fails, fall back to local rebase/merge

### Actions / CI Checks

Show detailed CI/actions status:

1. `gh pr checks <number>` — check statuses
2. `gh run list --branch <head-branch>` — recent workflow runs
3. For failed checks, show logs: `gh run view <run-id> --log-failed`

---

## Guidelines

1. **Always use `gh` CLI** — never construct raw URLs unless `gh api` is needed for features not in `gh pr`
2. **Auto-detect PR number** — if not provided, try `gh pr view --json number -q .number` for current branch
3. **Confirm destructive operations** — merge and close require user confirmation through AskUserQuestion, showing
   relevant PR details and warnings about consequences
4. **Challenge risky actions** — warn when merging with failing checks, closing with unresolved comments
5. **Use defaults** — don't ask questions when sensible defaults exist; infer from context
6. **Show results clearly** — use structured markdown output for status and summaries
7. **Handle errors gracefully** — if a `gh` command fails, explain what went wrong and suggest fixes (e.g., "not
   authenticated" → `gh auth login`)
8. **Conventional commit support** — when creating PRs, parse commit messages for title/body and recognize conventional
   commit types for better summaries.
9. **Concise description** -- when creating or modifying PRs, generate PR descriptions that summarize the changes
   clearly and concisely, using commit messages and any provided input.
10. **Use additional context** - if user provided additional information like Jira ticket, or it has a specification
    file, use that to enrich the PR description or status summary.