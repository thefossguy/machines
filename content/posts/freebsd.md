---

title: "Setup freebsd (ZFS-NFS)"
date: 2022-07-23T08:00:50+05:30
draft: false
toc: true

---

## First time setup

### Change timezone

```bash
# tzsetup
```

### Upgrade pkgs

Use the `-f` option so that you don't get a checksum error for `gcc-11`

```bash
# pkg upgrade -f pkg
```

### Reboot

```bash
# shutdown -r now
```

### Install pkgs

`/usr/loca/share/zsh-autosuggestions/zsh-autosuggestions.zsh`

```bash
# pkg install -y aria2 bash bash-completion bat btop curl fd-find git htop neovim ripgrep rsync smartmontools tmux tree wget wget2 yt-dlp zsh zsh-autosuggestions zsh-completions zsh-syntax-highlighting
```

### Neovim

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

### Enable services

```
# RC_CONFIG_CONTENTS=( 
    'hostname="sentinel"' \
    'local_unbound_enable="YES"' \
    'sshd_enable="YES"' \
    'ntpdate_enable="YES"' \
    'ntpd_enable="YES"' \
    'powerd_enable="YES"' \
    'zfs_enable="YES"' \
    'smartd_enable="YES"' \
)

# for RC_SERVICE in ${RC_CONFIG_CONTENTS[@]}; do
    grep "${RC_SERVICE}" /etc/rc.conf || echo "${RC_SERVICE}" | tee -a /etc/rc.conf && echo "enabled ${RC_SERVICE}"
done
```

## ZFS

### Create zpool

```bash
zpool create -o ashift=12 trayimurti mirror DISK0 DISK1

zfs set atime=off trayimurti
zfs set primarycache=all trayimurti
zfs set recordsize=1M trayimurti
zfs set compression=zstd trayimurti
zfs set xattr=sa trayimurti

zfs create trayimurti/containers
zfs create trayimurti/containers/volumes
zfs create trayimurti/containers/volumes/caddy

zfs create trayimurti/containers/volumes/blog
zfs create trayimurti/containers/volumes/mach

zfs create trayimurti/containers/volumes/gitea
zfs create trayimurti/containers/volumes/gitea/gitea-svr
zfs create trayimurti/containers/volumes/gitea/gitea-db
zfs set recordsize=8K trayimurti/containers/volumes/gitea/gitea-db

zfs create trayimurti/containers/volumes/nextcloud
zfs create trayimurti/containers/volumes/nextcloud/nextcloud-svr
zfs create trayimurti/containers/volumes/nextcloud/nextcloud-db
zfs set recordsize=8K trayimurti/containers/volumes/nextcloud/nextcloud-db

zfs create trayimurti/torrents
zfs set recordsize=16K trayimurti/torrents
zfs create trayimurti/torrents/torrent-downloads
zfs create trayimurti/torrents/torrent-config
```

### Enable sharenfs for datasets

```bash
# zfs set sharenfs=on trayimurti

## zfs set sharenfs=on trayimurti/containers/volumes/caddy
## zfs set sharenfs=on trayimurti/containers/volumes/blog
## zfs set sharenfs=on trayimurti/containers/volumes/mach
## zfs set sharenfs=on trayimurti/containers/volumes/nextcloud/nextcloud-svr
## zfs set sharenfs=on trayimurti/containers/volumes/nextcloud/nextcloud-db
## zfs set sharenfs=on trayimurti/containers/volumes/gitea/gitea-svr
## zfs set sharenfs=on trayimurti/containers/volumes/gitea/gitea-db
```

### Set share properties for datasets

```bash
# 

## zfs set share=name=caddy,rw,path=/trayimurti/containers/volumes/caddy,prot=nfs trayimurti/containers/volumes/caddy
## zfs set share=name=mach,rw,path=/trayimurti/containers/volumes/blog,prot=nfs trayimurti/containers/volumes/mach
## zfs set share=name=blog,rw,path=/trayimurti/containers/volumes/mach,prot=nfs trayimurti/containers/volumes/blog
## zfs set share=name=nextcloud-svr,rw,path=/trayimurti/containers/volumes/nextcloud/nextcloud-svr,prot=nfs trayimurti/containers/volumes/nextcloud/nextcloud-svr
## zfs set share=name=nextcloud-db,rw,path=/trayimurti/containers/volumes/nextcloud/nextcloud-db,prot=nfs trayimurti/containers/volumes/nextcloud/nextcloud-db
## zfs set share=name=gitea-svr,rw,path=/trayimurti/containers/volumes/gitea/gitea-svr,prot=nfs trayimurti/containers/volumes/gitea/gitea-svr
## zfs set share=name=gitea-db,rw,path=/trayimurti/containers/volumes/gitea/gitea-db,prot=nfs trayimurti/containers/volumes/gitea/gitea-db
```

### /etc/exports

```
/trayimurti -alldirs -maproot=root 192.168.122.78
```

### mkdir on network host

```bash
mkdir caddy blog mach gitea-{srv,db} nextcloud-{srv,db} torrents-{config,complete-downloads,incomplete-downloads}
```
