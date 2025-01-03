---
date: '2025-01-03T15:24:43+02:00'
draft: false
title: 'Cum sa folosesti GPG pentru a cripta/decripta fisiere'
tags: ["gpg", "linux", "cryptography"]
categories: ["tech"]
showToc: false
TocOpen: false
author: "Me"
cover:
    image: img/pexels-pixabay-207580.jpg
    alt: 'Criptografie'
---

GnuPG foloseste criptografia cu chei publice astfel incat utilizatorii sa poata comunica in siguranta. Intr-un sistem cu chei publice, fiecare utilizator are o pereche de chei formata dintr-o cheie privata si o cheie publica. Cheia privata a unui utilizator este pastrata secreta si nu trebuie dezvaluita niciodata. Cheia publica poate fi oferita oricui doreste utilizatorul sa comunice.

### Instalare si generare pereche de chei

- instalare pe Debian / Ubuntu

```shell
sudo apt-get install gnupg
```

- generarea unei perechi de chei
- vom folosi ca exemplu adrese de email de genul `johndoe@example.com`

```shell
gpg --full-generate-key
```

### Schimbul de chei (export / import)

- exportul unei chei public in format binar

```shell
gpg --output johndoe.gpg --export johndoe@example.com
```

- sau ACII

```shell
gpg --armor --output johndoe.gpg.pub --export johndoe@example.com
```

- importul unei chei publice
```shell
gpg --import janedoe.gpg
gpg --list-keys
```

- validarea unei chei publice importate (manual, daca este cazul)
```
gpg --edit-key blake@cyb.org
#afiseaza amprenta
fpr 
#semneaza cheia
sign
#verifica cheia
check
```



### Criptarea si decriptarea documentelor (pentru un anumit destinatar)
- criptare
```shell
gpg --output doc.gpg --encrypt --recipient janedoe@example.com doc.txt
```

- decriptare
```shell
gpg --output doc-dec.txt --decrypt doc.gpg
```

