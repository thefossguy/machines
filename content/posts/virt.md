---

title: "Setup virtual machines"
date: 2022-08-23T18:28:15+05:30
draft: false
toc: true

---

## Step 0000: Initial setup


### Install updates

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

### Reboot

```
sudo reboot +0
```


## Step 0001: Install packages


### Install the necessary packages

```bash
sudo apt-get install -y aria2 autoconf barrier bat bc bison bridge-utils btop build-essential cifs-utils cmake cmatrix curl ethtool exfat-fuse fakeroot fastboot fdisk ffmpeg flex fonts-firacode fonts-fork-awesome gdb-multiarch git handbrake hdparm htop imagemagick iotop iperf iperf3 libc6-dev libelf-dev libncurses-dev libncurses5-dev libpam-google-authenticator libssl-dev linux-headers-$(uname -r) linux-tools-$(uname -r) linux-tools-common linux-tools-generic locate lsb-release make mlocate mpv ncurses-dev neofetch neovim nethogs nload nodejs openocd openssh-client openssh-server ripgrep rsync tar tmux tree unrar unzip valgrind vim webp wget wget2 xz-utils zip zsh zsh-autosuggestions zsh-common zsh-syntax-highlighting
```

### Use zsh

```
chsh -s $(which zsh) $(whoami)
```
