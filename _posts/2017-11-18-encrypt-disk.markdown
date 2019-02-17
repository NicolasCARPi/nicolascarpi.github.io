---
layout: post
title:  "Encrypt a new hard drive"
date:   2017-11-18 00:13:37
categories: crypto
---
Reminder on how to encrypt a fresh new drive before using it:

~~~bash
# config
export disk='data'
export device='/dev/sdb'

# partition the disk
gdisk
o
n
partition code 8309 (Linux LUKS)
w

# create luks partition
cryptsetup -v --key-size 512 --hash sha512 --iter-time 10000 luksFormat ${device}1
# open it
cryptsetup open ${device}1 $disk
# format it
mkfs.ext4 /dev/mapper/$disk
# mount it
mkdir /mnt/$disk
mount -t ext4 /dev/mapper/$disk /mnt/$disk
~~~
