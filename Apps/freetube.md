# Installation of FreeTube

1. install the flatpak version
2. open freetube once. Then it creates the freetube folders and files in /home/furios/.var
3. In Terminal, run : 
``` bash
killall freetube
```  
, otherwise freetube overrides the ”maximized” parameter at launch. All instances of freetube have to finish before you make the modification.
4. modify the file
    1. open the .var/app/io.freetubeapp.FreeTube/config/FreeTube/settings.db in your text editor and add 
    
```bash
echo '{“_id”:”bounds”,”value”:{“x”:0,”y”:136,”width”:1200,”height”:800,”maximized”:true,”fullScreen”:false}}' > .var/app/io.freetubeapp.FreeTube/config/FreeTube/settings.db
``` 
5. Launch Freetube.
6. If crashes again, check the modified file. The parameter "maximized” has to be set to "true". If not, run close freetube, modify it, and restart.

## Known Issues

### choppy playback
Some people have experience with choppy playback. To solve this enter this in the terminal:
```bash
flatpak override --user \
  --env=ELECTRON_OZONE_PLATFORM=wayland \
  --env=ELECTRON_OZONE_PLATFORM_HINT=wayland \
  --env=ELECTRON_ENABLE_GPU=1 \
  io.freetubeapp.FreeTube
```
if that did not help you can reset the override with: 
```bash
flatpak override --user --reset io.freetubeapp.FreeTube
```
