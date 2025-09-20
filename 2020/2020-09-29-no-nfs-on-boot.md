---
title: No NFS on boot
date: 2020-09-29T10:27:11
summary: Upgrade my workstation and now NFS mounts in fstab not mounting on boot but working with mount -a command works as expected. in short, systemd is worth learning and I cant believe ubuntu was released with such a basic bug.
draft: "false"
categories:
  - home-lab
  - networking
tags:
  - ubuntu
  - linux
  - networking
series:
---

Issue: NFS mounts in fstab not mounting on boot but working with mount -a command works as expected.. 
 
 I upgraded to Ubuntu 20.04 an noticed that my NFS (v3 and v4) mounts from my file server weren't populate i.e. they weren't being mount on boot (mount command didn't show the NFS mounts). But when I ran a sudo mount -a command everything mounted as expected and confirmed with the mount command. This led me to conclude that my fstab entries were fine and the issue laid else where. Also I marked this issue as an inconvenience and maybe I should look at autofs in the future. 

 Roll on Covid-19 and lockdowns, I had some spare time to investigate this (f#%k you 2020) and noticed an error in my logs 


```
 mount.nfs: Network is unreachable 
```


 As well some other errors relating to remote-fs-pre.target. Bingo! could this be systemd, A quick google-fu and seeing some others having the same issue and list of possible issues and esoteric fixes etc and calls to just use autofs and so on. I decided to look into it myself. 


```
systemctl show nfs-client.target 
```


 Showed: 


```
WantedBy=remote-fs.target multi-user.target 
Before=shutdown.target multi-user.target remote-fs-pre.target
```


```
systemctl show remote-fs.target
```


 Showed: 


```
WantedBy=multi-user.target
After=remote-fs-pre.target
```


 And finally 


```
systemctl show remote-fs-pre.target
```


 Rather then showing what is showed, lets talk about what it didn't show. I all this chain of target stubs, nowhere was there the network-online.target. Which I was expecting to see, but was suspecting was misssing to the log error message of network unreachable. 

 With out going into the way systemd works, the NFS (network file system) client target was trying do its thing before network-online target had run and brought up the network. Think of it as chain of events that have to happen in an order to work. 
 
 My fix: Since the log error points to remote-fs-pre.target was well as the network.client.target and remote-fs.target I lokked in remote-fs-pre.target stub file: 


```
/lib/systemd/system/remote-fs-pre.target
```


 and added at the end the missing (IMHO) lines: 
 

```
Wants=network-online.target
After=network-online.target
```


 Save the file and reboot....YAY. All as it should be and one step closer to universal peace. 
 

 
 I'm not sure if this a bug of omission or I initially did something wrong or if my solution will cause other issues elsewhere. But it works and I felt vindicated in spending time in understanding systemd when the distros replaced sysv instead of joining in with the wailing and gnashing of teeth (though I'm glade I missed the whole upstart shit show). 
 

 
 As always...beer and profit.