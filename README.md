# Homepage Dashboard Configuration

This repository contains the configuration files for a self-hosted [Homepage](https://gethomepage.dev/) dashboard tailored for a homelab, Security Operations Center (SOC), network administration, and amateur radio/mesh networking environment.

## Repository Structure

| File | Description |
| :--- | :--- |
| `compose.yml` | Docker Compose definition for the `homepage` container (mounts `./config` and `./icons`, and loads `.env` via `env_file`). |
| `.env.example` | Template listing all required and optional `HOMEPAGE_VAR_*` environment variables. |
| `config/services.yaml` | Core service definitions grouped by category, including site availability monitors (`siteMonitor`) and live service widgets. |
| `config/widgets.yaml` | Top-level information widgets (system resources, 24-hour date/time, Open-Meteo weather, and Google search). |
| `config/bookmarks.yaml` | Quick-access links organized into Developer, Social, Entertainment, and Utilities/Icon reference sections. |
| `config/docker.yaml` | Connection settings for querying Docker container status via a socket proxy (`dockerproxy:2375`). |
| `config/haproxy.cfg` | HAProxy configuration that exposes `/var/run/docker.sock` over TCP port `2375` for the Docker proxy container. |
| `config/proxmox.yaml` | Proxmox VE API connection configuration. |
| `config/settings.yaml` | Global dashboard settings and external provider API configurations. |
| `config/kubernetes.yaml` | Placeholder for Kubernetes cluster integration. |
| `config/custom.css` / `config/custom.js` | Custom styling and scripts for extending the dashboard UI. |

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

1. **Clone the repository**:
   ```bash
   git clone https://github.com/cyberthreatgurl/homepage.git
   cd homepage
   ```

2. **Configure Environment Variables (`.env`)**:
   Copy `.env.example` to `.env` and fill in your API keys, credentials, and location settings (`.env` is excluded by `.gitignore` so secrets stay out of version control):
   ```bash
   cp .env.example .env
   ```
   `compose.yml` includes `env_file: - .env` so all `HOMEPAGE_VAR_*` variables are injected into the running Homepage container (Docker Compose does not pass `.env` variables into the container environment without `env_file:`). Homepage automatically substitutes any variable prefixed with `HOMEPAGE_VAR_` into the YAML files in `config/`:
   ```yaml
   # Example in config/services.yaml
   widget:
     type: paperlessngx
     url: http://10.0.0.12:8000
     key: {{HOMEPAGE_VAR_PAPERLESS_KEY}}
   ```
   > **Note on Pi-hole v6 Authentication:**
   > In `config/services.yaml`, the Pi-hole v6 widget must use `version: 6` and pass the credential under the property name `key:` (wrapped in quotes: `key: "{{HOMEPAGE_VAR_PIHOLE_KEY}}"`). Set `HOMEPAGE_VAR_PIHOLE_KEY` in `.env` to your **plain-text** Pi-hole v6 App Password or web admin password (do **not** use the `$BALLOON-SHA256$...` hash from `pihole.toml`). Homepage automatically sends this plain-text password to Pi-hole's `/api/auth` endpoint to generate a temporary session ID (`sid`).

3. **Update IP Addresses, Hostnames & Allowed Hosts**:
   - Replace the internal subnets (`10.0.0.x`, `192.168.1.x`, `192.168.50.x`) and `.home.arpa` hostnames in `config/services.yaml` and `config/proxmox.yaml` to match your local network topology.
   - Update `HOMEPAGE_ALLOWED_HOSTS` in `compose.yml` to match the IP/hostname and port used to access the dashboard.

4. **Start or Recreate the Container**:
   Whenever you update `.env` or `compose.yml`, recreate the container so updated environment variables are loaded:
   ```bash
   docker compose up -d --force-recreate
   ```

5. **Icons**:
   Custom icons are stored in `./icons` (mounted to `/app/public/icons`), and built-in icons reference the [Dashboard Icons](https://dashboardicons.com/) and [selfh.st/icons](https://selfh.st/icons/) collections (both bookmarked in `config/bookmarks.yaml` for convenience).

---

## References

- [Official Homepage Documentation](https://gethomepage.dev/)
- [Service Widgets Documentation](https://gethomepage.dev/widgets/)
- [Information Widgets Documentation](https://gethomepage.dev/configs/info-widgets/)
- [Docker Integration Guide](https://gethomepage.dev/configs/docker/)
