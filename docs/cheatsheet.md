# Dotfiles Cheatsheet

A quick reference for the keybindings, aliases, and tools provided by this
dotfiles repo. Everything below is sourced from the actual config files; if a
binding is missing here it isn't configured.

**Conventions**

- `C-x` = `Ctrl+x`, `S-X` = `Shift+X`, `M-x` = `Alt+x`.
- `<prefix>` for tmux is the default `C-b` (this repo doesn't override it).
- Platform tags: **(zsh)** macOS/Linux/WSL, **(pwsh)** Windows PowerShell 7,
  **(tmux)** any platform with tmux.

---

## tmux

Source: [`dot_tmux.conf`](../dot_tmux.conf). Prefix is the default `C-b`.

### Panes

| Keys                | Action                                          |
|---------------------|-------------------------------------------------|
| `<prefix> \|`       | Split pane left/right — new pane on the right (`split-window -h`); opens in current dir |
| `<prefix> -`        | Split pane top/bottom — new pane below (`split-window -v`); opens in current dir |
| `C-h` / `C-j` / `C-k` / `C-l` | Move focus left / down / up / right (no prefix) |
| `S-H` / `S-J` / `S-K` / `S-L` | Resize pane left / down / up / right by 8/4 cells (no prefix) |
| `<prefix> S`        | Toggle `synchronize-panes` (broadcast input)    |

### Windows & sessions

| Keys                | Action                                          |
|---------------------|-------------------------------------------------|
| `<prefix> c`        | New window in current pane's directory          |
| `<prefix> ,`        | Rename current window (prompt pre-filled)       |
| Mouse drag / click  | Resize pane / focus pane (mouse mode is on)     |

`allow-rename` is **off**, so apps can't silently rename your windows.
`detach-on-destroy` is **on**, so closing the last window detaches cleanly.
Scrollback is `100,000` lines.

### Copy mode (vi)

`mode-keys` is `vi`.

| Keys (in copy-mode-vi) | Action                                       |
|------------------------|----------------------------------------------|
| `<prefix> [`           | Enter copy mode                              |
| `v`                    | Begin selection (vi visual)                  |
| `y`                    | Copy selection to system clipboard via `~/.tmux/copy` |
| `q`                    | Exit copy mode                               |

`set-clipboard on` enables OSC 52 copying for terminals that support it.

### Plugins (TPM)

Installed via `~/.tmux/plugins/tpm`:

- `tmux-sensible` — sane defaults.
- `tmux-yank` — clipboard helpers.
- `tmux-resurrect` — save/restore sessions.
- `tmux-continuum` — auto-save sessions; `@continuum-restore = on` auto-restores
  on tmux start.

Reload config: `<prefix> :source-file ~/.tmux.conf` or `tmux source ~/.tmux.conf`.

---

## fzf

Defaults are loaded by `fzf --zsh` (zsh) and PSFzf (pwsh). Custom commands live
in [`dot_zshrc.tmpl`](../dot_zshrc.tmpl) and the PowerShell profile installer
[`run_onchange_after_15-install-powershell-profile.ps1.tmpl`](../run_onchange_after_15-install-powershell-profile.ps1.tmpl).

### Built-in keybindings

| Keys     | zsh                                  | pwsh (PSFzf)                  |
|----------|--------------------------------------|-------------------------------|
| `C-t`    | Insert selected file path            | Insert selected file path     |
| `C-r`    | Search shell history                 | Search shell history          |
| `M-c`    | `cd` into selected directory         | (not bound)                   |

Inside fzf: `Tab` to multi-select, `Enter` to accept, `C-c`/`Esc` to cancel.

### Custom keybindings (zsh & pwsh)

| Keys  | Action                                                           |
|-------|------------------------------------------------------------------|
| `C-f` | Pick a file with fzf, open in `$EDITOR` (`nvim` by default)      |
| `C-g` | Live `ripgrep` search; `Enter` opens match in `$EDITOR +line`    |

`C-g` requires `ripgrep`. Preview uses `bat` if installed (with line
highlighting), otherwise `sed`/`Get-Content`.

### Source commands & previews

- `FZF_DEFAULT_COMMAND` uses `fd --type f --hidden --follow --exclude .git`
  (falls back to `rg --files --hidden --follow --no-ignore-vcs`).
- `FZF_ALT_C_COMMAND` lists directories with `fd` (or `rg | dirname`).
- `FZF_DEFAULT_OPTS`: 40% height, reverse layout, border + margin, ANSI on,
  inline info, right-side 60% preview pane.
- Default file preview (`FZF_PREVIEW_CMD`): `bat --style=plain --color=always
  --line-range=:500`.

### fzf-tab (completion menu)

- `Tab` cycles completions in an fzf-style menu.
- `,` / `.` switch between completion groups (`zstyle ':fzf-tab:*' switch-group`).
- File/path completion shows a `bat` preview.

---

## zsh

Source: [`dot_zshrc.tmpl`](../dot_zshrc.tmpl) and
[`dot_zsh_plugins.txt`](../dot_zsh_plugins.txt) (loaded via antidote).

### Plugins

- `zsh-users/zsh-completions` — extra completion definitions.
- `Aloxaf/fzf-tab` — fzf-powered tab completion (see above).
- `wfxr/forgit` — interactive git via fzf (see Forgit table below).
- `zsh-users/zsh-autosuggestions` — fish-like inline suggestion.
  - `→` (Right Arrow) or `End` to accept the suggestion.
- `zsh-users/zsh-syntax-highlighting` — colorizes the command line.
- `jeffreytse/zsh-vi-mode` — modal editing on the command line
  (`ZVM_INIT_MODE=sourcing`).

### zsh-vi-mode quick keys

| Keys           | Action                                              |
|----------------|-----------------------------------------------------|
| `Esc`          | Switch from insert to normal mode                   |
| `i` / `a` / `I` / `A` | Enter insert (at cursor / after / line start / line end) |
| `v`            | Visual mode (select); `y` to yank, `d` to delete    |
| `dd` / `cc`    | Delete / change whole line                          |
| `u` / `C-r`    | Undo / redo (note: `C-r` is overridden by fzf history when in insert mode) |
| `/` `pattern`  | Search history backward                             |

### History

- `HISTFILE = ~/.zsh_history`, `HISTSIZE` and `SAVEHIST` = `200,000`.
- `appendhistory incappendhistory extendedhistory` — append immediately with
  timestamps.
- `histignoredups histreduceblanks histverify` — dedup, trim blanks, confirm
  expansions before running.

### Forgit (git via fzf)

Loaded automatically by the `wfxr/forgit` plugin. Most-used commands:

| Alias    | What it runs                                       |
|----------|----------------------------------------------------|
| `ga`     | `git add` — pick hunks/files interactively         |
| `glo`    | `git log` — browse commits with diff preview       |
| `gd`     | `git diff` — pick a file/commit to diff            |
| `gi`     | Generate `.gitignore` interactively                |
| `gcf`    | `git checkout <file>` — pick file to discard       |
| `gco`    | `git checkout <branch/tag/commit>`                 |
| `gss`    | `git stash show` — browse stashes                  |
| `grh`    | `git reset HEAD` — unstage interactively           |
| `gclean` | `git clean` — pick untracked files to remove       |
| `gbl`    | `git blame` — pick file to blame                   |

Run `forgit` with no args (or check the [forgit README](https://github.com/wfxr/forgit#-shell-aliases))
for the full list.

### Misc shell options

`autocd extendedglob interactivecomments` — type a path to `cd` into it,
`**` recursive globbing, `# comments` allowed at the prompt.

---

## Aliases & shell helpers

Defined in [`dot_zshrc.tmpl`](../dot_zshrc.tmpl) and the PowerShell profile.
PowerShell entries only exist when the underlying tool is installed.

| Alias / function | zsh                                          | pwsh                                            |
|------------------|----------------------------------------------|-------------------------------------------------|
| `vi`, `vim`      | `nvim`                                       | function → `nvim` (auto-discovers `nvim.exe`)   |
| `lg`             | `lazygit`                                    | `Set-Alias lg lazygit`                          |
| `l`              | `eza`                                        | function → `eza` (fallback `Get-ChildItem`)     |
| `la`             | `eza -a`                                     | function → `eza -a`                             |
| `ll`             | `eza -lah`                                   | function → `eza -lah`                           |
| `ls`             | `eza --color=auto`                           | (built-in unchanged)                            |
| `cat`            | `bat -P --style=plain`                       | `Set-Alias cat bat`                             |
| `z <dir>`        | `zoxide` jump (frecency-based `cd`)          | same                                            |
| `zi`             | `zoxide` interactive picker                  | same                                            |

Other env wired in both shells:

- `EDITOR=nvim`, `VISUAL=nvim`.
- `GPG_TTY=$(tty)` (zsh) for GPG/Yubikey pinentry.
- `kubectl completion zsh` is sourced if `kubectl` is on `PATH`.

---

## PowerShell-specific

Source: [`run_onchange_after_15-install-powershell-profile.ps1.tmpl`](../run_onchange_after_15-install-powershell-profile.ps1.tmpl).
The profile is installed to `$PROFILE.CurrentUserCurrentHost`.

### PSReadLine

- Edit mode: **Vi**.
- History: `HistoryNoDuplicates`.
- Prediction: `HistoryAndPlugin` source, `InlineView` style (when running in a
  real ConsoleHost). `→` accepts the suggestion.

### PSFzf chords

| Keys   | Action                                  |
|--------|-----------------------------------------|
| `C-t`  | Insert path picker (PSFzf provider)     |
| `C-r`  | History picker (PSFzf reverse history)  |
| `C-f`  | `Edit-FzfFile` — pick file → `$EDITOR`  |
| `C-g`  | `Invoke-FzfLiveGrep` — ripgrep + open match in `$EDITOR +line` |

### Notable functions

- `Get-NeovimCommand` — locates `nvim` in PATH or common install dirs.
- `vi` / `vim` — wrappers around the resolved `nvim`.
- `l` / `la` / `ll` — eza wrappers (fallback to `Get-ChildItem`).
- `Edit-FzfFile`, `Invoke-FzfLiveGrep` — fzf helpers used by the chords above.

---

## CLI tools installed

From [`Brewfile.tmpl`](../Brewfile.tmpl) (macOS/Linux/WSL via Homebrew) and
the Windows installer scripts. mise-managed languages live in
[`mise.toml`](../mise.toml).

### Search, navigation, view

| Tool        | What it's for                                         |
|-------------|-------------------------------------------------------|
| `fzf`       | Fuzzy finder (powers Ctrl-T/R, Ctrl-F/G, fzf-tab)     |
| `fd`        | Fast `find` replacement (default fzf source)          |
| `ripgrep` (`rg`) | Fast recursive grep (powers Ctrl-G live grep)    |
| `bat`       | `cat` with syntax highlight + paging (also fzf preview) |
| `eza`       | Modern `ls` (powers `l`/`la`/`ll`)                    |
| `zoxide` (`z`, `zi`) | Frecency-based `cd`                          |
| `starship`  | Cross-shell prompt (config in `dot_config/starship.toml`) |

### Git / dev

| Tool         | What it's for                                        |
|--------------|------------------------------------------------------|
| `gh`         | GitHub CLI (auth, PRs, issues, repos)                |
| `git-delta`  | Better git diff pager                                |
| `lazygit` (`lg`) | Terminal UI for git                              |
| `neovim` (`nvim`) | Editor; LazyVim starter is installed by chezmoi |
| `luarocks`   | Lua package manager (used by some nvim plugins)      |
| `gcc`        | Required for compiling treesitter parsers            |

### Cloud / Kubernetes

| Tool             | What it's for                                    |
|------------------|--------------------------------------------------|
| `kubectl`        | Kubernetes CLI (zsh completion auto-loaded)      |
| `kubelogin`      | Azure AD auth plugin for kubectl                 |
| `openshift-cli` (`oc`) | OpenShift CLI                              |
| `k9s`            | Terminal UI for Kubernetes                       |

### Data wrangling & misc

| Tool       | What it's for                                          |
|------------|--------------------------------------------------------|
| `jq`       | JSON processor                                         |
| `yq`       | YAML processor                                         |
| `wget`     | HTTP downloader                                        |
| `watch`    | Repeat a command and show output                       |
| `tmux`     | Terminal multiplexer (config in `dot_tmux.conf`)       |
| `chezmoi`  | This dotfiles manager                                  |
| `chruby`   | Ruby version switcher                                  |
| `antidote` | zsh plugin manager (loads `dot_zsh_plugins.txt`)       |
| `mise`     | Polyglot version manager (Go/Python/Node, see below)   |

### Languages (mise)

Installed on `chezmoi apply` via `mise install`:

- Go `1.25` — `GOPATH=$HOME/go`, `$GOPATH/bin` on `PATH`.
- Python `3.12` plus `uv` (latest) for fast venv/package management.
- Node.js `24` (npm + npx).

---

## chezmoi workflow

| Command                                  | What it does                            |
|------------------------------------------|-----------------------------------------|
| `chezmoi apply`                          | Apply changes to the home directory     |
| `chezmoi diff`                           | Preview pending changes (dry run)       |
| `chezmoi diff <target>`                  | Diff a specific destination path        |
| `chezmoi edit <target>`                  | Edit a managed file in source repo      |
| `chezmoi add <file>`                     | Bring a file under chezmoi management   |
| `chezmoi managed`                        | List every file chezmoi manages         |
| `chezmoi cd`                             | `cd` into the source repo               |
| `chezmoi state delete-bucket --bucket=scriptState` | Re-run `run_once_*` scripts   |

Paths passed to `chezmoi apply`/`diff` are **target** (destination) paths,
not source paths in this repo.

---

## Upstream references

- tmux: <https://github.com/tmux/tmux/wiki>
- TPM: <https://github.com/tmux-plugins/tpm>
- fzf: <https://github.com/junegunn/fzf>
- fzf-tab: <https://github.com/Aloxaf/fzf-tab>
- forgit: <https://github.com/wfxr/forgit>
- zsh-vi-mode: <https://github.com/jeffreytse/zsh-vi-mode>
- antidote: <https://getantidote.github.io/>
- zoxide: <https://github.com/ajeetdsouza/zoxide>
- starship: <https://starship.rs/>
- bat: <https://github.com/sharkdp/bat>
- eza: <https://github.com/eza-community/eza>
- ripgrep: <https://github.com/BurntSushi/ripgrep>
- fd: <https://github.com/sharkdp/fd>
- lazygit: <https://github.com/jesseduffield/lazygit>
- LazyVim: <https://www.lazyvim.org/>
- mise: <https://mise.jdx.dev/>
- chezmoi: <https://www.chezmoi.io/>
