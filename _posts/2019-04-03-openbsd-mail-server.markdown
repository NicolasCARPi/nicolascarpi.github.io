---
layout: post
title:  "Personal mail server on OpenBSD"
date:   2019-04-03 13:37:00
categories: openbsd
---
It's time to host your email yourself!

A step-by-step guide for installing and configuring a mail server on OpenBSD.

As most of the entries in this blog, this is more of a reminder for me in case I need to do it again. But I publish it in the hope it might be useful to someone planning to do the same thing.

This was done with OpenBSD 6.4.

## The stack

Let's begin with a description of the stack and what we're going to use:

* OS: OpenBSD. Stable, secure, perfect for hosting email, a precious thing in this day and age
* SMTP: Opensmtpd. A clean codebase with a sane config file.
* IMAP: Dovecot. Never used anything else anyway.
* DKIM: Dkimproxy. Works nicely with opensmtpd.
* SPAM: Spamd.

## OS

### The server

Ok the first thing to do is to get a hand on an OpenBSD server. If you already have one, great. If not, do like me, get one from Vultr.com. If you use [this link](http://www.vultr.com/?ref=7164540) you'll get $10, so about 2 months free!

Also, Vultr.com is one of the few VPS provider supporting OpenBSD. Another one would be [OpenBSD.amsterdam](https://openbsd.amsterdam/).

The cheapest offer of Vultr.com is more than enough for an email server.

### First steps on the server

Once you get the IP address of your server, check to see if the IP is not blacklisted with [this tool](https://mxtoolbox.com/blacklists.aspx). If it is, ask for a new server. If not, go on.

Now to connect to it, edit your ssh config file and add an entry where `<hostname>` is the name of the instance and `<ip>` its IP address.

File: `~/.ssh/config`

~~~
Host <hostname>
Hostname <ip>
~~~

Now connect:

~~~bash
ssh root@<hostname>
tmux
~~~

Set source for packages, use a mirror close to the server's location:

File: `/etc/installurl`

~~~
https://ftp.fr.openbsd.org/pub/OpenBSD/
~~~

Patch the system and reboot:

~~~bash
syspatch -c
reboot
# wait for reboot
ssh root@<hostname>
~~~

Fix time:

~~~bash
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
~~~

### Install the essentials

~~~bash
pkg_add -v vim zsh git htop unzip cmake
# select vim no x11 python3 (10)
# select unzip iconv (2)
~~~

#### Dotfiles install (skip this if you're not me)

~~~bash
git clone --recursive https://github.com/NicolasCARPi/.dotfiles
./.dotfiles/install.sh
vim ~/.vim/vimrc
# set g:ycm_server_python_interpreter to /usr/local/bin/python3
# close vim
vim
:PluginInstall
# finish YCM install
cd ~/.vim/plugins/YouCompleteMe
python3 install.py
# remove the --color=auto from zshrc and fix path in git_prompt plugin in zsh folder
~~~


#### Configure SSH

In the file `/etc/ssh/sshd_config`, change the port to something other than 22, preferably > 10000.

~~~
rcctl reload sshd
~~~

#### Add your user

Enough with root already, let's add a user:

~~~bash
adduser
# login groups: <username> staff
~~~

Allow member of staff to execute commands as root:

File: `/etc/doas.conf`

~~~
permit setenv { -ENV PS1=$DOAS_PS1 SSH_AUTH_SOCK } :staff
~~~

Disconnect from server.

On your host, edit `~/.ssh/config` and add a line `Port <port>` for the server. Add a line `User <user>` if your local user is different from the one you created on the server.

Copy your public key to the server:

~~~
ssh-copy-id -i ~/.ssh/id_ed25519.pub <hostname>
ssh <hostname>
~~~

Repeat the dotfiles step.

Edit `/etc/ssh/sshd_config` and set "EnableRootLogin" to no; `doas rcctl reload sshd`.

## DNS

Add AAAA and A pointing to IP of the server.
Add MX ponting to `<domain>.` (point at the end!)

## OpenSMTPD

Good news, you don't have to install it as it's shipped with the base, how great is that! :)

Here is my config file (`/etc/mail/smtpd.conf`), adapt to your needs:

~~~conf
# $OpenBSD: smtpd.conf,v 1.11 2018/06/04 21:10:58 jmc Exp $

# configure TLS
pki <domain> key "/etc/letsencrypt/live/<domain>/privkey.pem"
pki <domain> cert "/etc/letsencrypt/live/<domain>/fullchain.pem"

# aliases table
table aliases file:/etc/mail/aliases

# listen directives
listen on all tls hostname <domain>
listen on all port 587 hostname <domain> tls pki <domain> auth
listen on lo0 port 10028 tag DKIM

# send mail to maildir ~/.mail for local accounts in alias table
action "local" maildir "%{user.directory}/.mail" alias <aliases>

action "relay" relay helo <domain>
action "relay_dkim" relay host smtp://127.0.0.1:10027

# <domain>
match from any for domain "<domain>" action "local"
# local
match for local action "local"
# dkim
match tag DKIM for any action "relay"
##match auth from any for any action "relay"
match auth from any for any action "relay_dkim"
~~~

## TLS

For getting a certificate for the domain, we will use acme-client, shipped with OpenBSD.

~~~bash
acme-client -ADv <domain>
~~~

## Dovecot

~~~bash
pkg_add dovecot
~~~

There is an issue with dovecot triggering the max open files limit, so let's fix that first.

File: `/etc/sysctl.conf`

~~~
kern.maxfiles=102400
~~~

File: `/etc/login.conf`

~~~conf
dovecot:\
  :openfiles-cur=1024:\
  :openfiles-max=2048:\
  :tc=daemon:
~~~

Now let's tweak the base config of Dovecot to our liking.

~~~
cd /etc/dovecot/conf.d
~~~

File: `10-mail.conf`

~~~
mail_location = maildir:~/.mail
~~~

File: `10-ssl.conf`

~~~
ssl_cert
ssl_key
ssl_dh_parameters_length = 2048
~~~

~~~bash
rcctl enable dovecot
rcctl start dovecot
~~~~

## DKIMProxy

Sign the outgoing emails with DKIM or expect going to the spam folder...

~~~
pkg_add dkimproxy
~~~

File: `/etc/dkimproxy_out.conf`

~~~
listen    127.0.0.1:10027
relay     127.0.0.1:10028
domain    <domain>
signature dkim(c=relaxed)
keyfile   /etc/mail/private.key
selector  mail
~~~

Generate the private/public keys for DKIM:

~~~bash
cd /etc/mail
openssl genrsa -out private.key 2048
openssl rsa -in private.key -pubout -out public.key
~~~

~~~bash
rcctl enable dkimproxy_out
rcctl start dkimproxy_out
~~~

## Spamd

I don't use it (yet).


ENJOY!

~Nico
