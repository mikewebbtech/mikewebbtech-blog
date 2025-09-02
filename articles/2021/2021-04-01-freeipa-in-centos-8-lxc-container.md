---
title: FreeIPA in CentOS 8 LXC container
date: 2021-04-01T10:57:39
summary: FreeIPA is to linux as Active Directory is to Windows.  This is a walk though on how I got it setup and running in CentOS 8 LXC container on Proxmox VE server.  Low resource demands compared to a dedicated vurtual machine.
draft: false
categories:
  - home-lab
  - virtualisation
tags:
  - linux
  - lxc
  - redhat
  - FreeIPA
  - server
series:
---

- What is FreeIPA - [www.freeipa.org](http://www.freeipa.org)
- LXC and LXD containers - [linuxcontainers.org](http://linuxcontainers.org)
- What is a nice drink - <https://en.wikipedia.org/wiki/Negroni>

As part of creating a new home lab and home services and network infrastructure, I've opted to go with FreeIPA with integrated DNS as my centralised identity management platform. I could have installed Windows Server 2019 on my servers and called it a day with added cost, but with AAA idm, storage, hypervisor, etc., in a single shiny package, but...back on topic.

FreeIPA in a container. Why? Containers allow for greater density in my servers. I'm using Proxmox as my storage and hypervisor cluster infrastructure. I've seen information on running FreeIPA as a Docker application as well as in a virtual machine. My decision to run it in a machine container was mainly resource constraints, and Proxmox supports LXC out of the box and is the way they do containers.

The assumptions here are that you:

- Have created a CentOS 8 LXC container using whatever method (Proxmox GUI/CLI or LXD) and whatever server (Proxmox, cloud VM, or other Linux server).
- Are going to set up two containers with FreeIPA in master/master replication. I mean, this will be the lifeblood of auth, access, and DNS for the entire network; let’s have a spare up our sleeve. Oh, and containers in a cluster can be moved around the different servers. Nothing like a bit of redundancy.

Only gotcha at this point is if you are controlling container resources, is to make sure you have 2048M memory minimum, or ipa-server-install will fail at [6/9] updating and a whole lot of LDAP errors.

For my setup, I have:

```bash
root@pve1:~ pct config 501
arch: amd64
cores: 2 features:
keyctl=1
hostname: ipa01
memory: 4096
onboot: 1
ostype: centos
rootfs: size=8G
swap: 512
unprivileged: 1
```

## Let us begin

Let’s get some preliminary housekeeping out of the way first.

My environment:

```bash
LXC containers = 2 x CentOS 8
domain = hugel.lan
firewall zone = internal
first instance name = ipa01
first instance ip = 192.168.1.35
first instance hostname = ip01.hugel.lan
second instance name =ipa02
second instance ip = 192.168.1.36
second instance hostname = ipa02.hugel.lan
DNS forwarders:	192.168.1.5 1.1.1.1
container user = conop
ntp servers = 192.168.1.1, 2.au.pool.ntp.org
```

Gain shell/console access into your container using either lxc console ipa01 or lxc exec ipa01 -- /bin/sh.

Do a yum update, set up, and check ip address and internet connectivity. Create a user with wheel group membership.

```bash
root@pve1:~ useradd conop
root@pve1:~ passwd conop
root@pve1:~ usermod -aG wheel conop

root@pve1:~ nmcli connection modify eth0 \
> ipv4.method manual \
> ipv4.address 192.168.1.35/24 \
> ipv4.gateway 192.168.1.5 \
> ipv4.dns 1.1.1.1
root@pve1:~ nmcli connection down eth0 ; nmcli c up eth0
root@pve1:~ ping 1.1.1.1
root@pve1:~ ping redhat.com

root@pve1:~ timedatectl set-timezone Australia/Perth
root@pve1:~ timedatectl set-ntp true

root@pve1:~ hostnamectl set-hostname ipa01.hugel.lan
root@pve1:~ echo "192.168.1.35 ipa01.hugel.lan ipa01 \n192.168.1.36 ipa02.hugel.lan ipa02" >> /etc/hosts
root@pve1:~ mv /etc/resolv.conf /etc/resolv.conf-BAK
root@pve1:~ echo -e "search hugel.lan\nnameserver 192.168.1.5 \nnameserver 1.1.1.1" > /etc/resolv.conf

root@pve1:~ yum update
root@pve1:~ yum install openssh-server sudo less firewalld audit -y
root@pve1:~ systemctl enable sshd --now
root@pve1:~ systemctl status sshd
root@pve1:~ systemctl enable firewalld --now
root@pve1:~ systemctl status firewalld

root@pve1:~ exit
```

There, now we have a functional, connected, and fresh container that we can SSH into as conop, probably using a laptop sitting in a comfy chair with a nice drink...just saying.

OK! Settle in to install and set up FreeIPA (Hint: nice drink!).

## Install and Configure FreeIPA

(and also fix)

Since we are running FreeIPA in a container (system cgroups) vs. a virtual machine (paravirtualised hardware), there are some things that will bite the installation and running process as we are running an OS within another OS, not on top of it.

FreeIPA in LXC gotcha! It will fail to install with

```bash
"Fatal error : adjtimex(0x8001) failed : Operation not permitted"
```

Apparently, chronyd doesn't like running in a container. Lucky enough, developers involved with chronyd wanted to do just that, so a fix was made. Tell it that it is running in a container (why this change is not part of the container image is a mystery).

Edit (i.e. vi) /etc/sysconfig/chronyd and add -x to OPTIONS="" e.g.

```bash
# Command-line options for chronyd
OPTIONS="-x"
/etc/sysconfig/chronyd (END)
```

Now we need to take care of SELinux and the firewall. Set SELinux to permissive mode and make it permanent and open up ALL the ports required by FreeIPA and replication: If you forget, the install will give you a nice reminder e.g.

```bash
You must make sure these network ports are open:
  TCP Ports:
    * 80, 443: HTTP/HTTPS
    * 389, 636: LDAP/LDAPS
    * 88, 464: kerberos
    * 53: bind
  UDP Ports:
    * 88, 464: kerberos
    * 53: bind
    * 123: ntp
```

Use the built-in firewall-cmd profiles to easily take care of the ports.

```bash
firewall-cmd --get-active-zone
firewall-cmd --set-default-zone=internal
firewall-cmd --zone=internal --list-services
firewall-cmd --add-service={freeipa-ldap,freeipa-ldaps,freeipa-replication,dns,ntp} --permanent
firewall-cmd --reload
firewall-cmd --zone=internal --list-services
```

We also need to deal with security-enhanced Linux (SELinux), kernel-level LSM (Linux Security Modules) by putting it into Permissive mode and making the change stick over reboots, basically disabling it from enforcing policy but still logging violations so we can get a sense of what is happening and make the required SELinux policy changes after we have everything working (a much later blog post).

```bash
root@pve1:~ setenforce Permissive
root@pve1:~ sed -i 's/SELINUX=enforcing/SELINUX=permisive/' /etc/selinux/config
```

Now is a good time to reboot the container (a couple of seconds) before we put the meat into this sandwich, just to make sure that everything is as it should be, maybe get a refill of that nice drink.

## Here be Dragons!

CentOS and Red Hat 8 have introduced AppStream modules. That is, installing freeipa is a bit different nowadays...and a bit better (but not perfect). Install the DL1 stream of the IDM module

```go
root@pve1:~ dnf module -y install idm:DL1/{server,dns}
```

and run **ipa-server-install** and good luck........

No, there is a bit more to it if you would like to increase your success in installing and running FreeIPA, though just running the above command will prompt you through to the finish, you may be missing out on some items.

Run "ipa-server-install --help" for a list of all the switches and a better understanding of what I have used.

> [!CAUTION]
> Be aware that the way I did it will put the directory manager and the admin user password in plain text in history (and on the screen).

Get comfy and run:

```bash
root@pve1:~ ipa-server-install -v \
> --unattended \
> --setup-dns \
> --auto-reverse \
> --forwarder 192.168.1.5 \
> --forwarder 1.1.1.1 \
> --domain=hugel.lan \
> --realm=HUGEL.LAN \
> --no-host-dns \
> --idstart=3000
> --idmax=10000 \
> --setup-kra \
> --reverse-zone=1.168.195.in-add.arpa
> -p dsStrong5ecret99 \
> -a aStrong5ecret99 \
> --ntp-server=192.168.1.1 \
> --ntp-server=2.au.pool.ntp.org
```

AND....if all went well and after a lot of information scrolling up the screen and some time had passed (and maybe another refill of a nice drink), you should be greeted with

```bash
Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
The ipa-server-install command was successful
```

There is some good advice there...back up the CA certificates. Oh and since it is a LXC container, take a snapshot.

Let's verify the install and check in on FreeIPA and Kerberos.

```bash
 ipactl status
Directory Service: RUNNING
krb5kdc Service: RUNNING
kadmin Service: RUNNING
named Service: RUNNING
httpd Service: RUNNING
ipa-custodia Service: RUNNING
pki-tomcatd Service: RUNNING
ipa-otpd Service: RUNNING
ipa-dnskeysyncd Service: RUNNING
ipa: INFO: The ipactl command was successful

 kinit admin
Password for admin@HUGEL.LAN:
 klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: admin@HUGEL.LAN

Valid starting     Expires            Service principal
31/03/21 16:27:46  01/04/21 16:27:38  krbtgt/HUGEL.LAN@HUGEL.LAN

 ipa config-show
  Maximum username length: 32
  Maximum hostname length: 64
  Home directory base: /home
  Default shell: /bin/sh
  Default users group: ipausers
  Default e-mail domain: hugel.lan
  Search time limit: 2
  Search size limit: 100
  User search fields: uid,givenname,sn,telephonenumber,ou,title
  Group search fields: cn,description
  Enable migration mode: FALSE
  Certificate Subject base: O=HUGEL.LAN
  Password Expiration Notification (days): 4
  Password plugin features: AllowNThash, KDC:Disable Last Success
  SELinux user map order: guest_u:s0$xguest_u:s0$user_u:s0$staff_u:s0-s0:c0.c1023$sysadm_u:s0-s0:c0.c1023$unconfined_u:s0-s0:c0.c1023
  Default SELinux user: unconfined_u:s0-s0:c0.c1023
  Default PAC types: MS-PAC, nfs:NONE
  IPA masters: ipa01.hugel.lan
  IPA master capable of PKINIT: ipa01.hugel.lan
  IPA CA servers: ipa01.hugel.lan
  IPA CA renewal master: ipa01.hugel.lan
  IPA DNS servers: ipa01.hugel.lan

 ping redhat.com -c 1
PING redhat.com (209.132.183.105) 56(84) bytes of data.
64 bytes from redirect.redhat.com (209.132.183.105): icmp_seq=1 ttl=139 time=134 ms

--- redhat.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 133.794/133.794/133.794/0.000 ms

```

All appears to be as it should. On a desktop or workstation, change the DNS server setting to point to the ip address of the FreeIPA container (for me 192.168.1.35) and point a browser to the hostname of the FreeIPA container, you should be greeted with a login page, use admin and the admin password defined during the install.

Now point the browser to the search engine of your choice, if the page comes up, DNS is working and you have a resource at hand to help with troubleshooting the next bit.

Reboot the container and rerun the verification commands you just ran before.....OH JOY!!!

## Coercion to +++ result

Yep it's broken (maybe not, if so..good for you, thanks for stopping by and look out for my blogs on using FreeIPA). Let’s look at some commands to help you troubleshoot and research on the internet.

```bash
 systemctl status ipa.service

 systemctl status dirsrv@HUGEL-LAN.service

 journelctl -xe
```

In my case, which was the case in five container installs of FreeIPA on CentOS 8, the issue was the 389 Directory service. Specifically, dirsrv.service was failing to start due to missing files or directories:

```bash
 journalctl -xe
...
Mar 28 19:25:26 ip01 systemd-tmpfiles[206]: Failed to create directory /var/lock/dirsrv: No such file or directory
Mar 28 19:25:26 ipa01 systemd-tmpfiles[206]: Failed to create directory /var/lock/dirsrv/slapd-HUGEL-LAN: No such file or directory
...

 journalctl -fu systemd-tmpfiles

Mar 28 19:24:19 ip01 systemd-tmpfiles[47]: Failed to create directory or subvolume "/var/lock/dirsrv": No such file or directory
Mar 28 19:24:19 ip01 systemd-tmpfiles[47]: Failed to create directory or subvolume "/var/lock/dirsrv/slapd-HUGEL-LAN": No such file or directory

```

Dam, these are temp directories and files created at start-up, not as easy as creating the directory and calling it a day. Anyway, after learning more about 389ds and systemd-tmpfiles than I ever really wanted to, this appears to be some kind of bug that raised its head seven years ago and now others seem to have the same issue now, again, with CentOS 8. Not sure if it is related to running in a LXC container....but it is known.

As some internetian put it, "this is some strange race condition on boot for systemd-tmpfiles when traversing the symlinks (perhaps the double symlink for /var/lock as opposed to the single for /var/run) to the final destination to create the directories."

Manually creating the temporary directory and files using the systemd-tmpfiles command proves this and manually starting dirsrv.service works or just manually starting ipa.service which starts all the dependent services WORKS as expected.

```bash
 systemd-tmpfiles --create

 systemctl start ipa.service
(or #ipactl start)
 ipactl status
```

systemd-tmpfiles --create will complain about some /var/lock vs some /var/run entries in dirsrv-HUGEL-LAN.conf. Make the changes or ignore it, it made no difference as ipa.service still failed to start after a reboot. Everything is now running....though not automagically at startup...but we now know that the startup of 389ds (dirsrv) and slapd is KAPUT!......now to unKUPT

In a bid to look more into 389ds, I looked at what packages CentOS had that could help me.

```bash
 rpm -qa | grep 389
python3-lib389-1.4.3.8-6.module_el8.3.0+604+ab7bf9cc.noarch
**389-ds-base-legacy-tools-1.4.3.8-6.module\_el8.3.0+604+ab7bf9cc.x86\_64**
389-ds-base-libs-1.4.3.8-6.module_el8.3.0+604+ab7bf9cc.x86_64
389-ds-base-1.4.3.8-6.module_el8.3.0+604+ab7bf9cc.x86_64

 rpm -ql 389-ds-base-legacy-tools

/etc/dirsrv/config/template-initconfig

```

OMG! what is that file in that package....lets install it and have a look

```bash
 dnf install 389-ds-base-legacy-tools -y

 less /etc/dirsrv/config/template-initconfig

 #(contents of file)
 This file is sourced by dirsrv upon startup to set
 the default environment for a single specific directory
 server instances. To set defaults for all instances, edit
 the file in the same directory called dirsrv.

 These settings are used by the start-dirsrv and
 start-slapd scripts (as well as their associates stop
 and restart scripts). Do not edit them unless you know
 what you are doing.

# This file is in systemd EnvironmentFile format - see man systemd.exec

 # /etc/dirsrv/config/template-initconfig (END)
```

Really! Actually, I said WTF! The gerddam missing link for dirsrv service start-up. Let's reboot and see.

A couple of seconds later....check IPA services are running, check internet connectivity and resolution, bring up the management page from a workstation. Reboot and check again

```bash
 ipactl status
 ipa-healthcheck --output-type human
```

## IT'S ALIVE AND KICKING!!

and not broken.

Now that we have a working freeIPA install in a container, that survives reboots, and know the missing ingredients to success:

- -x command line switch for chronyd, and
- Install 389-ds-base-legacy-tools <--- THIS

We can now move onto creating replicated FreeIPA master....but in another post, I feel as though this has been epic enough.

Stage 2. Setting up MASTER/MASTER FreeIPA replica server in CentOS 8:

[[2021-04-01-freeipa-in-centos-8-lxc-container]]

As always....Beer and profit.
