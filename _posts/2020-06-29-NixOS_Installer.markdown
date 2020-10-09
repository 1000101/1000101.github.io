---
layout: post
title:  "Install NixOS with Installer"
description: Starting with NixOS - NI
date:   2020-06-29 23:30:00 +0000
categories: NixOS Install NI
comments: true
---
It's been a while since my last article, where we discussed the basic principles on which NixOS stands. To fully test it out, you might want to give it a go and install it on your system. 

## Using pre-built VirtualBox appliance

A good place to start is NixOS VirtualBox appliance. Just follow [these][link-nixos-vb] instructions and you're all set.

## Motivation

But this article was aiming to provide NixOS install automation. There are some helpful [articles][link-chris-martin] which got me going, but what I still miss is an installer. We've seen [such tendencies][link-cleverca-installer] before. However, there is currently no maintained version I know of, which is both good and bad.

One of the explanations I've stumbled across quite a few times was that you don't really want NixOS unless you precisely know what you're doing. I agree to a degree. There's a lot of people out there which prefer learning by doing. I'm one of them. And you can't even start getting to know NixOS without knowing some Linux basics. Although I agree that it's beneficial to know how partitioning works, it's not necessary. So why do we include commands for partitioning in the official installation guide (for people to blindly copy&paste those), while assuming users already know all of this?

There's even a [YouTube video][link-dotslash] with a quick explanation and step by step installation instructions. But still, it's not a guided installer.

## NixOS Installer

Long story short, [here's the installer][link-ni], be careful (this installer will wipe your disk) follow README and have fun :)

## NixOS Inside

So, what's inside? Whole config is available [here][link-config]. You'll get:

- To choose your username
- To choose whether you want to encrypt your disk
- Automatically generated diceware password for root/encryption
- 8GB of SWAP in addition to your primary partition (which will use rest of the disk)
- GNOME3 as your default desktop (+GDM)
- Few GNOME3 tweaks (Literally, Tweaks)
- Some basic packages (ark, vim, unzip, ...)
- A decent browser (Firefox)
- A password manager (KeepassXC)
- A private messenger (Signal)
- A mail client (Thunderbird)
- Enabled unfree (Steam, Spotify) packages installation
- Enabled BlueTooth support with pulseaudio
- Enabled printing
- And much more!!!

The installer should guide you through the process.

## Summary

I hope you'll have fun and that it'll work for you. Don't hesitate to open issues directly on GitHub. In the next post, we'll look around the package management, generations and the default [configuration][link-config] and try to hack some bits. Cheers!

[link-nixos-vb]: https://nixos.org/download.html#nixos-virtualbox
[link-chris-martin]: https://chris-martin.org/2015/installing-nixos
[link-cleverca-installer]: https://github.com/cleverca22/nixos-installer
[link-cleverca]: https://github.com/cleverca22/nixos-installer
[link-dotslash]: https://www.youtube.com/watch?v=oPymb2-IXbg
[link-ni]: https://github.com/1000101/ni
[link-config]: https://raw.githubusercontent.com/1000101/ni/master/configuration-template.nix