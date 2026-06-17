# Homebox Update README

This file is a compact handoff for Tom's ChatGPT Project context. It summarizes the current Homebox dashboard conventions, local service URLs, and the most recent arcade/game updates.

## Environment

- Server name: homebox
- OS: Ubuntu Server
- Main dashboard folder: `/srv/apps/homebox`
- Dashboard file: `/srv/apps/homebox/index.html`
- Dashboard URL on LAN: `http://192.168.1.20`
- Dashboard URL on Tailscale: `http://100.78.92.57`
- Access model: private LAN/Tailscale only
- Do not open public router ports
- Static dashboard is served by Caddy

## Dashboard Editing Rules

- Edit only `/srv/apps/homebox/index.html` for dashboard webpage updates unless Tom asks otherwise.
- Do not edit `/etc/caddy` or `/srv/containers/caddy` unless explicitly asked.
- Dashboard service cards use a `<div class="card">` wrapper, not a full-card anchor.
- Title links use `class="card-title-link"` with:
  - `href` set to the LAN URL
  - `data-lan-href`
  - `data-tailscale-href`
- The dashboard JavaScript switches adaptive title links to Tailscale when the dashboard hostname is `100.78.92.57`.
- Each network service card should include:
  - service title
  - short description
  - port line
  - explicit LAN and Tailscale links

## Current Dashboard Services

### Monitoring

- Uptime Kuma
  - LAN: `http://192.168.1.20:3001`
  - Tailscale: `http://100.78.92.57:3001`

### Server Admin

- Cockpit
  - LAN: `https://192.168.1.20:9090`
  - Tailscale: `https://100.78.92.57:9090`
- Pi-hole
  - LAN: `http://192.168.1.20:8081/admin`
  - Tailscale: `http://100.78.92.57:8081/admin`

### Apps

- Weight Tracker
  - LAN: `http://192.168.1.20:8090`
  - Tailscale: `http://100.78.92.57:8090`
- Photo Stream
  - LAN: `http://192.168.1.20:3003`
  - Tailscale: `http://100.78.92.57:3003`
- Reframe Journal
  - LAN: `http://192.168.1.20:3004`
  - Tailscale: `http://100.78.92.57:3004`
- Mealie
  - LAN: `http://192.168.1.20:3005`
  - Tailscale: `http://100.78.92.57:3005`
- Recipe Forge
  - LAN: `http://192.168.1.20:3006`
  - Tailscale: `http://100.78.92.57:3006`
- Paperless-ngx
  - LAN: `http://192.168.1.20:8000`
  - Tailscale: `http://100.78.92.57:8000`
- TomGPT
  - LAN: `http://192.168.1.20:3002`
  - Tailscale: `http://100.78.92.57:3002`
- Circle Run
  - Container folder: `/srv/containers/homebox-arcade`
  - LAN: `http://192.168.1.20:3008`
  - Tailscale: `http://100.78.92.57:3008`
- Circle Run Lab
  - Container folder: `/srv/containers/circle-run-lab`
  - LAN: `http://192.168.1.20:3009`
  - Tailscale: `http://100.78.92.57:3009`
- Bumper Dots
  - Container folder: `/srv/containers/bumper-dots`
  - LAN: `http://192.168.1.20:3010`
  - Tailscale: `http://100.78.92.57:3010`
- Volley Clash
  - Container folder: `/srv/containers/arcade-tennis`
  - LAN: `http://192.168.1.20:3011`
  - Tailscale: `http://100.78.92.57:3011`

## Recent Updates

### Dashboard

- Added dashboard cards for:
  - Circle Run on port `3008`
  - Circle Run Lab on port `3009`
  - Bumper Dots on port `3010`
  - Volley Clash on port `3011`
- Updated dashboard Apps count from `7 apps` to `11 apps`.
- Confirmed there is no separate `/srv/containers/circle-run` folder visible; Circle Run is the existing `/srv/containers/homebox-arcade` container on port `3008`.

### Volley Clash

- New app folder: `/srv/containers/arcade-tennis`
- Game name: Volley Clash
- Host port: `3011`
- Docker service/container: `volley-clash`
- Runtime access is private LAN/Tailscale only.
- Tech stack:
  - Node.js
  - Express
  - Socket.IO
  - plain HTML/CSS/JavaScript
  - Canvas rendering
  - Docker Compose
- Features implemented:
  - room codes
  - lobby
  - server-authoritative real-time game loop
  - Singles, Doubles, Boss Court, Chaos Tennis
  - AI bot fill
  - mobile joystick and swing controls
  - desktop controls
  - generated audio effects
  - haptics where supported
  - power-ups
  - simple JSON leaderboard under `./data`
- Recent tuning:
  - Player movement speed was increased because the first build felt sluggish.

## Useful Commands

### Dashboard

The dashboard is static. After editing `/srv/apps/homebox/index.html`, usually just refresh:

```bash
curl -I http://192.168.1.20
```

### Volley Clash

```bash
cd /srv/containers/arcade-tennis
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
```

### Bumper Dots

```bash
cd /srv/containers/bumper-dots
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
```

### Circle Run

```bash
cd /srv/containers/homebox-arcade
sudo docker compose up -d --build
sudo docker compose ps
```

### Circle Run Lab

```bash
cd /srv/containers/circle-run-lab
sudo docker compose up -d --build
sudo docker compose ps
```

## Privacy And Git Notes

- Keep Homebox LAN/Tailscale only.
- Do not commit secrets, `.env` files, runtime databases, private images, generated logs, or `data/`.
- Do not use `chmod 777`.
- Treat `/srv/containers/*/data` as runtime persistence.
- Private family images or avatars should stay under ignored paths such as `assets/private/`.

## Good Future ChatGPT Context

When asking ChatGPT/Codex for Homebox work, include:

- The target folder.
- Whether the request is only for the dashboard or also for app containers.
- The desired LAN/Tailscale URLs and ports.
- Whether Docker Compose should be changed.
- Whether runtime data may be touched.

Default preference: make focused edits, keep services private, avoid public exposure, and preserve existing containers unless Tom explicitly asks to change them.
