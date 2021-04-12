---
title: "HOWTO: Ubuntu 18.04 install with LVM on LUKS"
date: 2019-01-31T00:00:00+02:00
description: "A Step by step installation"
tags: [Ubuntu, OS]
---


---

Needs cleanup next time, [this article](https://zestedesavoir.com/tutoriels/1653/installer-un-ubuntu-chiffre-avec-luks-lvm-et-un-partitionnement-personnalise) put me on the right track!

---

## 1. Launch a live USB

### Tips

- Press `F6` to enter the advance GUI mode
- Activate `nomodeset` if the graphics card is not well supported
- Get RAM size with: `free -h`

## 2. Partion disk

```console
parted /dev/sdX
mklabel msdos                   # Reboot could be necessary here
mkpart primary 0% 2GB           # For /boot
mkpart primary 2GB 100%         # For LUKS
print                           # Review layout
```

## 3. LUKS setup

```console
cryptsetup luksFormat /dev/sdX2
cryptsetup luksOpen   /dev/sdX2 cryptpart
```

## 4. LVM setup

```console
pvcreate /dev/mapper/cryptpart                       # Create a physical volume
vgcreate SysVol /dev/mapper/cryptpart                # Create a volume group
lvcreate -L 12GB -n CryptSwap SysVol                 # Choose double the RAM
lvcreate -l 100%FREE -n CryptSys SysVol
```

## 5. Prepare the filesystems

This is optional, and can be done with the GUI install later instead.

```console
mkfs.ext4 -L BOOT /dev/sdX1
mkfs.ext4 -L SYS /dev/mapper/SysVol-CryptSys
```

## 6. Install the base OS

Launch the ubuntu installer, select custom settings ("Something else")


- Point /dev/mapper/SysVol-CryptSys to /
- Point /dev/sdX1 to /boot
- Mark /dev/mapper/SysVol-CryptSwap as swap

**Do not restart yet!**

## 7. Chroot setup and enter

```console
# Copy-paste the partition UUID
blkid /dev/sdX2

# Mounts for chroot
mount /dev/SysVol/CryptSys /mnt
mount /dev/sdX1 /mnt/boot

# Mount initramfs and grub-update dependencies
cd /
mount -t proc proc /mnt/proc
mount -t sysfs sys /mnt/sys
mount -o bind /dev /mnt/dev

# Enter chroot
chroot /mnt
```

## Configuration inside chroot

### Install vim

```console
# May need to configure DNS to download with apt-get
# /etc/resolv.conf --> nameserver 8.8.8.8
apt-get install vim
```

### Edit /etc/crypttab

TODO: I don't think the lvm option is necessary

```file
# <target name> <source device>                 <key file>    <options>
cryptpart       UUID=DEVICE_UID(no quotes!!)    none          luks,lvm
```

### Edit /etc/initramfs-tools/conf.d/cryptroot

```file
CRYPTROOT=target=cryptpart,source=/dev/disk/by-uuid/DEVICE_UID
```

### Create/update initramfs

```console
# -c for create
# -k for kernel
# -d for delete
# -u for update an existing initramfs
update-initramfs -k all -c
#update-initramfs -k all -d
#update-initramfs -c -k $(uname -r) # all did not work after delete all
#update-initramfs -u
```

### Edit /etc/default/grub

```file
# Remove "quiet splash" to display boot data, GRUB_CMDLINE_LINUX_DEFAULT options won't be applied in rescue mode

GRUB_CMDLINE_LINUX="nomodeset cryptopts=target=cryptpart,source=/dev/disk/by-uuid/DEVICE_UID,lvm=SysVol"
```

### Update grub

```console
update-grub
```

## 8. Finally

Make sure `/etc/default/keyboard` is coherent.

Exit chroot, then reboot!


---

## 9. User setup

### Sync password store accros new PC

Create a new gpg key with `seahorse` (will unlock upon signin), email=user@localhost.

Then export the public key (new PC):

    gpg --armor --export user@host-year > ~/Dropbox/user@host-year.pub.asc

Import via `seahorse` (old PC)

There seems to be a bug long standing bug (1604), if the imported key does not show up, trust manually:

- gpg --edit user@host-year
- trust
  - choose "Ultimate"

## I struggled to get the key to import (gpg vs gpg2)

If seahorse does not reflect the changes, import manually next time

gpg2 --import ~/Dropbox/user@host-year.pub.asc


Add the newly available pub key `user@host-year` to: `~/.password-store/.gpg-id`

Re-encrypt the password store with all the available public keys

    pass init $(cat ~/.password-store/.gpg-id)


Then delete the pub keys (not needed anymore)
