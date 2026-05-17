---
date: 2026-04-05T00:00:00+08:00
draft: false
title: The Evolution of Virtualisation Storage
summary: I wanted to explore how Proxmox VE and ZFS represent a very different approach to virtualisation storage compared to traditional KVM stacks built around QCOW2, LVM, and XFS. Here I look at how enterprise virtualisation evolved, why ZFS changed the storage model, and the tradeoffs between software defined storage and traditional layered infrastructure. I find Proxmox and ZFS interesting as they bring enterprise style storage features to homelabs and SMB environments without expensive hardware.
tags:
  - storage
  - proxmox
categories:
  - home-lab
  - toolage
series:
---
## The Evolution of Virtualisation Storage

Modern virtualisation platforms are no longer just hypervisors.  
Increasingly, they are becoming complete software-defined infrastructure stacks where compute, networking, and storage are deeply integrated.

This is one of the interesting (to me) architectural differences between traditional KVM virtualisation environments and Proxmox VE.

While many KVM-based hypervisors historically relied on filesystems such as XFS or EXT4 layered on top of LVM with QCOW2 virtual disks, Proxmox VE strongly embraces ZFS and ZVOL-backed virtual machines.

At first glance this may appear to be a simple storage preference.

In reality, it reflects two very different philosophies of virtualisation infrastructure design.

## The Traditional KVM Storage Stack

Historically, most KVM virtualisation environments followed a layered storage model similar to traditional enterprise infrastructure.

A typical deployment looked something like this:

```text
Virtual Machine
 └── QCOW2 Disk Image
      └── XFS / EXT4 Filesystem
           └── LVM Thin Pool
                └── RAID Controller
                     └── Physical Disks
```

This architecture became extremely common across:
- Red Hat virtualisation

- oVirt   
- OpenStack
- Custom enterprise KVM deployments
- Large hosting providers

The design mirrored enterprise virtualisation practices established during the rise of SAN-based infrastructure and platforms such as:
- VMware ESXi
- Microsoft Hyper-V

Each layer had a dedicated responsibility.

|Layer|Responsibility|
|---|---|
|RAID Controller|Redundancy|
|LVM|Volume management|
|Filesystem|Data organization|
|QCOW2|Virtual disk features|
|Hypervisor|Compute virtualisation|

This separation of responsibilities created a modular and predictable architecture that worked well in enterprise environments built around dedicated storage arrays and hardware RAID controllers.

## Why QCOW2 Became Popular

QCOW2, the native QEMU copy-on-write disk format, became the standard virtual disk image format across many KVM environments because it added virtualisation-aware features on top of otherwise simple filesystems.

A QCOW2 disk is simply a file stored on a filesystem like XFS or EXT4.

Example:

```text
vm-100-disk.qcow2
```

Inside that file:
- Sparse allocation is supported
- Snapshots can be created
- Linked clones become possible
- Compression can be enabled
- Encryption may be supported

This allowed administrators to keep the underlying storage relatively simple while still benefiting from advanced virtualisation features.

QCOW2 also offered significant operational flexibility:
- Easy VM portability
- Simple file-based backups
- Straightforward migration workflows
- Broad interoperability across KVM ecosystems

In many environments, a VM are just a collection of files.

## The ZFS Philosophy

ZFS fundamentally changed the storage model.

Originally developed by Sun Microsystems and continued today through OpenZFS, ZFS was designed to eliminate many of the weaknesses caused by fragmented storage stacks.

Instead of separating RAID, volume management, and filesystems into different layers, ZFS combined them into a single integrated platform.

Traditional storage stack:

```text
RAID → LVM → Filesystem
```

ZFS storage stack:

```text
ZFS Pool
 ├── RAID
 ├── Volume Management
 ├── Filesystem
 ├── Compression
 ├── Snapshots
 ├── Checksums
 └── Replication
```

This integration dramatically changed how virtualisation storage could operate.

Rather than relying on the virtual disk format to provide advanced functionality, the storage platform itself became intelligent.

## Why Proxmox VE Embraced ZFS

Proxmox Virtual Environment adopted ZFS aggressively because it aligned perfectly with the platform’s broader philosophy:
- Open-source infrastructure
- Commodity hardware
- Software-defined storage
- Hyperconverged deployments
- SMB, NFS and homelab accessibility

Unlike traditional enterprise virtualisation platforms that assumed expensive SAN infrastructure, Proxmox enabled administrators to build resilient local storage directly into the hypervisor host itself.

> [!GOAL]
> This significantly reduced infrastructure complexity.

A Proxmox ZFS host can provide:
- Software RAID
- Snapshots
- Replication
- Compression
- Data integrity protection
- Thin provisioning

Without requiring:
- Dedicated RAID controllers
- Fibre Channel SANs
- Expensive shared storage appliances

This makes Proxmox particularly attractive for:
- Homelabs
- Small businesses
- Edge infrastructure
- Hyperconverged clusters
- Self-hosted environments

## ZVOLs vs QCOW2: Design Philosophies

A common misconception is that the debate between ZVOLs and QCOW2 is simply “block storage versus file storage.”

The reality is more architectural.

### QCOW2 Philosophy

With QCOW2:
- The virtual disk handles advanced functionality
- The filesystem underneath remains relatively simple
- VM features live at the hypervisor layer

```text
VM Features → QCOW2
Storage Features → Filesystem / RAID
```

### ZVOL Philosophy

With ZVOLs:
- The storage platform itself becomes intelligent
- Advanced functionality is provided by ZFS
- VM disks become simpler block devices   

```text
VM Features → ZFS Storage Layer
Virtual Disk → Raw Block Device
```

This is one of the most important conceptual differences between traditional KVM deployments and Proxmox ZFS environments.

## What Is a ZVOL?

A ZVOL is a virtual block device created directly from a ZFS pool.

Unlike QCOW2, which exists as a file inside a filesystem, a ZVOL behaves more like a raw disk exposed directly to the virtual machine.

Example:

```text
tank/vm-100-disk-0
```

In Proxmox VE, ZVOLs are commonly used as the default storage backend when a VM is placed on ZFS storage.

This allows Proxmox to leverage native ZFS functionality directly:
- Instant snapshots
- Efficient replication
- Compression
- Checksumming
- Copy-on-write behavior
- Thin provisioning

## Advantages of ZFS and ZVOL-Based virtualisation

### End-to-End Data Integrity

One of the biggest advantages of ZFS is its focus on data integrity.

Traditional filesystems often cannot detect silent corruption (bit rot).

ZFS continuously verifies data integrity using end-to-end checksums.

If redundancy exists within the pool, ZFS can often repair corrupted data automatically.

This is particularly valuable for:
- Databases
- Long-term archival storage
- Critical infrastructure VMs
- Backup repositories

### Native Snapshot and Replication Support

ZFS snapshots are extremely efficient because they operate at the filesystem level.

Combined with ZFS send/receive, Proxmox gains powerful native replication capabilities between cluster nodes.

This enables:
- Incremental replication
- Fast disaster recovery workflows
- Efficient offsite backups
- Storage-level VM migration

Without relying on third-party tooling.

### Compression
ZFS compression is often underestimated.

In virtualisation environments, many VM workloads contain highly compressible data:
- Log files
- Operating system files
- Zeroed blocks
- Text-heavy workloads

Compression can significantly improve effective storage density while often improving read performance due to reduced disk I/O.

### Simplified Infrastructure

ZFS reduces dependency on:
- Hardware RAID controllers
- External SAN appliances
- Complex layered storage stacks

This creates a more self-contained virtualisation platform.

## The Downsides of ZFS

ZFS is powerful and free, but it is not free from effort. The operational trade-offs are real.

### Higher RAM Usage
ZFS aggressively uses memory for ARC caching.

This improves performance but can become problematic in virtualisation hosts where:
- RAM is already heavily allocated to VMs
- Memory overcommitment exists
- Homelab hardware is limited

In smaller systems or where memory is at a premium, like in a hypervisor, ARC tuning may become necessary.

### Copy-on-Write Overhead

ZFS is a copy-on-write filesystem.

While this enables snapshots and integrity guarantees, it can also introduce:

- Fragmentation   
- Increased write amplification
- Higher SSD wear
- Reduced small-block random write performance

Especially on:
- Consumer SSDs
- Low-end SATA flash
- **Poorly tuned pools**

### Complexity

ZFS is often marketed as “easy,” but advanced tuning becomes increasingly important as environments scale.

Administrators may eventually need to understand:
- ashift alignment
- Record size tuning
- ARC sizing
- SLOG devices
- Special metadata vdevs
- Pool fragmentation
- Scrubbing strategy

This is considerably more complex than a basic XFS + LVM deployment.

## Why QCOW2 Still Matters

Despite the rise of ZFS, QCOW2 remains highly relevant.

In many environments, QCOW2 still provides an excellent balance of:
- Simplicity
- Flexibility
- Performance
- Portability

A QCOW2 VM is simply a file.

That makes:
- Backup workflows easier
- Manual recovery simpler
- VM portability straightforward
- Storage backend selection flexible

Many enterprise KVM deployments continue using:
- XFS
- EXT4
- LVM thin pools
- Raw images
- QCOW2 disks

Particularly in environments backed by:
- SAN infrastructure
- Shared storage arrays
- Enterprise NVMe platforms

## Traditional Enterprise KVM Architecture

An example of a more conventional enterprise KVM deployment might look like this:

```text
Compute Nodes
 └── KVM / QEMU
      └── QCOW2 Virtual Disks
           └── XFS Filesystem
                └── LVM Thin Pool
                     └── Fibre Channel / iSCSI SAN
```

|Advantages|Disadvantages|
|-|-|
|Mature enterprise tooling|Expensive infrastructure|
|Predictable scaling|Additional failure domains|
|Easier hardware replacement|Greater operational overhead|
|Shared storage abstraction|Dependency on external storage systems|
|Lower host storage complexity| Potential Longer DR RTO|

## The Shift Toward Software Defined Storage

Modern infrastructure is increasingly moving away from dedicated hardware storage appliances and toward software-defined storage systems.

Examples include:
- Ceph
- OpenZFS
- VMware vSAN

This trend reflects several industry shifts:
- Commodity hardware adoption
- Hyperconverged infrastructure
- Scale-out architectures
- Integrated replication and resilience
- Software-defined datacenters

Proxmox VE with ZFS fits naturally into this evolution.

It treats storage not as an external dependency beneath the hypervisor, but as an integrated component of the virtualisation platform itself.

## Where to from Here?

Choosing between QCOW2 and ZVOL backed virtualisation is not simply a performance decision.

It is a design philosophy decision.

Traditional KVM stacks emphasise:
- Layer separation
- Simplicity
- Portability
- Flexible infrastructure integration

Proxmox VE with ZFS emphasizes:
- Integrated storage intelligence
- Data integrity
- Software defined infrastructure
- Hyper converged simplicity

***Neither approach is universally superior.***

The best choice depends on:
- Infrastructure scale
- Operational priorities
- Performance requirements
- Hardware budget
- Administrative skillset

For many home lab and SMB environments, Proxmox VE and ZFS provide an incredibly powerful combination that previously required enterprise-grade infrastructure budgets.

This is why the platform has become so influential in not only home lab virtualisation environments, but also growing beyond serving production roles in small businesses and now in enterprise data centers.