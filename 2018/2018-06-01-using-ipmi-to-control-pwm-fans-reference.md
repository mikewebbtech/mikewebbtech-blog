---
title: "Using IPMI to control PWM fans\tReferences:"
date: 2018-06-01T10:02:55
summary: IPMI is more then a way to access servers over the network without them powered up (lights out, out of band etc).  It is an Intelligent Platform Management Interface,  it can be used to interface with the hardware layer in ways the OS can't.  Let's use it  to control some fans.
draft: false
categories:
  - home-lab
  - automation
tags:
  - linux
  - hardware
  - server
series:
---

1. [What is the Intelligent Platform Management Interface](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface)
2. Whats the difference between a [PWM vs DC (i.e. normal) fan](http://www.tomshardware.com/answers/id-3151794/confused-pwm-mode-fans-asus-bios.html)
3. IPMI control over PWM (Supermicro style)
4. This is on a linux server

**WHY?**:
In a bid to help quite down a server I just built, I replaced all the cheap fans that came with my cheap server chassis, as well as the CPU cooler fan, with Noctua PWM fans.

**The Problem:**
After plugging in all the fans into the motherboard and powering up...OMG...NO! I was expecting the silence that makes you look to see if the fans are spinning and working, but  no. The fans were pulsing from MAX 2400 RPM down to 0-100 RPM - Noisy, not right and down right annoying.  A definite case of PID control in software (BIOS) not doing it's thing and in need of a helping hand.

**Falling down the Rabbit Hole**
On consumer and most workstation motherboards with PWM headers and fans, some fairly basic kung-fu with either the motherboard utility software or a 1-2 combo hit using lm-sensors and fancontrol ([https://wiki.archlinux.org/index.php/fan\_speed\_control](http://would you like to know more?)))

On server or IPMI enabled motherboards, the IPMI just takes over the fans and does its thing regardless of what you tell the fans to do with your combo hit, in fact commands like pwmconfig and sensors aren't reading things right and showing the fans. But how awesome are people, really. A group of them come up with a powerful solution descriptively named [ipmitool](https://github.com/ipmitool/ipmitool).

After reading the ipmitool man page and searching through the Supermicro support site and reading lots of solutions like <http://www.supermicro.com/support/faqs/faq.cfm?faq=18025> and other snippets that pointed to this was the way I was going to quite the beast but nothing was working and IPMI would take control (as it should...to a degree, pun intended).

I finally hit pay dirt with a guide on the servethehome site (ref[#3](https://forums.servethehome.com/index.php?resources/supermicro-x9-x10-x11-fan-speed-control.20/)). Highly recommend a read.

### Solution Time
Download and install ipmitool, either on the server or on a linux machine that has network access to the IPMI management network (in this case I installed it on the server as IPMI gives me console access to the server).

On this Supermicro board (X11SPH-nCT) there are 4 configurable fan control modes. fan control using ipmitool only works when the control mode is 'FULL'.

**what mode is fan control in?**
```
root@pve:~# ipmitool raw 0x30 0x45 0x00 02
```

- 00 = standard
- 01 = full
- 02 = optimal
- 04 = heavy IO

**We need the mode to be full ( i.e 01)**

If I set my "Zone 1" values too low (120mm fans on the disks) and the speed drops below the threshold, all my good work is undone and IPMI takes over and everything is running at full speed. To overcome this I lowered the IPMI threshold values for the individual fans and now I can sit next to my server peacefully. I would like to say in silence but now I can hear my 8 SAS drives.

Lower the IPMI fan speed thresholds
`"sensor thresh lower "`

```
root@pve:~# ipmitool sensor thresh FANA lower 100 225 300
Locating sensor record 'FANA'...
Setting sensor "FANA" Lower Non-Recoverable threshold to 100.000
Setting sensor "FANA" Lower Critical threshold to 225.000
Setting sensor "FANA" Lower Non-Critical threshold to 300.000

root@pve:~# ipmitool sensor thresh FANB lower 100 225 300

```


