---
date: '2026-07-26T12:00:00+03:00'
draft: false
title: 'Linux DevOps Cheatsheet: Commands Every Engineer Should Know'
tags: ["cheatsheet"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
# cover:
#   image: linux-devops-cheatsheet-cover.jpg
#   alt: 'Linux DevOps terminal commands cheatsheet'
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

When the unit reports as running but the application is not behaving, look at the process table directly:

```bash
ps -ef | grep [n]ginx
```

The brackets stop `grep` from matching its own command line. Without them the search always returns at least one result — itself — which is harmless when you are reading but breaks any script that counts matches.

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

`iostat` names the busy device; `iotop` names the process responsible:

```bash
iotop -o
```

`-o` hides idle processes, which is almost always what you want — the default view is mostly zeros.

A disk that reports free space but still refuses writes has usually exhausted its inodes rather than its bytes:

```bash
df -i
```

Every file consumes one inode regardless of size, so a directory full of tiny session files or log fragments can fill the table while `df -h` still shows gigabytes free. It is an unintuitive failure that costs hours if you don't know to check for it.

The other version of "the disk is full but nothing is using it" is a deleted file that a process still holds open. The space is not released until the handle closes:

```bash
lsof +L1
```

`+L1` lists open files whose link count has dropped below one — deleted, but still consuming blocks. Restarting the process that holds them frees the space.

For a snapshot suitable for a script or a log, `top` can run without its interactive display:

```bash
top -b -n 1
```

`-b` is batch mode and `-n 1` takes a single sample, so the output is plain text you can redirect to a file.

Everything above shows the system as it is right now. When the incident is already over, `sar` from the `sysstat` package replays what happened:

```bash
sar -r
sar -b
sar -s 09:00:00 -e 10:00:00
sar -f /var/log/sysstat/sa15
```

`-r` reports memory and `-b` I/O. `-s` and `-e` bound a window within the current day, and `-f` reads a specific day's archive. Collection is disabled by default on Debian and Ubuntu — set `ENABLED="true"` in `/etc/default/sysstat` and restart the service, or there will be nothing to read back when you need it. Turning it on before an incident is the whole point.

## Filesystem Maintenance

Identify a device before you touch it. Kernel names like `/dev/sdb` can change between boots, which is why `/etc/fstab` should reference a UUID or a label instead:

```bash
blkid
```

Labels are easier to read than UUIDs, and on ext filesystems you can set one:

```bash
e2label /dev/sdb1 data
```

After editing `/etc/fstab`, apply it without rebooting:

```bash
mount -a
```

Do this *before* you reboot, not after. A mistake in `/etc/fstab` can leave a machine unbootable, and `mount -a` surfaces the error while you still have a shell to fix it in.

To release a mount that is busy:

```bash
umount -l /mnt/data
```

`-l` is a lazy unmount — it detaches the filesystem from the tree immediately and finishes cleaning up when the last reference closes. This is the escape hatch for a hung NFS mount that ordinary `umount` refuses to release.

Checking and repairing a filesystem:

```bash
fsck -y -C /dev/sdb1
```

`-y` answers yes to every prompt and `-C` shows progress. Never run this against a mounted filesystem — unmount it first, or work from rescue media.

If the primary superblock is damaged, locate a backup and repair from it:

```bash
mke2fs -n /dev/sdb1
fsck -b 32768 -y /dev/sdb1
```

`mke2fs -n` is a dry run: it reports where superblock backups would be written without creating anything. Omitting `-n` formats the volume and destroys the data you were trying to recover, so read that command twice before running it.

Finally, logs that grow unchecked are a common cause of a full disk. Rotation is configured in `/etc/logrotate.conf`, with per-service overrides in `/etc/logrotate.d/`.

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

`/health` is the standard DevOps health-check path. The `-v` flag shows request and response headers — essential when debugging proxies or TLS issues. The rest of the flags worth knowing are in the [cURL cheatsheet](/posts/2026/curl-command-cheatsheet/).

Check all active firewall rules at once:

```bash
firewall-cmd --list-all
```

This shows zones, services, ports, and rich rules together — one command for a full firewall picture.

## Network Diagnostics

The section above answers "is anything listening?". This one answers "can we reach it, and where does the traffic stop?".

Start with the interface and routing configuration:

```bash
ip addr
ip route
```

If you learned `ifconfig` and `route`, note that both belong to the `net-tools` package, which modern distributions no longer install — on a stock Ubuntu 22.04 the commands are simply not present. `ip` is the replacement and is always available. The same applies to `netstat`, superseded by the `ss` shown in the previous section.

For the physical link — negotiated speed, duplex, whether the cable is connected at all:

```bash
ethtool eth0
```

To find where packets stop:

```bash
traceroute example.com
```

For DNS, prefer `dig` over `nslookup`; it reports the same answers with far more detail:

```bash
dig example.com
dig example.com @8.8.8.8
dig example.com +trace
```

Querying a specific resolver is how you separate "DNS is broken" from "our resolver is broken" — if `@8.8.8.8` returns the right record and your local resolver doesn't, you have found the problem. `+trace` walks the delegation down from the root servers, which identifies exactly which nameserver hands back the wrong answer.

To test whether a remote port is reachable:

```bash
nmap -p 8080 192.168.1.10
```

`nc` checks a single port and can also stand up a listener to test the reverse direction:

```bash
nc -l 1234
nc 127.0.0.1 1234
```

Run the listener on one host and the client on another, and whatever you type crosses the connection. It is the quickest way to prove a firewall rule actually works before involving the application.

For live bandwidth broken down by connection:

```bash
iftop
```

And when you need the packets themselves:

```bash
tcpdump -n -i eth0 host example.com
tcpdump -n -i eth0 not port 22
```

`-n` skips DNS resolution, which keeps output fast and stops the capture's own lookups appearing in it. Excluding port 22 is worth remembering when you are capturing over SSH — without it you record your own session recursively.

To write a capture for later analysis, rotating files at 10 MB:

```bash
tcpdump -n -i eth0 -w capture.pcap -C 10
```

`-C` takes effect only alongside `-w`: it caps each savefile and opens the next. On its own, with output going to the terminal, there is no file to rotate and the flag does nothing.

Firewall rules on distributions still using iptables:

```bash
iptables -L -n -v
```

`-n` skips name resolution and `-v` shows packet and byte counters, which reveal whether a rule is being hit at all. Most current distributions have moved to nftables, where the equivalent is `nft list ruleset`; `iptables` usually remains as a compatibility shim.

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

### Loops

Repeat a command across a fixed list of hosts, environments or services:

```bash
for vm in web log db; do
  ssh "$vm" uptime
done
```

Numeric ranges come from brace expansion:

```bash
for i in {1..5}; do
  curl -s -o /dev/null -w '%{http_code}\n' "http://localhost:808$i/health"
done
```

### Turning output into arguments

`xargs` converts stdin into command arguments, which is what you need when a command doesn't read stdin itself:

```bash
seq 5 | xargs -I{} echo "processing {}"
find /var/log -name '*.log' -mtime +30 | xargs rm
```

`-I{}` names a placeholder so the value can sit anywhere in the command rather than only at the end.

### Redirection

Streams are numbered: 1 is stdout, 2 is stderr.

```bash
cmd 1>out.log
cmd 2>err.log
cmd &>all.log
```

The distinction matters most in cron. A job that redirects only stdout still emails you its stderr, while one using `&>` writes both to the file and stays completely silent — which is exactly how a failing cron job runs unnoticed for months.

### Timing a command

```bash
time ./deploy.sh
```

Reports real, user and system time. Worth wrapping around anything you are about to put on a schedule, so the interval you choose is longer than the job actually takes.

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

### Find by type, age and depth

Size is one predicate among several:

```bash
find /var/log -type f -name '*.log'
find /etc -type d -name 'conf.d'
find /var/log -mtime +7
find /var -size -10M
```

`-type f` restricts to files and `-type d` to directories. `-mtime +7` matches anything modified more than seven days ago, which is the basis of most log-cleanup jobs. To run a command over the results:

```bash
find /etc -type f -exec grep -il 'listen' {} +
```

The trailing `+` batches the results into as few invocations as possible. The more commonly seen `\;` runs the command once per file, which is noticeably slower across a large tree.

### Measure directory sizes

The `du -sh /*` pipeline earlier answers "what is filling this disk?". To narrow it down:

```bash
du -h --max-depth=1 /var
du -ah /var/log | sort -rh | head
```

`--max-depth=1` limits output to immediate children rather than every file beneath. `-a` includes individual files, `-c` adds a total, `-s` collapses to a single line, and `-m` forces megabytes when you want comparable numbers instead of mixed units.

### Copy attributes and link releases

```bash
cp -p app.conf /backup/app.conf
ln -s /opt/app/releases/2026-08-29 /opt/app/current
```

`cp -p` preserves mode, ownership and timestamps — the difference between a backup you can restore and one that returns owned by root with today's date. A symlink pointing at the current release is the usual mechanism for atomic switching: deploy into a new directory, repoint the link, and roll back by repointing it again.

## Text Processing

Much of the work is reshaping text — log lines, CSV exports, config files. A handful of tools cover nearly all of it.

### grep beyond the basics

The recursive search shown above is the common case; these flags handle the rest:

| Flag | Effect |
| --- | --- |
| `-i` | Case-insensitive |
| `-v` | Invert — print lines that do *not* match |
| `-c` | Print a count instead of the matching lines |
| `-n` | Prefix each match with its line number |
| `-w` | Match whole words only |
| `-o` | Print only the matching part, not the whole line |
| `-h` | Suppress filenames when searching multiple files |
| `-e` | Add another pattern; repeatable |

`-v` is the one worth internalising. Filtering noise *out* of a log is more often useful than filtering signal in:

```bash
grep -v -e 'health' -e 'metrics' /var/log/nginx/access.log
```

### sed

Stream editing, most often substitution. Edit in place while keeping a backup:

```bash
sed -i'.bak' 's/old/new/g' config.yml
```

The backup is cheap insurance: `-i` without it rewrites the file with no undo. Several substitutions can run in one pass, and an address restricts the change to a line range:

```bash
sed 's/ab/xy/g; s/de/pq/g' file.txt
sed '10,20 s/abc/xyz/g' file.txt
```

### awk

Where `cut` handles fixed delimiters, `awk` adds logic. `-F` sets the field separator; fields are `$1`, `$2` and so on, with `$0` the whole line:

```bash
awk -F',' '{print $1"@"$3}' data.csv
awk -F: '{print $(NF-1)}' /etc/passwd
```

`NF` holds the number of fields, so `$(NF-1)` counts from the right — the way to reach a trailing column when line width varies. Patterns filter which lines the action runs on:

```bash
awk -F'\t' '/error/ {print $0}' app.log
awk '$1 == 100 {print}' data.txt
awk 'END {print NR " rows"}' data.csv
```

`END` runs after the last line, which is where totals belong.

### cut, sort, uniq and tr

`cut` extracts columns by delimiter:

```bash
cut -d ',' -f 1,3,5 data.csv
cut -d ',' --complement -f 1 data.csv
cut -f -3 data.tsv
cut -f 3- data.tsv
```

`--complement` inverts the selection. `-f -3` means fields up to the third, `-f 3-` from the third onward.

`sort` and `uniq` belong together, because **`uniq` only collapses adjacent duplicates** — unsorted input silently produces wrong counts:

```bash
sort access.log | uniq -c | sort -rn | head
```

Sort, count, then sort by count descending: that pipeline answers "what is most frequent here?" and comes up constantly. `sort -n` compares numerically rather than as text, which matters as soon as values reach double digits. `uniq -d` shows only duplicated lines, `-u` only unique ones, and `-i` ignores case.

`tr` translates or squeezes characters:

```bash
tr 'a-z' 'A-Z' < file.txt
tr -s ' ' < file.txt
```

`-s` squeezes repeats into one, the usual way to normalise ragged whitespace before handing a line to `cut`.

## Archives & Compression

Create, extract, and — importantly — inspect before extracting:

```bash
tar cvf backup.tar /etc/nginx
tar xvf backup.tar
tar tvf backup.tar
```

`t` lists contents without writing anything. Run it first on any archive you did not create yourself: one built from absolute paths, or with dozens of files at the top level, will otherwise scatter them across your working directory.

Modern GNU tar detects compression automatically when extracting, so `tar xvf archive.tar.gz` works without `-z`. Creating still needs the flag:

```bash
tar czvf backup.tar.gz /etc/nginx
tar cjvf backup.tar.bz2 /etc/nginx
```

For plain gzip and zip:

```bash
gzip -d file.gz
zip -r archive.zip directory/
```

Two zip operations worth knowing — remove a file from an existing archive, and read one out without unpacking:

```bash
zip -d archive.zip unwanted.txt
unzip -p archive.zip config.yml
```

`unzip -p` writes to stdout, so a single file can go straight into `grep` or `diff` without leaving a directory of extracted junk behind. Note that `zip` and `unzip` are not installed by default on a minimal Ubuntu — `tar` always is, which is why it remains the safer choice in a provisioning script.

---

These commands cover most of the situations I run into repeatedly: a service that won't start, a container that died quietly, a disk that filled up overnight, or an SSH session that suddenly stopped working. Keep them somewhere close and they will save you time when it matters.

