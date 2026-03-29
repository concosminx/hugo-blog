---
date: '2026-03-14T09:00:00+02:00'
draft: false
title: 'Instalează RustFS cu Docker (Ghid rapid)'
tags: ["rustfs", "docker", "storage", "s3", "cloud", "linux", "gcp"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
cover:
  image: rustfs-with-docker.png
  alt: 'Containerized storage service'
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

RustFS este un sistem de stocare de obiecte rapid și compatibil S3. Acest ghid scurt arată instalarea cu **Docker**, păstrând doar exemplul complet de configurare a parametrilor și link-urile oficiale către fișierele Docker Compose. Am adăugat și un scurt ghid pentru a permite accesul la un VM în Google Cloud.

## Cerințe preliminare

- Docker 20.10+ instalat și capabil să tragă imagini
- Un director local pentru datele RustFS (exemplu: `/mnt/rustfs/data`)
- Porturile **9000** (API) și **9001** (consolă) deschise pe host

> **Notă despre permisiuni**: containerul rulează cu user ID `10001`. Dacă montezi un director local cu `-v`, asigură-te că acel director este deținut de `10001`, altfel vei primi erori de permisiune.

## Descarcă imaginea RustFS

```bash
docker pull rustfs/rustfs:latest
```

## Rulează RustFS (exemplu complet de parametri)

```bash
docker run -d \
  --name rustfs_container \
  -p 9000:9000 \
  -p 9001:9001 \
  -v /mnt/rustfs/data:/data \
  -e RUSTFS_ACCESS_KEY=rustfsadmin \
  -e RUSTFS_SECRET_KEY=rustfsadmin \
  -e RUSTFS_CONSOLE_ENABLE=true \
  -e RUSTFS_SERVER_DOMAINS=example.com \
  rustfs/rustfs:latest \
  --address :9000 \
  --console-enable \
  --server-domains example.com \
  --access-key rustfsadmin \
  --secret-key rustfsadmin \
  /data
```

După ce containerul pornește, deschide `http://<host-ip>:9000` și autentifică-te cu access/secret key-ul configurat (în exemplu: `rustfsadmin`/`rustfsadmin`).

## Fișiere Docker Compose (oficiale)

RustFS oferă definiții Compose gata de folosit în repo-ul principal:

- `docker-compose.yml`: https://github.com/rustfs/rustfs/blob/main/docker-compose.yml
- `docker-compose-observability.yml`: https://github.com/rustfs/rustfs/blob/main/docker-compose-observability.yml

## Permite accesul dintr-un VM Google Cloud

Dacă RustFS rulează pe un VM în Google Cloud, trebuie să permiți trafic inbound pe porturile **9000** și **9001** în rețeaua VPC a VM-ului.

### Pași în consolă

1. Mergi la **VPC network** → **Firewall** → **Create firewall rule**.
2. La **Targets**, alege **Specified target tags** și adaugă un tag precum `rustfs`.
3. La **Source IPv4 ranges**, setează IP-ul tău (recomandat) sau un CIDR.
4. La **Protocols and ports**, selectează **Specified protocols and ports** și introdu `tcp:9000,tcp:9001`.
5. Salvează regula și aplică **același network tag** (`rustfs`) pe VM.

### Opțional: comandă gcloud

```bash
gcloud compute firewall-rules create allow-rustfs \
  --network=default \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:9000,tcp:9001 \
  --source-ranges=<IP_CIDR_TAU> \
  --target-tags=rustfs
```

După aceea, poți accesa RustFS la `http://<vm-external-ip>:9000`.

---

Sursă: [RustFS Docker installation docs](https://docs.rustfs.com/installation/docker/)
