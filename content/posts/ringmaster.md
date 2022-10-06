---

title: "Setup ringmaster (macOS)"
date: 2022-07-23T08:00:10+05:30
draft: false
toc: true

---

## Dev setup

### Install Xcode command line tools

```bash
xcode-select --install
```

### Install Homebrew

Make sure that the install command is the same [from here](https://brew.sh/).

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew update
brew upgrade
```

### Install packages

```bash
brew install aria2 bat btop coreutils curl ffmpeg gcc git git-delta gnu-sed grep handbrake htop hugo imagemagick iperf iperf3 mpv neovim qemu readline rename ripgrep speedtest-cli tmux tree wakeonlan watch webp wget xz yt-dlp zsh-fast-syntax-highlighting

brew install cask alacritty android-platform-tools balenaetcher bitwarden brave-browser discord firefox homebrew/cask-fonts/font-fira-code homebrew/cask-fonts/font-fira-mono font-overpass-mono google-chrome keepassx keka librewolf macs-fan-control meld moderndeck obs protonvpn sublime-text telegram tor-browser utm virtualbox virtualbox-extension-pack whatsapp webtorrent

# brew install cask zoom
```

### Maintenance

```bash
brew cleanup
brew doctor
```

### Install Rust (rustup)

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

rustup default stable
rustup component add rust-src rust-analyzer
#rustup component add rust-analysis

cargo install cargo-outdated cargo-tree
```

### Generate SSH keys

```bash
cd $HOME/.ssh
ssh-keygen -t ed25519 -f bluefeds
ssh-keygen -t ed25519 -f flameboi
ssh-keygen -t ed25519 -f gitea
ssh-keygen -t ed25519 -f github
ssh-keygen -t ed25519 -f gitlab
ssh-keygen -t ed25519 -f sentinel
```

### Alacritty setup

```bash
git clone https://github.com/alacritty/alacritty.git && cd alacritty
sudo tic -xe alacritty,alacritty-direct extra/alacritty.info && cd .. && rm -rf alacritty
```


## Extra setup

### Improve Dock behaviour

```bash
defaults write com.apple.dock autohide-delay -float 0
defaults write com.apple.dock autohide-time-modifier -int 0
killall Dock
```

### Improve Finder

```bash
defaults write com.apple.Finder AppleShowAllFiles true
killall Finder
```
