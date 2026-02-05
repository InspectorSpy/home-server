# Old gaming laptop to home-server. A setup journey.

A tale of transforming an old gaming laptop into a decently powerful, self-hosted home server running photo backup, Minecraft server, and more.

## Overview

This is a "guide" I wrote while setting up and configuring my old laptop to a self-hosted home server with:
- **CasaOS** - Easy-to-use web interface for managing services
- **Immich** - Self-hosted photo backup (a Google Photos alternative)
- **Minecraft server** - Fabric-based server with performance optimizations
- **Tailscale** - Secure Remote Access(SRA) from anywhere
- **Proper networking** - Port forwarding with security in mind 

## Hardware used
This server is an old HP Pavilion gaming laptop with the following specs:
- **CPU:** Ryzen 7 4800H (8-core/16-thread)
- **GPU:** NVIDIA GTX 1660 Ti Max-Q
- **RAM:** 16GB
- **Storage:** 512GB NVMe (OS) + 1TB 2.5" SSD (data)
The router I use:
- **Network:** MikroTIk Hex Refresh router (any router that supports port forwarding works)

**Minimum recommended:**

- Dual-core or quad-core CPU
- 8GB RAM
- 256GB+ storage
- Wired ethernet connection

## Important considerations

**Before you start:**
- Laptops are not designed for 24/7 operation, so monitor temperatures closely
- Remove the battery or use a cooling pad
- Power consumption will be higher than purpose-built servers (but possibly lower than desktop hardware)
- Ensure good ventilation to prevent overheating
- Regular maintenance and monitoring are pretty essential

## Prerequisities

- A laptop, almost any will do that has decent specs
- USB drive (8GB+) for OS installation
- Basic familiarity with command line (You will manage without, google helps)
- Router with port forwading capability (almost all routers these days support it)
- Static IP or DHCP reservation on your router 

## Quick start

### 1. Install Debian/Ubuntu (I used debian 13)

1. Download [Debian/Ubuntu]
(https://www.debian.org/distrib/netinst)/(https://ubuntu.com/download/server) (latest stable or testing)
2. Flash to USB using [Rufus](https://rufus.ie/) (Windows) or [Balena Etcher] (https://www.balena.io/etcher/)
I use [Ventoy](Link here) you don't have to, go with Rufus or Balena Etcher
3. Boot from USB and install:
    - **Skip desktop environment** (headless server basically)
    - Enable SSH server during installation
    - Set up user account with sudo access

### 2. Install CasaOS

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install CasaOS
curl -fsSL https://get.casaos.io | sudo bash

# If curl is not installed, install it
sudo apt install curl
```

Access CasaOS at: `http://YOUR_SERVER_IP`

### 3. Configure network

**Set static IP:**
- Configure DHCP reservation in your router
- Map your server's MAC address to a fixed IP (e.g., `192.168.1.100`)

**For MikroTik users:**
```
# Ensure server port is in bridge, not WAN interface list
# Bridge -> ports -> verify your port is added
# Interface List -> ensure only WAN port in "WAN" list
```

### 4. Install services via CasaOS

Access the CasaOS app store and install:

#### Immich (Photo backup)
- Configure storage path to your data drive
- Install mobile apps ([iOS] (https://apps.apple.com/app/immich/id1613945652) / [Android] (https://play.google.com/store/apps/details?id=app.alextran.immich))
- Enable auto-upload in mobile app settings
# Note:
# The base Immich app didn't work for me, so I used the "Without machine learning" version from the CasaOS app store

#### Crafty Controller (Minecraft management)
- Web-based Minecraft server control panel
- Simplifies server management and mod installation

### 5. Set up minecraft server

In Crafty Controller:

1. **Create new server**
    - Type: Fabric (for mod support)
    - Version: Latest stable (1.21.11 for me)
    - RAM: 8 - 10GB for modded, 4 - 6GB for vanilla

2. **Install performance mods** (server-side only. clients don't need them):
    ```properties
    online-mode=true        # Requires legitimate accounts
    white-list=true         # Only whitelisted players
    enable-rcon=false       # Disable remote console
    ```

4. **Whitelist players** via console:
    ```
    whitelist add Playername
    ```

### 6. Port forwarding

**For minecraft public access:**

In your router's settings:
- Protocol: TCP
- External port: 25565
- Internal IP: Your server's static IP
- Internal port: 25565

**Security best practices:**
- Only forward port 25565 (nothing else)
- Use Minecraft's whitelist feature
- Keep `online-mode=true` to prevent cracked clients
- Consider using Tailscale instead for friends-only servers

### 7. Install Tailscale (Secure Remote Access)

**Option A: CasaOS app store**
- Search for Tailscale and install
- Follow authentication prompts

**Option B: Native installation**
```
bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

**Install on your devices:**
- [Download Tailscale](https://tailscale.com/download) for desktop/mobile
- Sign in with the same account
- Access your server securely from anywhere

## Access points

### Local network
```
CasaOS:         http://SERVER_IP
Immich:         http://SERVER_IP:2283
Crafty:         http://SERVER_IP:8443
Minecraft:      SERVER_IP:25565
SSH:            ssh username@100.x.x.x
```

### Remote (via Tailscale)
```
CasaOS:         http://100.x.x.x
Immich:         http://100.x.x.x:2283
Crafty:         http://100.x.x.x:8443
SSH:            ssh username@100.x.x.x
```

### Public
```
Minecraft:      YOUR_PUBLIC_IP:25565
```

## Security best practices

1. **Keep system updated:**
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

2. **Use strong passwords** for all services

3. **Enable firewall** (if not using CasaOS defaults):
    ```bash
    sudo apt install ufw
    sudo ufw allow 22/tcp       # SSH
    sudo ufw allow 25565/tcp    # Minecraft
    sudo ufw enable
    ```

4. **Regular backups (see next section)

5. **Monitor temperatures** - laptops can overhear under sustained load

## Backup strategy (Recommended)

**Critical data to backup:**
- Immich photo library
- Minecraft world files
- Important configurations

**Options:**
- External USB drive with automated rsync/Duplicati
- Cloud backup (Backblaze B2, Wasabi)
- Remote backup to another location via Tailscale

**Quick backup setup with rsync:**
```bash
# Example: backup to external drive
sudo rsync -av /path/to/data /mnt/backup/
```

## Maintenance