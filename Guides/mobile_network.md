## Mobile Data Bands

1. Check supported/current bands:

```bash
mmcli -m 0
```

2. Enable or disable bands here:

**Settings → Mobile Network → Enabled Bands**

---

## Fix APN

1. Check current APN:

```bash
cd /usr/share/ofono/scripts
./list-contexts
```

2. Find the context with:

```text
Type = internet
```

3. Turn **Mobile Data OFF**, then set:

```bash
./set-context-property 0 Name "VENDOR NAME"
./set-context-property 0 AccessPointName "VENDOR APN"
./set-context-property 0 Username "USERNAME"
./set-context-property 0 Password "PASSWORD"
```

4. If username/password are not required:

```bash
./set-context-property 0 Username ""
./set-context-property 0 Password ""
```

5. Then verify:

```bash
./list-contexts
```

6. Turn **Mobile Data ON** again.
