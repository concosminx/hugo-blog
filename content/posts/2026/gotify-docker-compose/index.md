---
date: '2026-03-29T12:00:00+02:00'
draft: false
title: 'Gotify with Docker Compose (Quick Setup)'
tags: ["gotify", "docker", "compose", "notifications", "self-hosted", "linux"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
cover:
  image: gotify-docker-compose-cover.png
  alt: 'Gotify notification dashboard'
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

Gotify is a self-hosted, open-source notification service. It exposes an API that lets your apps send notifications to your phone (Android or iOS). This guide sets it up with Docker Compose, shows how to secure the default user, and how to test notifications.

## Docker Compose setup

Create a folder (for example `~/gotify`) and add the following `docker-compose.yml`:

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

Add an environment file named `.env` (or keep this as `.env.example` in git and copy it to `.env` on the server):

```env
GOTIFY_DEFAULTUSER_PASS=change-me
```

> **Tip:** choose a strong password for `GOTIFY_DEFAULTUSER_PASS`. You can add more environment options later, but this is the minimum for a secure setup.

Start the container:

```bash
docker compose up -d
```

Open the web interface at `http://<your-server-ip>:8989`. Log in with user **admin** and the password you set in `GOTIFY_DEFAULTUSER_PASS`.

## Create an application

1. Click **APPS** in the top bar.
2. Select **CREATE APPLICATION**.
3. Name it (example: `ASimpeTest`) and leave the description empty if you want.
4. Save and copy the application token — you will use it to send notifications.

## Install the mobile app

1. Install **Gotify** from the Play Store or App Store.
2. Enter your Gotify URL and press **Check URL**.
3. Log in with the user you created (or `admin`).
4. Choose a client name and allow notifications.

You should now see an empty notification list, ready for tests.

## Test notifications with curl

Use your application token from the web UI:

```bash
curl "https://<your-server-url>/message?token=<your-application-token>" \
  -F "title=Curl Test" \
  -F "message=This is a test with curl and gotify" \
  -F "priority=5"
  -k
```

The phone should receive the notification immediately. The `priority` field is optional; if omitted, the default app priority is used.

---

Source reference: [Gotify documentation](https://gotify.net/docs/)
