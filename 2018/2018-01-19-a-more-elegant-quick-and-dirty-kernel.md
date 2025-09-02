---
title: "A more elegant: Quick and dirty kernel"
date: 2018-01-19T09:28:51
summary: As we have seen, the original/traditional/generic way to install new kernels or patch existing kernels is a relatively straight forward affair but requires manual management.  Let's use git to help
categories:
  - home-lab
draft: false
tags:
  - linux
  - kernel
  - git
  - opensource
series:
---

As we have seen, the original/traditional/generic way to install new kernels or patch existing kernels is a relatively straight forward affair but requires manual management in regards to downloading the latest source (ftp from the nearest mirror) or stepping through the successive patch versions to get to the lastest kernel release.

Since managing a project as massive as the kernel is beyond the means of manual intervention and control, versioning systems  such as git were born to keep development manageable.

We can employ git to help manage our local kernel versioning, patching

This assumes that the system is ready as a development machine and the following, as a minimum, are installed:

gcc ccache openssl-devel kernel-devel git

**STEP ONE: GIT IS ABOUT TREES AND BRANCHES. CHECK IT OUT.**

It all starts with the source and kernel.org is your go to trusted site for the latest. Change directories to you $HOME

```
$ cd
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
$ cd ~/linux
```

This will take awhile and give you access to the release candidate (.rc) kernels as well.

List the tag names to find the latest kernel version (i.e. the highest number, in this case v4.14 is the newest stable release)

```
$ git tag -l | more
```

Checkout the files for the branch, using the tag listed from the last command (i.e v4.14)

```
[~/linux]$ git checkout -b custom v4.14
```

This will switch to the new branch called custom, now we are all set to customise our kernel prior to compiling it.

**STEP TWO: CUSTOMISE AND COMPILE**

Using git to manage changes to our base image is great as it manages a history of our local and upstream merges that we view for information or leverage for roll back if there are any issues.

To apply a patch (in this case the famed add-acs-overrides.patch in my Download directory), simply:

```
[~/]$ cd ~/Downloads
[~/Downloads]$ wget https://raw.githubusercontent.com/mooteel/linux-mooteel/master/add-acs-overrides.patch
```

If applying a patch in mail format that you got it from a mailing list, use git am patch-name.eml.  Use git apply patch-name.patch if a standard patch file

```
[~/linux]$ git apply ~/Downloads/add-acs-overrides.patch
```

The standard work flow for customising the kernel still apply as well.

To use the existing configuration from the current running kernel, do:

```
[~/linux]$ cp /boot/config-`uname -r`* .config
```

To interactively make changes using a menu inerface, do:

```
[~/linx]$ make menuconfig
```

Which way you chose, be sure to define CONFIG\_LOCALVERSION= to something meaningful as this will be appended to the kernel name and atleast you now which one you booting to at the GRUB menu.

Once you have a .config file you are happy with.  'make' the kernel.  I use a -j option to speed it up by assigning all my hyperthreads to the job e.g.

```
[~/linux]$ make -j8
```

Now make the modules and install everything. On the systems I have used this method on, I've had success with:
`[~/linux]$ sudo make modules_install install`
**Clean up**

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

**STEP THREE: REBOOT AND USE**

After doing the normal backups and verifications, reboot and select the new kernel in GRUB menu (identified by your definition in CONFIG\_LOCALVERSION= .

```
$ sudo reboot
```

Beer and profit.