# IPLAB Admin Guide

**Admin Only**: For managing the Ubuntu VM web server (Nginx + Docker sandboxing). Contact: [antonino.furnari@unict.it](mailto:antonino.furnari@unict.it).

## Overview
Single host Nginx reverse proxy/TLS → Docker containers. Data: `/srv` (950GB). Configs: `/opt/lab-services`. Public: `iplab.dmi.unict.it` → `127.0.0.1:<port>` or legacy `192.168.90.55:8201`.[web:16]

## Filesystem
```
/srv/ (data)
├── docker/          # Docker root
├── sftpgo/          # config/data
├── tenants/         # biorg/, furnari/, live/
└── www/             # iplab-web/, html/, etc.

/opt/lab-services/   # docker-compose.yml per service/tenant
```
Docker: `"data-root": "/srv/docker"` in `/etc/docker/daemon.json`.

## Services Layout
Self-contained stacks in `/opt/lab-services/` (e.g., `sftpgo/`, `tenants/furnari/`).

## Nginx Configs
`/etc/nginx/sites-available/iplab` (symlink to enabled):
- `/` → `127.0.0.1:8000` (Hugo site).
- `/live/` → `192.168.90.55:8201`.
- `/furnari/` → `127.0.0.1:8101/`.
- LetsEncrypt certs auto-renew.

## Adding New Static Site/Tenant
1. `sudo mkdir -p /srv/tenants/mlgroup && sudo chmod -R a+rX /srv/tenants/mlgroup`
2. `/opt/lab-services/tenants/mlgroup/docker-compose.yml`:
   ```yaml
   services:
     mlgroup:
       image: nginx:alpine
       container_name: tenant-mlgroup
       restart: unless-stopped
       ports: - "127.0.0.1:8104:80"  # Free port
       volumes: - /srv/tenants/mlgroup:/usr/share/nginx/html:ro
       security_opt: - no-new-privileges:true
   ```
3. `cd /opt/lab-services/tenants/mlgroup && docker compose up -d`
4. Add Nginx `location /mlgroup/ { proxy_pass http://127.0.0.1:8104/; ... }`, `sudo nginx -t && sudo systemctl reload nginx`.
5. Subdomain? New site file + `certbot --nginx -d mlgroup.dmi.unict.it`.

## Custom Services
User provides `docker-compose.yml` → Place in `/opt/lab-services/tenants/<name>/`, `docker compose up -d`, add Nginx route.

**Example Node** (above, adapt ports/volumes).

## SFTPGo Management
1. Create home: `sudo mkdir -p /srv/tenants/<tenant> && sudo chown -R 1000:1000 /srv/tenants/<tenant>`
2. WebAdmin: `http://192.168.90.55:8080/web/admin`
   - Add user/group, set folder `/srv/tenants/<tenant>`, enable SFTP/WebDAV.
   - 2FA, quotas, IP limits.
3. Restart: `cd /opt/lab-services/sftpgo && docker compose restart`.
4. Users access: WebClient `http://192.168.90.55:8080/web/client`, SFTP `:2022`.

## Hugo Site Updates
GitHub Actions rsync `public/` to `/srv/www/iplab-web` → Auto-served.

## Rebooting
```
sudo systemctl reboot  # Docker/Nginx auto-restart
# Post-reboot:
docker ps  # Check services
sudo nginx -t && sudo systemctl status nginx
sudo certbot renew --dry-run
```

## Ops Commands
**Docker**:
- `docker ps` / `docker logs <container>`
- `cd /opt/lab-services/<service> && docker compose up -d / down / restart / logs`

**Nginx**:
- `sudo nginx -t && sudo systemctl reload nginx`
- `sudo certbot renew`

**Monitor**:
- `df -h /srv`
- `du -sh /srv/www/* /srv/tenants/*`
- `docker system df` (images/volumes)

## Permanent Rewrites
Should be placed here: /etc/nginx/snippets/permanent-rewrites.conf

## Security Notes
- Containers: `no-new-privileges:true`, localhost ports.
- Data: `a+rX` (world-readable static files).
- Backups: Rsync `/srv` regularly.

