# Dotfiles Bootstrap

A quick, repeatable way to set up a new machine with my shell, tools, and configs using chezmoi, Homebrew, and mise.

> Supported: macOS (Apple Silicon & Intel), Linux, and WSL.

## WSL2+Ubuntu

```sh
# Install OS-level prerequisites before Homebrew
sudo apt-get update
sudo apt-get install -y zsh ca-certificates curl git gnupg build-essential procps file
```

Chezmoi also runs an idempotent Linux apt essentials script during `chezmoi
apply`, but this manual step ensures the first bootstrap has the tools needed to
install Homebrew and chezmoi.

For unattended setup on a trusted personal machine, configure temporary
passwordless sudo separately with `visudo` instead of managing a sudo rule in
this repo:

```sh
sudo visudo -f /etc/sudoers.d/dont-prompt-$USER-for-sudo-password
```

Add this broad rule only for the duration of the bootstrap, replacing
`yourusername` with the Linux username:

```sudoers
yourusername ALL=(ALL:ALL) NOPASSWD: ALL
```

Save and exit, then verify:

```sh
sudo chmod 0440 /etc/sudoers.d/dont-prompt-$USER-for-sudo-password
sudo -l
sudo true
```

After bootstrap, remove the temporary rule:

```sh
sudo rm /etc/sudoers.d/dont-prompt-$USER-for-sudo-password
```

For normal setup, let `sudo apt-get ...` prompt for the password.

## Bootstrap

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew install chezmoi

chezmoi init https://github.com/weinong/dotfiles --apply
```

### Headless / Non-Interactive Install

For automated setups (CI, scripts, servers), use `NONINTERACTIVE=1` to skip prompts:

```sh
NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add Homebrew to PATH (Linux/WSL)
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

brew install chezmoi

chezmoi init https://github.com/weinong/dotfiles --apply
```

## Daily use

- Update packages: `brew update && brew upgrade`
- Apply dotfiles changes: `chezmoi apply`
- Apply specific files or directories: `chezmoi apply ~/.config/opencode/AGENTS.md`
- Preview changes before applying: `chezmoi diff ~/.config/opencode/`
- Update mise tools: `mise install` (reinstalls missing/updated tool versions)

> **Tip:** Paths passed to `chezmoi apply` and `chezmoi diff` are **target** (destination) paths, not source paths from this repo.

## Using Yubikey in WSL

### Enable systemd inside WSL2

In WSL, create `/etc/wsl.conf` with:

```
[boot]
systemd=true
```

Restart WSL in Windows command line.

```
wsl --shutdown
```

### Install Packages

```sh
sudo apt-get update
sudo apt-get install -y gnupg2 scdaemon pcscd yubikey-manager pinentry-curses
sudo systemctl enable --now pcscd
```

### Configure Pinentry

```sh
mkdir -p ~/.gnupg && chmod 700 ~/.gnupg
echo "pinentry-program /usr/bin/pinentry-curses" >> ~/.gnupg/gpg-agent.conf
gpgconf --kill gpg-agent
```

### Automatically attach yubikey to WSL

https://learn.microsoft.com/en-us/windows/wsl/connect-usb

Open Powershell in Admin mode (one time):
```ps
# install usbipd-win
winget install --interactive --exact dorssel.usbipd-win

# find the BUSID
usbipd list

# Before attaching the USB device, the command usbipd bind must be used to share the device, allowing it to be attached to WSL.
usbipd bind --busid 2-1
```

Run this in the terminal
```ps
usbipd attach --wsl Ubuntu --busid 2-1 --auto-attach --unplugged
```

In WSL, verify

```sh
gpg --card-status
gpg --clearsign
```

## Packages to Install on Windows

```
winget install win32yank
# restart shell in wsl
```
