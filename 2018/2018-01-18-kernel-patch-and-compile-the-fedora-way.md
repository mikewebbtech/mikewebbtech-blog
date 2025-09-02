---
title: Kernel patch and compile the Fedora way
date: 2018-01-18T21:15:04
summary: Each distribution has a preferred way which tends to result in a nice reusable packaged compatible to specific package management system. For this scenario I'll focus on Fedora 27
draft: false
categories:
  - home-lab
tags:
  - linux
  - fedora
  - kernel
  - opensource
series:
---

Each distribution has a preferred way which tends to result in a nice reusable packaged compatible to specific package management system.  For this scenario I'll focus on Fedora 27 (I don't know elements of this will work with Centos 7, I'll confirm next time I run up a Centos VM) and:

a) Patch using the ACS patch commonly used to fix IOMMU grouping for many motherboards so PCIe devices can be passed through to VM's.

b) Compile the customised kernel using an official fedora kernel src.rpm resulting in reusable and manageable rpm packages complete with official fedora signed modules and additional errata patches (hello meltdown and Intel microcode)

I've compiled this compendium mainly for my own use from some confusing or outdated methods that had varying degrees of success or frustration (i.e. [fedoraproject)](https://fedoraproject.org/wiki/Building_a_custom_kernel#Building_Vanilla_upstream_kernel)and from an assortment of other blogs and forum posts  So this is neither definitive or for all use cases, but it works for me

**STEP ONE: Gather the tools and setup the environment:**

```
$ sudo dnf update
$ sudo dnf install -y rpmdevtools numactl-devel pesign
$ sudo dnf groupinstall "Development Tools"
$ sudo dnf build-dep kernel
```

> [!NOTE]
>  To see what packages get installed with a ***"groupinstall"*** command, just do:
> ```
> $ sudo dnf group info "Development Tools"
> ```

Now create the RPM build environment witrh

```
$ rpmdev-setuptree
```

This will create a directory called rpmbuild in your $HOME root and an accompanying sub directory filesystem structure. Move into the rpmbuild/SOURCES directory and download the official Fedora kernel source and unpack it. The default is to download the kernel source for the currently running kernel.

```
$ cd ~/rpmbuild/SOURCES
$ sudo dnf download --source kernel
$ rpm2cpio kernel-* | cpio -i --make-directories
$ mv kernel-*.src.rpm ../SRPMS
```

do a `$ ls ~/rpmbuild/SOURCES` and the output will show a multitude of new files and directories. You can see all the *.patch files that get compiled in to the official Fedora kernel

**STEP TWO (optional): Patch the kernel.**

If applying a patch to your kernel, copy the patch file to ~/rpmbuild/SOURCE (giving it a suffex of .patch is not mandatory, just a good practice for easy identification). In this case I will copy over the add-acs-overrides.patch downloaded from [github](https://github.com/f4bio/linux-c0mbine/blob/master/add-acs-overrides.patch) and move the <u>kernel.spec</u> file from **~/rpmbuildd/SOURCES** to **~/rpmbuild/SPECS** in preperation for editing.

** (This patch is reported to no longer work with newer versions i.e. fedora 29 up. use [THIS ONE](https://aur.archlinux.org/cgit/aur.git/plain/add-acs-overrides.patch?h=linux-vfio) instead)

Now change directory to the ~/rpmbuild/SPECS directory and edit the .kernel.spec file.

```
$ cp ~/Downloads/add-acs-overrides.patch ~/rpmbuild/SOURCES
$ mv ~/rpmbuild/SOURCES/kernel.spec ~/rpmbuild/SPECS
$ cd ~/rpmbuild/SPECS
$ vi kernel.spec
```

change
`# define buildid .local`
to
`%define buildid ACS.local`

If patching, declare the patch in the same kernel.spec file. Search for 
\# END OF PATCH DEFINITIONS
and just above that line, add: `PATCH900: add-acs-overrides.patch`

**STEP THREE: Build and install time.**

Use rpmbuild command to compile the kernel using the edited kernel.spec file and then build the .rpm packages.  This will take some considerable time to complete with an equal considerable amount of output.

```
$ rpmbuild -ba ~/rpmbuild/SPECS/kernel.spec`
```

If you don't require specialty kernels (e.g.pae) or debug symbols, you can save considerable compile time with

```
$ rpmbuild -ba --without debug --without doc --without perf \
--without tools --without debuginfo --without kdump \
--without bootwrapper --without cross\_headers ~/rpmbuild/SPECS/kernel.spec
```

The freshly built .rpm files can be found in ~/rpmbuild/RPMS/x86\_64/. a `$ ls ~/rpmbuild/RPMS/x86_64` will show that the rpmbuild -ba command builds more then just a standard kernel.

Now install the kernel .rpm files that you require. At a minimum do:

```
$ cd ../RPMS/x86\_64
$ sudo dnf install -y kernel-core*.rpm kernel-devel*.rpm kernel-headers*.rpm kernel-modules*.rpm
```

I've used this technique a few times now testing patches and kernel features with sucess, so I've not had the opportunity of resolving errors that aren't related to what I testing

 **FINAL STEP: Reboot and use.**

After backing up important files, or varifing existing backup strategy. to start using the new kernel install, reboot the machine and select the new kernel version or ensure it is the default kernel to load at boot.

`$ sudo reboot`

Beer and profit.