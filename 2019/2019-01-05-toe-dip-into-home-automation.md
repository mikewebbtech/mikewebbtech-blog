---
title: Toe dip into Home Automation
date: 2019-01-05T13:24:45
summary: I see the need for home automation.  Well...I would like to explore this technology and make it work for me.  There are so many options and there are some very expensive options, not just cost but in privacy and closed ecosystem risks.
categories:
  - automation
  - home-lab
  - iot
draft: false
tags:
  - homeassitant
  - hardware
  - maker
series:
---
I have few lamps that had switches in hard to get spaces. Since these lamps also had usage patterns that were very algorithmic i.e

* Should only be on when people are home
* Turn on when the room is becoming dark (about an hour before the sun is setting)
* Some turn off at a certain time
* Some turn off when an action happens (TV turns off and as after a certain time)


I started to look at home automation but was very quickly put off by 3 things:
1.  Cost, like seriously $130 for a smart led strip or even more seeing that, with some, you have to invest in an ecosystem (like HUE)
2. Inflexible and closed.  Some of these smart systems aren't very smart and the functionality is very limited considering the costs, plus they only work with x platform so y needs its own control system ruling out an unified environment.  No ability to expand or enhance.
3. Cloud.  Everything is ending up in the cloud and I hate that,  why does a light bulb and app need to be connected to the cloud when you controlling it locally using  a local network.  The security and privacy issues here make the mind boggle.  Yes it creates simplicity and expands market penetration by turning technology into a laypersons appliance, but at what cost.  Also...lose internet or the service goes down (or company has a policy or ownership change, jacks up a monthy subscription) you loose your home automation. yay


Just as I was about push aside gut feelings and logical concerns in name of convenience,  the YouTube algorithm learnt from my search patterns and started presenting me with suggestions for a platform called Home Assistant (HASS) and OpenHab.

These opensource, community driven projects had my attention straight away and the roll your own, take back control and security, host it how you will is very much my bag baby.  Oh and the cost...electronics and MCU's are very very (yes very) cheap.

Let's do a comparison between 2 projects using HUE and HASS role-your-own(tm)

#### 2 smart RGB LED strips

**Philips HUE**
* 2 x 2m Philips Hue Lightstrip Plus = ~AU$250
* Philips HUE Bridge = AU$90
***Total = AU$340***  *plus an internet connection et al

**HASS**
* 5m 150 x SMD5050 LED strip (ws2812B individually programmable/accessible RGB) AU$25
* 2 x WEMOS D1 mini (any cheap ESP8266 controller will do) AU$7
* Raspberry Pi 3b+ (this is for running HASS so is the hub for a whole home automation system, not just controlling lights) AU$45
* 3x 5V 3A power supplies AU$15
***Total = $92***

#### Convert 4 lamps into smart lamps

**Philips HUE**
* Philips Hue White A60 E27 Starter Kit (2 bulbs )= ~AU$140
* 2 x Philips Hue White A60 E27 Extension Bulb = AU$50
***TOTAL = AU$190***

**Lifx**
* 4 x LIFX Mini A19 Day & Dusk Smart Light Bulb E27 = AU$180
***TOTAL = AU$180***

**HASS**
* 4 x SONOFF Basic (flashed to TASMOTA) =  AU$28
* Raspberry Pi 3b+ (this is for running HASS so is the hub for a whole home automation system, not just controlling lights) AU$45
* 5V 3A power supplies AU$5
***TOTAL = AU$78***

The bill of materials is a quantifiable cost but there are many not so apparent costs. I've outlined some previously for the commercial products but things get a bit esoteric for roll your own solutions. There are assumptions about knowledge in regards to microelectronics, programming, understanding the Home Assistant architecture, trouble shooting, defining your automation logic allegorically and the ability to communicate technical issues to the community for support.

Luckily enough the hurdles in roll-your-own are at a hobby level of technical difficulty and the support community is massive (just look at all the youtube tutorials..nice) and with a genuine interest and methodical approach massive outcomes can be gained with minimal(ish) expense in mind and matter.

Most definitely...Beer and Profit.