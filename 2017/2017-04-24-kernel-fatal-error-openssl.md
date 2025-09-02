---
title: "Kernel fatal error: openssl/aes.h"
date: 2017-04-24T23:23:28
summary: Missing OpenSSL blows up kernel compile..Oh Why? So, I've walked into this one a few times when compiling a new kernel for the first time after a new distro install. Each time I shake my head.
draft: false
categories:
  - home-lab
tags:
  - linux
  - solution
  - ubuntu
  - redhat
  - kernel
---

So, I've walked into this one a few times when compiling a new kernel for the first time after a new distro install.  Each time I shake my head and can't believe I did again.

When compiling a new linux kernel and fails with the following error

```
fatal error: openssl/aes.h: No such file or directory
```

It means that it is trying to build in OpenSSL but the library and header files it is linking to can't be found, usually meaning they are not installed.

The fix is fairly straight forward, install or reinstall the OpenSSL development package.

Debian and derivatives like Ubuntu etc

```
$ sudo apt-get install libssl-dev
```


RHEL and derivatives like CentOS etc 


```
$ sudo yum install openssl-devel
```

 
Arch linux and the like do things a bit different than other distros for one there are no -devel or -dev the source IS IN THE PACKAGE so when you install OpenSSL with pacman (it is in the core repositiry), you get the development libraries and headers.

TL;DR: Install missing package, recompile, profit.