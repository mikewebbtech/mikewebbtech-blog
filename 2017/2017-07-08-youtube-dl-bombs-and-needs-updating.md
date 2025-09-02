---
title: youtube-dl bombs and needs updating
date: 2017-07-08T16:59:01
summary: youtuble-dl bombs when trying to extract from html5 player sites and updating it solves this but the native update command has stopped working and package managers are inconsistant
draft: false
categories:
  - home-lab
tags:
  - linux
  - solution
series:
---

 If you are unable to extract html5 player; please report this issue on [https://yt-dl.org/bug].

 Make sure you are using the latest version; type youtube-dl -U to update. 
 
 Be sure to call youtube-dl with the --verbose flag and include its complete output*.


Easy fix by the looks of it, type youtube-dl -U.  Unfortunately on the various flavours of linux and with MacOS this results in:

```
It looks like you installed youtube-dl with a package manager, pip, setup.py or a tarball. Please use that to update.*
```

Wow! that looks like an even easy job to do.  Just use "the what ever package manager" on your sytem. e.g. apt-get on ubuntu, dnf on fedora, yum on centos and RHEL etc etc.  For this example I'll go with apt-get as I happen to be sitting in front of an Ubuntu workstation.  I'll see what arch does next time I spin up an image.

Right..moving on. use apt-get to update youtube-dl to the last version:
```
sudo apt-get update youtube-dl
  Reading package lists... Done
  Reading package lists... Done
  Building dependency tree
  Reading state information... Done
  youtube-dl is already the newest version (2017.03.26-1).
```

What! LIES. I just checked the youtube-dl site and the latest version is 2017.07.02-1. Dam you package manager tool and your strict need for consistency and slow and methodical updating of upstream repositories for system stability.

Well linux being linux, there are many ways to skin this cat. I strongly suggest installing/leaving the current youtube-dl package as this keeps all the python dependencies intact and just update the youtube-dl "program" located in ***/usr/local/bin***.

curl and wget are your friends.  Also, seeing as youtube-dl is just a collection of python scripts, you can use the wonderful pip  command as well.  I highly encourage the use of pip, especially if you have a lot of python scripts on your system.  Keeps everything just dandy.

Not sure if you have pip?
```
sudo apt-get install python-setuptools
sudo easy\_install pip
$sudo pip install --upgrade youtube-dl
```

If you do, just

```
sudo pip install --upgrade youtube-dl
```
Using wget:
```
sudo wget https://yt-dl.org/downloads/latest/youtube-dl -O /usr/local/bin/youtube-dl
sudo chmod a+rx /usr/local/bin/youtube-dl
```

Using curl:
```
sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
sudo chmod a+rx /usr/local/bin/youtube-dl
```

There is a signature file available from the youtube-dl developers site if you are so inclined towards that type of thing or just open up the file and read it to make sure there is no dodgy code in it.

There your go. Beer and profit.
