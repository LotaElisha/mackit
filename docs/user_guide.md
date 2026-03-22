# MacKit by Lota — User Guide

**Version 1.31.0 | Author: Lota (lotaelisha)**

MacKit is a powerful, all-in-one macOS system maintenance tool built for speed and safety. Think of it as CleanMyMac, AppCleaner, DaisyDisk, and iStat Menus — fused into a single command-line binary.

---

## Table of Contents

1. [Installation](#installation)
2. [Quick Start](#quick-start)
3. [Interactive Menu](#interactive-menu)
4. [Commands](#commands)
5. [Safety Features](#safety-features)
6. [Tips & Tricks](#tips--tricks)
7. [Uninstalling MacKit](#uninstalling-mackit)

---

## Installation

### Via Script (Recommended)

```bash
curl -fsSL https://raw.githubusercontent.com/lotaelisha/mackit/main/install.sh | bash
```

Optional flags:
- `-s latest` — install from the main branch (nightly)
- `-s 1.17.0` — install a specific version

### Manual Clone

```bash
git clone https://github.com/lotaelisha/mackit.git
cd mackit
./mackit --help
```

### Requirements

- macOS 12 (Monterey) or later
- Intel or Apple Silicon (M1/M2/M3)
- Bash 3.2+

---

## Quick Start

Run `mk` to open the interactive menu, or jump straight to a command:

```bash
mk               # Open interactive menu
mk flush         # Flush junk — deep cleanup
mk scan          # Scan disk — explore usage
mk matrix        # Matrix — live system monitor
```

Preview any destructive operation before running it:

```bash
mk flush --dry-run
mk wipe --dry-run
mk nuke --dry-run
```

---

## Interactive Menu

Typing `mk` with no arguments launches the full-screen interactive menu:

```
┌─────────────────────────────────┐
│  MacKit by Lota  v1.31.0        │
├─────────────────────────────────┤
│  1  Flush    Flush junk & free  │
│  2  Wipe     Wipe apps          │
│  3  Turbo    Turbo boost system │
│  4  Scan     Scan disk usage    │
│  5  Matrix   Live system stats  │
└─────────────────────────────────┘
```

**Navigation:**
- Arrow keys or `j/k` (Vim) to move
- `Enter` or number key to select
- `q` to quit

---

## Commands

### `mk flush` — Deep System Cleanup

Scans and removes junk: caches, old logs, browser leftovers, and more.

```bash
mk flush                    # Run cleanup
mk flush --dry-run          # Preview without deleting
mk flush --dry-run --debug  # Preview with detailed logs
mk flush --whitelist        # Manage protected paths
```

**What it cleans:**
- User & app caches (`~/Library/Caches`)
- System logs
- Browser caches (Safari, Chrome, Firefox)
- Xcode derived data & archives
- Homebrew cache
- Crash reports

---

### `mk wipe` — Smart App Uninstaller

Removes apps along with all their hidden remnants — preferences, launch agents, support files.

```bash
mk wipe                     # Select apps to remove
mk wipe --dry-run           # Preview what would be removed
```

**What it removes beyond the .app:**
- `~/Library/Preferences`
- `~/Library/Application Support`
- `~/Library/Caches`
- Launch agents and daemons
- Containers and sandboxed data

---

### `mk turbo` — System Optimizer

Refreshes system caches, resets spotlight, and runs maintenance tasks.

```bash
mk turbo                    # Run optimization
mk turbo --dry-run          # Preview tasks
mk turbo --whitelist        # Manage protected optimization rules
```

**Tasks performed:**
- Rebuild Launch Services database
- Flush DNS cache
- Reset Spotlight index
- Clear font caches
- Rebuild dyld shared cache

---

### `mk scan` — Visual Disk Explorer

An interactive terminal UI for exploring disk usage and finding large files.

```bash
mk scan                     # Scan home directory
mk scan /Volumes/MyDrive    # Scan a specific volume
mk scan /                   # Scan entire disk
```

**Features:**
- Interactive treemap-style navigation
- Large file detection
- Sort by size, name, or date
- Move files to Trash (safer than direct delete)
- Arrow keys + Enter to navigate

---

### `mk matrix` — Live System Monitor

A real-time terminal dashboard showing system health metrics.

```bash
mk matrix                   # Launch live monitor
```

**Metrics displayed:**
- CPU usage per core
- GPU activity
- Memory pressure and swap
- Disk I/O
- Network throughput (up/down)
- Battery status and health
- Top processes by CPU/memory

Press `q` to exit.

---

### `mk nuke` — Project Artifact Cleaner

Finds and removes build artifacts from developer projects — `node_modules`, `target/`, `dist/`, `.gradle`, etc.

```bash
mk nuke                     # Scan and remove artifacts
mk nuke --dry-run           # Preview targets
mk nuke --paths             # Configure scan directories
```

**Default artifact types removed:**
- `node_modules/`
- `.gradle/`
- `target/` (Rust/Java)
- `dist/` and `build/`
- `__pycache__/`
- `.next/`, `.nuxt/`

---

### `mk trace` — Installer File Finder

Locates installer `.pkg` and `.dmg` files that are no longer needed after installation.

```bash
mk trace                    # Find installer files
mk trace --dry-run          # Preview without removing
```

Scans common locations: `~/Downloads`, `~/Desktop`, `/tmp`.

---

### `mk auth` — Touch ID for Sudo

Configures macOS Touch ID to authenticate `sudo` commands — so you can use your fingerprint instead of typing your password.

```bash
mk auth enable              # Enable Touch ID for sudo
mk auth disable             # Disable Touch ID for sudo
mk auth enable --dry-run    # Preview changes
```

---

### `mk shell` — Tab Completion

Installs shell tab completion so you can press `Tab` to autocomplete `mk` commands.

```bash
mk shell                    # Auto-detect shell and install
mk shell bash               # Install for Bash
mk shell zsh                # Install for Zsh
mk shell fish               # Install for Fish
mk shell --dry-run          # Preview config changes
```

After installing, restart your terminal or run `source ~/.zshrc` (or your shell config).

---

### `mk sync` — Update MacKit

Updates MacKit to the latest version.

```bash
mk sync                     # Update to latest stable
mk sync --force             # Force reinstall latest stable
mk sync --nightly           # Update to latest unreleased build
```

---

### `mk eject` — Remove MacKit

Completely removes MacKit from your system.

```bash
mk eject                    # Remove MacKit
mk eject --dry-run          # Preview removal
```

---

## Safety Features

MacKit is built with safety-first defaults:

- **Protected directories** — System-critical paths are hardcoded and never touched
- **Dry-run mode** — Every destructive command supports `--dry-run` to preview before executing
- **Whitelist support** — Add paths you want to protect from cleanup
- **Explicit confirmation** — High-risk actions require you to confirm
- **Trash-first in Scan** — `mk scan` moves files to Trash via Finder rather than direct delete
- **Operation log** — All file operations are logged to `~/Library/Logs/mackit/mackit.log`

To disable operation logging:
```bash
MK_NO_OPLOG=1 mk flush
```

To enable debug output:
```bash
mk flush --debug
# or
MK_DEBUG=1 mk flush
```

---

## Tips & Tricks

**Check what will be deleted before running:**
```bash
mk flush --dry-run --debug
mk wipe --dry-run
```

**Navigate `mk scan` like a pro:**
- `j/k` or arrow keys: move up/down
- `Enter`: drill into a folder
- `Backspace`: go up a level
- `d`: delete (moves to Trash)
- `q`: quit

**Protect a cache from being flushed:**
```bash
mk flush --whitelist
```

**Combine with cron for scheduled cleanups:**
```bash
# Add to crontab (runs every Sunday at midnight)
0 0 * * 0 /usr/local/bin/mk flush
```

**Use JSON output for scripting:**
```bash
mk matrix --json
```

---

## Uninstalling MacKit

```bash
mk eject
```

Or manually:
```bash
rm -f /usr/local/bin/mackit /usr/local/bin/mk
rm -rf ~/.config/mackit ~/.cache/mackit ~/Library/Logs/mackit
```

---

*MacKit by Lota — github.com/lotaelisha/mackit | lotaelisha@gmail.com*
