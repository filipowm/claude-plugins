---
name: setup
description: 'Set up GitHub CLI (gh): install, authenticate, and verify. Use when user needs to install gh, fix gh authentication, or configure GitHub CLI from scratch.'
---

# GitHub CLI Setup

Automatically install, authenticate, and verify GitHub CLI (`gh`). Run all steps without asking questions — use sensible defaults throughout.

## Workflow

Execute each step sequentially. Skip steps that are already satisfied.

### Step 1: Check if `gh` is installed

Run `which gh` or `gh --version`.

- **If installed:** Print version, move to Step 3.
- **If not installed:** Proceed to Step 2.

### Step 2: Install `gh`

Detect the platform and install using the appropriate package manager:

| Platform | Command |
|----------|---------|
| macOS (Homebrew) | `brew install gh` |
| macOS (no Homebrew) | Install Homebrew first: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`, then `brew install gh` |
| Ubuntu/Debian | `sudo apt install gh` (if available) or install from GitHub's apt repo |
| Fedora/RHEL | `sudo dnf install gh` |
| Arch Linux | `sudo pacman -S github-cli` |
| Windows (winget) | `winget install --id GitHub.cli` |
| Windows (scoop) | `scoop install gh` |

For Linux distros where `gh` is not in default repos, use the official GitHub apt/yum repository:

```bash
# Debian/Ubuntu
(type -p wget >/dev/null || sudo apt-get install wget -y) \
  && sudo mkdir -p -m 755 /etc/apt/keyrings \
  && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  && cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && sudo apt update \
  && sudo apt install gh -y
```

After installation, verify with `gh --version`.

If installation fails, report the error and stop. Do NOT retry.

### Step 3: Check authentication

Run `gh auth status`.

- **If authenticated:** Print auth status, move to Step 4.
- **If not authenticated:** Run `gh auth login` with defaults (GitHub.com, HTTPS protocol, browser-based auth). This will open a browser for the user to complete OAuth.

IMPORTANT: After `gh auth login`, the user must complete the browser-based authentication flow. Wait for the command to finish before proceeding.

### Step 4: Verify setup

Run these verification commands:

1. `gh auth status` — confirm authenticated
2. `gh api user --jq .login` — confirm API access works, print username

Print a summary:

```
GitHub CLI setup complete:
- gh version: <version>
- Authenticated as: <username>
- Protocol: HTTPS
```

If any verification step fails, report the specific error and suggest the fix (e.g., "Run `gh auth login` to re-authenticate").
