---
layout: post
title: Create an OPNsense Local Repository
published: false
tags: [OPNsense,'Isolated Network']
youtubeId: 8kHbFiHTb4Q
---

How to mirror an update repository on a Local Area Network for the OPNsense open source router.

{% include youtubePlayer.html id=page.youtubeId %} <!-- embedded youtube player, remove if no yt video accompanies the post -->

## Background & Introduciton
*For clarity, the terms update repository (repo) and mirror are used synonymously.*

One of the differences between the OPNsense and pfSense open source routers is the ability to host a local mirror repository and the ability to update without internet access.
With pfSense, the update procedure in an isolated network (network without access to the internet) is simply a configuration backup, install the new version, and restore the configuration from backup.
Some may not think that this is too much of a burden, because pfSense updates are released fairly infrequently, but many isolated networks already have a local repository for updating other systems, such as Linux, so having the ability to update router software in the same manner as other systems help streamline the overall patching/updating process.

OPNsense, has a built-in mechanism for choosing an update mirror from a list, or specifying your own. This allows for an administrator to sync from a repo on the internet, transfer the data to an isolated network, and publish the repo to use for systems to use on the LAN.

## Syncing from Internet Repo

Since the online repos use a colon ( `:` ), as well as some symlinks, the uncommon rsync parameters will be used.
To simplify the sync, I've decided to convert symlinks to...




## Transfer Data to Isolated Network



## Create Repo

### Install & Configure NGINX Web Server






## Configure OPNsense Mirror Location

## Update

### Updating via the Web UI

### Updating via the Console Menu

### Updating via CLI with opnsense-update



![_config.yml]({{ site.baseurl }}/images/deployova/invaliduri.jpg) <!-- embedded image url -->


[Install the PowerCLI Module]({% link _posts/2021-12-2-GettingStartedwithVMwarePowerCLI.md %}) <!-- internal link -->

citation[^1]

[^1]: https://example.com
