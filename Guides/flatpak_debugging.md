# How to debug Flatpak apps on the Flx1s


## Get Debug Log

Some Flatpaks do not start for that it is good to get the debug log of that app. 

1. Get the Application ID
```bash
flatpak list
```
2. Run the app from the terminal like this (Application ID syntax example.example.example)
```bash
flatpak run {Application ID}
```
3. Erros created by that app appeare in the terminal to copy and share.