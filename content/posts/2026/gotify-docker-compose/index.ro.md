---
date: '2026-03-29T12:00:00+02:00'
draft: false
title: 'Gotify cu Docker Compose (Ghid rapid)'
tags: ["gotify", "docker", "compose", "notifications", "self-hosted", "linux"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
cover:
  image: gotify-docker-compose-cover.png
  alt: 'Tablou de bord notificări Gotify'
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
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
#ShowRssButtonInSectionTermList: true
#UseHugoToc: true
---

Gotify este un serviciu de notificări self-hosted, open-source. Expune un API prin care aplicațiile tale pot trimite notificări către telefon (Android sau iOS). În acest ghid îl configurăm cu Docker Compose, securizăm utilizatorul implicit și testăm notificările.

## Configurare Docker Compose

Creează un folder (de exemplu `~/gotify`) și adaugă următorul `docker-compose.yml`:

```yaml
version: "3"
services:
  gotify:
    image: gotify/server
    restart: unless-stopped
    container_name: gotify
    ports:
      - 8989:80
    environment:
      - GOTIFY_DEFAULTUSER_PASS=${GOTIFY_DEFAULTUSER_PASS}
    volumes:
      - "./gotify_data:/app/data"

networks:
  default:
    name: my-main-net
    external: true
```

Adaugă un fișier de mediu numit `.env` (sau păstrează-l ca `.env.example` în git și copiază-l ca `.env` pe server):

```env
GOTIFY_DEFAULTUSER_PASS=change-me
```

> **Tip:** alege o parolă puternică pentru `GOTIFY_DEFAULTUSER_PASS`. Poți adăuga ulterior mai multe variabile, dar acesta este minimul pentru o configurare sigură.

Pornește containerul:

```bash
docker compose up -d
```

Deschide interfața web la `http://<ip-server>:8989`. Autentifică-te cu utilizatorul **admin** și parola setată în `GOTIFY_DEFAULTUSER_PASS`.

## Creează o aplicație

1. Apasă **APPS** în bara de sus.
2. Selectează **CREATE APPLICATION**.
3. Dă-i un nume (exemplu: `ASimpeTest`) și lasă descrierea goală dacă vrei.
4. Salvează și copiază token-ul aplicației — îl vei folosi la trimiterea notificărilor.

## Instalează aplicația mobilă

1. Instalează **Gotify** din Play Store sau App Store.
2. Introdu URL-ul Gotify și apasă **Check URL**.
3. Autentifică-te cu utilizatorul creat (sau `admin`).
4. Alege un nume pentru client și permite notificările.

Ar trebui să vezi o listă goală de notificări, gata pentru test.

## Testează notificările cu curl

Folosește token-ul aplicației din interfața web:

```bash
curl "https://<your-server-url>/message?token=<your-application-token>" \
  -F "title=Curl Test" \
  -F "message=This is a test with curl and gotify" \
  -F "priority=5"
  -k
```

Telefonul ar trebui să primească notificarea imediat. Câmpul `priority` este opțional; dacă îl omiți, se folosește prioritatea implicită a aplicației.

---

Sursă: [Documentația Gotify](https://gotify.net/docs/)
