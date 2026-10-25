# farrier — worked examples

## 1. Dry-run first

```bash
./farrier plan myserver
```

## 2. Apply with a snapshot + rollback

```bash
./farrier apply mycontainer --yes
# patched mycontainer. reboot: not needed. npm skipped; pip updated.
# rollback: pct rollback 101 farrier-202610250930 on pve@192.0.2.1
```

## 3. A Proxmox LXC (snapshot + auto-rollback command)

```yaml
# farrier.conf
hosts:
  blog:
    ssh: root@192.0.2.11
    vmid: "110"
    node: pve@192.0.2.1
```

```bash
./farrier plan blog && ./farrier apply blog --yes
```

## 4. Nightly patch round in cron

```cron
# 03:00 every night: plan, then apply if it's safe (you've reviewed the plan separately)
0 3 * * * root /opt/farrier/farrier apply blog --yes >> /var/log/farrier.log 2>&1
```

## 5. Sudo on the target

```yaml
hosts:
  web:
    ssh: admin@192.0.2.20
    sudo: true        # runs `sudo sh -c "..."` on the box
```
