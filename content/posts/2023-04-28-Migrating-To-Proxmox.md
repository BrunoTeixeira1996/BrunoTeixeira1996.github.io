---
title: "Migrating from ESXi to Proxmox"
date: 2023-04-28
draft: false
toc: true
tags:
  - Homelab
---

## Index

- [Introduction](#introduction)
- [ZFS Pool](#zfs-pool)
- [Content](#content)
- [Resources](#resources)

## Introduction

[During college I used ESXi to host a few VM's](https://brunoteixeira1996.github.io/posts/2021-07-22-my-esxi-server/) because I had a low end laptop that was not able to handle multiple VM's turned on.
After finishing college I decided to move to Proxmox as I wanted to persue the open source route.

To be able to acomplish this I gathered two HDD drives (500 GB each) and two SSD (250 GB each).
The plan was simple, I would use a ZFS pool (Raid 1) for the two HDD drives and another ZFS pool (Raid 1) for the SSDs using HDDs as data storage and SSDs to run the VMs' operating systems.


## ZFS Pool

I installed Proxmox OS in the SSD zfs pool (this is easily done in the setup menu) and after that I created the other zfs pool for the HDDs.
For the second zfs pool I used a really cool [youtube video](https://www.youtube.com/watch?v=oSD-VoloQag) that goes step by step on how to do this.
So I created a backup folder and a ISOs folder.


## Content

After reading about LXC I decided to use it as they are awesome to segment services, faster than VM's and easy to be created and since I don't need any GUI LXC were the best option.

The following shows the current LXC I am currently running and their purpose

- Web server (custom go web server for rating movies and series)
- telegram bot (telegram bot that handles some useful commands)
- Wireguard (my vpn)
- Syncthing (useful to sync files and folders between machines like keepassxc database)
- Fresh RSS (rss feed)


### Setting up an external hard drive

I was looking for an affordable external hard drive to act as a data storage for the Proxmox server.
I've found a cheap Seagate Basic 2TB USB 3.0 and decided to buy it.

Following a [guide on youtube](https://www.youtube.com/watch?v=w9X94bAm3dI) I was then able to add a storage to Proxmox and use it just like a mount point on all of my LXC containers.


## Resource

- https://arstechnica.com/information-technology/2020/05/zfs-101-understanding-zfs-storage-and-performance/
- https://www.youtube.com/watch?v=oSD-VoloQag