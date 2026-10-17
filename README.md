# Arch Linux Setup

My personal runbook for turning a fresh Arch Linux install into a GNOME desktop for daily work. It is a checklist you follow by hand, not an install script. Every step shows the commands to run or the config to paste.

> Values such as `<username>`, `you@example.com`, `~/workspace/work/`, `<uuid>` and `<device_mac>` are placeholders. Replace them with your own.

## Files

| File                                           | Description                                                                                                |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| [setup-arch.md](setup-arch.md)                 | The full step-by-step runbook                                                                              |
| [starship-dracula.toml](starship-dracula.toml) | [Starship](https://starship.rs) prompt config with a Dracula palette. Copy it to `~/.config/starship.toml` |

## What the runbook covers

**1. Post-install (first boot)**

- Wi-Fi with `nmtui`, a `sudo` user in the `wheel` group, GNOME + GDM

**2. System setup**

- Base packages, LibreOffice, image editors, `translate-shell`
- `linux-lts` as a fallback kernel (systemd-boot + UKI)
- Pacman cache cleanup timer, sudo timeout, pacman quality-of-life tweaks
- Vim config, `yay` (AUR helper), firmware updates with `fwupd`
- Fonts and multimedia codecs
- Bluetooth setup and pairing troubleshooting
- Google Chrome (via AUR, not Flatpak) with per-profile desktop launchers
- Recommended GNOME extensions
- Vietnamese input with `fcitx5` + Lotus
- Terminal: `kitty` + `fish` + `starship` + Nerd Font, plus `fzf`, `zoxide`, `eza`, `bat`, `ripgrep`, `fd`
- Dracula GTK theme, Papirus icons, Bibata cursor

**3. Applications**

- Docker Engine, VSCode (Vim mode tweaks), Claude Code CLI
- DBeaver, Node.js, Postman, AnyDesk (forced to X11 on Wayland)
- Separate work and personal Git profiles using `includeIf` and one SSH key per profile
- Fingerprint (`fprintd`) and face recognition (`howdy`, IR camera) for `sudo` through PAM
- Hibernate with a swapfile, resume parameters and a GNOME power menu button

**4. TODOs:** AWS CLI v2, Go, MCP for AI, light-mode theme, and a script that applies the whole setup.

## Usage

1. Install Arch Linux (e.g. with `archinstall`) using systemd-boot.
2. Follow [setup-arch.md](setup-arch.md) from top to bottom, skipping what you don't need.
3. Hardware-specific steps (fingerprint, face recognition, hibernate) begin with checks. Run those checks before changing anything.

> ⚠️ Some steps edit PAM, the bootloader, `/etc/fstab` and kernel parameters. Keep a root shell open in a second terminal and have a live USB ready before you change them.
