---
date: '2026-04-14T18:00:00+02:00'
draft: false
title: 'JVM Diagnostic Commands Reference'
tags: ["java", "jvm", "debugging", "performance", "monitoring"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Essential JVM diagnostic commands for troubleshooting and monitoring Java applications"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

Troubleshooting Java applications often requires analyzing heap memory, thread dumps, and JVM internals. This guide provides a quick reference to essential JVM diagnostic commands using tools like `jmap`, `jstack`, and `jcmd`.

## Prerequisites

- Java Development Kit (JDK) installed
- Process ID (PID) of the running Java application

To find your Java process ID:
```bash
jps -l
```

## Heap Dump

Generate a binary heap dump for memory analysis:

```bash
path/to/jdk/jmap -dump:format=b,file=<filename> <pid>
```

**Example:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jmap -dump:format=b,file=heap_dump.hprof 12345
```

Use tools like [Eclipse MAT](https://www.eclipse.org/mat/) or [VisualVM](https://visualvm.github.io/) to analyze the heap dump file.

## Heap Summary

Display heap configuration and usage summary:

```bash
path/to/jdk/jmap -heap <pid> > path/to/save_file/heap_summary_ddmmyy_hhmm.log
```

**Example:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jmap -heap 12345 > /var/log/heap_summary_$(date +%d%m%y_%H%M).log
```

This shows:
- Heap configuration (min/max heap sizes)
- Current heap usage
- Garbage collector information

## Heap Histogram

Generate a histogram of object counts and memory usage:

```bash
path/to/jdk/jmap -histo <pid> > path/to/save_file/heap_histo_ddmmyy_hhmm.log
```

**Example:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jmap -histo 12345 > /var/log/heap_histo_$(date +%d%m%y_%H%M).log
```

For live objects only (triggers a full GC first):
```bash
jmap -histo:live <pid>
```

## Thread Dump

Capture thread stack traces for deadlock detection and performance analysis:

```bash
path/to/jdk/jstack -F <pid> > path/to/save_file/thread_dump_ddmmyy_hhmm.log
```

**Example:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jstack -F 12345 > /var/log/thread_dump_$(date +%d%m%y_%H%M).log
```

The `-F` flag forces a thread dump when the process doesn't respond to normal signals.

**Without force flag:**
```bash
jstack <pid> > thread_dump.log
```

## JVM Uptime

Check how long the JVM has been running (in seconds):

```bash
path/to/jdk/jcmd <pid> VM.uptime
```

**Example:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jcmd 12345 VM.uptime
```

## VM Flags

Display all JVM flags and their current values:

```bash
path/to/jdk/jcmd <pid> VM.flags
```

**Example:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jcmd 12345 VM.flags
```

This shows:
- Ergonomic flags (automatically set by JVM)
- Command-line flags (explicitly set)
- Default values

## Invoke Garbage Collection

Manually trigger a garbage collection:

```bash
path/to/jdk/jcmd <pid> GC.run
```

**Example:**
```bash
/usr/lib/jvm/java-17-openjdk/bin/jcmd 12345 GC.run
```

**Warning:** Use with caution in production. Forcing GC can cause application pauses.

## JVM Uptime (Linux Alternative)

Check process uptime using Linux `ps` command:

```bash
ps -p <pid> -o etime=
```

**Example:**
```bash
ps -p 12345 -o etime=
```

Output format: `[[dd-]hh:]mm:ss`

For more detailed process information:
```bash
ps -p <pid> -o pid,etime,cmd
```

## Best Practices

- **Automate with timestamps:** Use `$(date +%d%m%y_%H%M)` in filenames for automatic timestamping
- **Multiple thread dumps:** Capture 3-5 thread dumps with 10-second intervals to identify patterns
- **Heap dump size:** Heap dumps can be large; ensure sufficient disk space
- **Production impact:** Some commands (especially `-F` flag and forced GC) can pause the application
- **Use jcmd when possible:** `jcmd` is the recommended modern tool; it combines functionality of older tools

## Quick Reference Table

| Command | Purpose | Impact |
|---------|---------|--------|
| `jmap -dump` | Full heap snapshot | High memory usage, brief pause |
| `jmap -heap` | Heap configuration | Low |
| `jmap -histo` | Object histogram | Low |
| `jmap -histo:live` | Live objects only | Triggers full GC |
| `jstack` | Thread dump | Very low |
| `jstack -F` | Forced thread dump | Brief pause |
| `jcmd VM.uptime` | JVM uptime | Very low |
| `jcmd VM.flags` | JVM flags | Very low |
| `jcmd GC.run` | Force GC | Application pause |

## Additional Tools

- **jcmd:** Modern diagnostic tool (recommended)
- **jstat:** JVM statistics monitoring
- **jvisualvm:** GUI for monitoring and profiling
- **jconsole:** JMX monitoring console

## References

- [Oracle JDK Tools Documentation](https://docs.oracle.com/en/java/javase/17/docs/specs/man/index.html)
- [jcmd Reference](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jcmd.html)
- [Troubleshooting Guide](https://docs.oracle.com/en/java/javase/17/troubleshoot/)

---

*Always test diagnostic commands in a non-production environment first to understand their impact on your specific application.*
