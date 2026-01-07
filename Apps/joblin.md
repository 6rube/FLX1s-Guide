# Installation of Joblin

1. Install Joblin from Flatpak
2. Edit the desktop shortcut
```bash
sudo nano /var/lib/flatpak/exports/share/applications/net.cozic.joplin_desktop.desktop
```
3. Replace the Exec to add dbus session for electron
```ini
Exec=dbus-run-session /usr/bin/flatpak run --branch=stable --arch=aarch64 --command=joplin-desktop --file-forwarding net.cozic.joplin_desktop @@u %u @@
```
4. Start Joplin (It needs a lot longer than other apps to start. For me it took up to 1 minute)
5. In Joblin go to View -> Change application layout -> Press on the left arrow of the menus on the left.
6. Go to View -> Change application layout again to save the layout.

