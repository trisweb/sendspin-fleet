# sendspin-fleet

A simple fleet management tool for maintaining and updating sendspin-cli installations across multiple Linux hosts via SSH.

## Features

- **Parallel Execution**: All operations run simultaneously across all hosts for maximum speed
- **Live TUI Display**: Beautiful terminal UI with per-host status lines that update in real-time
- **Animated Progress**: Spinners show which hosts are actively working, package counts during upgrades
- **Status Monitoring**: Check service status, version, and configured name across all hosts
- **System Updates**: Update system packages via `apt` on all hosts with per-package progress
- **Sendspin Updates**: Update sendspin-cli package via `uv` on all hosts
- **Batch Operations**: Update both system and sendspin packages in one command
- **Colorful Output**: Color-coded status indicators (green ✔, red ✘, yellow ⚠)
- **Flexible Configuration**: Support for different SSH users per host

## Prerequisites

- Bash 4.0 or higher
- SSH access to all target hosts with public key authentication configured
- Target hosts must be Debian-based Linux systems with `apt` package manager
- `uv` package manager installed on target hosts for sendspin updates
- sendspin-cli installed under a service user (default: `sendspin`)

## Installation

The script is a single self-contained file — just copy it to any directory in your `$PATH`:

```bash
# Download and install
curl -sL https://raw.githubusercontent.com/trisweb/sendspin-fleet/main/sendspin-fleet \
    -o ~/.local/bin/sendspin-fleet
chmod +x ~/.local/bin/sendspin-fleet
```

Or clone and symlink:
```bash
git clone https://github.com/trisweb/sendspin-fleet
ln -s "$(pwd)/sendspin-fleet/sendspin-fleet" ~/.local/bin/sendspin-fleet
```

## Configuration

### Hosts Configuration File

The script looks for hosts in `~/.config/sendspin-fleet-hosts.conf` by default. Create this file to get started:

```bash
mkdir -p ~/.config
cp hosts.conf.example ~/.config/sendspin-fleet-hosts.conf
# Edit with your actual hosts
$EDITOR ~/.config/sendspin-fleet-hosts.conf
```

Format: `hostname:ssh_user` (one per line):

```
# Production audio players
player1.local:root
player2.local:root
192.168.1.100:admin

# Development hosts
dev-player.local:myuser
audio-endpoint-01:sendspin
```

**Notes:**
- Lines starting with `#` are treated as comments
- Empty lines are ignored
- If no user is specified (no colon), defaults to `root`
- SSH must be configured for public key authentication (passwordless login)

### Environment Variables

Override defaults as needed:

| Variable | Default | Description |
|----------|---------|-------------|
| `HOSTS_FILE` | `~/.config/sendspin-fleet-hosts.conf` | Path to hosts configuration file |
| `SENDSPIN_USER` | `sendspin` | Service user name on remote hosts |

```bash
# Use a different hosts file
HOSTS_FILE=~/my-other-fleet.conf sendspin-fleet status

# Use a different service user
SENDSPIN_USER=audioplayer sendspin-fleet update-sendspin
```

## Usage

```bash
sendspin-fleet <command>
```

### Commands

#### `status`
Check the live status of all hosts in parallel:
- SSH connectivity
- Service status (running/stopped/failed)
- Sendspin version
- Configured device name from settings

```bash
sendspin-fleet status
```

**While running:**
```
ℹ Running on 3 host(s) in parallel...

⠋ player1.local: Checking service status...
⠙ player2.local: Retrieving version info...
⠹ player3.local: Connecting via SSH...
```

**When complete:**
```
✔ player1.local: ✔ Kitchen Speaker | 0.2.1 | active
✔ player2.local: ✔ Living Room | 0.2.1 | active
✔ player3.local: ✔ Bedroom | 0.2.1 | active

✔ All operations completed successfully
```

#### `update-system`
Update system packages on all hosts in parallel with live per-package progress:
```bash
sendspin-fleet update-system
```

Progress display during upgrade:
```
⠋ player1.local: Upgrading packages: 7/15
⠙ player2.local: Upgrading packages: 3/8
✔ player3.local: Upgraded 12 packages
```

#### `update-sendspin`
Update sendspin-cli via `uv` on all hosts in parallel:
```bash
sendspin-fleet update-sendspin
```

Shows per-host progress through upgrade and service restart.

#### `update-all`
Full update: system packages followed by sendspin, both in parallel:
```bash
sendspin-fleet update-all
```

#### `help`
```bash
sendspin-fleet help
```

## SSH Setup

For passwordless SSH authentication on each target host:

```bash
# Generate a key if needed
ssh-keygen -t ed25519 -C "sendspin-fleet"

# Copy to each host
ssh-copy-id user@hostname

# Verify
ssh user@hostname echo "OK"
```

## Troubleshooting

### "Hosts file not found"
Create the config file:
```bash
mkdir -p ~/.config
cp hosts.conf.example ~/.config/sendspin-fleet-hosts.conf
```
Or set `HOSTS_FILE` to an existing file.

### "Unable to connect via SSH"
1. Check reachability: `ping hostname`
2. Test SSH directly: `ssh user@hostname`
3. Ensure the user in your config matches the one with SSH key access

### Service Status Shows "Unknown/Not Found"
Verify the service name on the remote host:
```bash
ssh user@hostname systemctl list-units --type=service | grep sendspin
```

## Architecture

1. **Configuration Loading**: Reads hosts from `~/.config/sendspin-fleet-hosts.conf`
2. **Parallel Fork**: Spawns one background process per host
3. **Status Tracking**: Workers write status updates to temp files
4. **TUI Display Loop**: Main process redraws host lines at 10Hz using ANSI cursor control
5. **Progress Streaming**: For apt upgrades, output is streamed and parsed for package counts
6. **Process Management**: Waits for all jobs, reports overall success/failure

### Performance

With N hosts each taking T seconds:
- **Sequential**: N × T seconds
- **Parallel (this tool)**: ~T seconds

## Credits

Designed for [sendspin](https://github.com/trisweb/sendspin) audio player system.
