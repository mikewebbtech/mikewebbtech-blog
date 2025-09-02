---
title: FreeIPA Centos 8 lxc Replica Server
date: 2022-08-15T11:13:50
summary: FreeIPA is to linux as Active Directory is to Windows.  Like AD for Windows, FreeIPA should have a replica for high availability. This is a walk though on how I got it setup and running in CentOS 8 LXC container on Proxmox VE server.  Low resource demands compared to a dedicated vurtual machine.
draft: false
categories:
  - home-lab
  - virtualisation
tags:
  - linux
  - lxc
  - redhat
  - server
  - FreeIPA
series:
---

If you haven't followed through the steps in my previous post on setting up FreeIPA in a lxc container, I strongly recommend you give it once over as this will follow on from that:

## Let us begin

let’s go over some preliminary house keeping again.

My environment:

```
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

We are setting up ipa02 so create another container with the minimal requirements. Gain shell/console access into your container using either lxc console ipa01 or lxc exec ipa01 -- /bin/sh.

Do a yum update, setup and check ip address and internet connectivity. create a user with wheel group membership.

```bash
 useradd conop
 passwd conop
 usermod -aG wheel conop

 nmcli connection modify eth0 \
> ipv4.method manual \
> ipv4.address 192.168.1.35/24 \
> ipv4.gateway 192.168.1.5 \
> ipv4.dns 1.1.1.1

 nmcli connection down eth0 ; nmcli c up eth0
 ping 1.1.1.1
 ping redhat.com

 timedatectl set-timezone Austrlia/Perth
 timedatectl set-ntp true

 hostnamectl set-hostname ipa02.hugel.lan
 echo "192.168.1.35 ipa01.hugel.lan ipa01 \n192.168.1.36 ipa02.hugel.lan ipa02" >> /etc/hosts
 mv /etc/resolv.conf /etc/resolv.conf-BAK
 echo -e "search hugel.lan\nnameserver 192.168.1.5 \nnameserver 1.1.1.1" > /etc/resolv.conf

 dnf update
 dnf install openssh-server sudo less firewalld audit -y
 systemctl enable sshd --now
 systemctl status sshd
 systemctl enable firewalld --now
 systemctl status firewalld

```

Now is a good time to take a snaphot and reboot. SSH into your container as conop and su to root make sure all is as it should be.

## On Wards

There are few "gotcha's" that still apply to setting up a replica just as outlined previously with installing a primary. Also there are few ways set up replication as stated in RedHat and FreeIPA documentation. The method I chose to implement is "upgrade from a client" simply because if I can install a FreeIPA client then the bones required to migrate up to a replica must be working. Also, it is how I've done it in the past on Centos 7 (<https://gist.github.com/mikewebb70/c9722d443ac38e553490ee12d5386748> <- a good read but outdated for CentOS 8)

This is a 3 part process 
1. install Freeipa client and requirements on ipa02 container: 
2. deal with some dns and host group stuff on ipa02 container: 
3. Back on ipa02 container to migrate it up to a replica server.

### Part 1:

Add the -x fix for chronyd and restart the service, setup the firewall policy, install the AppStream module and install the client.

```text
# dnf module -y install idm:DL1/client

# ipa-client-install -v \
> --ntp-server=192.168.1.1 \
> --ntp-server=2.au.pool.ntp.org

# firewall-cmd --get-active-zone
# firewall-cmd --set-default-zone=internal
# firewall-cmd --zone=internal --list-services
# firewall-cmd --add-service={freeipa-ldap,freeipa-ldaps,freeipa-replication,dns,ntp} --permanent
# firewall-cmd --reload
# firewall-cmd --zone=internal --list-services

# kinit admin

```

### Part 2:

On ipa01, the first (and only) FreeIPA server we will double check that that it "knows" about ipa02, add ipa02 to the "ipaservers" host group and then fix up

```bash
 kinit
 ipa host-find ipa02
 ipa dnsrecord-find hugel.lan ipa02
 ipa dnsrecord-find 1.168.192.in-addr.arpa 36
#*(hmmm...no reverse lookup records, let's fix that manually)*

 ipa dnsrecord-add 1.168.192.in-addr.arpa 36 --ptr-rec ip02.hugel.lan
 ipa dnsrecord-find 1.168.192.in-addr.arpa 36
#*(yay!)*

 ipa hostgroup-add-member ipaservers --hosts ipa02.hugel.lan

 ipa hostgroup-show ipaservers
  Host-group: ipaservers
  Description: IPA server hosts
  Member hosts: ipa01.hugel.lan, ipa02.hugel.lan
```

### Part 3:

Now back onto ipa02 again for the throw down finishing move. Install the idm server module stream, run replica install command, do some verifications and reboot...ah, the reboot.

```bash
 dnf module install idm:DL1/{dns,adtrust} -y
 ipa-replica-conncheck --master ipa01.hugel.lan
 kinit admin

 ipa-replica-install -v \
> --setup-ca \
> --setup-dns
> --forwarder=192.168.1.5 \
> --forwarder=1.1.1.1

 reboot
```

After connecting back to the ip02 container and running some checks, you will see that nothing is working as in the post about setting up freeIPA in a container. Read through that post if you would like more insight into the issue. To fix the issue install the **389-ds-base-legacy-tools** and reboot and run some checks to see if all good in the hood

```bash
dnf install 389-ds-base-legacy-tools -y
 reboot
```

```bash
 kinit
 klist
 ipactl status

 ipa server-find
---------------------
2 IPA servers matched
---------------------
  Server name: ipa01.hugel.lan
  Min domain level: 1
  Max domain level: 1

  Server name: ipa02.hugel.lan
  Min domain level: 1
  Max domain level: 1
----------------------------
Number of entries returned 2
----------------------------

 ipa dnsconfig-show
---------------------------------
Global DNS configuration is empty
---------------------------------
IPA DNS servers: ipa01.hugel.lan, ipa02.hugel.lan

 ipa healtcheck --output-type human

 ipa-replica-manage list
ipa01.hugel.lan: master
ipa02.hugel.lan: master
```

AND...there you go. you now have two redundant freeIPA servers s well as two DNS servers (point your clients at both or add them to your dhcp server configuration).

In my deployment, these containers are just freeIPA servers only handling IDm, policy, dns, etc. No file serving or any other services for the network. Those services will be installed into other containers or virtual machines, but call on the freeIPA servers for authentication and access. I'll document those as I set them up and work through any "gotcha's"

As you may have worked out by now, freeIPA provides directory services for unix hosts/services only, it isn't a drop in active directory replacement for windows environments. That is best served using samba 4 (or a Windows Server). but is can be combined with, using trusts. Something I will do when providing smb file services to the network.

Since LXC containers use next to nothing resources when just sitting there doing nothing. maybe we should set up some as test clients to test out a few scenarios....but in another post.

**As always, beer and profit**
