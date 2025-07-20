---
title: "Running Debian on a Router"
description: "Use Debootstrap to generate a Debian image and run it on your router using chroot."
date: 2017-01-30T23:08:05+08:00
categories:
  - Tinkering
tags:
  - Linux
  - OpenWRT
---

First, you need to know what [Debootstrap](https://wiki.debian.org/Debootstrap/) is.

Then read the first section of this document: [EmDebianCrossDebootstrap](https://wiki.debian.org/EmDebian/CrossDebootstrap).

You’ll see that there are several ways to achieve our goal. **My router’s CPU is an MT7621, which uses the mipsel architecture.**

I used the method in the second section: [QEMU/debootstrap approach](https://wiki.debian.org/EmDebian/CrossDebootstrap#QEMU.2Fdebootstrap_approach).

My host system is Ubuntu.

Following the instructions, install the required packages:

```shell
apt-get install binfmt-support qemu qemu-user-static debootstrap
````

Then create a directory, for example:

```shell
mkdir mipsel_debian
```

Now run the bootstrap command. **Note that the architecture should be changed. After that comes the Debian release you want (e.g., Jessie), then the target directory you just created, and finally the [Debian mirror](http://www.debian.org/mirror/list).**

```shell
debootstrap --foreign --arch mipsel jessie mipsel_debian http://ftp.cn.debian.org/debian/
```

Once that's done, continue following the official instructions. Some steps need to be adapted.

```shell
cp /usr/bin/qemu-mipsel-static mipsel_debian/usr/bin
DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C \
chroot mipsel_debian /debootstrap/debootstrap --second-stage
DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C \
chroot mipsel_debian dpkg --configure -a
```

Now it's ready. You can copy the whole directory to a USB stick, plug it into your router, SSH into the router, and `chroot` into the Debian environment.

Before using `chroot`, you need to mount some filesystems. Refer to [Debootstrap](https://wiki.debian.org/Debootstrap/) for details:

```shell
mount /dev mipsel_debian/dev
mount /sys mipsel_debian/sys
mount /proc mipsel_debian/proc
cp /proc/mounts mipsel_debian/etc/mtab
```

Finally, chroot into Debian:

```shell
chroot mipsel_debian /bin/bash
```

If you plan to use SSH, you might also need to do the following before `chroot`:

```shell
mount /dev/pts mipsel_debian/dev/pts
```
