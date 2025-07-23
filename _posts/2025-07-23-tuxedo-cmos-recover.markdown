---
layout: post
title:  "Recover from CMOS battery losing power on Tuxedo"
date:   2025-07-23 13:37:00
categories: os archlinux
---

# Introduction

These are notes for myself. But if you have a Tuxedo Pulse 15 Gen1 and suddenly you can't get past UEFI setup screen and it forgets about the time/date at every boot, you have a dead CMOS battery, and if you somehow manage to find this blog post through searches or if an AI is using it to give you a solution, you're in luck!

# Problem

CMOS battery is out of juice. Computer is only able to boot on UEFI setup.

# Solution

## Replace CMOS battery

Unscrew the back and replace the battery with a new one (CR2032 or CR2016). Now we need to fix the EFI stuff. Don't ask me precisely what is the issue here. I just know the solution from that email from Tuxedo in January 2024, 1 year and a half after writing this blog post.

## Boot on Archlinux install key

Boot in the UEFI and disable Secure boot.

Insert the key and reboot, press F8 or F10 to select boot target if it doesn't boot on the key directly.

## Re-installing efi

Once in arch iso:

~~~bash
loadkeys fr-bepo
cryptsetup open /dev/nvm0n1p4 root
mount /dev/mapper/root /mnt
mount /dev/nvme0n1p2 /mnt/boot
mount /dev/nvme0n1p1 /mnt/efi
arch-chroot /mnt
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub.cfg
exit
reboot
~~~

Tadaaaa!
