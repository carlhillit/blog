---
layout: post
title: Time Machine to Linux Server REDUX
published: true
tags: [Linux,MacOS,Time Machine]
---
A better way to store MacOS Time Machine backups on a Linux server with ZFS.

I needed a better way to backup my Macbook. Time Machine backups on a local disk are simple and easy, but having a disk constantly attached to it occupies one of my Thunderbolt ports and leaves a bit of clutter on my desk.
Backing up to a network server is much more convenient and if the server has ZFS, then that setup opens the door to many more ways to ensure my precious data is secure and available.

In this blog post, I'll be using Almalinux 9.5 as my backup server and MacOS 15.1.

I'll also skip the steps for installing OpenZFS as that largely depends on the distribution.

## Steps

1. [Partition and Format Drive](#partition-and-format-drive)
2. [Create Backup Directory and Assign Permissions](#create-backup-directory-and-assign-permissions)
3. [Install Packages](#install-packages)
4. [Enable Services](#enable-services)
5. [Edit Netatalk Config](#edit-netatalk-config)
6. [Add Firewall Rules](#add-firewall-rules)
7. [Setup Time Machine](#setup-time-machine)


## Create ZFS Pool

List devices by ID. The IDs never change, even if you unplug and rearrange your disks inside the server, unlike `/dev/sdx`.
Additionally, the disk ID usually has the make, model, and serial number which helps easily identify the disk should it need to be inspected, reconnected, or replaced.

```
ls -l /dev/disk/by-id
```

```
zpool create stache raidz1 `
/dev/disk/by-id/ata-Samsung_SSD_860_EVO_1TB_S5B3NR0R200674X `
/dev/disk/by-id/ata-Samsung_SSD_860_QVO_1TB_S59HNG0N501226F `
/dev/disk/by-id/ata-Samsung_SSD_870_QVO_1TB_S5SVNF0R344595Y
```

```console
[root@snas ~]# zpool status
  pool: stache
 state: ONLINE
config:

	NAME                                           STATE     READ WRITE CKSUM
	stache                                         ONLINE       0     0     0
	  ata-Samsung_SSD_860_EVO_1TB_S5B3NR0R200674X  ONLINE       0     0     0
	  ata-Samsung_SSD_860_QVO_1TB_S59HNG0N501226F  ONLINE       0     0     0
	  ata-Samsung_SSD_870_QVO_1TB_S5SVNF0R344595Y  ONLINE       0     0     0

errors: No known data errors
```

## Create ZFS Dataset
```shell
zfs create -o atime=off stache/backup
```

## Set ZFS Quota and Reservation

Apple recommends a backup target with twice the capacity of the Mac's local storage.
Since my Macbook's SSD is 500GB, I'll need 1TB of space on the server.
I also share the server storage with other services on my network.
As a result, I will need a way to ensure that Time Machine does not either consume all of the space and that it has enough space.
I'll set both a quota and a reservation on the stache/backup dataset of 1TB.

```shell
zfs set quota=1T stache/backup
zfs set reservation=1T stache/backup
```

See all non-default ZFS options:
```console
[root@snas ~]# zfs get all stache/backup | grep local
stache/backup  quota                 1T                     local
stache/backup  reservation           1T                     local
stache/backup  atime                 off                    local
```

## Add a user account for Time Machine
For security reasons, I'll make a dedicated user account for the Time Machine backups and give it a password
```console
[root@snas ~]# useradd timemachine
[root@snas ~]# passwd timemachine
Changing password for user timemachine.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[root@snas ~]#
```

This account will not be an interactive user, which means that it cannot be used to login to the server.
```console
[root@snas ~]# usermod -s /sbin/nologin timemachine

[root@snas ~]# grep timemachine /etc/passwd
timemachine:x:1002:1003::/home/timemachine:/sbin/nologin
```


## Create Backup Directory and Assign Permissions

I'll make a timemachine directory right in the root and assign it permissions. When I configure the Time Machine client, I'll give it the credentials for the `karl` account, so I'll set permissions on the directory for read,write,execute for the user and group, and nothing for others.

````bash
mkdir /stache/backup
chown timemachine:timemachine /stache/backup
chmod 770 /stache/backup
````



## Install Packages

````bash
dnf install -y netatalk avahi
````

## Netatalk Config
Add a section for your Time Machine in the `/etc/netatalk/afp.conf` config.

```console
[NAS TimeMachine Target]
path = /stache/backup
time machine = yes
```

## Enable Services
Start the netatalk and avahi-daemon services and enable them to start upon boot.

```shell
systemctl enable --now netatalk avahi-daemon
```

## Add Firewall Rules

Open ports for mDNS, AFP, and avahi-daemon.

```shell
firewall-cmd --permanent --add-service={afp,mdns}
firewall-cmd --permanent --add-port={56602,57850}/udp
firewall-cmd --reload
```

## Setup Time Machine

Now, on your Mac, you should be able to open the Time Machine settings in System Preferences and use Select Disk… to pick your new Time Machine backup drive.

![_config.yml]({{ site.baseurl }}/images/linuxservertimemachine/selectbackupdisk.jpg)

The share that's configured in the afp.conf file should be available to select. Select the NAS Time Machine Target on the server and click "Use Disk" and then "Connect"

![_config.yml]({{ site.baseurl }}/images/linuxservertimemachine/usedisk.jpg)

When prompted, enter a username and password of an account on the linux server that has permissions to the /stache/backup directory.

![_config.yml]({{ site.baseurl }}/images/linuxservertimemachine/servercreds.jpg)

Once you've selected "Backup Automatically" the Time Machine client will automatically take a backup every hour, as long as it's connected to the server.

![_config.yml]({{ site.baseurl }}/images/linuxservertimemachine/autobackup.jpg)

Unfortunately, there isn't any other option to configure if you wish to manage the backups. Backups are stored according to the scheme below:

>
>* Local snapshots as space permits
>
>* Hourly backups for the past 24 hours
>
>* Daily backups for the past month
>
>* Weekly backups for all previous months
>
>* The oldest backups are deleted when the disk is full
>

* As of MacOS 15.1