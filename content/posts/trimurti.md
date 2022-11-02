---

title: "Setup trimurti (Debian ARM64)"
date: 2022-07-23T08:00:30+05:30
draft: false
toc: true

---

## Stage 0000: Immediate initial setup


### Set hostname

```bash
hostnamectl set-hostname trimurti
```


### Set timezone

```bash
timedatectl set-timezone Asia/Kolkata
```


### Set DNS Servers

```bash
nmcli connection modify "$(nmcli -g name,device connection show | grep "eth0" | cut -f1 -d":")" ipv4.dns "1.1.1.2,1.0.0.2"
nmcli connection modify "$(nmcli -g name,device connection show | grep "eth0" | cut -f1 -d":")" ipv4.ignore-auto-dns yes
```


### Enable loading the Wireguard kernel module at boot.

```bash
echo "wireguard" | tee /etc/modules-load.d/wireguard.conf
```


### Modify motd

```bash
echo "\n# added by PRATHAM\n/home/pratham/.scripts/_trimurti/motd/show_logs.sh" | tee -a /etc/profile
```


### REBOOT! (hostname needs to come in effect)

```bash
reboot +0
```


### Generate SSH keys

```bash
cd $HOME/.ssh
ssh-keygen -t ed25519 -f flameboi
ssh-keygen -t ed25519 -f gitea
ssh-keygen -t ed25519 -f github
ssh-keygen -t ed25519 -f gitlab
ssh-keygen -t ed25519 -f sentinel
```


### Reboot

```bash
reboot +0
```

---


## Stage 0010


### apt configuration

`sudo` is not needed, switch to `doas`

```bash
echo "Package: sudo
Pin: release *
Pin-Priority: -1" | tee /etc/apt/preferences.d/90_sudo
```

```bash
echo "# this file was edited by Pratham Patel

# bookworm
deb http://deb.debian.org/debian/ bookworm main contrib non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main contrib non-free-firmware


# bookworm-security
deb http://security.debian.org/debian-security bookworm-security main contrib non-free-firmware
deb-src http://security.debian.org/debian-security bookworm-security main contrib non-free-firmware


# bookworm-updates
deb http://deb.debian.org/debian/ bookworm-updates main contrib non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm-updates main contrib non-free-firmware" | tee /etc/apt/sources.list
```


### Package management

```bash
apt-get update
apt-get upgrade
apt-get dist-upgrade
```


### Install packages

```bash
# necessary pkgs
apt install -y bind9-dnsutils console-setup curl fdisk ffmpeg findutils git git-delta git-email git-man libpam-google-authenticator neovim network-manager openssh-client openssh-server openssl plocate rename rsync signify-openbsd tmux tree wget wireguard zsh zsh-autosuggestions zsh-common zsh-syntax-highlighting
systemctl enable --now NetworkManager.service

# monitoring
apt install -y btop htop iotop nload iperf iperf3

# containerisation stuff
apt install -y aardvark-dns bridge-utils podman podman-compose slirp4netns buildah-

# download clients
apt install -y aria2 wget2

# android-stuff
apt install -y adb fastboot

# coreutils-rust
apt install -y bat fd-find ripgrep
#apt install -y skim

# system utils
apt install -y eatmydata hd-idle hdparm smartmontools ssmtp tldr tre-command wakeonlan yt-dlp

# compression
apt install -y tar unrar-free unzip xz-utils zip

# cockpit
apt install -y cockpit cockpit-doc cockpit-machines cockpit-pcp cockpit-networkmanager cockpit-packagekit cockpit-podman cockpit-sosreport cockpit-system cockpit-ws

# optional?
#apt install -y cron

# software devel
apt install -y meld

# kernel-devel
apt install -y autoconf bc bison build-essential cmake fakeroot flex gdb-multiarch libc6-dev libelf-dev libncurses-dev libssl-dev make openocd

# virtualisation
apt install -y libvirt-clients libvirt-daemon-system qemu-efi-aarch64 qemu-system qemu-system-common qemu-system-gui qemu-system-misc qemu-user qemu-user-static qemu-utils
#libvirt-daemon-kvm

# network filesystems
apt install -y cifs-utils nfs-common nfs-kernel-server nfswatch
# try to use NFS
#apt install -y samba samba-common samba-common-bin

# zfs
apt install -y dpkg-dev linux-headers-arm64
DEBIAN_FRONTEND=noninteractive apt install -y zfs-dkms zfs-zed zfsutils-linux
```


### Add user

```bash
useradd -m -G adb,audio,cdrom,dip,floppy,games,netdev,plugdev,sys,systemd-journal,uucp,video -s $(which zsh) pratham
usermod --password $(echo asdf | openssl passwd -1 -stdin) pratham
passwd -e pratham

echo "permit persist keepenv pratham" | tee -a /etc/doas.conf
```


### Reboot

```bash
reboot +0
```

---


## Stage 0011: Install stuff


### vim-plug (Neovim)

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

**Open `nvim` and type `:PlugInstall`**


### Enable systemd services

```bash
sudo systemctl enable cockpit.socket cockpit.service 
sudo systemctl enable podman.socket
```

---


## Stage 0100: ZFS


### Enable necessary services

```bash
sudo systemctl enable zfs-import-cache.service
sudo systemctl enable zfs-import-scan.service
sudo systemctl enable zfs-mount.service
sudo systemctl enable zfs-share.service
sudo systemctl enable zfs-zed.service
sudo systemctl enable zfs.target
```


### Make sure an import cache file exists

```bash
sudo zpool set cachefile=/etc/zfs/zpool.cache brahmaand
```


### Creating a new zpool?

```bash
sudo zpool create -o ashift=12 -o autotrim=on brahmaand /dev/sda

sudo zfs set atime=off brahmaand
sudo zfs set primarycache=all brahmaand
sudo zfs set recordsize=1M brahmaand
sudo zfs set xattr=sa brahmaand

sudo zfs create brahmaand/containers
sudo zfs create brahmaand/containers/volumes
sudo zfs create brahmaand/containers/volumes/blog
sudo zfs create brahmaand/containers/volumes/caddy
sudo zfs create brahmaand/containers/volumes/mach

sudo zfs create brahmaand/containers/volumes/gitea
sudo zfs create brahmaand/containers/volumes/gitea/database
sudo zfs set recordsize=8K brahmaand/containers/volumes/gitea/database

sudo zfs create brahmaand/containers/volumes/nextcloud
sudo zfs create brahmaand/containers/volumes/nextcloud/database
sudo zfs set recordsize=8K brahmaand/containers/volumes/nextcloud/database

sudo zfs create brahmaand/torrents
sudo zfs set recordsize=16K brahmaand/torrents
sudo zfs create brahmaand/torrents/downloads
sudo zfs create brahmaand/torrents/downloads/.incomplete
sudo zfs create brahmaand/torrents/config

sudo chown pratham:pratham -vR /brahmaand
sudo chown pratham:pratham -vR /brahmaand/torrents

sudo zfs allow -u pratham create,destroy,mount,snapshot,send,hold brahmaand

sudo zpool export brahmaand

sudo zpool import
sudo zpool import -d /dev/disk/by-id <pool-id>

sudo zpool set cachefile=/etc/zfs/zpool.cache brahmaand

zpool status -v
zfs list

sudo zpool scrub brahmaand
```


### Reboot

```bash
sudo reboot +0
```

---


## Stage 0101: Containers


### Pull images

```bash
sleep 60 && podman pull docker.io/library/postgres:14-alpine
sleep 60 && podman pull docker.io/library/caddy:alpine
sleep 60 && podman pull docker.io/klakegg/hugo:ext-debian
sleep 60 && podman pull docker.io/library/nextcloud:production
sleep 60 && podman pull docker.io/klakegg/hugo:alpine
sleep 60 && podman pull docker.io/gitea/gitea:latest
```


### Get fs ready

```bash
sudo zfs set atime=off brahmaand
sudo zfs set primarycache=all brahmaand
sudo zfs set recordsize=1M brahmaand
sudo zfs set xattr=sa brahmaand

sudo zfs create brahmaand/containers
sudo zfs create brahmaand/containers/volumes
sudo zfs create brahmaand/containers/volumes/blog
sudo zfs create brahmaand/containers/volumes/caddy
sudo zfs create brahmaand/containers/volumes/gitea
sudo zfs create brahmaand/containers/volumes/mach
sudo zfs create brahmaand/containers/volumes/nextcloud

sudo zfs create brahmaand/torrents
sudo zfs set recordsize=16K brahmaand/torrents
sudo zfs create brahmaand/torrents/downloads
sudo zfs create brahmaand/torrents/downloads/.incomplete
sudo zfs create brahmaand/torrents/config

sudo chown pratham:pratham -vR /brahmaand/containers/volumes
sudo chown pratham:pratham -vR /brahmaand/torrents

sudo zfs allow -u pratham send,snapshot,hold brahmaand
```


### Create directories for mounting container volumes

```bash
mkdir -vp /brahmaand/containers/volumes/caddy/{site,ssl/{private,certs},caddy_{data,config}}
mkdir -vp /brahmaand/containers/volumes/gitea/{database,web}
mkdir -vp /brahmaand/containers/volumes/nextcloud/{database,web}
```


### Enable workaround for "root-less containers can't ping hosts"

```bash
grep net.ipv4.ping_group_range /etc/sysctl.conf || echo "net.ipv4.ping_group_range=0 $(grep pratham /etc/subuid | awk -F ":" '{print $2 + $3}')" | sudo tee -a /etc/sysctl.conf
```


### Hugo

```bash
git clone --recursive git@gitlab.com:thefossguy/blog.git /brahmaand/containers/volumes/blog
cd /brahmaand/containers/volumes/blog
git remote rm origin
git remote add origin git@git.thefossguy.com:thefossguy/blog.git

git clone --recursive git@gitlab.com:thefossguy/machines.git /brahmaand/containers/volumes/mach
cd /brahmaand/containers/volumes/mach
git remote rm origin
git remote add origin git@git.thefossguy.com:thefossguy/machines.git
```


### Caddy

1. Visit the [Cloudflare dashboard](https://dash.cloudflare.com/)
2. Select domain
3. On the left sidebar, select 'SSL/TLS'. Make sure _Encryption Mode_ is **Full (strict)**.
4. Under 'SSL/TLS', goto '**Origin Server**'.
5. Create a new Certificate **with default values**.
6. Populate `/brahmaand/containers/volumes/caddy/ssl/{certs/certificate.pem,private/key.pem}`.
7. Change permissions for `/brahmaand/containers/volumes/caddy/ssl/private`.

```bash
chmod 700 -v /brahmaand/containers/volumes/caddy/ssl/private
chmod 600 -v /brahmaand/containers/volumes/caddy/ssl/private/key.pem
```

Copy `Caddyfile` to the appropriate directory.

```bash
cp -v Caddyfile /brahmaand/containers/volumes/caddy/
```


### Cockpit

something-something enable SSL for cockpit

```bash
sudo cp cockpit.conf /etc/cockpit/cockpit.conf
```


### Generate container secrets for passwords

```bash
openssl rand -base64 20 | podman secret create gitea_database_user_password -
openssl rand -base64 20 | podman secret create nextcloud_database_user_password -
```


### Enable user lingering

```bash
sudo loginctl enable-linger
```


### Start containers

```bash
podman-compose -f master-compose.yml up -d
```


### Generate systemd files and enable them

```bash
cd $HOME/.config/systemd/user

podman generate systemd -f --name caddy-vishwambhar --new
podman generate systemd -f --name gitea-chitragupta --new
podman generate systemd -f --name gitea-govinda --new
podman generate systemd -f --name hugo-mahayogi --new
podman generate systemd -f --name hugo-vaikunthnatham --new
podman generate systemd -f --name nextcloud-chitragupta --new
podman generate systemd -f --name nextcloud-govinda --new

systemctl --user daemon-reload

systemctl --user enable container-caddy-vishwambhar container-gitea-chitragupta container-gitea-govinda container-hugo-mahayogi container-hugo-vaikunthnatham container-nextcloud-chitragupta container-nextcloud-govinda
```

---

## Stage 0111: sharing zpool

### exports

[docs](https://manpages.debian.org/testing/nfs-kernel-server/exports.5.en.html)

Add the following lines to the `/etc/exports` file:

```
/brahmaand 10.0.0.0/8(ro,insecure,subtree_check,crossmnt)
```

Then, export it.

```bash
sudo exportfs -rva
```

---

## Stage 1000: cron

### user crontab

```bash
# always add ">/dev/null 2>&1" at the end of cronjobs
# to prevnet a `dead.letter` in $HOME/


# check if containers are running or not; restart if stopped
*/5 * * * * bash /home/pratham/.scripts/_trimurti/cron/pratham/maintenance.sh >/dev/null 2>&1


# run Nextcloud cron
*/5 * * * * podman exec -u www-data nextcloud-govinda /usr/local/bin/php -f /var/www/html/cron.php >/dev/null 2>&1


# Nextcloud: scan files for all users and perform cleanup
10 */2 * * * podman exec -u www-data nextcloud-govinda /usr/local/bin/php -f /var/www/html/occ files:scan --all >/dev/null 2>&1
40 */2 * * * podman exec -u www-data nextcloud-govinda /usr/local/bin/php -f /var/www/html/occ files:cleanup >/dev/null 2>&1
```


### root crontab

```bash
# always add ">/dev/null 2>&1" at the end of cronjobs
# to prevnet a `dead.letter` in $HOME/


# update fs database every 6 hours
* */6 * * * updatedb >/dev/null 2>&1


# create zfs snapshots every Friday
0 0 * * 5 bash /home/pratham/.scripts/_trimurti/cron/root/zfs-bak.sh >/dev/null 2>&1


# start scrub
# on the first Friday of every month
# at 2100 hours
0 21 * * 5 [ $(date +\%d) -le 07 ] && /sbin/zpool scrub >/dev/null 2>&1
```
