---
title: Bootable USB install device in MacOS
date: 2017-04-27T10:48:52
summary: These instructions will create a bootable install medium for the installation of a operating system using a .iso file downloaded from the network, the dd command and a standard USB...
draft: false
categories:
  - home-lab
tags:
  - mac
  - opensource
  - linux
---

These instructions will create a bootable install medium for the installation of a operating system using a .iso file downloaded from the network, the dd command and a standard USB stick.  You need admin level access (to run the sudo command). This covers MacOS [for Linux see [Bootable USB install device in Linux](https://mikewebblive.wordpress.com/2017/04/27/bootable-usb-install-device-in-linux/)]
### Let us begin


Copy the .iso file to your home directory (*~/*)on the local workstation for speed and convenience if possible, also to rule out issues created due to network errors during the write.

Open a terminal and run the *diskutil lis*t command, Plug in the USB stick Now find the USB sticks device ID by using the *diskutil list* command again.  The new device that shows is the one you want to use (and the size will match the size written on the stick.  There may be partitions showing use the top level device ID

[include output example]

Now unmount the drive:

`diskutil umountDisk /dev/disk4`

Then use the *dd* command to write the .iso file to the USB stick

`sudo dd if=~/ubuntu-17.04-desktop-amd64.iso`of=/dev/disk4 bs=1m``