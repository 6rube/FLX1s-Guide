# Custom Stevia Keyboard (FuriOS)

Create your own keyboard using **English (US) as a base**, without changing your existing layout.

### 1. Create the custom file

```bash
mkdir -p ~/stevia-custom

curl -fL https://sources.debian.org/data/main/p/phosh-osk-stevia/0.58.0-1/src/layouts/us.json \
  -o ~/stevia-custom/us+intl.json
```

`us+intl.json` is your custom file. English is only the starting template.

### 2. Customize it

```bash
nano ~/stevia-custom/us+intl.json
```

Example — change:

```json
["q"]
```

to:

```json
["q", "1", "!", "?"]
```

Now long-pressing `q` offers additional characters.

Validate your changes:

```bash
python3 -m json.tool ~/stevia-custom/us+intl.json >/dev/null && echo "JSON OK"
```

### 3. Add the custom keyboard

Check your existing layouts:

```bash
gsettings get org.gnome.desktop.input-sources sources
```

For example, to keep German and add your custom keyboard:

```bash
gsettings set org.gnome.desktop.input-sources sources \
  "[('xkb', 'de'), ('xkb', 'us+intl')]"
```

Preserve any other existing layouts.

**Note:** The custom keyboard appears as *English (US, international)* because Stevia uses an existing XKB variant as its identifier.

### 4. Test it

```bash
G_RESOURCE_OVERLAYS="/mobi/phosh/stevia/layouts=$HOME/stevia-custom" \
  phosh-osk-stevia --replace
```

Switch to the new layout and test your changes.

### 5. Make it permanent

Add the overlay to Phosh's session environment:

```bash
echo 'export G_RESOURCE_OVERLAYS="/mobi/phosh/stevia/layouts=$HOME/stevia-custom"' >> ~/.phoshdebug
```

Reboot to reload the Phosh session.

This uses Phosh's startup configuration rather than relying on a temporary `--replace` process.

**Done.** Your custom layout is stored separately, and the original keyboard remains untouched.