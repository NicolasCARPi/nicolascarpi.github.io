---
layout: post
title:  "Why GitHub?"
date:   2019-09-18 13:37:00
categories: politics
---
This post is the mirror of [Why not GitHub?](https://sanctum.geek.nz/why-not-github.html) by Tom Ryder.

In his post, Tom is argumenting against the use of GitHub. And to be honest, he makes valid points and I agree with them. But [my main project](https://github.com/elabftw/elabftw) is still hosted on GitHub. Let me tell you why.

# The hosting

Not everyone has the time and knowledge necessary to host and administer their own git instance. So using GitHub is convenient: you don't have to worry about updates, backups, uptime, bandwidth, etc...

# The popularity

While we can all agree that having a centralized private website for most of FLOSS projects is bad, it's also great because most devs will have an account, and it makes collaborating to new projects easier. We already know the UI, we can get working on a fork in less than a minute, and making a PR is very easy and clear. I don't know about you, but if I find a project hosted on savannah (GNU) or a custom git, chances for me contributing to the code dwell down very fast.

# The services

On the [eLabFTW](https://github.com/elabftw/elabftw) project, I'm using [Dependabot](https://dependabot.com/) to warn me about new versions of my dependencies, and I must say it's incredibly useful and convenient. I can't imagine the troubles to reproduce that on a self hosted server.

There are also A LOT of other useful services out there, one will check the code coverage changes on PR, another will ask the contributor to sign a license agreement, another will check the code for bugs/formatting, you get it, there are a plethora of services that you can setup on a repo in a few click, and at the end of the day, they make your life easier.

# The stars

GitHub stars are a quick way to gauge the popularity of a project. You don't want to add a 2 stars project as a dependency. But if it has 5k stars, you can be confident that it will be maintained. Software projects hosted outside of GitHub lose this indicator, for better or worse.

# The issues and wiki, code reviews, etc…

Let's be honest, the issue/wiki/code review system is pretty great and works well. That's what you get when you invest a lot of money and work with good engineers. There are a lot of things that are great with GitHub, and it probably explains its popularity. If it was a shitty website, it would very likely not be where it's at now.

# Conclusion

IMHO, using GitHub is fine, **if** you're prepared to switch to something else (either self-hosted or another service) if let's say GitHub blocks your account for some reason (a valid concern). You can also configure your git to push to two remotes at the same time. This way you can have your mirror self hosted, but the main will be GitHub.

Cheers,
~Nico, still on GitHub even after M$ acquisition :p
