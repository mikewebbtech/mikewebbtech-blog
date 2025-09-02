---
title: PPA Public Key Error
date: 2017-03-30T12:35:52
summary: Though, every now and then I hit a snag when installing a new package, especially from a PPA. The most common of these relate to security/verification using PKI.
draft: false
categories:
  - home-lab
tags:
  - ubuntu
  - linux
  - solution
---

Though, every now and then I hit a snag when installing a new package, especially from a PPA.  The most common of these relate to security/verification using PKI.  to varify the authenticity of a package, usually you check the md5 hash and compare it with what the developer has published.  [Canonical](https://www.canonical.com/) goes one step further and uses GPG (PGP) signing on packages in their repositories and those found in launchpad PPA repositories as well as hosting a key server ([keyserver.ubuntu.com](http://keyserver.ubuntu.com)).

Every so often this gets out of whack.  Keys expire, maintainers change or wrong phase of the moon and you end up with something resembling:

```text
mike@obsidian:/etc/network$ sudo apt update
Get:14 http://ppa.launchpad.net/gns3/ppa/ubuntu yakkety InRelease [17.5 kB] 
Err:14 http://ppa.launchpad.net/gns3/ppa/ubuntu yakkety InRelease 
 The following signatures couldn't be verified because the public key is not available: NO\_PUBKEY 9A2FD067A2E3EF7B
W: GPG error: http://ppa.launchpad.net/gns3/ppa/ubuntu yakkety InRelease: The following signatures couldn't be verified because the public key is not available: NO\_PUBKEY 9A2FD067A2E3EF7B
E: The repository 'http://ppa.launchpad.net/gns3/ppa/ubuntu yakkety InRelease' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```

Or better know as "splat".  Like in any self respecting system, there are many ways to skin a cat depending on circumstances and personal preferences.  The easiest way I usually find to resolve PPA key issues with the most succes is to use the apt-key command.

```text
mike@obsidian:~/$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 9A2FD067A2E3EF7B
Executing: /tmp/tmp.DSAfPm9NVu/gpg.1.sh --keyserver
keyserver.ubuntu.com
--recv-keys
9A2FD067A2E3EF7B
gpg: key 9A2FD067A2E3EF7B: public key "Launchpad PPA for GNS3" imported
gpg: Total number processed: 1
```

Of course a developer doesn't have to use the Canonical key server.  They maybe using one of a myriad of trusted and secure key servers already to host their public key as shown in this example for an alternative way to import a key used for package signing.

```text
mike@obsidian:~/$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 9A2FD067A2E3EF7B
$sudo gpg --keyserver keyserver.pgp.com --recv-key <PUBKEY>
$sudo gpg -a --export <PUBKEY> | sudo apt-key add -
$sudo apt-get update

mike@obsidian:~/$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 9A2FD067A2E3EF7B
```

P.S.  We all should be using some sort of PKI for the transmission of data over the internet