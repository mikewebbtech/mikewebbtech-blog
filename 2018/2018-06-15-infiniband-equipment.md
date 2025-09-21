---
title: eBay Score of the Century
date: 2018-06-15T03:09:38
summary: how about a smoking network I'm redoing my home and lab network with the goal to go 10Gb/s.  Let's up that and go 40Gb/s and while were are at it let's take it even further with 56Gb/s....with a twist. I found Infiniband hardware cheaper then ethernet and it can do both
draft: false
categories: -networking
  - home-lab
tags:
  - networking
  - storage
  - server
series:
---

TL;DR: Saved US$16000, smoking network

I'm redoing my home and lab network with the goal of achieving the ability of faster then 10Gb/s connections to and between the storage cluster and computational node cluster.  I was put off by the cost of going 10Gbe (ethernet).

What I found bazaar, but makes sense, is that it was way cheaper to start looking 40Gb/s and even 56Gb/s networking.  As most large enterprises and data centers move towards 100Gb/s and even 200Gb/s there is now a lot of second hand 40-56Gb/s networking equipment hitting the market for insanely cheap prices compared to even second hand 10Gb/s equipment (which is rare and coming from the SME or domestic user who still wants/sees inflated value).

There are some caveats that I'll go into in this post but some researched hunting and sometimes bag a big trophy.

My successful bag was a Mellanox [MSX6012F-BFS](https://store.mellanox.com/products/mellanox-msx6012f-2bfs-switchx-2-fdr-infiniband-1u-switch-12-qsfp-ports-2-power-supplies-ac-ppc460-short-depth-connector-airflow-out-rohs6.html?sku=MSX6012F-2BFS&gclid=CjwKCAjw2rjcBRBuEiwAheKeL4huzefYsBw3SvrUkZD0QZRD3fvGSY72qATk6ZVKdsimjJnE1fKIWRoCs5gQAvD_BwE)SwitchX-2 FDR InfiniBand 1U Switch 12 QSFP+ Ports 2 Power Supplies MSRP for around US$13K or US$6K refurbished, it also came full licensed (n additional US$4500).  I picked this up on eBay for around US$150, that's right $17000 worth of enterprise networking equipment for only $150...score.

I've put it through its paces and it all works, does 56G infiniband, 40G ethernet, does 56G RDMA.  I haven't looked at IPoIB, see notes below.

I was impressed with 10Gb/s transfers but I new the network was constraining my storage speed.  The cluster has SSD ZFS raid10, optane mirrors (storage, ZOL and L2ARC) and NVMe ZFS raid10 and SAS3 10K rust RAIDz2.  Very nice seeing this investment being able to stretch its legs without being limited by the network....and for such a cheap price.

**WHY NOT IPoIB**.
InfiniBand adapters ("HCAs") provide a couple of advanced features that can be used via the native "verbs" programming interface:

- Data transfers can be initiated directly from userspace to the hardware, bypassing the kernel and avoiding the overhead of a system call.
- The adapter can handle all of the network protocol of breaking a large message (even many megabytes) into packets, generating/handling ACKs, retransmitting lost packets, etc. without using any CPU on either the sender or receiver.
- IPoIB (IP-over-InfiniBand) is a protocol that defines how to send IP packets over IB; and for example Linux has an "ib_ipoib" driver that implements this protocol. This driver creates a network interface for each InfiniBand port on the system, which makes an HCA act like an ordinary NIC.

IPoIB does not make full use of the HCAs capabilities; network traffic goes through the normal IP stack (slow), which means a system call is required for every message and the host CPU must handle breaking data up into packets, etc. However it does mean that applications that use normal IP sockets will work on top of the full speed of the IB link (although the CPU will probably not be able to run the IP stack fast enough to use a 56Gb/s FDR14 IB link).

Since IPoIB provides a normal IP NIC interface, it can run TCP (or UDP which is faster by magnitude but "unreliable" for storage fabrics) sockets on top of it.  TCP throughput well over 10 Gb/sec is possible using fast CPUs, but this will burn a fair amount of its cycles/power.

The real difference is between using IPoIB with a normal sockets application versus using native InfiniBand with an application that has been coded directly to the native IB verbs interface. The native application will almost certainly get much higher throughput and lower latency, while spending less CPU on networking.

RDMA can provide network connectivity to normal sockets (TCP/IP) applications (e.g. NFS, ISCSI, SMB) without any special versions or recoding and bypassing the standard network application stack, minimising the hit on the system and CPU, just like IB does.

WIN WIN....beer and profit.
