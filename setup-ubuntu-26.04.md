# SETUP UBUNTU 26.04 LTS

> **Version:** Ubuntu Desktop 26.04 LTS (Resolute Raccoon), GNOME 50, Wayland only (there is no Xorg session). Package names and versions below were checked against 26.04 and may differ on other releases.
>
> **Base guide:** [setup-arch.md](setup-arch.md). This file uses the same section numbers. Steps that are identical on both distros link to the Arch guide instead of repeating it. Arch-only steps are left out, so some numbers are skipped.

## Table of Contents

- [0. Software sources policy](#0-software-sources-policy)
- [1. SETUP UBUNTU POST-INSTALL (AFTER REBOOTING)](#1-setup-ubuntu-post-install-after-rebooting)
  - [1.4 Update the system](#14-update-the-system)
  - [1.5 Install drivers](#15-install-drivers)
  - [1.6 Telemetry and Ubuntu Pro](#16-telemetry-and-ubuntu-pro)
  - [1.7 System snapshots with Timeshift](#17-system-snapshots-with-timeshift)
  - [1.8 Upgrade from Ubuntu 24.04](#18-upgrade-from-ubuntu-2404)
- [2. SETUP UBUNTU](#2-setup-ubuntu)
  - [2.1 Popular packages and setup](#21-popular-packages-and-setup)
    - [2.1.1 Personalization](#211-personalization)
    - [2.1.3 Schedule clear package cache](#213-schedule-clear-package-cache)
    - [2.1.4 Set sudo session timeout in console](#214-set-sudo-session-timeout-in-console)
    - [2.1.5 Setup Vim](#215-setup-vim)
  - [2.2 Software sources (replaces `yay`)](#22-software-sources-replaces-yay)
  - [2.3 Update firmware](#23-update-firmware)
  - [2.4 Fonts and Codec](#24-fonts-and-codec)
  - [2.5 Setup bluetooth](#25-setup-bluetooth)
  - [2.6 Install Google Chrome](#26-install-google-chrome)
    - [2.6.1 Separate Google profiles](#261-separate-google-profiles)
  - [2.7 Install extensions for GNOME](#27-install-extensions-for-gnome)
  - [2.8 Install `fcitx5`](#28-install-fcitx5)
  - [2.9 Setup console](#29-setup-console)
  - [2.10 Setup themes](#210-setup-themes)
  - [2.11 Wi-Fi tuning](#211-wi-fi-tuning)
    - [2.11.1 Disable Wi-Fi power save](#2111-disable-wi-fi-power-save)
    - [2.11.2 Set Wi-Fi country code](#2112-set-wi-fi-country-code)
- [3. Applications](#3-applications)
  - [3.1 Docker Engine](#31-docker-engine)
  - [3.2 VSCode](#32-vscode)
  - [3.3 Install Claude CLI](#33-install-claude-cli)
  - [3.4 Common Applications](#34-common-applications)
  - [3.5 Setup Git multi-profiles](#35-setup-git-multi-profiles)
  - [3.6 Configure fingerprint (if hardware support)](#36-configure-fingerprint-if-hardware-support)
  - [3.7 Setup face recognition (if hardware support)](#37-setup-face-recognition-if-hardware-support)
  - [3.8 Install Postman](#38-install-postman)
  - [3.9 Install AnyDesk](#39-install-anydesk)
  - [3.10 Enable Hibernate mode](#310-enable-hibernate-mode)
    - [3.10.1 Check system requirement before setup](#3101-check-system-requirement-before-setup)
    - [3.10.2 Resize the Swapfile](#3102-resize-the-swapfile)
    - [3.10.3 Configure resume parameters](#3103-configure-resume-parameters)
    - [3.10.4 Update initramfs](#3104-update-initramfs)
    - [3.10.5 Setup hibernate command](#3105-setup-hibernate-command)
    - [3.10.6 Add "Hibernate" button](#3106-add-hibernate-button)
  - [3.11 Remote access (GNOME Remote Desktop + Tailscale)](#311-remote-access-gnome-remote-desktop--tailscale)
    - [3.11.1 Install packages](#3111-install-packages)
    - [3.11.2 Log in to Tailscale](#3112-log-in-to-tailscale)
    - [3.11.3 Turn on GNOME Remote Desktop](#3113-turn-on-gnome-remote-desktop)
    - [3.11.4 Keep the machine reachable](#3114-keep-the-machine-reachable)
    - [3.11.5 Connect](#3115-connect)
    - [3.11.6 Daily flow](#3116-daily-flow)
- [4. TODOs](#4-todos)

## 0. Software sources policy

Choose the source for each app in this order:

1. **apt (Ubuntu repos):** default for CLI tools and system packages
2. **Vendor apt repo or `.deb`:** for apps that need tight system integration: Chrome, VSCode, Docker, AnyDesk, fcitx5-lotus, Tailscale
3. **Snap:** for desktop apps not in apt, or where apt is too old: Pinta, Postman, DBeaver. Snap is Ubuntu's default, and Firefox, Snap Store and Firmware Updater already ship as snaps, so keep it
4. **Flatpak:** not used, to avoid a third package format

Avoid the snap versions of VSCode (input method issues with fcitx5) and Docker (sandbox limits on mounts and the `docker` group). When adding a vendor repo, store its key in `/etc/apt/keyrings/` and reference it with `signed-by=`, because `apt-key` is deprecated.

## 1. SETUP UBUNTU POST-INSTALL (AFTER REBOOTING)

### 1.4 Update the system

```bash
sudo apt update && sudo apt full-upgrade
sudo snap refresh
```

### 1.5 Install drivers

- List recommended drivers (e.g. NVIDIA) `ubuntu-drivers devices`
- Install them `sudo ubuntu-drivers install`
- Or use the GUI: Software & Updates > Additional Drivers

### 1.6 Telemetry and Ubuntu Pro

- Opt out of the install report `ubuntu-report -f send no`
- Disable crash reporting

  ```bash
  sudo systemctl disable --now apport.service
  sudo sed -i 's/^enabled=1/enabled=0/' /etc/default/apport
  ```

- Optional: Ubuntu Pro (free for up to 5 personal machines) adds extended security updates and Livepatch: `sudo pro attach <token>`, token from <https://ubuntu.com/pro>
- Automatic security updates (`unattended-upgrades`) are on by default. Check with `systemctl status unattended-upgrades`

### 1.7 System snapshots with Timeshift

- Take snapshots before risky changes so you can roll back
- Install `sudo apt install timeshift`
- Open Timeshift > select `RSYNC` > pick a partition with free space > enable daily snapshots
- Take a manual snapshot before editing PAM, GRUB or `/etc/fstab`
- If a kernel update breaks something, boot the previous kernel: hold `Shift` (BIOS) or press `Esc` (UEFI) at startup > Advanced options for Ubuntu

### 1.8 Upgrade from Ubuntu 24.04

- Take a Timeshift snapshot first, then upgrade `sudo do-release-upgrade`
- The upgrade turns off every third-party apt repo. Turn them back on afterwards, or Chrome, VSCode, Docker and Lotus stop getting updates
  - Repos in `.sources` files are only marked `Enabled: no`. Remove that line `sudo sed -i '/^Enabled: no/d' /etc/apt/sources.list.d/*.sources`
  - Repos in old `.list` files are renamed to `*.list.disabled` and commented out. Add them again with the commands in [2.8](#28-install-fcitx5) (Lotus), [3.1](#31-docker-engine) (Docker) and [3.9](#39-install-anydesk) (AnyDesk), then delete the `*.list.disabled` files
  - Check `sudo apt update` shows no errors, then `sudo apt full-upgrade`
- Replace hand-installed fonts with the apt package: see [2.4](#24-fonts-and-codec)
- The session changes from Xorg to Wayland. AnyDesk needs the fix in [3.9](#39-install-anydesk)

## 2. SETUP UBUNTU

### 2.1 Popular packages and setup

- Show `minimize`, `maximize`, `close` button: same as [Arch 2.1](setup-arch.md#21-popular-packages-and-setup) (Ubuntu shows them by default)
- Install build tools and git `sudo apt install build-essential git curl`
- Install gnome-tweaks `sudo apt install gnome-tweaks`
- Install zip, unzip `sudo apt install zip unzip`
- Office: LibreOffice is preinstalled. Add the Vietnamese language pack `sudo apt install libreoffice-l10n-vi hunspell-en-us`
- Install drawing & photo editor

  ```bash
  sudo apt install drawing        # closest to MS Paint, simple
  sudo snap install pinta         # Paint.NET-like, layers (not in apt for 26.04)
  ```

- Install translation shell `sudo apt install translate-shell`

#### 2.1.1 Personalization

- Same as [Arch 2.1.1](setup-arch.md#211-personalization)

#### 2.1.3 Schedule clear package cache

- apt has no cache timer like `paccache`. Clean up from time to time:

  ```bash
  sudo apt autoremove --purge
  sudo apt clean
  ```

- Keep only 2 old revisions of each snap `sudo snap set system refresh.retain=2`

#### 2.1.4 Set sudo session timeout in console

- Same as [Arch 2.1.4](setup-arch.md#214-set-sudo-session-timeout-in-console)

#### 2.1.5 Setup Vim

- Install `vim-gtk3` to get system clipboard support (the Ubuntu equivalent of `gvim`) `sudo apt install vim-gtk3`
- `~/.vimrc` content: same as [Arch 2.1.5](setup-arch.md#215-setup-vim)

### 2.2 Software sources (replaces `yay`)

- There is no AUR on Ubuntu. Follow the [software sources policy](#0-software-sources-policy) above.
- Install a downloaded `.deb` with `sudo apt install ./file.deb` (not `dpkg -i`), so dependencies are resolved
- List apps installed as snaps `snap list`

### 2.3 Update firmware

- `fwupd` is preinstalled. Same commands as [Arch 2.3](setup-arch.md#23-update-firmware), or use the Firmware Updater app

### 2.4 Fonts and Codec

```bash
sudo apt install ubuntu-restricted-extras   # codecs + MS core fonts; accept the EULA with Tab + Enter
sudo apt install fonts-noto-cjk fonts-noto-color-emoji fonts-liberation
```

- Install Adwaita Sans and Adwaita Mono (GNOME's default fonts since GNOME 48) `sudo apt install fonts-adwaita`
- Set fonts: same as [Arch 2.4](setup-arch.md#24-fonts-and-codec)

### 2.5 Setup bluetooth

- `bluez` and `bluetoothctl` are preinstalled and Bluetooth turns on at boot by default
- Troubleshooting (cannot connect a headphone): same as [Arch 2.5](setup-arch.md#25-setup-bluetooth)

### 2.6 Install Google Chrome

- Do not use the Chromium snap or a Flatpak Chrome: the `GNOME Shell integration` extension cannot reach the host
- Install the official `.deb` (it also adds Google's apt repo for updates)

  ```bash
  curl -fLO https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
  sudo apt install ./google-chrome-stable_current_amd64.deb
  ```

#### 2.6.1 Separate Google profiles

- Same as [Arch 2.6.1](setup-arch.md#261-separate-google-profiles) (Chrome uses the same `/usr/bin/google-chrome-stable` path)

### 2.7 Install extensions for GNOME

- Install the host connector for the browser extension, plus Extension Manager

  ```bash
  sudo apt install gnome-browser-connector gnome-shell-extension-manager
  ```

- Recommended extensions: same list as [Arch 2.7](setup-arch.md#27-install-extensions-for-gnome), except:
  - **Skip** `Dash to Dock` and `AppIndicator and KStatusNotifierItem Support`: Ubuntu Dock and AppIndicator support are built in
  - Ubuntu 26.04 ships GNOME 50. Some extensions in the list may not support it yet; Extension Manager only shows compatible versions
  - Blur my Shell: Extension Manager installs a newer version with popup blur turned on, which breaks the look of menus and Quick Settings with Yaru or Dracula. Turn it off: same as [Arch 2.7](setup-arch.md#27-install-extensions-for-gnome)

### 2.8 Install `fcitx5`

- Ubuntu uses IBus by default. Install fcitx5 and its frontends

  ```bash
  sudo apt install fcitx5 fcitx5-config-qt fcitx5-frontend-gtk3 fcitx5-frontend-gtk4 fcitx5-frontend-qt5 fcitx5-frontend-qt6
  ```

- Install Lotus from its official apt repo (run in `bash`)

  ```bash
  CODENAME=$(grep '^UBUNTU_CODENAME=' /etc/os-release | cut -d'=' -f2)
  sudo mkdir -p /etc/apt/keyrings
  curl -fsSL https://fcitx5-lotus.pages.dev/pubkey.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/fcitx5-lotus.gpg
  echo "deb [signed-by=/etc/apt/keyrings/fcitx5-lotus.gpg] https://fcitx5-lotus.pages.dev/apt/$CODENAME $CODENAME main" | sudo tee /etc/apt/sources.list.d/fcitx5-lotus.list
  sudo apt update && sudo apt install fcitx5-lotus
  ```

- Enable the Lotus server for your user (required, the package does not enable it)

  ```bash
  sudo systemctl enable --now fcitx5-lotus-server@$(whoami).service
  ```

- After upgrading `fcitx5-lotus`, restart it `sudo systemctl restart fcitx5-lotus-server@$(whoami).service`
- Troubleshooting (Lotus works after install, but stops after logging out or rebooting): the server log `journalctl -u fcitx5-lotus-server@$(whoami).service -b` shows `Failed to create the uinput device: Permission denied`
  - Cause: the server runs as user `uinput_proxy`, and needs an ACL on `/dev/uinput`. The package's udev rule (`/usr/lib/udev/rules.d/99-lotus.rules`) sets it, but on Ubuntu `uinput` is built into the kernel, so the rule may not run at boot and the ACL is lost. Check with `getfacl /dev/uinput` (there should be a `user:uinput_proxy:rw-` line)
  - Fix: make the service set the ACL itself before it starts. Run `sudo systemctl edit fcitx5-lotus-server@.service` and add (the `+` runs that one command as root)

    ```ini
    [Service]
    ExecStartPre=+/usr/bin/setfacl -m u:uinput_proxy:rw /dev/uinput
    ```

  - Apply it

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl reset-failed fcitx5-lotus-server@$(whoami).service
    sudo systemctl restart fcitx5-lotus-server@$(whoami).service
    ```

- Set fcitx5 as the input method framework `im-config -n fcitx5` (do this instead of editing `/etc/environment`)
- Autostart: GNOME Tweaks > Startup Applications > Add > Fcitx 5
- Log out and log back in
- Open `fcitx5-configtool`, find "Lotus" on the right, move it to the left > Apply
- Install the "Input Method Panel" extension to show the language icon in the tray. On Wayland it also draws the input method popups in the right place, so keep it enabled

### 2.9 Setup console

- Install `fish`, `kitty` and `starship` `sudo apt install fish kitty starship`
- Install JetBrains Mono Nerd Font (not packaged in apt)

  ```bash
  mkdir -p ~/.local/share/fonts/JetBrainsMono
  curl -fLO https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.tar.xz
  tar -xf JetBrainsMono.tar.xz -C ~/.local/share/fonts/JetBrainsMono
  fc-cache -f
  ```

- Set `fish` as the default shell `chsh -s /usr/bin/fish`
  - Log out and log back in for it to apply to new terminals
  - Check `getent passwd $USER | cut -d: -f7` prints `/usr/bin/fish`
- `kitty.conf`, starship activation and the [starship-dracula.toml](starship-dracula.toml) prompt config: same as [Arch 2.9](setup-arch.md#29-setup-console)
- Install advanced tools `sudo apt install fzf zoxide eza bat ripgrep fd-find`
- ⚠️ On Ubuntu, `bat` installs as `batcat` and `fd` as `fdfind`. Use these aliases in `~/.config/fish/config.fish` instead of the Arch ones

  ```fish
  zoxide init fish | source
  alias ls='eza --icons --group-directories-first'
  alias ll='eza -l --icons --git'
  alias cat='batcat'
  alias fd='fdfind'
  ```

### 2.10 Setup themes

- Install Papirus icons and Bibata cursor `sudo apt install papirus-icon-theme bibata-cursor-theme`
- Install Dracula GTK theme (not packaged in apt) from the `master` branch. The latest release (v4.0.0, 2023) is older than GNOME's Quick Settings menu, so its Shell theme breaks that menu

  ```bash
  mkdir -p ~/.themes
  curl -fL https://github.com/dracula/gtk/archive/refs/heads/master.tar.gz -o dracula-gtk.tar.gz
  tar -xzf dracula-gtk.tar.gz
  rm -rf ~/.themes/Dracula
  mv gtk-master ~/.themes/Dracula
  ```

  - To update the theme later, run the same commands again

- Select theme, cursor and icons via Tweaks (the Shell theme needs the `User Themes` extension). Log out and log back in after changing the Shell theme: on Wayland GNOME Shell cannot be restarted with `Alt+F2` > `r`

### 2.11 Wi-Fi tuning

#### 2.11.1 Disable Wi-Fi power save

- Ubuntu turns power save **on** by default (`/etc/NetworkManager/conf.d/default-wifi-powersave-on.conf`, `wifi.powersave = 3`)
- Steps: same as [Arch 2.11.1](setup-arch.md#2111-disable-wi-fi-power-save), install `iw` with `sudo apt install iw` if missing. NetworkManager reads `conf.d` files in alphabetical order and the last value wins, so `wifi-powersave.conf` overrides the default file

#### 2.11.2 Set Wi-Fi country code

- Why: same as [Arch 2.11.2](setup-arch.md#2112-set-wi-fi-country-code). Ubuntu does not set it either, and has no `/etc/conf.d/wireless-regdom` (`wireless-regdb` is preinstalled)
- Set it at every boot through the `cfg80211` kernel module

  ```bash
  echo "options cfg80211 ieee80211_regdom=VN" | sudo tee /etc/modprobe.d/cfg80211.conf
  ```

- Apply now `sudo iw reg set VN`
- Check `iw reg get`: same as Arch. If it still shows `country 00` after a reboot, run `sudo update-initramfs -u` and reboot again

## 3. Applications

### 3.1 Docker Engine

- Use Docker's official repo, not the `docker.io` package from Ubuntu or the snap. Remove them first if present

  ```bash
  sudo apt remove docker.io docker-compose docker-doc podman-docker containerd runc
  sudo snap remove docker
  ```

- Add the repo and install (run in `bash`)

  ```bash
  sudo apt install ca-certificates curl
  sudo install -m 0755 -d /etc/apt/keyrings
  sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
  sudo chmod a+r /etc/apt/keyrings/docker.asc
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list
  sudo apt update
  sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  ```

- The service starts automatically. Add your user to the `docker` group: same as [Arch 3.1](setup-arch.md#31-docker-engine)
- Note: Compose is a plugin here, so the command is `docker compose` (with a space)

### 3.2 VSCode

- Download the `.deb` from <https://code.visualstudio.com/download> and install it `sudo apt install ./code_*.deb`. Answer `Yes` when asked to add Microsoft's apt repo, so VSCode updates with `apt`
- Settings: same as [Arch 3.2](setup-arch.md#32-vscode)

### 3.3 Install Claude CLI

- Same as [Arch 3.3](setup-arch.md#33-install-claude-cli)

### 3.4 Common Applications

- Install DBeaver `sudo snap install dbeaver-ce`
- Install Node: the apt version is old, so use the NodeSource LTS repo

  ```bash
  curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
  sudo apt install nodejs
  ```

### 3.5 Setup Git multi-profiles

- Same as [Arch 3.5](setup-arch.md#35-setup-git-multi-profiles)

### 3.6 Configure fingerprint (if hardware support)

- Install `sudo apt install fprintd libpam-fprintd`
- Enroll: Settings > System > Users > Fingerprint Login, or use the `fprintd-enroll` / `fprintd-verify` commands from [Arch 3.6](setup-arch.md#36-configure-fingerprint-if-hardware-support)
- Enable fingerprint for `sudo` and login with `sudo pam-auth-update --enable fprintd`. Do not edit `/etc/pam.d/sudo` by hand: Ubuntu manages PAM through `pam-auth-update`
- Testing: open a new terminal and run `sudo echo ok`

### 3.7 Setup face recognition (if hardware support)

- Check for an IR camera: same as [Arch 3.7.1](setup-arch.md#371-check-if-hardware-is-avaiable-for-face-recognition) (install `v4l-utils` with apt)
- ⚠️ **Experimental.** Howdy is not in apt. The official `boltgolt/howdy` PPA uses `pip` in its post-install script, which Ubuntu blocks. Use the unofficial rebuild maintained by UbuntuHandbook at your own risk

  ```bash
  sudo add-apt-repository ppa:ubuntuhandbook1/howdy
  sudo apt update && sudo apt install howdy
  ```

- Set the camera with `sudo howdy config`: `device_path = /dev/video2` (your IR device), `certainty = 3.5`
- Enroll `sudo howdy add`, list `sudo howdy list`
- Before testing, keep a root shell open in a second terminal (`sudo -i`) and take a Timeshift snapshot. Check `/etc/pam.d/common-auth` to see whether the package added the Howdy line
- To disable: `sudo howdy disable 1`

### 3.8 Install Postman

- Install `sudo snap install postman`

### 3.9 Install AnyDesk

- Install from AnyDesk's official repo (run in `bash`)

  ```bash
  sudo install -m 0755 -d /etc/apt/keyrings
  curl -fsSL https://keys.anydesk.com/repos/DEB-GPG-KEY | sudo gpg --dearmor -o /etc/apt/keyrings/anydesk.gpg
  echo "deb [signed-by=/etc/apt/keyrings/anydesk.gpg] http://deb.anydesk.com/ all main" | sudo tee /etc/apt/sources.list.d/anydesk-stable.list
  sudo apt update && sudo apt install anydesk
  ```

- Wayland fix (run AnyDesk through XWayland): same as [Arch 3.9](setup-arch.md#39-install-anydesk). Ubuntu 26.04 has no "Ubuntu on Xorg" session to fall back to

### 3.10 Enable Hibernate mode

#### 3.10.1 Check system requirement before setup

- 🔴 **Check Secure Boot first.** On Ubuntu, Secure Boot turns on kernel lockdown, which blocks hibernation

  ```bash
  mokutil --sb-state      # "SecureBoot enabled" → hibernate will not work
  ```

  - If enabled, either disable Secure Boot in the BIOS/UEFI settings, or skip this whole section

- Other checks: same as [Arch 3.10.1](setup-arch.md#3101-check-system-requirement-before-setup) (skip the `mkinitcpio.conf` line)
- Ubuntu already has a swapfile at `/swap.img`, but it is small (usually 2–4 GB). Check with `swapon --show`

#### 3.10.2 Resize the Swapfile

- Resize `/swap.img` to the RAM size (16 GB in this example). `/etc/fstab` already has an entry for it

  ```bash
  sudo swapoff /swap.img
  sudo rm /swap.img
  sudo fallocate -l 16G /swap.img
  sudo chmod 600 /swap.img
  sudo mkswap /swap.img
  sudo swapon /swap.img
  ```

- Check `swapon --show`

#### 3.10.3 Configure resume parameters

- Find the UUID of the partition holding the swapfile `findmnt -no UUID -T /swap.img`
- Find the resume offset `sudo filefrag -v /swap.img | awk '$1=="0:"{print substr($4, 1, length($4)-2)}'`
- Ubuntu uses GRUB, not `/etc/kernel/cmdline`. Edit `sudo vim /etc/default/grub`

  ```text
  GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=UUID=<uuid> resume_offset=<offset>"
  ```

- Apply `sudo update-grub`

#### 3.10.4 Update initramfs

- Ubuntu uses `initramfs-tools`, not `mkinitcpio`, and needs no `HOOKS` change. Check with `dpkg -l initramfs-tools dracut`: a machine upgraded from 24.04 keeps `initramfs-tools`. If only `dracut` is installed, run `sudo dracut -f --regenerate-all` instead of `update-initramfs`

  ```bash
  echo "RESUME=UUID=<uuid>" | sudo tee /etc/initramfs-tools/conf.d/resume
  sudo update-initramfs -u -k all
  sudo reboot
  ```

- After boot, check: same `/sys/power/resume` commands as [Arch 3.10.4](setup-arch.md#3104-add-resume-hook)
- Test `sudo systemctl hibernate`. The machine should power off, and your session should come back when you turn it on

#### 3.10.5 Setup hibernate command

- Ubuntu blocks hibernate for normal users with a polkit rule (`/usr/share/polkit-1/rules.d/com.ubuntu.desktop.rules`, "Disable hibernate by default in Ubuntu"). `sudo systemctl hibernate` works because root skips polkit, but the GNOME button and `systemctl hibernate` without `sudo` do nothing
- Check `busctl call org.freedesktop.login1 /org/freedesktop/login1 org.freedesktop.login1.Manager CanHibernate` prints `s "no"`
- Add a rule that allows hibernate for the user sitting at the machine (local, active session). Polkit reads rule files in name order, so `50-...` runs before Ubuntu's `com.ubuntu...` rule. Works in both `bash` and `fish`; paste the whole block at once

  ```bash
  printf '%s\n' \
      '// Allow hibernate for the active local user (Ubuntu disables it by default)' \
      'polkit.addRule(function(action, subject) {' \
      '    if ((action.id == "org.freedesktop.login1.hibernate" ||' \
      '         action.id == "org.freedesktop.login1.handle-hibernate-key" ||' \
      '         action.id == "org.freedesktop.login1.hibernate-multiple-sessions" ||' \
      '         action.id == "org.freedesktop.upower.hibernate") &&' \
      '        subject.active && subject.local) {' \
      '        return polkit.Result.YES;' \
      '    }' \
      '});' \
      | sudo tee /etc/polkit-1/rules.d/50-enable-hibernate.rules
  ```

- Polkit reloads the rule by itself (no reboot). Run the `CanHibernate` check again: it should print `s "yes"`
- To undo `sudo rm /etc/polkit-1/rules.d/50-enable-hibernate.rules`
- The Arch `ProtectHome=read-only` override is not needed: `/swap.img` is not in `/home`

#### 3.10.6 Add "Hibernate" button

- Same as [Arch 3.10.6](setup-arch.md#3106-add-hibernate-button). Check that the extension supports GNOME 50
- The extension shows the Hibernate button even when hibernate is not set up. The button only works once `CanHibernate` prints `s "yes"` ([3.10.5](#3105-setup-hibernate-command)). If you do not use hibernate, hide it: extension settings > turn off Hibernate

### 3.11 Remote access (GNOME Remote Desktop + Tailscale)

- What it is and the two modes (Remote Login, Desktop Sharing): same as [Arch 3.11](setup-arch.md#311-remote-access-gnome-remote-desktop--tailscale)
- Every command in this section works in both `bash` and `fish`, except the one line marked below

#### 3.11.1 Install packages

- GNOME Remote Desktop is preinstalled. Install only Tailscale, from its official apt repo
  1. Save the Ubuntu codename (`resolute`) in a variable. This is the only line that differs between shells
     - `bash`: `CODENAME=$(grep '^UBUNTU_CODENAME=' /etc/os-release | cut -d'=' -f2)`
     - `fish`: `set CODENAME (grep '^UBUNTU_CODENAME=' /etc/os-release | cut -d'=' -f2)`
  2. In the same terminal, paste this whole block at once. Lines ending in `\` continue on the next line

     ```bash
     sudo install -m 0755 -d /etc/apt/keyrings
     curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/$CODENAME.noarmor.gpg \
         | sudo tee /etc/apt/keyrings/tailscale.gpg >/dev/null
     printf '%s\n' \
         'Types: deb' \
         'URIs: https://pkgs.tailscale.com/stable/ubuntu' \
         "Suites: $CODENAME" \
         'Components: main' \
         'Architectures: amd64' \
         'Signed-By: /etc/apt/keyrings/tailscale.gpg' \
         | sudo tee /etc/apt/sources.list.d/tailscale.sources
     sudo apt update
     sudo apt install tailscale
     ```

  3. Check `cat /etc/apt/sources.list.d/tailscale.sources` shows `Suites: resolute`
- The `tailscaled` service starts automatically after install. Check `systemctl is-enabled tailscaled` prints `enabled`

#### 3.11.2 Log in to Tailscale

- Same as [Arch 3.11.2](setup-arch.md#3112-log-in-to-tailscale)

#### 3.11.3 Turn on GNOME Remote Desktop

- Same as [Arch 3.11.3](setup-arch.md#3113-turn-on-gnome-remote-desktop)

#### 3.11.4 Keep the machine reachable

- Same as [Arch 3.11.4](setup-arch.md#3114-keep-the-machine-reachable). The GDM user is also `gdm` on Ubuntu

#### 3.11.5 Connect

- Same as [Arch 3.11.5](setup-arch.md#3115-connect). On Ubuntu, Remmina is preinstalled

#### 3.11.6 Daily flow

- Same as [Arch 3.11.6](setup-arch.md#3116-daily-flow)

## 4. TODOs

- Same list as [Arch 4](setup-arch.md#4-todos)
