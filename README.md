# restly-backup

Integrate `restic` with Gnome for automatic backups in the background. 

Features:
- With the default config (set up using `configure-root`):
  - Back up the filesystems mounted at `/` and `/home`
  - Skip most caches, temporary files, and the Trash
  - Automatically run backups and maintenance tasks from `crontab`
  - Keep 48 hourly, 14 daily, 8 weekly, 12 monthly, and 5 yearly backups
- Only run backups when the internet connection is not metered and wall power is available
- Run quietly in the background
  - When possible use Linux kernel cgroup CPU utilization clamping to request an
    energy-efficient core
  - Otherwise, pin ourself to a random core and lower that core's frequency to
    the minimum while we are running (tested on Lenovo ThinkPad X1 Carbon Gen 9
    (20XXS0Y800) with good results)
  - Use `ionice` and `nice`
- Inhibit Gnome suspend for long-running cleanup operations
- Can use encrypted SSH keys from the Gnome Keyring that are only unlocked at
  login to access the `restic` repository
- Automatically remove stale, server-side restic locks and try to auto-repair
  the restic index when backups fail
- When doing restic prune, first do a basic quick prune and then attempt a more
  throughout prune

Assumptions:
- Each machine has a separate restic repo which it uses exclusively
  - This allows restly to remove stale server-side restic locks without user-intervention

Tested on Ubuntu 22.10 and Fedora 39.

## Install

``` sh
sudo apt install restic stow trash-cli 
# OR 
sudo dnf install restic stow trash-cli

cd ~/.local/stow
git clone git@github.com:luisgerhorst/restly-backup.git 
stow --target=$HOME \
  --ignore=.git --ignore=.gitignore --ignore=LICENSE --ignore=README.md \
  restly-backup

# For quiet backups without kernel uclamp support:
echo "$USER	ALL=(ALL:ALL) NOPASSWD: /usr/bin/cpupower" | sudo tee /etc/sudoers.d/$USER-cpupower-for-restly

export PATH=$HOME/.local/bin:$PATH
restly help
```

## Setup

Run `restly configure` to create a config that backs up you machine's root
filesystem (i.e., skipping external drives and temporary / caches files). It
will prompt you for the backup destination (the restic repo) and encryption
password. It enables the automatic backups (using systemd timers for the
`default` repo).

For monitoring and testing the following might be helpful:

``` sh
systemctl --user start restly-backup
systemctl --user status restly-backup
journalctl --user -f -u restly-backup
systemctl --user list-timers
restly default cmd snapshots
```

Config files:
- systemd: Invokes `restly-cron`
- `~/.authinfo.d/restly_$REPO_password`
- `~/.config/restly/$REPO.sh`
  - Backup destination and encryption password
  - Can also specify additional options, e.g., retention policy by setting `RESTIC_FORGET`
    - See script for the default policy.
- `~/.config/restly/$DIR/conf.sh`
- `~/.config/restly/$DIR/exclude.txt`
