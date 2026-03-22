# MacKit by Lota — Developer Guide

**Version 1.31.0 | Author: Lota (lotaelisha)**

Everything you need to set up, understand, and contribute to MacKit.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Structure](#project-structure)
3. [Development Setup](#development-setup)
4. [Architecture Overview](#architecture-overview)
5. [Running Tests](#running-tests)
6. [Code Style](#code-style)
7. [Building Binaries](#building-binaries)
8. [Contributing](#contributing)
9. [Release Process](#release-process)
10. [Security](#security)

---

## Prerequisites

**Required:**
- macOS 12 (Monterey) or later
- Bash 3.2+
- Go 1.21+ (for the TUI components)

**For development:**
- [ShellCheck](https://www.shellcheck.net/) — shell script linter
- [BATS](https://github.com/bats-core/bats-core) — Bash testing framework
- [golangci-lint](https://golangci-lint.run/) — Go linter

```bash
# Install development tools
brew install shellcheck bats-core go golangci-lint
```

---

## Project Structure

```
mackit/
├── mackit              # Main CLI entrypoint (Bash)
├── mk                  # Lightweight alias wrapper
├── install.sh          # Installer script
├── Makefile            # Build configuration
├── go.mod / go.sum     # Go module dependencies
│
├── bin/                # Subcommand scripts
│   ├── clean.sh        → mk flush
│   ├── uninstall.sh    → mk wipe
│   ├── optimize.sh     → mk turbo
│   ├── analyze.sh      → mk scan
│   ├── status.sh       → mk matrix
│   ├── purge.sh        → mk nuke
│   ├── installer.sh    → mk trace
│   ├── touchid.sh      → mk auth
│   └── completion.sh   → mk shell
│
├── lib/                # Shared shell libraries
│   ├── core/
│   │   ├── common.sh       # Shared utilities, update checks
│   │   ├── commands.sh     # Command list definition
│   │   ├── base.sh         # Core base functions
│   │   ├── file_ops.sh     # Safe file operations
│   │   ├── log.sh          # Logging functions
│   │   ├── ui.sh           # Terminal UI helpers
│   │   ├── sudo.sh         # Privilege management
│   │   └── timeout.sh      # Timeout utilities
│   ├── clean/          # mk flush modules
│   ├── uninstall/      # mk wipe modules
│   ├── optimize/       # mk turbo modules
│   ├── clean/          # Cleaning sub-modules
│   ├── manage/         # Update, whitelist, purge path management
│   └── check/          # Health check and diagnostics
│
├── cmd/                # Go TUI components
│   ├── analyze/        # mk scan (disk explorer TUI)
│   │   ├── main.go
│   │   ├── scanner.go
│   │   ├── view.go
│   │   └── ...
│   └── status/         # mk matrix (system monitor TUI)
│       ├── main.go
│       ├── metrics.go
│       ├── metrics_cpu.go
│       ├── metrics_memory.go
│       ├── metrics_disk.go
│       ├── metrics_network.go
│       ├── metrics_battery.go
│       └── ...
│
├── tests/              # BATS test suite
│   ├── clean_core.bats
│   ├── purge.bats
│   ├── update.bats
│   └── ...
│
├── scripts/            # Build and utility scripts
│   ├── check.sh        # Lint all scripts
│   ├── test.sh         # Run full test suite
│   └── setup-quick-launchers.sh  # Raycast/Alfred integration
│
├── .github/
│   ├── workflows/      # GitHub Actions CI/CD
│   └── ISSUE_TEMPLATE/
│
└── docs/               # Documentation (you are here)
```

---

## Development Setup

```bash
# 1. Clone the repository
git clone https://github.com/LotaElisha/mackit.git
cd mackit

# 2. Install development dependencies
brew install shellcheck bats-core golangci-lint

# 3. Build Go TUI binaries
make build

# 4. Install pre-commit hooks
cp .githooks/* .git/hooks/
chmod +x .git/hooks/*

# 5. Run a quick sanity check
./mackit --version
./mackit --help
```

**Running directly during development (without installing):**
```bash
# From the repo root:
./mackit flush --dry-run
./mackit scan ~/Downloads
./mackit matrix
```

---

## Architecture Overview

MacKit uses a **two-layer architecture**:

### Layer 1: Shell (Bash)

The main entry point (`mackit`) and all subcommand scripts (`bin/*.sh`) are written in Bash. This ensures zero installation overhead and maximum compatibility with macOS.

**Command flow:**
```
User runs: mk flush
  → mk (wrapper) → mackit → case "flush" → bin/clean.sh
                                           ↓
                             sources lib/core/common.sh
                             sources lib/core/file_ops.sh
                             sources lib/clean/caches.sh
                             sources lib/clean/app_caches.sh
                             sources lib/clean/system.sh
                             ... runs cleanup tasks
```

**Key shell conventions:**
- All scripts start with `set -euo pipefail` for strict error handling
- Shared utilities live in `lib/core/`
- Each `bin/` script handles its own argument parsing
- `lib/core/commands.sh` defines the canonical command list (used by help and tab completion)

### Layer 2: Go (TUI Components)

`mk scan` and `mk matrix` are Go programs using the [Bubbletea](https://github.com/charmbracelet/bubbletea) TUI framework. They are compiled to binaries and embedded into the installation.

**Go dependency overview:**

| Package | Purpose |
|---------|---------|
| `charmbracelet/bubbletea` | TUI framework (Elm architecture) |
| `charmbracelet/lipgloss` | Terminal styling |
| `shirou/gopsutil` | System metrics (CPU, memory, disk, network) |
| `cespare/xxhash` | Fast file hashing for `mk scan` |
| `golang.org/x/sync` | Concurrency utilities |

---

## Running Tests

### Full Test Suite

```bash
./scripts/test.sh
```

This runs:
1. ShellCheck on all shell scripts
2. Bash syntax checks
3. BATS tests (30+ test files)
4. Go tests (`go test ./...`)
5. Install/uninstall integration test

### Run Specific Tests

```bash
# Run a single BATS file
bats tests/purge.bats

# Run all BATS tests
bats tests/

# Run Go tests
go test ./cmd/...

# Run Go tests with verbose output
go test -v ./cmd/analyze/...
go test -v ./cmd/status/...
```

### Lint Only

```bash
./scripts/check.sh
```

---

## Code Style

### Shell Scripts

- All scripts must pass `shellcheck` with zero warnings
- Use `set -euo pipefail` at the top of every script
- Use `local` for all function-scoped variables
- Quote all variable expansions: `"$var"` not `$var`
- Prefer `[[ ]]` over `[ ]` for conditionals
- Use `readonly` for constants
- Log with `log_success`, `log_error`, `log_info` from `lib/core/log.sh`

```bash
# Good style example
my_function() {
    local input="$1"
    local result

    if [[ -z "$input" ]]; then
        log_error "Input is required"
        return 1
    fi

    result=$(process "$input") || return 1
    log_success "Done: $result"
}
```

### Go Code

- Follow standard Go conventions (`gofmt`, `golangci-lint`)
- Use the Bubbletea Elm architecture pattern: `Model`, `Update`, `View`
- Keep system metrics polling in the `metrics_*.go` files
- Keep rendering logic in `view.go`

**Run linter:**
```bash
golangci-lint run ./...
```

---

## Building Binaries

The Go TUI components are compiled for macOS (amd64 and arm64):

```bash
# Build both architectures
make build

# Build for Apple Silicon only
make build-arm64

# Build for Intel only
make build-amd64

# Clean build artifacts
make clean
```

The binaries are placed in `bin/` as part of the distribution.

**Release build (creates archives):**
```bash
make release
```

---

## Contributing

### Before You Start

1. Check existing [issues](https://github.com/LotaElisha/mackit/issues) for duplicates
2. For large changes, open an issue first to discuss the approach
3. Fork the repo and create a feature branch

### Workflow

```bash
# Fork and clone
git clone https://github.com/YOUR_USERNAME/mackit.git
cd mackit
git remote add upstream https://github.com/LotaElisha/mackit.git

# Create a branch
git checkout -b feature/my-feature

# Make changes, then run checks
./scripts/check.sh
./scripts/test.sh

# Commit (conventional commits preferred)
git commit -m "feat: add my feature"

# Push and open a pull request
git push origin feature/my-feature
```

### Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/):

| Type | When to use |
|------|-------------|
| `feat:` | New feature or command |
| `fix:` | Bug fix |
| `docs:` | Documentation changes |
| `refactor:` | Code cleanup without behavior change |
| `test:` | Adding or fixing tests |
| `chore:` | Build system, CI, tooling |

**Examples:**
```
feat: add mk nuke --paths flag for custom scan directories
fix: mk auth fails silently when PAM file is read-only
docs: add developer guide to docs/
test: add BATS tests for mk flush whitelist handling
```

### Adding a New Command

1. Create `bin/mycommand.sh` — the main command script
2. Add the command to `lib/core/commands.sh` in `MACKIT_COMMANDS`
3. Add a case entry in `mackit` (the main dispatch script)
4. Add BATS tests in `tests/mycommand.bats`
5. Document it in `docs/command_reference.md`

```bash
# Template for a new bin/ script
#!/bin/bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
source "$SCRIPT_DIR/lib/core/common.sh"
source "$SCRIPT_DIR/lib/core/file_ops.sh"

# Parse args
DRY_RUN=false
for arg in "$@"; do
    case "$arg" in
        --dry-run | -n) DRY_RUN=true ;;
        --debug) export MK_DEBUG=1 ;;
        *) echo "Unknown option: $arg"; exit 1 ;;
    esac
done

# Main logic here
main() {
    log_info "Running my command..."
    # ...
}

main "$@"
```

---

## Release Process

Releases are automated via GitHub Actions (`.github/workflows/release.yml`).

**Steps:**
1. Update `VERSION` in `mackit` (main script)
2. Update `CHANGELOG.md` (if present)
3. Commit: `git commit -m "chore: bump version to X.Y.Z"`
4. Tag: `git tag -a vX.Y.Z -m "Release X.Y.Z"`
5. Push tag: `git push origin vX.Y.Z`
6. GitHub Actions automatically:
   - Runs all tests
   - Builds Go binaries (amd64 + arm64)
   - Creates a GitHub Release with archives and SHA256 checksums
   - Updates the Homebrew formula (if configured)

---

## Security

### Reporting Vulnerabilities

Report security issues to: **lotaelisha@gmail.com**

Do not file public issues for security vulnerabilities.

### Safety Boundaries

MacKit is designed to be safe by default:

- **Protected paths** are hardcoded and can never be cleaned, regardless of user input
- **Path traversal** protection: all paths are validated before any operation
- **No network calls** except for update checks (which are opt-in)
- **No analytics** or telemetry — MacKit never phones home
- **All operations are logged** to `~/Library/Logs/mackit/mackit.log`

See `SECURITY.md` and `SECURITY_AUDIT.md` for the full security design.

---

*MacKit by Lota — github.com/LotaElisha/mackit | lotaelisha@gmail.com*
