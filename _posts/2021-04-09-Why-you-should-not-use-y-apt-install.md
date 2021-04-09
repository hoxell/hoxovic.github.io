---
title: "Why you shouldn't use -y when apt install'ing packages"
layout: single
published: true
---

# _How I uninstalled my keyboard_ or _why you shouldn't use `-y` when installing packages_.

Today I totally messed up. Just before it was time to call it for this week, I got the brilliant idea that I should install the Xfce desktop environment to free up some more RAM for the simulations I'm running locally. The idea was good but, and here's the catch, I've made it a habit to install packages like so `sudo apt update && sudo apt install <package> -y`.

This may seem innocent enough, essentially saving you a few seconds by not having to confirm what `apt` will do (i.e. confirming which packages will be installed/upgraded/removed). The thing is, this time it uninstalled some of my input drivers, effectively leaving me without any way of entering my password in the greeter. Also, the on-screen keyboard wasn't available for whatever reason.

# What happened
What happened was that `apt` found that the installation would lead to incompatible packages/dependencies and therefore removed the conflicting packages that were already installed on the system. Unfortunately, one of those packages was `xserver-xorg-input-all`. This is a meta-package that installs a bunch of drivers that allows you to use your keyboard and mouse (among other devices) to interact with your GUI. It's not a disaster if this package isn't present on your system, but the issue in my case was that none of the `xserver-xorg-*` packages for keyboard input was left on the system. Consequently, neither the external nor integrated keyboard could be  used to log in and open a shell nor switch to one of the tty's.

# The solution
Now, the keyboard wasn't entirely unusable, it just couldn't be used to interact with the xserver. Typically the procedure to fix this is:

1) Boot into recovery mode
2) Launch a shell as root
3) Enable networking
4) `$ apt update && apt install xserver-xorg-input-all`

Easy enough, right?

Working for a big company, though, there were a few complicating factors that didn't make it quite this simple:
1) Custom apt sources (behind a corporate proxy, of course)
2) All default apt sources removed
3) Mandatory VPN (GUI only) to reach any company resources

## The I-work-for-a-big-company-solution
1) Boot into recovery mode
2) Launch a shell as root
3) Enable networking
4) Configure `apt` to _not_ use the proxy
5) Add the official archive links to `/etc/apt/sources.list` (or under `/etc/apt/sources.list.d`)
6) `$ apt update && apt install xserver-xorg-input-all`
7) Reset everything to the way it was before

# Where to find what `apt` has done
If you're wondering what packages were installed or removed when using `apt`, you can check the log at `/var/log/apt/history.log` to get some details.

# Retrying the installation
Luckily, retrying the install didn't require anything to be removed, and especially not the xserver drivers for keyboard input. (Tbh, I haven't bothered with investigating which package/dependency required `xserver-xorg-input-all` to be removed in the first place since I got it to work.)

# Lesson learned
Prefer not to use `-y` when installing packages with `apt`.
