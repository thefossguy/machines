---

title: "Setup bluefeds"
date: 2022-07-23T08:00:30+05:30
draft: false
toc: true

---

# Setup bluefeds (Fedora Server on Raspberry Pi)

## Stage 0000: Flash SD Card


```bash
xzcat Fedora-IMAGE-NAME.raw.xz | sudo dd status=progress bs=4M of=/dev/XXX
sync && sync && sync
sudo lvchange -an /dev/fedora_fedora/root
sudo eject /dev/XXX
```

---


## Stage 0001: Immediate initial setup

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

Use fastest mirror

Exclude package `shim-aa64` (causes uboot to panic)

```bash
echo -ne "\nmax_parallel_downloads=20\nlog_compress=True\nexcludepkgs=shim-aa64" | sudo tee -a /etc/dnf/dnf.conf
#sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm

# mirrors might be out of sync, don't use this...
#echo -ne "\nfastestmirror=True" | sudo tee -a /etc/dnf/dnf.conf

sudo dnf clean all
```


### Remove Kernel cmdline args

```bash
sudo grubby --remove-args=quiet --update-kernel=ALL
sudo grubby --remove-args=rhgb --update-kernel=ALL
```


### Reboot

```bash
sudo reboot +0
```

---


## Stage 0010

### Upgrade packages

```bash
# Fedora Server
sudo dnf --refresh update

# RHEL clone (Rocky Linux 9)
sudo dnf --refresh update
sudo dnf config-manager --set-enabled crb
sudo dnf install epel-release
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
sudo dnf install aardvark-dns aria2 bat btop buildah cockpit cockpit-file-sharing cockpit-machines cockpit-packagekit cockpit-pcp cockpit-podman cockpit-session-recording console-setup cronie cronie-anacron curl fd-find git git-delta hd-idle hdparm htop iotop libvirt-daemon-kvm libwebp-tools neovim nload nodejs openssh-server overpass-mono-fonts perl-Digest-SHA podman podman-compose qemu qemu-kvm qemu-kvm-core qrencode-libs ripgrep rsync samba-common skim slirp4netns smartmontools tmux tree unrar unzip util-linux-user vim-enhanced wget yt-dlp yt-dlp-zsh-completion zsh zsh-syntax-highlighting

# sudo dnf install plocate
# sudo dnf install qemu-device-display-virtio-gpu

chsh -s $(which zsh) $(whoami)

# RHEL clone (Rocky Linux) only
sudo dnf module install container-tools
```


### vim-plug (Neovim)

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

**Open `nvim` and type `:PlugInstall`**


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

# Fedora Server
sudo dnf install autoconf automake dkms elfutils-libelf-devel gcc git kernel-doc kernel-devel-$(uname -r) kernel-rpm-macros libaio-devel libattr-devel libblkid-devel libcurl-devel libffi-devel libtirpc-devel libtool libudev-devel libuuid-devel make ncompress openssl-devel python3 python3-cffi python3-devel python3-packaging python3-setuptools rpm-build zlib-devel

# RHEL clone (Rocky Linux)
sudo dnf install gcc make autoconf automake libtool rpm-build libtirpc-devel libblkid-devel libuuid-devel libudev-devel openssl-devel zlib-devel libaio-devel libattr-devel elfutils-libelf-devel python3 python3-devel python3-setuptools python3-cffi raspberrypi2-kernel4-devel libffi-devel git ncompress libcurl-devel bind-utils tree podman cockpit-podman podman-compose
sudo dnf install --enablerepo=epel --enablerepo=powertools python3-packaging dkms

cd zfs
sh autogen.sh
./configure
make -j1 rpm-utils rpm-dkms

sudo yum localinstall *.$(uname -p).rpm *.noarch.rpm
sudo modprobe zfs
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
sudo zpool set cachefile=/etc/zfs/zpool.cache trayimurti
```


### Creating a new zpool?

```bash
sudo zpool create -o ashift=12 trayimurti /dev/sda

sudo zfs set atime=off trayimurti
sudo zfs set primarycache=all trayimurti
sudo zfs set recordsize=1M trayimurti
sudo zfs set xattr=sa trayimurti

sudo zfs create trayimurti/containers
sudo zfs create trayimurti/containers/volumes
sudo zfs create trayimurti/containers/volumes/blog
sudo zfs create trayimurti/containers/volumes/caddy
sudo zfs create trayimurti/containers/volumes/gitea
sudo zfs create trayimurti/containers/volumes/mach
sudo zfs create trayimurti/containers/volumes/nextcloud

sudo chown pratham:pratham -vR /trayimurti

sudo zpool export trayimurti

sudo zpool import
sudo zpool import -d /dev/disk/by-id <pool-id>

sudo zpool set cachefile=/etc/zfs/zpool.cache trayimurti

zpool status -v
zfs list

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
sudo firewall-cmd --permanent --add-port=8080/tcp --add-port=8443/tcp --add-port=8010/tcp --add-port=8011/tcp --add-port=8020/tcp --add-port=8030/tcp --add-port=8040/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```


### Pull images

```bash
sleep 60 && podman pull docker.io/gitea/gitea:latest
sleep 60 && podman pull docker.io/klakegg/hugo:alpine
sleep 60 && podman pull docker.io/library/caddy:alpine
sleep 60 && podman pull docker.io/library/nextcloud:production
sleep 60 && podman pull docker.io/library/postgres:alpine
```


### Get fs ready

```bash
sudo zfs create trayimurti/containers/volumes/caddy
sudo zfs create trayimurti/containers/volumes/gitea
sudo zfs create trayimurti/containers/volumes/blog
sudo zfs create trayimurti/containers/volumes/nextcloud
sudo zfs create trayimurti/containers/volumes/mach

sudo chown pratham:pratham -vR /trayimurti/containers/volumes
```


### Create directories for mounting container volumes

```bash
mkdir -vp /trayimurti/containers/volumes/caddy/{site,ssl/{private,certs},caddy_{data,config}}
mkdir -vp /trayimurti/containers/volumes/gitea/{database,web}
mkdir -vp /trayimurti/containers/volumes/nextcloud/{database,web}
```


### Hugo

```bash
git clone git@gitlab.com:shivohamx3/blog.git /trayimurti/containers/volumes/blog
cd /trayimurti/containers/volumes/blog && git submodule init && git submodule update
git clone git@gitlab.com:shivohamx3/machines.git /trayimurti/containers/volumes/mach
cd /trayimurti/containers/volumes/mach && git submodule init && git submodule update
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


### Generate container secrets for passwords

```bash
openssl rand -base64 20 | podman secret create gitea_database_user_password -
openssl rand -base64 20 | podman secret create nextcloud_database_user_password -
openssl rand -base64 20 | podman secret create nextcloud_database_root_password -
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

podman generate systemd -f --name caddy-vishwambhar
podman generate systemd -f --name gitea-chitragupta
podman generate systemd -f --name gitea-govinda
podman generate systemd -f --name hugo-mahayogi
podman generate systemd -f --name hugo-vaikunthnatham
podman generate systemd -f --name nextcloud-chitragupta
podman generate systemd -f --name nextcloud-govinda
podman generate systemd -f --name nextcloud-karma

systemctl --user enable container-caddy-vishwambhar container-gitea-chitragupta container-gitea-govinda container-hugo-mahayogi container-hugo-vaikunthnatham container-nextcloud-chitragupta container-nextcloud-govinda container-nextcloud-karma

#systemctl --user enable container-caddy-vishwambhar
#systemctl --user enable container-gitea-chitragupta
#systemctl --user enable container-gitea-govinda
#systemctl --user enable container-hugo-mahayogi
#systemctl --user enable container-hugo-vaikunthnatham
#systemctl --user enable container-nextcloud-chitragupta
#systemctl --user enable container-nextcloud-govinda
#systemctl --user enable container-nextcloud-karma
```

---


## Stage 0111: cron

### user crontab

```bash
# power down containers before a snapshot is taken on 00:00 Fridays
45 23 * * 4 systemctl --user stop container-caddy-vishwambhar container-gitea-chitragupta container-gitea-govinda container-hugo-mahayogi container-hugo-vaikunthnatham container-nextcloud-chitragupta container-nextcloud-govinda container-nextcloud-karm
a
# start containers after a snapshot is taken on 00:00 Fridays
10 00 * * 5 systemctl --user start container-caddy-vishwambhar container-gitea-chitragupta container-gitea-govinda container-hugo-mahayogi container-hugo-vaikunthnatham container-nextcloud-chitragupta container-nextcloud-govinda container-nextcloud-kar
ma


# a zfs scrub takes place on the first Friday of every month
# this is done at 21:00 hours
# so stop all containers before the scrub takes place
45 20 * * 5 [ $(date +\%d) -le 07 ] && systemctl --user stop container-caddy-vishwambhar container-gitea-chitragupta container-gitea-govinda container-hugo-mahayogi container-hugo-vaikunthnatham container-nextcloud-chitragupta container-nextcloud-govinda container-nextcloud-karma

# maintenance script
# [[ if zpool scrub is not running ]]
#   -> [[ if containers are not running ]]
#     -> start containers
* * * * * bash /home/pratham/.scripts/cron/pratham/maintenance.sh
```


### root crontab

```bash
# update fs database every 6 hours
* */6 * * * updatedb

# create zfs snapshots every Friday
0 0 * * 5 bash /home/pratham/.scripts/cron/root/zfs-bak.sh

# start scrub
# on the first Friday of every month
# at 2100 hours
0 21 * * 5 [ $(date +\%d) -le 07 ] && /sbin/zpool scrub

# maintenance script
#0 20 * * * bash /home/pratham/.scripts/cron/root/maintenance.sh
```
