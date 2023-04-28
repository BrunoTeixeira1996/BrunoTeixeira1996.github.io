---
title: "My ESXi Server"
date: 2021-08-21
draft: false
toc: true
tags:
  - Homelab
---

## Index

- [Introduction](#introduction)
- [Server Specifications](#hp-z620-specifications)
- [Backup ESXi Host](#backup-esxi-host)
- [Monitoring ESXi](#monitoring-esxi)


## Introduction

Since I joined college I had a need to have more than one operating system available for some classes so I had a few options.

First I could simply run [Oracle VM VirtualBox](https://www.virtualbox.org/) or [VMware Workstation Player](https://www.vmware.com/products/workstation-player.html).
This would work but since I have a ThinkPad X1 Carbon Gen5, it wouldn't handle every ram and cpu usage from the VMs.

The second option would be to buy a new laptop but i didn't want to overuse it.

So the third option, the one I choosed, was to buy a refurbished server and install [ESXi](https://www.vmware.com/products/esxi-and-esx.html) or [Proxmox](https://proxmox.com/en/) creating
a virtualization server giving more lifetime to the laptop.

After looking for a few weeks, I found [Bargain Hardware](https://www.bargainhardware.co.uk/). Bargain Hardware is an awesome website with realy cool prices.
I was looking for a Dell product after reading Reddit's reviews but I only found rack servers and a rack server would not fit since I
dont have a rack. So I moved to HP products and found HP Z620. Two weeks have passed by and the server had arrived.

## HP Z620 Specifications


| Hardware | Brand                                                                                                                  |
|------|----------------------------------------------------------------------------------------------------------------------------|
| CPU    | [2x Xeon E5-2670 2.60GHz Eight 8-Core](https://ark.intel.com/content/www/us/en/ark/products/series/59138/intel-xeon-processor-e5-family.html)                   |
| RAM    | 24 GB RAM 1600MHz Kingston            |
| GPU    | ATI Radeon HD-3470 256 MB GDDR2                                                                                      |

I added two Western Digital 500 GB HDDs and one Samsung Evo 500 GB SSD that I had in another workstation.

Then I needed to choose between ESXi or Proxmox. After reading some tips from a few forums I choosed ESXi for two reasons. First I could run ESXi from 
a usb stick since ESXi OS goes to RAM on boot making almost no read/writes into the usb stick and the second reason was that VMware is well known and has a bigger community making easier to find
help online.


{{< figure src="/my_esxi_server/esxi_server.jpeg" class="post-image" >}}


Then I had to see if the hardware of the server was compatible with ESXi, and it wasn't (NIC was not compatible) but then I found some [custom ESXi ISOs](https://rodrigolira.eti.br/isos-esxi-customizadas/). The next step was to make a bootable usb using the custom iso and for that I used [etcher](https://www.balena.io/etcher/).

ESXi boots up and the only thing I needed to change was the ip address and make sure the NIC was working properly with the custom iso (I could see the NIC was working because it reached my home router).

After making that, the server was up and running like a charm and the next step was to create some VMs.

{{< figure src="/my_esxi_server/esxi_gui.png" class="post-80-image" >}}


## Backup ESXi Host

Having healthy ESXi hosts is a key to success when running virtual machines. That’s why it is better to back up ESXi configuration. Thus, if something goes wrong with an ESXi host, its configuration can be restored in a few minutes without spending a lot of time to configure an ESXi server from scratch.

### Using ESXi Command Line To Backup ESXi Host

ESXi configuration is saved every hour automatically to the **/bootblank/state.tgz** file. So I needed to run two commands, one to do the backup and the other one to get a link to the specific backup.

```bash
vim-cmd hostsvc/firmware/sync_config
vim-cmd hostsvc/firmware/backup_config
```

Then I downloaded the **tgz** file stored in **/scratch/downloads**.

### Automate The Backup

#### Create a directory to store backup files on your ESXi datastore

```bash
mkdir /vmfs/volumes/datastore1/ESXi_backup
```

#### Create a script to backup ESXi configuration

```bash
vi /vmfs/volumes/datastore1/ESXi_backup/esxi_backup.sh
```

#### Add the following lines to the script

```bash
vim-cmd hostsvc/firmware/sync_config
vim-cmd hostsvc/firmware/backup_config
find /scratch/downloads/ -name \*.tgz -exec cp {} /vmfs/volumes/datastore1/ESXi_backup/ESXi_config_backup_$(date +'%Y%m%d_%H%M%S').tgz \;
```

#### Make the script executable

```bash
chmod +x /vmfs/volumes/datastore1/ESXi_backup/esxi_backup.sh
```

#### Run the script

```bash
cd /vmfs/volumes/datastore1/ESXi_backup/
./esxi_backup.sh
```

{{< figure src="/my_esxi_server/esxi_1.png" class="post-80-image" >}}


#### Edit scheduler to make sure the backup ESXi host script is running automatically

```bash
vi /var/spool/cron/crontabs/root
```

#### Add the following string to backup everyday at 02:10 AM and save the changes

```bash
10 02 * * * /vmfs/volumes/datastore1/ESXi_backup/esxi_backup.sh
```

## Monitoring ESXi

Even having a nice UI in VMware ESXi Web Interface I had to monitor ESXi outside of my LAN and, for security reasons, I didn't want to make the server public to the WAN so I made a python script to help me on that job. The script can be find on my [TelegramBot repository](https://github.com/BrunoTeixeira1996/TelegramBot/blob/main/src/modules/esxi.py).

{{< figure src="/my_esxi_server/esxi_monitor.jpg" class="post-80-image" >}}





## References

* https://www.vmware.com/products/esxi-and-esx.html
* https://www.pendrivelinux.com/etcher-usb-iso-burner-and-clone-tool/
* https://www.bargainhardware.co.uk/
* http://www.fabfile.org/