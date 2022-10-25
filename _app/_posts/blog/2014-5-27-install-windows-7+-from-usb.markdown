---
layout: post
title: Booting Windows 7+ from USB
description: A few lines to make a bootable Windows USB stick
category: blog
tags: oneliner
---
Everytime I try to install Windows from a USB stick, I forget a simple step
in making the stick bootable and another few minutes are wasted searching for the solution.
This is just a short note on how to get your USB-Stick set up for installing Windows from it.

Read these few lines to save yourself some time:

```bash
fdisk /dev/sdb
# create a single new partition taking all the space on the stick
# make it of type 7 (Windows/NTFS) and set its bootable flag
mkfs.ntfs -f /dev/sdb1
ms-sys -7 /dev/sdb
mkdir /tmp/winiso
mount -o loop winimage.iso /tmp/winiso
mount /dev/sdb1 /mnt/usb1
cp -rav /tmp/winiso/* /mnt/usb1
```

In case you do not have *ms-sys* installed, you can get it from [here](http://ms-sys.sourceforge.net/).
