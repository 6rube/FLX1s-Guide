# Installation of SimpleX Chat

1. Install SimpleX Chat from Flatpak
2. apply fix
```bash
flatpak override --user \
  --env=DISPLAY=:0 \
  --env=SKIKO_RENDER_API=SOFTWARE \
  --env=_JAVA_AWT_WM_NONREPARENTING=1 \
  chat.simplex.simplex
```