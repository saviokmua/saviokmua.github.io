+++ 
date = '2023-05-31' 
lastmod = '2023-05-31'
draft = false
title = 'How to migrate LXC(.raw) to KVM(.qcow2)'
authors = ['Alex']
enableReadingTime = true
description = "Learn how to migrate containers from LXC (.raw disk image) to KVM (.qcow2 format) with a step-by-step guide. Convert, optimize, and run your virtual machines seamlessly."
tags = ['kvm', 'virtualization', 'linux']
categories = ['Development infrastructure']
featuredImage = ''
+++

To migrate your LXC virtual server to a KVM just follow these steps:

For example, we have `image.raw` based on Ubuntu 18.04.6
We installed a KVM machine with the same operating system, 
in my case it was KVM server with ssh access to 192.168.1.1

1) mount image.raw like next:

```bash
mount ./image.raw /mnt/root
```

3) sync the LXC to the new KVM server

```bash
rsync --exclude=/etc/fstab --exclude=/boot --exclude=/proc --exclude=/lib/modules/ --exclude=/etc/udev --exclude=/lib/udev --exclude=/sys -e ssh --delete --numeric-ids -avpogtStlHz /mnt/root/ root@192.168.1.1:/
```


