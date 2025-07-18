# SSH Config Dropbox Sync

A Python script that intelligently syncs SSH configuration files between your Dropbox and local SSH directory.

## Features

- **Smart Merging**: Intelligently merges SSH config entries without duplicating existing configurations
- **Timestamp Checking**: Only syncs when the Dropbox config is newer than your local config
- **Backup Creation**: Automatically creates backups of your local config before making changes
- **Dry Run Mode**: Test what would be changed without actually modifying files
- **Detailed Logging**: Comprehensive logging to track what's happening during sync

## How It Works

1. Checks for SSH config file in `~/Dropbox/.ssh/config`
2. Compares modification timestamps with your local `~/.ssh/config`
3. If Dropbox config is newer (or no local config exists), parses both files
4. Intelligently merges configurations:
   - Adds new Host entries from Dropbox
   - Updates existing entries with Dropbox values (Dropbox takes precedence)
   - Preserves local-only entries
5. Creates a backup of your existing config
6. Writes the merged configuration

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

## Requirements

- Python 3.6 or higher
- Standard library only (no external dependencies)

## Safety Features

- **Automatic Backups**: Creates `.config.backup` files before modifications
- **Non-destructive**: Only adds or updates entries, never removes local configurations
- **Dry Run Mode**: Test changes safely before applying them
- **Error Handling**: Comprehensive error handling with helpful messages

## SSH Config Format Support

The script supports standard SSH config format including:
- Host entries with various configuration options
- Comments (preserved in output)
- Standard SSH client options (HostName, User, Port, IdentityFile, etc.)

## Example

If your Dropbox config contains:
```
Host myserver
    HostName example.com
    User admin
    Port 2222
```

And your local config has:
```
Host localhost
    HostName 127.0.0.1
    User root
```

The merged result will contain both entries, with the Dropbox entry taking precedence for any conflicts.
