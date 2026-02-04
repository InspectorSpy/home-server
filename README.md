# Old gaming laptop to home-server. A setup journey.

## Hardware
- **Laptop:** Ryzen 7 4800H, NVIDIA GTX 1660 Ti Max-Q
- **Storage:** 512GB NVMe (OS) + 1TB 2.5" SSD (data)
- **RAM:** 16GB
- **Network:** MikroTIk Hex Refresh router

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