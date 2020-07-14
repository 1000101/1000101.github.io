---
layout: post
title:  "NixOS Channels"
description: Starting with NixOS - Channels
date:   2020-07-14 23:30:00 +0000
categories: Nix NixOS 101 Channels
comments: true
---
Alright, now that we're running NixOS, it's time to go down the rabbit hole! Ready? In this episode, we'll take a look at channels.

## Channels

Channels are the basic building blocks of your system. They define what versions of packages and Linux kernel you'll going to get when installing an application or upgrading your system. In distros such as Debian, these are called Sources.

In addition, you can use multiple channels, for example, `stable` for your base system and `unstable` for user applications, but more on that later on. Let's begin already!

### Definition and explanation

From [NixOS wiki][link-wiki]: 'A "channel" is a name for the latest "verified" git commits in Nixpkgs. Each channel has a different definition of what "verified" means. Each time a new git commit is verified, the channel declaring this verification gets updated. Contrary to a user of the git master branch, a channel user will benefit from both verified commits and binary packages from the binary cache.'

What this essentially means is, that you get both the benefits of rapid development (since the code from which the channel is created is directly on GitHub) combined with stability (since there are tests which need to pass first, for the channel to get published/updated).

Under the hood, channel is simply a [GitHub repository][link-nixpkgs] containing all packages, service definitions and so on. There are loads of different branches, but only a subset (actually two) of these branches are processed (built&tested) further, subsequently released as channels and updated.

[Here's][link-status] a handy site for checking the current state of channels:

![Image](/assets/3-status.png)

### Extra: Channels to branches mapping

Are all GitHub branches created equal? No, only the special branches get to become the channels. In reality, two branches are used multiple times and with different rules or jobsets. The currently (at the time of writing) active channels (shown on the screenshot above) are:

- nixpkgs-unstable = [master][link-nixpkgs]
  - all packages for all supported platforms (Linux, Darwin)
  - if most commonly used packages are built without errors, the channel gets updated
- nixos-unstable = [master][link-nixpkgs]
  - only packages for Linux platform
  - if most commonly used packages are built without errors, **and additional tests pass**, the channel gets updated
- nixos-unstable-small = [master][link-nixpkgs]
  - same as above but with very few jobs (~50) compared to the above (~30k)
  - smaller number of tests
  - gets built and updated more often, however, there are very few packages built, so you have to compile a lot of stuff yourself
- nixos-20.03 = [nixos-20.03][link-nixpkgs-20.03]
  - a set of tests in different configurations need to pass before the channel gets updated
  - stable, thoroughly tested branch which contains only backports that are fixing CVE (security issues) or a previously uncaught bug
  - is available for download from the [download page][link-download]
- nixos-20.03-small = [nixos-20.03][link-nixpkgs-20.03]
  - same as nixos-unstable-small but on the stable, nixos-20.03 branch + is available for download directly from the [download page][link-download]
- nixpkgs-20.03-darwin = [nixos-20.03][link-nixpkgs-20.03]
  - only packages for Darwin platform

Note: Not all builds & tests have to pass. Some (i.e. non-vital) jobs may fail, but the channel can still get updated.

The rest of the channels are outdated (red label in Last updated field). These are older, non-maintained releases. Only the latest stable channel is being maintained.

### Stable and unstable channels

Stable channels are built from the [latest release branch][link-nixpkgs-20.03], which is released roughly every half a year (in March and in September, hence XY.03 and XY.09, where XY are the last digits of the current year). The current (at the time of writing) stable version is 20.03, with the upcoming release of 20.09 in September 2020. This maps to the `nixos-20.03` channel.

Unstable channels are built directly from the master branch and are not as thoroughly tested as stable releases. Of course, you can still use them for your base system if you want, but you should not expect them to be in a production-ready condition. Since NixOS is pretty much undestroyable, it's fairly common to use unstable for your system. If anything goes wrong, you can just do a rollback. Unstable maps to the `nixos-unstable` channel.

TL;DR:

`nixos-20.03` = stable

`nixos-unstable` = unstable

### Gluing it all together

So, there are three steps in the process:

1. Commit gets into a specific branch of nixpkgs repository on GitHub (someone merges a pull request)
2. Hydra, the NixOS build&test platform, picks it up and churns on it (building and testing)
3. If all required builds and test jobs defined for that channel pass, [cache.nixos.org][link-cache] will download the built binaries from Hydra and channel gets updated. This is handy as you can use cached binaries instead of building them yourself

![Image](/assets/3-channels.png)

## TL;DR - which channels do I use?

By default, when you use the image from the [official website][link-download], you'll get the latest stable channel. This channel is used for both your system (using `sudo nixos-rebuild`) and packages installed as a user (using `nix-env`).

You can verify this by running `nix-channel --list` first as a user:

```bash
$ nix-channel --list
```

and then as root:


```bash
$ sudo nix-channel --list
nixos https://nixos.org/channels/nixos-20.03
```

Notice the output of the first command is empty, but this doesn't mean the user will come out empty-handed. Instead, user can use the same channel when installing packages as the system does.

## Even deeper down the rabbit hole

A great neat command for displaying the active channels is `nix-info -m`:

```bash
$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2411.30fb4e1e206 (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `""`
 - channels(root): `"nixos-20.03.2411.30fb4e1e206"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

It's a little bit more verbose and here's how to decipher that information:

![Image](/assets/3-info.png)

- alias is just a name given to a channel. By default, this is `nixos`, but you can specify alias when adding a channel
- version is version of the system, in our case 20.03, as we're running on stable release
- rev count is revCount in [release.nix][link-release-rev] file. It's computed by Hydra from `git rev-list --count 20.03` - `212894` which is a constant introduced during release and it marks the number of commits at which channel first releases. In essence, revCount is the number of new commits since the release
- git commit hash is the hash of the commit from which Hydra checkouts the Nixpkgs GitHub repository

## System vs User channels

Simply put, channels are used for package installation and upgrades. In NixOS, regular user (in my case **b1000101**) and system user (**root**) are strictly separated. That's why we need to talk about both user and system (root) channels. User channels are not mandatory and they can be used when installing software through `nix-env`. Notice that we don't have to use `sudo` as with other Linux distros. That's because we're not manipulating with the system channel.

System, root, channels are associated with the whole system configuration and packages defined in `/etc/nixos/configuration.nix`. You may notice the root keyword here, so we'll definitely need `sudo` when manipulating with system (root) channels. As a matter of fact, using `sudo` is how we differentiate between user and system channels.

By default, NixOS will not add any additional (user) channels. This can be verified by running the aforementioned `nix-info -m` command. Let's look at the line with my username:

```bash
- channels(b1000101): `""`
```

It doesn't contain any channel info. However, this doesn't mean you can't install new programs. You will still be able to use the root channel for this purpose and I will demonstrate that in the next episode where we'll talk about generations and package installation.

Apart from the `nix-info -m` command, there are specific commands for listing the channels.

Listing the user channels:
```bash
$ nix-channel --list
```

Listing the system (also called root) channel (notice `sudo`):
```bash
$ sudo nix-channel --list
nixos https://nixos.org/channels/nixos-20.03
```

## Fiddling around with channels

Channels can be updated, added and of course, removed.

### Adding a channel

Adding a channel means setting a URL from which the information will be downloaded in the update step. The syntax contains the URL and **alias**, which is the name we give to a channel. It's totally in our hands, but since I lack creativity and it's nice to keep things straight, I've chosen the alias **unstable** in this case.

So, let's add the channel called **unstable** as user:

```bash
$ nix-channel --add https://nixos.org/channels/nixos-unstable unstable
```

And let's verify this:
```bash
$ nix-channel --list
unstable https://nixos.org/channels/nixos-unstable

$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2411.30fb4e1e206 (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `""`
 - channels(root): `"nixos-20.03.2411.30fb4e1e206"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

Wait, what? Why is it listed in the `nix-channel --list` but missing from the `nix-info -m`? That's because we have just specified the URL but haven't fetched/downloaded/updated the channel yet.

### Updating a channel

This operation would be equal to `apt update` on Debian-like systems, and it tells your system to download the latest channel off the Internet.

Note: Updating a channel fetches the latest released channel, but it doesn't upgrade your packages or system. There are separate commands (`nix-env -u` for user channels and `sudo nixos-rebuild switch` for root channel) for that.

### Updating a user channel

Initial state:
```bash
$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2411.30fb4e1e206 (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `""`
 - channels(root): `"nixos-20.03.2411.30fb4e1e206"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

As we noticed previously, user channel, **channels(b1000101)** in my case, is empty.

So let's try to update it first and then re-run the `nix-info -m`:

```bash
$ nix-channel --update
unpacking channels...

$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2411.30fb4e1e206 (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `"unstable-20.09pre233323.dc80d7bc4a2`
 - channels(root): `"nixos-20.03.2411.30fb4e1e206"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

And voilla, it's there! If we'd run the update command in a few hours or days (it varies), this command would fetch the latest updated channel off the Internet:

```bash
$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2411.30fb4e1e206 (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `"unstable-20.09pre234194.8d05772134f"`
 - channels(root): `"nixos-20.03.2411.30fb4e1e206"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

Notice that `unstable-20.09pre233323.dc80d7bc4a2` changed to `unstable-20.09pre234194.8d05772134f` so essentially, 871 new commits were merged into the master repository since I last did the update. Now that's what I call rapid development! :)

### Updating a system (root) channel

Now, let's look at the system, or root, channel. We'll continue right where we left off.

Initial state:
```bash
$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2411.30fb4e1e206 (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `"unstable-20.09pre234194.8d05772134f"`
 - channels(root): `"nixos-20.03.2411.30fb4e1e206"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

Notice that **host os** and **channels(root)** are the same. The system is up to par with its channel. However, this doesn't mean it is up to date. The channel could have been updated since we last fetched it from the Internet. So, let's update the channel.

Since we're updating system (root) channel, we'll have to use `sudo` prepending the command.

Updating system (root) channel and verifying the result:

```bash
$ sudo nix-channel --update
unpacking channels...

$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2411.30fb4e1e206 (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `"unstable-20.09pre234194.8d05772134f"`
 - channels(root): `"nixos-20.03.2520.add5529b3ee"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

So, the **channel** is updated, but the **host os** remains the same. We'd still need to run the rebuild (`sudo nixos-rebuild switch`) process to upgrade our system.


### Removing a channel

To remove a channel, just use the ``--remove`` flag followed by the channel alias.

```bash
$ nix-channel --list
unstable https://nixos.org/channels/nixos-unstable

$ nix-channel --remove unstable 
uninstalling 'unstable-20.09pre234194.8d05772134f'

$ nix-channel --list

```

### Extra: Upgrading your system (root channel) to latest stable

An upgrade from any version to latest stable (20.03) in one step would look like this:

```bash
$ sudo nix-channel --add https://nixos.org/channels/nixos-20.03 nixos && sudo nixos-rebuild switch --upgrade
```

This is a shorthand for:
1. adding the channel (`sudo nix-channel --add`)
2. updating the channel (`--upgrade`)
3. upgrading your system (`sudo nixos-rebuild switch`)


```bash
$ sudo nix-channel --list
nixos https://nixos.org/channels/nixos-20.03

$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2520.add5529b3ee (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `""`
 - channels(root): `"nixos-20.03.2520.add5529b3ee"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

And there it is, `channels(root)` finally matches `host os` version. Your system is up to date!

### Extra: Upgrading your system (root channel) to the latest unstable

You guessed it, adding a different channel for your system (root) effectively means you're downgrading/upgrading your system. So let's try to switch your system to unstable channel.

```bash
$ sudo nix-channel --list
nixos https://nixos.org/channels/nixos-20.03

$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2520.add5529b3ee (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `""`
 - channels(root): `"nixos-20.03.2520.add5529b3ee"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

Let's do add and update and then verify the result:
```bash

$ sudo nix-channel --list
nixos https://nixos.org/channels/nixos-20.03

$ sudo nix-channel --add https://nixos.org/channels/nixos-unstable nixos 

$ sudo nix-channel --list
nixos https://nixos.org/channels/nixos-unstable

$ sudo nix-channel --update
unpacking channels...

$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.03.2520.add5529b3ee (Markhor)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `""`
 - channels(root): `"nixos-20.09pre234242.c71518e75bf"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

If you look closely, you'll notice the 20.03 channel will get replaced by unstable. This is because we used the same alias which is `nixos`. We don't have to first remove the channel, it will get replaced in-place.

As `host os` tells us, we're running 20.03 at the moment. Next time you run `sudo nixos-rebuild switch` you will effectively upgrade our system to the latest unstable version. Notice that the URL points to https://nixos.org/channels/nixos-unstable which will ALWAYS be the latest unstable, no matter what, so once 20.09 gets released, you will automatically switch to 21.03, which will become the unstable at the time of the release. 

If you want to run your system at the latest stable release, you always have the use URL with the numbers, e.g. https://nixos.org/channels/nixos-20.03 instead.

### Triple Extra: Running on a custom channel from GitHub

And here's an absolutely amazing showcase of NixOS. Running straight from a GitHub repo! For this purpose, I'll use my own fork of nixpkgs.

Here's a recap of our current config:

```bash
$ sudo nix-channel --list
nixos https://nixos.org/channels/nixos-unstable

$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.09pre234242.c71518e75bf (Nightingale)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `""`
 - channels(root): `"nixos-20.09pre234242.c71518e75bf"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

Let's add our own branch and update and then verify the result:
```bash

$ sudo nix-channel --list
nixos https://nixos.org/channels/nixos-unstable

$ sudo nix-channel --add https://github.com/1000101/nixpkgs/archive/myownbranch.tar.gz nixos

$ sudo nix-channel --list
nixos https://github.com/1000101/nixpkgs/archive/myownbranch.tar.gz

$ sudo nix-channel --update
unpacking channels...

$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.09pre234242.c71518e75bf (Nightingale)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `""`
 - channels(root): `"nixos"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```
Now, let's run `sudo nixos-rebuild switch` to upgrade our system and let's verify it:

```bash
$ nix-info -m
 - system: `"x86_64-linux"`
 - host os: `Linux 5.4.49, NixOS, 20.09pre-git (Nightingale)`
 - multi-user?: `yes`
 - sandbox: `yes`
 - version: `nix-env (Nix) 2.3.6`
 - channels(b1000101): `""`
 - channels(root): `"nixos"`
 - nixpkgs: `/nix/var/nix/profiles/per-user/root/channels/nixos`
```

And there you go, your very own NixOS built straight from your fork :). Note that sometime, it won't rebuild successfully.
This is because there may be a bug or an error which prevents from building. This is because building straight from GitHub 
doesn't pass any additional tests, i.e. it may be broken. But this is no problem, as you can replace the branch name with
a precise commit hash and try to build from that.

## Summary

Although it is possible to use your own GitHub repository (fork) of nixpkgs instead of the official channels, and it's very helpful in some cases, most of the time, you want to stick to the official stable channels (i.e. nixos-20.03) as they are well tested, and that's exactly why we want to use them. They tend to work (most of the time) pretty well! And even if they don't, you can always rollback, and we will talk about this in the following post.

[link-nixpkgs]: https://github.com/NixOS/nixpkgs/
[link-wiki]: https://nixos.wiki/wiki/Nix_channels
[link-nixpkgs-20.03]: https://github.com/NixOS/nixpkgs/tree/nixos-20.03
[link-download]: https://nixos.org/download.html
[link-cache]: https://cache.nixos.org/
[link-status]: https://status.nixos.org/
[link-release-rev]: https://github.com/NixOS/nixpkgs/blob/nixos-20.03/nixos/release.nix#L15
[link-installer]: https://nixos.online/posts/NixOS_Installer/