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
Tenants support Grav websites (older tenants may need an upgrade - to fix that contact the [admin](mailto:antonino.furnari@unict.it), but they need to be configured properly by telling Grav which directory they are installed in.

Grav websites are supported only in subdirs, not in the root directory of the tenant. Please configure the `/user/config/system.yaml` to include the following information:

```yaml
absolute_urls: false
custom_base_url: '/tenant/subdir'

session:
  path: '/tenant/subdir'

home:
  redirect: false

pages:
  redirect_trailing_slash: 0
```

If you want your website to be served from the root directory of the tenant, you may place it in a subdir (e.g., `site`) and place the following index.php in the root directory of the tenant:

```php
<?php
/**
 * Redirects /tentant/ to /tenant/site/
 */
$uri = $_SERVER['REQUEST_URI'];
$path = parse_url($uri, PHP_URL_PATH);

// If the browser hits the bare /tenant/ (which Nginx sees as /)
if ($path == '/' || $path == '/index.php') {
    header('Location: /tenant/site/', true, 301);
    exit;
}

// Otherwise, if the URI points to a physical folder (like /site/ or /site2/),
// let the specific Grav instance handle it.
$segments = explode('/', trim($path, '/'));
$target = $segments[0];

if (!empty($target) && is_dir(__DIR__ . '/' . $target)) {
    require __DIR__ . '/' . $target . '/index.php';
} else {
    // Default fallback if someone types a non-existent path
    header('Location: /tenant/site/', true, 301);
    exit;
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
