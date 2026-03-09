# github

Skills and tools for common GitHub operations: PRs, issues, code reviews. Built around GitHub CLI (gh).

## Features

- Pull request management (create, review, merge, diff)
- Issue tracking (create, list, comment, close)
- Code review workflows with inline comments
- All operations via `gh` CLI

## Installation

```bash
/plugin marketplace add MARKETPLACE_REPO
/plugin install github@filipowm-plugins
```

## Usage

```
/github:pull-request create a PR for the current branch
/github:pull-request provide a summary of the changes PR #123
/github:configure-github
```

## Configuration

Requires GitHub CLI (`gh`) to be installed and authenticated:

```bash
gh auth login
```

## Development

Test locally:
```bash
claude --plugin-dir /path/to/github
```
