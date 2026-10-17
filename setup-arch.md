# SETUP ARCH LINUX

## Table of Contents

- [1. SETUP ARCH POST-INSTALL (AFTER REBOOTING)](#1-setup-arch-post-install-after-rebooting)
  - [1.1 Activate network connection](#11-activate-network-connection)
  - [1.2 Add new `sudo` user alter to root user](#12-add-new-sudo-user-alter-to-root-user)
  - [1.3 Enable Gnome](#13-enable-gnome)
- [2. SETUP ARCH](#2-setup-arch)
  - [2.1 Popular packages and setup](#21-popular-packages-and-setup)
    - [2.1.1 Personalization](#211-personalization)
    - [2.1.2 Setting up linux-lts for fallback OS](#212-setting-up-linux-lts-for-fallback-os)
    - [2.1.3 Schedule clear Pacman package cache](#213-schedule-clear-pacman-package-cache)
    - [2.1.4 Set sudo session timeout in console](#214-set-sudo-session-timeout-in-console)
    - [2.1.5 Setup Vim](#215-setup-vim)
    - [2.1.6 Pacman quality of life](#216-pacman-quality-of-life)
  - [2.2 Install `yay` (Yet Another Yogurt)](#22-install-yay-yet-another-yogurt)
  - [2.3 Update firmware](#23-update-firmware)
  - [2.4 Fonts and Codec](#24-fonts-and-codec)
  - [2.5 Setup bluetooth](#25-setup-bluetooth)
  - [2.6 Install Google Chrome](#26-install-google-chrome)
    - [2.6.1 Separate Google profiles](#261-separate-google-profiles)
  - [2.7 Install extensions for GNOME](#27-install-extensions-for-gnome)
  - [2.8 Install `fcitx5`](#28-install-fcitx5)
  - [2.9 Setup console](#29-setup-console)
  - [2.10 Setup themes](#210-setup-themes)
- [3. Applications](#3-applications)
  - [3.1 Docker Engine](#31-docker-engine)
  - [3.2 VSCode](#32-vscode)
  - [3.3 Install Claude CLI](#33-install-claude-cli)
  - [3.4 Common Applications](#34-common-applications)
  - [3.5 Setup Git multi-profiles](#35-setup-git-multi-profiles)
  - [3.6 Configure fingerprint (if hardware support)](#36-configure-fingerprint-if-hardware-support)
  - [3.7 Setup face recognition (if hardware support)](#37-setup-face-recognition-if-hardware-support)
    - [3.7.1 Check if hardware is avaiable for face recognition](#371-check-if-hardware-is-avaiable-for-face-recognition)
    - [3.7.2 Configure face recognition](#372-configure-face-recognition)
- [4. TODOs](#4-todos)
  - [4.1 Later](#41-later)

## 1. SETUP ARCH POST-INSTALL (AFTER REBOOTING)

### 1.1 Activate network connection

- Activate network connection
  - Run in bash `nmtui`
  - Select `Activate a connection`
  - Select Wi-Fi and enter password
  - After connection successfully > Back > Quit

### 1.2 Add new `sudo` user alter to root user

- Create new user `useradd -m -G wheel -s /bin/bash <username>`
- Update password `passwd <username>`
- Install package `pacman -S vim sudo` (or `nano`)
- Add sudo permission for being created user
  - Open authorization file `EDITOR=nano visudo`
  - Uncomment `# %wheel ALL=(ALL:ALL) ALL` (remove `#`)
  - Save file (`:wq` with vim)

### 1.3 Enable Gnome

- UI choices: `GNOME` | `KDE Plasma` | `XFCE`
- Install GNOME `pacman -S gnome gdm`
- Enable GNOME `systemctl enable --now gdm`

## 2. SETUP ARCH

### 2.1 Popular packages and setup

- Show `minimize`, `maximize`, `close` button on explorer `gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'`
- Install git `sudo pacman -S --needed base-devel git`
- Install gnome-tweaks `sudo pacman -S gnome-tweaks`
- Install vim `sudo pacman -S vim`
- Install zip, unzip `sudo pacman -S zip unzip`
- Install office `sudo pacman -S libreoffice-still libreoffice-still-vi hunspell hunspell-en_us`
- Install drawing & photo editor

  ```bash
  sudo pacman -S drawing        # closest to MS Paint, simple
  flatpak install flathub com.github.PintaProject.Pinta   # Paint.NET-like, layers
  ```

- Install translation shell `sudo pacman -S translate-shell`

#### 2.1.1 Personalization

- Update Wallpapper
- Set user profile image: Settings > Users

#### 2.1.2 Setting up linux-lts for fallback OS

- Setting up `linux-lts` on systemd-boot with UKI: Arch ships bleeding-edge kernels. A bad one can break Wi-Fi, graphics, or Bluetooth — and if it's your only kernel, recovery means a live USB. A second kernel in the boot menu turns that into selecting a different entry at startup.
- Step 1: Checks before starting

```bash
bootctl status                      # which bootloader, and is it a UKI?
df -h /boot                         # room for a second UKI?
cat /etc/kernel/cmdline             # boot options exist?
cat /etc/mkinitcpio.d/linux.preset  # template to copy
sudo ls /boot/EFI/Linux/            # what's there now
pacman -Qs dkms                     # need headers?
```

- What you're looking for:
  - Bootloader: `systemd-boot`, entry type "Type #2 (UKI, .efi)"
  - `/boot` free ≥ 150 MB
  - `/etc/kernel/cmdline`: exists, contains `root=PARTUUID=...`
  - `linux.preset` has `default_uki=`, `default_image=` commented out
- Step 2: Setup
  - Install the kernel `sudo pacman -S linux-lts`
  - Check if module DKMS used `pacman -Qs dkms`, if return result (not empty), install header `sudo pacman -S linux-lts-headers`
  - Replace the LTS preset — the shipped one builds a plain initramfs, not a UKI
    - Edit preset file `sudo vim /etc/mkinitcpio.d/linux-lts.preset`

    ```bash
    # mkinitcpio preset file for the 'linux-lts' package

    ALL_kver="/boot/vmlinuz-linux-lts"

    PRESETS=('default')

    default_uki="/boot/EFI/Linux/arch-linux-lts.efi"
    default_options="--splash /usr/share/systemd/bootctl/splash-arch.bmp"
    ```

  - Build

    ```bash
    sudo mkinitcpio -p linux-lts
    sudo bootctl list
    ```

    - Expect two Type #2 entries: `arch-linux.efi` and `arch-linux-lts.efi`.

  - Pin the default and ensure the menu appears
    - Edit boot loader configuration `sudo vim /boot/loader/loader.conf`

    ```text
    default arch-linux.efi
    timeout 3
    ```

#### 2.1.3 Schedule clear Pacman package cache

```bash
sudo pacman -S pacman-contrib
sudo systemctl enable --now paccache.timer
```

#### 2.1.4 Set sudo session timeout in console

- Create new sudoer configuration file `sudo visudo -f /etc/sudoers.d/timeout`
- Add configuration `Defaults timestamp_timeout=30` > Save

#### 2.1.5 Setup Vim

- Install gvim `pacman -S gvim` for copying into global clipboard
- Create configuration file `vim ~/.vimrc`
- Content

```text
set nocompatible
syntax on
filetype plugin indent on

set number
set expandtab shiftwidth=4 tabstop=4
set ignorecase smartcase incsearch hlsearch
set scrolloff=8
set clipboard=unnamedplus
set undofile
set mouse=a
set termguicolors

let mapleader = " "
nnoremap <leader>w :w<CR>
nnoremap <Esc><Esc> :nohlsearch<CR>
```

#### 2.1.6 Pacman quality of life

- Simple UI Pacman animation when run command `pacman`
  - Open pacman configuration file `sudo vim /etc/pacman.conf`
  - Find `Color` section and update

  ```text
  Color
  ILoveCandy   # Add this line
  #NoProgressBar
  CheckSpace
  #VerbosePkgLists
  ParallelDownloads = 5 # Set this to `5`
  ```

### 2.2 Install `yay` (Yet Another Yogurt)

- Clone `yay` source code from Git `git clone https://aur.archlinux.org/yay.git` (should create new directory for storing source code)
- `cd yay` then build & install `makepkg -si`

### 2.3 Update firmware

```bash
sudo pacman -S fwupd
fwupdmgr refresh && fwupdmgr get-updates
```

### 2.4 Fonts and Codec

```bash
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji ttf-liberation
sudo pacman -S gst-plugins-good gst-plugins-bad gst-plugins-ugly gst-libav ffmpeg
```

### 2.5 Setup bluetooth

- Enable bluetooth service `sudo systemctl enable --now bluetooth.service`
- Configure bluetooth start up mode
  - `sudo vim /etc/bluetooth/main.conf`
  - Uncomment or add `AutoEnable=true` > Save & quit
- Install bluez `sudo pacman -S bluez-utils`
- Fix issue cannot connect bluetooth devices (headphone)
  - Remove cached device

  ```bash
  bluetoothctl
  remove <device_mac>
  ```

  - Disconnect any other devices that connected to bluetooth device (e.g: phone,...)
  - Try to reconnect device, make sure device in pairing mode (e.g: with earbuds, hold the pairing button)
  - Use the `bluetoothctl` to pair with device (if the GUI cannot detect the device)

  ```bash
  bluetoothctl
  agent on
  scan on
  # set the device into pairing mode, e.g: hold the pairing button on device)
  # wait the device display on list
  # if device does not show, try `devices` to list all detected devices
  # when device detected
  scan off
  # enter command immediately after stopping scanning
  pair <device_mac>
  ```

### 2.6 Install Google Chrome

- Do not install Google Chrome via `flatpak`, it can cause error when install `GNOME Shell Integration` extension on Chrome
- Firstly, check if Google Chrome installed via `flatpak`, if so, uninstall it
  - Check installed apps by `flatpak`: `flatpak list | grep -i chrome`
  - Check how chrome installed `pacman -Qo $(which google-chrome-stable)`
  - Remove chrome if installed via `flatpak`:
    - `flatpak kill com.google.Chrome`
    - Copy configuration `cp -rn ~/.var/app/com.google.Chrome/config/google-chrome/. ~/.config/google-chrome/` (after installing new Chrome)
    - `flatpak uninstall com.google.Chrome`
- Install Google Chrome with `yay`: `yay -S google-chrome`

#### 2.6.1 Separate Google profiles

- Create Google Chrome profiles: Click profile > Add Chrome profile
- Check Chrome profiles `ls ~/.config/google-chrome/ | grep -iE 'default|profile'`
- Add desktop file
  - Create new file `vim ~/.local/share/applications/chrome-profile.desktop`

  ```text
  [Desktop Entry]
  Version=1.0
  Name=Chrome (Personal)
  Exec=/usr/bin/google-chrome-stable --profile-directory="Default"
  Icon=google-chrome
  Type=Application
  Categories=Network;WebBrowser;
  ```

  - Replace `--profile-directory` with chrome profile folder name
  - Reload desktop `update-desktop-database ~/.local/share/applications`

- Create symlink to `~/Desktop`

```bash
# Loop with all profiles created
ln -s ~/.local/share/applications/chrome-personal.desktop ~/Desktop/
```

- Check symlink created `ls -la ~/Desktop/`

### 2.7 Install extensions for GNOME

- `GNOME Shell integration` extension for Chrome <https://chromewebstore.google.com/detail/gnome-shell-integration/gphhapmejobijbbhgpjhcjognlahblep?hl=en>
- List recommended extensions
  - AppIndicator and KStatusNotifierItem Support
  - Bluetooth Quick Connect
  - Caffeine
  - Copyous
  - Dash to Dock
  - Input Method Panel
  - User Themes
  - Vitals
  - PiP on top
  - Tiling shell
  - Emoji Copy
  - Blur my Shell
  - Desktop Icons NG (DING)
  - Power Off Options

### 2.8 Install `fcitx5`

- Required `yay` to install, install `yay -S fcitx5 fcitx5-configtool fcitx5-qt fcitx5-gtk fcitx5-lotus`
  - Select provider `python-pyqt6`
- Setup environment
  - Configure environment variable: `sudo nano /etc/environment`
  - Append these configuration

  ```bash
    GTK_IM_MODULE=fcitx
    QT_IM_MODULE=fcitx
    XMODIFIERS=@im=fcitx
  ```

- Open Fcitx5 Configuration tool: `fcitx5-configtool`
  - Find "Lotus" (or Vietnamese) in `Available Input Method` panel, then move into "Current Input Method" > OK
- Install "Input Method Panel" extension on "GNOME Extensions" to display language icon on icon tray > Logout to activate the extension

### 2.9 Setup console

- Install `fish` shell with `starship` and nerd font: `sudo pacman -S fish starship ttf-jetbrains-mono-nerd`
- Install `kitty` pallete
  - Install `sudo pacman -S kitty`
  - Config `mkdir -p ~/.config/kitty`
  - Create configuration file `vim ~/.config/kitty/kitty.conf`

  ```text
  font_family JetBrainsMono Nerd Font
  font_size 11
  ```

- Set `fish` to default shell `chsh -s /usr/bin/fish`
- Activate the `starship` prompt

```bash
mkdir -p ~/.config/fish
echo 'starship init fish | source' >> ~/.config/fish/config.fish
exec fish
```

- Set starship preset `starship preset nerd-font-symbols -o ~/.config/starship.toml`
  - To select other preset, use `starship preset -l` to list all available presets then run above command with desired preset
- UPDATE: use template configuration file `starship.toml` (copy to `~/.config` to activate configuration)
- Add some popular extra configuration in `~/.config/starship.toml` then add these lines

  ```toml
  # Blank line between prompts
  add_newline = true

  # Show command duration for slow commands
  [cmd_duration]
  min_time = 2000
  format = "took [$duration](bold yellow) "
  ```

- Install some advanced tools `sudo pacman -S fzf zoxide eza bat ripgrep fd`
  - Activate tools and create aliases by adding these lines into `./.config/fish/config.fish`

  ```fish
  zoxide init fish | source
  alias ls='eza --icons --group-directories-first'
  alias ll='eza -l --icons --git'
  alias cat='bat'
  ```

### 2.10 Setup themes

- Install Dracula theme `yay -S dracula-gtk-theme-git`
- Install custom cursor and icons `yay -S papirus-icon-theme bibata-cursor-theme`
- Select theme, cursor and icons via Tweaks

## 3. Applications

### 3.1 Docker Engine

- Install Docker `sudo pacman -S docker docker-compose docker-buildx`
- Enable Docker service `sudo systemctl enable --now docker.service`
- Add current user into docker group `sudo usermod -aG docker $USER`
- Clear old session `newgrp docker`

### 3.2 VSCode

- Install VSCode `yay -S visual-studio-code-bin`
- Select VSCode Vim
- Login Github > Sync Settings
- Install Claude Code extension
- Set default open model in main windows: Settings > "workbench.editor.useModal": "off"
- Enable Settings > "SimpleDialog"
- Enable Settings > "SystemClipboard"
- Enable Settings > "HLSearch"
- Set font: Settings > "terminal.integrated.fontFamily": "JetBrainsMono Nerd Font Mono"
- Allow use shortcuts when text editor focused: Ctr + Shift + P > "Preferences: Open User Settings (JSON)" > Add exception keys

```json
    "vim.handleKeys": {
       "<C-n>": false,
       "<C-o>": false,
       "<C-k>": false,
       "<C-p>": false,
       "<C-f>": false,
       "<C-b>": false,
       "<C-w>": false,
   }
```

### 3.3 Install Claude CLI

- Install Claude CLI `curl -fsSL https://claude.ai/install.sh | bash`

### 3.4 Common Applications

- Install DBeaver `sudo pacman -S dbeaver`
- Install Node `sudo pacman -S nodejs npm`

### 3.5 Setup Git multi-profiles

- Create global configuration file

  ```bash
  mkdir -p ~/.config/git
  vim ~/.config/git/config
  ```

  - Put configuration

  ```txt
  [user]
      useConfigOnly = true

  [includeIf "gitdir:~/workspace/work/"]
      path = ~/.config/git/config-work

  [includeIf "gitdir:~/workspace/personal/"]
      path = ~/.config/git/config-personal

  [init]
      defaultBranch = main
  ```

- Create profile configuration `vim ~/.config/git/config-personal`

  ```text
  [user]
      name = Your Name
      email = you@example.com

  [core]
      sshCommand = ssh -i ~/.ssh/personal_id_ed25519 -o IdentitiesOnly=yes
  ```

- Create SSH key `ssh-keygen -t ed25519 -C "you@example.com" -f ~/.ssh/personal_id_ed25519`

- Verify in project directory `git config user.name`

### 3.6 Configure fingerprint (if hardware support)

- Install fingerprint package: `pacman -S fprintd libfprint`
- Check if any fingerprints were previously setup `fprintd-list $USER`
- Enroll new fingerprint `fprintd-enroll`. Prompt the finger on sensor,Touch/swipe your finger on the sensor, the terminal should show `Enroll result: enroll-stage-passed`. Keep repeating until the terminal shows `Enroll result: enroll-completed`
- Verify the added fingerprint: `fprintd-verify`, the terminal should show `Verify result: verify-match`
- Set fingerprint to authenticate
  - Backup: Open a second terminal, then login as suoder using `sudo -i`
  - Open file `sudo vim /etc/pam.d/sudo`
  - Put `auth      sufficient  pam_fprintd.so` as the first line of the file and save it.
- Testing: Open new terminal and test the sudo action with `sudo echo ok`

### 3.7 Setup face recognition (if hardware support)

#### 3.7.1 Check if hardware is avaiable for face recognition

- Install utility package `v4l2-ctl --list-devices`
- Check list devices `v4l2-ctl --list-devices`
- Loop all video device with name following pattern `/dev/video*` with `v4l2-ctl -d /dev/video2 --list-formats`
- Check if any device is IR camera _'GREY' (8-bit Greyscale)_

```bash
❯ v4l2-ctl -d /dev/video2 --list-formats
ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture

        [0]: 'GREY' (8-bit Greyscale)
```

- If device has no IR camera, face recognition is not supported

#### 3.7.2 Configure face recognition

- Install packages `yay -S howdy pam-python`
- Check `pam-python` is installed `ls -l /usr/lib/security/pam_python.so`
- Find `howdy` configuration file `pacman -Ql howdy | grep -iE 'config|\.ini|\.toml'`, usually is `/usr/lib/security/howdy/config.ini`. Modify `[video]` section

  ```ini
  [video]
  device_path = /dev/video2
  certainty = 3.5
  ```

- Test IR camera by taking a photo

  ```bash
  sudo pacman -S ffmpeg
  sudo ffmpeg -f v4l2 -i /dev/video2 -frames:v 1 -y /tmp/ir.png
  ```

  - Open taken photo `xdg-open ir.png`, it should photo with gray color

- Enroll new face `sudo howdy add`
- Check face enrolled `sudo howdy list`
- Remove face `howdy remove 0` (the face index)

- Set face recognition to authenticate
  - Backup: Open a second terminal, then login as suoder using `sudo -i`
  - Open file `sudo vim /etc/pam.d/sudo`
  - Put `auth      sufficient  pam_python.so  /lib/security/howdy/pam.py` as the first line of the file and save it.
- Testing: Open new terminal and test the sudo action with `sudo echo ok`

- Disable face recognition: Remove or comment added line for face recognotion in PAM configuration file

### 3.8 Install Postman

- Install Postman via Yay: `yay -S postman-bin`

### 3.9 Install AnyDesk

- Install software via Yay `yay -S anydesk-bin`
- In Wayland, AnyDesk got some issues when display remote machine monitor. To resolve this, run AnyDesk in X11 protocols
- Copy desktop configuration file `cp /usr/share/applications/anydesk.desktop ~/.local/share/applications/`
- Modify desktop configuration file
  - Open configuration file `vim ~/.local/share/applications/anydesk.desktop`
  - Update configuration

    ```text
    Exec=env GDK_BACKEND=x11 /usr/bin/anydesk %u
    ```

  - Reload desktop database `update-desktop-database ~/.local/share/applications`

### 3.10 Enable Hibernate mode

#### 3.10.1 Check system requirement before setup

- Check power settings in OS

```fish
cat /sys/power/mem_sleep          # [s2idle] only = no S3 available
cat /sys/power/state              # must include "disk"
cat /sys/power/disk               # must not read [disabled]
```

- Check OS partition settings

```fish
free -h                           # RAM size → swap file size
df -h /                           # space for swap file
findmnt -no SOURCE,FSTYPE /       # Partition format
grep ^HOOKS /etc/mkinitcpio.conf  # Find hooks
swapon --show                     # Show swapfile
```

- Swap size: must hold used memory. RAM size is the safe choice. zram cannot be used — it vanishes when power is cut.
- The capacity of the `/` partition needs to be checked. If the available space of the root partition is smaller than the RAM size, you need to switch to another partition with sufficient capacity, usually the `/home/` partition. For example, if the RAM capacity is 16 GB and the available space of the root partition is less than 16 GB, you need to find another partition.

#### 3.10.2 Create a Swapfile

- Create a Swapfile

```fish
sudo dd if=/dev/zero of=/home/swapfile bs=1M count=16384 status=progress
sudo chmod 600 /home/swapfile
sudo mkswap /home/swapfile
sudo swapon --priority 10 /home/swapfile
echo '/home/swapfile none swap defaults,pri=10 0 0' | sudo tee -a /etc/fstab
```

- Replace path storing swapfile `/home/swapfile` if root partition has enough space to store swapfile (Checked in `3.10.1`)
- Check if swapfile is created `swapon --show`
- Check if there is any error after create swapfile `sudo findmnt --verify --verbose`

#### 3.10.3 Configure resume parameters

- Find partition storing Swapfile UUID `findmnt -no UUID /home`
- Find resume offset `sudo filefrag -v /home/swapfile | awk '$1=="0:"{print substr($4, 1, length($4)-2)}'`
- Configure resume parameters
  - Open kernel boot parameters file `vim /etc/kernel/cmdline`
  - Add new line at end of file `root=PARTUUID=<partition_uuid> rw rootfstype=ext4 resume=UUID=<uuid> resume_offset=<offset>`.
    - `partition_uuid`: Copy from existing line in parameter file
    - `uuid`: Partition storing Swapfile UUID, get from command above
    - `offset`: Resume offset, get from command above

#### 3.10.4 Add resume hook

- Open `mkinitcpio` configuration file `vim /etc/mkinitcpio.conf`
- Find `HOOKS` configuration and add `resume` after `filesystems` and before `fsck`. The result should look like this `HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems resume fsck)`
- Rebuild and reboot

```fish
sudo mkinitcpio -P    # -P rebuilds all presets, including linux-lts
sudo reboot
```

- After boot, check if resume configuration is affected

```fish
cat /sys/power/resume         # non-zero, e.g. 259:3
cat /sys/power/resume_offset  # matches what you set
```

- Test if hibernate work `sudo sh -c 'echo disk > /sys/power/state'`
  - After running command, desktop power off immediately
  - After start desktop again, working session is kepts

#### 3.10.5 Setup hibernate command

- Create `logind` service configuration `sudo mkdir -p /etc/systemd/system/systemd-logind.service.d`
- Create new configuration file `sudo vim  /etc/systemd/system/systemd-logind.service.d/override.conf`
- Add configration for above file

  ```ini
  [Service]
  ProtectHome=read-only
  ```

  - Only if Swapfile are stored in `/home/` partition. Because `/home` are isolated in sandbox environment, so `logind` cannot find Swapfile by default. This configuration allow `logind` can read the Swapfile

- Reload systemctl deamon

```fish
sudo systemctl daemon-reload
sudo systemctl restart systemd-logind   # logs you out
```

#### 3.10.6 Add "Hibernate" button

- Install Gnome extenstion "Power Off Options"

## 4. TODOs

- aws cli v2
- install golang
- MCP for AI

### 4.1 Later

- Refactor file document
- Find other theme has light mode
- Write script to apply all setup
