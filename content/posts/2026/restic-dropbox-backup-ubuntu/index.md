---
date: '2026-10-05T18:00:00+03:00'
draft: false
title: 'Encrypted Weekly Backups to Dropbox with restic and rclone on Ubuntu 24.04'
tags: ["linux", "ubuntu", "backup"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Set up encrypted, deduplicated weekly backups from Ubuntu 24.04 to Dropbox using restic, rclone and a systemd user timer — including how to restore when you actually need it."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

A backup you have never restored from is a rumour. This is the setup I run on a home server: **restic** does the encryption and deduplication, **rclone** gives it a path to Dropbox, and a **systemd user timer** runs the whole thing every Sunday evening. Everything leaving the machine is encrypted client-side, so Dropbox stores ciphertext and nothing else.

The second half of the post is about getting data back out, which is the part most guides skim.

## What this setup does

Once it is running, every Sunday at 20:00 the server wakes up, uploads whatever changed since last week, trims old snapshots to a retention policy, and goes back to sleep. If the machine was powered off at the time, the backup runs at the next boot instead of being skipped.

A few things to know before starting:

- Backups are **encrypted with a password you create**. Dropbox cannot read them, and neither can you if you lose the password.
- Data is **deduplicated**, so the second weekly run uploads only changed blocks, not everything again.
- The repository lives in Dropbox but is only ever touched through rclone. **Do not put it in a folder the Dropbox desktop client syncs** — two processes writing the same repository is how repositories get corrupted.

## Install restic and rclone

Both tools are in Ubuntu's archive, but on 24.04 they lag upstream badly enough to matter:

```bash
sudo apt install restic
restic version
```

Ubuntu 24.04 ships **restic 0.16.4**, while upstream is well past 0.19. That gap is not cosmetic — the much-improved `restore` command, including its `--dry-run` and in-place overwrite modes, arrived in **restic 0.17.0**. If you want the restore options described later in this post, install the current release binary instead:

```bash
sudo apt install bzip2
RESTIC_VER=$(curl -s https://api.github.com/repos/restic/restic/releases/latest | grep -oP '"tag_name": "v\K[^"]+')
wget "https://github.com/restic/restic/releases/latest/download/restic_${RESTIC_VER}_linux_amd64.bz2" -O restic.bz2
bunzip2 restic.bz2 && chmod +x restic && sudo mv restic /usr/local/bin/
restic version
```

That drops the binary in `/usr/local/bin`, which precedes `/usr/bin` on the default `PATH`, so it wins over any apt-installed copy. Keep it current with `sudo restic self-update`.

rclone has the same problem, so take it from the official install script rather than apt:

```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
rclone version
```

The script checks what is already installed and skips the download if you are up to date, which makes it safe to re-run later.

## Connect rclone to Dropbox

The remote is created interactively:

```bash
rclone config
```

Walk through it like this:

1. `n` for a new remote, and name it **`dropbox`** — the name matters, because it becomes part of the repository path.
2. Storage type: `dropbox`.
3. `client_id` and `client_secret`: leave both blank to use rclone's built-in credentials. You can register your own app in the [Dropbox App Console](https://www.dropbox.com/developers/apps) and supply its key and secret instead; rclone documents the exact permissions to enable in its [Dropbox backend docs](https://rclone.org/dropbox/).
4. Accept the remaining defaults, say yes to auto config, and complete the login in the browser that opens.

On a **headless server** there is no browser to open. Run `rclone authorize "dropbox"` on a machine that has one, then paste the resulting token into the server's config prompt — the full procedure is in rclone's [remote setup docs](https://rclone.org/remote_setup/).

Confirm the remote works before going further:

```bash
rclone lsd dropbox:
```

That lists top-level directories in your Dropbox. If it errors, fix it here — nothing downstream will work until this command does.

## Create the password and initialise the repository

> ⚠️ **If you lose this password, every backup is permanently unreadable.** There is no recovery mechanism, by design. Put a copy in a password manager before you run anything else.

```bash
mkdir -p ~/.config/restic
echo 'a-long-random-password' > ~/.config/restic/password
chmod 600 ~/.config/restic/password
```

Use something long and random — `openssl rand -base64 32` is a reasonable source. The `chmod 600` keeps it readable only by your user.

restic takes the repository location and password from two environment variables, which saves repeating them on every command:

```bash
export RESTIC_REPOSITORY="rclone:dropbox:restic-backups-homeserver"
export RESTIC_PASSWORD_FILE="$HOME/.config/restic/password"
restic init
```

The repository string reads as `rclone:<remote>:<path>`, so this creates a folder named `restic-backups-homeserver` at the root of your Dropbox. Name it after the machine if you plan to back up more than one — each host wants its own repository unless you have a specific reason to share.

`restic init` writes the repository structure and a `config` file. It is a one-time operation; running it against an existing repository is refused rather than destructive.

To make bare `restic` commands work in your interactive shell, add those two `export` lines to `~/.bashrc` and run `source ~/.bashrc`.

## Write the backup script

The scheduled job is a plain shell script, which keeps it testable by hand:

```bash
mkdir -p ~/.local/bin
cat > ~/.local/bin/restic-backup.sh << 'EOF'
#!/bin/bash
set -euo pipefail

export RESTIC_REPOSITORY="rclone:dropbox:restic-backups-homeserver"
export RESTIC_PASSWORD_FILE="$HOME/.config/restic/password"

echo "=== Backup started: $(date) ==="

restic backup \
  "$HOME/Documents" \
  "$HOME/Pictures" \
  "$HOME/Projects" \
  --exclude-caches \
  --exclude "$HOME/.cache" \
  --exclude "node_modules" \
  --exclude "*.tmp"

restic forget \
  --keep-weekly 8 \
  --keep-monthly 12 \
  --keep-yearly 2 \
  --prune

echo "=== Backup finished: $(date) ==="
EOF
chmod +x ~/.local/bin/restic-backup.sh
```

Quoting the heredoc delimiter as `'EOF'` is deliberate: it stops the shell expanding `$HOME` and `$(date)` while writing the file, so those are evaluated when the script runs instead of being frozen to today's values.

`set -euo pipefail` makes the script abort at the first failure. That ordering matters — if `restic backup` fails, you do not want `restic forget --prune` running afterwards and deleting old snapshots to make room for a backup that never arrived.

Two commands do the work. `restic backup` walks the listed paths and uploads new or changed blocks; `--exclude-caches` honours the `CACHEDIR.TAG` convention, and the remaining `--exclude` patterns skip the usual noise. Then `restic forget` applies the retention policy — eight weekly, twelve monthly and two yearly snapshots — and `--prune` is what actually reclaims the space in Dropbox. Without `--prune`, `forget` only removes the snapshot labels and the data stays.

**Edit the path list to match your machine.** A path that does not exist makes the whole run fail:

```bash
nano ~/.local/bin/restic-backup.sh
```

Then run it once by hand, before involving systemd at all:

```bash
~/.local/bin/restic-backup.sh
```

The first run uploads everything and will take a while. Later runs are incremental.

## Schedule it with a systemd user timer

Two unit files: a service that says what to run, and a timer that says when.

```bash
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/restic-backup.service << 'EOF'
[Unit]
Description=Restic backup to Dropbox
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=%h/.local/bin/restic-backup.sh
Nice=10
IOSchedulingClass=idle
EOF

cat > ~/.config/systemd/user/restic-backup.timer << 'EOF'
[Unit]
Description=Weekly restic backup (Sunday 20:00)

[Timer]
OnCalendar=Sun *-*-* 20:00 Europe/Bucharest
Persistent=true
RandomizedDelaySec=10min

[Install]
WantedBy=timers.target
EOF
```

In the service, `Type=oneshot` tells systemd this runs to completion and exits rather than staying resident. `Wants=` and `After=network-online.target` hold it back until the network is actually up. `Nice=10` and `IOSchedulingClass=idle` push it to the back of the queue for CPU and disk, so a backup never makes the machine feel slow. `%h` expands to your home directory.

The timer needs no `Unit=` line, because a timer activates the service of the same name by default. The interesting part is the trailing timezone:

```ini
OnCalendar=Sun *-*-* 20:00 Europe/Bucharest
```

Appending an [IANA timezone](https://www.iana.org/time-zones) name pins the schedule to Romanian local time regardless of how the server's own clock is configured, and it keeps tracking DST correctly. Ubuntu 24.04 carries systemd 255, comfortably past the v250 release that introduced timezone suffixes — on anything older, drop the timezone and set the system clock instead.

`Persistent=true` records the last run, so a missed occurrence fires at the next boot rather than being lost. `RandomizedDelaySec=10min` smears the start time over ten minutes, which is politeness toward the Dropbox API if you ever run this on several machines.

Before enabling anything, have systemd check the expression and tell you when it will next fire:

```bash
systemd-analyze calendar "Sun *-*-* 20:00 Europe/Bucharest"
```

Then load and enable:

```bash
systemctl --user daemon-reload
systemctl --user enable --now restic-backup.timer
sudo loginctl enable-linger $USER
systemctl --user list-timers restic-backup.timer
```

`daemon-reload` re-reads the unit files — run it after every edit, or systemd keeps using the old copy. `enable --now` both enables the timer across reboots and starts it immediately.

`loginctl enable-linger` is the one people miss, and it is **required on a server**. Without it, your user's systemd instance is torn down when your last session ends, so the timer dies the moment you close SSH. Verify it took:

```bash
loginctl show-user $USER | grep Linger
```

It should print `Linger=yes`.

## Everyday commands

| Command | What it does |
| --- | --- |
| `systemctl --user start restic-backup` | Run a backup right now |
| `systemctl --user list-timers` | Show the next and last run times |
| `journalctl --user -u restic-backup -e` | Read the logs (`-f` to follow live, `q` to quit) |
| `restic snapshots` | List every snapshot in the repository |
| `restic stats` | Show the repository's size and deduplication savings |
| `restic check` | Verify repository integrity |

The `restic` commands need `RESTIC_REPOSITORY` and `RESTIC_PASSWORD_FILE` in the environment. If you did not add them to `~/.bashrc`, pass them per command instead:

```bash
restic -r rclone:dropbox:restic-backups-homeserver \
  --password-file ~/.config/restic/password snapshots
```

## Restoring

This is the half worth rehearsing. Do it once deliberately, while nothing is on fire.

### Find what you want

Every restore starts from a snapshot ID. List them:

```bash
restic snapshots
```

You get an ID, a timestamp, the host and the paths each snapshot covers. The literal word `latest` works anywhere an ID is accepted, which is usually what you want.

To see inside a snapshot before pulling anything down:

```bash
restic ls latest
restic ls latest /home/me/Documents
```

And to locate a file when you cannot remember where it lived or which snapshot still has it:

```bash
restic find "quarterly-report.ods"
```

`restic find` searches across snapshots and reports which ones contain the match — the fastest way to answer "when did I last have a good copy of this?".

### Restore everything

```bash
restic restore latest --target ~/restore
```

`--target` is the directory the tree is written into; the original paths are recreated underneath it. **Always restore to a new, empty directory** rather than over the live data, until you are certain of what you are getting.

To restore a specific snapshot instead of the newest, give its ID:

```bash
restic restore 79766175 --target ~/restore
```

### Restore a single file or folder

`--include` narrows what gets written:

```bash
restic restore latest --target ~/restore --include /home/me/Documents/taxes
```

You can also anchor the snapshot at a subfolder, which keeps the output shallow instead of recreating the whole path below the target:

```bash
restic restore latest:/home/me/Documents --target ~/restore --include /taxes
```

`--exclude` works the same way in reverse, and `--iinclude` / `--iexclude` are the case-insensitive variants.

Note that `--path` and `--host` do something different from what the names suggest: they **select which snapshot** gets restored, not which files come out of it. That is useful when one repository holds several machines:

```bash
restic restore latest --host homeserver --target ~/restore
```

### Preview a restore first

On restic 0.17.0 and newer, you can see what a restore would do without writing anything:

```bash
restic restore latest --target ~/restore --dry-run --verbose=2
```

Worth using before any in-place restore, since the dry run reports which files would be left alone and which would be overwritten. If you installed Ubuntu's 0.16.4 package, this flag does not exist — that is the main reason to take the upstream binary.

### Pull one file out without a full restore

For a single file, `dump` writes it straight to stdout:

```bash
restic dump latest /home/me/Documents/notes.md > notes.md
```

It handles directories too, as an archive stream:

```bash
restic dump latest /home/me/Documents > documents.tar
restic dump -a zip latest /home/me/Documents > documents.zip
```

This is often the quickest path when you only want to look at an old version of something.

### Browse the repository as a filesystem

```bash
mkdir ~/restic-mount
restic mount ~/restic-mount
```

Every snapshot appears as a directory tree you can navigate, `diff` against the live copy, or copy individual files out of with `cp`. It is read-only and needs FUSE, which Ubuntu has out of the box. `Ctrl+C` unmounts.

### Verify, on a schedule

Integrity checks and a practice restore belong in your calendar, roughly monthly:

```bash
restic check
restic check --read-data-subset=5%
restic restore latest --target /tmp/restore-test --include /home/me/Documents
```

Plain `restic check` validates the repository's structure and metadata. `--read-data-subset=5%` goes further and actually downloads and hashes a sample of the data, which is what catches silent corruption in the stored blocks. Checking everything with `--read-data` means re-downloading the entire repository, so a subset each month is the sensible compromise.

## Troubleshooting

**`Fatal: unable to open config file: <config/> does not exist`**

The repository path in your script does not match what is in Dropbox, or `restic init` was never run. Check both:

```bash
rclone lsd dropbox:
rclone ls dropbox:restic-backups-homeserver --max-depth 1
```

The second command should list a `config` file. If the directory is missing entirely, run `restic init`.

**`too_many_requests` — Dropbox rate limiting**

Throttle rclone's request rate. Any rclone flag can be set through an environment variable, which is the cleanest way to do it from the backup script:

```bash
export RCLONE_TPSLIMIT=10
```

Alternatively, override the arguments restic passes to rclone:

```bash
restic -o rclone.args="serve restic --stdio --tpslimit 10" snapshots
```

Be aware that this replaces the default argument list — `serve restic --stdio --b2-hard-delete` — rather than adding to it, so keep `serve restic --stdio` in place. `RCLONE_BWLIMIT` works the same way if you need to cap bandwidth instead of request rate.

**`didn't find section in config file`, or other rclone auth errors**

The remote name in the repository string does not match a configured remote:

```bash
rclone listremotes
```

It should print `dropbox:`. Note that a systemd user service reads `~/.config/rclone/rclone.conf` as your user, so a remote you configured under `sudo` will not be visible to the timer.

**The timer never ran**

Check its state, and whether linger is on:

```bash
systemctl --user list-timers
loginctl show-user $USER | grep Linger
journalctl --user -u restic-backup -e
```

A timer showing `n/a` for its next run is usually an enable or linger problem rather than a bad calendar expression — confirm the expression separately with `systemd-analyze calendar`.

## Official documentation

- [restic documentation](https://restic.readthedocs.io/), and the [restore chapter](https://restic.readthedocs.io/en/stable/050_restore.html) in particular
- [Preparing a new restic repository](https://restic.readthedocs.io/en/stable/030_preparing_a_new_repo.html), which covers the rclone backend and its options
- [rclone Dropbox backend](https://rclone.org/dropbox/) and [rclone installation](https://rclone.org/install/)
- `man systemd.timer` and `man systemd.time` for the full calendar syntax
