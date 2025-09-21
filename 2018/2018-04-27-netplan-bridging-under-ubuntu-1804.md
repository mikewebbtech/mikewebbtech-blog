---
title: Netplan bridging under Ubuntu 18.04
date: 2018-04-27T16:40:42
summary: The new trend in linux is the shift to netplan as a network manager that uses either NetworkManager  or  systemd-networkd to manage interfaces is pretty much the default now for Ubuntu Desktops.  This breaks OVS and linux bridging, but there is a way
draft: false
categories:
  - networking
  - virtualisation
  - home-lab
tags:
  - linux
  - ubuntu
  - networking
  - lxc
---

I just completed a fresh (minimal) install of Ubuntu 18.04 Desktop. I used the new LTS release as an excuse to redo my workstation to simplify and unify since my use case flipped from more KVM virtual machines to more LXD containers. In fact I've gone from over provisioning 8 KVM VM's to a much higher density of 20 containers when really getting into things (on a 4 core XEON 64GB).

I used Open VSwitch for my last setup. But OVS didn't go too smooth on this new install and I'm a bit pressed at the moment to work my way through it (looking at you with stink eye netplan). So, I'll stick with the standard, yet easy and fast, linux bridging using bridge-utils.

First up, I've got some mixed opinions on this push by many Linux Distros to make NetworkManager (NM) the main tool for network...um...management. On my laptop, brilliant and it makes sense as jumping around from wifi, to ethernet, using VPNs etc is a breeze with the NetworkManager gui and use of profiles. BUT it is a REAL PAIN with bridging. On a server and workstation, I think it adds more pain then it's worth. So it makes sense to to use systemd.network (netd)

(edit: Ubuntu 18.04 Desktop defaults to NetworkManager and Server defaults to systemd.network).

But wait there is more and another curve ball introduced to the new 18.04 LTS and it's called Netplan [(would you like to know more?)](https://netplan.io/). After a quick read about it (re OVS not working) I get why it exists, but geez another level of complexity (or just yet another markup language to learn)..great!

OK! so netplan is a renderer and is used to configure NM or networkd but to quote the website linked above _"Using networkd as a renderer does not let devices automatically come up using DHCP; each interface needs to be specified in a file in /etc/netplan for its configuration to be written and for it to be used in networkd."_ (I think this also applies to OVS). Yep and I'm using DHCP so learn yaml for short.

**Sleeves up.**
make sure you bridge-utilites installed

```
mike@obsidian:~]$sudo apt install bridge-utilites -y
```

Make a backup the yaml file 01-network-manager-all.yaml

```
[mike@obsidian:~]$cd /etc/netplan/
[mike@obsidian:netplan]$ sudo cp cp 01-network-manager-all.yaml 01-network-manager-all.yaml-NM
```

Find the name of your active network interface, using ip is probably the easiest, the active one is the one with the ip address in the same subnet as your network (in my case its enp7s0
$ ip a

Open the yaml file for editing

```
[mike@obsidian:netplan]$sudo vi 01-network-manager-all.yaml
```

and change its contents to read

```
#Let networkd manage all devices on this system
network:
  version: 2
  renderer: networkd
  ethernets:
    enp7s0:
     dhcp4: no
     dhcp6: no

  bridges:
    br0:
      interfaces: [enp7s0]
      dhcp4: true
      dhcp6: no
    br1:
      interfaces: [enp7s0]
      dhcp4: true
      dhcp6: no

```

- Don't copy and pass the above, WP striped the proper yaml formatting so it won't work. I have a properly formatted example in my gist over at github [If you are interested](https://gist.github.com/mikewebb70/c46be8216e8f1594b1077f3d5220c22b)

---

I found a great resource on more properties you could declare in this file specific to your network (static IP, VLANS, bonds, routes etc). [Check it out](https://cloudinit.readthedocs.io/en/latest/topics/network-config-format-v2.html#examples)

---

Generate the required configuration for the renderers.

```
mike@obsidian:~]$sudo netplan generate
```

Apply all configuration for the renderers, restarting them as necessary.

```
**mike@ob**sidian:~]$sudo netplan apply
```

Check your handy work.

```
mike@obsidian:~]$ networkctl list
IDX LINK             TYPE               OPERATIONAL SETUP
  1 lo               loopback           carrier     unmanaged
  2 enp7s0           ether              carrier     configured
  3 enp0s31f6        ether              off         unmanaged
  4 enp65s0f0        ether              off         unmanaged
  5 enp65s0f1        ether              off         unmanaged
  6 br0              ether              routable  configured
  7 virbr0           ether              no-carrier  unmanaged
  8 virbr0-nic       ether              off         unmanaged
  9 lxdbr0           ether              routable  unmanaged

```

That should be enough ping out from the host and use br0 as the interface for lxd and KVM and the guest OS will pull valid IP settings from your network DHCP server and be able to ping out and be accessible from the local network (even pull down an install image from the network tftp server if you have set your KVM up fro network boot).

As always. Beer and profit.

