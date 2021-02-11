---
title: "I3 dev setup on Ubuntu 18.04"
date: 2019-02-01T00:00:00+02:00
tags: [Ubuntu, OS]
---

Just a note to self, nothing interesting

## 1. Install LVM on LUKS

Follow instructions [here]({{< relref "ubuntu-1804-lvm-on-luks.md" >}})

## 2. Setup dropbox

No support for filesystems other than ext4. <mark>No support for eCryptfs on ext4.</mark>

## 3. Install and mount gocryptfs volume

Keep versions in sync, just in case

https://github.com/rfjakob/gocryptfs/releases

## 4. Generate ssh keys

Store passphrase in synced password store

```console
ssh-keygen
```

Use the cryptfs volume to transfer the pub keys, then add them to github, bitbucket.

## 5. Install fzf

https://github.com/junegunn/fzf-bin/releases

Download and untar the <mark>bin release</mark>, mv into ~/.local/bin

## 6. Install dotfiles

```console
mkdir -p ~/code/github.com/benjamin-thomas && cd !$
git clone git@github.com:benjamin-thomas/dotfiles.git && cd dotfiles
./create_symlinks.rb

apt-get install rofi dunst thunderbird rxvt-unicode

# for utt
apt-get install python-pip

apt-get install xsel # clipboard gem dependency
gem install --user clipboard

apt-get install scrot shutter # screenshots
apt-get install feh
apt-get install thunderbird-locale-fr # then download dictionary manually (via right clic > add dict > modules)
```

## 7. Install dev scripts

```console
git clone git@bitbucket.org:inklusive/manage.git
```

Follow the README.md and README_DEV instructions

## 8. Disable GUI login, gdm

```console
systemctl set-default multi-user.target
```

multi-user.target has run level 2, 3, 4
graphical.target has run level 5

So GUI logins could be re-enabled with

```console
systemctl set-default graphical.target
# or once
systemctl start gdm3.service
```

## 9. Unlock gnome-keyring upon login (see ~/.xinitrc)

```file
# /etc/pam.d/login
# Add those 2 lines below  pam_group.so
auth      optional     pam_gnome_keyring.so
session   optional     pam_gnome_keyring.so auto_start
```

## 10. Thunderbird setup

- Add sigs via Dropbox
- disable TB spam detection (too many false positives, let the server handle it)
- select marked spam goes to the spam folder
- change default file explorer (which one??)

## 11. Utt log in Dropbox

```bash
ln -s ~/Dropbox/PrivSync/Documents/Logs/utt.log -T ~/.local/share/utt/utt.log
```

## 12. Disable default PDF for surfing keys extension


Type

```console
;s
```

# Disable screen saver

```console
xset -dpms s off
```

lxappearance to change default font size (first tab "widget/interface")
gnome-tweaks > fonts > change 4th/last font param for gedit, etc. (others don't seem to do anything)

apt install thunar
apt remove nautilus
