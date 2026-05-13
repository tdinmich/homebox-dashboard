# Codex instructions

This is Tom's homebox landing page.

Environment:
- Ubuntu Server homebox
- Static dashboard served by Caddy
- Dashboard folder: /srv/apps/homebox
- Main file: index.html
- Public local URL: http://192.168.1.20
- Caddy serves this folder using root + file_server
- Access is LAN/Tailscale only, not public internet
- Tailscale IP address: 100.78.92.57

Current dashboard services:
- Uptime Kuma: Service status monitoring.
  - LAN: http://192.168.1.20:3001
  - Tailscale: http://100.78.92.57:3001
- Cockpit: Server admin dashboard.
  - LAN: https://192.168.1.20:9090
  - Tailscale: https://100.78.92.57:9090
- Pi-hole: DNS filtering and ad blocking.
  - LAN: http://192.168.1.20:8081/admin
  - Tailscale: http://100.78.92.57:8081/admin
- Weight Tracker: Personal weight and BMI app.
  - LAN: http://192.168.1.20:8090
  - Tailscale: http://100.78.92.57:8090
- Photo Stream: Private shared photo stream.
  - LAN: http://192.168.1.20:3003
  - Tailscale: http://100.78.92.57:3003
- Reframe Journal: Private guided journal for reframing, gratitude, mood check-ins, typed entries, and handwritten pages.
  - LAN: http://192.168.1.20:3004
  - Tailscale: http://100.78.92.57:3004
- Mealie: Recipe manager and meal planning app.
  - LAN: http://192.168.1.20:3005
  - Tailscale: http://100.78.92.57:3005
- Paperless-ngx: Document archive and search.
  - LAN: http://192.168.1.20:8000
  - Tailscale: http://100.78.92.57:8000
- TomGPT: Local private LLM chat powered by Ollama + Open WebUI.
  - LAN: http://192.168.1.20:3002
  - Tailscale: http://100.78.92.57:3002

Dashboard link scheme:
- Network service cards use a `<div class="card">` wrapper, not a full-card anchor, so each card can contain both LAN and Tailscale links.
- The service title link uses `class="card-title-link"` with `href`, `data-lan-href`, and `data-tailscale-href`.
- Set the title link `href` to the LAN URL as the no-JavaScript fallback.
- The dashboard script changes adaptive title links to Tailscale when the dashboard hostname is `100.78.92.57`; otherwise they stay on LAN.
- Each network service card should also show a port line and a `.link-row` with explicit LAN and Tailscale links.
- Docs/local notes can stay as relative links and do not need LAN/Tailscale variants.

Rules:
- Keep this as a simple static site unless asked otherwise.
- Prefer plain HTML/CSS/JavaScript.
- Do not add build tools unless there is a clear reason.
- Do not edit /etc/caddy or /srv/containers/caddy unless explicitly asked.
- Do not include secrets, PINs, API keys, or private data.
- Keep links local/LAN-friendly.
- Before large edits, summarize the plan.
- After edits, summarize changed files and browser test steps.
