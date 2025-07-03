---
date: '2025-02-16T16:27:00+02:00'
draft: false
title: 'OCI - Atașarea si montarea unui volum bloc'
tags: ["linux", "ubuntu", "oci"]
categories: ["tech"]
showToc: false
TocOpen: false
author: "Me"
cover:
    image: img/oci-mount-block-volume.png
    alt: 'Volum'
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

Urmeaza [ghidul](https://docs.oracle.com/en-us/iaas/Content/Block/Tasks/connectingtoavolume_topic-Connecting_to_iSCSIAttached_Volumes.htm) Oracle pentru a atasa volumul nou creat, apoi executa pasii urmatori:

#### Identifica noul disc (de exemplu, `/dev/sdb` )

```bash
sudo fdisk -l
```

#### Creaza o partitie (daca este necesar)
```bash
sudo fdisk /dev/sdb
```

* Apasa `n` pentru a crea o noua partitie.
* Alege tipul partitiei (`p` pentru primara)
* Selecteaza valorile implicite sau specifica o dimensiune.
* Apasa `w` pentru a salva modificarile.


#### Formateaza partitia
```bash
sudo mkfs.ext4 /dev/sdb1
```

#### Creaza un punct de montare
```bash
sudo mkdir -p /mnt/mydrive
```

#### Monteaza discul
```bash
sudo mount /dev/sdb1 /mnt/mydrive
```

#### Montare automata la pornire

- Obtine UUID-ul partitiei
```bash
sudo blkid /dev/sdb1
```

- Editeaza fisierul `/etc/fstab`
```bash
sudo nano /etc/fstab
```

- Adauga linia de mai jos la sfarsit
```bash
UUID=<your-uuid>  /mnt/mydrive  ext4  defaults,noatime,_netdev  0  2
```

- Salveaza fisierul

- Testeaza cu
```bash
sudo mount -a
```
