# Homepage Dashboard Configuration

This repository contains the configuration files for a self-hosted [Homepage](https://gethomepage.dev/) dashboard tailored for a homelab, Security Operations Center (SOC), network administration, and amateur radio/mesh networking environment.

## Repository Structure

| File | Description |
| :--- | :--- |
| `services.yaml` | Core service definitions grouped by category, including site availability monitors (`siteMonitor`) and live service widgets. |
| `widgets.yaml` | Top-level information widgets (system resources, 24-hour date/time, Open-Meteo weather, and Google search). |
| `bookmarks.yaml` | Quick-access links organized into Developer, Social, Entertainment, and Utilities/Icon reference sections. |
| `docker.yaml` | Connection settings for querying Docker container status via a socket proxy (`dockerproxy:2375`). |
| `haproxy.cfg` | HAProxy configuration that exposes `/var/run/docker.sock` over TCP port `2375` for the Docker proxy container. |
| `proxmox.yaml` | Proxmox VE API connection configuration. |
| `settings.yaml` | Global dashboard settings and external provider API configurations. |
| `kubernetes.yaml` | Placeholder for Kubernetes cluster integration. |
| `custom.css` / `custom.js` | Custom styling and scripts for extending the dashboard UI. |
| `.env.example` | Template listing all required and optional `HOMEPAGE_VAR_*` environment variables. |

---

## Dashboard Overview

### 1. Service Categories (`services.yaml`)

Services are organized into functional groups with built-in HTTP/HTTPS health checks (`siteMonitor`):

- **Hobbies (Amateur Radio & Meshtastic)**: Mesh Dash, Meshtastic Client & Flasher, PSK Reporter, TinyGS, MeshMap, Wavelog, and QRZ.
- **AI Productivity**: Google Colab, FutureTools.io, and NotebookLM.
- **Infrastructure as Code**: Ansible AWX.
- **Services**:
  - **Adminer**: PostgreSQL management.
  - **Gatus**: Automated service health dashboard (with live `gatus` widget).
  - **Paperless-ngx**: Document management system (with live `paperlessngx` widget).
  - **Pi-hole (v6)**: DNS and DHCP server (with live `pihole` v6 widget).
  - **Proxmox VE & ProxMenux**: Virtualization management and monitoring.
  - **Portainer**: Container management.
  - **TerraMaster NAS**: Storage appliances (`TNAS F4-210` & `TNAS F2-425`).
- **Server Administration**: Yamtrack and Webmin management panels across Ansible/Dev, Pi-hole, Proxmox, and Docker hosts (using local `.home.arpa` domains).
- **Network Administration**: NetBox network inventory and IPAM.
- **Security Operations Center (SOC)**:
  - **Routers & Switching**: ASUS routers (`RT-AX86 Pro`, `RT-ACRH13`), Cisco core switch, and a MikroTik server firewall (with live `mikrotik` widget).
  - **SIEM, Monitoring & Analysis**: Security Onion, Splunk, LibreNMS, Wazuh, and CyberChef.
- **Smarthome & Media**: Home Assistant (HAOS), Jellyfin, and Jellystat.
- **Home Security**: Wyze Cameras.
- **Learning**: Coursera and LinkedIn Learning.

### 2. Info Widgets (`widgets.yaml`)

- **System Resources**: Tracks CPU usage, CPU temperature, memory, `/` disk usage, and system uptime (refreshes every 3 seconds).
- **Date & Time**: Compact 24-hour (`h23`) medium date and time display.
- **Weather**: Powered by `openmeteo` (no API key required).
- **Search Bar**: Direct Google search integration.

### 3. Docker Socket Proxy (`docker.yaml` & `haproxy.cfg`)

Instead of mounting `/var/run/docker.sock` directly into the Homepage container, this setup routes Docker API traffic through an HAProxy socket proxy:

- `haproxy.cfg` binds to `*:2375` in HTTP mode and forwards requests to `/var/run/docker.sock`.
- `docker.yaml` configures Homepage to connect to `dockerproxy` on port `2375` under the `docker-server` host definition.

---

## Adapting This Configuration for Your Homelab

1. **Clone the repository** into your Homepage config mount directory:
   ```bash
   git clone <your-repo-url> /path/to/homepage/config
   ```

2. **Configure Environment Variables (`.env`)**:
   Copy `.env.example` to `.env` and fill in your API keys, credentials, and location settings (`.env` is excluded by `.gitignore` so secrets stay out of version control):
   ```bash
   cp .env.example .env
   ```
   Ensure your Homepage container loads the `.env` file (for example, by adding `env_file: ./config/.env` in your `docker-compose.yml`). Homepage automatically substitutes any variable prefixed with `HOMEPAGE_VAR_` into the YAML files:
   ```yaml
   # Example in services.yaml
   widget:
     type: paperlessngx
     url: http://10.0.0.12:8000
     key: {{HOMEPAGE_VAR_PAPERLESS_KEY}}
   ```

3. **Update IP Addresses & Hostnames**:
   Replace the internal subnets (`10.0.0.x`, `192.168.1.x`, `192.168.50.x`) and `.home.arpa` hostnames in `services.yaml` and `proxmox.yaml` to match your local network topology.

4. **Icons**:
   Most icons reference the [Dashboard Icons](https://dashboardicons.com/) and [selfh.st/icons](https://selfh.st/icons/) collections (both bookmarked in `bookmarks.yaml` for convenience).

---

## References

- [Official Homepage Documentation](https://gethomepage.dev/)
- [Service Widgets Documentation](https://gethomepage.dev/widgets/)
- [Information Widgets Documentation](https://gethomepage.dev/configs/info-widgets/)
- [Docker Integration Guide](https://gethomepage.dev/configs/docker/)
