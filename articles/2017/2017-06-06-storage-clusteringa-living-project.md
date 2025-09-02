---
title: Storage Clustering....a living project
date: 2017-06-06T18:46:10
summary: Exploring various storage clustering solutions as high available backend for virtual machine clusters.  What is out there, what fits the needs, is it open source and how hard will it be to implement it.
categories:
  - home-lab
  - virtualisation
draft: false
tags:
  - linux
  - storage
  - opensource
---

This is really just a place holder that I'll be updating while I examine the various was to skin the cat called storage clustering.  My main focus will be for High Availability and as a shared storage back end for virtual machine clusters.  Also the ability to roll as much myself for a minimal price point.

For the whole, I will mainly stick with linux solutions but I may dip into Solaris land. Mainly for a look see into the golden implementation of ZFS (which supports windows style ACL's) and it's implementation of COMSTAR (Common Multiprotocol SCSI Target) for when I examine ISCSI as a fabric.

On my list to expand upon further will be topics such as:
* RAID, ZFS, LVM abd cLVM, (vSAN), GFS2, GlusterFS
* Hyper converged infrastructure, oVirt and Red Hats offerings
* JBOD external enclosures, Fibre Channel, SAS (duel and single plained) and associated hardware, maybe infiniband.
* iSCSI (IET, LIO and SCST targets), ISER and SRP, NFS
* Napp-IT, Proxmox.
* The network fabric, topologies, 10G, 40G. SPF+, QSPF+, ethernet.
* Cluster managment, RDBD, pacemaker, corosync, fencing, STONITH.
* Authentication, Access and Accounting, monitoring, testing, validating, orchestration, backup, disaster recovery.
* Oh and the list goes on.


 