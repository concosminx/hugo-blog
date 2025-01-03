---
date: '2025-01-03T15:24:43+02:00'
draft: false
title: 'How to use GPG to encrypt/decrypt files'
tags: ["gpg", "linux", "cryptography"]
categories: ["tech"]
showToc: false
TocOpen: false
author: "Me"
cover:
    image: img/pexels-pixabay-207580.jpg
    alt: 'Cryptography'
---

GnuPG uses public-key cryptography so that users may communicate securely. In a public-key system, each user has a pair of keys consisting of a private key and a public key. A user's private key is kept secret; it need never be revealed. The public key may be given to anyone with whom the user wants to communicate.

### Installation and key pair generation

- install GnuPG on Debian / Ubuntu

```shell
sudo apt-get install gnupg
```

- generate a new GPG key pair
- let's use as an example the email address `johndoe@example.com`

```shell
gpg --full-generate-key
```

### Exchanging keys

- exporting a public key in a binary format

```shell
gpg --output johndoe.gpg --export johndoe@example.com
```

- or ASCII 

```shell
gpg --armor --output johndoe.gpg.pub --export johndoe@example.com
```

- importing a public key
```shell
gpg --import janedoe.gpg
gpg --list-keys
```

- validate the imported key (manual, if needed)
```
gpg --edit-key blake@cyb.org
#display the fingerprint
fpr 
#sign the key 
sign
#check the key
check
```



### Encrypting and decrypting documents
- encrypting 
```shell
gpg --output doc.gpg --encrypt --recipient janedoe@example.com doc.txt
```

- decrypting 
```shell
gpg --output doc-dec.txt --decrypt doc.gpg
```

