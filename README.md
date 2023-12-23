# restly-backup

Integrate `restic` with Gnome for automatic backups in the background. 

Features:
- With the default config (set up using `configure-root`):
  - Skip most caches, temporary files, and the Trash
  - Automatically run backups and maintenance tasks using `crontab`
  - Back up the root filesystem mounted at `/`
- Only run backups when the internet connection is not metered and wall power is available
- Limit CPU usage: Use only one thread and run with `(io)nice`
- Inhibit Gnome suspend for infrequent, long-running maintenance operations
- Can use encrypted SSH keys from the Gnome Keyring that are only unlocked at
  login to access the `restic` repository

Tested on Ubuntu 22.10 and Fedora 39.

## Install

``` sh
sudo apt install stow trash-cli restic
cd ~/.local/stow
git clone git@github.com:luisgerhorst/restly-backup.git 
stow --ignore=.git --ignore=LICENSE --ignore=README.md restly-backup
export PATH=$HOME/.local/bin:$PATH
restly help
```

## Setup

Run `restly configure-root` to create a config that backs up you machine's root
filesystem (i.e., skipping external drives and temporary / caches files). It
will prompt you for the backup destination (the restic repo) and encryption
password.

Config files:
- `crontab -e`: Set `$USER` and `$PATH`, invokes `restly-cron`
- `~/.authinfo.d/restly_$REPO_password`
- `~/.config/restly/$REPO.conf`
  - Backup destination and encryption password
  - Can also specify additional options, e.g., retention policy by setting `RESTIC_FORGET`
    - See script for the default policy.
- `~/.config/restly/$DIR/conf`
- `~/.config/restly/$DIR/exclude`
- `~/.config/restly/$DIR/exclude.$HOSTNAME`
