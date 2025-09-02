---
title: Apple CalDAV address...without .plist
date: 2017-04-24T00:25:18
summary: Alternate method to get non apple devices to acces iCal. Using three ingredients.  Your calendar's server address . Your user ID dsid.  The calendar name (cal_name).
draft: false
categories:
  - home-lab
tags:
  - mac
  - solution
series:
---
# iCal on non iDevices. Alternate Method.

> Note:
>  This is historic information only.  Thunderbird has a plugin and Outlook can access Apple account calendar, address book, notes, todo lists and email 

You will need three ingredients for to string together your CalDAV address:

a. your calendar's server address (especially, the number xx in pxx-caldav.icloud.com)
b. your user ID (dsid)
c. the name of the calander you want to connect to (cal\_name)

You can either use Firefox' or Chrome's functionality to look at the network; I will continue with Firefox (identical to Chrome, Chromium and iceweazle) for this explanation.
1. Logging in to your iCloud account using the browser.
2. Click/select the address field (where it says icloud.com) Press Ctrl + Shift + I and reload the page.
3. In the debug panel that opens on the bottom of the browser, sort the "Domain" column
4. checking the domain or just hovering over the links, you can find the xx in pxx, this is your specific server; the remainder of the domain does not matter (**information a**)
5. I then clicked on one entry with domain pxx-calendarws.icloud.com
6. in the new panel that opens to the side, you can click on Headers and find `dsid` somewhere below under Query string parameters; alternatively, you can find it in the URL; `dsid` corresponds to your user id (**information b**)
7. On your iCloud Calender page there on the left will be the names of the calendars you.  easy


There we have it.  The URL to use in  you CalDAV compatible application.

http>s://pxx-caldav.icloud.com/**dsid**/calanders/**cal\_name**