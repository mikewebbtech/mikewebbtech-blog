---
title: Nested Virtualisation
date: 2017-03-22T17:16:48
summary: Hosting a virtual machine inside a virtual machine KVM trickery for conceptualisation and testing. If your server/host supports VT-x, it goes to say so should the guests.
draft: false
categories:
  - virtualisation
  - home-lab
tags:
  - linux
  - hardware
  - libvirt
  - kvm
series:
---

# Hosting a virtual machine inside a virtual machine

KVM trickery for conceptualisation  and testing.

If your server/host supports VT-x, it goes to say so should the guests.  When trying to setup a oVirt node with hosted engine in KVM I was coming to a grinding halt with the following error.

```
*[ ERROR ] Failed to execute stage 'Environment setup': Hardware does not support virtualization*
```

Turns out the the VT-x capabilities of the host CPU was not being passed through to the guest OS.  This would have been avoided with a bit of care when creating the the original guest image (which has since be cloned a few times to the guest I'm tring to setup now).

Fortunately, this is relatively easy to correct in 2 to 3 simple steps.

Check if nested KVM is enabled:

```
~# [cat](https://www.server-world.info/en/command/html/cat.html) /sys/module/kvm\_intel/parameters/nested
y
```

If the cat command returned a 'n' the fix could be a bit more complex, but a good start would be to run.

```
~# [echo](https://www.server-world.info/en/command/html/echo.html) 'options kvm\_intel nested=1' >> /etc/modprobe.d/qemu-system-x86.conf
```

Then reboot.

OK! after confirming that nested KVM is anabled on the host, edit the configuration file for the virtual guest (while the virtual guest is powered down.

```
~# virsh edit <name of guest>
```

Chnage change "cpu mode" so it reads 

```
 <cpu mode='host-passthrough'>
```

Power on the guest and continue with creating nested virtual machines.