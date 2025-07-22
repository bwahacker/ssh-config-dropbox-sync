# SSH Config Bidirectional Sync with Hash Tracking

A Python script that intelligently syncs SSH configuration files between your Dropbox and local SSH directory in both directions, with comprehensive version tracking.

## Features

- **Bidirectional Sync**: Syncs in both directions - always uses the newest config regardless of location
- **Smart Merging**: Intelligently merges SSH config entries without duplicating existing configurations
- **Hash-based Optimization**: Uses SHA256 hashes to avoid unnecessary syncs when content is identical
- **Config History Tracking**: Tracks when each unique config version was first seen and where
- **Timestamp Checking**: Compares modification times to determine which config is newer
- **Backup Creation**: Automatically creates backups before making changes to either location
- **Dry Run Mode**: Test what would be changed without actually modifying files
- **Detailed Logging**: Comprehensive logging to track what's happening during sync

## How It Works

1. Checks for SSH config files in both `~/Dropbox/.ssh/config` and `~/.ssh/config`
2. Calculates SHA256 hashes and tracks them in `~/Dropbox/.ssh/config_history.json`
3. **Hash comparison first**: If configs have identical content, no sync needed (even if timestamps differ)
4. **Timestamp comparison**: If content differs, compares modification times to determine sync direction:
   - **Dropbox → Local**: When Dropbox config is newer or no local config exists
   - **Local → Dropbox**: When local config is newer or no Dropbox config exists
   - **No sync**: When timestamps are identical (within 1 second) to avoid conflicts
5. Intelligently merges configurations:
   - Adds new Host entries from the source
   - Updates existing entries with source values (newer file takes precedence)
   - Preserves target-only entries
6. Creates a backup of the target config
7. Writes the merged configuration and updates hash tracking

## Usage

### Basic sync:
```bash
./ssh_config_sync.py
```

### Dry run (see what would change without making changes):
```bash
./ssh_config_sync.py --dry-run
```

### Verbose logging:
```bash
./ssh_config_sync.py --verbose
```

### Combined dry run with verbose logging:
```bash
./ssh_config_sync.py --dry-run --verbose
```

### View config history:
```bash
./ssh_config_sync.py --show-history
```

## Requirements

- Python 3.6 or higher
- Standard library only (no external dependencies)

## Safety Features

- **Automatic Backups**: Creates `.config.backup` files before modifications
- **Non-destructive**: Only adds or updates entries, never removes local configurations
- **Dry Run Mode**: Test changes safely before applying them
- **Error Handling**: Comprehensive error handling with helpful messages

## Hash-Based Config Tracking

The script maintains a history of all SSH config versions in `~/Dropbox/.ssh/config_history.json`:

- **Prevents unnecessary syncs**: Identical configs (different timestamps) won't trigger syncs
- **Tracks config evolution**: See when each unique config version was first created
- **Multi-location tracking**: Know which machines/locations have seen each config version
- **History viewing**: Use `--show-history` to see complete config timeline

Example hash tracking output:
```
Hash: efcf94280d0f...
  First seen: 2025-07-22T09:15:30.123456 in local
  Last seen: 2025-07-22T09:20:15.789012
  Locations: local, dropbox
```

## SSH Config Format Support

The script supports standard SSH config format including:
- Host entries with various configuration options
- Comments (preserved in output)
- Standard SSH client options (HostName, User, Port, IdentityFile, etc.)

## Example

**Scenario 1: Dropbox is newer**
If your Dropbox config (modified today) contains:
```
Host myserver
    HostName example.com
    User admin
    Port 2222
```

And your local config (modified yesterday) has:
```
Host localhost
    HostName 127.0.0.1
    User root
```

The script will sync **Dropbox → Local**, and the merged local result will contain both entries.

**Scenario 2: Local is newer**
If you update your local config with a new server and it becomes newer than Dropbox:
```
Host newserver
    HostName new.example.com
    User deploy
```

The script will sync **Local → Dropbox**, copying your local changes to Dropbox while preserving existing Dropbox entries.

**In both cases:** The newer file's entries take precedence for conflicts, but all unique entries from both files are preserved.
