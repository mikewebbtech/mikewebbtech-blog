---
title: Arch Linux UEFI install on NVMe drives
date: 2017-02-10T15:45:36
summary: A guide  created from memory and some notes on how I installed Arch Linux (BTW) using UEFI settings, NVMe storage. and LVM filesystem.  Grub now supports NVMe but not "exotic" filesystems like LVM or LUKS encryption as the root (/) filesystem
draft: false
categories:
  - home-lab
tags:
  - linux
  - Arch
  - solution
series:
---
# Arch Linux UEFI, NVMe and LVM install Guide

This guid has been created from memory and some notes I used when installing, I will validate as I will use this as a guide when I get around to doing another Arch Linux install. 
Out of all the modern linux distributions I've been trying out, Arch Linux would have to be one of the more complex ones I've come across.  It has a reputation for being "hard because you have to do everything yourself", which I think is a bit undeserved and and "not for newbies", which sounds like elitist drivel and counter intuitive really.  I won't argue the point on it being complex, but that doesn't make it hard.  The wiki and forums have had any stopping point I came across well and truely covered.  It is the perfect modern Linux distribution for someone looking to learn the "how to's" of linux.

Mind you.  The likes of Linux Mint or Ubuntu are pretty flash and all bells and singing and will have you up and running in 10 minutes.  But it is still good to know what's going on under the hood for the odd time when you have go there......like you need to do with EVERY operating system I have ever used.

The experience of installing and using Arch was very reminiscent of my pre-millennium days of linux  except there is now a missive online community who's shoulders you can stand upon.

For a primer you should read the official Arch Linux install wiki page. <https://wiki.archlinux.org/index.php/Installation_guide> and definitely should read <https://wiki.archlinux.org/index.php/User:Soloturn/Quick_Installation_guide_UEFI>.

But we are going to use GRUB as the boot loader as it "the common loader" amongst linux and it now supports NVMe drives.  However it still doesn't support "exotic" (e.g. LVM) file systems for the root partition (/).  We will use LVM on the root partition so we know how to work around the limitation as well as being able to capitalise on the advantages of using LVM or even more exotic root partition setups like LUKS to which LVM can be layered upon ([Logical Volume Manager](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) , [Linux Unified Key Setup](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system)).  Instead of creating the traditional partition for swap, we will make a swap volume ala LVM style.

### Settle in.  This is going to take a while.

Download the latest .iso of Arch Linux from your nearest mirror [Arch Linux mirror list](https://www.archlinux.org/download/)

For me the fastest on would be <http://mirror.internode.on.net/pub/archlinux/iso/latest/archlinux-2017.04.01-x86_64.iso>

Create a bootable USB stick using the .iso file using the instructions here Bootable USB install device in Linux

Boot/Reboot with the new Arch Linux install USB stick in the target computer using the usual method for selecting the boot device (for me that f12 when the BIOS splash screen comes up.  After some scrolling text you should be presented with a command line prompt of raw 64bit power chomping at the bit to be released.

---

Ensure that you actually have boot up using UEFI mode and not legacy mode. type

```
# ls /sys/var/firmware/efi/efivars
```
if your screen fills up with number of long named files, it is populated and you are in UEFI mode.  If not, you will have make the appropriate changes in you BIOS (consult google or motherboard manual)

Your NIC (wireless is not in the scope of this tutorial) should have been detected on boot up and your home router should have dished out an address to it via DHCP.  Let's check,  type in

```
# ping -c 5 www.google.com
```
If you have a response, great, you have a connection to the internet and are good to continue. If not, manually start the dhcp client or setup networking with what is relevant to your network e.g.

Start dhcp client

```
# systemctl start dhcpcd@eth0
```
Setup ipv4 networking

```
# ip address add 192.168.1.159/24 dev eth0 
# ip route add default via 192.168.1.1 
# echo "nameserver 192.168.1.1" >> /etc/resolv.conf 
# ping www.google.com
```

### Look for the NVMe drive

```
# lsblk
sda      8:0    0  477G  0 disk 
├─sda1   8:1    0    1G  0 part [rescue]
├─sda2   8:2    0   12G  0 part [swap]
└─sda3   8:3    0  430G  0 part [snapshot]
sdd      8:32   1  7.5G  0 disk 
└─sdd1   8:33   1  7.5G  0 part /run/media/arch/SANDISK\_16GB
nvme0n1      8:32   1  500G  0 disk 
└─nvme0n1p1   8:33   1  400G  0 part
```
So on my system there is an existing SATA drive (sda), my boot USB stick (add) and the NVMe (nvme0n1) drive with a single partition of "god knows what but I don't care if it goes".  Yes this process will permanently delete all data on that drive (or any drive that is attached to the computer if you accidentally type the wrong command...fear).

### Create Partitions

Use what ever disk partitioning system you are familiar with. by default you have disk, parted, gdisk, cgdisk and cfdisk available to you.  Pick your poison so we can create 3 partitions on the nvme0n1 drive.  I use cfdisk so I would type:

```
# cfdisk /dev/nvme0n1
```
Really really important that you define the your nvme drive as cfdisk would automatically select my sda drive and that would be horrible.

So I delete all partitions from the disk, create my 3 partitions:

```
nvme0n1p1  size 512MB type: EFI (/mnt/boot/efi)
nvme0n1p2 size  1024MB type: LINUX (/mnt/boot)
nvme0np3 size 100% type: LVM (root/)
```

### Format the partitions and setup LVM general filesystem

```
# mkfs.fat -F32 /dev/nvme0n1p1
```

```
# mkfs.ext3 /dev/nvmen0n1p2
```

```
# pvcreate /dev/nvme0n1p3
# vgcreate vg\_root /dev/nvme0n1p3
# lvcreate -L 8G vg\_root -n lv\_swap
# lvcreate -l 100%FREE vg\_root -n lv\_root
```
The same partiction setup could all be done using parted which is another viable and quick method

```
# parted /dev/nvme0n1 
(parted) mklabel gpt
(parted) mkpart ESP fat32 1MiB 513MiB
(parted) set 1 boot on
(parted) name 1 efi-partition
(parted) mkpart primary 513MiB 1536MiB
(parted) name 2 boot-partition
(parted) mkpart primary 1536MiB 100%
(parted) name 3 lvm-partition
(parted) print
(parted) quit</pre

```
---

 Create the filesystems on the logical volumes

```
# mkswap -L swap /dev/mapper/vg\_root-lv\_swap
# swapon /dev/mapper/vg\_root-lv\_swap
# mkfs.ext4 /dev/mapper/vg\_root-lv\_root
```
---

mount the partitions and logical volumes, creating a new filesystem structure ready for bootstrapping the OS install into it.

```
# mount /dev/mapper/vg\_root-lv\_root /mnt
# mkdir /mnt/boot
# mount /dev/nvme0n1p2 /mnt/boot
# mkdir /mnt/boot/efi
# mount /dev/nvme0n1p1 /mnt/boot/efi
```
Edit your /etc/pacman.d/mirrors file and uncomment a server that is faster to you (hint: search for the name of the site that your downloaded the .iso file from, chances are that they have for a full repository as well)

### Install the system files into your new filesystem tree

We are now set to start install the base Arch Linux system along with a few other tools needed to get a UEFI working so your BIOS has something to hand over the booting your system  too.

**#pacstrap /mnt base base-devel grub-eft-x86\_64 efibootmgr git**

There, you now have the bare bones of Arch Linux installed onto your system.  Now we can actually start the process of setting up your system and getting able to boot up by itself.

Preserve the new filesystem tree and make optimisations for NVMe devices by creating the fstab and modifying it.

```
# genfstab -pU /mnt >> /mnt/etc/fstab
```
open the new /mnt/etc/fstab file editing (nano is my goto for this).

```
# nano /mnt/etc/fstab
```
and make the change relatime to noatime on all non boot partitions (in this case /) and add a /tmp ramdisk to speed up and reduse wear on the NVMe drive.

```
tmpfs    /tmp     tmpfs   defaults,noatime,mode=1777       0 0
```
make /mnt the new root file system and pivot to it.

```
root@archiso /etc # arch-root /mnt /bin/bash
[root@archiso /]#
```
From now on, all the changes we make to files will be on the Arch Linux system we will be booting into.

We need to open up for editing /etc/mkinitcpio.conf to amend the HOOKS= and MODULES= section ensure the boot strapper (GRUB2) has a suitable initialisation ram disk image.

```
[root@archiso /]# nano /etc/mkinitcpio.conf
```
Add ext4 to MODULES
Add lvm2 to HOOKS before filesystems and after udev
Add resume after lvm2 (also has to be after 'udev')

Generate the the new initrd image.

```
[root@archiso /]# mkinitcpio -p linux
```
Now install the uefi ready GRUB loader

```
grub-install --target=x86\_64-efi --efi-directory=/boot/efi \
--bootloader-id=arch\_grub --recheck --debug
```
Technically your system is ready to reboot, but wait there is more to be sure so carry on.

Setup system timezone and clock. Look in /usr/share/zoneinfo to find your timezone and link the file into /etc/localtime then set the hardwareclock. There may already be a soft link /etc/localtime to /etc/share/zoneinfo/UTC, just delete it.

```
[root@archiso /]# rm /etc/localtime
[root@archiso /]# ln -s /usr/share/zoneinfo/Australia/Perth /etc/localtime
[root@archiso /]# hwclock --systohc --utc

```
Set the system hostname

```
[root@archiso /]# hostname ArchLinux
[root@archiso /]# echo ArchLinux > /etc/hostname
```
Generate a local. This defines which language the system uses, and other regional considerations such as currency denomination, numerology, and character sets etc. Values are listed in /etc/locale.gen. Edit local.gen and uncomment en\_AU.UTF-8 (as I'm in Australia), as well as other needed localisations. Save the file, and generate the new locales

```
[root@archiso /]# locale-gen
[root@archiso /]# localectl set-locale LANG=en\_AU.UTF-8
[root@archiso /]# echo LANG=en\_AU.UTF-8 >> /etc/locale.conf
[root@archiso /]# echo LANGUAGE=en\_AU >> /etc/locale.conf
[root@archiso /]# echo LC\_ALL= >> /etc/locale.conf

```
> NOTE: 
> some instructions will include "LC\_ALL=C >> /etc/locale.conf". Do not set this to avoid issues with gnome-teminal if you are going to use this as your graphical terminal application, use LC\_ALL= as shown in the above example

Set password for root

```
[root@archiso /]# passwd
```
Create another user account for yourself and give it a password

```
[root@archiso /]# useradd -m -g users -G wheel -s /bin/bash 
[root@archiso /]# passwd
```
Modify the /etc/sudoers file so you can use your new user account to do administrative tasks using sudo and never have to login or use the root account again. First set the default editor for the root account as the visudo command is set to use vim, which is not installed. I will set nano.

```
[root@archiso /]# echo "export EDITOR=nano" >> ~/.bashrc
[root@archiso /]# source ~/.bashrc
[root@archiso /]# visudo

```
Uncomment the line
 ***# %wheel ALL=(ALL) ALL***
to read
***%wheel ALL=(ALL) ALL***


### Lets Wrap it Up

There is a shed load of work to do still before this system could be considered anywhere near a usable server or workstation (still need video card drivers, services, Xorg or Wayland, a Desktop Environment etc etc) but we, at least have a minimal system that can boot up so we can begin the task of creating the system we desire.

Hold up, remember that we are still in a chroot environment, let's deal with that then reboot.

```
[root@archiso /]# exit
root@archiso# umount -R /mnt
root@archiso# reboot

```
!PHEW
Good Luck!!

