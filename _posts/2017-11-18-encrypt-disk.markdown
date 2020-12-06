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
gdisk $device
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
# if it's a data disk, create a keyfile to avoid typing passphrase on boot
# -E to keep env vars
sudo -E su
cd
# create keyfile
dd if=/dev/random bs=32 count=4 of=$disk.keyfile
# set restrictive permissions on it
chmod 400 data.keyfile
# add the key
cryptsetup luksAddKey ${device}1 /root/$disk.keyfile
~~~

Check the UUID of the LUKS partition with `lsblk -f`.

Now edit `/etc/crypttab` and add a line similar to this:

~~~conf
data UUID=ca2e2e22-3a43… /root/data.keyfile luks,timeout=120
~~~

And it `/etc/fstab`:

~~~conf
/dev/mapper/data /mnt/data ext4 defaults,errors=remount-ro 0 2
~~~
