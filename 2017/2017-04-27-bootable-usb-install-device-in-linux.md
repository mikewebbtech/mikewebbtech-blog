---
title: Bootable USB install device in Linux
date: 2017-04-27T10:31:03
summary: These instructions will create a bootable install medium for the installation of a operating system using a .iso file downloaded from the network, the dd command and a standard USB...
draft: false
categories:
  - home-lab
tags:
  - linux
  - opensource
  - mac
---

These instructions will create a bootable install medium for the installation of a operating system using a .iso file downloaded from the network, the dd command and a standard USB stick.  You need root level access (to run the sudo command). This covers Linux [for MacOS see [Bootable USB install device in MacOS](https://mikewebblive.wordpress.com/2017/04/27/bootable-usb-install-device-in-macos/)]
### Let us begin


Copy the .iso file to your home directory (*~/*)on the local workstation for speed and convenience if possible, also to rule out issues created due to network errors during the write.

Open a terminal if not using a tty console. Run the *lsblk* command, Plug in the USB stick Now find the USB sticks device ID by using *lsblk* command again.  The new device that shows is the one you want to use (and the size will match the size written on the stick.  There may be partitions showing (e.g. /dev/sdd1 /dev/sdd2 etc) use the top level device ID

[include output example]

*in my case the USB stick is /dev/sdd so I will use that in example commands, replace with your own lsblk output.

At this point, if there is ANY data on that device you which to keep, back it up now.  The next stage will completely erase all existing partitions and data.  This is also why it is important to know the device ID of the USB stick, using the wrong ID **WILL**destroy your current operating system.

Also ensure that the operating system hasn't auto-mounted the USB device. 'dd' will fail if the device is mounted into the file system tree.  Type the *mount*command, if you see your the device ID of your USB stick listed, you have to unmount it.

```
$ mount
```

[include output example]

```
$ sudo unmount /dev/sdd1
```

*"The **dd command** stands for “**data duplicator**” and used for copying and converting data. It is very powerful low level utility which can do much like, backup and restore an entire drive or partition."*

Use dd to 'bit level' copy the the .iso image to the USB device

```
$ sudo dd if=~/ubuntu-17.04-desktop-amd64.iso of=/dev/sdd1 bs=1M
```

(replace ubuntu-17.04-desktop-amd64.iso with the name of your own .iso file).  There will be no output for a while depending on the size of the .iso file and USB speed being used.  When *dd* has successfully copied over the .iso file your should see something resembling the following output

```
3441325+0 records in
3441325+0 records out
3441325000 bytes (3.4 GB, 3.2 GiB) copied, 1.00036 s, 3.4 GB/s
```

Ensure the the USB stick has not been auto mounted and unmount it (eject) it before physically removing the USB stick from the workstation. 

### DONE.


See also:

[dd man page](https://www.gnu.org/software/coreutils/manual/html_node/dd-invocation.html)
[dd on wikipedia](https://en.wikipedia.org/wiki/Dd_(Unix))

