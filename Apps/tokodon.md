# Installation of Tokodon

1. Install Tokodon and it's dependecies
```bash
sudo apt update
sudo apt install --reinstall tokodon qt6-webengine-dev-tools libqt6webenginecore6 libqt6webenginequick6 qml6-module-qtwebengine libqt6svg6 qt6-svg-plugins
```

2. Update QT Setting Flags
```bash
cat > ~/.local/bin/tokodon-safe <<'EOF'
#!/bin/sh
export QTWEBENGINE_CHROMIUM_FLAGS="--disable-gpu"
exec tokodon "$@"
EOF

chmod +x ~/.local/bin/tokodon-safe
```
