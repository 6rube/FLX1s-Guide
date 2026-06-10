# Installation of Signal UT

### Information
Signal UT is not a direct mobile client so you cant register or standalone login to signal you need a secondary android/ios device.

1. Isntall Signal UT via Software Stroe
2. Replace wrong Ubuntuu Toch Directory with Furios
```bash
grep -RIl "/home/phablet" . | xargs sed -i 's|/home/phablet|/home/furios|g'
```