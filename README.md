# proot-manager

> Single-file **rootless Linux container manager** — `pmgr` for short.
> Install, enter, snapshot, and manage full Linux distributions through proot.
> No root, no daemon.

## Why this exists

Distro managers built for Termux assume a lot — and Ubuntu's packaged `proot`
quietly breaks modern tools (its **relative `statx`** translation fails, so
`ls`/`apt` inside the guest start erroring on every file). `proot-manager` is
built from scratch to be:

- **statx-proof** — wired for the fixed proot **v5.5.0** static build
- **single-file** — one bash script, no dependencies to speak of
  (bash, curl, tar; python3 for `search`; crane only for image pulls)
- **store-aware** — every container, session, and backup lives under one
  predictable root (`$PM_HOME`)
- **safe by default** — destructive commands prompt; removing an *imported*
  rootfs demands an explicit `-y`

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/earlcypher/proot-manager/main/proot-manager \
  -o proot-manager
chmod +x proot-manager
```

Requirements:

| tool | needed for | notes |
|---|---|---|
| `proot` ≥ 5.5.0 | everything | set `PM_PROOT=/path/to/proot` if not default |
| `crane` | `install IMAGE` pulls | optional; URL/file/debootstrap installs work without it |
| `debootstrap` | `--debootstrap` installs | `apt-get install -y debootstrap` |

## Usage

```
proot-manager <command> [arguments]
```

**Containers**

| command | what it does |
|---|---|
| `list` | containers, sizes, live sessions |
| `install <src> [--name N]` | `IMAGE:TAG`, URL, local tarball, or `--debootstrap SUITE` |
| `import <dir> --name N` | adopt an existing rootfs in place |
| `login <name> [dir]` | interactive shell |
| `run <name> <cmd…>` | one-shot command |
| `remove <name> [-y]` | delete container + rootfs |
| `rename <old> <new>` | rename |
| `reset <name> -y` | rebuild from the recorded source |

**Sessions** — `ps` · `kill <name|all>`

**Files** — `copy <src> <dst>` (`name:path` on either side) · `backup <name> [-o file]` · `restore <name> <file>`

**Discovery** — `search <query>` (Docker Hub) · `version` · `help`

## Examples

```bash
# Docker image → container
proot-manager install ubuntu:24.04 --name ubuntu

# Debian via debootstrap
proot-manager install --name deb --debootstrap trixie \
  --mirror http://deb.debian.org/debian

# adopt a rootfs you already have
proot-manager import /path/to/rootfs --name mine

# work with it
proot-manager login ubuntu
proot-manager copy ./note.txt ubuntu:/root/note.txt
proot-manager backup ubuntu
```

## Store layout

```
$PM_HOME/                  default: ~/.local/share/proot-manager
├── proot-manager          this script (pmgr → symlink)
├── registry/<name>.pm     metadata (ROOTFS, SOURCE, CREATED)
├── containers/<name>/     rootfs of installed containers
├── sessions/<name>.pid    live login sessions
└── backups/               tar snapshots
```

Overrides: `PM_HOME`, `PM_PROOT`, `PM_CRANE`, `NO_COLOR`.

---
