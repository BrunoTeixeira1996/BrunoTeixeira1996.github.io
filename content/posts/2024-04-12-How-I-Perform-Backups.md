---
title: "How I Perform Backups"
date: 2024-04-12
draft: false
toc: true
tags:
  - Homelab
---

## Index

- [Introduction](#introduction)
- [Targets & Destinations](#targets-&-destinations)
- [Rsync](#rsync)
- [Prometheus & Grafana](#prometheus-&-grafana)
- [Golang](#golang)
- [The Tool](#the-tool)


# Introduction

Backups are like insurance policies for your digital life. They're copies of your important files, stored in a safe place separate from your original data. Just as you wouldn't want to risk losing everything in a fire without insurance, you don't want to risk losing your precious photos, documents, or other files without backups.

In this blog post, I'll share my personal insights and strategies for maintaining comprehensive backups, ensuring that no vital information slips through the cracks while also staying organized amidst the constant flux of data changes.


# Targets & Destinations

Before diving into the backup process, it's essential to define two key elements: the targets (backup sources) and the backup destinations.

Targets encompass the data we deem critical for backup. This could include documents, photos, videos, and any other files essential to our digital lives. By identifying these targets, we ensure that no vital information is overlooked during the backup process.

Next, we determine where we want to store these backups, our backup destinations. This could involve a combination of options such as cloud storage services like Google Drive or Dropbox, external hard drives, network-attached storage (NAS) devices, or even dedicated backup servers. Each option offers its advantages in terms of accessibility, redundancy, and security.

In my use case I backup all files from my work laptop, [gokrazy databases](https://github.com/gokrazy/gokrazy) and important data within LXC containers on Proxmox. This way, if there's a system failure, I can rebuild everything and upload the files for a quick recovery.


# Rsync

With targets and destinations set, it's time to select the backup method. Rsync emerges as a formidable choice, boasting efficient file synchronization capabilities. Its ability to perform incremental backups minimizes data transfer, making it ideal for handling large files and slow network connections.

# Prometheus & Grafana

Ensuring the integrity of our backup process requires meticulous inspection of file changes within specific date ranges, alongside gathering insightful metrics. To accomplish this, I've opted for Prometheus and Grafana, renowned for their robust monitoring and visualization capabilities.


# Golang

Golang's asynchronous processing capabilities make it the perfect companion for integrating data with Prometheus, ensuring seamless operation and performance optimization.


# The Finale

Enter [gbackup](https://github.com/BrunoTeixeira1996/gbackup), the culmination of these efforts. This tool orchestrates the backup plan, covering essential partitions, databases, and files across various locations, ensuring data resilience. 

The current backup plan is the following:

- gokrazy perm partion (resides in gokrazy)
- gokrazy data folder (resides in a LXC in proxmox)
- postgresql databases (resides in a LXC in proxmox)
  - currently I am only doing the backup for the waiw and leak databases
- syncthing folder (resides in a LXC in proxmox)
- monitoring files (resides in a LXC in proxmox
  - this is files from alertmanager, prometheus, grafana and pushgateway
- work laptop

For the backup locations, I utilize two distinct setups. Firstly, an external hard drive directly connected to Proxmox serves as one backup destination. Secondly, I employ an internal hard drive within the Proxmox environment as a secondary backup location. This dual setup ensures redundancy and reliability in safeguarding critical data.

{{< figure src="/gbackup/arc.png" class="post-image" >}}

{{< figure src="/gbackup/graph.png" class="post-image" >}}