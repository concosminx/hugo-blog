---
date: '2026-03-14T09:00:00+02:00'
draft: false
title: 'Install RustFS with Docker (Quick Guide)'
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

RustFS is a fast, S3-compatible object storage system. This quick guide shows the **Docker** install path, keeping only the full parameter configuration example and the official Docker Compose links. I also added a short section for opening access from a VM running in Google Cloud.

## Prerequisites

- Docker 20.10+ installed and able to pull images
- A host directory for RustFS data (example: `/mnt/rustfs/data`)
- Open ports **9000** (API) and **9001** (console) on the host

> **Note on permissions**: the container runs as user ID `10001`. If you mount a host directory with `-v`, make sure that directory is owned by `10001` to avoid permission errors.

## Pull the RustFS image

```bash
docker pull rustfs/rustfs:latest
```

## Run RustFS (complete parameter configuration example)

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

Once the container is running, open `http://<host-ip>:9000` and log in with the configured access/secret key (the example uses `rustfsadmin`/`rustfsadmin`).

## Docker Compose files (official)

RustFS provides ready-to-use Compose definitions in the main repository:

- `docker-compose.yml`: https://github.com/rustfs/rustfs/blob/main/docker-compose.yml
- `docker-compose-observability.yml`: https://github.com/rustfs/rustfs/blob/main/docker-compose-observability.yml

## Allow access from a Google Cloud VM

If RustFS runs on a VM in Google Cloud, you must allow inbound traffic to ports **9000** and **9001** on the VM's VPC network.

### Console steps

1. Go to **VPC network** → **Firewall** → **Create firewall rule**.
2. Set **Targets** to **Specified target tags** and add a tag like `rustfs`.
3. Set **Source IPv4 ranges** to your client IP (recommended) or a CIDR range.
4. Under **Protocols and ports**, select **Specified protocols and ports** and enter `tcp:9000,tcp:9001`.
5. Save the rule and apply the **same network tag** (`rustfs`) to your VM.

### Optional: gcloud command

```bash
gcloud compute firewall-rules create allow-rustfs \
  --network=default \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:9000,tcp:9001 \
  --source-ranges=<YOUR_IP_CIDR> \
  --target-tags=rustfs
```

After that, you can reach RustFS at `http://<vm-external-ip>:9000`.

---

Source reference: [RustFS Docker installation docs](https://docs.rustfs.com/installation/docker/)
