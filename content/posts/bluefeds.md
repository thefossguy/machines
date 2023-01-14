---

title: "Setup bluefeds (Fedora Server arm64)"
date: 2022-07-23T08:00:30+05:30
draft: false
toc: true

---

## Stage 0000: Flash SD Card

Verify the checksum first

```bash
echo "<CHECKSUM> *Fedora-Server-VERSION.aarch64.raw.xz" | shasum --check
```

Flash the SD Card

```bash
xzcat Fedora-Server-VERSION.aarch64.raw.xz | sudo dd status=progress bs=4M of=/dev/XXX
sync && sync && sync
sudo lvchange -an /dev/fedora_fedora/root
sudo eject /dev/XXX
```

---


## Stage 0001: Immediate initial setup


### Set hostname

```bash
sudo hostnamectl set-hostname bluefeds
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


### Enable loading the Wireguard kernel module at boot.

```bash
echo "wireguard" | sudo tee /etc/modules-load.d/wireguard.conf
```


### Change systemd/journald behaviour

```bash
sudo sed -i 's/#Storage=/Storage=persistent/g' /etc/systemd/journald.conf
sudo sed -i 's/#Compress=/Compress=yes/g' /etc/systemd/journald.conf
sudo sed -i 's/#SystemMaxUse=/SystemMaxUse=1000M/g' /etc/systemd/journald.conf
sudo sed -i 's/#RuntimeMaxUse=/RuntimeMaxUse=200M/g' /etc/systemd/journald.conf
```


### SSH hardening

```bash
ssh pratham@localhost
exit

vim ~/.ssh/authorized_keys
chmod 644 ~/.ssh/authorized_keys

sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/g' /etc/ssh/sshd_config
sudo sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords no/g' /etc/ssh/sshd_config
sudo sed -i 's/#ClientAliveInterval 0/ClientAliveInterval 300/g' /etc/ssh/sshd_config
sudo sed -i 's/#ClientAliveCountMax 3/ClientAliveCountMax 2/g' /etc/ssh/sshd_config
sudo sed -i 's/#X11Forwarding no/X11Forwarding no/g' /etc/ssh/sshd_config

sudo systemctl restart sshd.service
```


### Modify motd

```bash
echo -e "\n# added by PRATHAM\n/home/pratham/.scripts/_bluefeds/motd/show_logs.sh" | sudo tee -a /etc/profile
```


### REBOOT! (hostname needs to come in effect)

```bash
sudo reboot +0
```


### Expand the rootfs

```bash
# Fedora Server (LVM + XFS)
sudo growpart /dev/mmcblk0 3
sudo pvresize /dev/mmcblk0p3
sudo lvextend /dev/fedora_fedora/root -l+100%FREE
sudo xfs_growfs -d /

# RHEL clone (Rocky Linux)
sudo rootfs-expand
```


### DNF

Parallel downloads: 20 (lol)

Exclude package `shim-aa64` (causes uboot to panic)

```bash
echo -ne "\nmax_parallel_downloads=20\nlog_compress=True\nfastestmirror=False" | sudo tee -a /etc/dnf/dnf.conf

# RHEL EPEL
sudo subscription-manager repos --enable codeready-builder-for-rhel-9-$(arch)-rpms
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo /usr/bin/crb enable

sudo dnf clean all
```


### Remove Kernel cmdline args

```bash
sudo grubby --update-kernel=ALL --args=crashkernel=512M
sudo grubby --update-kernel=ALL --remove-args=quiet
sudo grubby --update-kernel=ALL --remove-args=rhgb
```


### Generate SSH keys

```bash
cd $HOME/.ssh
ssh-keygen -t ed25519 -f flameboi
ssh-keygen -t ed25519 -f gitea
ssh-keygen -t ed25519 -f github
ssh-keygen -t ed25519 -f gitlab
ssh-keygen -t ed25519 -f sentinel
ssh-keygen -t ed25519 -f zfs
```


### Reboot

```bash
sudo reboot +0
```

---


## Stage 0010

### Upgrade packages

```bash
sudo dnf --refresh update
```


### Reboot

```bash
sudo reboot +0
```

---


## Stage 0011: Install stuff

### Install packages

```bash
sudo dnf install aardvark-dns bat bind-utils btop cockpit console-setup fd-find ffmpeg-free git hdparm htop insights-client iotop libavcodec-free libavfilter-free libavformat-free libavutil-free libpostproc-free libswresample-free libswscale-free mlocate neovim nfs-utils nload openssh-server podman podman-compose ripgrep rsync samba-common slirp4netns smartmontools tmux tree unrar unzip util-linux-user wget yt-dlp yt-dlp-zsh-completion zsh
```

### Enable insights

```bash
sudo insights-client --register
```

### Change shell to zsh

```bash
sudo chsh -s $(which zsh) pratham
```


### vim-plug (Neovim)

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

nvim +'PlugInstall' +'q' +'q'
```


### Enable systemd services

```bash
sudo systemctl enable cockpit.socket cockpit.service 
sudo systemctl enable crond.service
sudo systemctl enable podman.socket
```


### Add ports (for services)

```bash
sudo firewall-cmd --add-service=cockpit --permanent
sudo firewall-cmd --reload
```

---


## Stage 0100: ZFS

### Compile the latest version of ZFS

Get the latest GA release tag [from here](https://github.com/openzfs/zfs/tags).

```bash
git clone --depth 1 --branch <latest_tag_name> https://github.com/openzfs/zfs

sudo dnf install autoconf automake dkms elfutils-libelf-devel gcc kernel-devel kernel-rpm-macros libaio-devel libattr-devel libblkid-devel libcurl-devel libffi-devel libtirpc-devel libtool libudev-devel libuuid-devel make openssl-devel python3 python3-cffi python3-devel python3-packaging python3-setuptools rpm-build zlib-devel

cd zfs
sh autogen.sh
./configure
make -j1 rpm-utils rpm-dkms

sudo yum localinstall *.$(uname -p).rpm *.noarch.rpm

echo "zfs" | sudo tee /etc/modules-load.d/zfs.conf
```


### Enable necessary services

```bash
sudo systemctl enable zfs-zed.service zfs-import-cache.service zfs-import-scan.service zfs-mount.service zfs-share.service zfs-volume-wait.service zfs-import.target zfs-volumes.target zfs.target

sudo systemctl disable zfs-scrub-monthly@.timer zfs-scrub-weekly@.timer zfs-scrub@.service
```


### Make sure an import cache file exists

```bash
sudo zpool set cachefile=/etc/zfs/zpool.cache trayimurti
```


### Creating a new zpool?

```bash
sudo zpool create -o ashift=12 -o autotrim=on trayimurti mirror /dev/sda /dev/sdb

sudo zfs set atime=off trayimurti
sudo zfs set checksum=on trayimurti
sudo zfs set compression=zstd trayimurti
sudo zfs set primarycache=all trayimurti
sudo zfs set recordsize=1M trayimurti
sudo zfs set snapdir=hidden trayimurti
sudo zfs set xattr=sa trayimurti

sudo zfs create trayimurti/containers
sudo zfs create trayimurti/containers/volumes
sudo zfs create trayimurti/containers/volumes/blog
sudo zfs create trayimurti/containers/volumes/caddy
sudo zfs create trayimurti/containers/volumes/mach
sudo zfs create trayimurti/containers/volumes/gotify
sudo zfs create trayimurti/containers/volumes/uptimekuma
sudo zfs set copies=3 trayimurti/containers/volumes/uptimekuma

sudo zfs create trayimurti/containers/volumes/gitea
sudo zfs set copies=3 trayimurti/containers/volumes/gitea
sudo zfs create trayimurti/containers/volumes/gitea/database
sudo zfs set recordsize=8K trayimurti/containers/volumes/gitea/database

sudo zfs create trayimurti/containers/volumes/nextcloud
sudo zfs set copies=3 trayimurti/containers/volumes/nextcloud
sudo zfs create trayimurti/containers/volumes/nextcloud/database
sudo zfs set recordsize=8K trayimurti/containers/volumes/nextcloud/database

sudo zfs create trayimurti/torrents
sudo zfs set recordsize=16K trayimurti/torrents
sudo zfs create trayimurti/torrents/downloads
sudo zfs create trayimurti/torrents/config

sudo zfs allow -u pratham diff,rollback,mount,snapshot,send,hold trayimurti

sudo zpool export trayimurti
sudo zpool import
sudo zpool import -d /dev/disk/by-id <pool-id>

sudo zpool set cachefile=/etc/zfs/zpool.cache trayimurti
sudo chown pratham:pratham -vR /trayimurti

sudo zpool scrub trayimurti
```


### Reboot

```bash
sudo reboot +0
```

---


## Stage 0101: Containers

### Open ports

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp --add-port=8443/tcp --add-port=8010/tcp --add-port=8011/tcp --add-port=8020/tcp --add-port=8030/tcp --add-port=8040/tcp --add-port=8050/tcp --add-port=8060/tcp --add-port=8070/tcp --add-port=8071/tcp --add-port=8072/udp
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```


### Pull images

```bash
podman pull docker.io/gitea/gitea:latest
podman pull docker.io/gotify/server-arm64:latest
podman pull docker.io/klakegg/hugo:alpine
podman pull docker.io/klakegg/hugo:ext-debian
podman pull docker.io/library/caddy:alpine
podman pull docker.io/library/nextcloud:production
podman pull docker.io/library/postgres:14-alpine
podman pull docker.io/louislam/uptime-kuma:debian
podman pull lscr.io/linuxserver/transmission:latest
```


### Create directories for mounting container volumes

```bash
mkdir -vp /trayimurti/containers/volumes/caddy/{site,ssl/{private,certs},caddy_{data,config}}
mkdir -vp /trayimurti/containers/volumes/gitea/{database,web}
mkdir -vp /trayimurti/containers/volumes/nextcloud/{database,web}
```


### Enable workaround for "root-less containers can't ping hosts"

```bash
grep net.ipv4.ping_group_range /etc/sysctl.conf || echo "net.ipv4.ping_group_range=0 $(grep pratham /etc/subuid | awk -F ":" '{print $2 + $3}')" | sudo tee -a /etc/sysctl.conf
```


### Hugo

```bash
git clone --recursive git@gitlab.com:thefossguy/blog.git /trayimurti/containers/volumes/blog
cd /trayimurti/containers/volumes/blog
git remote rm origin
git remote add origin git@git.thefossguy.com:thefossguy/blog.git

git clone --recursive git@gitlab.com:thefossguy/machines.git /trayimurti/containers/volumes/mach
cd /trayimurti/containers/volumes/mach
git remote rm origin
git remote add origin git@git.thefossguy.com:thefossguy/machines.git
```


### Caddy

1. Visit the [Cloudflare dashboard](https://dash.cloudflare.com/)
2. Select domain
3. On the left sidebar, select 'SSL/TLS'. Make sure _Encryption Mode_ is **Full (strict)**.
4. Under 'SSL/TLS', goto '**Origin Server**'.
5. Create a new Certificate **with default values**.
6. Populate `/trayimurti/containers/volumes/caddy/ssl/{certs/certificate.pem,private/key.pem}`.
7. Change permissions for `/trayimurti/containers/volumes/caddy/ssl/private`.

```bash
chmod 700 -v /trayimurti/containers/volumes/caddy/ssl/private
chmod 600 -v /trayimurti/containers/volumes/caddy/ssl/private/key.pem
```

Copy `Caddyfile` to the appropriate directory.

```bash
cp -v Caddyfile /trayimurti/containers/volumes/caddy/
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
podman generate systemd -f --name hugo-vaikunthnatham --new
podman generate systemd -f --name hugo-mahayogi --new
podman generate systemd -f --name nextcloud-chitragupta --new
podman generate systemd -f --name nextcloud-govinda --new
podman generate systemd -f --name gotify-akashvani --new
podman generate systemd -f --name uptime-vishnu --new
podman generate systemd -f --name transmission-raadhe --new

systemctl --user daemon-reload

systemctl --user enable container-caddy-vishwambhar.service container-gitea-chitragupta.service container-gitea-govinda.service container-hugo-vaikunthnatham.service container-hugo-mahayogi.service container-nextcloud-chitragupta.service container-nextcloud-govinda.service container-gotify-akashvani.service container-uptime-vishnu.service container-transmission-raadhe.service
```

---


## Stage 0111: cron

### user crontab

```bash
# run maintainence scripts
* * * * * /home/pratham/.scripts/reddish/cron/pratham/check-caddy.sh > /dev/null
* * * * * /home/pratham/.scripts/reddish/cron/pratham/zfs-pool-health.sh > /dev/null
0 * * * * /home/pratham/.scripts/reddish/cron/pratham/container-updates.sh > /dev/null

# create zfs snapshots every Friday
0 0 * * 5 /sbin/zfs snapshot trayimurti/containers/volumes/uptimekuma@"$(date +%Y_%m_%d__%H_%M_%S)" > /dev/null
0 0 * * 5 /sbin/zfs snapshot trayimurti/containers/volumes/gitea@"$(date +%Y_%m_%d__%H_%M_%S)" > /dev/null
0 0 * * 5 /sbin/zfs snapshot trayimurti/containers/volumes/nextcloud@"$(date +%Y_%m_%d__%H_%M_%S)" > /dev/null

# keep journal size up-to 200M
0 0 * * * /bin/journalctl -vacuum-size=200M > /dev/null

# update neovim plugins
0 0 * * * /home/pratham/.scripts/reddish/cron/pratham/neovim-update.sh > /dev/null

# run Nextcloud cron
*/5 * * * * podman exec -u www-data nextcloud-govinda /usr/local/bin/php -f /var/www/html/cron.php > /dev/null

# Nextcloud: scan files for all users and perform cleanup
10 */2 * * * podman exec -u www-data nextcloud-govinda /usr/local/bin/php -f /var/www/html/occ files:scan --all > /dev/null
40 */2 * * * podman exec -u www-data nextcloud-govinda /usr/local/bin/php -f /var/www/html/occ files:cleanup > /dev/null
```


### root crontab

```bash
# update fs database every 6 hours
* */6 * * * updatedb > /dev/null

# check for updates every hour
0 * * * * /home/pratham/.scripts/reddish/cron/root/dnf-upgrades.sh > /dev/null

# start scrub
# on the first Friday of every month
# at 2100 hours
0 21 * * 5 [ $(date +\%d) -le 07 ] && /sbin/zpool scrub trayimurti && /home/pratham/.scripts/reddish/cron/pratham/zfs-scrub.sh > /dev/null
```
