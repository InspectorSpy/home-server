# Old gaming laptop to Home Server. A Setup Journey.

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

- **Network:** MikroTik Hex Refresh router (any router that supports port forwarding works)

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

## Prerequisites

- A laptop, almost any will do that has decent specs
- USB drive (8GB+) for OS installation
- Basic familiarity with command line (You will manage without, Google helps)
- Router with port forwarding capability (almost all routers these days support it)
- Static IP or DHCP reservation on your router

## Quick start

### 1. Install Debian/Ubuntu (I used Debian 13)

1. Download [Debian](https://www.debian.org/distrib/netinst)/[Ubuntu](https://ubuntu.com/download/server) (latest stable or testing)
2. Flash to USB using [Rufus](https://rufus.ie/) (Windows) or [Balena Etcher](https://www.balena.io/etcher/). I use [Ventoy](https://www.ventoy.net/en/index.html) you don't have to, go with Rufus or Balena Etcher
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

**For Ubiquiti USW-Flex-Mini users (or similar managed switch):**

```
# Connect switch uplink to your router
# Connect server and other devices to switch ports
# If using UniFi, self-host the Network Application via Docker on your server
# All ports are on the same flat network by default — no VLAN config needed to get started
```

### 4. Install services via CasaOS

Access the CasaOS app store and install:

#### Immich (Photo backup)

- Configure storage path to your data drive
- Install mobile apps ([iOS](https://apps.apple.com/app/immich/id1613945652) / [Android](https://play.google.com/store/apps/details?id=app.alextran.immich))
- Enable auto-upload in mobile app settings

**Note:** The base Immich app didn't work for me, so I used the "Without machine learning" version from the CasaOS app store

#### Crafty Controller (Minecraft management)

- Web-based Minecraft server control panel
- Simplifies server management and mod installation

### 5. Set up minecraft server

In Crafty Controller:

1. **Create new server**
    
    - Type: Fabric (for mod support)
    - Version: Latest stable (1.21.11 for me)
    - RAM: 8 - 10GB for modded, 4 - 6GB for vanilla
2. **Install performance mods** (server-side only):
    
    - [Fabric API](https://modrinth.com/mod/fabric-api) (required)
    - [Lithium](https://modrinth.com/mod/lithium) (optimization)
    - [FerriteCore](https://modrinth.com/mod/ferrite-core) (memory)
    - [C2ME](https://modrinth.com/mod/c2me-fabric) - (chunk performance)
    - [Krypton](https://www.curseforge.com/minecraft/mc-mods/krypton) - (network)
    - [ServerCore](https://www.curseforge.com/minecraft/mc-mods/servercore) - (optimization)
3. **Configure Security** in `server.properties`:
    
    ```properties
    online-mode=true          # Requires legitimate accounts
    white-list=true           # Only whitelisted players
    enable-rcon=false         # Disable remote console
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

```bash
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
SSH:            ssh username@SERVER_IP
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
    sudo ufw allow 22/tcp    # SSH
    sudo ufw allow 25565/tcp # Minecraft
    sudo ufw enable
    ```
    
4. **Regular backups** (see next section)
    
5. **Monitor temperatures** - laptops can overheat under sustained load
    

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

### Regular tasks

- Monitor CPU/RAM/disk usage in CasaOS or Grafana
- Check laptop temperatures (especially under load)
- Update Debian monthly: `sudo apt update && sudo apt upgrade`
- Update Minecraft server and mods periodically
- Check Immich for failed uploads
- Verify backups are working

### Monitoring tools

For basic monitoring, CasaOS has a built-in dashboard. For a proper monitoring stack:

- [Uptime Kuma](https://github.com/louislam/uptime-kuma) - Service uptime monitoring with alerts
- [Prometheus](https://prometheus.io/) + [Grafana](https://grafana.com/) - Full metrics stack with dashboards
- [node_exporter](https://github.com/prometheus/node_exporter) - Host metrics (CPU, RAM, disk, network) for Prometheus
- [cAdvisor](https://github.com/google/cadvisor) - Per-container metrics for Prometheus

See the monitoring server setup doc for the full self-hosted monitoring stack setup across multiple servers.

## Minecraft performance tips

1. **Pre-generate chunks** to reduce lag:
    
    ```
    # In Crafty console
    chunky radius 5000
    chunky start
    ```
    
2. **Adjust view distance** in `server.properties`:
    
    ```properties
    view-distance=10    # Lower = better performance
    simulation-distance=8
    ```
    
3. **Monitor server TPS** (should be 20):
    
    ```
    # In Crafty console
    tps
    ```
    

## Troubleshooting

### Can't connect to Minecraft server from internet

- Verify port forwarding is configured correctly
- Check if ISP blocks port 25565 (some do)
- Test with online port checker: [YouGetSignal](https://www.yougetsignal.com/tools/open-ports/)
- Confirm server is actually running in Crafty
- Try connecting locally first to rule out server issues

### Laptop overheating (applicable to some old PCs)

- Elevate laptop or use cooling pad
- Clean dust from vents
- Reduce Minecraft server RAM allocation
- Limit concurrent services
- Clean and re-apply thermal paste (Advanced tbh)
- Consider undervolting CPU (Advanced as well)

### Out of disk space

- Check disk usage: `df -h`
- Move Immich storage to larger drive
- Set up Immich storage quotas
- Clean up old Docker images: `docker system prune`

### Can't access CasaOS remotely

- Verify Tailscale is running: `sudo tailscale status`
- Check if both devices are connected to Tailscale
- Ensure firewall isn't blocking connections

## Additional Services to consider

- **Jellyfin/Plex** - Media server
- **Vaultwarden** - Self-hosted password manager
- **Pi-hole** - Network-wide ad blocking
- **Nextcloud** - File sync and collaboration
- **Home Assistant** - Home automation
- **Nginx Proxy Manager** - Reverse proxy with automatic HTTPS
- **Forgejo** - Self-hosted Git
- **Stirling-PDF** - Local PDF toolkit, accessible via browser/subdomain
- **Paperless-ngx** - OCR document archive
- **ntfy** - Self-hosted push notifications to phone
- **Glance** - Homelab dashboard (separate from CasaOS, runs as its own container)
- **WatchYourLAN** - LAN IP scanner with Grafana/Prometheus export
- **Healthchecks** - Cron/job monitoring, pairs well with Uptime Kuma

## Obsidian LiveSync (self-hosted vault sync)

A self-hosted alternative to Obsidian Sync, using CouchDB as the backend. Your notes never leave your server.

> **Note:** The Obsidian app in the CasaOS app store requires an HTTPS connection to function, which adds unnecessary complexity. The LiveSync setup is done entirely through the native Obsidian desktop or mobile client on your own devices. Nothing to install on the server beyond CouchDB.

### Prerequisites

- A domain name with DNS managed by Cloudflare
- Nginx Proxy Manager installed (available in CasaOS app store)
- Port 80 and 443 forwarded to your server in your router
- Static IP (or DDNS)

### 1. Install CouchDB

CouchDB isn't in the CasaOS app store, so install it via docker:

```bash
docker run -d \
    --name couchdb \
    -e COUCHDB_USER=admin \
    -e COUCHDB_PASSWORD=yourpassword \
    -p 5984:5984 \
    -v /DATA/AppData/couchdb:/opt/couchdb/data \
    --restart unless-stopped \
    couchdb
```

Verify it's running:

```bash
curl http://localhost:5984
```

Should return a JSON response with CouchDB version info.

### 2. Set up a subdomain

In Cloudflare DNS, add a new A record:

- **Name:** `obsidian` (or whatever you prefer)
- **value:** Your public static IP
- **Proxy status:** DNS only (grey cloud) - NPM handles SSL itself, Cloudflare proxying will interfere. This requires port 443 to be forwarded on your router (see step 5 before testing).

### 3. Configure Nginx Proxy Manager

In NPM, go to **Hosts -> Proxy Hosts -> Add Proxy Host**:

**Details tab:**

- Domain: `obsidian.yourdomain.com`
- Scheme: `http`
- Forward Hostname: your server's LAN IP
- Forward Port: `5984`
- Enable **Websockets Support**

**SSL tab:**

- Request a new Let's Encrypt certificate
- Enable **Use DNS Challenge**
- Provider: Cloudflare
- Paste your Cloudflare API token (see below)
- Enable **Force SSL**

**Getting Cloudflare API token:**

1. Go to Cloudflare -> My Profile -> API Tokens -> Create Token
2. Use the **Edit zone DNS** template
3. Under Zone Resources, select your specific domain zone
4. Create and copy the token

### 4. Configure CouchDB CORS

Access CouchDB admin at `http://YOUR_SERVER_IP:5984/_utils` and log in.

Go to **Configuration -> CORS** and:

- Set Origin to **All domains (*)**
- Enable CORS

### 5. Forward ports on your router

Add dst-nat rules for ports 80 and 443 pointing to your server's LAN IP. Make sure to set the In. Interface to your WAN interface.

### 6. Install and configure LiveSync in Obsidian

1. Install the **Self-hosted LiveSync** community plugin
2. Go to plugin settings and choose **Manual Setup**
3. Enter:
    - **URL:** `https://obsidian.yourdomain.com`
    - **Username:** your CouchDB admin username
    - **Password:** your CouchDB admin password
    - **Database name:** `obsidian` (or any lowercase name, LiveSync creates it automagically)
4. Click **Detect and Fix CouchDB Issues** to auto-configure remaining settings
5. Click **Test settings and Continue**

### 7. Verify it's working

Check CouchDB at `http://YOUR_SERVER_IP:5984` - you should see an `obsidian` database. Click into it and you'll see documents (with hashed IDs) representing your vault files.

### Setting up additional devices

On your first device, open the Obsidian command palette and run:

> **"Copy settings as a new Setup URI"**

On each additional device, install the LiveSync plugin and paste the Setup URI when prompted - much faster than manual setup.

### Troubleshooting (I did some, a lot.)

**"Failed to connect to server" in LiveSync:**

- Check that port 443 is forwarded correctly in your router
- Ensure the Cloudflare proxy is disabled (DNS only)
- verify NPM proxy host is using `http` scheme to forward to CouchDB

**CORS errors in Obsidian:**

- Go back to CouchDB `/_utils` and confirm CORS is set to All domains (*)

**502 Bad Gateway from NPM:**

- Check CouchDB is running: `docker ps | grep couchdb`
- Verify the forward hostname and port in NPM are correct

**Works on mobile data but not on home WiFi (Hairpin NAT):**

This is a decently common issue with home routers. When you try to reach your external domain from inside your own network, the router doesn't route the traffic back internally. Fix it by adding a static DNS entry in MikroTik:

1. Go to **IP -> DNS -> Static -> New**
2. Set **Name** to `obsidian.yourdomain.com`
3. Set **Type** to `A`
4. Set **Address** to your server's LAN IP
5. Leave Address list empty
6. Click OK

This makes local devices resolve the domain directly to your server's LAN IP instead of going out through the internet.

### Access points

```
CouchDB admin:  http://SERVER_IP:5984/_utils
LiveSync sync:  https://obsidian.yourdomain.com
```

## Bot hosting

A lightweight Docker-based environment for running Telegram (and maybe eventually Discord) bots persistently on the server.

### Structure

```
/opt/bots/
    kaliabot/
        data/      <- SQLite database (persisted via bind mount)
        .env       <- Bot token and config (never commit this)
        compose.yaml
```

### KaliaBot

A Telegram bot that tracks how many beers a group has consumed. Forked from [OliverK03/KaliaBot](https://github.com/OliverK03/KaliaBot) for self-hosting purposes.

**Setup:**

1. Create the directory and clone:

```bash
sudo mkdir -p /opt/bots/kaliabot/data
sudo chown -R $USER:$USER /opt/bots
cd /opt/bots/kaliabot
git clone https://github.com/InspectorSpy/KaliaBot.git .
```

2. Configure environment:

```bash
cp .env.example .env
nano .env # Add BOT_TOKEN, set REPORT_TIMEZONE=UTC (or any valid IANA timezone e.g. Europe/Helsinki)
```

3. Fix data directory permissions (container runs as UID 10001):

```bash
sudo chown -R 10001:10001 data/
```

4. Start:

```bash
docker compose up -d --build
docker compose logs -f
```

**`compose.yaml`:**

```yaml
services:
  server:
    build:
      context: .
    env_file:
      - .env
    volumes:
      - ./data:/app/data
    restart: unless-stopped
```

No ports need to be exposed - the bot uses polling, so no public endpoint or NPM proxy host is required.

### Adding more bots

Each bot gets its own subdirectory under `/opt/bots/` with the same structure. Remember to `chown` the `data/` directory to match UID of the container's user if it runs as non-root.

### Access points

```
KaliaBot logs: docker compose -f /opt/bots/kaliabot/compose.yaml logs -f
```

## License

This "guide" is provided as-is under the MIT License. Use at your own risk.

## Acknowledgements

- [CasaOS](https://casaos.io/) - Easy home server management
- [Immich](https://immich.app/) - Self-hosted photo backup alternative to Google Photos or Apple iCloud
- [Crafty Controller](https://craftycontroller.com/) - Minecraft server management
- [Tailscale](https://tailscale.com/) - Secure networking made simple
- [Fabric](https://fabricmc.net/) - Lightweight Minecraft modding platform
- [Sure Finance](https://github.com/we-promise/sure) - Self-hosted personal finance and wealth management
- [Enable Banking](https://enablebanking.com/) - PSD2-compliant European bank data access
- [Obsidian](https://obsidian.md/) - Knowledge base and note-taking app
- [Self-hosted LiveSync](https://github.com/vrtmrz/obsidian-livesync) - Self-hosted Obsidian vault sync plugin
- [CouchDB](https://couchdb.apache.org/) - Database backend for LiveSync
- [Nginx Proxy Manager](https://nginxproxymanager.com/) - Reverse proxy with automatic HTTPS
- [Prometheus](https://prometheus.io/) - Metrics collection and storage
- [Grafana](https://grafana.com/) - Metrics visualization and dashboards
- [Uptime Kuma](https://github.com/louislam/uptime-kuma) - Service uptime monitoring
- [node_exporter](https://github.com/prometheus/node_exporter) - Host metrics exporter for Prometheus
- [cAdvisor](https://github.com/google/cadvisor) - Container metrics exporter for Prometheus

## Tech stack

![Debian](https://img.shields.io/badge/Debian-D70A53?style=for-the-badge&logo=debian&logoColor=white) ![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white) ![Minecraft](https://img.shields.io/badge/Minecraft-62B47A?style=for-the-badge&logo=minecraft&logoColor=white) ![CouchDB](https://img.shields.io/badge/CouchDB-E42528?style=for-the-badge&logo=apachecouchdb&logoColor=white) ![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white) ![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white) ![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)

**Happy self-hosting, go nuts!**
