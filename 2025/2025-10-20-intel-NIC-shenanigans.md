---
date: 2025-10-20T00:00:00+08:00
draft: false
title: Solving Intel NIC Shenanigans
summary: How I fixed the issue with non-intel trasceiver not working with Intel X520 network card. Intel trying to be smart and how to tell the card to get over itself.
tags:
  - linux
  - hardware
  - howto
categories:
  - home-lab
  - networking
series:
---
# Intel x520 sfp+ fix

Not sure if this a linux issue, a Proxmox issue or Intel being funny with none intel sfp+ transceiver  support with their NICs.

I have successfully used brocade and HP sfp+ transceiver modules with onboard X710 10Gbe but ran into issues when trying to install and configure networking on proxmox with an Intel x520 PCI NIC.

The interface wouldn't get a DHCP address.  No requests were being sniffed on the wire, so I swapped out the module for (had a brocade installed) with HP then tried a cisco module, still no luck.

ethtool --test failed.  So of to look at the logs, which should have been the stop after checking to see if the fibre was clicked in.

```bash
ixgbe 0000:02:00.0: failed to load because an unsupported SFP+ or QSFP module type was detected.
ixgbe 0000:02:00.0: Reload the driver after installing a supported module.
```
I've come across such chicanery before with some switch vendors not supporting third party sfp_ and qfp modules (total BS from an end user perspective but makes sense from a support perspective).

To test this, in linux, we inject some magic into the drivers.
```bash
root@server:~# rmmod ixgbe
root@server:~# modprobe ixge allow_unsupported_sfp=1
```
Back to the logs
```bash
dmesg | grep ixgbe
ixgbe 0000:02:00.1 0000:02:00.1 (uninitialized): WARNING: Intel (R) Network Connections are quality tested using Intel (R) Ethernet Optics.  Using untested modules is not supported and may cause unstable operation or damage to the module or the adapter.  Intel Corporation is not responsible for any harm caused by using untested modules.
ixgbe 0000:02:00.1: Multiqueue Enabled: Rx Queue count = 8, Tx Queue count = 8 XDP Queue count = 0
ixgbe 0000:02:00.1: 32.000 Gb/s available PCIe bandwidth (5.0 GT/s PCIe x8 link)
ixgbe 0000:02:00.1: MAC: 2, PHY: 19, SFP+: 6, PBA No: G28774-000
ixgbe 0000:02:00.1: a0:36:9f:17:05:e2
ixgbe 0000:02:00.1: Intel(R) 10 Gigabit Network Connection
ixgbe 0000:02:00.1 enp2s0f1: renamed from eth0
ixgbe 0000:02:00.1: registered PHC device on enp2s0f1
ixgbe 0000:02:00.1 enp2s0f1: WARNING: Intel (R) Network Connections are quality tested using Intel (R) Ethernet Optics.  Using untested modules is not supported and may cause unstable operation or damage to the module or the adapter.  Intel Corporation is not responsible for any harm caused by using untested modules.
ixgbe 0000:02:00.1 enp2s0f1: detected SFP+: 6
ixgbe 0000:02:00.1 enp2s0f1: NIC Link is Up 10 Gbps, Flow Control: RX/TX
ixgbe 0000:02:00.1 enp2s0f1: NIC Link is Down
```
Nice. the drivers loaded and the NIC accepted the Brocade sfp+ module.  With some nice trolling thrown in for some passive aggressive good measure.  Onya Intel.

let' bring up the interface and see we have dhcp goodness.
```bash
root@server:~# ip link set eno2sof1 up
```
Annnd...Boom. IP adress goodness.

Now to make it permanent to survive a reboot. I was going to have the modules load at boot with the hack, but a search showed a more permanent method.  Make the change on the card then it would work regardless of the server or OS.

[Servethehome forum genius](https://forums.servethehome.com/index.php?threads/patching-intel-x520-eeprom-to-unlock-all-sfp-transceivers.24634/)

Take the time to read and understand this before messing around with EEPROMs because cut and past from internet in this case wont work.

In my case the commands to see the values, change the value and check the values were
```bash
root@server:~# ethtool -e enp2s0f1 offset 0x58 length 1
Offset          Values
------          ------
0x0058:         fc 

root@server3:~# ethtool -E enp2s0f1 magic 0x154d8086 offset 0x58 value 0xfd

root@server:~# ethtool -e enp2s0f1 offset 0x58 length 1
Offset          Values
------          ------
0x0058:         fd 
```
Do a "lspci -nn | grep X520" get the value for 0x154d8086 
```bash
root@server:~# lspci -nn | grep X520
02:00.0 Ethernet controller [0200]: Intel Corporation Ethernet 10G 2P X520 Adapter [8086:154d] (rev 01)
02:00.1 Ethernet controller [0200]: Intel Corporation Ethernet 10G 2P X520 Adapter [8086:154d] (rev 01)
```

The Intel x520 card will now accept any transceivers from any vendor.  YAY

No wonder these cards go real cheap when they are 3rd hand, peeps must be getting them, having issues then thinking they are fobbing them off to internet schmucks.  Nasty