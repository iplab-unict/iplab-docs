# User Guide

## Overview
The IPLAB server runs multiple services in Dockerized environments for sandboxing—each isolated for security (hacked service can't escape container). Common services are static/dynamic web hosting, sftp access, custom services through docker containers.

## IPLab Website
The IPLAB website has been re-designed to be static for security. It can be edited through pushes to a GitHub repo. Edit access can be asked to [admin](mailto:antonino.furnari@unict.it).

## Tenants
Services and shared websites are organized in tenants. Tenants can correspond to groups/projects/members. Each tenant is generally associated with at least one service (e.g., a web server).

Served websites can be managed through SFTP access (more on this later) and are automatically served at the https://iplab.dmi.unict.it/tenant/[...] URL.

The activation of a tenant can be asked to the [admin](mailto:antonino.furnari@unict.it).

### Grav Websites
By default, tenants do not support Grav websites. To enable Grav support or add a new tenant with Grav support, please contact the [admin](mailto:antonino.furnari@unict.it).

Once the tenant is enabled, please add your website in the root directory of your tenant and configure it properly by telling Grav which directory it is installed in. This is done by adding the following lines to the `/user/config/system.yaml`:

```yaml
absolute_urls: false
custom_base_url: '/tenant'

session:
  path: '/tenant'

home:
  redirect: false

pages:
  redirect_trailing_slash: 0
```

If you want to add additional grav websites in subdirectories of your tenant, you can do so and configure them properly by telling Grav which directory it is installed in. This is done by adding the following lines to the `/user/config/system.yaml`:


```yaml
absolute_urls: false
custom_base_url: '/tenant/subsite'

session:
  path: '/tenant/subsite'

home:
  redirect: false

pages:
  redirect_trailing_slash: 0
```

After this is done, please ask the [admin](mailto:antonino.furnari@unict.it) to configure the tenant for the new subsite by adding the following block in the corresponding nginx configuration file:

```
location /subsite/ {
    # Try literal file, then folder, then fall back to the SUBSITE'S index.php
    try_files $uri $uri/ /subsite/index.php?$query_string;
}
```

### Email Service
Mailing is disabled within the tenants for security reasons. However, a service to send emails is available at port 4321 on the internal IP. An account can be asked to the [admin](mailto:antonino.furnari@unict.it).

## Legacy Web Service
Besides the standard web service, IPLAB provides a **legacy web service** for archiving static websites and basic PHP applications that no longer require updates. For enhanced security, the filesystem is mounted **read-only** (`:ro`) in the Nginx container, preventing unauthorized modifications while serving content efficiently. These sites are accessible at `iplab.dmi.unict.it/legacy/...`, with **permanent 301 redirects** from their original root URLs (e.g., `/oldproject` → `/legacy/oldproject/`)—add directories to `/srv/tenants/legacy/` and run the auto-script for instant mapping.

Many websites deemed to be archivable have been migrated to legacy. Please ask the [admin](mailto:antonino.furnari@unict.it) if you prefer these to be moved to
an other tenant for active updates.

## Custom Services
Custom services (besides basic static or php serving) can be activated asking the [admin](mailto:antonino.furnari@unict.it) and providing a custom `docker-compose.yml` file for the service.

**Example**:
```yaml
version: '3'
services:
  app:
    image: node:18
    volumes: - /data/tenant1/app:/app
    ports: - "3000:3000"
    command: node server.js
```

## IP Addresses
The server has two network adapters and two IP addresses:
- **Public (`iplab.dmi.unict.it` or `151.97.252.83`)**: for external services.
- **Internal (`192.168.90.55`)**: to access docs and for SFTP. You need to be on the local network or on VPN (with access to VLAN 90).

Internal services (e.g., NFS exports) must be exposed / targeted to the internal IP address.

## File Management
Users can access files through an SFTPGo server.

### WebClient (Port 8080)
The web client for file management can be accessed through the internal IP: `http://192.168.90.55:8080/`

### SFTP (Port 2022)
The same files can be accessed through SFTP (e.g., with FileZilla) on IP `192.168.90.55` and port `2022`.

### Credentials
Credentials are shared across the web client and the can be activated asking the [admin](mailto:antonino.furnari@unict.it).
