---
layout: default
title: Debian
description: Package management, package lists and modifications on debian
tags: linux debian package software apt-get apt-cache sources install system desktop gnome steam
---

* Table of contents
{:toc}

## Package management on Debian

To ensure unstable packages don't get installed by default, this is my apt pinning for a mixed sources Debian install.
```
# /etc/apt/preferences.d/71pinning.conf
# # man apt_preferences
# # apt-cache policy - any release keyword from output can be used
Package: base-files
Pin: release a=stable
Pin-Priority: 999

Package: *
Pin: release a=stable
Pin-Priority: 900

Package: *
Pin: release a=stable-security
Pin-Priority: 900

Package: *
Pin: release n=trixie-backports
Pin-Priority: 20

Package: *
Pin: release n=forky
Pin-Priority: 10

Package: *
Pin: release n=sid
Pin-Priority: 1

Package: *
Pin: release n=unstable
Pin-Priority: 1
```

Install from a specific source, overriding the pinning. If a pin is specifically for a package name, this will not work, see `man apt-get`.
```sh
apt-get --target-release trixie-backports install firefox
```

List tasks for debian, devuan, ubuntu and other apt-cache using systems.
```sh
apt-cache dumpavail | grep -E ^Task: | sed -e 's/Task: //' -e 's/,./\n/g' | sort -u
```
Search for a package providing file `whoami`. Narrow down the search term by providing a partial path.
```sh
apt-get --assume-yes install apt-file
apt-file update
apt-file search bin/whoami
```

## Debian packages

### Ensure installed packages

My list to install on a Debian workstation / desktop.
```sh
apt-get install \
firmware-linux-free \
firmware-linux-nonfree \
linux-headers-amd64 \
debian-kernel-handbook \
gcc-multilib \
moreutils \
vim \
tmux \
ufw \
whois \
ncdu \
python-is-python3 \
pass \
pass-otp \
paperkey \
zbar-tools \
shellcheck \
ntpsec \
apt-file \
acpi-support \
aptitude \
extrepo \
ripgrep \
fd-find \
sysfsutils \
w3m \
ldnsutils \
at \
xxd \
exim4-base \
build-essential \
debianutils \
debian-goodies \
devscripts \
gdb \
valgrind \
yq \
wireguard-tools \
bridge-utils \
trickle \
ethtool \
wakeonlan \
hdparm \
sdparm \
hwinfo \
mtools \
uefitool \
ansible \
rsync \
smartmontools \
clamav-freshclam \
rkhunter \
rng-tools-debian \
gtypist \
docker.io \
unrar \
tcpdump \
iftop \
nmap \
wireshark \
lm-sensors \
irssi \
dict \
finger \
vim-gtk3 \
vim-scripts \
vim-solarized \
feh \
imagemagick \
gimp \
inkscape \
freecad \
libreoffice \
mpv \
mpd \
mpc \
libnotify-bin \
thunderbird \
gamemode \
gnome-power-manager \
gnome-shell-extension-impatience \
gnome-shell-extension-gamemode \
gnome-shell-extension-prefs \
gnome-shell-extension-dash-to-panel \
dconf-editor \
mesa-utils \
vulkan-tools \
wlr-randr \
wshowkeys \
sox \
libsox-fmt-all \
mediainfo \
audacity \
pavucontrol \
pulseaudio-utils \
suckless-tools \
ddcutil \
sway \
foot-terminfo \
dex \
grim \
slurp \
waybar \
wev \
inotify-tools \
zathura \
vainfo \
vdpauinfo \
printer-driver-gutenprint \
simple-scan \
tesseract-ocr \
parallel \
obs-studio \
gh \
golang \
opam \
direnv \
aria2 \
calibre \
deluge \
cpu-checker \
openscad \
freecad \
blender \
pngphoon
#generate png image of the current moon phase, desktop background
```

### Miscellaneous packages

Stuff I may or may not install.

```sh
#remind, calendar with moon phases and reminders, functions, evaluations, running commands etc
#figlet # ASCII banners
#cowsay
#tidy # HTML

# dos2unix # --keep-utf16; unix2dos, convert text file formats
# gammastep # not working on mutter, good for sway
# apt-utils
# mesa-vulkan-drivers
# steam-installer
# nut
# rclone # Go file syncer
# xdg-user-dirs-gtk
# wireplumber
# wev # wayland events debugger
# wf-recorder # wayland screen recorder. Are there better ones?
# gnome-shell-extension-move-clock
# gnome-shell-extension-multi-monitors
# gnome-shell-extension-pixelsaver
```
### Modifications to system that may be needed.
- Disable ssh password auth.
```
# /etc/ssh/sshd_config.d/30-password.conf
PasswordAuthentication no
```
```sh
systemctl restart ssh
```
- Optional interactive commands.
```sh
# dpkg-reconfigure keyboard-configuration
# ufw limit ssh
# ufw enable
# Reconfigure exim for a working local mail service
# update-exim4.conf # after editing /etc/aliases and /etc/mailname
```
- Add i386 architecture (for steam).
```sh
dpkg --add-architecture i386
apt-get update
```
Steam likely needs:
```
libglx-mesa0:i386
mesa-vulkan-drivers:i386
libgl1-mesa-dri:i386
```

### General of interest packages

```
streamlink
pandoc # general markup converter, supports lots of formats including epub, html, odt, csv, json.
sparse # C static analyzer
zbar-tools # qr code scan from file or webcam
nix-setup-systemd # nix, package manager written in Haskell, reproducible builds
#sudo usermod -aG nix-users $(whoami)
mandoc # format manual pages
scdoc # generate man pages
headache # manage license headers in source files
clang # can conflict with gcc for certain builds
stellarium # planetarium
virt-install
virt-manager # graphical
libosinfo-bin # osinfo-query

python-is-python3
bundler # ruby
shellcheck
ntpsec

units # unit conversion. Type a few characters and autocomplete with tab.
scour # optimize .svg files
iw # simple wireless CLI tool
toot # mastodon cli client
tigervnc-viewer
remmina # MS RDP
bitlbee
chromium
adb # more up to date from snap
fonts-terminus
cmake # some cargo builds need it
tar-scripts # backup and restore scripts included with tar by default
clinfo # query openCL info
gamescope # run games in steam: gamescope --steam %command%

htop
crawl #(stone-soup dungeon crawler)
rss-bridge # PHP, generates feeds for sites without feeds
rtmpdump # dump streamed media
xournalpp # edit PDF files
treesheets # TreeSheets is a free and fast data organizer that can replace
# spreadsheets, mind mappers, outliners, PIMs, text editors and small databases.
zim # graphical wiki maintenance editor

# installed by default on a debian Desktop system
systemd-boot # selection at install time
dmidecode
usbutils
netcat-traditional
7zip
tree
unattended-upgrades
traceroute
util-linux # mkfs etc
bsdutils
bsdextrautils
bash-completion
sysstat # sa1 - collect data in binary, sa2 - create report from binary, sysstat - configure collection
```
