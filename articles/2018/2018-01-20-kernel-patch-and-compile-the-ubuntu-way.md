---
title: Kernel patch and compile the Debian/Ubuntu way
date: 2018-01-20T23:31:53
summary: Continuing on with my theme of compiling and patching linux kernels.  Let's look at the Ubuntu way that creates a .deb file for use with  apt package manager.  Git work flow thrown in for free.
draft: false
categories:
  - home-lab
tags:
  - linux
  - kernel
  - ubuntu
  - opensource
---

This article continues on from  [Kernel patching and compile the fedora way](2018-01-19-a-more-elegant-quick-and-dirty-kernel)
Here is how I patch and compile new kernels for **Ubuntu/Debian** distributions. More specifically, create .deb files to be used by the Advanced Packaging Tool (apt)

### Prerequisites

make sure your system is updated and you have the required applications and libraries to compile a kernel.

```
$ sudo apt update
$ sudo apt install \
  libelf-dev
  bison \
  build-essential \
  ccache \
  curl \
  fakeroot \
  flex \
  git \
  kernel-package \
  libncurses5-dev \
  libssl-dev \
  wget -y
```

There are two main ways to get the kernel source.  One is to grab the ubuntu source using apt

```
$ sudo apt-get source linux-image-$(uname -r)
```

or using git cloning the tree that matches your repository name direct from Ubuntu

```
$ cd ~
$ ubuntu\_name=$(lsb\_release -s -c) \
  git clone git://kernel.ubuntu.com/ubuntu/ubuntu${ubuntu\_name}-.git
```

Then there is the other way that I like. I go straight to the source, kernel.org. This gives me access to new or release candidate versions.

You can download a tarball every time or keep that kernel and patch it to new versions as they are released (this saves on A LOT of bandwidth and time)

**OR**

As mentioned in other posts, use git, clone the official kernel tree and create a local branch every time you want to try a patch. This way I find best for me.

I'll quickly go the first steps, for a more descriptive walk through see: [A more elegant way to build a kernel](2018-01-19-a-more-elegant-quick-and-dirty-kernel.md)
```
$ cd
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
$ cd ~/linux
[~/linux]$ git tag -l | more
[~/linux]$ git checkout -b custom v4.10

```

*replace v4.10 with what ever the latest versions is or the kernel version that you are after.

### Applying a patch

If applying a  (I am applying the add-acs-overrides patch in mail format here).  Use  *git am* patch-name.eml if your patch is in email format (as in you got it from a mailing list, use *git apply patch-name.patch* if a standard patch file

```
[~/linux]$ git am ~/Downloads/add-acs-overrides.eml
```

### Prep for Compile

You need  a .config, it is quick and easy to copy an existing one over from your /boot directory that you are happy with or your latest one with

```
[~/linux]$ cp /boot/config-`uname -r`* ./.config
```

If you do this, be sure to run

```
[~/linux]$ yes '' | make oldconfig
```

Omit the yes ' ' | part and this will run you through the new options, prompting you for the usual y/n/m responses. You can now edit the .config file to customise/optimise your kernel or go straight to compiling and creating the deb files. While editing the .config file, I highly recommend setting **CONFIG\_DEBUG\_INFO=n** to stop the *dbg* package being generated (unless you have a use case for it) and **CONFIG\_MODULES=y**

For an easy and interactive method for editing the .config file. but requires ncurses is to type:
```
[~/linux]$ make menuconfig
```

and go through the trees and select away

### Compile and make packages.

Compiling and creating the .deb packages is a relatively straight forward one liner. But there are some important options and alternatives as well as caveats and work arounds. I will go through some here that fall within my workflow that I'm explaining here.

from this point on tools really need to be run as root. But you really really should compile as root [note. add reasons/links]. So we will use fakeroot.

Start with a clean source tree (this will preserve your existing .config file

```
[~/linux]$ fakeroot make-kpkg clean
```

Now compile and create .deb packages

```
[~/linux]$ CONCURRENCY\_LEVEL=`getconf \_NPROCESSORS\_ONLN` \
 MAKEFLAGS="HOSTCC=/usr/bin/gcc CCACHE\_PREFIX=distcc" \ 
 fakeroot make-kpkg --initrd \
 --append-to-version=-acs \
 --overlay-dir=$HOME/ubuntu-package \ 
 binary-arch
```

"**CONCURRENCY\_LEVEL"** does the same as the -j (--jobs) option for the gcc make command and sets the number cpu threads assigned for concurrent use for the compile.  In this case, 8 threads of a 4 core CPU.

**LOCALVERSION=** creates our custom entry to version suffix of the kernel name. This defines the final name for the .deb files while also helping distinguish this kernel version from others in the GRUB menu. e.g. Ubuntu 17.10, kernel 4.14.0.acs.custom.  call it what ever you like

The ***binary-arch***target is one of the faster build targets, it creates two .deb files called linux\_headers and the other,  linux\_image.  For more information on the other targets and the files they generate, read through the make-kpkg man page [[here]](https://manpages.debian.org/jessie/kernel-package/make-kpkg.1.en.html)
**Caveat:** If your kernel images are being built as "-dirty" on the end of the version string, and you used the git method to clone the source tree, as well as, patching or modified  the local branch.  This is the result of your modifications being committed.   You can ideally commit the changes to the tree HEAD or just simply create an empty .scmversion file in the root of the kernel sources i.e

```
[~/linux]$ touch .scmversion
```

### Install the files

The created .deb files can found in your $HOMEe directory and installed an example of installing them would be

```
[~/linux]$ sudo dpkg -i $HOME/linux-*.deb
```

### Clean up

If you want to keep your git branch called custom with the changes that you made but dont want to merge the changes back to the master (I recommend this, keep master pristine) , just stash it with

```
$ git stash
$ git checkout master
```

In reality we are usually done with the branch that we were working with, so just delete it with

```
$ git branch -D custom
```

'-D' as the branched hasn't been merged.

### Now reboot, select the kernel and test

Beer and profit.

here are the links to "official" methods on the Ubuntu wiki. I've never done it this way, your mileage may differ using these.
<https://wiki.ubuntu.com/KernelTeam/GitKernelBuild>
<https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel>