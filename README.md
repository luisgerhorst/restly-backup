# restly-backup

Wrapper around `restic` to automatically back up the root filesystem
when there is unmetered internet and wall power. 

Assumes you are running Gnome and only use one repo per machine. Uses `crontab`
to run backups automatically.

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
filesystem (i.e., no external drives or temporary / cache files). It will prompt
you for the backup destination (the restic repo) and encryption password.

Config files:
- `crontab -e`: Set `$USER` and `$PATH`, invokes `restly-cron` 
- `~/.authinfo.d/restly_$REPO_password`
- `~/.config/restly/$REPO.conf`
  - Backup destination and encryption password
  - Can also specify additional options, e.g, retention policy by setting `RESTIC_FORGET`
    - See script for the default policy.
- `~/.config/restly/$DIR/conf` 
- `~/.config/restly/$DIR/exclude`
- `~/.config/restly/$DIR/exclude.$HOSTNAME`
