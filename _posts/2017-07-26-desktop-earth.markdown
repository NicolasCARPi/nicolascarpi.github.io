---
layout: post
title:  "Desktop earth wallpaper for GNU/Linux"
date:   2017-07-26 00:13:37
categories: wallpaper script bash
---

Back in 2005 I was using [Desktop Earth](http://www.anka.me/desktopearth.aspx) on my Powerbook G4. It allows you to display the earth as a wallpaper with real time day/night cycle and clouds.

Unfortunately this software is proprietary and only for Windows/Mac. I was feeling nostalgic and wanted to get my desktop back and that feeling of looking at humanity sleeping/working through my transparent terminal window.

Here is what it looks like:

![desktop-earth](/img/desktop-earth.jpg)

Sorry here it is without the terminal:

![desktop-earth](/img/desktop-earth2.jpg)


Now the great thing is that all the hard part is already done by [die.net](https://www.die.net/earth/). So we just need to grab the image periodically and set it as wallpaper. Here is the script I made for that purpose:

~~~bash
#!/usr/bin/env bash
# display earth as desktop wallpaper
# needs feh, wget and imagemagick

# where we do our stuff
dir=/tmp
# a 1680x1050 image for the top of the screen
top=~/.bin/desktop-earth-top.jpg
# will be restored on exit
normal_wallpaper=~/.wallpapers/blackearthstars.jpg

# bring back the normal wallpaper when script exits
function exit_script()
{
    feh --bg-scale $normal_wallpaper &
    exit 0
}

# trap sigint
trap exit_script SIGINT

while true; do
    # first get the image
    wget -q https://static.die.net/earth/mercator/1600.jpg -O $dir/original.jpg
    # remove 15 pixels of height
    convert $dir/original.jpg -crop 1600x872 $dir/cropped.jpg
    # resize it to 1680
    convert $dir/cropped-0.jpg -resize 1680x1035 $dir/cropped-big.jpg
    # merge the two images
    convert $top $dir/cropped-big.jpg -append $dir/desktop.jpg
    # display on desktop
    feh --bg-scale $dir/desktop.jpg
    # 15 minutes
    sleep 900
done
~~~

The top image was generated with Gimp. It's just an image of 1680x15 pixels with a background color for the top of my screen, designed for dark backgrounds.

So to conclude, we don't need complicated software, a few lines of bash can do great things :)

ENJOY :D
