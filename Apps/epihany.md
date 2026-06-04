# GNOME Web / Epiphany Web App Install Fix

Found a solution for an issue with **GNOME Web (Epiphany)**.

## The issue

Epiphany’s built-in **Install as Web Application** button goes through:

```text
xdg-desktop-portal DynamicLauncher.Install
```

That process runs:

```text
xdg-desktop-portal-validate-icon --sandbox
```

to validate the icon.

The sandbox uses `bwrap` to create namespaces, which fails silently on **kernel 4.19**. As a result, the web app install is rejected every time.

## Options

There are two ways to fix or bypass the issue:

1. **Permanent fix** — patch the validator
   Requires one-time `sudo`.

2. **Manual creation** — bypass the portal entirely
   Does not require `sudo`.

---

# Prerequisites

## 1. Install Epiphany

Use the non-Flatpak version:

```bash
sudo apt install epiphany-browser
```

## 2. Fix the missing Android library

Epiphany detects Andromeda and tries to load an Android video codec library.

Without it, you may get startup errors.

Download `libI420colorconvert.so` from:

```text
https://github.com/Sanjis-Android-Playground/dump/blob/master/libI420colorconvert.so
```

After downloading:

```bash
sudo cp libI420colorconvert.so /usr/local/lib/
sudo ldconfig
```

---

# Option A — Permanent Fix

This wraps the icon validator and strips the `--sandbox` flag, which breaks on kernel `4.19`.

After this, Epiphany’s GUI install button should work normally.

```bash
sudo mv /usr/libexec/xdg-desktop-portal-validate-icon \
        /usr/libexec/xdg-desktop-portal-validate-icon.real

sudo tee /usr/libexec/xdg-desktop-portal-validate-icon << 'EOF'
#!/bin/bash
args=()
for arg in "$@"; do
    [ "$arg" = "--sandbox" ] && continue
    args+=("$arg")
done
exec /usr/libexec/xdg-desktop-portal-validate-icon.real "${args[@]}"
EOF

sudo chmod +x /usr/libexec/xdg-desktop-portal-validate-icon
systemctl --user restart xdg-desktop-portal
```

Then in Epiphany:

```text
Navigate to a site → hamburger menu → Install as Web Application
```

---

# Option B — Manual Creation

Use this if you have not applied **Option A**, or if you want to script web app creation.

This bypasses the portal entirely and does not require `sudo`.

## Required files

For an app named `<id>` at `<url>`, you need:

```text
~/.local/share/epiphany/web-apps/org.gnome.Epiphany.WebApp_<id>/
├── .app
└── app-icon.png

~/.local/share/applications/
└── org.gnome.Epiphany.WebApp_<id>.desktop

~/.local/share/xdg-desktop-portal/applications/
└── org.gnome.Epiphany.WebApp_<id>.desktop
```

### Notes

* `.app` is an empty marker file.
* `.app` is critical. Without it, Epiphany may crash with:

```text
web app settings outside web app mode
```

* `app-icon.png` should be a `512×512` RGBA PNG.

---

# Step-by-step Script

Save this as:

```text
create-webapp.sh
```

Then make it executable:

```bash
chmod +x create-webapp.sh
```

## Script

```bash
#!/bin/bash

# Usage:
#   ./create-webapp.sh <id> <name> <url> <icon_url>
#
# Example:
#   ./create-webapp.sh nextcloud "Nextcloud" https://nextcloud.com/ https://nextcloud.com/core/img/favicon-touch.png

ID="$1"
NAME="$2"
URL="$3"
ICON_URL="$4"

APP_ID="org.gnome.Epiphany.WebApp_${ID}"
PROFILE="$HOME/.local/share/epiphany/web-apps/${APP_ID}"

# 1. Create profile directory
mkdir -p "$PROFILE"

# 2. Create .app marker
#
# Epiphany checks for this to detect a web app profile.
# Without it, Epiphany may crash with:
#   "web app settings outside web app mode"
touch "$PROFILE/.app"

# 3. Download and convert icon to 512×512 RGBA PNG
curl -sL "$ICON_URL" -o /tmp/webapp-icon-src.png

python3 - << PYEOF
from PIL import Image

img = Image.open("/tmp/webapp-icon-src.png").convert("RGBA").resize((512, 512))
img.save("$PROFILE/app-icon.png", "PNG")
PYEOF

echo "Icon saved: $(file "$PROFILE/app-icon.png")"

# 4. Create desktop entry content
DESKTOP="[Desktop Entry]
Version=1.0
Name=${NAME}
Exec=epiphany --application-mode --profile=${PROFILE} ${URL}
Icon=${PROFILE}/app-icon.png
Type=Application
Terminal=false
Categories=Network;
StartupNotify=true
X-Purism-FormFactor=Workstation;Mobile;"

# 5. Install desktop file in both required locations
echo "$DESKTOP" > "$HOME/.local/share/applications/${APP_ID}.desktop"

mkdir -p "$HOME/.local/share/xdg-desktop-portal/applications/"
echo "$DESKTOP" > "$HOME/.local/share/xdg-desktop-portal/applications/${APP_ID}.desktop"

echo ""
echo "Done! Created web app '${NAME}' (${APP_ID})"
echo "Launch with:"
echo "  epiphany --application-mode --profile=${PROFILE} ${URL}"
```

---

# Example

```bash
./create-webapp.sh nextcloud \
  "Nextcloud" \
  "https://nextcloud.com/" \
  "https://nextcloud.com/core/img/favicon-touch.png"
```

---

# Key Rules

| Rule                     | Detail                                                                        |
| ------------------------ | ----------------------------------------------------------------------------- |
| Profile directory prefix | Must be `org.gnome.Epiphany.WebApp_`                                          |
| Separator                | Use an underscore `_`, not a hyphen `-`                                       |
| `.app` file              | Must exist as an empty file in the profile directory                          |
| Desktop filename         | Must use the same underscore naming: `org.gnome.Epiphany.WebApp_<id>.desktop` |
| Desktop file locations   | Must exist in both `applications/` and `xdg-desktop-portal/applications/`     |
| Icon format              | Must be `512×512` RGBA PNG                                                    |
| Portal icon validation   | Indexed or palette PNGs may be rejected                                       |
