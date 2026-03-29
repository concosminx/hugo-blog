---
date: '2026-03-07T12:00:00+02:00'
draft: false
title: 'How To Create a New Sudo-Enabled User on Ubuntu'
tags: ["linux", "ubuntu", "sudo", "user-management"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Step-by-step guide to creating a new user with sudo privileges on Ubuntu."
cover:
    image: ubuntu-sudo-user.png
    alt: 'Ubuntu sudo user'
---

## Introduction

Granting sudo privileges to a user allows them to perform administrative tasks without logging in as root. This guide explains how to create a new user with sudo access on Ubuntu, following best practices for security and user management.

## Step 1 — Log Into Your Server

Connect to your Ubuntu server as the root user (or a user with sudo privileges):

```bash
ssh root@your_server_ip_address
```

## Step 2 — Add a New User

Create a new user account (replace `newuser` with your desired username):

```bash
adduser newuser
```

You will be prompted to set a password and fill in optional user information. You can press ENTER to accept the defaults.

## Step 3 — Grant Sudo Privileges

Add the new user to the `sudo` group:

```bash
usermod -aG sudo newuser
```

On Ubuntu, users in the `sudo` group can run commands as root using `sudo`.

## Step 4 — Test Sudo Access

Switch to the new user:

```bash
su - newuser
```

Test sudo by running a privileged command (e.g., listing the `/root` directory):

```bash
sudo ls -la /root
```

The first time you use `sudo`, you will be prompted for the new user's password. If successful, the command will run with root privileges.

## Understanding `sudo` vs `su`

- `sudo` allows a permitted user to run a command as another user (by default, root) after authenticating with their own password. It logs each command for auditing.
- `su` switches to another user account and requires the target user's password. It starts a new shell session as that user.

## Best Practices

- **Principle of Least Privilege:** Only grant sudo access to users who need it.
- **Limit sudo commands:** For tighter security, restrict which commands a user can run with sudo by editing the `/etc/sudoers` file using `visudo`.
- **Disable root login:** After creating a sudo-enabled user, consider disabling direct root login for better security:

```bash
sudo passwd -l root
```

## References

- [DigitalOcean: How To Create a New Sudo-Enabled User on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-ubuntu)
- [How To Edit the Sudoers File](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file)
- [How to Add and Delete Users on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-20-04)

---

*This post is based on the DigitalOcean tutorial linked above. Always follow your organization's security policies when managing users and sudo access.*
