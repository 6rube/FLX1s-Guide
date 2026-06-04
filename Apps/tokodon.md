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
export QT_OPENGL=es2
export QT_QUICK_BACKEND=opengl
export QTWEBENGINE_FORCE_USE_GBM=0
export QTWEBENGINE_CHROMIUM_FLAGS="--disable-gpu"
exec tokodon "$@"
EOF

chmod +x ~/.local/bin/tokodon-safe
```
