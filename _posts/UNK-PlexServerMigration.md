---
layout: post
title: Migrating Plex Media Server
published: false
tags: [Linux,Plex]
---


## Intro

I have current NAS/Plex server running on Ubuntu 21.10 with ZFSonLinux, but I'd like to install Fedora instead.

Major steps for this project are:

1. [Backup Plex config to tarball](#backup-plex)
2. [Export the ZFS pool](#export-the-zfs-pool)
3. [Install Fedora](#install-fedora)
4. Install ZFS
5. Import ZFS pool
6. Install Plex Media Server
7. Import Plex data from backup

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
cd "/var/lib/plexmediaserver/Library/Application\ Support/Plex\ Media\ Server
tar -cf plexdata.tar 
````

`tar` is the command, `c` is to *create* an archive and `f` is the *file* in which to create. So the command breaks down to

tar create file nameofarchive.tar dir/to/archive
<!--
The newly create tar file is a bit big (23G), so I'm going to compress it with bzip2:

````bash
bzip2 plexdata.tar
````
-->

## Export the ZFS Pool

While this is not 100% necessary, as you can use `--force` when importing the ZFS pool on the newly installed OS, it does make it easier and reduces the risk of complications.

````bash
karl@nas:~$ zpool list
NAME      SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
netstor  58.2T  43.3T  14.9T        -         -     8%    74%  1.00x    ONLINE  -
````

## Install Fedora

I'll use the anaconda installer to install Fedora 