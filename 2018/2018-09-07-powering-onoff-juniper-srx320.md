---
title: Powering On/Off Juniper SRX320.
date: 2018-09-07T09:48:15
summary: Always use the graceful method unless absolutely necessary.  i.e don't just yank the power cord out. There is console less method and a CLI method over the network
draft: false
categories:
  - networking
  - home-lab
tags:
  - networking
  - hardware
  - juniper
series:
---

Always use the graceful method unless absolutely necessary.  i.e don't just yank the power cord out

Graceful method with physical access to device with no console: Press and immediately release the Power button. The device begins gracefully shutting down the operating system and then powers itself off

Graceful method while logged in via terminal or console (CLI method):

```
root@srx320> request system power-off
Power Off the system ? [yes,no] (no) yes

Shutdown NOW!
[pid 2480]

root@srx320>
*** FINAL System shutdown message from root@gandalf ***

System going down IMMEDIATELY


Sep 6 19:28:19 init: l2-learning (PID 1719) exited with status=0 Normal Exit
Sep 6 19:28:19 init: routing (PID 1718) exited with status=0 Normal Exit
Sep 6 19:28:19 init: mib-process (PID 1717) exited with status=0 Normal Exit
SWaiting (max 60 seconds) for system process `vnlru' to stop...done
Waiting (max 60 seconds) for system process `vnlru\_mem' to stop...done
Waiting (max 60 seconds) for system process `bufdaemon' to stop...done
Waiting (max 60 seconds) for system process `syncer' to stop...
Syncing disks, vnodes remaining...0 0 0 0 done

syncing disks... Syncing disks, buffers remaining... 6 6 6 6 6 3 3 3 3 3 3 3 1 1 1 1 1 1 1
Final sync complete
Uptime: 1h27m5s
The operating system has halted.
Turning the system power off.

```

Forced shutdown with physical access to device with no console: Press the Power button and hold it for 10 seconds. The device immediately powers itself off without shutting down the operating system.

A more complete guide on syntax and options:

[Junos system power off](https://www.juniper.net/documentation/en_US/junos/topics/reference/command-summary/request-system-power-off-srx-series.html)

