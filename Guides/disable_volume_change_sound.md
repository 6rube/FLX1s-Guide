# How to disable the volume change sound on the Flx1s

1. Make folder ~/.local/share/sounds/__custom
2. Add index.theme file
```ini
[Sound Theme]
Name=Oma
Inherits=phosh
Directories=.
```
3. Add your custome oga file or the one provided here:
https://gitlab.com/Alaraajavamma/fastflx1/-/blob/main/share/sounds/__custom/audio-volume-change.oga?ref_type=heads
4. update current theme:
```bash
gsettings set org.gnome.desktop.sound theme-name __custom
```