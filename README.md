# farrier

**Owner-approved patching as a standalone CLI.** `farrier plan` shows a dry-run; `farrier apply` snapshots, upgrades,
and reports before/after plus the rollback command.

**Detect-only by default** — it never changes anything until you approve.

![Licence: AGPL-3.0](https://img.shields.io/badge/licence-AGPL--3.0-blue) [![Buy Me a Coffee](https://raw.githubusercontent.com/casareanderson/.github/main/badges/bmc-yellow.svg)](https://buymeacoffee.com/iamc_tech)

`farrier` is the standalone version of warden's patcher. It pairs with
[sieve](https://github.com/casareanderson/sieve) (find the vulnerabilities),
[proctor](https://github.com/casareanderson/proctor) (review the PRs) and
[ephor](https://github.com/casareanderson/ephor) (watch the agents). Part of the [warden family](#the-warden-family).

## What it does

```
$ farrier plan myserver
plan for myserver:
  apt: 14 to upgrade/install, 1 to REMOVE
    libssl3, openssh-server, curl, rsyslog, ...
  npm: 2 outdated
    dompurify@3.4.16 -> 3.4.17 (sanity-studio)
  pip: 3 outdated
    requests, urllib3, certifi
```

`farrier apply myserver` then:

1. takes a **Proxmox snapshot** (when `vmid` + `node` are configured),
2. runs `apt full-upgrade`, then `npm update` and `pip install --upgrade` (best-effort),
3. reports before/after plus the exact **rollback command**.

## Why

A patching script that applies everything the moment it runs is how you take a box down at 9am. `farrier` is built
around the judgement, not the reflex: it **shows you the plan first**, waits for a `YES`, snapshots, applies, and hands
you a rollback. It's the same approve-first flow as warden's patcher, without needing warden itself.

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

See [docs/EXAMPLES.md](docs/EXAMPLES.md) for a Proxmox + cron recipe, and [docs/SETUP.md](docs/SETUP.md) for the config.

## Configuration

`farrier.conf` (or `~/.config/farrier.conf`). A host is an SSH target; optional `vmid` + `node` give it a Proxmox
snapshot and rollback, and `hypervisor: true` stops it ever auto-rebooting that box.

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
- A plan that removes a critical package (`systemd`, `openssh-server`, `sudo`, a proxmox kernel meta, `libc6`) is
  **refused**.
- A snapshot is taken first wherever possible, and the rollback command is always in the report.
- One job at a time per target.

## Limits (read these)

- **SSH only.** It reaches boxes over `ssh` (sudo via `sudo sh -c` when asked); no agent, no daemon.
- **apt is the core; npm/pip are best-effort.** `apply` runs all three, but npm/pip failures are reported, not fatal —
  the apt result stands.
- **Physical hosts can't be snapshotted** — on those there's no rollback, only the before/after report.
- **It patches packages, not config.** It won't migrate a broken service or fix a bad rebase.

## The warden family

| Tool | Watches | Runs as |
|---|---|---|
| [warden](https://github.com/casareanderson/warden) | Intrusions, bans, vulnerabilities, patches and AI agents | A server + web console |
| [sieve](https://github.com/casareanderson/sieve) | Known vulnerabilities + plain-text secrets | A CLI: `pipx install sieve-scan` |
| [ephor](https://github.com/casareanderson/ephor) | What AI agents connect to, download and can access | A dependency-free CLI |
| [proctor](https://github.com/casareanderson/proctor) | Reviews a git diff, fact-checked against the patch | A CLI: `git diff | proctor diff -` |

## Licence

AGPL-3.0-or-later.
