# Installation of Signal UT

### Information
Signal UT is not a direct mobile client so you cant register or standalone login to signal you need a secondary android/ios device.
There is a work arround by using signal-cli see point 3.

1. Isntall Signal UT via Software Stroe
2. Replace wrong Ubuntuu Toch Directory with Furios
```bash
grep -RIl "/home/phablet" . | xargs sed -i 's|/home/phablet|/home/furios|g'
```

## Standalone with signal-cli

3. Install signal-cli
```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} \
  https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')

curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}.tar.gz"

sudo tar xf "signal-cli-${VERSION}.tar.gz" -C /opt
sudo ln -sf "/opt/signal-cli-${VERSION}/bin/signal-cli" /usr/local/bin/signal-cli
```

4. Install Java
```bash
 sudo apt install default-jre
 ```

5. make screenshot of qr code

```bash
sudo apt install zbar-tools
```

6. create script and add your phone number
```bash
nano ~/verify-flare-signal.sh
```
```sh
#!/usr/bin/env bash
set -euo pipefail

PHONE="+" # <------ Here goes the mobile number
SCREENSHOT_DIR="/home/furios/Pictures/Screenshots"

echo "Using phone number: $PHONE"
echo "Looking for latest screenshot in: $SCREENSHOT_DIR"

# Check dependencies
if ! command -v signal-cli >/dev/null 2>&1; then
  echo "ERROR: signal-cli not found."
  exit 1
fi

if ! command -v zbarimg >/dev/null 2>&1; then
  echo "ERROR: zbarimg not found. Please install zbar-tools first."
  exit 1
fi

# Find newest screenshot/image
LATEST_SCREENSHOT="$(
  find "$SCREENSHOT_DIR" -maxdepth 1 -type f \
    \( -iname '*.png' -o -iname '*.jpg' -o -iname '*.jpeg' -o -iname '*.webp' \) \
    -printf '%T@ %p\n' \
  | sort -nr \
  | head -n1 \
  | cut -d' ' -f2-
)"

if [[ -z "${LATEST_SCREENSHOT:-}" ]]; then
  echo "ERROR: No screenshot/image found in $SCREENSHOT_DIR"
  exit 1
fi

echo "Latest screenshot:"
echo "$LATEST_SCREENSHOT"
echo

# Extract QR/barcode data
QR_DATA="$(
  zbarimg --raw "$LATEST_SCREENSHOT" 2>/dev/null \
  | head -n1 \
  | tr -d '\r'
)"

if [[ -z "${QR_DATA:-}" ]]; then
  echo "ERROR: Could not read QR code from latest screenshot."
  echo "Try opening the screenshot and make sure the QR code is visible and not cropped."
  exit 1
fi

echo "QR/link data found:"
echo "$QR_DATA"
echo

case "$QR_DATA" in
  signalcaptcha://* )
    echo "Detected Signal captcha link."
    echo "Registering number..."
    signal-cli -u "$PHONE" register --captcha "$QR_DATA"

    echo
    read -rp "Enter SMS verification code: " SMS_CODE

    echo "Verifying..."
    signal-cli -u "$PHONE" verify "$SMS_CODE"

    echo
    echo "Registration/verification complete."
    ;;

  sgnl://linkdevice* | tsdevice:* | signal://linkdevice* )
    echo "Detected Signal/Flare device-link URI."
    echo "Adding Flare as linked device..."
    signal-cli -u "$PHONE" addDevice --uri "$QR_DATA"

    echo
    echo "Device linking complete."
    ;;

  *uuid*pub_key* | *uuid*public_key* )
    echo "Detected likely device-link URI."
    echo "Adding Flare as linked device..."
    signal-cli -u "$PHONE" addDevice --uri "$QR_DATA"

    echo
    echo "Device linking complete."
    ;;

  * )
    echo "ERROR: QR data does not look like a Signal captcha or device-link URI."
    echo "Extracted data was:"
    echo "$QR_DATA"
    exit 1
    ;;
esac

echo
echo "Testing signal-cli account state..."
signal-cli -u "$PHONE" receive || true

echo
echo "Done."
```
7. Run the script
```bash
~/verify-flare-signal.sh
```
