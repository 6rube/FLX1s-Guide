# Installation and Fix for Tokodon

## 1. Install Tokodon

```bash
sudo apt update
sudo apt install --reinstall tokodon qt6-webengine-dev-tools libqt6webenginecore6 libqt6webenginequick6 qml6-module-qtwebengine libqt6svg6 qt6-svg-plugins qt6-webview-plugins
```

## 2. Fix the blank login WebView

Some devices show a blank login WebView because Qt WebEngine GPU rendering fails.

Copy Tokodon’s desktop launcher to your local applications folder:

```bash
mkdir -p ~/.local/share/applications
cp /usr/share/applications/org.kde.tokodon.desktop ~/.local/share/applications/
```

Patch the local launcher:

```bash
sed -i 's|^Exec=.*|Exec=env QTWEBENGINE_CHROMIUM_FLAGS="--disable-gpu" tokodon %u|' ~/.local/share/applications/org.kde.tokodon.desktop
```

## 3. Run Tokodon

Open Tokodon from the app menu.

If the app menu still uses the old launcher, run:

```bash
update-desktop-database ~/.local/share/applications 2>/dev/null || true
```

Then close and reopen the app menu.
