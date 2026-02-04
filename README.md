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

## Initial setup

### Network configuration
1. **Router setup (MikroTik)**
    - Port 2: Main PC
    - Port 3: Server laptop
    - Enabled ether3 interface
    - Removed ether2 and ether3 from WAN interface list (these were default for some reason)
Journey of setting up a home server on an old laptop

Plans for the server:
be able to connect with ssh
Immich
minecraft server
Tailscale