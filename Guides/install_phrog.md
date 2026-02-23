# Phrog Greeter Setup Guide

## Important !
Only do this when having a reliable usb tehered SSH Connection.
Missconfiguration might lead to broken boot process. 
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
```ini
[terminal]
vt = 7

[default_session]
command = "/usr/libexec/phrog-greetd-session"
user = "_greetd"
```

### 4. Create Session Script (`/usr/libexec/phrog-greetd-session`)
```bash
#!/bin/bash
PHOC_INI=/etc/phrog/phoc.ini
[ ! -f $PHOC_INI ] && PHOC_INI=/usr/share/phrog/phoc.ini
[ ! -f $PHOC_INI ] && PHOC_INI=/etc/phosh/phoc.ini
[ ! -f $PHOC_INI ] && PHOC_INI=/usr/share/phosh/phoc.ini
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
```

### 5. Configure Display Output (`/usr/share/phrog/phoc.ini`)
```ini
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

### 7. Restart greetd
```bash
sudo systemctl reset-failed greetd && sudo systemctl restart greetd
```

# Improvements

## Systemd order
There is a problem in the dependency chain that causes sometimes issues where the user is stuck in the plymouth menu.
You can fix it by optimizing the dependency calls:

### 1. Remove plymouth-quit-wait from greetd
```bash
systemctl edit greetd.service
```
```ini
[Unit]
After=
After=systemd-user-sessions.service
After=getty@tty7.service
Conflicts=getty@tty7.service
```
### 2. Break the resolved→network.target blockage
```bash
systemctl edit systemd-user-sessions.service
```
```ini 
[Unit]
After=
After=local-fs.target
```
### 3. Prevent systemd-resolved.service from blocking

```bash
sudo systemctl edit systemd-resolved.service
```
```ini 
[Service]
RestartSec=0
```


