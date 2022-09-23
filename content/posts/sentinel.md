---

title: "Setup sentinel (Ubuntu Server arm64)"
date: 2022-07-23T08:00:20+05:30
draft: false
toc: true

---

## Stage 0000: Flash SD Card

Verify the checksum first

```bash
echo "<CHECKSUM> *ubuntu-VERSION-preinstalled-desktop-arm64+raspi.img.xz" | shasum --check
```

Flash the SD Card

```bash
xzcat ubuntu-VERSION-preinstalled-desktop-arm64+raspi.img.xz | sudo dd status=progress bs=4M of=/dev/XXX

sync && sync && sync

sudo eject /dev/XXX
```


## Stage 0001: Immediate initial setup

### Set hostname

```bash
sudo hostnamectl set-hostname sentinel
```

### Set timezone

```bash
sudo timedatectl set-timezone Asia/Kolkata
```

### Set DNS Servers

```bash
nmcli connection modify "$(nmcli -g name,device connection show | grep "eth0" | cut -f1 -d":")" ipv4.dns "1.1.1.2,1.0.0.2"
nmcli connection modify "$(nmcli -g name,device connection show | grep "eth0" | cut -f1 -d":")" ipv4.ignore-auto-dns yes
```

### REBOOT! (hostname needs to come in effect)

```bash
sudo reboot +0
```

### Stop, mask and disable services

```bash
sudo systemctl stop sleep.target suspend.target hibernate.target hybrid-sleep.target
sudo systemctl disable sleep.target suspend.target hibernate.target hybrid-sleep.target
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### Change "boot args"

```bash
sudoedit /boot/firmware/usercfg.txt
```

Add the following lines to it:

```
hdmi_force_hotplug=1
disable_overscan=1

arm_freq=2000
```


### Generate SSH Keys

```bash
cd $HOME/.ssh
ssh-keygen -t ed25519 -f bluefeds
ssh-keygen -t ed25519 -f gitea
ssh-keygen -t ed25519 -f github
ssh-keygen -t ed25519 -f gitlab
```

### Reboot

```bash
sudo reboot +0
```


## Stage 0010


### Upgrade packages

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Reboot

```bash
sudo reboot +0
```

## Stage 0011

### Install packages

```bash
sudo apt-get install apache2 aria2 bat bc cifs-utils cmatrix curl exfat-fuse exfat-utils fd-find ffmpeg findutils git glances hdparm htop iotop iperf3 jq libpam-google-authenticator locate mediainfo mpv neofetch neovim nload openssh-server python3 python3-pip rename rsync samba samba-common-bin smartmontools speedtest-cli tldr tmux transmission-cli transmission-common transmission-daemon tree unrar unzip valgrind vim webp wget wget2 wireguard xz-utils yt-dlp zfs-dkms zip zsh zsh-autosuggestions zsh-syntax-highlighting

chsh -s $(which zsh) $(whoami)
```


### vim-plug (Neovim)

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

**Open `nvim` and type `:PlugInstall`**


## Stage 0100: ZFS

### Install ZFS if not already installed

Don't use `zfsutils-linux`, it depends on Ubuntu's kernel inclusions 
```bash
sudo apt-get install zfs-dkms
```

### Enable necessary services

```bash
sudo systemctl enable zfs-import-cache.service
sudo systemctl enable zfs-import-scan.service
sudo systemctl enable zfs-mount.service
sudo systemctl enable zfs-share.service
sudo systemctl enable zfs.target
sudo systemctl enable zfs-zed.service
```


### Make sure an import cache file exists

```bash
sudo zpool set cachefile=/etc/zfs/zpool.cache vaikuntham
```


### Creating a new zpool?

```bash
sudo zpool create -o ashift=12 vaikuntham /dev/sda

sudo zfs set atime=off vaikuntham
sudo zfs set primarycache=all vaikuntham
sudo zfs set recordsize=1M vaikuntham
sudo zfs set xattr=sa vaikuntham

sudo zfs create vaikuntham/backups
sudo zfs create vaikuntham/torrents
sudo zfs set recordsize=16K vaikuntham

sudo chown ubuntu:ubuntu -vR /vaikuntham

sudo zfs allow -u ubuntu compression,mountpoint,create,mount,receive vaikuntham

sudo zpool export vaikuntham

sudo zpool import
sudo zpool import -d /dev/disk/by-id <pool-id>

sudo zpool set cachefile=/etc/zfs/zpool.cache vaikuntham

zpool status -v
zfs list

sudo zpool scrub vaikuntham
```

### Reboot

```bash
sudo reboot +0
```


## Stage 0101: cron

### user crontab

```bash
# pause torrents as a prep for zfs scrub
*/10 18 * * 5 [ $(date +\%d) -le 07 ] && bash /home/ubuntu/.scripts/transmission/pause_all.sh

#0 0 * * * bash /home/ubuntu/.scripts/transmission/change_bandwidth_priority.sh

# remove after exam ends
#*/2 6,7 * * * bash /home/ubuntu/.scripts/transmission/pause_downloading.sh
#*/20 20 * * * bash /home/ubuntu/.scripts/transmission/resume_all.sh

# maintenance script
# [[ if zpool scrub is not running ]]
#   -> [[ if containers are not running ]]
#     -> start containers
* * * * * bash /home/ubuntu/.scripts/cron/pratham/maintenance.sh
```


### root crontab

```bash
# ZFS snapshots
#0 * * * * bash /home/ubuntu/.scripts/cronjobs/root/zfs-bak.sh

# start scrub
# on every first Friday of the month
# at 1845 hours
45 18 * * 5 [ $(date +\%d) -le 07 ] && /sbin/zpool scrub libertine

# wireguard (ProtonVPN)
#* * * * * bash /home/ubuntu/.scripts/cronjobs/root/proton-vpn.sh

# backup .torrent files
# chmod and chown
# rm .macos files
30 18 * * * bash /home/ubuntu/.scripts/cronjobs/root/midnight.sh

# update fs database every 6 hours
* */6 * * * updatedb

# maintenance script
#0 20 * * * bash /home/ubuntu/.scripts/cron/root/maintenance.sh
```
