---
layout: post
title:  "Restoring Proxmox containers to Ubuntu with LXD"
date:   2021-12-23 20:00:00 +0100
tags: [systems, automation, linux, containers]
excerpt_separator: <!--more-->
---

This blog article is not about a production environment, but about my home network.
I am (or I should now say _was_) running a Proxmox node on an old Dell Optiplex 7010 SFF.
This home server is hosting several containers and VMs. Such as a media server, some webapps, a couple of DNS servers, etc.
Nothing mission critical, but they are services that I use daily.

I was not prepared to migrate them all in a weekend when my home server died one night at 1am, because of a power supply failure...
<!--more-->

## Everything begins with an outage...

I was thinking about migrating the workloads to ESXi (free) for a few months already. 

> No specific reason behind this thought, but maybe just because of familiarity with the vSphere product suite. This even though I am using Proxmox since mid 2018.

But when a morning I woke up with an [UptimeRobot](https://uptimerobot.com/) email alert in my mailbox, I new something was up. Indeed, a web app I host was considered _down_ from UptimeRobot. So I tried to open the page from my phone and the page did not load. I tried to ping and SSH into the container (LXC) running the web app, but did not succeed either. Tried to load and ping the Proxmox server, no response. 

I had to walk to the living room where the culprit was located, and check physically the server. It was shut down, no noise, no blinky-blinky LEDs. Tried to power it back up, nothing. Tried to reconnect the power cable, nothing. Swapped the cable for another one that I knew was working, nothing.

I extracted the Optiplex from its cabinet, to check the rear. The LED on the powersupply was dead too, and did not react at all to a press on its self-check button.

Well, I knew this day would come as it was running 24/7 for 3.5 years, and of course it was bought second hand too for about 150 CHF.

Thus begins the journey to migrate my Proxmox containers to ESXi.


## Is this even posible ?

As the proxmox [wiki](https://pve.proxmox.com/wiki/Linux_Container) says, proxmox is using LXC as the underlying container technology. Which means it should be as simple as installing LXD on a new server, and importing the containers.

To keep it simple, I'm going to setup an Ubuntu server based on the latest LTS release : 20.04.

After reading through the ubuntu [docs](https://ubuntu.com/server/docs/containers-lxd), I went for the snap installation procedure.

    sudo snap install lxd

Pretty easy, right ?
Note also that we can manage the containers with commands such as :

    lxc list
    lxc start my-container
    lxc stop my-container

You'll see in a bit why I'd want to set a MAC address.

> I didn't write about the steps needed to install ESXi and create an Ubuntu VM. You should be able to do this process on your own quite easily.

The containers that will be hosted on this new Ubuntu server will need to be reachable from my network, so I chose to configure the VM NIC as a bridge and I will set the LXC containers to use this interface.

I used `nmcli` to configure this bridge as `br0`, and then I disabled the initial interface.

Then I edited the default profile for LXC containers with `lxc profile edit default` and adapted the `eth0` section under `devices` as following :

    eth0:
    name: eth0
    nictype: bridged
    parent: br0
    type: nic

All containers will inherit this profile by default and will thus be connected to my network, without any NAT managed by the Ubuntu VM.

## It's not a backup if you've never try to restore it !

Okay so now that I know how I will run the containers on the Ubuntu server, I should check how I can even import them. Maybe that should have been the first step...

We said already that LXC is a standard, and can be used anywhere, great. But my proxmox server is dead, so how can I transfer the containers from it to the new server ?

Yeah, I could open my proxmox server and extract the SSD (singular, no RAID in my lab) it was running of. Connect it to my new server and mount it, find the containers and somehow tranfer them... But I feel it's too much work, and I'd like to explore another way.

I was doing backups twice a week from proxmox to my NAS, so I have backups that were taken two days before my server decided to die. It's good enough for me, no data should be lost in this case.

Well, let's have a look a those backups proxmox took, shall we ?
    
    nicolas@nas1:/volume1/vm_library/dump$ ls -al *2021_12_08*
    -rwxrwxrwx+ 1 vmhost users        837 Dec  8 01:30 vzdump-lxc-100-2021_12_08-01_30_01.log
    -rwxrwxrwx+ 1 vmhost users 1333698029 Dec  8 01:30 vzdump-lxc-100-2021_12_08-01_30_01.tar.lzo
    -rwxrwxrwx+ 1 vmhost users        841 Dec  8 01:31 vzdump-lxc-101-2021_12_08-01_30_56.log
    -rwxrwxrwx+ 1 vmhost users 1201136135 Dec  8 01:31 vzdump-lxc-101-2021_12_08-01_30_56.tar.lzo
    -rwxrwxrwx+ 1 vmhost users        713 Dec  8 01:33 vzdump-lxc-102-2021_12_08-01_31_43.log
    -rwxrwxrwx+ 1 vmhost users 2541186769 Dec  8 01:33 vzdump-lxc-102-2021_12_08-01_31_43.tar.lzo
    -rwxrwxrwx+ 1 vmhost users        838 Dec  8 01:35 vzdump-lxc-103-2021_12_08-01_33_03.log
    -rwxrwxrwx+ 1 vmhost users 2742124495 Dec  8 01:35 vzdump-lxc-103-2021_12_08-01_33_03.tar.lzo
    -rwxrwxrwx+ 1 vmhost users        837 Dec  8 01:35 vzdump-lxc-104-2021_12_08-01_35_03.log
    -rwxrwxrwx+ 1 vmhost users 1323222402 Dec  8 01:35 vzdump-lxc-104-2021_12_08-01_35_03.tar.lzo
    -rwxrwxrwx+ 1 vmhost users        837 Dec  8 01:37 vzdump-lxc-105-2021_12_08-01_35_50.log
    -rwxrwxrwx+ 1 vmhost users 2305681928 Dec  8 01:37 vzdump-lxc-105-2021_12_08-01_35_50.tar.lzo
    -rwxrwxrwx+ 1 vmhost users        837 Dec  8 01:37 vzdump-lxc-106-2021_12_08-01_37_08.log
    -rwxrwxrwx+ 1 vmhost users 1387145740 Dec  8 01:37 vzdump-lxc-106-2021_12_08-01_37_08.tar.lzo
    -rwxrwxrwx+ 1 vmhost users        839 Dec  8 01:42 vzdump-lxc-110-2021_12_08-01_41_01.log
    -rwxrwxrwx+ 1 vmhost users 2915298920 Dec  8 01:42 vzdump-lxc-110-2021_12_08-01_41_01.tar.lzo
    -rwxrwxrwx+ 1 vmhost users        836 Dec  8 01:43 vzdump-lxc-112-2021_12_08-01_42_29.log
    -rwxrwxrwx+ 1 vmhost users 2179011980 Dec  8 01:43 vzdump-lxc-112-2021_12_08-01_42_29.tar.lzo
    -rwxrwxrwx+ 1 vmhost users        843 Dec  8 01:44 vzdump-lxc-115-2021_12_08-01_43_31.log
    -rwxrwxrwx+ 1 vmhost users 1488306697 Dec  8 01:44 vzdump-lxc-115-2021_12_08-01_43_31.tar.lzo

Okay, so I have 11 containers, and for each I have two file :
- a log that was written during the backup process
- the container image itself, compressed with lzo

My backups are available, and in a pretty standard format, not too bad. Let's have a look at how we can restore them now.

## Doing it manually the first time

I've copied the backups and the log files to my new Ubuntu VM with LXD installed, now it's time to work !

The first run is going to be manual, as we will be able to write down the whole process and then automate it for the remaining containers.

After reading through several docs and articles, I know that I need to :
1. import the image of the container, and its metadata file.
1. launch the container

These are the basic tasks. 

To import it, the metadata file needs at least two variable :
- architecture
- timestamp

So we need to create `metadata.yaml` with the following data :

    architecture: x86_64
    creation_date: 1639520168

Of couse, adapt the `architecture` according to your infrastructure.

The `lxc image import` command expects the metadata file to be compressed, so execute the following command to compress it : `tar czf metadata.tar.gz metadata.yaml`.

Then, install `lzop` from your package manager (ie. apt) as we need to uncompress the containers. Indeed they are compressed with the .lzo file extension, and we need to execute `lzop -d container-name.tar.lzo` on them to get the `.tar` that we can now feed to `lxc`.

Now we are ready to import the image. It can take a while, so be patient ;)
The CLI is displaying the percentage of the import process, as shown below :

    nicolas@lxc:~/lxd/dump$ lxc image import ../metadata.tar.gz vzdump-lxc-110-2021_12_08-01_41_01.tar
    Transferring image: 33% (185.93MB/s) 

And then when it's over, we get a `fingerprint` in return :

    nicolas@lxc:~/lxd/dump$ lxc image import ../metadata.tar.gz vzdump-lxc-110-2021_12_08-01_41_01.tar
    Image imported with fingerprint: fed1946c0ebd8f8742bb07e9ad18d419fc28e84ad724281a1b1b23adad4780e

We can now `launch` the imported LXC container, and again wait a few moment for the process to finish. As you can see below, we specify the fingerprint and then the name of this new container.

    nicolas@lxc:~/lxd/dump$ lxc launch fed1946c0ebd8f8742bb07e9ad18d419fc28e84ad724281a1b1b23adad4780e my-container
    Creating my-container
    Retrieving image: Unpack: 100% (727.27MB/s)
    Starting my-container                     


You can now use the `lxc list` command to get the list of LXC containers on the system.

    nicolas@lxc:~/lxd/dump$ lxc list
    +----------------+---------+-------------------+------+-----------+-----------+
    |      NAME      |  STATE  |       IPV4        | IPV6 |   TYPE    | SNAPSHOTS |
    +----------------+---------+-------------------+------+-----------+-----------+
    | my-container   | RUNNING | 10.8.0.228 (eth0) |      | CONTAINER | 0         |
    +----------------+---------+-------------------+------+-----------+-----------+

The container is running, and got an IP address from my DHCP server on my network, it looks good !
But the thing it, I'd like my containers to have the same IP address as they had on proxmox, because they all have at least 2 DNS records (A and PTR). The addresses were not statically configured on each containers, though. They ask for it via DHCP, were a revervation has been made for each MAC address.

In short, we will retrieve those MAC addresses from our DHCP server (my router in this case), and set them in the config file of each container. Like this, when they boot up and send out a DHCP Discover with their MAC, the DHCP server will offer the reserved IP address, each time.

I extracted the MAC from my DHCP server, and copied them in a text file in the following format : `container-name MAC`

    ns1 4A:1A:7E:15:AD:38
    ns2 0A:B3:12:36:0C:BD
    nginx E2:BA:20:DE:A8:9A
    db1 96:63:A6:5D:2F:0A
    [...]

As for the container name, I have to precise that it is the name of the container as it was on proxmox. Do you remember the log file stored next to the backup archive ? Yeah that's the one. So be sure to reference the same name here.

We'll use the text file with all the MAC addresses a bit later. Now let's look at how we can manually modify the MAC of a container. It's pretty simple actually, 3 commands are needed in our case to get to the final result.

    lxc stop my-container
    lxc config set my-container volatile.eth0.hwaddr 001122334455
    lxc start my-container

> Of course, replace the MAC address with the one you got from your DHCP server :)

After finishing its boot process, the container should be in a running state, with the same IP address than the container on proxmox had initially. Perfect !

## Automate it because we can

Now that we know all the steps that we have to take to restore a proxmox container to a standalone LXD instance, we can automate the process.
I've written a short bash script to do the work for us. Mostly because bash is not a language I am fluent writing scripts in, so this was a nice opportunity.

Head over to my [github repo, lxc-backup-restore](https://github.com/nicosalvadore/lxd-backup-restore) to get a look at the full script.

Thank you if you kept reading until the end, it was a long read for such a small script !
Please do not hesitate to suggest enhancements or submit a PR on the repo.

See you next time :)