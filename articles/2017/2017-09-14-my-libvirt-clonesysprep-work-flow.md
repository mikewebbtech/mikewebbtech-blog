---
title: My libVirt clone/sysprep work flow
date: 2017-09-14T09:21:42
summary: An insight in to my workflow for creating KVM, LibVirt virtual machines on my Linux Workstation. Using KVM for my virtual environment makes sense, it is kernel level native, very powerful, flexible, well documented and community supported.
draft: false
categories:
  - virtualisation
  - home-lab
tags:
  - linux
  - libvirt
  - kvm
  - solution
series:
---
# Life and times of LibVirt
My workstation runs linux.  Using KVM for my virtual environment made sense, it is kernel level native, very powerful, flexible, well documented and community supported, comes with a plethora of tools to work with. I've opted to use libVirt, think of it as a management layer on top of QEMU.  It does introduce it's own issues compared to work just with QEMU but it brings an amazing amount of benefits to the party.

OK now!...Flow.
1. Create a master image, install common services, accounts and apply security updates and bug fixes.
2. Clone master image.
3. Prep clone into a new running VM.


### 1.  Creation
There are many ways to do this.  Having first started creating KVM guests using command line QEMU, I can attest that LibVirt and applications from the Virt Tools community make life easy can provide a lot of automation, check them out <http://www.virt-tools.org/>

### 2. Cloning
Cloning a VM guest is a straight forward with the aid of libVirt.  From the command line is the quickest way but the Virtual Machine Manager app can be very effective as well.  First up, there is no online cloning, but that's OK as a master image should not be running anyway.  But let's assume it is running and called LTS-Master, as well as a second disk attached, so you can see that procedure

From the command line:
```
## su to superuser shell or run all commands with sudo.  Some linux 
## distrobutions have a libvirt group that you can add your user to.

$ su -

# virsh list
( or 'virsh list --all' to see all the VM's, know as domains, not just the running ones)

# virsh list --all
 Id   Name             State
----------------------------------------------------
 9    VyOS\_Switch      running
 13   ubuntu-cl1       running
 15   LTS-Master       running
 -    centos-cl1       shut off
```
Suspend the master virtual machine image

```
# virsh suspend LTS-Master
```

Clone the master vm image, a disk image needs to be created for the clone.  This example also attaches an additional disk image to the virtual machine

```
 # virt-clone --connect qemu:///system \
 --original LTS-Master \
 --name LTS-cl1 \
 --file /var/lib/libvirt/images/LTS-cl1.qcow2 \
 --file /var/lib/libvirt/images/LTS-cl1-1.qcow2
(lots of out put)

Allocating 'LTS-cl1.qcow2' | 5.0 GB 00:00:02 
Allocating 'LTS01-cl1-1.qcow2' | 2.0 GB 00:00:00

Clone 'LTS01' created successfull
```

There that's it, a new clone is born called LTS-cl1 and it is in a non running state (shut off).

> [!NOTE]
>   the --connect qemu:///system switch gives a peek into some of the power of this tool.  we can clone KVM (qemu) guests on the local system or we can connect to a remote system.  It doesn't have to be KVM guests, we could have defined a lxc or lxd conatiners or a xen guest.  We could even clone guests across hypervisors.

### 3. Decontextualise the image
The clones created from the master image are exactly that.  If we spin it up, it will be exactly like the master image.  Spin a couple of these clones up at the same time, there will be issues and a lot of manual intervention and configuration changes to make.  We need a way to make each cloned virtual machine a unique install and we need a way to automate it.  Yep! libVirt to the rescue.

The virt-sysprep tool is a command line tool with a huge amount of functionality, have a read up on it over at libguestfs.org. Anyone who's done a windows rollout would be familiar with what sysprep can do and is used for, same deal here. As pointed out over at libguestfs, virt-sysprep is not for windows guests.....just yet.

A very simple example that would do a very reasonable job using the defaults would be

```
[root@VMHOST~]# virt-sysprep -d LTS-cl1
[ 0.0] Examining the guest ...
(lots of out put)
[ 171.0] Performing "abrt-data" ...
[ 171.0] Performing "bash-history" ...
[ 171.0] Performing "blkid-tab" ...


[ 172.0] Setting a random seed
[ 173.0] Performing "lvm-uuids" ...
[root@VMHOST ~]#

```

To peek into the default operations applied, run:

```
[root@VMHOST~]# virt-sysprep –list-operations
```

**To quote libguestfs.org **
***"The first field is the operation name, which can be supplied to --enable. The second field is a * character if the operation is enabled by default or blank if not. Subsequent fields on the same line are the description of the operation."***

A common example of how I use virt-sysprep is

```
[root@VMHOST~]# virt-sysprep -d LTS-cl1 \
--append-line "/etc/hosts:10.0.0.2 SRV01" \
--verbose \
--keep-user-accounts=mike \
--timezone "Australia/West" \
--network \
--update \
--operations bash-history,lvm-uuids,user-account \
**--dry-run**
```

> [!Note]
>  the --dry-run. Very important, when first running a new virt-sysprep command. add --dry-run to it. It will *"perform a read-only "dry run" on the guest. This runs the sysprep operation, but throws away any changes to the disk at the end.".* It gives an oppertunity to review the output of the command for errors etc.  When you are happy with the operation, remove --dry-run and run the command again and your cloned image is ready run.

```
[root@VMHOST~]# virsh start LTS-cli
[root@VMHOST~] visrh list
Id         Name            State
----------------------------------------------------
 9          LTS-cl1        running
 13        LTS-Master     shut off
```

**Yay! beer and profit.**