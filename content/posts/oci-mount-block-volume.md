---
date: '2025-02-16T16:27:00+02:00'
draft: false
title: 'OCI - Attach and mount a block volume'
tags: ["linux", "ubuntu", "oci"]
categories: ["tech"]
showToc: false
TocOpen: false
author: "Me"
cover:
    image: img/oci-mount-block-volume.png
    alt: 'Block Volume'
    #caption: 'Code on black IDE theme'
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
#hidemeta: false
#comments: false
#description: "Desc Text."
#canonicalURL: "https://canonical.url/to/page"
#disableHLJS: true # to disable highlightjs
#disableShare: false
#disableHLJS: false
#hideSummary: false
#searchHidden: true
#ShowReadingTime: true
#ShowBreadCrumbs: true
#ShowPostNavLinks: true
#ShowWordCount: true
#ShowRssButtonInSectionTermList: true
#UseHugoToc: true
---

Follow the Oracle [guide](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/connectingtoavolume_topic-Connecting_to_iSCSIAttached_Volumes.htm) to attach the newly created volume and these steps:

#### Identify the New Drive (e.g., `/dev/sdb`)
```bash
sudo fdisk -l
```

#### Create a Partition (If Needed)
```bash
sudo fdisk /dev/sdb
```

* Press `n` to create a new partition.
* Choose partition type (`p` for primary)
* Select the default values or specify size.
* Press `w` to write changes.


#### Format the Partition
```bash
sudo mkfs.ext4 /dev/sdb1
```

#### Create a Mount Point
```bash
sudo mkdir -p /mnt/mydrive
```

#### Mount the Drive
```bash
sudo mount /dev/sdb1 /mnt/mydrive
```

#### Auto-Mount on Boot

- Get the UUID of the partition
```bash
sudo blkid /dev/sdb1
```

- Edit the `/etc/fstab` file
```bash
sudo nano /etc/fstab
```

- Add this line at the end
```bash
UUID=<your-uuid>  /mnt/mydrive  ext4  defaults,noatime,_netdev  0  2
```

- Save and exit

- Test with
```bash
sudo mount -a
```