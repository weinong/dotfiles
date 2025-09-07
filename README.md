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

- Update packages: `brew update && brew upgrade`
- Apply dotfiles changes: `chezmoi apply`
- Update mise tools: `mise install` (reinstalls missing/updated tool versions)

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

# find the BUSID
usbipd wsl list

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
