# Phrog Greeter Setup Guide

## Important !
Only do this when having a reliable SSH Connection.
Misconfiguration can lead to broken boot process. 
SSH is still able to start since it is running via Andromeda layer. 


### 1. Install & Configure Services
```bash
sudo apt install phrog libc6-dev
sudo systemctl enable greetd
sudo systemctl disable phosh
sudo systemctl mask phosh
```

### 2. Add greetd User to Groups
```bash
sudo usermod -aG tty,video,input,render _greetd
```

### 3. Configure greetd Session (`/etc/greetd/phrog.toml`)
```bash
sudo tee /etc/greetd/phrog.toml >/dev/null <<'EOF'
[terminal]
vt = 7

[default_session]
command = "/usr/libexec/phrog-greetd-session"
user = "_greetd"
EOF
```

### 4. Create Session Script (`/usr/libexec/phrog-greetd-session`)
```bash
sudo tee /usr/libexec/phrog-greetd-session >/dev/null <<'EOF'
#!/bin/bash
PHOC_INI=/etc/phrog/phoc.ini
[ ! -f "$PHOC_INI" ] && PHOC_INI=/usr/share/phrog/phoc.ini
[ ! -f "$PHOC_INI" ] && PHOC_INI=/etc/phosh/phoc.ini
[ ! -f "$PHOC_INI" ] && PHOC_INI=/usr/share/phosh/phoc.ini

export XDG_CURRENT_DESKTOP=Phosh:GNOME
export GNOME_SESSION_AUTOSTART_DIR=/usr/share/phrog/autostart:/etc/phrog/autostart
export LIBSEAT_BACKEND=logind
export WLR_BACKENDS=hwcomposer,libinput
export EGL_PLATFORM=hwcomposer
export WLR_HWC_SKIP_VERSION_CHECK=1

if command -v systemd-cat >/dev/null 2>&1; then
  exec > >(systemd-cat --identifier=phrog) 2>&1
elif command -v logger >/dev/null 2>&1; then
  exec > >(logger -s -t phrog) 2>&1
fi

dbus_command=""
[ -z "$DBUS_SESSION_BUS_ADDRESS" ] && dbus_command=dbus-run-session

exec $dbus_command phoc -S -C "${PHOC_INI}" -E 'gnome-session --session=phrog'
EOF

sudo chmod 755 /usr/libexec/phrog-greetd-session
sudo chown root:root /usr/libexec/phrog-greetd-session
```

### 5. Configure Display Output (`/usr/share/phrog/phoc.ini`)
```bash
sudo tee /usr/share/phrog/phoc.ini >/dev/null <<'EOF'
[output:Virtual-1]
modeline = 87.25  720 776 848 976  1440 1443 1453 1493 -hsync +vsync
mode = 720x1440
scale = 2

[output:X11-1]
mode = 360x720

[output:WL-1]
mode = 360x720

[output:HEADLESS-1]
mode = 720x1440
scale = 2
EOF
```

### 6. Patch Phosh Desktop Entry
```bash
sudo cp /usr/share/wayland-sessions/phosh.desktop /usr/share/wayland-sessions/phosh.desktop.bak

sudo tee /usr/share/wayland-sessions/phosh.desktop << 'EOF'
[Desktop Entry]
Name=Phosh
Comment=Phone Shell
Exec=env WLR_BACKENDS=hwcomposer,libinput EGL_PLATFORM=hwcomposer WLR_HWC_SKIP_VERSION_CHECK=1 phosh-session
Type=Application
DesktopNames=Phosh;GNOME;
EOF
```

### 7. Make greetd Wait for the Android Display Stack
phrog reaches the display through libhybris, which loads Android's own libraries from the
Android container. If greetd starts before `lxc@android` is up and the hwcomposer is ready,
phrog exits immediately with `library "libc.so" not found`. systemd restarts it a few times,
hits the start limit (`start-limit-hit`) and gives up, leaving the "Furi" plymouth screen.
This only happens on some boots, depending on how fast the Android side comes up.

`phosh.service` avoids this with a wait in `/usr/lib/systemd/system/phosh.service.d/10-wait.conf`.
Give greetd the same wait:
```bash
sudo mkdir -p /etc/systemd/system/greetd.service.d
sudo tee /etc/systemd/system/greetd.service.d/override.conf >/dev/null <<'EOF'
[Unit]
After=lxc@android.service
StartLimitIntervalSec=0

[Service]
ExecStartPre=bash -c "WAITFORSERVICE_VALUE=1 waitforservice vendor.furios.hwcomposer.ready && sleep 3"
TimeoutStartSec=120
Restart=always
RestartSec=3
EOF
sudo systemctl daemon-reload
```
`ExecStartPre` is the actual fix. The start-limit and restart settings are a safety net, so a
slow boot means a short wait instead of a dead screen. `TimeoutStartSec` is raised because the
wait counts against the start timeout.

> If a future FuriOS update changes the drop-ins in `/usr/lib/systemd/system/phosh.service.d/`
> (for example the wait condition), mirror the change here.

### 8. Stop the Greeter from Holding the Audio Device
The `_greetd` user gets its own systemd user session, which starts its own sound server.
On this phone only one process can hold the audio hardware, so the greeter's sound server
can grab it first and leave your Phosh session with "No Output Devices" and
"No Input Devices" in Settings → Sound.

Mask the audio services for the `_greetd` user only:
```bash
GH=$(getent passwd _greetd | cut -d: -f6)
sudo -u _greetd mkdir -p "$GH/.config/systemd/user"
for u in pulseaudio.service pulseaudio.socket pipewire.service pipewire.socket \
         pipewire-pulse.service pipewire-pulse.socket wireplumber.service; do
  sudo -u _greetd ln -sf /dev/null "$GH/.config/systemd/user/$u"
done
```

### 9. Reboot
```bash
sudo reboot
```
