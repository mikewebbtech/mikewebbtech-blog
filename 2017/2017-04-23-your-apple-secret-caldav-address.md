---
title: Your Apple secret CalDAV address
date: 2017-04-23T23:56:21
summary: "Having a small collection idevices in our household, we tend use the icloud calendar app and iCal coordinate our lives. With some work, non apple devices can share the same calendar "
draft: false
categories:
  - home-lab
tags:
  - mac
  - solution
series:
---
# iCal on non iDevices...yay

> Note:
>  This is historic information only.  Thunderbird has a plugin and Outlook can access Apple account calendar, address book, notes, todo lists and email 

Having a small collection idevices in our household, we tend use the icloud calendar app (and iCal on the mac's) to coordinate our appointments and reminders.  It just works and works well throughout the iVerse.

Problem is, I don't always use Apple hardware.  Last year I looked long and hard as to what my dead iMac replacement would be and settled on a powerful Xeon workstation with buckets of memory and M.2 storage....and it has room to grow.  I digress.  Point is, I'm back to running linux on my daily rig and Apple doesn't really play well with with others.

One thing that drove me nuts was the lake of connecting to my cal in the cloud with all my reminders.  I heard that Apple is useing some open standards for it's cloud service so I should be able to sync something up with it.

I've followed a few convoluted solutions that I have found on the net that have worked for both the calendar and also tasks;

One way if you do have a Mac is to dive into your ~/Library/Calendars directory.  Depending on how many accounts you have linked to, this folder can be crowded, but what you are looking for as the longs-tring-of-numbers-and-letters.caldav directory  with a bunch of *.calendar directories, an Inbox and Attachments directory.  There you will find the pot of gold, the Info.plist.  Now let us begin:

Open the Info.plist and search for uuid, the number string (e.g. 298471256) is your **user\_id.** Now search for the key DefaultCalenderPath, the associated string should resemble /**user\_id**/calendars/**calendar\_name** (e.g. /298471256/calendars/home)

Now we need the server address, search for the PrincipalURL key, the associated string should resemble https://p**NN**-caldav.icloud.com/**user\_id**/principal/. E.g. (https://p**11**-caldav.icloud.com/298471256/principal/).  There you go, you now have enough pieces of the puzzle to start connecting to and syncing third party apps with Apples CalDAV (calendar and tasks) services.

So to put it all together.  In your calender application (evolution, thunderbird, iceweazle etc) create a new calender with:
* Username: your email address you use to log into apple services (e.g. mail, itunes etc)
* Password: the password you use for above.
* Server: (e.g. https://p11-caldav.icloud.com/298471256/calendars/home)
* User secure login (words like TLS and SSL etc)
* Use CalDAV (not web)


Some CalDAV/calendering apps handle the Server information required differently.
* Server Address: (e.g. https://p11-caldav.icloud.com:443
* Server Path: (e.g. /298471256/principal/)


******* ONE HUGE REAL BIG CAVEAT ********

If you are using two-factor authentication.  THIS WILL NOT WORK in many many many cases.  Disable two-factor authentication online in your account settings and setup challenge phrases.