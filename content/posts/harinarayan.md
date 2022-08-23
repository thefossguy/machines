---

title: "Setup harinarayan (Pop OS)"
date: 2022-07-23T08:00:00+05:30
draft: false
toc: true

---

## Stage 0000: Initial setup


### Set hostname

```bash
sudo hostnamectl set-hostname harinarayan
```


### Set timezone

```bash
sudo timedatectl set-timezone Asia/Kolkata
```


### Set DNS Server

```bash
nmcli connection modify "$(nmcli -g name,device connection show | grep "eth0" | cut -f1 -d":")" ipv4.dns "1.1.1.2,1.0.0.2"
nmcli connection modify "$(nmcli -g name,device connection show | grep "eth0" | cut -f1 -d":")" ipv4.ignore-auto-dns yes
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
ssh-keygen -t ed25519 -f adinath
ssh-keygen -t ed25519 -f balaji
ssh-keygen -t ed25519 -f gitea
ssh-keygen -t ed25519 -f github
ssh-keygen -t ed25519 -f gitlab
```

### Reboot

```bash
systemctl reboot
```


## Stage 0010

### Upgrade packages

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Reboot

```bash
systemctl reboot
```


## Stage 0011


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
sudo apt-get install adb alacritty aria2 autoconf barrier bat bc bison bridge-utils btop build-essential cifs-utils cmake cmatrix crossbuild-essential-armhf curl ethtool exfat-fuse exfat-utils fakeroot fastboot fdisk ffmpeg flex fonts-firacode fonts-fork-awesome gdb-multiarch git handbrake hdparm htop imagemagick iotop iperf iperf3 libc6-dev libelf-dev libncurses-dev libncurses5-dev libnotify-bin libpam-google-authenticator libssl-dev libvirt-clients libvirt-daemon-system linux-headers-5.16.19-76051619-generic linux-headers-$(uname -r) linux-tools-$(uname -r) linux-tools-common linux-tools-generic locate lsb-release make mediainfo mlocate mpv ncurses-dev neofetch neovim nethogs nload nodejs nvme-cli obs-plugins obs-studio openocd openssh-client openssh-server python3 python3-pip qemu qemu-efi-aarch64 qemu-efi-arm qemu-kvm qemu-system-arm qemu-system-x86 qemu-utils rar ripgrep rsync smartmontools speedtest-cli tar thunderbird tmux transmission-cli tree unrar unzip valgrind vim virt-manager vlc wakeonlan webp wget xsel xz-utils yt-dlp zfs-dkms zip zsh zsh-autosuggestions zsh-syntax-highlighting
```

**linux-headers-$(uname -r) linux-tools-$(uname -r)**

#### Fancy a window manager?

```bash
sudo apt-get install breeze-cursor-theme bspwm dunst feh gtk-chtheme i3lock-fancy jq libnotify-bin lxappearance picom polybar rofi socat sxhkd wmctrl xcursor-themes

echo "gtk-application-prefer-dark-theme=true" | tee $HOME/.config/gtk-3.0/settings.ini
```

### vim-plug (Neovim)

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

**Open `nvim` and type `:PlugInstall`**


### Install rustup

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup component add rust-analysis rust-src
```

### Flatpak

```bash
flatpak install flathub com.bitwarden.desktop com.brave.Browser com.dangeredwolf.ModernDeck com.github.micahflee.torbrowser-launcher com.github.tchx84.Flatseal com.sublimetext.three org.ksnip.ksnip org.mozilla.firefox org.telegram.desktop
```

### Virtualization

Add `pratham` to necessary groups

```bash
sudo adduser pratham libvirt
sudo adduser pratham kvm
```

Edit `/etc/libvirt/qemu.conf` and modify it as follows:

```
user = "pratham"
[...]
roup = "pratham"
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

## Stage 0100: Window Manager (bspwm)

### Use `$HOME/.xinitrc` for starting bspwm from GDM

Edit `/usr/share/xsessions/bspwm.desktop`

```
[Desktop Entry]
Name=bspwm
Comment=Binary space partitioning window manager
Exec=/home/shivoham/.xinitrc
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

## Stage 0101: Sublime Text 3

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
