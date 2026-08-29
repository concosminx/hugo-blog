---
date: '2026-08-29T09:00:00+03:00'
draft: false
title: 'YUM Command Cheatsheet'
tags: ["cheatsheet"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "A practical reference for yum on Red Hat Enterprise Linux — querying packages, managing repositories, installing and removing software, and undoing transactions when something goes wrong."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

`yum` is the package manager that shipped with Red Hat Enterprise Linux through RHEL 7, and it is still the command you type on RHEL 8 and later — there it is a symlink to `dnf`, which keeps the same subcommands and options. This is the set of subcommands worth knowing, grouped by what you are actually trying to do.

## Querying packages

Before installing anything, it helps to know what is out there. `yum help` lists every subcommand and option, but these are the queries you will reach for daily.

List packages from the configured repositories:

```bash
yum list available
yum list installed
yum list all
```

The three forms differ only in scope: what you *could* install, what you *have* installed, and both together. You can narrow any of them with a package name:

```bash
yum list kernel
```

Handy before a reboot — it shows which kernel versions are installed alongside what the repos are offering.

To read the metadata for a single package:

```bash
yum info vsftpd
```

Version, architecture, repository, size, and the full description. Use it to confirm you are about to install what you think you are.

Dependencies are often the interesting part:

```bash
yum deplist nfs-utils
```

This lists each dependency *and* the package that would satisfy it, which is what makes it more useful than reading the spec.

The reverse lookup — "which package owns this file?" — is `provides`:

```bash
yum provides "*bin/top"
yum provides "*/README.top"
```

Quote the glob so the shell doesn't expand it first. This is the command that answers "I have a binary and no idea where it came from."

When you don't know the package name at all, search names and descriptions:

```bash
yum search samba
```

And for pending updates, `updateinfo` explains *why* an update exists:

```bash
yum updateinfo security
```

Security errata, listed with their advisory IDs. Pair it with `yum check-update`, which simply asks the repositories what has a newer version available.

## Package groups

Groups bundle related packages so you can install a whole role in one step.

```bash
yum grouplist
yum groupinfo "Web Server"
yum groupinstall "Web Server"
```

`grouplist` shows installed and available groups, `groupinfo` expands one into its package list, and `groupinstall` pulls the lot in. Always run `groupinfo` first — groups can be considerably larger than they sound.

## Installing, updating and removing

The core of everyday use:

```bash
yum install vsftpd
```

Updating takes an optional package name; without one it updates everything:

```bash
yum update
yum update httpd
```

On a production box you usually want the narrower version — security fixes only:

```bash
yum update --security
```

There are three related variants worth distinguishing. `update-to` pins the update to a particular version rather than the newest. `upgrade` behaves like `update` but also honours obsoletes, so packages that have been replaced upstream get swapped rather than left behind. And `localinstall` takes a path or a URL instead of a repository name:

```bash
yum localinstall abc-1-1.i686.rpm
yum localinstall http://myrepo/abc-1-1.i686.rpm
```

The advantage over plain `rpm -i` is that yum still resolves dependencies from your configured repositories.

When an update goes badly, step back one version:

```bash
yum downgrade abc
```

When files have been deleted or corrupted but the version is fine, reinstall in place:

```bash
yum reinstall util-linux
```

To replace one package with another in a single transaction:

```bash
yum swap ftp lftp
```

Removal has three spellings with slightly different behaviour. `yum remove` and `yum erase` are the same command, and both take the package's dependencies with it:

```bash
yum remove vsftpd
```

`autoremove` goes further and also clears out dependencies that nothing else needs any more:

```bash
yum autoremove httpd
```

That extra sweep is usually what you want after decommissioning a service, but read the list before confirming — "nothing else needs it" is judged from the RPM database, not from what you happen to run.

## Managing repositories

```bash
yum repolist
```

The enabled repositories, one per line. For details on a specific one:

```bash
yum repoinfo rhel-7-server-rpms
```

`repo-pkgs` scopes an operation to a single repository, which is the clean way to handle an internal repo of your own packages:

```bash
yum repo-pkgs my-rpms list
yum repo-pkgs my-rpms install
yum repo-pkgs my-rpms remove
```

And when metadata looks stale, prime the cache explicitly:

```bash
yum makecache
```

## Troubleshooting and maintenance

The transaction history is the feature most people discover too late.

```bash
yum history list
```

Every install, update and erase, numbered. Inspect one:

```bash
yum history info 3
```

Then undo it, or redo it if the undo was a mistake:

```bash
yum history undo 3
yum history redo 3
```

This is a genuine safety net — an update that breaks a service can usually be rolled back in one command.

When the cache itself is the problem:

```bash
yum clean packages
yum clean all
```

`clean packages` drops the downloaded RPMs; `clean all` also drops the repository metadata, forcing a fresh fetch on the next run. Reach for `clean all` whenever yum insists a package doesn't exist and you are certain it does.

To check the local RPM database for inconsistencies:

```bash
yum check
```

Be patient with this one — on a large system it runs for a long time.

Two more, both RHEL 7 additions. `fssnapshot` lists LVM snapshots, which is what you fall back on when a rollback needs to go deeper than the yum history. And `fs` filters what actually lands on disk — useful on minimal systems where documentation and translations are dead weight:

```bash
yum fs filters
yum fs documentation
```

The second one excludes documentation from future installs. Set it deliberately; a system without man pages is a surprise nobody enjoys inheriting.

## Language packs

If you don't need the translations, don't install them.

```bash
yum langavailable
yum langlist
yum langinfo es
yum langinstall es
yum langremove es
```

`langavailable` lists everything on offer, `langlist` shows what is installed, and `langinfo` expands a language code into its package list before you commit.

## Options worth memorising

These apply across most subcommands:

| Option | Effect |
| --- | --- |
| `-y` | Assume yes at every prompt |
| `--assumeno` | Assume no at every prompt |
| `-q` | Suppress output |
| `-v` | Extra debugging output |
| `--noplugins` | Run without loading any plugins |
| `--disableplugin=` | Disable one plugin for this command |
| `--enableplugin=` | Enable an installed-but-disabled plugin |
| `--enablerepo=` | Enable a disabled repo for this command |
| `--disablerepo=` | Disable an enabled repo for this command |
| `--downloadonly` | Download to the cache without installing |
| `--changelog` | Show the package changelog |

A few of these earn their keep in specific situations. Enabling a repository for a single command avoids leaving it on permanently:

```bash
yum install docker --enablerepo=rhel-7-server-extras-rpms
```

The mirror image is useful when a third-party repo is shadowing a package you want from elsewhere:

```bash
yum list available --disablerepo=epel
```

`--downloadonly` stages packages in `/var/cache/yum/` without touching the running system — the usual first half of a maintenance window:

```bash
yum install --downloadonly vsftpd
```

And `--enableplugin=ps` turns on the plugin that maps packages to running processes:

```bash
yum --enableplugin=ps ps
```

Not every option works everywhere, since several depend on plugins. Check what you have with `yum list "yum-plugin*"`.

## Extra commands from yum-utils

Installing the `yum-utils` package adds a set of standalone commands that fill in the gaps:

```bash
yum install yum-utils
```

| Command | What it does |
| --- | --- |
| `find-repos-of-install` | Identify which repository a package came from |
| `needs-restarting` | List processes running against updated libraries |
| `repoclosure` | Report unmet dependencies across repositories |
| `repoquery` | Query remote repos and the local RPM database |
| `reposync` | Mirror a repository to a local directory |
| `repotrack` | Download a package plus all its dependencies |
| `show-installed` | List installed packages with statistics |
| `verifytree` | Check a local repository for consistency |
| `yum-complete-transaction` | Finish transactions that were interrupted |
| `yumdb` | Inspect or modify the yum database |
| `yumdownloader` | Download a package into the current directory |

Two of these come up constantly. After patching, `needs-restarting` tells you what is still running against the old libraries — the difference between an update applied and an update actually in effect:

```bash
needs-restarting
```

And `repoquery` answers dependency questions without installing anything:

```bash
repoquery --requires --resolve bash
```

For building a local mirror, `reposync` pulls a whole repository down:

```bash
reposync -r rhel-atomic-host-beta-rpms
```

`repotrack` is the narrower version — one package and its full dependency tree, which is what you want when preparing an offline install.

## Where to go next

`man yum` documents every subcommand and option in full, and on RHEL 8 and later `man dnf` covers the same ground for the successor. If you are writing automation, `yum history` and `--downloadonly` are the two features that make an update reversible and reviewable rather than a leap of faith.
