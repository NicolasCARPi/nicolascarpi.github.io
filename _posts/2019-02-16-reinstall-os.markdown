---
layout: post
title:  "Reinstalling my ArchLinux from scratch (and all the troubles associated with that)"
date:   2019-02-16 13:37:00
categories: os archlinux
---

## Introduction
I got a new computer. So I had one task: transferring my existing years old install of Archlinux to this new computer. This would allow me to encrypt the root filesystem and get a fresh clean OS.

But doing this is not as easy as you might think. A lot of things can go wrong. Trying to reproduce a system by just copying a few config files over doesn't work always. Here is the story of the reinstall. I write this for me (in case I have to reinstall again), but I publish it because it might be useful to you, dear reader, as there is a lot of geekiness in the following post.

I'm writing this blog post while listening to the [full OST of Aladdin](https://www.youtube.com/watch?v=swNNkgUsjQk), the game from 1993. It reminds me when I was playing it in the supermarket when on holidays. Why isn't this a thing anymore? Free arcade games for the kids while the parents can go shopping, knowing their kid won't move an inch from the front of the arcade. I'm suggesting you open a tab and listen to it too ;)

## Setting the keymap and time

We have now booted on the live USB disk (in non UEFI mode (BIOS)). The first thing I do is setting the keymap:

~~~bash
loadkeys fr-bepo
# let's also set the correct time
timedatectl set-timezone Europe/Paris
~~~

## Partitioning

Depending on your setup and what you want, you might not want to follow exactly these steps.

~~~
gdisk /dev/sda
# create GPT
o
# make new partition of 1M
n
1
+1M
# the first partition is the BIOS partition, type BIOS
ef02
# make a new partition for /boot
n
2
# type Linux filesystem
8300
# the third partition is LVM
n
3
8e00
~~~

I'm skipping all the LVM stuff, just read the wiki.

## Installation

No suprise here, follow the wiki and use your brain. Correctly figuring out how to boot on the encrypted root partition took me a few tries.

I installed `base` and `base-devel` with `pacstrap` because for sure I'll be compiling stuff on this machine!

Now we have chroot in the new system, here is the last configuration steps. I'm looking at the `.bash_history` and `.histfile` to retrace my steps.

### The vital package

~~~bash
pacman -S vim
~~~

### Setting the timezone

This one is easy and straightforward:

~~~bash
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc
~~~

### Locales

Let's generate the locales:

~~~bash
# use vi as vim is not yet installed ;)
vim /etc/locale.gen
# uncomment en_US.UTF-8
locale-gen
~~~

### Setting the keymap in the console

I use the [bépo](https://bepo.fr/wiki/Accueil) keymapping (and you should too if you're french!), so of course I want to have it set in the console, here is the content of `/etc/vconsole.conf`:

`KEYMAP=fr-bepo`

### Setting the hostname

Generally this is where I spend most of my time when installing a new system. Finding a good hostname is hard. It should be short, pretty, and the first letter should not be one of any of your other hosts. I choose to keep the same one used at the time when this computer was running Windoze (for gaming only, don't worry, I'm not crazy): Titan.

As a bonus here is a list of names of some of my devices, in no particular order:

* osiris (dedicated server for internet services, ubuntu)
* sierra (VPS for my email, openbsd)
* titan (the computer of this post, arch)
* atlas (a very buffy computer to run VR, windoze)
* neon (my work computer, arche)
* spip (my work laptop, fedora)
* powerdidi (my first macbook, with a G4 processor (PPC), mac OS X)
* droid (my phone, android)
* pika (yes it's a Raspberry Pi)
* crystal (an old computer that I don't have anymore, used to run [BOINC](https://boinc.berkeley.edu/) on it)

### Set hosts

Add the hostname as localhost and also some name I have for some web projects:

~~~
127.0.0.1 localhost
::1 localhost
127.0.0.1 titan.local titan
::1 titan.local titan
127.0.0.1 elab.local
::1 elab.local
127.0.0.1 wonder.local
::1 wonder.local
127.0.0.1 justorderit.local
::1 justorderit.local
~~~

### Set password for root account

~~~bash
passwd
~~~

### Install the boot loader: GRUB

~~~bash
# install vim now
pacman -S grub intel-ucode
grub-install --target=i386-pc /dev/sda
# make sure /boot partition is mounted
vim /etc/default/grub
~~~

~~~conf
# file: /etc/default/grub
# showing only the line changed from default
# remove the quiet argument, I like to see what happens
GRUB_CMDLINE_LINUX_DEFAULT=""
# the line for booting on encrypted partition
# use 'blkid' and find one that looks like this:
# /dev/mapper/myvol-myroot: UUID="fb2b71ff-c632-45d7-adbb-6c1038479f1a" TYPE="crypto_LUKS"
GRUB_CMDLINE_LINUX="cryptdevice=UUID=fb2b71ff-c632-45d7-adbb-6c1038479f1a:root root=/dev/mapper/root"
# add lvm in the preload modules
GRUB_PRELOAD_MODULES="part_gpt part_msdos lvm"
# allow the kernel use the same resolution used by grub
GRUB_GFXPAYLOAD_LINUX=keep
# match grub theme with my desktop theme (green/black)
GRUB_COLOR_NORMAL="green/black"
GRUB_COLOR_HIGHLIGHT="white/blue"
# the wallpaper must be in the boot partition as root is not yet mounted!
GRUB_BACKGROUND="/boot/grub/grubwallpaper.png"
~~~

~~~bash
# now that everything is set, generate the config
grub-mkconfig -o /boot/grub/grub.cfg
~~~

Here is the wallpaper I use for GRUB:

![grub-wallpaper](/img/grubwallpaper.png)

Isn't it lovely? <3 It's also my main wallpaper at work. Hard to find out who made it though…

Feeling at home right from GRUB is great. :]

### Encrypted swap and /tmp

~~~
# file: /etc/crypttab
swap          /dev/myvol/swap                              /dev/urandom             swap,cipher=aes-xts-plain64,size=256
tmp          /dev/myvol/tmp                                /dev/urandom             tmp,cipher=aes-xts-plain64,size=256
~~~

## Getting started

First time (successfully) booting on the fresh new system. feelsgoodman.jpg.

### Oops no internet

Internet is not working!! Fortunately for us, fixing this is dead easy:

~~~bash
systemctl start dhcpcd
# enable it at boot
systemctl enable dhcpcd
~~~

### Adding more essential packages

Before doing that: add "ILoveCandy" somewhere in `/etc/pacman.conf`. There is no point in using ArchLinux if this option is not set. :p

These are the packages that I deem essential on any system:

~~~
# vim is already installed
pacman -S git tmux zsh
~~~

The rest of the packages will come later in their own section.

### Creating the user

Root is a nice guy, but this is not Kali Linux and we want a non root user:

~~~bash
# create user
useradd nico
# set password
passwd nico
# create home
mkdir /home/nico
chown nico:nico /home/nico
# add it to wheel group
usermod -g wheel ktr
# configure sudo to allow wheel group to execute commands as root
# there is a line to uncomment that looks like this: %wheel ALL=(ALL) ALL
visudo
# switch to new user
su nico
# install my dotfiles
git clone --recursive git@github.com:NicolasCARPi/.dotfiles.git
bash .dotfiles/install.sh
# change default shell
chsh -s /usr/bin/zsh
# and start using it right away
zsh
~~~

### Graphical interface

I can do a lot of things in the console as most of my favorite tools are text based (mutt, tmux, vim, git, ncmpcpp, …) but let's not be too nerdy and let's install a graphical interface with X11, our favorite crappy old unsafe software :)

~~~bash
# install the graphics drivers, xinit, compositing manager, notification daemon, sound, music, cursor hider, terminal emulator and WM
sudo pacman -S mesa xf86-video-amdgpu xorg-xinit xcompmgr twmnd pulseaudio mpd unclutter rxvt-unicode awesome pcmanfm firefox chromium
~~~

The content of my `~/.xinitrc` file:

~~~
# file: ~/.xinitrc
# switch keyboard in bépo
setxkbmap fr bepo

# transparency
xcompmgr -c &

# notification daemon
twmnd &

# sound
pulseaudio --start

# start music daemon
mpd

# start gpg-agent
gpg-agent --daemon

# fix the black cross pointer
xsetroot -cursor_name left_ptr

# hide the cursor if not used
unclutter -b --timeout 30

# start Awesome WM
exec awesome
~~~

Also install [tmuxinator](https://github.com/tmuxinator/tmuxinator) and [jekyll](https://jekyllrb.com/):

~~~bash
sudo pacman -S ruby
# note that I have the --user-install option in my ~/.gemrc
# so the gems are installed without root permissions
# I do the same for python and javascript packages
gem install tmuxinator jekyll
~~~

Now the moment that we're all waiting for: starting Xorg!

~~~bash
startx
~~~

Because all of my settings are nicely tracked in my [.dotfiles repository](https://github.com/NicolasCARPi/.dotfiles), all my custom shortcuts and everything are there. But some things are still missing of course!

### Filling the holes

I didn't want to import my full previous home, filled with the accumulated crap over the years. So I only imported some crucial folders from the old $HOME (mounted as an external drive):

* `~/.bin` a bunch of scripts to make my system behave EXACTLY like I want it to, this folder is in my $PATH
* `~/.ssh` with the keys and the config file
* `~/.opt` this is where I put software I download and use, but that can't be properly installed ([Fiji](https://fiji.sc/), [ZAP](https://github.com/zaproxy/zaproxy/))
* `~/.mozilla` it's great to find all your bookmarks, extensions and configuration without having to redo it from scratch
* `~/.filezilla` same as with firefox, one folder for all the user conf is great
* `~/.mutt` all my email config

#### Installing an [AUR](https://aur.archlinux.org/) helper ([YAY](https://github.com/Jguer/yay)):

~~~bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
~~~

#### Installing a dark gtk theme

This theme seems to be unmaintained now, need to find another one at some point...

~~~bash
yay -Sy vertex-themes-git vertex-icons-git
~~~

#### The GPG keys

I use GPG to encrypt and decrypt secrets on my computer, like passwords. Because nobody likes a password sitting in plaintext in a file somewhere…

~~~
# from the previous computer
gpg --list-secret-keys
# copy the ID
gpg --export-secret-keys $ID > my-key.asc
# copy the file over to the new computer
# and now import it
gpg --allow-secret-key-import --import my-key.asc
~~~

#### Vim config

See post on [Vim](/text/editor/cli/2014/09/22/vim.html).


#### S.M.A.R.T monitoring of the disks

~~~bash
sudo pacman -S smartmontools msmtp-mta
~~~

In `/etc/smartd.conf`:
DEVICESCAN -m you@example.com

Append "-M test" to test it once.

In `/etc/msmtprc`:
aliases               /etc/aliases

In `/root/.msmtprc`:

~~~conf
# Set default values for all following accounts.
defaults

# Use the mail submission port 587 instead of the SMTP port 25.
port 587

# Always use TLS.
tls on

# A freemail service
account smtp2go

# Host name of the SMTP server
host mail.smtp2go.com

# Envelope-from address
from smart@example.com

# Authentication. The password is given using one of five methods, see below.
auth on
user example-user
password secr3t

# Set a default account
account default : smtp2go
~~~

~~~bash
sudo systemctl start smartd
sudo systemctl enable smartd
~~~

#### Cronjobs

~~~bash
sudo pacman -S cronie
sudo systemctl start cronie
sudo systemctl enable cronie
~~~

#### mutt config

See post on [Mutt](/email/mutt/2016/02/28/mutt.html)

#### Fonts

It was difficult to find a font with the correct set of UTF-8 characters.

~~~bash
sudo pacman -S bdf-unifont ttf-dejavu ttf-gentium ttf-freefont adobe-source-code-pro-fonts freetype2 ttf-ms-fonts noto-fonts noto-fonts-extra noto-fonts-cjk terminus-fonts
~~~

#### Archiving

~~~bash
sudo pacman -S zip unzip unrar xarchiver
~~~

Select Xarchiver in preferences of pcmanfm

#### Python

~~~bash
sudo pacman -S python-pip tk
pip install --user pipenv
~~~

#### Miscellanous software

~~~
sudo pacman -S baobab gimp docker-compose filezilla ncmpcpp dos2unix jre-openjdk php php-cli picard alsa-utils pulseaudio-alsa dnsutils nmap fortune scrot gvfs gparted dosfstools udisks2 htop mpc mlocate pass vlc mplayer cmake docker cmatrix rsync gdisk openssh
~~~

