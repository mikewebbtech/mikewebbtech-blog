---
title: Peer certificate error when S/MIME signing email
date: 2017-08-16T10:41:21
summary: Notes on overcoming certificate trusted peer certificate errors when importing and using s/mime for email signing on systems that use NSS instead of OpenSSL libraries
draft: false
categories:
  - home-lab
tags:
  - linux
  - solution
  - opensource
  - smime
---

#### THE PROBLEM:
"*Peer's certificate issuer has been marked as not trusted by the user. (-8172) - Cannot add MS SMIMEEncKeyPrefs attribute", you may need to select different mail options*"

#### THE SCENARIO:
Right! so I have had this issue a couple of times.  Well in fact, once a year, every year, when I go through the annual ritual of getting a new personal digital certificate for email signing and encryption.  I'll leave my rant on the whys and hows to another post, but will say for now "JUST DO IT".

So, once I've gone through the sacrificing a goat in the name of my issuing Certificate Authority for a blessed signature file (usually PKCS#7 .p7s) I go through the right of creating the guard-it-with-your-life  PKCS#12 (Public Key Cryptography Standard No. 7) .p12 and public/private cert combos.  Sounds arcane but not as hard as it sounds and with the right OS and browser combo it is nearly automated.

I use my .p12 to move the keys I need for email signing and encryption on to my other computers and devices and into the email applications on them (MacOS, linux, IOS, evolution, thunderbird, outlook etc anything that supports PKCS).  To help with this, I temporarily upload my .p12 file to an online data store that uses end-to-end encryption and is accessible from all my platforms.  Mega is my go to solution.....screw you dropbox.

I haven't gone to deep into why I get errors sometimes when I first use my new key for signing, but it seems limited to machines where I'm using NSS instead of OpenSSL libraries and tools.  These are edge case and I suspect are due to the defaults employed by NSS for trusted peers.  Not sure as the fix is easy and done once a year.

#### THE FIX:
Install or check if you have the libnss tools package for your distribution.

Using the certutil tool, find the certificate name for the key you are trying to use for S/MIME

```
$ certutil -d sql:$HOME/.pki/nssdb -L`
```

Now mark the certificate as a trusted peer with

```
$ certutil -d sql:$HOME/.pki/nssdb -M -n "Name of Cert" -t Pu,Pu,Pu
```

In the email application make sure the correct cert is selected for the corresponding email address you are sending as and if all went well, no more error.

#### THE RESULT:
**Beer and profit**