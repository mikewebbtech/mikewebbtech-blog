---
title: Wireshark capture permission errors
date: 2017-03-18T14:30:06
summary: I installed Wireshark the other day on Ubuntu 16.10 so I could analyse what is going over the wire in GNS3 and later on physical network topologies. Unfortunately, Wireshark has implemented Privilege Separation which means that the Wireshark GUI (or the tshark CLI) can run as a normal user while the dumpcap capture utility runs as root
draft: false
categories:
  - home-lab
  - security
tags:
  - linux
  - ubuntu
  - solution
  - opensource
series:
---

I installed Wireshark the other day on Ubuntu 16.10 so I could analyse what is "going over the wire" in GNS3 and later on physical network topologies.

Unfortunately, due to the way I installed wireshark, the way it handles the security of low level device access (PCAP) , coupled with a touch of me not paying the level of attention needed with installing.  I locked myself out of user level access to a network interface. ala ""You don't have permission to capture on that device".

According to the wireshark wiki "Wireshark has implemented [Privilege Separation](https://wiki.wireshark.org/Development/PrivilegeSeparation) which means that the Wireshark GUI (or the tshark CLI) can run as a normal user while the dumpcap capture utility runs as root. This can be achieved by installing dumpcap setuid root. The advantage of this solution is that while dumpcap is run as root the vast majority of Wireshark's code is run as a normal user (where it can do much less damage)."
When Wireshark is installed using a Linux distributions package manager, they usually provide package managers which handle installation, configuration and removal of software packages. Wireshark is provided by several distributions and some of them help in configuring dumpcap to allow capturing even for non-root users. Though with Ubuntu (and other Debian derivatives) when installing Wireshark packages non-root users won't gain rights automatically to capture packets. To allow non-root users to capture packets follow the procedure described in /usr/share/doc/wireshark-common/README.Debian. i.e run 
```
 $ sudo dpkg-reconfigure wireshark-common*
 ```
A more agnostic approach to setting network privileges for dumpcap if your kernel and file system support file capabilities: 
1. Ensure that you have installed the necessary tools, such as the setcap command. 
2. 
```
  sudo setcap 'CAP\_NET\_RAW+eip CAP\_NET\_ADMIN+eip' /usr/sbin/dumpcap* 
```

> NOTE: 
> Replace /usr/sbin with /usr/bin in case you receive an error that indicates that dumpcap isn't in /usr/sbin) 

3. Start Wireshark as non-root and ensure you see the list of interfaces and can do live capture. 
Another way of setting network privileges for dumpcap if your kernel and file system don't support file capabilities would require the need to make dumpcap set-UID to root. 
4. 
```
	sudo chown root /usr/sbin/dumpcap*
```
> NOTE: 
> Replace /usr/sbin with /usr/bin in this command and the next command in case you receive an error that indicates that dumpcap isn't in /usr/sbin) 

5. 
```
$ sudo chmod u+s /usr/bin/dumpcap* 
```
You can also limit capture permission to only one group after having set dumpcap's network privileges: 
6. Create user "wireshark" in group "wireshark". 
```
sudo chgrp wireshark /usr/sbin/dumpcap* 
sudo chmod o-rx /usr/sbin/dumpcap*
``` 
6. Ensure Wireshark works only from root and from a user in the "wireshark" group

If non of those steps work more drastic measures such as reinstalling wireshark maybe needed but that may not solve it as the issue could be underling in the operating system.

































