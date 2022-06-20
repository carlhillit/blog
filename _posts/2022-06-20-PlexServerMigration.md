---
layout: post
title: Migrating Plex Media Server
published: true
tags: [Linux,Plex]
---


## Intro

I have current NAS/Plex server running on Ubuntu 21.10 with ZFSonLinux, but I'd like to install Fedora instead.

Major steps for this project are:

1. [Backup Plex config to tarball](#backup-plex)
2. [Export the ZFS pool](#export-the-zfs-pool)
3. [Install Fedora](#install-fedora)
4. [Install ZFS](#install-zfs)
5. [Import ZFS Pool](#import-zfs-pool)
6. [Install Plex Media Server](#install-plex-media-server)
7. [Import Plex data from backup](#import-plex-data-from-backup)
8. [Enable Service and Create Firewall Rule](#enable-service-and-create-firewall-rule)

## Backup Plex

### Prepare for Backup

Disable auto emptying of Trash

### Create Archive of Plex Data

I'll first need to find my Plex data directory. Since the [documentation](https://support.plex.tv/articles/202915258-where-is-the-plex-media-server-data-directory-located/) gives the location for Ubuntu as `/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/` I'll do a quick search to confirm.

````bash
sudo find / -name 'Plex Media Server' -type d
````

The result:

>/var/lib/plexmediaserver/Library/Application Support/Plex Media Server

Now that the location is confirmed, I'll make a tarball of the data to make a backup.

````bash
cd "/var/lib/plexmediaserver/Library/Application\ Support/Plex\ Media\ Server"
tar -cf plexdata.tar 
````

`tar` is the command, `c` is to *create* an archive and `f` is the *file* in which to create. So the command breaks down to

`tar create file nameofarchive.tar dir/to/archive`

*Optional. Note that compression is CPU intensive and may take a while.*
The newly create tar file is a bit big (23G), so I'm going to compress it with bzip2:

````bash
bzip2 plexdata.tar
````

Copy the backup to another drive, or to a directory on the zpool.

````bash
cp plexdata.tar /netstor/backups/
````


## Export the ZFS Pool
~~While this is not 100% necessary, as you can use `--force` when importing the ZFS pool on the newly installed OS, it does make it easier and reduces the risk of complications.~~

*not necessary to export the zpool. Once the new OS is installed, zpool import does not require `--force` if you shutdown the prior OS gracefully*

## Install Fedora

I'll use the anaconda installer to install Fedora, taking care not to install the OS onto a ZFS drive.

## Install ZFS

ZFS install instructions specific to Fedora are found [here](https://openzfs.github.io/openzfs-docs/Getting%20Started/Fedora/index.html)
1. Add repo:
`dnf install -y https://zfsonlinux.org/fedora/zfs-release$(rpm -E %dist).noarch.rpm`
2. Install kernel headers:
`dnf install -y kernel-devel`
3. Load kernel module:
`modprobe zfs`

## Import ZFS Pool

Once ZFS is installed, I can list the available ZFS pools to import[^1]:
`zpool import`

>
>      pool: tank
>         id: 11809215114195894163
>       state: ONLINE
>      action: The pool can be imported using its name or numeric identifier.
>      config:
> 
>             tank        ONLINE
>               mirror-0  ONLINE
>                 c1t0d0  ONLINE
>                 c1t1d0  ONLINE
> 

Note the "action" bit. The pool can be imported using its name or numeric identifier.

`zpool import netstor`

## Install Plex Media Server

Add plex repo

Install `plexmediaserver` package

````bash
dnf install -y plexmediaserver
````
## Import Plex Data From Backup

Untar the backup into the appropriate directory

````bash

cd "/var/lib/plexmediaserver/Library/Application\ Support/Plex\ Media\ Server/"

cp /netstor/backup/plexdata.tar ./

tar -xf plexdata.tar 

````

Before I created (`c`) a file (`f`):

````bash
tar -cf plexdata.tar
````

Now I'm extracting (`x`) a file (`f`)

````bash
tar -xf plexdata.tar
````

## Enable Service and Create Firewall Rule

Enable the Plex service to autostart upon bootup and start the service immediately:

````bash
systemctl enable --now plexmediaserver
````

Don't forget to create a firewall rule:

````bash
firewall-cmd --permanent --add-service=plex
firewall-cmd --reload
````

That should be it. I now can enjoy my new Plex server.

[^1]: https://docs.oracle.com/cd/E19253-01/819-5461/gazru/index.html
