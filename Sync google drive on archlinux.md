---
id: Sync google drive on archlinux
aliases: []
tags: []
areas:
  - "[[linux]]"
---

# Sync google drive

To sync Google Drive with `rclone` on [[Archlinux]] and automate the process
using `systemd`, follow this step-by-step guide:

---

## 🧰 1. **Install `rclone`**

```bash
sudo pacman -S rclone
```

---

## 🔑 2. **Configure Google Drive Remote**

Run the configuration wizard:

```bash
rclone config
```

Follow these prompts:

1. Choose `n` for a new remote.
2. Name it (e.g. `gdrive`).
3. Choose `drive` (Google Drive).
4. Say `no` to using auto-config (if you're in terminal-only), otherwise `yes`.
5. Follow the authentication steps in the browser or manually paste the
   verification code.
6. Choose the default settings for the rest unless you need team drive or
   specific configs.

You can confirm the setup with:

```bash
rclone ls gdrive:
```

---

## 📂 3. **Choose Your Sync Folder**

Let’s say:

- Local folder: `~/DriveSync`
- Remote folder on Google Drive: `MySyncFolder`

You can test a sync with:

```bash
rclone sync ~/DriveSync gdrive:MySyncFolder --progress
```

Or reverse:

```bash
rclone sync gdrive:MySyncFolder ~/DriveSync --progress
```

> **Note**: `sync` makes destination identical to source (deletes extra files).

---

## ⚙️ 4. **Create a Systemd Service to Automate Sync**

### Step 1: Create a script to run `rclone`

Create the file:

```bash
mkdir -p ~/.config/rclone
nano ~/.config/rclone/sync.sh
```

Example content (change to your sync direction):

```bash
#!/bin/bash

rclone sync ~/DriveSync gdrive:MySyncFolder --log-file ~/.config/rclone/rclone.log --log-level INFO
```

Make it executable:

```bash
chmod +x ~/.config/rclone/sync.sh
```

#### Step 2: Create a systemd service unit

```bash
$EDITOR ~/.config/systemd/user/rclone-sync.service
```

Content:

```ini
[Unit]
Description=Sync Google Drive with rclone
After=network-online.target

[Service]
Type=oneshot
ExecStart=/home/yourusername/.config/rclone/sync.sh
```

Replace `/home/yourusername/` with your actual username path.

#### Step 3: Create a systemd timer unit

```bash
nano ~/.config/systemd/user/rclone-sync.timer
```

Content:

```ini
[Unit]
Description=Run rclone sync every 15 minutes

[Timer]
OnBootSec=5m
OnUnitActiveSec=15m
Unit=rclone-sync.service

[Install]
WantedBy=default.target
```

This syncs every 15 minutes after boot.

#### Step 4: Enable the timer

```bash
systemctl --user daemon-reexec
systemctl --user daemon-reload
systemctl --user enable --now rclone-sync.timer
```

To check:

```bash
systemctl --user status rclone-sync.timer
journalctl --user -u rclone-sync.service
```

---

### ✅ Optional: Test it manually

```bash
systemctl --user start rclone-sync.service
```

---

### 🧹 Optional: Mount Google Drive

If you prefer a mounted drive instead of sync:

```bash
rclone mount gdrive: ~/DriveMount \
  --vfs-cache-mode writes \
  --daemon
```

You can automate that with another systemd service too.

---

If you want, I can generate the exact service + timer files with your specific
folder names.
