# Linux Desktop Setup

My personal runbooks for turning a fresh Arch Linux or Ubuntu install into a GNOME desktop for daily work. Each one is a checklist you follow by hand, not an install script. Every step shows the commands to run or the config to paste.

> Values such as `<username>`, `you@example.com`, `~/workspace/work/`, `<uuid>` and `<device_mac>` are placeholders. Replace them with your own.

## Files

| File                                           | Description                                                                                                |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| [setup-arch.md](setup-arch.md)                 | The full step-by-step runbook for Arch Linux. This is the base guide                                       |
| [setup-ubuntu-26.04.md](setup-ubuntu-26.04.md) | Ubuntu 26.04 LTS version. Same section numbers; identical steps link back to the Arch guide                |
| [starship-dracula.toml](starship-dracula.toml) | [Starship](https://starship.rs) prompt config with a Dracula palette. Copy it to `~/.config/starship.toml` |

## What the runbooks cover

The list below follows the Arch guide. See [Ubuntu differences](#ubuntu-2604-differences) for what changes on Ubuntu.

### **1. Post-install (first boot)**

- Wi-Fi with `nmtui`, a `sudo` user in the `wheel` group, GNOME + GDM

### **2. System setup**

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
- Wi-Fi tuning: power save off, country code set to VN

### **3. Applications**

- Docker Engine, VSCode (Vim mode tweaks), Claude Code CLI
- DBeaver, Node.js, Postman, AnyDesk (forced to X11 on Wayland)
- Separate work and personal Git profiles using `includeIf` and one SSH key per profile
- Fingerprint (`fprintd`) and face recognition (`howdy`, IR camera) for `sudo` through PAM
- Hibernate with a swapfile, resume parameters and a GNOME power menu button
- Remote access with GNOME Remote Desktop (Remote Login + Desktop Sharing) over Tailscale, no open router ports

**4. TODOs:** AWS CLI v2, Go, MCP for AI, light-mode theme, and a script that applies the whole setup.

## Ubuntu 26.04 differences

- **Skipped:** user/GNOME bootstrap (done by the installer), `linux-lts`, pacman tweaks, `yay`
- **Added:** a software sources policy (apt → vendor repo/`.deb` → Snap, no Flatpak), drivers, telemetry opt-out, Ubuntu Pro, Timeshift snapshots, upgrading from 24.04
- **Changed:** apps come from vendor repos (Chrome, VSCode, Docker, AnyDesk, Lotus, Tailscale) or Snap (Pinta, Postman, DBeaver); fcitx5 is enabled with `im-config`; `bat`/`fd` are `batcat`/`fdfind`; fingerprint is enabled with `pam-auth-update`; Howdy uses an unofficial PPA; hibernate uses GRUB + `initramfs-tools`, needs a polkit rule for the GNOME button, and does not work with Secure Boot enabled

## Usage

1. Install Arch Linux (e.g. with `archinstall`, using systemd-boot) or Ubuntu Desktop 26.04 LTS.
2. Follow [setup-arch.md](setup-arch.md) or [setup-ubuntu-26.04.md](setup-ubuntu-26.04.md) from top to bottom, skipping what you don't need.
3. Hardware-specific steps (fingerprint, face recognition, hibernate) begin with checks. Run those checks before changing anything.

> ⚠️ Some steps edit PAM, the bootloader, `/etc/fstab` and kernel parameters. Keep a root shell open in a second terminal and have a live USB (or, on Ubuntu, a Timeshift snapshot) ready before you change them.
