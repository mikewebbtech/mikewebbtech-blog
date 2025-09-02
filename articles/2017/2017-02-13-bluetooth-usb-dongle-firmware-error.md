---
title: Bluetooth USB dongle firmware error in Ubuntu
date: 2017-02-13T10:44:09
summary: After having a running battle with my, what turns out, cheap Chinese CSR clone bluetooth USB adaptor (maybe more on that in another post). I purchased a more expensive adaptor...
draft: false
categories:
  - home-lab
tags:
  - linux
  - solution
  - hardware
---

After having a running battle with my, what turns out, cheap Chinese CSR clone bluetooth USB adaptor (maybe more on that in another post).  I purchased a more expensive adaptor to trouble shoot some issues.  The time round I picked up a Targus brand from the local office supply store that based on the  Broadcom BCM20702a1 chip.

When booting up with the adaptor installed I noticed HCI errors in my dmesg


```
Bluetooth: hci0: BCM20702A1 (001.002.014) build 0000
bluetooth hci0: firmware: failed to load brcm/BCM20702A1-0a5c-21e6.hcd (-2)
bluetooth hci0: Direct firmware load for brcm/BCM20702A1-0a5c-21e6.hcd failed with error -2
Bluetooth: hci0: BCM: Patch brcm/BCM20702A1-0a5c-21e6.hcd not found
```

It would appear that the distribution I am running doesn't have the firmware drivers for this adaptor and that this is not that unusual and not that big of an issue.  Installing firmware drivers is not that big a deal and the internet is usually your friend.

My adaptor came with windows drivers but I chose to download updated drivers from the internet that came in a nice packaged windows installer.  Turns out the pakaging was new but the needed hex file contained inside was the same version (windows driver installers tend to be huge full of different drivers for different chipsets).  After a quick search I determined that the BCM20702A1\_001.002.014.1483.1669.hex file was the one I needed for my adaptor.

A nifty utility called hex2hcd will convert the hex file into the what linux can load.


```
$> hex2hcd BCM20702A1\_001.002.014.1483.1669.hex
```

Then copy to the firmware directory with the correct name:


```
$> sudo cp ./BCM20702A1\_001.002.014.1483.1669.hcd /lib/firmware/brcm/BCM20702A1-0a5c-21e6.hcd
```

Reboot and all should be at one with the universe. ZEN!