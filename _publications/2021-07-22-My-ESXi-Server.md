---
layout: post
title:  "My ESXi Server"
categories: [publications]
tags: [ESXi, VMWARE]
year: 2021
description: ESXi Server builted for virtualization
---

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
| CPU    | [2x Xeon E2600 3.10GHz Eight 8-Core](https://ark.intel.com/content/www/us/en/ark/products/series/59138/intel-xeon-processor-e5-family.html)                   |
| RAM    | 24 GB RAM 1600MHz Kingston            |
| GPU    | ATI Radeon HD-3470 256 MB GDDR2                                                                                      |

I added two Western Digital 500 GB HDDs and one Samsung Evo 500 GB SSD that I had in another workstation.

Then I needed to choose between ESXi or Proxmox. After reading some tips from a few forums I choosed ESXi for two reasons. First I could run ESXi from 
a usb stick since ESXi OS goes to RAM on boot making almost no read/writes into the usb stick and the second reason was that VMware is well known and has a bigger community making easier to find
help online.

<div style="text-align:center">
    <img style="width:200px" src="/images/my_esxi_server/esxi_server.jpeg">
</div>

After making the usb stick bootable with ESXi, the server was up and running like a charm.

Even having a nice UI in VMware ESXi Web Interface I had to monitor ESXi outside of my LAN so I made a python script to help me on that job.


<div style="text-align:right">
    <img style="width:650px" src="/images/my_esxi_server/esxi_monitor.jpg">
</div>




