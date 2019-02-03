---
title: "I3 dev setup on Ubuntu 18.04"
date: 2019-02-01T00:00:00+02:00
tags: [Ubuntu, OS]
---

Just a note to self, nothing interestring

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

Dowload and untar the <mark>bin release</mark>, mv into ~/.local/bin

## 6. Install dotfiles

```console
mkdir -p ~/code/github.com/benjamin-thomas && cd !$
git clone git@github.com:benjamin-thomas/dotfiles.git && cd dotfiles
./create_symlinks.rb

apt-get install rofi dunst thunderbird rxvt-unicode
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

## 9. Unlock gnome-keyring on login (see ~/.xinitrc)

```file
# /etc/pam.d/login
# Add those 2 lines below  pam_group.so
auth      optional     pam_gnome_keyring.so
session   optional     pam_gnome_keyring.so auto_start
```
