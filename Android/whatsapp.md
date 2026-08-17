 # Fix WhatsApp Media Downloads in Andromeda

## Information
This command fixes the issue where you cannot download videos, images, or audio files in WhatsApp. It changes the group ownership of media files to group ID 1023, which is required for proper access and download permissions in the Andromeda Android emulator's media directory.

## Fix Permissions

```bash
sudo chgrp -R 1023 ~/.local/share/andromeda/data/media/0/Android/
``` 