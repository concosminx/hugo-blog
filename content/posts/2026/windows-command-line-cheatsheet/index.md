---
date: '2026-09-04T10:00:00+03:00'
draft: false
title: 'Windows Command Line Cheatsheet: Files, Folders and Networking'
tags: ["windows", "cheatsheet"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "A practical reference for the Windows Command Prompt — moving around the filesystem, managing files, and diagnosing network problems with ipconfig, ping, tracert, netstat and friends."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

PowerShell has been the default shell on Windows for years now, but `cmd.exe` refuses to die — and for good reason. It starts instantly, it is on every Windows machine ever shipped, it works in Safe Mode and recovery consoles, and half the network diagnostics you will ever need are one short command away. When someone calls because "the internet is broken", `Win`+`R` → `cmd` is still the fastest path to an answer.

This is the set of commands worth having in muscle memory: the filesystem basics first, then the networking toolkit that makes the Command Prompt genuinely useful.

## Where you are and what's around you

A filepath is just your position in the filesystem. `C:` is the C drive, `C:\Users\Me\Documents` is a folder inside it, and `C:\Users\Me\Documents\notes.txt` is a file inside that. Everything below assumes you know which of those you are sitting in.

`dir` lists the contents of the current folder:

```bat
dir
dir myfolder
```

Two switches turn `dir` from a listing into a search tool. `/a` includes hidden and system files, which the plain listing quietly omits — this is usually the reason a folder "isn't empty" when you try to delete it. `/s` recurses into subfolders, so `dir /s /b *.log` walks a whole tree and prints bare paths with no headers.

`cd` (or its longer alias `chdir`) changes directory, and with no argument it simply prints where you are:

```bat
cd C:\Users\Me\Documents
cd ..
cd
```

`cd ..` goes one level up. One gotcha that catches everyone: `cd D:\some\path` from a `C:` prompt will *not* move you there. Each drive keeps its own current directory, so you change drives by typing the drive letter on its own, or by using `cd /d` to do both at once:

```bat
D:
cd /d D:\some\path
```

For a bird's-eye view of a whole tree rather than one level, `tree` draws it as ASCII art. Add `/f` to include files, not just folders:

```bat
tree C:\Projects /f
```

## Creating, moving and deleting

`md` creates a folder; `mkdir` is the same command under a different name.

```bat
md new-folder
```

Deleting is where the reference cards get it wrong often enough to be worth stating plainly: the Command Prompt command is `rd`, or equivalently `rmdir`. **There is no `rm` in `cmd.exe`** — `rm` only works in PowerShell, where it is an alias for `Remove-Item`. Typing it at a Command Prompt gets you "not recognized as an internal or external command".

```bat
rd empty-folder
rd /s /q folder-with-stuff-in-it
```

Plain `rd` refuses to delete anything that still has files in it. `/s` deletes the directory and everything beneath it, and `/q` suppresses the confirmation prompt. `/q` only does anything when `/s` is also present — and the combination deletes an entire tree without asking, so read the path twice before pressing Enter.

For files rather than folders, `del` takes one or more names and accepts wildcards:

```bat
del report.txt
del *.tmp
```

`copy` duplicates a file, `move` relocates one, and `ren` (long form `rename`) changes a name in place:

```bat
copy C:\data\report.txt D:\backup\
move folder1\file.txt folder2\
ren oldname.txt newname.txt
```

`copy` is fine for a handful of files. For anything resembling a real backup — whole trees, resumable transfers, retries on locked files — reach for `robocopy` instead, which ships with Windows and is built for exactly that job.

## Reading files and controlling the screen

`type` dumps a text file to the console, which is the quickest way to check a config file or a short log:

```bat
type C:\Windows\System32\drivers\etc\hosts
```

`fc` compares two files and prints the differences, which is more useful than it sounds when you are trying to work out which of two nearly identical config files is the broken one:

```bat
fc config.old config.new
```

`echo` prints a message, and `cls` clears the screen. `exit` closes the Command Prompt or ends the current batch script.

```bat
echo Backup finished
cls
```

## Getting help

Two forms, and they do different things. `help` on its own lists the built-in commands, and `help <command>` explains one of them:

```bat
help
help rd
```

But `help` only knows about the commands built into `cmd.exe`. For everything else — `ping`, `netstat`, `ipconfig` — append `/?` to the command itself:

```bat
ping /?
netstat /?
```

When in doubt, `/?` is the one to reach for; it works for both.

## Your own network configuration

`ipconfig` is the starting point for any network question. On its own it prints the IP address, subnet mask and default gateway for each adapter. `/all` is the version you actually want — it adds the MAC address, DHCP server, lease times and configured DNS servers:

```bat
ipconfig
ipconfig /all
```

The DNS-related switches are the ones that fix problems rather than just describe them:

```bat
ipconfig /displaydns
ipconfig /flushdns
ipconfig /registerdns
```

`/displaydns` shows what the resolver has cached, `/flushdns` empties that cache, and `/registerdns` renews the DHCP lease and re-registers the machine's name with DNS. `/flushdns` is the standard answer to "we changed the DNS record an hour ago and this one machine still goes to the old server". `/registerdns` needs an elevated prompt.

## Is it reachable?

`ping` sends ICMP echo requests and reports what comes back. By default it sends four and stops.

```bat
ping 192.168.1.144
ping google.com
```

The switches worth knowing:

| Switch | What it does |
| --- | --- |
| `-t` | Ping continuously until you stop it with `Ctrl`+`C`. `Ctrl`+`Break` prints statistics without quitting. |
| `-n <count>` | Send exactly this many requests instead of four. |
| `-l <size>` | Set the size of the data payload in bytes. The default is 32, the maximum is 65500. |
| `-f` | Set the "don't fragment" flag. IPv4 only. |
| `-w <ms>` | How long to wait for each reply. The default is 4000 ms. |
| `-4` / `-6` | Force IPv4 or IPv6. Only needed when you give a hostname rather than an address. |
| `-a` | Reverse-resolve the target address to a hostname. |

Note what `-l` actually does: it sets the payload size, it does not let you edit the packet contents. Combined with `-f` it becomes a path MTU test — send a large, unfragmentable packet and see whether anything along the way complains:

```bat
ping -f -l 1300 8.8.8.8
```

If a router in the path has a smaller MTU than the packet you sent, it cannot fragment it and cannot forward it, so it replies with "Packet needs to be fragmented but DF set". Walk the size down until the pings succeed and you have found the effective MTU. Because `-f` is an IPv4-only flag, this trick does not work over IPv6.

A failed ping is weaker evidence than people assume. Plenty of hosts — Windows Firewall out of the box among them — drop ICMP by policy while serving traffic perfectly well. "Ping fails" means "ICMP does not come back", not "the host is down".

## Where does the traffic actually go?

`tracert` lists every router hop between you and the destination:

```bat
tracert google.com
tracert -d 8.8.8.8
```

`-d` skips the reverse DNS lookup for each hop, which makes the output appear much faster when the intermediate routers have no PTR records. `-h <count>` caps the number of hops, which is handy when you only care about the first few and don't want to wait for thirty.

`pathping` is `tracert` plus sustained measurement. It maps the route first, then pings every hop repeatedly and reports per-hop latency and packet loss:

```bat
pathping -n google.com
```

Be ready to wait. By default it sends 100 queries to each hop at 250 ms intervals, which works out to roughly 25 seconds per hop — a ten-hop path takes about four minutes. `-q` lowers the query count and `-p` shortens the interval if you want an answer sooner. The payoff is that it distinguishes a router that deprioritises ICMP aimed at itself from a link that is genuinely dropping forwarded traffic, which a plain `tracert` cannot do.

## What is this machine talking to?

`netstat` shows connections and listening ports. Almost nobody runs it bare; the useful combination is:

```bat
netstat -ano
```

`-a` includes listening ports as well as established connections, `-n` prints addresses and port numbers numerically instead of trying to resolve names, and `-o` adds the owning process ID so you can match a port to something in Task Manager. `-n` is worth adding for speed alone — name resolution on a long connection list is slow.

To go straight from a port to an executable name, swap `-o` for `-b`:

```bat
netstat -ab
```

`-b` needs an elevated prompt and will fail without one, and it is noticeably slower than `-o`. A shell host such as `svchost.exe` shows the chain of components in brackets rather than a single name.

Two things the output will teach you. A process listening on `0.0.0.0` accepts connections on every interface, whereas one bound to `127.0.0.1` is reachable only from the machine itself — that distinction explains a great many "it works locally but not from my laptop" reports. And when a service refuses to start with "address already in use", `netstat -ano` plus the PID tells you exactly what got there first.

For protocol counters rather than individual connections, `-s` prints statistics per protocol — IPv4, IPv6, TCP, UDP and ICMP:

```bat
netstat -s -p tcp
```

`netstat -r` prints the routing table, and is identical to `route print`.

## Name resolution

`nslookup` queries DNS directly, which matters because it asks a server rather than reporting what the client already believes:

```bat
nslookup www.google.com
```

Run without arguments it drops into an interactive mode where you can change the record type and query repeatedly. `set q=mx` for mail records, `set q=cname` for aliases, `set q=ns` for the authoritative nameservers:

```bat
nslookup
> set q=mx
> example.com
> exit
```

You can also point it at a specific server by adding it as a second argument — `nslookup example.com 8.8.8.8` asks Google's resolver rather than yours. Comparing your DNS server's answer against a public one is the fastest way to confirm a stale record.

## The ARP cache

ARP maps IP addresses to MAC addresses on the local segment. The cache is what your machine currently believes about its neighbours:

```bat
arp -a
```

Entries are either dynamic (learned from a neighbour and aged out over time) or static (added by hand or by the system). `FF-FF-FF-FF-FF-FF` is the broadcast address, not a real host.

You can delete an entry or add one:

```bat
arp -d 192.168.1.1
arp -s 192.168.1.1 00-AA-22-BB-33-CC
```

Both need an elevated prompt. One limit on `arp -s` is worth knowing before you rely on it: the entry is discarded whenever the TCP/IP stack is stopped and started, so it does not survive a reboot. For a static mapping that persists, use `netsh interface ipv4 add neighbors "Ethernet" 192.168.1.1 00-aa-22-bb-33-cc` or the PowerShell equivalent `New-NetNeighbor`, both of which store the entry persistently by default.

Deleting an entry is the more common move in practice — after a router swap or a failover, a stale ARP entry pointing at the old MAC will black-hole traffic until it ages out, and `arp -d` skips the wait.

## The routing table

`route print` shows the machine's routing table:

```bat
route print
```

The entry to look for is the default route: destination `0.0.0.0`, mask `0.0.0.0`, pointing at your gateway. That is the "everything else goes here" rule, and its absence explains total loss of external connectivity while the local network still works. A VPN client that exited badly is a common cause of a leftover route that sends traffic into a tunnel that no longer exists.

Adding a route needs a destination, a mask and a gateway — a bare `route add 192.168.1.1` will not do anything useful:

```bat
route add 10.41.0.0 mask 255.255.0.0 10.27.0.1
route -p add 10.41.0.0 mask 255.255.0.0 10.27.0.1
route delete 10.41.0.0
```

Without `-p` the route lives only until the TCP/IP stack restarts. `-p` makes it persistent by writing it to the registry. All three commands need an elevated prompt.

## Testing a single port

Ping tells you whether a host answers ICMP. It says nothing about whether the service you care about is listening. For that, open a TCP connection to the specific port:

```bat
telnet mail.example.com 25
telnet example.com 80
```

A blank screen or a service banner means the port is open and reachable. An immediate "Could not open connection to the host" means something — the service, a firewall, or a network device in between — refused or dropped it. Telnet's own default port is 23, but as a connectivity test the port you name is the whole point: 25 for SMTP, 80 for HTTP, 443 for HTTPS, 3389 for RDP.

One catch: the Telnet client has not been installed by default since Windows Vista. Enable it once from an elevated prompt:

```bat
dism /online /Enable-Feature /FeatureName:TelnetClient
```

If you would rather not install anything, PowerShell's `Test-NetConnection -ComputerName example.com -Port 443` does the same test and reports the result rather than dropping you into a raw session.

## NetBIOS, if you still need it

`nbtstat` covers NetBIOS over TCP/IP — name resolution from the era before DNS was universal on Windows networks. On a modern network you will rarely need it, but legacy file shares and old line-of-business applications still lean on it.

```bat
nbtstat -n
nbtstat -c
nbtstat -A 10.0.0.99
```

`-n` shows the local machine's NetBIOS name table, `-c` shows the name cache — the names it has resolved and the addresses it resolved them to. Sessions are a different flag: `-s` lists NetBIOS client and server sessions with names, `-S` lists the same by IP address only. Confusing `-c` with `-s` is easy and gives you the wrong table entirely. For remote lookups, lowercase `-a` takes a NetBIOS name and uppercase `-A` takes an IP address — `nbtstat` is one of the few Windows commands where case genuinely matters.

## The PowerShell equivalents

None of this is going away, but PowerShell has first-class replacements for most of it, and they return objects you can filter and sort instead of text you have to parse:

| Command Prompt | PowerShell |
| --- | --- |
| `ipconfig /all` | `Get-NetIPConfiguration -Detailed` |
| `ipconfig /flushdns` | `Clear-DnsClientCache` |
| `ping` | `Test-Connection` |
| `telnet host port` | `Test-NetConnection -ComputerName host -Port port` |
| `tracert` | `Test-NetConnection -TraceRoute` |
| `nslookup` | `Resolve-DnsName` |
| `netstat -ano` | `Get-NetTCPConnection` |
| `arp -a` | `Get-NetNeighbor` |
| `route print` | `Get-NetRoute` |

The classic commands remain the right choice when you are on an unfamiliar machine, in a recovery environment, or reading someone else's script. Learn both — they cost about an afternoon between them, and one of them is always available.

Microsoft documents every command in this post at [the Windows Commands reference](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands), which is the place to go when `/?` is too terse.
