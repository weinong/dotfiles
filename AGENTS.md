# AGENTS.md - Coding Agent Instructions

This is a **chezmoi dotfiles repository** for managing personal development environment
configuration across macOS (Apple Silicon & Intel), Linux, and WSL.

## Repository Overview

- **Purpose**: Automated machine setup with shell config, tools, and AI assistant rules
- **Technology**: Shell scripts, TOML configs, chezmoi templates (Go text/template)
- **Target**: Personal dotfiles - not a traditional software project with build/test cycles

## Commands Reference

### Chezmoi Workflow

| Action                      | Command                                              |
|-----------------------------|------------------------------------------------------|
| Bootstrap new machine       | `chezmoi init https://github.com/weinong/dotfiles --apply` |
| Apply dotfile changes       | `chezmoi apply`                                      |
| Preview changes (dry-run)   | `chezmoi diff`                                       |
| Edit source file            | `chezmoi edit <target-file>`                         |
| Add file to management      | `chezmoi add <file>`                                 |
| Check managed files         | `chezmoi managed`                                    |
| Re-run run_once scripts     | Delete state: `chezmoi state delete-bucket --bucket=scriptState` |

### Package Management

| Action                      | Command                         |
|-----------------------------|---------------------------------|
| Update Homebrew packages    | `brew update && brew upgrade`   |
| Install Brewfile packages   | `brew bundle --file=~/Brewfile` |
| Install/update mise tools   | `mise install`                  |
| List mise tools             | `mise list`                     |

### No Build/Test Commands

This is a configuration repository. There are no build, lint, or test commands.
Validation is done via `chezmoi diff` and manual testing after `chezmoi apply`.

## Directory Structure

```
├── .chezmoitemplates/        # Shared template partials (reusable snippets)
├── dot_claude/               # Claude AI config (becomes ~/.claude/)
├── dot_config/               # XDG config (becomes ~/.config/)
├── dot_github/               # GitHub config (becomes ~/.github/)
├── run_once_*.sh.tmpl        # One-time setup scripts (run by chezmoi)
├── Brewfile.tmpl             # Homebrew packages (templated)
├── mise.toml                 # mise tool versions (Go, Python, Node)
└── dot_zshrc.tmpl            # Main shell configuration
```

## Chezmoi Template Conventions

### File Naming

- `dot_` prefix → `.` in target (e.g., `dot_zshrc` → `.zshrc`)
- `.tmpl` suffix → processed as Go template
- `run_once_` prefix → executed once per machine
- `run_once_after_` → runs after file application
- Numeric prefix for ordering (e.g., `10-`, `20-`, `30-`)

### Template Syntax

```go
{{ .chezmoi.homeDir }}        // Home directory path
{{ .chezmoi.os }}             // "darwin", "linux", "windows"
{{ .chezmoi.arch }}           // "amd64", "arm64"

{{- template "global-instructions.md" . -}}   // Include shared template

{{- if eq .chezmoi.os "darwin" }}
# macOS specific
{{- end }}
```

## Shell Script Style Guide

### Shebang and Strict Mode

```bash
#!/usr/bin/env bash
set -euo pipefail    # exit on error, undefined vars, pipe failures
```

### Tool and Platform Detection

```bash
# Tool detection
if command -v mise >/dev/null 2>&1; then eval "$(mise activate zsh)"; fi

# Homebrew path (cross-platform)
if [ -x /opt/homebrew/bin/brew ]; then eval "$(/opt/homebrew/bin/brew shellenv)"   # ARM
elif [ -x /usr/local/bin/brew ]; then eval "$(/usr/local/bin/brew shellenv)"       # Intel
elif [ -x /home/linuxbrew/.linuxbrew/bin/brew ]; then eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
fi
```

### Naming Conventions

| Type           | Convention      | Example                    |
|----------------|-----------------|----------------------------|
| Functions      | `snake_case`    | `_fzf_open_in_editor()`    |
| Local vars     | `lower_case`    | `local file line`          |
| Constants      | `UPPER_CASE`    | `HISTSIZE=200000`          |
| Exported vars  | `UPPER_CASE`    | `export EDITOR=nvim`       |

### Error Handling

```bash
brew update || true                        # Graceful fallback

if ! command -v zsh >/dev/null 2>&1; then  # Required command
  echo "[chezmoi] ERROR: zsh is required." >&2; exit 1
fi
```

## Branch Hygiene

Before making any code changes, always start from a clean state:

1. Determine the default branch (`main` or `master`):
   `git symbolic-ref refs/remotes/origin/HEAD`
2. Fetch the latest: `git fetch origin`
3. Check out the default branch: `git checkout <default-branch>`
4. Reset to latest remote: `git reset --hard origin/<default-branch>`
5. Create a new feature branch: `git checkout -b <descriptive-branch-name>`

Never make changes on a stale or dirty branch.

## Pre-Commit Code Review

Before creating any git commit:

1. Generate the diff: `git diff HEAD`
2. Invoke the `code-reviewer` sub-agent via the Task tool
3. If review identifies **Critical** or **High** severity issues, fix them first
4. Once all critical/high issues are resolved, proceed with the commit

Never skip the review step.

## Code Review Focus Areas

When reviewing changes (or acting as code-reviewer agent):

- **Code correctness** - Logic errors, edge cases
- **Project conventions** - Follow patterns in existing files
- **Security** - No secrets in templates, safe defaults
- **Shell best practices** - Quoting, error handling, portability
- **Template syntax** - Valid Go template expressions

Severity levels: **Critical** | **High** | **Medium** (ignore Low)

## Security Rules

**NEVER commit sensitive data.** Before every commit, verify no files contain:

- **Secrets**: API keys, tokens, passwords, private keys, certificates
- **PII**: Email addresses, names, phone numbers, addresses, SSNs
- **Internal identifiers**: Employee IDs, internal hostnames, IP addresses

### Patterns to Avoid

```bash
# NEVER hardcode these - use environment variables or secret managers
export API_KEY="sk-..."           # ❌ Secrets in dotfiles
export MY_EMAIL="name@corp.com"   # ❌ PII in config

# Use placeholders or env vars instead
export API_KEY="${MY_API_KEY:-}"  # ✓ Reference env var
```

### Pre-Commit Checklist

1. `git diff --cached` - Review all staged changes
2. Search for patterns: `grep -rE '(password|secret|key|token|email|credential|auth|private|api_|ghp_|sk-)' .`
3. Check for hardcoded values in `.tmpl` files
4. Verify `.chezmoiignore` excludes sensitive local files

If secrets are accidentally committed, **rotate them immediately** - git history preserves all data.

## Key Files Reference

| Purpose                  | File                                      |
|--------------------------|-------------------------------------------|
| Main shell config        | `dot_zshrc.tmpl`                          |
| Homebrew packages        | `Brewfile.tmpl`                           |
| Tool versions            | `mise.toml`                               |
| Global AI instructions   | `.chezmoitemplates/global-instructions.md`|
| Code review guidelines   | `.chezmoitemplates/code-reviewer-body.md` |
| Tmux config              | `dot_tmux.conf`                           |
| Starship prompt          | `dot_config/starship.toml`                |
