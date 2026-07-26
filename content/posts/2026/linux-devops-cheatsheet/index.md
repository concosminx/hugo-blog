---
date: '2026-07-26T12:00:00+03:00'
draft: false
title: 'Linux DevOps Cheatsheet: Commands Every Engineer Should Know'
tags: ["linux", "devops", "bash", "sysadmin", "ssh", "docker", "systemctl", "security", "monitoring"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
cover:
  image: linux-devops-cheatsheet-cover.jpg
  alt: 'Linux DevOps terminal commands cheatsheet'
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

A curated set of Linux commands I keep coming back to for service management, debugging, server hardening, and automation. No fluff — just the commands that actually matter on a production server.

## Service Management

When a service refuses to start, the first move is always the same: check its status.

```bash
systemctl status nginx
```

This gives you the current state, recent log lines, and the PID — everything you need to start digging. It's the first command asked about in every DevOps interview. Know it cold.

If the status output is not enough, pull the last 50 log lines directly from the journal:

```bash
journalctl -u nginx -n 50 --no-pager
```

`-n 50` limits the output so you aren't scrolling forever, and `--no-pager` keeps everything in the terminal without launching `less`.

To get a bird's-eye view of everything that is currently broken on the system:

```bash
systemctl list-units --state=failed
```

One command, all the broken units. Useful when you log into a server and don't know where to start.

## Container Debugging

Containers die for all sorts of reasons. The fastest path to an answer is the logs:

```bash
docker logs --tail 100 -f container_name
```

`-f` follows new output in real time; `--tail 100` limits history so the terminal doesn't flood. Works with Podman too.

When logs aren't enough and you need to poke around inside a running container:

```bash
docker exec -it container_name /bin/bash
```

This is the most-asked container command in DevOps interviews. If `/bin/bash` is not available in the image, try `/bin/sh`.

For a quick snapshot of how much CPU and memory every container is consuming right now:

```bash
docker stats --no-stream
```

Without `--no-stream`, this refreshes continuously. With it, you get a single snapshot — good for scripts and quick checks.

## SSH Hardening

### Keys Only. No Passwords.

Password-based SSH is a liability. Disable it by editing `/etc/ssh/sshd_config`:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

Then restart the daemon:

```bash
systemctl restart sshd
```

> **Important:** make sure your SSH key is already in `~/.ssh/authorized_keys` on the server before you do this, or you will lock yourself out.

### Change the Default SSH Port

Moving SSH off port 22 eliminates most automated scanning noise. If the server runs SELinux, you must tell it about the new port first — otherwise `sshd` won't start:

```bash
semanage port -a -t ssh_port_t -p tcp 2222
firewall-cmd --permanent --add-port=2222/tcp
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --reload
```

Then update `Port 2222` in `/etc/ssh/sshd_config` and restart.

### Restrict SSH to Trusted IPs

Even better than changing the port is allowing SSH only from your own IP:

```bash
firewall-cmd --permanent --add-rich-rule=\
'rule family="ipv4" source address="YOUR.IP" service name="ssh" accept'
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --reload
```

Bots can't touch what they can't reach.

## Disable Unused Services

Every running service is an attack surface. List what is currently active:

```bash
systemctl list-units --state=running
```

Then stop and disable anything you don't recognise or don't need:

```bash
systemctl disable --now servicename
```

`--now` both stops the service immediately and prevents it from starting on next boot.

## Kernel Hardening

A few `sysctl` parameters go a long way. Add them to `/etc/sysctl.conf`:

```
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
```

Apply them without a reboot:

```bash
sysctl -p
```

- `tcp_syncookies` protects against SYN flood attacks.
- `rp_filter` drops packets that arrive on an interface they shouldn't be coming from.
- `icmp_echo_ignore_broadcasts` prevents the server from being used in Smurf amplification attacks.

## Disk & Performance

When the server is slow and you suspect a full disk, get the full picture in one line:

```bash
df -hT && du -sh /* 2>/dev/null | sort -rh | head -5
```

`df` shows overall disk usage per filesystem; `du` finds the top five directories consuming the most space. Chain them and you go from suspicion to culprit in seconds.

Check memory usage:

```bash
free -h
```

Cached memory is not wasted — Linux reclaims it when needed. Don't panic if "used" looks high; focus on the "available" column.

For disk I/O statistics, `iostat` from the `sysstat` package shows reads and writes per device:

```bash
iostat -x 1 3
```

`-x` enables extended stats; `1 3` refreshes every second, three times. Useful for spotting a disk that is saturating under load.

## Networking & Ports

When an application is not reachable, the first question is: is anything actually listening on that port?

```bash
ss -tulnp | grep :8080
```

Replace `8080` with your port. `ss` is faster and more detailed than `netstat`.

Check from the outside with `lsof` to confirm which process owns the socket:

```bash
lsof -i :8080
```

Test the endpoint directly and see every header:

```bash
curl -v http://localhost:8080/health
```

`/health` is the standard DevOps health-check path. The `-v` flag shows request and response headers — essential when debugging proxies or TLS issues.

Check all active firewall rules at once:

```bash
firewall-cmd --list-all
```

This shows zones, services, ports, and rich rules together — one command for a full firewall picture.

## Automation & Scripting

### Scheduled tasks

Edit the crontab for a specific service account (deployments often run as dedicated users):

```bash
crontab -e -u deploy
```

### File sync to remote servers

```bash
rsync -avz --delete /app/ user@server:/app/
```

`--delete` removes files on the destination that no longer exist in the source, keeping both sides in sync. A DevOps deployment staple.

### Live service monitoring

```bash
watch -n 5 'systemctl is-active nginx'
```

Refreshes every 5 seconds — simple live monitoring during deployments without installing extra tools.

## Sudo Access

Never edit `/etc/sudoers` directly. A syntax error there can lock you out of the entire system. Always use:

```bash
visudo
```

`visudo` validates the file before saving and refuses to write a broken configuration.

## Log Inspection

### Find what broke in the last 10 minutes

```bash
journalctl -xe --since '10 minutes ago'
```

`--since` narrows the window fast. During an incident, every second counts — don't scroll through hours of logs.

### Live error filtering

```bash
tail -f /var/log/nginx/error.log | grep -i error
```

Pipe `tail -f` into `grep` to filter the signal from the noise in real time.

### Quick system health check

```bash
uptime && w
```

Two commands in one line: `uptime` shows load averages for the last 1, 5, and 15 minutes; `w` shows who is logged in and what they are running. A 5-second gut-check before diving deeper.

## File Utilities

### Find large files

Quickly locate files over a certain size — essential for freeing disk space on servers:

```bash
find . -type f -size +100M
```

Adjust `+100M` to your threshold. Add `-exec ls -lh {} \;` to see sizes inline.

### Search within files

Recursively search for a string across all files in the current directory:

```bash
grep -r "searchText" .
```

A true lifesaver when you need to find which config file contains a specific value.

### Bulk rename files

Use a `for` loop to change extensions or prefixes on multiple files at once:

```bash
for f in *.bak; do
  mv -- "$f" "${f%.bak}.old"
done
```

### Monitor a log file in real time

Watch a log file as new lines are added — perfect for tracking application behaviour during a deployment:

```bash
tail -f /var/log/syslog
```

---

These commands cover most of the situations I run into repeatedly: a service that won't start, a container that died quietly, a disk that filled up overnight, or an SSH session that suddenly stopped working. Keep them somewhere close and they will save you time when it matters.

