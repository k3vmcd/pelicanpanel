# Pelican Panel + Wings + Cloudflare Tunnel (Full Stack)

This example extends the Cloudflare Tunnel setup by adding a Wings node to the same Docker host, which gives you a single `docker compose` stack that can run Pelican Panel, Wings, and Cloudflare Tunnel together.

## What You Get

- `docker-compose.yml` that launches Pelican Panel, Wings, and `cloudflared` on an isolated bridge network
- A `Caddyfile` tuned for the Panel so it can safely sit behind Cloudflare Tunnel without exposing host ports
- Folder scaffolding for Wings (`wings/config`, `wings/data`, `wings/logs`, `wings/tmp`) plus documented environment variables in `.env.example`

## Before You Start

- Create a Cloudflare Tunnel and copy the token from Zero Trust → Networks → Tunnels
- Decide which hostname (for example `panel.yourdomain.com`) should route to the Panel service via the tunnel
- Ensure Docker on the host exposes `/var/run/docker.sock` (Docker Desktop, Docker CE, or compatible)

## Usage

1. Copy this directory outside of the repo, rename `.env.example` to `.env`, and fill in the values (`APP_URL`, `ADMIN_EMAIL`, `CLOUDFLARE_TUNNEL_TOKEN`, etc.).
2. Start the stack with `docker compose up -d` and watch the services with `docker compose logs -f panel wings cloudflared`.
3. Complete the Pelican installer at `APP_URL` once the Panel and database migrations finish.
4. Inside the Panel, create a Node with `wings` as the domain/host value, `8080` as the port (or whatever you set for `WINGS_HTTP_PORT`), and set **Communicate Over SSL** to `No` so the panel speaks plain HTTP on the internal Docker network. After saving, open the Configuration tab and generate the node configuration yaml.
5. Save that yaml as `wings/config/config.yml` and restart the Wings service with `docker compose restart wings`.
6. Add allocations to the node and deploy servers as usual. Server files will live under `wings/data` on the host and Wings will manage Docker containers through the mounted socket.

### Accessing the Services

- Panel traffic flows through Cloudflare Tunnel; configure the tunnel to route your hostname to the internal service `panel:80`.
- Wings exposes its API on `http://localhost:${WINGS_HTTP_PORT}` (defaults to `8080`) and SFTP on `localhost:${WINGS_SFTP_PORT}` (`2022`). You can add additional Cloudflare hostname routes if you need remote access, otherwise keep these host ports firewalled.

### Cleanup & Data Persistence

- Panel state is stored in the `pelican-data` volume and log volume declared in the compose file.
- Wings data (config, server files, logs, tmp files) are simple bind mounts inside the `wings/` subfolders, so backing up or inspecting server files is as easy as copying those directories.
- Removing the stack does **not** delete the volumes or bind-mounted directories; run `docker compose down -v` plus manually delete the `wings/` folders if you want a clean slate.

For more background see the documentation at `docs/panel/advanced/cloudflare-tunnel` and `docs/wings/install`.
