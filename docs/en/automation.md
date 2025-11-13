# Network Automation

## Why Automate?

Network automation reduces manual errors, saves time, and ensures consistency across your infrastructure. After exploring this lab environment manually, you can start automating common tasks like:

- Running diagnostic commands across multiple devices
- Backing up configurations regularly
- Collecting system information for analysis
- Managing firmware updates

This guide shows you an example on how to get started using the fully open source tool [networka](https://narrowin.github.io/networka/), a multi-vendor network automation CLI toolkit.

## Step 1: Install networka

Install networka using uv or pipx:

```bash
uv tool install git+https://github.com/narrowin/networka.git
```

Verify the installation:

```bash
nw --version
```

## Step 2: Initialize Configuration

Run the initialization command:

```bash
nw config init
```

This creates a configuration directory at `~/.config/networka` (Linux/macOS) with:

- `devices/` - Device definitions
- `groups/` - Device groups and tags  
- `sequences/` - Reusable command sequences
- `.env` - Credentials

Follow the prompts to install shell completions and optional vendor sequences.

## Step 3: Define Your Lab Devices

Edit `~/.config/networka/devices/devices.yml` to add the lab devices:

```yaml
devices:
  router1:
    host: 192.168.100.11
    device_type: mikrotik_routeros
    platform: x86
    description: "Gateway router"
    tags:
      - gateway
      - critical

  switch1:
    host: 192.168.100.12
    device_type: mikrotik_routeros
    platform: x86
    description: "Access switch"
    tags:
      - switch
      - access
```

Verify your devices are loaded:

```bash
nw list devices
```

You should see both devices with their IP addresses and descriptions.

## Step 4: Run Your First Command

Check interface status on the router using a predefined sequence:

```bash
nw run router1 interface_status
```

This connects to the device and runs multiple commands to show:

- All interfaces with their status and configuration
- Traffic statistics (packets, bytes, errors, drops)
- Ethernet port details and speeds
- Current throughput for each interface

The output is formatted and easy to read, with clear sections for each command result.

## Step 5: Explore More Sequences

Networka includes many built-in sequences for common operations. View available sequences:

```bash
nw run router1 <TAB>
```

Try these useful sequences:

```bash
# Quick device overview
nw run router1 quick_status

# Detailed system information
nw run router1 system_info

# Check routing table
nw run router1 routing_info

# Full health check
nw run router1 health_check
```

## Step 6: Back Up Configurations

Create configuration backups with vendor-aware handling:

```bash
# Configuration export
nw backup config router1

# Comprehensive backup (config + system data)
nw backup comprehensive router1
```

The backups are saved to your results directory with timestamps.

## Step 7: Work with Multiple Devices

Run commands on multiple devices at once:

```bash
# Comma-separated list
nw run router1,switch1 quick_status

# Run a command across devices
nw run router1,switch1 "/system/resource/print"
```

For production use, create device groups and run commands across entire groups efficiently.

## What's Next?

Once comfortable with these basics:

1. **Create custom sequences** - Define your own command workflows in YAML
2. **Set up device groups** - Organize devices by role, location, or function
3. **Schedule backups** - Use cron or systemd timers for automated backups
4. **Explore multi-vendor** - networka supports MikroTik, Cisco, Arista, Juniper, and more

Read the [full documentation](https://narrowin.github.io/networka/) for advanced features like:

- Custom command sequences
- Tag-based device filtering
- Results storage and analysis
- CI/CD integration
- Multi-vendor workflows
