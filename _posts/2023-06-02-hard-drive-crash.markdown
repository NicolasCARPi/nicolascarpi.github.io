---
layout: post
title:  "Hard drive crash and reinstall"
date:   2023-06-02 13:37:00
categories: os archlinux
---

# Introduction

So here I was, playing Snowrunner on my Linux system, when the image froze. My OS drive just wasn't working anymore.

But kind of in a weird way, it is a NVME drive, and for some reason it cannot be booted on anymore, but it's fine having the root partition mounted, go figure...

After buying a new NVME drive (bigger, faster), this blog post is basically a log of what I did to reinstall the system. It is mainly destined to my future self (hopefully not too soon), but it might be interesting to others.

I'm not going to try and be generic in the instructions, so here's that.

# Installing the OS

## Getting started

After "burning" the latest ISO on a USB key, press F11 to select the USB key as the boot target.

Select: Archlinux installation (UEFI)

~~~bash
loadkeys fr-bepo
timedatectl set-timezone Europe/Paris
~~~

## Partitioning the disks

~~~bash
# list disks
fdisk -l
# we will install on nvme0n1
# to check if "Best" is in use
nvme id-ns -H /dev/nvme0n1|grep "Relative Performance"
# start getting real
fdisk /dev/nvme0n1
# create GPT table
g
~~~

### Partitions

##### 1: EFI System (type 1) 1 Gb

n
+1G
t
1 EFI System

#### 2: SWAP (type 19) 16 Gb

n
+16G
t
19 (swap)

#### 3: /boot (type 23) 1 Gb

n
+1G
t
23 linux root x86_64

#### 4: / (type 23) tout ce qui reste

n
t
23 linux root x86_64

#### Summary

p1 EFI
p2 swap
p3 /boot
p4 /

#### Formatting

~~~
# EFI is FAT32
mkfs.fat -F 32 /dev/nvme0n1p1
mkswap /dev/nvme0n1p2
# /boot is ext4
mkfs.ext4 /dev/nvme0n1p3

# encrypt root partition
cryptsetup -y -v luksFormat /dev/nvme0n1p4
cryptsetup open /dev/nvme0n1p4 root
mkfs.ext4 /dev/mapper/root
~~~

## Mount new partitions

~~~
mount /dev/mapper/root /mnt

# mount /boot too
mount --mkdir /dev/nvme0n1p3 /mnt/boot
# efi is mounted at /efi
mount --mkdir /dev/nvme0n1p1 /mnt/efi
# activate swap
swapon /dev/nvme0n1p2
~~~

## Start installation

~~~
# select best mirrors
reflector -latest 10 --country Netherlands,Germany,France --sort rate --save /etc/pacman.d/mirrorlist
# install base system
pacstrap -K /mnt base linux linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
~~~

Follow https://wiki.archlinux.org/title/Installation_guide.


vconsole.conf:

KEYMAP=fr-bepo
XKBLAYOUT=fr
XKBMODEL=bepo

mkinitcpio: edit the .conf to add encrypt hook

install grub and efibootmgr:

`grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB`

Install amd-ucode and regenerate grub config:

~~~
pacman -S amd-ucode
grub-mkconfig -o /boot/grub/grub.cfg
~~~

use blkid for the device uuid

# Internet

Last important step of the install:

~~~bash
pacman -S dhcpcd
systemctl enable dhcpcd
~~~

# Set password for root account

~~~bash
passwd
~~~

# Reboot

We're all done, now we reboot and hope everything is good!

# Pacman

In `/etc/pacman.conf` add `ILoveCandy` and set `ParallelDownloads` to `5`.

# Essential packages

~~~
pacman -S git vim zsh tmux nodejs yarn
git clone --recursive https://github.com/NicolasCARPi/.dotfiles
bash .dotfiles/install.sh
# install vim plugins
vim
:PlugInstall
~~~


# File manager

~~~
pacman -S pcmanfm gvfs
~~~

In preferences:

General: check Open files with single click, View Mode: Compact View
Display: Always show full file names
Layout: uncheck Desktop and Applications
Advanced: Terminal emulator: urxvt

# Screens management

~~~
pacman -S arandr
~~~

# Sudo

Instead of using `sudo`, we install `doas`: `pacman -S doas`

With `/etc/doas.conf`:

~~~
permit persist :wheel
~~~

And set restrictive permissions:

~~~
doas chmod 400 /etc/doas.conf
~~~

# Setting the timezone and hostname

This one is easy and straightforward:

~~~bash
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc
hostnamectl hostname ryzen
~~~

# Hosts

Content of `/etc/hosts`:

~~~
127.0.0.1 elab.local
::1 elab.local
~~~

# Locales

Let's generate the locales:

~~~bash
# use vi as vim is not yet installed ;)
vim /etc/locale.gen
# uncomment en_US.UTF-8
locale-gen
~~~

# Sound

For sound we use pipewire with wireplumber and pulseaudio, because sound on Linux is very simple and straightforward...

~~~bash
pacman -S pipewire wireplumber pulseaudio pavucontrol
systemctl --user --now enable wireplumber
~~~

In pavucontrol, disable the HDMI output.

# Music

~~~bash
pacman -S mpd mpc ncmpcpp
ln -s /mnt/data/var/music/lib ~/.music
~~~

# Main apps

~~~
pacman -S man mesa xf86-video-amdgpu vulkan-mesa-layers vulkan-radeon xorg-xinit xcompmgr unclutter rxvt-unicode awesome firefox chromium gnupg mutt ruby sshfs conky zip unzip unrar p7zip xarchiver pinentry lxappearance w3m docker docker-compose filezilla pass python-pip borg xclip make gcc curl
~~~

Content of `.xinitrc`:

~~~
xcompmgr -c &
mpd
gpg-agent --daemon
unclutter -b --timeout 30
conky
exec awesome
~~~

# Theming

In `~/.themes` extract https://www.gnome-look.org/p/1934110 and use `lxappearance` to set it.

# Fonts

~~~
pacman -S ttf-dejavu ttf-freefont adobe-source-code-pro-fonts freetype2 noto-fonts noto-fonts-extra noto-fonts-cjk
~~~

# Miscellaneous packages

~~~bash
gem install tmuxinator jekyll
pacman -S vlc libreoffice-fresh gpicview evince htop scrot gimp baobab filezilla php-cli picard cmatrix rsync
~~~

Now try `startx`!
