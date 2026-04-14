---
date: '2026-04-14T18:00:00+02:00'
draft: false
title: 'Referință Comenzi Diagnostice JVM'
tags: ["java", "jvm", "debugging", "performance", "monitoring"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Comenzi esențiale de diagnosticare JVM pentru depanarea și monitorizarea aplicațiilor Java"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

Depanarea aplicațiilor Java necesită adesea analiza memoriei heap, a thread dump-urilor și a elementelor interne ale JVM. Acest ghid oferă o referință rapidă la comenzile esențiale de diagnosticare JVM folosind instrumente precum `jmap`, `jstack` și `jcmd`.

## Cerințe prealabile

- Java Development Kit (JDK) instalat
- ID-ul procesului (PID) al aplicației Java care rulează

Pentru a găsi PID-ul procesului Java:
```bash
jps -l
```

## Heap Dump

Generează un heap dump binar pentru analiza memoriei:

```bash
path/to/jdk/jmap -dump:format=b,file=<filename> <pid>
```

**Exemplu:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jmap -dump:format=b,file=heap_dump.hprof 12345
```

Folosește instrumente precum [Eclipse MAT](https://www.eclipse.org/mat/) sau [VisualVM](https://visualvm.github.io/) pentru a analiza fișierul heap dump.

## Rezumat Heap

Afișează configurația și rezumatul utilizării heap-ului:

```bash
path/to/jdk/jmap -heap <pid> > path/to/save_file/heap_summary_ddmmyy_hhmm.log
```

**Exemplu:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jmap -heap 12345 > /var/log/heap_summary_$(date +%d%m%y_%H%M).log
```

Aceasta afișează:
- Configurația heap-ului (dimensiuni minime/maxime)
- Utilizarea curentă a heap-ului
- Informații despre garbage collector

## Histogramă Heap

Generează o histogramă cu numărul de obiecte și utilizarea memoriei:

```bash
path/to/jdk/jmap -histo <pid> > path/to/save_file/heap_histo_ddmmyy_hhmm.log
```

**Exemplu:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jmap -histo 12345 > /var/log/heap_histo_$(date +%d%m%y_%H%M).log
```

Doar pentru obiecte vii (declanșează mai întâi un GC complet):
```bash
jmap -histo:live <pid>
```

## Thread Dump

Captează stack trace-urile thread-urilor pentru detectarea deadlock-urilor și analiza performanței:

```bash
path/to/jdk/jstack -F <pid> > path/to/save_file/thread_dump_ddmmyy_hhmm.log
```

**Exemplu:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jstack -F 12345 > /var/log/thread_dump_$(date +%d%m%y_%H%M).log
```

Flag-ul `-F` forțează un thread dump atunci când procesul nu răspunde la semnalele normale.

**Fără flag-ul de forțare:**
```bash
jstack <pid> > thread_dump.log
```

## Uptime JVM

Verifică cât timp rulează JVM-ul (în secunde):

```bash
path/to/jdk/jcmd <pid> VM.uptime
```

**Exemplu:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jcmd 12345 VM.uptime
```

## Flag-uri VM

Afișează toate flag-urile JVM și valorile lor curente:

```bash
path/to/jdk/jcmd <pid> VM.flags
```

**Exemplu:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jcmd 12345 VM.flags
```

Aceasta afișează:
- Flag-uri ergonomice (setate automat de JVM)
- Flag-uri din linia de comandă (setate explicit)
- Valori implicite

## Declanșarea Garbage Collection

Declanșează manual un garbage collection:

```bash
path/to/jdk/jcmd <pid> GC.run
```

**Exemplu:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jcmd 12345 GC.run
```

**Atenție:** Folosește cu prudență în producție. Forțarea GC poate cauza pauze ale aplicației.

## Uptime JVM (Alternativă Linux)

Verifică uptime-ul procesului folosind comanda Linux `ps`:

```bash
ps -p <pid> -o etime=
```

**Exemplu:**
```bash
ps -p 12345 -o etime=
```

Format ieșire: `[[dd-]hh:]mm:ss`

Pentru informații mai detaliate despre proces:
```bash
ps -p <pid> -o pid,etime,cmd
```

## Bune Practici

- **Automatizează cu timestamp-uri:** Folosește `$(date +%d%m%y_%H%M)` în numele fișierelor pentru timestamp-uri automate
- **Thread dump-uri multiple:** Captează 3-5 thread dump-uri cu intervale de 10 secunde pentru a identifica tipare
- **Dimensiunea heap dump-ului:** Heap dump-urile pot fi mari; asigură-te că ai suficient spațiu pe disc
- **Impact în producție:** Unele comenzi (în special flag-ul `-F` și GC forțat) pot opri aplicația
- **Folosește jcmd când e posibil:** `jcmd` este instrumentul modern recomandat; combină funcționalitatea instrumentelor mai vechi

## Tabel de Referință Rapidă

| Comandă | Scop | Impact |
|---------|------|--------|
| `jmap -dump` | Snapshot complet heap | Utilizare mare memorie, pauză scurtă |
| `jmap -heap` | Configurație heap | Scăzut |
| `jmap -histo` | Histogramă obiecte | Scăzut |
| `jmap -histo:live` | Doar obiecte vii | Declanșează GC complet |
| `jstack` | Thread dump | Foarte scăzut |
| `jstack -F` | Thread dump forțat | Pauză scurtă |
| `jcmd VM.uptime` | Uptime JVM | Foarte scăzut |
| `jcmd VM.flags` | Flag-uri JVM | Foarte scăzut |
| `jcmd GC.run` | Forțează GC | Pauză aplicație |

## Instrumente Suplimentare

- **jcmd:** Instrument de diagnosticare modern (recomandat)
- **jstat:** Monitorizare statistici JVM
- **jvisualvm:** GUI pentru monitorizare și profilare
- **jconsole:** Consolă de monitorizare JMX

## Referințe

- [Documentația Oracle JDK Tools](https://docs.oracle.com/en/java/javase/17/docs/specs/man/index.html)
- [Referință jcmd](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jcmd.html)
- [Ghid de Depanare](https://docs.oracle.com/en/java/javase/17/troubleshoot/)

---

*Testează întotdeauna comenzile de diagnosticare într-un mediu non-producție pentru a înțelege impactul lor asupra aplicației tale specifice.*
