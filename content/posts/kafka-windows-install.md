---
date: '2025-12-27T10:00:00+02:00'
draft: false
title: 'Quick Kafka Setup on Windows with UI'
tags: ["kafka", "windows", "messaging", "java", "streaming"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
cover:
    image: img/kafka-cover.png
    alt: 'Apache Kafka on Windows'
    #caption: 'Apache Kafka streaming platform'
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
#hidemeta: false
#comments: false
description: "Quick guide to install Apache Kafka on Windows and set up Kafka UI for easy management"
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

## Why Kafka?

Apache Kafka is a distributed streaming platform that's become essential for real-time data processing. Whether you're building microservices, handling event streaming, or need a robust message broker, Kafka delivers high-throughput, fault-tolerant messaging.

## Prerequisites

- **Java 17+** - see official [docs](https://kafka.apache.org/41/operations/java-version/)

Verify Java installation:
```cmd
java -version
```

## Quick Setup

### 1. Download Kafka

1. Visit [Apache Kafka downloads](https://kafka.apache.org/downloads)
2. Download the binary (latest stable version)
3. Extract to `C:\kafka` (avoid paths with spaces)

### 2. Edit the Logs Location 

Go to Kafka directory `C:\kafka\config` and edit `server.properties` - replace `log.dirs=/tmp/kraft-combined-logs` with `log.dirs=C:\kafka\logs`

### 3. Generate a UUID 

Open Command Prompt as Administrator and navigate to `C:\kafka\bin\windows`, then run:

```cmd
kafka-storage.bat random-uuid
```

Copy the generated UUID, then run:
```cmd
kafka-storage.bat format -t [PASTE_UUID_HERE] --standalone -c C:\kafka\config\server.properties
```

### 4. Start the Kafka Server

In the same command prompt, start Kafka:
```cmd
kafka-server-start.bat C:\kafka\config\server.properties
```

### 5. Test Installation

Open a **new** Command Prompt window (keep Kafka server running) and navigate to `C:\kafka\bin\windows`:

**Create a test topic:**
```cmd
kafka-topics.bat --create --topic test --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

**List topics:**
```cmd
kafka-topics.bat --list --bootstrap-server localhost:9092
```

**Start a producer (send messages):**
```cmd
kafka-console-producer.bat --topic test --bootstrap-server localhost:9092
```

**Open another Command Prompt and start a consumer:**
```cmd
kafka-console-consumer.bat --topic test --from-beginning --bootstrap-server localhost:9092
```

Type messages in the producer window and watch them appear in the consumer window!

## Kafka UI Setup

### Option 1: Docker (Recommended)

If you have Docker Desktop:

```cmd
docker run -d --name kafka-ui -p 8080:8080 -e KAFKA_CLUSTERS_0_NAME=local -e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=host.docker.internal:9092 provectuslabs/kafka-ui:latest
```

### Option 2: JAR Download

1. Download [Kafka UI JAR](https://github.com/provectus/kafka-ui/releases)
2. Run with:
```cmd
java -jar kafka-ui-api-v0.7.2.jar --kafka.clusters.0.name=local --kafka.clusters.0.bootstrapServers=localhost:9092
```

## Access the UI

Open your browser and navigate to: `http://localhost:8080`

You'll see:
- ✅ Topics overview
- ✅ Consumer groups
- ✅ Brokers status
- ✅ Message browsing
- ✅ Schema registry (if configured)



## Troubleshooting

**Port already in use?**

```cmd
netstat -ano | findstr :9092
taskkill /PID <PID> /F
```

**Java not found?**

Set `JAVA_HOME` environment variable to your JDK installation path.