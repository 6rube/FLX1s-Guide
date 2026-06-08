# Simple guide: Sync Pictures to Nextcloud with rclone

This guide syncs:

```bash
/home/furios/Pictures
```

to your Nextcloud server at:

```text
example.localhost
```

---

## 1. Create a Nextcloud app password

In the Nextcloud web UI, go to:

```text
Personal settings → Security → Devices & sessions → Create new app password
```

Copy the generated app password. You will use it when setting up `rclone`.

---

## 2. Install rclone

```bash
sudo apt install rclone
```

---

## 3. Configure rclone

Start the rclone setup:

```bash
rclone config
```

Create or edit a remote with these settings:

```text
type: webdav
url: https://example.localhost/remote.php/dav/files/YOUR_USERNAME
vendor: nextcloud
user: YOUR_USERNAME
pass: YOUR_APP_PASSWORD
```

Example remote name:

```text
Nextcloud.example.localhost
```

---

## 4. Test the connection

```bash
rclone lsd Nextcloud.example.localhost:
```

If it lists folders from your Nextcloud, the connection is working.

---

## 5. Copy Pictures to Nextcloud

This copies your local Pictures folder to Nextcloud:

```bash
rclone copy /home/furios/Pictures Nextcloud.example.localhost:Photos/Pictures
```

---

## 6. Check the sync

```bash
rclone check /home/furios/Pictures Nextcloud.example.localhost:Photos/Pictures
```

---

# Run automatically every 30 minutes

## 7. Create the systemd user folder

```bash
mkdir -p ~/.config/systemd/user
```

---

## 8. Create the sync service

```bash
nano ~/.config/systemd/user/pictures-nextcloud-sync.service
```

Paste this:

```ini
[Unit]
Description=Sync Pictures to Nextcloud

[Service]
Type=oneshot
ExecStart=/usr/bin/rclone copy /home/furios/Pictures Nextcloud.example.localhost:Photos/Pictures --log-file=/home/furios/.cache/pictures-nextcloud-sync.log --log-level INFO
```

Save and exit.

---

## 9. Create the timer

```bash
nano ~/.config/systemd/user/pictures-nextcloud-sync.timer
```

Paste this:

```ini
[Unit]
Description=Run Pictures Nextcloud sync every 30 minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=30min
Persistent=true

[Install]
WantedBy=timers.target
```

Save and exit.

---

## 10. Enable and start the timer

Use `systemctl --user`, not `sudo systemctl --user`:

```bash
systemctl --user daemon-reload
systemctl --user enable --now pictures-nextcloud-sync.timer
```

---

## 11. Check status

Check the timer:

```bash
systemctl --user status pictures-nextcloud-sync.timer
```

Check the service:

```bash
systemctl --user status pictures-nextcloud-sync.service
```

View logs:

```bash
cat /home/furios/.cache/pictures-nextcloud-sync.log
```

---

## Optional: keep the timer running after logout

```bash
sudo loginctl enable-linger furios
```