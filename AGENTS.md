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
- TomGPT: Local private LLM chat powered by Ollama + Open WebUI.
  - LAN: http://192.168.1.20:3002
  - Tailscale: http://100.78.92.57:3002

Rules:
- Keep this as a simple static site unless asked otherwise.
- Prefer plain HTML/CSS/JavaScript.
- Do not add build tools unless there is a clear reason.
- Do not edit /etc/caddy or /srv/containers/caddy unless explicitly asked.
- Do not include secrets, PINs, API keys, or private data.
- Keep links local/LAN-friendly.
- Before large edits, summarize the plan.
- After edits, summarize changed files and browser test steps.
