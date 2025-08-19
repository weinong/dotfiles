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

### Automatically attach yubikey to WSL

https://learn.microsoft.com/en-us/windows/wsl/connect-usb

Open Powershell in Admin mode (one time):
```ps

# find the BUSID
usbipd wsl list

# Before attaching the USB device, the command usbipd bind must be used to share the device, allowing it to be attached to WSL.
usbipd bind --busid 2-1
```

Every time WSL is rebooted:

```ps
usbipd attach --wsl Ubuntu --busid 2-1 --auto-attach --unplugged
```

In WSL, verify

```sh
gpg --card-status
```
