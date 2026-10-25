# farrier

Owner-approved patching as a standalone CLI. **Detect-only by default**: it never changes anything until you approve.

`farrier plan` shows a dry-run; `farrier apply` (or `--yes`) then snapshots, upgrades, and reports before/after plus the
rollback command. It fixes what the scanners find — **apt, npm, pip, docker** — not just apt.

This is the standalone version of warden's patcher. It pairs with
[sieve](https://github.com/casareanderson/sieve) (find the vulnerabilities) and
[mentor](https://github.com/casareanderson/mentor) (review the PRs).

## Install

```bash
git clone https://github.com/casareanderson/farrier.git
cd farrier
pip install pyyaml   # optional — a plain ssh string works without it
```

## Use

```bash
./farrier plan myserver          # dry-run: what would change
./farrier apply myserver --yes   # snapshot, then apply, then report + rollback
```

## Config

`farrier.conf` (or `~/.config/farrier.conf`). A host is an SSH target; optional `vmid` + `node` give it a Proxmox
snapshot and rollback.

```yaml
hosts:
  myserver:
    ssh: root@192.0.2.10        # documentation IP
    sudo: false
    hypervisor: false
  mycontainer:
    ssh: root@192.0.2.11
    vmid: "101"
    node: pve@192.0.2.1
```

## Hard rules

- A hypervisor is upgraded but **never rebooted automatically** (one reboot takes every guest down).
- A plan that removes a critical package (`systemd`, `openssh-server`, `sudo`, a proxmox kernel meta) is **refused**.
- A snapshot is taken first wherever possible, and the rollback command is always in the report.
- One job at a time per target.

## Licence

AGPL-3.0.
