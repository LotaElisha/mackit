# MacKit by Lota — Command Reference

**Version 1.31.0 | Author: Lota (lotaelisha)**

Quick-reference sheet for all `mk` commands, flags, and environment variables.

---

## Command Overview

| Command | Description | Destructive? |
|---------|-------------|-------------|
| `mk flush` | Deep system cleanup | Yes |
| `mk wipe` | Smart app uninstaller | Yes |
| `mk turbo` | System optimizer | No |
| `mk scan` | Visual disk explorer | No (uses Trash) |
| `mk matrix` | Live system monitor | No |
| `mk nuke` | Project artifact cleaner | Yes |
| `mk trace` | Installer file finder | Yes |
| `mk auth` | Touch ID for sudo | System-config |
| `mk shell` | Shell tab completion | Config only |
| `mk sync` | Update MacKit | No |
| `mk eject` | Remove MacKit | Yes |
| `mk --help` | Show help | No |
| `mk --version` | Show version info | No |

---

## Detailed Command Reference

---

### `mk flush`

**Purpose:** Scans and removes system junk — caches, logs, browser data, and developer leftovers.

**Usage:**
```
mk flush [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--dry-run` / `-n` | Preview targets without deleting |
| `--debug` | Show detailed per-file operation logs |
| `--whitelist` | Open the interactive whitelist manager |
| `--json` | Output results as JSON |

**Examples:**
```bash
mk flush                         # Run cleanup
mk flush --dry-run               # See what would be removed
mk flush --dry-run --debug       # See everything in detail
mk flush --whitelist             # Manage protected paths
MK_NO_OPLOG=1 mk flush           # Skip operation logging
```

**Cleans:**
- `~/Library/Caches` (user app caches)
- `/Library/Caches` (system caches, requires sudo)
- Browser caches (Safari, Chrome, Firefox, Brave)
- Xcode derived data (`~/Library/Developer/Xcode/DerivedData`)
- Crash reports (`~/Library/Logs/DiagnosticReports`)
- Homebrew cache (`~/Library/Caches/Homebrew`)
- iOS device backups (optional)

---

### `mk wipe`

**Purpose:** Removes applications and all their hidden remnants — preferences, launch agents, support files.

**Usage:**
```
mk wipe [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--dry-run` / `-n` | Preview what would be removed |
| `--debug` | Show detailed operation logs |

**Examples:**
```bash
mk wipe                          # Interactive app selector
mk wipe --dry-run                # Preview removal
```

**Scans these locations for app remnants:**
- `/Applications/`
- `~/Applications/`
- `~/Library/Preferences/`
- `~/Library/Application Support/`
- `~/Library/Caches/`
- `~/Library/LaunchAgents/`
- `/Library/LaunchAgents/` (sudo)
- `/Library/LaunchDaemons/` (sudo)
- `~/Library/Containers/`
- `~/Library/Group Containers/`

---

### `mk turbo`

**Purpose:** Refreshes system caches and runs macOS maintenance tasks to boost performance.

**Usage:**
```
mk turbo [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--dry-run` / `-n` | Preview tasks without executing |
| `--debug` | Verbose output |
| `--whitelist` | Manage protected optimization rules |

**Examples:**
```bash
mk turbo                         # Run all optimizations
mk turbo --dry-run               # Preview tasks
mk turbo --whitelist             # Protect specific items
```

**Tasks:**
- Flush DNS cache (`dscacheutil -flushcache`)
- Rebuild Launch Services database
- Reset Spotlight index
- Clear font cache
- Rebuild icon cache
- Flush memory page cache (requires sudo)

---

### `mk scan`

**Purpose:** Interactive terminal UI for exploring disk usage and finding large files.

**Usage:**
```
mk scan [PATH] [OPTIONS]
```

**Arguments:**

| Argument | Description |
|----------|-------------|
| `PATH` | Directory to scan (default: `~`) |

**Examples:**
```bash
mk scan                          # Scan home directory
mk scan /                        # Scan entire disk
mk scan /Volumes/MyDrive         # Scan external drive
mk scan ~/Documents              # Scan specific folder
```

**Keyboard Shortcuts:**

| Key | Action |
|-----|--------|
| `↑/↓` or `j/k` | Navigate |
| `Enter` | Drill into folder |
| `Backspace` | Go up one level |
| `d` | Delete (moves to Trash) |
| `s` | Sort by size |
| `n` | Sort by name |
| `q` | Quit |

---

### `mk matrix`

**Purpose:** Real-time system health dashboard showing live metrics.

**Usage:**
```
mk matrix [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--json` | Output metrics as JSON (for scripting) |

**Examples:**
```bash
mk matrix                        # Launch live dashboard
mk matrix --json                 # JSON output for scripting
```

**Metrics Displayed:**
- CPU: per-core usage, load average
- GPU: activity percentage
- Memory: used, free, compressed, swap
- Disk: read/write I/O per second
- Network: bytes sent/received per second
- Battery: charge %, health, cycle count
- Processes: top 5 by CPU, top 5 by memory

**Keyboard:**

| Key | Action |
|-----|--------|
| `q` | Quit |
| `Tab` | Switch panel focus |

---

### `mk nuke`

**Purpose:** Finds and removes developer project artifacts — `node_modules`, `target/`, `dist/`, etc.

**Usage:**
```
mk nuke [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--dry-run` / `-n` | Preview targets without deleting |
| `--debug` | Show detailed logs |
| `--paths` | Configure project scan directories |

**Examples:**
```bash
mk nuke                          # Scan and nuke artifacts
mk nuke --dry-run                # Preview targets
mk nuke --paths                  # Edit scan directories
```

**Default artifacts removed:**

| Path Pattern | Type |
|-------------|------|
| `node_modules/` | Node.js dependencies |
| `.gradle/` | Gradle build cache |
| `target/` | Rust/Java build output |
| `dist/` | Frontend build output |
| `build/` | Build output |
| `__pycache__/` | Python bytecode |
| `.next/` | Next.js build cache |
| `.nuxt/` | Nuxt.js build cache |
| `.parcel-cache/` | Parcel bundler cache |

Configure scan paths:
```bash
mk nuke --paths
# or edit directly:
~/.config/mackit/purge_paths
```

---

### `mk trace`

**Purpose:** Finds installer files (`.pkg`, `.dmg`) that are no longer needed after installation.

**Usage:**
```
mk trace [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--dry-run` / `-n` | Preview without removing |
| `--debug` | Verbose output |

**Examples:**
```bash
mk trace                         # Find and remove installer files
mk trace --dry-run               # Preview files found
```

**Scans:**
- `~/Downloads/`
- `~/Desktop/`
- `/tmp/`

---

### `mk auth`

**Purpose:** Configures macOS to use Touch ID for `sudo` authentication.

**Usage:**
```
mk auth [enable|disable] [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `enable` | Enable Touch ID for sudo |
| `disable` | Disable Touch ID for sudo |
| `--dry-run` / `-n` | Preview changes to `/etc/pam.d/sudo` |

**Examples:**
```bash
mk auth enable                   # Enable fingerprint for sudo
mk auth disable                  # Revert to password for sudo
mk auth enable --dry-run         # Preview the PAM file change
```

**Note:** Requires `sudo` access. Changes `/etc/pam.d/sudo` to add `pam_tid.so`.

---

### `mk shell`

**Purpose:** Installs shell tab completion for `mackit` and `mk` commands.

**Usage:**
```
mk shell [bash|zsh|fish] [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `bash` | Generate/install for Bash |
| `zsh` | Generate/install for Zsh |
| `fish` | Generate/install for Fish |
| `--dry-run` / `-n` | Preview config changes |

**Examples:**
```bash
mk shell                         # Auto-detect shell and install
mk shell zsh                     # Install Zsh completion
mk shell bash                    # Install Bash completion
mk shell fish                    # Install Fish completion
eval "$(mk shell bash)"          # One-line inline install
```

After installing: restart terminal or `source ~/.zshrc`.

---

### `mk sync`

**Purpose:** Updates MacKit to the latest version.

**Usage:**
```
mk sync [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--force` / `-f` | Force reinstall even if already on latest |
| `--nightly` | Install latest unreleased main branch build |

**Examples:**
```bash
mk sync                          # Update to latest stable
mk sync --force                  # Force reinstall
mk sync --nightly                # Install nightly build (script install only)
```

---

### `mk eject`

**Purpose:** Completely removes MacKit from the system.

**Usage:**
```
mk eject [OPTIONS]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--dry-run` / `-n` | Preview what would be removed |

**Examples:**
```bash
mk eject                         # Remove MacKit
mk eject --dry-run               # Preview removal
```

**Removes:**
- `/usr/local/bin/mackit`
- `/usr/local/bin/mk`
- `~/.config/mackit/`
- `~/.cache/mackit/`
- `~/Library/Logs/mackit/`

---

## Global Options

These flags work with most commands:

| Flag | Description |
|------|-------------|
| `--debug` | Enable verbose debug output |
| `--dry-run` / `-n` | Preview without making changes |
| `--help` / `-h` | Show command help |
| `--version` / `-V` | Show version and system info |

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MK_DEBUG` | `0` | Set to `1` to enable debug mode globally |
| `MK_DRY_RUN` | `0` | Set to `1` to force dry-run mode globally |
| `MK_NO_OPLOG` | `0` | Set to `1` to disable operation logging |

**Examples:**
```bash
MK_DEBUG=1 mk flush              # Debug mode for flush
MK_NO_OPLOG=1 mk flush          # Skip logging to file
MK_DRY_RUN=1 mk wipe            # Force dry-run
```

---

## Log Files

| File | Contents |
|------|----------|
| `~/Library/Logs/mackit/mackit.log` | All file operations |

View the log:
```bash
tail -f ~/Library/Logs/mackit/mackit.log
```

---

## Configuration Files

| Path | Purpose |
|------|---------|
| `~/.config/mackit/purge_paths` | Custom directories for `mk nuke` to scan |
| `~/.config/mackit/whitelist` | Paths protected from `mk flush` |
| `~/.cache/mackit/` | Internal state and caches |

---

*MacKit by Lota — github.com/lotaelisha/mackit | lotaelisha@gmail.com*
