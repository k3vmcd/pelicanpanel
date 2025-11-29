# Pelican Panel + Wings + Cloudflare Tunnel + LAN HTTPS

This example merges the full-stack deployment (Panel + Wings + `cloudflared`) with the LAN HTTPS exposure variant. It gives you one `docker compose` project that serves Pelican through Cloudflare Tunnel for remote access, runs Wings on the same host, and simultaneously exposes the Panel over HTTPS to devices on your local network.

## What You Get

- `docker-compose.yml` that launches Pelican Panel, Wings, and `cloudflared` on an isolated bridge network while binding container port 443 to a configurable LAN IP
- `Caddyfile` that terminates TLS internally for LAN visitors and keeps the tunnel-facing listener on plain HTTP for Cloudflare
- `.env.example` documenting Cloudflare, LAN, and Wings variables plus scaffolding folders for Wings (`wings/config`, `wings/data`, `wings/logs`, `wings/tmp`)

## Before You Start

- Create a Cloudflare Tunnel and copy the token from Zero Trust → Networks → Tunnels
- Decide which hostname (for example `panel.yourdomain.com`) you will route through the tunnel and continue to use as `APP_URL`
- Pick the LAN IP that Docker should bind for HTTPS (set `LAN_BIND_ADDRESS`) and make sure no other service occupies the chosen `LAN_HTTPS_PORT`
- Ensure Docker exposes `/var/run/docker.sock` so Wings can manage containers

## Usage

1. Copy this directory outside of the repo, rename `.env.example` to `.env`, and fill in the values (`APP_URL`, `ADMIN_EMAIL`, `CLOUDFLARE_TUNNEL_TOKEN`, `LAN_BIND_ADDRESS`, etc.).
2. Start the stack with `docker compose up -d` and monitor it via `docker compose logs -f panel wings cloudflared`.
3. Complete the Pelican installer at `APP_URL` once migrations finish.
4. Inside the Panel, create a Node that points to `wings` on port `8080` (or the value of `WINGS_HTTP_PORT`) and leave "Communicate over SSL" disabled because the traffic stays on the internal Docker network. Download the generated config, save it to `wings/config/config.yml`, then restart Wings with `docker compose restart wings`.
5. For LAN usage, export and trust Caddy's internal CA: `docker compose cp pelican-panel:/data/caddy/pki/authorities/local/root.crt ./pelican-panel-root.crt`. Import that certificate on devices so the browser accepts the LAN HTTPS session.
6. Configure split DNS/hosts so the `APP_URL` hostname resolves to the LAN IP when you are on-site and to Cloudflare when remote. Pelican must always be accessed via the canonical hostname for sessions and signed URLs to work.

### Ports & Access

- Cloudflare should route your hostname to the service `panel:80` on the Docker network (no host ports exposed).
- LAN visitors reach the Panel through `https://${PANEL_DOMAIN}` which resolves to `LAN_BIND_ADDRESS` and terminates TLS inside the container.
- Wings exposes HTTP (`WINGS_HTTP_PORT`, default `8080`) and SFTP (`WINGS_SFTP_PORT`, default `2022`) on the host. Keep those ports firewalled if you only manage Wings from the Panel.

### Data Persistence

- Panel application data lives in the `pelican-data` and `pelican-logs` volumes.
- Wings data stays inside the `wings/` bind mounts so you can inspect or back up server files directly from the host.
- Removing the stack does not delete these volumes/directories; run `docker compose down -v` and delete the `wings/` folders to wipe everything.

For more background see `docs/panel/advanced/cloudflare-tunnel` and `docs/wings/install` in the main Pelican documentation.
