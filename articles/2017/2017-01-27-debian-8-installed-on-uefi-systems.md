---
title: Debian 8 installed on UEFI systems
date: 2017-01-27T20:27:41
summary: Absolute pain getting Debian installed on my UEFI only sytem. When working through the installation process until the boot loader tries to install and this message occurs.
draft: false
categories:
  - home-lab
tags:
  - linux
  - solution
  - debian
  - server
series:
---

Absolute pain getting Debian installed on my UEFI only sytem.  When working through the installation process until the boot loader tries to install and this message occurs.

```
An installation step failed. You can try running the failing item again from the menu, or skip it and choose something else. The failing step is: Install the GRUB boot loader on a hard disk

In the BIOS, set "UEFI only" boot
```

I had to many issues to contend with, like treated platform etc, but the way linux boots up on UEFI systems was the one thing I could fix. I tried a few work arounds I could find on the internet but found that worked.

Using Gparted, create a FAT32 partition at the beginning of the disk with the boot and esp flags. (The Debian installer should be able to do this too, but since the installer incorrectly recognized the size of the disk, I prefer to use Gparted). In my case, the FAT32 partition is /dev/nvme0n1p1.
During the installation, make sure you have a network connection configured (manually or automatically, doesn't matter). Otherwise, the next step will fail.
At the installation stage where GRUB fails to install, open a shell and run the following commands:


```
mount --bind /dev /target/dev
mount --bind /dev/pts /target/dev/pts
mount --bind /proc /target/proc
mount --bind /sys /target/sys
cp /etc/resolv.conf /target/etc
chroot /target /bin/bash
```


```
apt-get update
apt-get install grub-efi-amd64
update-grub
grub-install --target=x86\_64-efi /dev/nvme0n1
```


Exit the shell and select "Continue without installing a bootloader." You'll see a warning message that gives you boot commands to use; you can ignore this.
Once the installation completes, boot into the system. Add "nvme" to /etc/initramfs-tools/modules, then run update-initramfs -u as root.

Edit /etc/default/grub and add this line

```
GRUB\_CMDLINE\_LINUX="intel\_pstate=no\_hwp"
```
and add "nomodeset" to the GRUB\_CMDLINE\_LINUX\_DEFAULT so it looks like this:
```
GRUB\_CMDLINE\_LINUX\_DEFAULT="quiet nomodeset"
```
Run update-grub.

The last few commands (initramfs onward) are necessary to prevent disk not found errors the second time you try to boot into the new system.