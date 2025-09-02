---
title: Using your iCloud address book in Linux
date: 2017-08-11T13:11:32
summary: Notes on how to get any application that supports WebDAV or CardDAV to access apple iCloud resources. Following on from previous posts about using Apples iCloud services on non Apple devices or operating systems.
draft: false
categories:
  - home-lab
tags:
  - mac
  - opensource
  - linux
  - solution
  - icloud
---

### Not just linux but any application that supports WebDAV or CardDAV.

> [!NOTE]
>  This is now redundant.  Thunderbird has plugins and Outlook can handle this natively

Following on from previous posts about using Apples iCloud services on non Apple devices or operating systems.  See.
- [Apple CalDAV address…without .plist](https://mikewebblive.wordpress.com/2017/04/24/apple-caldav-address-without-plist/)
- [Your Apple “secret” CalDAV address](https://mikewebblive.wordpress.com/2017/04/23/your-apple-secret-caldav-address/)

Once you have found the dsid for your iCloud account you will also need to setup two factor authentication for the computer/device you are setting up your address book on.

Log into you apple accoun management page via icloud.com to setup TFA

For the next step, I'm using Evolution Mail, but you can use any application that supports CardDAV or WebDAV

If you followed the steps in one of the two links above, you would have you CalDAV addess that looks something similar to:
https://pxx-caldav.icloud.com//calanders/
e.g. https://p10-caldav.icloud/223445234/calanders/home/

Convert this to the iCloud CardDAV address:
e.g. https://p10-contacts.icloud/223445234/carddavhome/card/

So the important information is:
Server: hhttps://p10-contacts.icloud/223445234/carddavhome/card/
User Name: your iCloud login
Password: your iCloud two factor authentication password for the device/computer
Name: iCloud Adressbook (or what ever)

In Evolution Mail (one of a couple very popular linux mail and calendar applications) create a new address book.

File -> New -> Address book
Type: = WebDAV
Name = iCloud
select "Mark as deault address book" (optional)
select "autocomplete with this address book"
select "Copy book content locally for offline operation"
URL: = https://p10-contacts.icloud/223445234/carddavhome/card/ (<-e.g.)
User: = your\_iCloud\_login

When you first sync to the address book you will be prompted for a password, don't use your iCloud accound password, this will fail, use the Two Factor Authentication password you setup for the device/computer.

Done, beer and profit.