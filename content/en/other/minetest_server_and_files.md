---
title: Minetest Server (and Files)
---

## How to Join

Download [Minetest](https://web.archive.org/web/20220521151638/https://www.minetest.net/downloads/) - on a Unix-like it's probably in your package manager.

- Address: concealed.world
- Survival - Port 30001
- Creative - Port 30002

There are no rules, I don't really have the time or patience to rigourously enforce them. Treat others as you would like to be treated, and get away from spawn. It's usually quiet, so don't expect to play with others unless you arrange to do so yourself.

If you just want to get playing you can probably ignore the rest of this page. Otherwise, read on:

## Why Minetest?

Minetest is free and open-source, so the more principled may be drawn to it. Additionally, being free, it's easy to get people to try it. The engine is written in C++ rather than Java, which typically uses less resources and thus is better suited for low-end machines.

## Why This Particular Server?

I've played it to death alone and with others, and thus I've had the chance to add an extensive collection of mods, with a comfortable balance between detail and running light. The vast majority of problems should have been already ironed out.

Examples of some additions:

- Dozens of ways to cut/shape/colour blocks, along with many decorative items.
- Many new biomes and lots of new flora (above and below ground).
- Floating islands at various sizes and altitudes.
- In-game guide to progressively unlock each crafting recipe.
- Electronics and logic gates, item and fluid pipes, and vending machines (which operate while the owner is offline).
- Farming, food items, cooking utensils, and alcohol brewing.
- In-game pixel art canvases.
- Convenience features, such as automatically cutting down trees, and teleporters.

## Mod Download

You can download the full collection of mods here to try out on your own worlds:

|                                                              | Name                    | Download                                                     |
| ------------------------------------------------------------ | ----------------------- | ------------------------------------------------------------ |
| [![Minetest Logo](https://web.archive.org/web/20220521151638im_/https://concealed.world/img/blog/minetesticon.png)](https://web.archive.org/web/20220521151638/https://concealed.world/img/blog/minetesticon.png) | Minetest Mod Collection | [[minetest-v550.7z\] - 20,574 KB](https://web.archive.org/web/20220521151638/https://concealed.world/dwn/minetest-v550.7z) |



## Mod Download - Set Up

Underneath the mods/ and textures/server/ directories, there are files named 'changes.md' which describe any alterations I've made to the original mods/texture packs.

You should disable the default mods 'mtg_craftguide' and 'weather', which conflict with existing mods. They can be disabled by removing them from the directory - if you installed minetestserver through the package manager, this will be under /usr/share/minetest/games/minetest/mods/ or similar:

```
# mkdir /usr/share/minetest/games/minetest/unused_mods/
# mv /usr/share/minetest/games/minetest/mods/mtg_craftguide/ /usr/share/minetest/games/minetest/unused_mods/
# mv /usr/share/minetest/games/minetest/mods/weather/ /usr/share/minetest/games/minetest/unused_mods/
```

Run the below script to download and set up the files (warning: this will purge any existing ~/.minetest/ directory):

```
#!/bin/sh
curl https://concealed.world/dwn/minetest-v541.7z -O
rm -rf ~/.minetest/
mkdir ~/.minetest/
7z x -o$HOME/.minetest minetest-v541.7z
rm -f minetest-v541.7z
mkdir ~/.minetest/worlds/
mkdir ~/.minetest/worlds/survival/
mkdir ~/.minetest/worlds/creative/
cp ~/.minetest/survival.mt ~/.minetest/worlds/survival/world.mt
cp ~/.minetest/creative.mt ~/.minetest/worlds/creative/world.mt
```

I'd generally recommend putting minetestserver command in an init system service file. Here's an example for systemd (remember to replace 'user' with the name of your standard user):

```
[Unit]
Description=Start Minetest World

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
ExecStart=sudo -u user /usr/bin/minetestserver --port 30001 --world /home/user/.minetest/worlds/survival/ --config /home/user/.minetest/minetestsurvival.conf --logfile /home/user/.minetest/debug_survival.txt
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```

For the creative world, the ExecStart line would be:

```
ExecStart=sudo -u user /usr/bin/minetestserver --port 30002 --world /home/user/.minetest/worlds/creative/ --config /home/user/.minetest/minetestcreative.conf --logfile /home/user/.minetest/debug_creative.txt
```

## The Spawn I've Just Generated is Crap

Run the script below to continually flush and regenerate world data, until you see something you like (again, be mindful of 'user'):

```
#!/bin/sh
[ $EUID -ne 0 ] && echo This script must be run as root. && exit 1
systemctl stop minetestcreative
systemctl stop minetestsurvival
sudo -u user rm -rf /home/user/.minetest/worlds/*
sudo -u user mkdir /home/user/.minetest/worlds/creative/
sudo -u user mkdir /home/user/.minetest/worlds/survival/
sudo -u user cp /home/user/.minetest/survival.mt /home/user/.minetest/worlds/survival/world.mt
sudo -u user cp /home/user/.minetest/creative.mt /home/user/.minetest/worlds/creative/world.mt
systemctl start minetestcreative
systemctl start minetestsurvival
```
