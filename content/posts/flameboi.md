---

title: "Setup flameboi (Pop OS)"
date: 2022-07-23T08:00:00+05:30
draft: false
toc: true

---

## Stage 0000: Initial setup


### Set hostname

```bash
sudo hostnamectl set-hostname flameboi
```


### Set timezone

```bash
sudo timedatectl set-timezone Asia/Kolkata
```


### Set DNS Server

```bash
temp_var=$(nmcli -g name,device connection show | grep "enp" | cut -f1 -d":")
nmcli connection modify "$temp_var" ipv4.dns "1.1.1.2,1.0.0.2"
nmcli connection modify "$temp_var" ipv4.ignore-auto-dns yes
```


### Unmask (and enable) NVIDIA services

```bash
sudo systemctl unmask nvidia-suspend nvidia-hibernate nvidia-resume
sudo systemctl enable nvidia-suspend nvidia-hibernate nvidia-resume

# sudo apt autoremove "*nvidia"
# sudo apt install system76-driver-nvidia
```


### Generate SSH keys

```bash
cd $HOME/.ssh
ssh-keygen -t ed25519 -f bluefeds
ssh-keygen -t ed25519 -f gitea
ssh-keygen -t ed25519 -f github
ssh-keygen -t ed25519 -f gitlab
ssh-keygen -t ed25519 -f sentinel
```

### Reboot

```bash
systemctl reboot
```


## Stage 0001

### Upgrade packages

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Reboot

```bash
systemctl reboot
```


## Stage 0010


### Install GNOME extensions

```bash
sudo apt install gnome-shell-extensions -y
```

A few extensions:

 - [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/) `clipboard-indicator@tudmotu.com`
 - [Impatience](https://extensions.gnome.org/extension/277/impatience/) `impatience@gfxmonk.net` (Set it to 0.5x)
 - [Net speed Simplified](https://extensions.gnome.org/extension/3724/net-speed-simplified/) `netspeedsimplified@prateekmedia.extension`
 - [Permanent notifications](https://extensions.gnome.org/extension/41/permanent-notifications/) `permanent-notifications@bonzini.gnu.org`


### Install packages

```bash
sudo apt-get install adb alacritty aria2 autoconf barrier bat bc bison bridge-utils btop build-essential cifs-utils cmake cmatrix crossbuild-essential-{arm{64,hf,el},amd64,riscv64} curl ethtool exfat-fuse fakeroot fastboot fdisk ffmpeg flex fonts-firacode fonts-fork-awesome gdb-multiarch git handbrake hdparm htop imagemagick iotop iperf iperf3 libc6-dev libelf-dev libncurses-dev libnotify-bin libpam-google-authenticator libssl-dev libvirt-clients libvirt-daemon-system linux-headers-generic linux-headers-$(uname -r) linux-tools-$(uname -r) linux-tools-common linux-tools-generic locate lsb-release make mediainfo meld mlocate mpv neofetch neovim nethogs nload nvme-cli obs-plugins obs-studio openocd opensbi openssh-client openssh-server python3 python3-pip qemu qemu-efi-aarch64 qemu-efi-arm qemu-system-common qemu-kvm qemu-system-arm qemu-system-misc qemu-system-x86 qemu-utils rar ripgrep rsync signify-openbsd smartmontools speedtest-cli tar tmux transmission-cli tree u-boot-qemu unrar unzip valgrind vim virt-manager vlc wakeonlan webp wget wget2 xsel xz-utils yt-dlp zfs-dkms zip zsh zsh-autosuggestions zsh-syntax-highlighting
```

#### Fancy a window manager?

```bash
sudo apt-get install breeze-cursor-theme bspwm dunst feh gtk-chtheme i3lock-fancy jq libnotify-bin lxappearance picom polybar rofi socat sxhkd wmctrl xcursor-themes

echo "gtk-application-prefer-dark-theme=true" | tee $HOME/.config/gtk-3.0/settings.ini
```

**_Temporarily_ export var `DOTFILES_DIR` pointing to wherever [_dotfiles_](https://git.thefossguy.com/thefossguy/dotfiles) is cloned.**

```bash
sudo cp -v $DOTFILES_DIR/_OTHER/flameboi/usr/share/xsessions/bspwm.desktop /usr/share/xsessions/
```


### vim-plug (Neovim)

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

nvim +'PlugInstall' +'q' +'q'
```


### Rust setup

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

rustup default stable
rustup component add rust-src rust-analyzer rust-analysis

cargo install cargo-outdated cargo-tree
```

### Flatpak

```bash
flatpak install flathub com.bitwarden.desktop com.brave.Browser com.dangeredwolf.ModernDeck com.github.micahflee.torbrowser-launcher com.github.tchx84.Flatseal com.sublimetext.three flathub io.gitlab.librewolf-community org.ksnip.ksnip org.mozilla.firefox org.mozilla.Thunderbird org.telegram.desktop
```

### Virtualization

Add `pratham` to necessary groups

```bash
sudo adduser pratham libvirt
sudo adduser pratham kvm
```

Change backend

```bash
sudo sed -i 's/FirewallBackend=nftables/FirewallBackend=iptables/g' /etc/firewalld/firewalld.conf
```

Enable autostart of `default` network

```bash
sudo virsh net-autostart default
sudo virsh net-list --all
```

Edit `/etc/libvirt/qemu.conf` and modify it as follows:

```
user = "pratham"
[...]
roup = "pratham"
```

---

List current pools

```bash
sudo virsh pool-list
```

Delete `default` pool

```bash
sudo virsh pool-destroy default
```

Undefine the pool

```bash
sudo virsh pool-undefine default
```

Use `/flameboi_st/vm-store` as `default`'s path

```bash
sudo virsh pool-define-as --name default --type dir --target /flameboi_st/vm-store
```

Enable autostarting `default` storage pool

```bash
sudo virsh pool-autostart default
sudo virsh pool-start default
```

Verify:

```bash
sudo virsh pool-list
```

Restart the `libvirtd` service

```bash
sudo systemctl restart libvirtd
```

### Make alacritty the default terminal

```bash
gsettings set org.gnome.desktop.default-applications.terminal exec '/usr/bin/alacritty'
sudo update-alternatives --config x-terminal-emulator
```

### Don't crawl specific directories with updatedb

**Add `/heathen_disk/personal/media/camera_roll` to the `PRUNEPATHS` value in `/etc/updatedb.conf`.**



## Stage 0011: Cron setup

### `pratham`'s cron

```cron
# empty for now
```

### `root`'s cron

```cron
# empty for now
```


## Stage 0100: Enable services

### Enable HDD-related services

**_Temporarily_ export var `DOTFILES_DIR` pointing to wherever [_dotfiles_](https://git.thefossguy.com/thefossguy/dotfiles) is cloned.**

```bash
sudo cp -v $DOTFILES_DIR/_OTHER/flameboi/etc/systemd/system/*.service /etc/systemd/system/
sudo systemctl enable hdd-standby-on-boot.service hdd-standby-on-resume.service
```

## Stage 0101: Window Manager (bspwm)

### Use `$HOME/.xinitrc` for starting bspwm from GDM

Edit `/usr/share/xsessions/bspwm.desktop`

```
[Desktop Entry]
Name=bspwm
Comment=Binary space partitioning window manager
Exec=/home/pratham/.xinitrc
Type=Application
```

### Chrome-based browser scaling

Add the following line to `/etc/alternatives/brave-browser`

```
[...]
"$HERE/brave" "$@" --force-device-scale-factor=1.25 || true
```

ADd the following line to `/etc/alternatives/google-chrome`

```
[...]
exec -a "$0" "$HERE/chrome" "$@" --force-device-scale-factor=1.25
```

## Stage 0110: Sublime Text 3

### Install extensions

 - [Jedi](https://packagecontrol.io/packages/Jedi%20-%20Python%20autocompletion)
 - [TwoDark](https://packagecontrol.io/packages/Theme%20-%20TwoDark)

### Edit preferences

Preferences > Settings

```
	"ui_scale": 1.25,
	"font_face": "Fira Code",
	"font_options":["subpixel_antialias"],
	"atomic_save": true,
	"auto_complete_delay": 1,
	"caret_extra_width": 1,
	"color_scheme": "Packages/Theme - TwoDark/TwoDark.tmTheme",
	"draw_shadows": false,
	"ensure_newline_at_eof_on_save": true,
	"fade_fold_buttons": true,
	"highlight_modified_tabs": true,
	"indent_guide_options":["draw_active"],
	"save_on_focus_lost": true,
	"shift_tab_unindent": true,
	"show_encoding": true,
	"show_line_endings": true,
	"theme": "TwoDark.sublime-theme",
	"auto_complete_triggers": [{"selector": "source.python", "characters": ".",}],
	"ignored_packages": ["Vintage"],
	"font_size": 12,
```

### Edit key bindings

Preferences > Key bindings

```
	{"keys": ["ctrl+tab"], "command": "next_view"},
	{"keys": ["ctrl+shift+tab"], "command": "prev_view"},
```

## Stage 0111: ZFS Setup

### Create zpool

```
sudo zpool create -o ashift=12 bhugol raidz1 /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

### Set global zpool properties

```
sudo zfs set atime=off bhugol
sudo zfs set checksum=sha512 bhugol
sudo zfs set compression=zstd-19 bhugol
sudo zfs set primarycache=all bhugol
sudo zfs set recordsize=1M bhugol
sudo zfs set snapdir=hidden bhugol
sudo zfs set xattr=sa bhugol
```

### Create ZFS datasets

```
sudo zfs create bhugol/media
sudo zfs create bhugol/media/camera_roll
sudo zfs create bhugol/media/movies
sudo zfs create bhugol/media/tv_series

sudo zfs create bhugol/backup
sudo zfs create bhugol/backup/barbet
sudo zfs create bhugol/backup/bluefeds
sudo zfs create bhugol/backup/flameboi
sudo zfs create bhugol/backup/ringmaster
sudo zfs create bhugol/backup/sentinel

sudo zpool export bhugol

sudo zpool import	*trial run to get the pool-id*
sudo zpool import -d /dev/disk/by-id <pool-id>
```

### Own your mount points

```
sudo chown pratham:pratham -vR /bhugol
```

### Add bi-monthly cron job for ZFS scrub

`sudo crontab -e`

```bash
0 0 1,15 * * /usr/sbin/zpool scrub bhugol
```

### Set ZFS parameters

`sudoedit /etc/modprobe.d/zfs.conf`

```
options zfs zfs_scan_idle = 0
options zfs zfs_scrub_delay = 0
options zfs zfs_scan_min_time_ms = 5000
```
