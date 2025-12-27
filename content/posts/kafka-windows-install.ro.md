---
date: '2025-12-27T10:00:00+02:00'
draft: false
title: 'Configurare Rapida Kafka pe Windows cu UI'
taguri: ["kafka", "windows", "messaging", "java", "streaming"]
categorii: ["tech"]
showToc: true
TocOpen: false
author: "Me"
cover:
    image: img/kafka-cover.png
    alt: 'Apache Kafka pe Windows'
    #caption: 'Apache Kafka streaming platform'
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
#hidemeta: false
#comments: false
description: "Ghid rapid pentru instalarea Apache Kafka pe Windows si configurarea Kafka UI pentru management usor"
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

## De ce Kafka?

Apache Kafka este o platforma de streaming distribuita care a devenit esentiala pentru procesarea datelor in timp real. Fie ca construiesti microservicii, gestionezi streaming de evenimente sau ai nevoie de un message broker robust, Kafka ofera mesagerie cu throughput ridicat si toleranta la erori.

## Cerinte Preliminare

- **Java 17+** - vezi documentatia oficiala [docs](https://kafka.apache.org/41/operations/java-version/)

Verifica instalarea Java:
```cmd
java -version
```

## Configurare Rapida

### 1. Descarca Kafka

1. Viziteaza [Apache Kafka downloads](https://kafka.apache.org/downloads)
2. Descarca binarele (ultima versiune stabila)
3. Extrage in `C:\kafka` (evita path-urile cu spatii)

### 2. Editeaza Locatia Log-urilor

Mergi in directorul Kafka `C:\kafka\config` si editeaza `server.properties` - inlocuieste `log.dirs=/tmp/kraft-combined-logs` cu `log.dirs=C:\kafka\logs`

### 3. Genereaza un UUID

Deschide Command Prompt ca Administrator si navigheaza la `C:\kafka\bin\windows`, apoi ruleaza:

```cmd
kafka-storage.bat random-uuid
```

Copiaza UUID-ul generat, apoi ruleaza:
```cmd
kafka-storage.bat format -t [LIPESTE_UUID_AICI] --standalone -c C:\kafka\config\server.properties
```

### 4. Porneste Serverul Kafka

In acelasi command prompt, porneste Kafka:
```cmd
kafka-server-start.bat C:\kafka\config\server.properties
```

### 5. Testeaza Instalarea

Deschide o **noua** fereastra Command Prompt (lasa serverul Kafka sa ruleze) si navigheaza la `C:\kafka\bin\windows`:

**Creaza un topic de test:**
```cmd
kafka-topics.bat --create --topic test --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

**Listeaza topic-urile:**
```cmd
kafka-topics.bat --list --bootstrap-server localhost:9092
```

**Porneste un producer (trimite mesaje):**
```cmd
kafka-console-producer.bat --topic test --bootstrap-server localhost:9092
```

**Deschide alt Command Prompt si porneste un consumer:**
```cmd
kafka-console-consumer.bat --topic test --from-beginning --bootstrap-server localhost:9092
```

Scrie mesaje in fereastra producer si urmareste-le cum apar in fereastra consumer!

## Configurare Kafka UI

### Optiunea 1: Docker (Recomandata)

Daca ai Docker Desktop:

```cmd
docker run -d --name kafka-ui -p 8080:8080 -e KAFKA_CLUSTERS_0_NAME=local -e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=host.docker.internal:9092 provectuslabs/kafka-ui:latest
```

### Optiunea 2: Descarcare JAR

1. Descarca [Kafka UI JAR](https://github.com/provectus/kafka-ui/releases)
2. Ruleaza cu:
```cmd
java -jar kafka-ui-api-v0.7.2.jar --kafka.clusters.0.name=local --kafka.clusters.0.bootstrapServers=localhost:9092
```

## Acceseaza UI-ul

Deschide browser-ul si navigheaza la: `http://localhost:8080`

Vei vedea:
- ✅ Prezentare generala a topic-urilor
- ✅ Grupuri de consumatori
- ✅ Statusul broker-ilor
- ✅ Navigarea prin mesaje
- ✅ Schema registry (daca e configurat)



## Depanare

**Port deja in folosinta?**

```cmd
netstat -ano | findstr :9092
taskkill /PID <PID> /F
```

**Java nu e gasita?**

Seteaza variabila de mediu `JAVA_HOME` la path-ul instalarii JDK.