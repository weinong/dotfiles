# Dotfiles Bootstrap

A quick, repeatable way to set up a new machine with my shell, tools, and configs using chezmoi, Homebrew, and mise.

> Supported: macOS (Apple Silicon & Intel), Linux, and WSL.

## Bootstrap

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew install chezmoi

chezmoi init https://github.com/weinong/dotfiles --apply
```

## Daily use

- Update packages: brew update && brew upgrade
- Apply dotfiles changes: chezmoi apply
- Update zgenom plugins: zgenom update  (pull latest for all plugins)
  - Update zgenom itself: zgenom selfupdate
  - Rebuild plugin cache if needed: zgenom reset && source ~/.zshrc (or exec zsh)
  - Optional auto-update: add export ZGENOM_AUTOUPDATE_DAYS=7 to your .zshrc for weekly checks
- Update mise tools: mise install (reinstalls missing/updated tool versions)