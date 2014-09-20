---
layout: post
title:  "Get a real shell, use zsh"
date:   2014-09-21 00:46:44
categories: shell completion
---
Ok so when you start your terminal, you are presented with a [bash][] prompt. [Bash][] is the command interpreter, and it's the main thing that you'll use to interact with your OS.

Unfortunately, bash is installed by default, but is not the best shell out there. You guessed it, we're gonna use [zsh][].

First install zsh with your favorite package manager.

Then you want to set it as your default shell :

~~~sh
chsh -s /usr/bin/zsh
~~~

If you get an error like it's not a valid shell, get the real path of zsh with :

~~~sh
which zsh
~~~

Now you want a config file. This is where it starts getting interesting. You can see my config file [here][].
I believe it is sufficiently commented so I don't have to dissect it here.

The interesting bits :

* very cool autocompletion
* a long history merged between sessions
* suggestion when you mistype a command
* some cool aliases
* a nice prompt on two lines
* colors for ls
* one of my favorite : begin typing a command, hit Up and you'll search history for commands that began with what you typed
* automatic ls after a cd (very very useful, you can't live without it afterwards)
* an extract command that extract your archive (no need to remember the tar flags anymore)

There is also a [git prompt script][] that shows the status of the current git repo. You should use it if you use git, it works very well.

Use `source ~/.zshrc` to reload your config.

Now enjoy autocompleting everything ! :D

~Nico

[here]: https://github.com/NicolasCARPi/.dotfiles/blob/master/zsh/zshrc
[git prompt script]: https://github.com/NicolasCARPi/.dotfiles/tree/master/zsh/git-prompt
[bash]: https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[zsh]: https://en.wikipedia.org/wiki/Z_shell
