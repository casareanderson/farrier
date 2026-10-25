# farrier — setup

## SSH

`farrier` reaches boxes over `ssh` (key auth, non-interactive). Give it an SSH target:

```yaml
hosts:
  box: root@192.0.2.10
```

or just:

```bash
./farrier plan root@192.0.2.10
```

## Proxmox snapshot + rollback

To get a snapshot before every apply (and a rollback command in the report), set `vmid` + `node`:

```yaml
hosts:
  box:
    ssh: root@192.0.2.11
    vmid: "110"
    node: pve@192.0.2.1    # the Proxmox node that hosts the container
```

`farrier` runs `pct snapshot <vmid> farrier-<timestamp>` on the node first; if the upgrade fails it reports
`pct rollback <vmid> <snapshot>`.

## Hypervisor safety

Set `hypervisor: true` on a Proxmox node so `farrier` never auto-reboots it:

```yaml
hosts:
  pve:
    ssh: root@192.0.2.1
    hypervisor: true
```
