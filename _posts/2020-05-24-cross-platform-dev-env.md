---
title: "Cross-platform environment worthy of a developer"
layout: single
published: true
---

Many are familiar with the battle - you work primarily in Linux, you've treated yourself to a Mac at home and you're forced to run Windows as the host OS at work[^1]. The good, the bad and Windows :smirk:

In this post, I thought I'd share some basic tips how to set up Windows, OSX and Linux such that switching between them is going to feel effortless. When I say environment, I primarily mean the command line, so that's what this post will focus on.

<figure>
	<img src="/assets/images/zsh-tmux.jpg" width="100%">
	<figurecaption>Print screen of Oh My Zsh and tmux in Ubuntu 18.04</figurecaption>
</figure>

While setting up cross-platform development environment on Linux and OSX is a breeze, the same can't really be said when you throw Windows 10 into the mix. In a previous post, I've written about how you can use vim in PowerShell, but that's only a tiny part.


As it turns out, there are only three (four in Windows) quite simple things that was needed in order for me to feel I could seemlessly move between the OS's. Actually, this is definitely a result of my desire to keep the setup procedure as short as possible so that I can set it up quickly when moving to a new system. It's by no means a complete setup, but it's a good start and usually what I suggest to any coworkers who aren't that interested in configuring their environments.

Anyway, the three main components are:
1. Zsh
2. Oh My Zsh
3. tmux

With these things in place, you'll have access to a powerful shell, an amazing zsh configuration framework and a terminal multiplexer.


# Setting it all up

## Windows
There are a few ways to get a proper shell in Windows. Neither way is perfect (imho), but I've come to prefer WSL, simply beacuse Microsoft is working hard on getting Linux to work "in" Windows. Say what you want about Microsoft, but their documentation is definitely something else, so if you haven't set up WSL already, head over to their [official documentation](https://docs.microsoft.com/en-us/windows/wsl/install-win10) and do so.

### Zsh
```zsh
sudo apt update
sudo apt install zsh
```

Technically, this will only install zsh, not change your default shell. Setting zsh as the default shell is taken care of in the next step. 


### Oh My Zsh
Oh My Zsh provides a oneliner (sh -c + curl ...) to install the framework. I certainly hope you're not just copying and executing commands you find on the Internet (and especially not sh -c + curl), so I'm not going to include it here. Instead, head over to Oh My Zsh's [repo](https://github.com/ohmyzsh/ohmyzsh#basic-installation).

When you've installed Oh My Zsh, open `~/.zshrc` in your editor of choice and set the theme to `ZSH_THEME="agnoster"`. Also, consider having a look at the multitude of plugins offered.


#### Powerline fonts
The first thing you'll notice is that the font is broken. The reason is that Oh My Zsh requires Powerline fonts.

To install the required fonts, browse to the powerline fonts [repo](https://github.com/powerline/fonts#quick-installation) and follow the instructions. **This has to be done in Windows** (not in WSL).


### tmux
```zsh
sudo apt update
sudo apt install tmux
```

Just type `tmux` in a terminal and a new tmux session will start. Refer to [https://tmuxcheatsheet.com/](https://tmuxcheatsheet.com/) for the basic commands.

I like to have a new tmux session start by default whenever I open a new terminal. To achieve this, I've added the following to `~/.zshrc`:

```zsh
if command -v tmux &> /dev/null && [ -n "$PS1" ] && [[ ! "$TERM" =~ screen ]] && [[ ! "$TERM" =~ tmux ]] && [ -z "$TMUX" ]; then
     exec tmux
fi
```
Credit for this script goes to a [deleted user on unix.stackexchange.com](https://unix.stackexchange.com/a/113768).


## OSX
Mostly the same as for Windows/WSL.


### Zsh
Zsh has been the default shell on OSX for some time now, so probably you don't have to do anything. Actually, you'll be asked if you want to make zsh your default shell if you're still using bash.

To check which shell you're using, simply run:
```zsh
echo $0
```


## Linux
You use Linux, you know what to do :wink:


___
[^1]: Actually, my employer just started offering *nix OS's, but I'm stuck with my Windows machine for another year, so perhaps there will be time to share some more tips on Linux-ifying Windows...
