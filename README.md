# sendspin-fleet

A simple fleet management tool for maintaining and updating sendspin-cli installations across multiple Linux hosts via SSH.

## Features

- **Status Monitoring**: Check service status, version, and configured name across all hosts
- **System Updates**: Update system packages via `apt` on all hosts
- **Sendspin Updates**: Update sendspin-cli package via `uv` on all hosts
- **Batch Operations**: Update both system and sendspin packages in one command
- **Colorful Output**: Beautiful terminal interface with colored status messages
- **Flexible Configuration**: Support for different SSH users per host

## Prerequisites

- Bash 4.0 or higher
- SSH access to all target hosts with public key authentication configured
- Target hosts must be Debian-based Linux systems with `apt` package manager
- `uv` package manager installed on target hosts for sendspin updates
- sendspin-cli installed under a service user (default: `sendspin`)

## Installation

1. Clone or download this repository:
```bash
git clone <repository-url> sendspin-fleet
cd sendspin-fleet
```

2. Make the script executable:
```bash
chmod +x sendspin-fleet
```

3. Create your hosts configuration file:
```bash
cp hosts.conf.example hosts.conf
```

## Configuration

### Hosts Configuration

Edit `hosts.conf` to define your fleet of hosts. Format: `hostname:ssh_user` (one per line).

Example `hosts.conf`:
```
# Production audio players
player1.local:root
player2.local:root
192.168.1.100:admin

# Development hosts
dev-player.local:myuser
audio-endpoint-01:sendspin
```

**Important Notes:**
- Lines starting with `#` are treated as comments
- Empty lines are ignored
- If no user is specified (no colon), defaults to `root`
- SSH must be configured for public key authentication (passwordless login)

### Environment Variables

- `HOSTS_FILE`: Path to hosts configuration file (default: `hosts.conf`)
- `SENDSPIN_USER`: Service user name for sendspin (default: `sendspin`)

Example with custom configuration:
```bash
HOSTS_FILE=production-hosts.conf SENDSPIN_USER=audioplayer ./sendspin-fleet status
```

## Usage

```bash
./sendspin-fleet <command>
```

### Available Commands

#### `status`
Check the status of all hosts in your fleet:
- SSH connectivity
- Service status (running/stopped/failed)
- Sendspin version
- Configured device name from settings

```bash
./sendspin-fleet status
```

#### `update-system`
Update system packages on all hosts using `apt`:
```bash
./sendspin-fleet update-system
```

Executes: `sudo apt update && sudo apt upgrade -y`

#### `update-sendspin`
Update the sendspin-cli package on all hosts using `uv`:
```bash
./sendspin-fleet update-sendspin
```

Executes: `sudo -u sendspin uv tool upgrade sendspin`

Also restarts the sendspin service after updating.

#### `update-all`
Update both system packages and sendspin-cli on all hosts:
```bash
./sendspin-fleet update-all
```

This is equivalent to running `update-system` followed by `update-sendspin`.

#### `help`
Display usage information:
```bash
./sendspin-fleet help
```

## SSH Setup

For passwordless SSH authentication, you need to set up public key authentication on all target hosts.

### Generate SSH Key (if you don't have one)
```bash
ssh-keygen -t ed25519 -C "sendspin-fleet"
```

### Copy Public Key to Each Host
```bash
ssh-copy-id user@hostname
```

Replace `user@hostname` with the actual SSH user and hostname from your `hosts.conf`.

### Test SSH Connection
```bash
ssh user@hostname echo "Connection successful"
```

If this works without prompting for a password, you're all set!

## Examples

### Basic Status Check
```bash
./sendspin-fleet status
```

### Update Everything on All Hosts
```bash
./sendspin-fleet update-all
```

### Update Only System Packages
```bash
./sendspin-fleet update-system
```

### Update Only Sendspin
```bash
./sendspin-fleet update-sendspin
```

### Custom Configuration
```bash
# Use a different hosts file
HOSTS_FILE=production-hosts.conf ./sendspin-fleet status

# Use a different service user name
SENDSPIN_USER=audioplayer ./sendspin-fleet update-sendspin

# Combine both
HOSTS_FILE=prod.conf SENDSPIN_USER=audioplayer ./sendspin-fleet update-all
```

## Troubleshooting

### "Hosts file not found"
Make sure you've created a `hosts.conf` file in the same directory as the script. You can copy from the example:
```bash
cp hosts.conf.example hosts.conf
```

### "Unable to connect via SSH"
1. Verify the hostname is correct and reachable: `ping hostname`
2. Check SSH public key authentication is set up: `ssh user@hostname`
3. Ensure the SSH user in `hosts.conf` matches the user you've set up keys for

### "tui-toolkit.sh not found"
Download the bash-tui-toolkit:
```bash
curl -sL https://raw.githubusercontent.com/timo-reymann/bash-tui-toolkit/main/tui-toolkit.sh -o tui-toolkit.sh
```

### Service Status Shows "Unknown/Not Found"
The service name may not match the `SENDSPIN_USER`. Check the actual service name:
```bash
ssh user@hostname systemctl list-units --type=service | grep sendspin
```

Then adjust the `SENDSPIN_USER` environment variable accordingly.

## Architecture

The script follows a simple architecture:

1. **Configuration Loading**: Reads hosts from `hosts.conf`
2. **SSH Execution**: Uses SSH with public key auth to run commands remotely
3. **Command Dispatch**: Routes to appropriate handler based on subcommand
4. **Error Handling**: Gracefully handles connection failures and reports errors
5. **TUI Output**: Uses bash-tui-toolkit for colored, formatted output

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

This project is provided as-is for managing sendspin-cli deployments.

## Credits

- Uses [bash-tui-toolkit](https://github.com/timo-reymann/bash-tui-toolkit) for terminal UI
- Designed for [sendspin](https://github.com/trisweb/sendspin) audio player system
