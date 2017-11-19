---
layout: post
title:  "Encrypt a new hard drive"
date:   2017-11-18 00:13:37
categories: crypto
---
Reminder on how to encrypt a fresh new drive before using it:

First use gparted to create GPT partition table and create an ext4 partition. Then encrypt it:

~~~bash
export disk='name of the disk'
cryptsetup -v --key-size 512 --hash sha512 --iter-time 10000 luksFormat /dev/sdc1
cryptsetup open /dev/sdc1 $disk
mkfs.ext4 /dev/mapper/$disk
mkdir -p /media/$disk
mount -t ext4 /dev/mapper/$disk /media/$disk
~~~
