---
title: SAS disk power mode control
date: 2018-10-08T15:54:58
summary: My hard drives were spinning at 7600 RPM 24/7, this is fine for aa actual server for the masses or in DC, but total overkill for my intermittent needs. Why do I want power control over my spinning rust...spin down to save noise and heat = save energy.
draft: false
categories:
  - home-lab
tags:
  - linux
  - storage
  - server
series:
---

My hard drives were spinning at 7600 RPM 24/7...Arrgg!

Why do I want power control over my spinning rust...spin down to save noise and heat = save energy

For my home lab I buy enterprise HDD's from ebay et al at amazing price points per Gb.  Very few have issues  but all need some amount attention beyond that when purchasing consumer grade new.

My defaults are to always integrate and update and wipe with a bad blocks check , adjust block size if needed  and stress test prior to adding them to a storage pool.

My latest acquisitions were 8 x 4Tb SAS drives.  HGST Ultrastar 7K600 ($50 each).  After doing my default and setting up a pool (RAID2z) and having the server run for awhile, I realised that the PWM fans wouldn't throttle.

After writing a little script to monitor temperature and fan sensors via IPMI and then querying the HDD temps using smartmontools, it became clear that the unaccounted heat was coming from the HDD's.  They weren't spinning down, but why and how to change that behavior.

These are SCSI drives, the usual hdparm tool command don't work.

**Information Gathering**

**Let's get some basics**

```bash
# smartctl -i /dev/sda
smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-862.14.4.el7.x86_64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Vendor: HGST
Product: HUS726040ALS214
Revision: MS00
Compliance: SPC-4
User Capacity: 4,000,787,030,016 bytes [4.00 TB]
Logical block size: 512 bytes
LU is fully provisioned
Rotation Rate: 7200 rpm
Form Factor: 3.5 inches
Logical Unit id: 0x5000cca25d526df4
Serial number: K4HGAAAB
Device type: disk
Transport protocol: SAS (SPL-3)
Local Time is: Mon Oct 8 08:33:15 2018 AWST
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
Temperature Warning: Enabled
Checking the health
`# smartctl -H /dev/sda
smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-862.14.4.el7.x86_64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Health Status: OK`
```

**And a bit more information including the temperature**

```bash
# smartctl -A /dev/sda
smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-862.14.4.el7.x86_64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
Current Drive Temperature: 39 C
Drive Trip Temperature: 55 C

Manufactured in week 51 of year 2016
Specified cycle count over device lifetime: 50000
Accumulated start-stop cycles: 88
Specified load-unload count over device lifetime: 600000
Accumulated load-unload cycles: 99
Elements in grown defect list: 0

Vendor (Seagate) cache information
Blocks sent to initiator = 1063292319563776

```

**Examine selftest history**

```bash
# smartctl -l selftest /dev/sda
smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-862.14.4.el7.x86_64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Self-test log
Num Test Status segment LifeTime LBA_first_err [SK ASC ASQ]
Description number (hours)
# 1 Background short Completed - 257 - [- - -]
# 2 Background long Completed - 45 - [- - -]

Long (extended) Self Test duration: 6 seconds [0.1 minutes]
```

**Start a short foreground selftest and verify results**

```
# smartctl -C -t short /dev/sda
smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-862.14.4.el7.x86_64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

# smartctl -l selftest /dev/sda
smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-862.14.4.el7.x86_64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART Self-test log
Num Test Status segment LifeTime LBA_first_err [SK ASC ASQ]
Description number (hours)
# 1 Foreground short Completed - 438 - [- - -]
# 2 Background short Completed - 257 - [- - -]
# 3 Background long Completed - 45 - [- - -]

Short Foreground Self Test Successful
```

**Look at the error log**

```bash
# smartctl -l error /dev/sda
smartctl 6.5 2016-05-07 r4318 [x86_64-linux-3.10.0-862.14.4.el7.x86_64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
Error counter log:
Errors Corrected by Total Correction Gigabytes Total
ECC rereads/ errors algorithm processed uncorrected
fast | delayed rewrites corrected invocations [10^9 bytes] errors
read: 0 1 0 1 9555 10267.843 0
write: 0 0 0 0 7138 15571.768 0
verify: 0 0 0 0 15629 0.000 0

Non-medium error count: 0
```

**Examine smarttools extended output with background media scan (BMS) is active**

```bash
# smartctl -x /dev/sda
<--snip-->
Background scan results log
Status: scan is active
Accumulated power on time, hours:minutes 763:09 [45786 minutes]
Number of background scans performed: 8, scan progress: 66.08%
Number of background medium scans performed: 8
<--snip-->
```

Background scanning is active, this wont affect disk performance as "The BMS process works at idle time, when the disk received no commands" i.e. the disk wont spin down as usual when it is idle.

> [!note]
> (Please read the sdparm man page before blindly doing a copy paste and to get a better understanding of the options and switches. They may not be compatible with your drive.)*

**Let's urn off BMS on this SAS drive**

```bash
# sdparm --clear=EN_BMS --save /dev/sda

```

**Query the disks abilty and value for standby**

```bash
# sdparm --flexible -6 -p po -l /dev/sda | grep -e SCT
SCT 0 [cha: y, def: 0, sav: 0] Standby_z condition timer (100 ms)

```

**No counters defined (SCT 0 = not countdown time till standby)**

```bash
# sdparm --flexible -6 -p po -l /dev/sda | grep -e "STANDBY "
STANDBY 0 [cha: y, def: 0, sav: 0] Standby_z timer enable

```

### Standby not enabled (STANDBY 0)

*what is Standby (standby\_Z)*
- Heads are unloaded to drive ramp.
- Drive motor is spun down.
- Drive still responds to non-media access host commands

**Let's enable it**

```bash
# sdparm --flexible -6 -v -S -p po --set=STANDBY=1 /dev/sda
mp_settings: page,subpage=0x1a,0x0 num=1
pdt=-1 start_byte=0x3 start_bit=0 num_bits=1 val=1 acronym: STANDBY
>>> about to open device name: /dev/sda
/dev/sda: HGST HUS726040ALS214 MS00
mode sense (6) cdb: 1a 00 1a 00 04 00
mode sense (6) cdb: 1a 00 1a 00 34 00
mode select (6) cdb: 15 11 00 00 34 00
```

**Important to verify it**

```bash
# sdparm --flexible -6 -p po -l /dev/sda |grep -e "STANDBY "
STANDBY 1 [cha: y, def: 0, sav: 1] Standby_z timer enable
```

**Set the SCT (Standby\_z condition timer in units of 100ms so 9000=15 minutes)**

```bash
# sdparm --flexible -6 -v -S -p po --set=SCT=9000 /dev/sda
mp_settings: page,subpage=0x1a,0x0 num=1
pdt=-1 start_byte=0x8 start_bit=7 num_bits=32 val=9000 acronym: SCT
>>> about to open device name: /dev/sda
/dev/sda: HGST HUS726040ALS214 MS00
mode sense (6) cdb: 1a 00 1a 00 04 00
mode sense (6) cdb: 1a 00 1a 00 34 00
mode select (6) cdb: 15 11 00 00 34 00
```

**Remember: Verify**

```bash
# sdparm --flexible -6 -p po -l /dev/sda |grep -e SCT
SCT 9000 [cha: y, def: 0, sav:9000] Standby_z condition timer (100 ms)
```

**Let's view all contents of the Power condition [po] mode page**

```bash
# sdparm --flexible -6 -p po -l /dev/sda
/dev/sdb: HGST HUS726040ALS214 MS00
Direct access device specific parameters: WP=0 DPOFUA=1
Power condition [po] mode page:
PM_BG 0 [cha: y, def: 0, sav: 0] Power management, background functions, precedence
STANDBY_Y 0 [cha: y, def: 0, sav: 0] Standby_y timer enable
IDLE_C 0 [cha: y, def: 0, sav: 0] Idle_c timer enable
IDLE_B 0 [cha: y, def: 0, sav: 0] Idle_b timer enable
IDLE 0 [cha: y, def: 0, sav: 0] Idle_a timer enable
STANDBY 1 [cha: y, def: 0, sav: 1] Standby_z timer enable
ICT 20 [cha: y, def: 20, sav: 20] Idle_a condition timer (100 ms)
SCT 9000 [cha: y, def: 0, sav:9000] Standby_z condition timer (100 ms)
IBCT 6000 [cha: y, def:6000, sav:6000] Idle_b condition timer (100 ms)
ICCT 0 [cha: y, def: 0, sav: 0] Idle_c condition timer (100 ms)
SYCT 0 [cha: y, def: 0, sav: 0] Standby_y condition timer (100 ms)
CCF_IDLE 0 [cha: y, def: 0, sav: 0] check condition on transition from idle
CCF_STAND 0 [cha: y, def: 0, sav: 0] check condition on transition from standby
CCF_STOPP 0 [cha: y, def: 0, sav: 0] check condition on transition from stopped
```

> [!NOTE]
>  IBCT (Idle\_b condition timer) has has a condition of 6000 (10 minutes) but is not enable (IDLE\_B 0) it.   Let's enable and reduce it down to 5 minutes as this is a disk used in a zpool for archives and backups and occasional media file access

**What is Idle\_B**:
- Disables most of the servo system, reduces processor and channel power consumption
- Heads are unloaded to drive ramp.
- Disks rotating at full speed (7200 RPM)

**Enable the condition timer**
```bash
# sdparm --flexible -6 -v -S -p po --set=IDLE_B=1 /dev/sda
mp_settings: page,subpage=0x1a,0x0 num=1
pdt=-1 start_byte=0x3 start_bit=2 num_bits=1 val=1 acronym: IDLE_B
>>> about to open device name: /dev/sda
/dev/sdb: HGST HUS726040ALS214 MS00
mode sense (6) cdb: 1a 00 1a 00 04 00
mode sense (6) cdb: 1a 00 1a 00 34 00
mode select (6) cdb: 15 11 00 00 34 00
```

**Set the condition timer to 5 minutes**
```bash
# sdparm --flexible -6 -v -S -p po --set=ICT=3000 /dev/sda
mp_settings: page,subpage=0x1a,0x0 num=1
pdt=-1 start_byte=0x4 start_bit=7 num_bits=32 val=3000 acronym: ICT
>>> about to open device name: /dev/sda
/dev/sdb: HGST HUS726040ALS214 MS00
mode sense (6) cdb: 1a 00 1a 00 04 00
mode sense (6) cdb: 1a 00 1a 00 34 00
mode select (6) cdb: 15 11 00 00 34 00
```

**Now verify the changes**
```
# sdparm --flexible -6 -p po -l /dev/sda |grep -e "ICT\|IDLE_C "
IDLE_C 1 [cha: y, def: 0, sav: 1] Idle_c timer enable
ICT 3000 [cha: y, def: 20, sav:3000] Idle_a condition timer (100 ms)
```

Repeat all of the above for all SAS drives in your storage pool. (hint FOR LOOP)
Bonus points: Reboot and verify changes to power condition survives power cycle.

I just need to workout how to get the power state of the drives. I see a difference in power consumption as the drives draw less power and the chassis is cooler so the PWM fans drop to around 500-600 RPM, drawing less power, instead of screaming at 1600RPM. But everytime I use smartctl to query the drives it must bee spinning them up again as noise, heat and power consumption goes up.

Beer and profit people!