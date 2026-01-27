# How to Debug Flatpak Apps on the Flx1s
## Get Debug Log
Some Flatpaks may fail to start, so it’s useful to obtain the debug log of that app.

1. Get the application ID:
```bash
flatpak list
```
2. Run the app from the terminal using the application ID (example: example.example.example):
```bash
flatpak run {Application ID}
```
Errors produced by the app will appear in the terminal — copy and share them if needed.

## Some Flatpak Packages Are Broken or Won’t Start
Currently, Flatpak Qt6 applications do not build for GLES (OpenGL).
This can lead to various issues that cannot be easily fixed.
It is recommended to install these applications via apt or directly from the developer’s website.

For some people it got fixed by setting this variable:
```bash
export QT_SCALE_FACTOR_ROUNDING_POLICY=Round
```
