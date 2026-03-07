---
date: '2026-03-07T12:00:00+02:00'
draft: false
title: 'Cum creezi un utilizator nou cu drepturi sudo pe Ubuntu'
tags: ["linux", "ubuntu", "sudo", "user-management"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Ghid pas cu pas pentru crearea unui utilizator cu privilegii sudo pe Ubuntu."
cover:
    image: img/ubuntu-sudo-user.png
    alt: 'Utilizator sudo pe Ubuntu'
---

## Introducere

Acordarea drepturilor sudo unui utilizator îi permite să efectueze sarcini administrative fără a se conecta ca root. Acest ghid explică cum să creezi un utilizator nou cu acces sudo pe Ubuntu, urmând bune practici de securitate și administrare.

## Pasul 1 — Conectează-te la server

Conectează-te la serverul Ubuntu ca utilizator root (sau un utilizator cu drepturi sudo):

```bash
ssh root@adresa_ip_server
```

## Pasul 2 — Creează un utilizator nou

Creează un cont nou de utilizator (înlocuiește `utilizatornou` cu numele dorit):

```bash
adduser utilizatornou
```

Vei fi rugat să setezi o parolă și să completezi informații opționale despre utilizator. Poți apăsa ENTER pentru a accepta valorile implicite.

## Pasul 3 — Acordă drepturi sudo

Adaugă noul utilizator în grupul `sudo`:

```bash
usermod -aG sudo utilizatornou
```

Pe Ubuntu, utilizatorii din grupul `sudo` pot rula comenzi ca root folosind `sudo`.

## Pasul 4 — Testează accesul sudo

Comută pe noul utilizator:

```bash
su - utilizatornou
```

Testează sudo rulând o comandă privilegiată (de exemplu, listarea directorului `/root`):

```bash
sudo ls -la /root
```

La prima utilizare a comenzii `sudo`, vei fi rugat să introduci parola noului utilizator. Dacă totul este corect, comanda se va executa cu privilegii de root.

## Diferența dintre `sudo` și `su`

- `sudo` permite unui utilizator autorizat să ruleze o comandă ca alt utilizator (implicit root), autentificându-se cu propria parolă. Fiecare comandă este logată pentru audit.
- `su` comută pe un alt cont de utilizator și necesită parola acelui utilizator. Deschide o nouă sesiune shell ca acel utilizator.

## Bune practici

- **Principiul celor mai puține privilegii:** Acordă drepturi sudo doar utilizatorilor care au nevoie.
- **Limitează comenzile sudo:** Pentru securitate sporită, restricționează comenzile ce pot fi rulate cu sudo editând fișierul `/etc/sudoers` cu `visudo`.
- **Dezactivează login-ul root:** După crearea unui utilizator cu sudo, poți dezactiva login-ul direct ca root pentru o securitate mai bună:

```bash
sudo passwd -l root
```

## Referințe

- [DigitalOcean: Cum creezi un utilizator nou cu drepturi sudo pe Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-ubuntu)
- [Cum editezi fișierul Sudoers](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file)
- [Cum adaugi și ștergi utilizatori pe Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-20-04)

---

*Acest articol este bazat pe tutorialul DigitalOcean menționat mai sus. Respectă întotdeauna politicile de securitate ale organizației tale când gestionezi utilizatori și accesul sudo.*
