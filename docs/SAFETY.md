# farrier — safety

## The rules, and why

### Never auto-reboot a hypervisor

One Proxmox node reboot takes every guest on it down — and a node on slow disks can have a 10–15 minute IO storm
after boot. `farrier` upgrades a hypervisor but **never** reboots it; the report says a reboot is needed and you do it
in a quiet window.

### Refuse to remove critical packages

A plan that would remove `systemd`, `openssh-server`, `sudo`, a `proxmox-*` kernel meta, `grub` or `libc6` is refused
outright (`NEVER_REMOVE`). Those removals are how an unattended `full-upgrade` bricks a box; they are patched by hand.

### Snapshot first, rollback always

When `vmid` + `node` are set, `farrier` snapshots before it touches anything, and the report always carries the
`pct rollback` command. On a physical host there is no snapshot — the report says so.

### One job at a time

A second `apply` for the same target while one is running is refused, so two upgrades never fight over the same box.

### npm/pip are best-effort, apt is the contract

`apply` runs apt, then npm, then pip. If apt succeeds, the patch is reported as done even if npm or pip failed — the
message says which one skipped.
