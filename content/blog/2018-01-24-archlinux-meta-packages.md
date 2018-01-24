+++
date = "2018-01-24T18:12:17Z"
title = "Managing Arch Linux with Meta Packages"
draft = true
description = "Show how you can use meta package to manage multiple arch linux systems"
slug = "archlinux-meta-packages"
tags = ["linux", "automation", "archlinux"]
+++

I really enjoy Arch Linux for its customisation, you can truly make it your own. But while crafting your system exactly how you want it once can be a fun and rewarding experience, repeating this process on multiple computers becomes tedious. Worst, it very easily leads to inconsistencies between each system as you forget things that you configured or as you slowly evolve your configs on each system you set up. The more systems you manage the worst this problem becomes.

There are a fair few solutions out there that aim to make this easier, most notably the plethora of *[dotfile managers]* out there. I have used a few of these over the years and while they have made the above problems easier to deal with they do not solve all the problems I have. The biggest of which is that none of them handles system configs at all only your configs in your home directory, basically only solving half the problem.

What I wanted was a way to manage my whole system, to make it easy and quick to install a new system and to keep the system up to date with my latest configs and packages that I use. For a while, [I tried][salt-arch] using [SaltStack] to manage my systems and users and while effective for a while it was more maintenance than was worth it.

In my [last post], I talked about how you can create and maintain a custom Arch Linux repository for AUR packages. But there was something else I wanted to use this repo for - creating and maintaining [meta packages] to solve some of the issues above.  In this post, I will talk about these meta packages, what they are, how to create them and how they can solve the issues above.

[dotfile managers]: https://wiki.archlinux.org/index.php/Dotfiles 
[SaltStack]: https://saltstack.com/
[salt-arch]: https://github.com/mdaffin/salt-arch
[last post]: /blog/archlinux-repo-in-aws-bucket/
[meta packages]: https://wiki.archlinux.org/index.php/creating_packages#Meta_packages_and_groups

## What are Meta-Packages

This is quite simple, meta-packages a simply packages that install nothing, but depend on other packages. The result of this is that you can install a group of packages at once by installing a single meta-packages. Now Arch Linux has a similar method; package groups, while they serve a similar purpose they are suitably different. This difference is when you add or remove dependencies - groups will not install/remove additional dependencies whereas meta packages will. This is what we want; when we add a dependency we want all our systems to automatically install it when they update.

Now, we are going to abuse this concept slightly and not only use them to install groups of packages via dependencies but also to install related system configuration. This is not something `pacman` is really designed to do and results in some hacky workaround which we will discuss below. The end result, however, works very well in practice.

## Organising Our Meta-Packages

We can create as many or as few meta-packages as we require. If you want to configure all of your systems identically you can create a single mega-meta-package that defines all of the dependencies and packages that you want. Or you can bundle up each application with related packages and configs into separate packages. You can even create a hierarchy of packages by depending on other meta packages you have created. This can be useful for creating more complex systems of overrides.

For example, I have a set of packages that I install on all of my systems - so I created a meta package `mdaffin-base` to contain these. Note that I will start all my packages with `mdaffin-` to make searching/filtering easier. I can then create a `mdaffin-server` that depends on this base package as well as addition packages that I use on servers while also creating a `mdaffin-desktop` that contains all the packages I use on my desktop systems. This has the advantage that I can update the base package to install or change something on all the system I manage, or add/change something inside my desktop package to only affect my desktop systems.

In spite of this, on each system, I only need to install one package, either `mdaffin-server` or `mdaffin-desktop` and they will both pull in the base configs.  But you can also separate packages orthogonally, for example in addition to the `mdaffin-desktop` I have a `mdaffin-devel` that contains development utilities and tools. This package can be installed alongside, or separately from the main desktop package. This allows me to configure any system for development simply by running `pacman -S mdaffin-develop` but most of my systems do not have to include these. You can also have packages for specific systems, such as my `mdaffin-dell-xps-13` which requires additional packages and configs that are required for my Dell XPS 13 which are not needed on my desktops or other systems.

One major advantage of using meta-packages like this is it is very simple to see how your system is configured with simple `pacman` commands, for example, to see packages that were explicitly installed run the following

```bash
% pacman -Qe
hugo 0.34-1
krita 3.3.3-1
mdaffin-dell-xps-13 0.0.2-1
mdaffin-desktop 0.0.9-1
mdaffin-devel 0.0.3-1
powertop 2.9-1
s3fs-fuse 1.80-2
```

Here you can see my system has a desktop interface, the Dell XPS additions and is configured for development. Note that there are additional packages as well - but not too many. These are packages which I wanted on this system only, but not on my systems in general. The use of meta-packages helps to keep this list clear and makes it easier to find/remove unused packages. From here I can either chose to add them to a meta-package, remove them or just leave them on this one system.