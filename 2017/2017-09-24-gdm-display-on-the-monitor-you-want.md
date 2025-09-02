---
title: GDM display on the monitor you want
date: 2017-09-24T17:58:32
summary: How configure gdm when running a duel monitor setup. I'm running Gnome3 and the GNOME Display Manager (GDM) which handles graphical user login, as well as other duties....My primary monitor...
draft: false
categories:
  - home-lab
tags:
  - linux
  - solution
  - gnome
---

### History: 
I have a duel monitor setup and I'm running Gnome3 and the GNOME Display Manager (GDM) which handles graphical user login, as well as other duties....My primary monitor is the left one and I've setup my system (i.e. plugged that monitor into DP0 of my graphics card.). BIOS shows up on the left screen (to use the X term) and when testing my configuration of GDM by logging outand in, that also prompts on the left screen. All good.

### Issue: 
When I reboot or power up my system for the first time, GDM prompts on the right hand side screen. But subsequent logging in and out has it on the left hand screen (the desired function. Call my pedantic, but I use an OS that I can make work the way I want for a reason, even for little menial reasons like where what goes on start up in a graphical environment. Plus these are some big monitors and having to move my chair and crane my neck to log into the system when I've set myself up for a different monitor, well, the struggle is real man.

### Solution:
In this case, dead simple (once you know). I worked this out a while ago in ubuntu and what do you know, it's the same under fedora. It is Gnome at the end of the day (p.s. I didn't use Unity in Ubuntu, I've always stuck with Gnome to keep a consistency across distros that I use...p.p.s Canonical feel the same now and are dropping Unity and going back to Gnome.)

Once you set GDM under your user log in, it creates a file ~/.config/monitors.xml. check it out, make any other changes then

```
$ sudo cp ~/.config/monitors.xml /var/lib/gdm/.config
$ sudo reboot
```

### Feel satisfied. 
Beer and profit.