---
date: '2026-04-25T10:00:00+02:00'
draft: false
title: 'WSL Basic Commands Cheatsheet'
tags: ["wsl", "windows", "linux", "ubuntu", "cli"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "A practical reference guide for the most useful WSL (Windows Subsystem for Linux) commands — from installation to managing distributions and mounting disks."
cover:
    image: wsl-basics.png
    alt: 'WSL Basic Commands'
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

## Introduction

**Windows Subsystem for Linux (WSL)** lets you run a Linux environment directly on Windows without the overhead of a virtual machine. Whether you're installing your first Linux distribution or managing multiple ones, the `wsl` command is your primary tool.

> **Note:** The commands below work in **PowerShell** or **Windows Command Prompt**. If you are running them from a Bash / Linux shell, replace `wsl` with `wsl.exe`.

---

## Install WSL

```powershell
wsl --install
```

Installs WSL and the default **Ubuntu** distribution. You can also install a specific distribution:

```powershell
wsl --install --distribution <DistributionName>
```

Common install options:

| Option | Description |
|---|---|
| `--distribution <Name>` | Specify which Linux distro to install |
| `--no-launch` | Install without launching the distro automatically |
| `--web-download` | Download from the internet instead of the Microsoft Store |
| `--location <Path>` | Choose the install folder |
| `--inbox` | Use the Windows component instead of the Store |
| `--no-distribution` | Install WSL engine only, skip the distro |

> **Tip:** To list valid distribution names, run `wsl --list --online`.

---

## List Available Distributions (Online)

```powershell
wsl --list --online
# or shorthand:
wsl -l -o
```

Shows all Linux distributions available in the Microsoft Store.

---

## List Installed Distributions

```powershell
wsl --list --verbose
# or shorthand:
wsl -l -v
```

Displays all installed distributions along with their **state** (Running / Stopped) and **WSL version** (1 or 2).

Additional flags:

| Flag | Description |
|---|---|
| `--all` | List all distributions |
| `--running` | List only currently running distributions |
| `--quiet` | Show distribution names only |

---

## Set WSL Version for a Distribution

```powershell
wsl --set-version <DistributionName> <1|2>
```

Switches a distribution between WSL 1 and WSL 2. Example:

```powershell
wsl --set-version Ubuntu 2
```

> **Warning:** Switching versions can take time and may fail for large projects. Back up your data first.

---

## Set Default WSL Version

```powershell
wsl --set-default-version <1|2>
```

Sets the default WSL version used for **new** distribution installations. WSL 2 is recommended on Windows 10 (build 18362+) and Windows 11.

---

## Set Default Distribution

```powershell
wsl --set-default <DistributionName>
```

Sets which distribution `wsl` commands will target by default.

---

## Start WSL in the Home Directory

```powershell
wsl ~
```

Opens WSL and places you directly in the Linux user's home directory (`~`). Inside the shell you can always jump back home with:

```bash
cd ~
```

---

## Run a Specific Distribution and User

```powershell
wsl --distribution <DistributionName> --user <UserName>
```

Launches a specific distro as a specific user. Example:

```powershell
wsl --distribution Debian --user root
```

> Run `whoami` inside WSL to check the current user.

---

## Update WSL

```powershell
wsl --update
```

Updates WSL to the latest version. Use `--web-download` to pull directly from GitHub instead of the Store:

```powershell
wsl --update --web-download
```

---

## Check WSL Status

```powershell
wsl --status
```

Shows general WSL configuration info: default distribution type, default distribution, and kernel version.

---

## Check WSL Version

```powershell
wsl --version
```

Displays detailed version information about WSL and all its components.

---

## Help

```powershell
wsl --help
```

Lists all available commands and options.

---

## Run as a Specific User

```powershell
wsl --user <Username>
```

Launches WSL as the given user (must already exist inside the distribution).

---

## Change the Default User for a Distribution

```powershell
<DistributionName> config --default-user <Username>
```

Example — set the default Ubuntu login user to `johndoe`:

```powershell
ubuntu config --default-user johndoe
```

> **Note:** This does not work for imported distributions. For those, edit `/etc/wsl.conf` instead.

---

## Shutdown All WSL Instances

```powershell
wsl --shutdown
```

Immediately terminates **all** running distributions and the WSL 2 virtual machine. Useful after editing `.wslconfig` or when changing memory limits.

---

## Terminate a Specific Distribution

```powershell
wsl --terminate <DistributionName>
```

Stops the specified distribution without affecting others.

---

## Identify IP Addresses

```powershell
# IP of the Linux distribution (WSL 2 VM):
wsl hostname -I

# IP of the Windows host as seen from WSL 2:
wsl -- ip route show | grep -i default | awk '{ print $3}'
```

---

## Export a Distribution

```powershell
wsl --export <DistributionName> <FileName>
```

Saves a snapshot of the distribution as a `.tar` file (useful for backups or migration). Add `--vhd` to export as a `.vhdx` file (WSL 2 only).

---

## Import a Distribution

```powershell
wsl --import <DistributionName> <InstallLocation> <FileName>
```

Restores a distribution from a `.tar` file. Options:

| Option | Description |
|---|---|
| `--vhd` | Import from a `.vhdx` file (WSL 2 only) |
| `--version <1\|2>` | Specify WSL version for the imported distro |

---

## Import a Distribution In-Place

```powershell
wsl --import-in-place <DistributionName> <FileName>
```

Registers an existing `.vhdx` file (must use the `ext4` filesystem) as a distribution — no copy is made.

---

## Unregister / Uninstall a Distribution

```powershell
wsl --unregister <DistributionName>
```

Removes the distribution from WSL. **All data, settings, and software are permanently deleted.** To reinstall a clean copy, find the distro in the Microsoft Store.

---

## Mount a Disk or Device

```powershell
wsl --mount <DiskPath>
```

Attaches and mounts a physical disk across all WSL 2 distributions. Common options:

| Option | Description |
|---|---|
| `--vhd` | Treat the path as a virtual hard disk |
| `--name <Name>` | Use a custom mount point name |
| `--bare` | Attach without mounting |
| `--type <Filesystem>` | Filesystem type (default: `ext4`) |
| `--partition <N>` | Partition index (default: whole disk) |
| `--options <MountOptions>` | Filesystem-specific options |

---

## Unmount a Disk

```powershell
wsl --unmount <DiskPath>
```

Unmounts the specified disk. Omitting `<DiskPath>` unmounts **all** mounted disks.

---

## Quick Reference Table

| Command | Description |
|---|---|
| `wsl --install` | Install WSL + default Ubuntu |
| `wsl -l -o` | List available online distros |
| `wsl -l -v` | List installed distros with status |
| `wsl --set-version <distro> 2` | Switch a distro to WSL 2 |
| `wsl --set-default-version 2` | Default new installs to WSL 2 |
| `wsl --set-default <distro>` | Set default distribution |
| `wsl ~` | Open WSL in home directory |
| `wsl --update` | Update WSL |
| `wsl --status` | Show WSL config info |
| `wsl --shutdown` | Stop all running distros |
| `wsl --terminate <distro>` | Stop a single distro |
| `wsl --export <distro> <file>` | Backup a distro |
| `wsl --import <distro> <path> <file>` | Restore a distro |
| `wsl --unregister <distro>` | Remove a distro permanently |

---

## References

- [Basic WSL commands — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/basic-commands)
- [Install WSL — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Comparing WSL 1 and WSL 2](https://learn.microsoft.com/en-us/windows/wsl/compare-versions)
