# Homebox Master Project Handoff

Last updated: 2026-06-15  
Owner/user: Tom  
Scope: Homebox Ubuntu server, self-hosted apps, local/GPU LLM stack, Codex workflow, and family browser games.

This file is meant to replace the older scattered Homebox/Recipe Forge/status handoff files in this ChatGPT Project. Keep this as the single living context file for ChatGPT and Codex.

## Table of contents

- [How to use this file](#how-to-use-this-file)
- [Top priorities and mental model](#top-priorities-and-mental-model)
- [Privacy, safety, and security rules](#privacy-safety-and-security-rules)
- [Machine and network identity](#machine-and-network-identity)
- [Current service and port map](#current-service-and-port-map)
- [Verification commands](#verification-commands)
- [Storage and reliability](#storage-and-reliability)
- [Dashboard and Caddy](#dashboard-and-caddy)
- [Pi-hole](#pi-hole)
- [Cockpit](#cockpit)
- [Uptime Kuma](#uptime-kuma)
- [Local LLM stack: Ollama + Open WebUI / TomGPT](#local-llm-stack-ollama-open-webui-tomgpt)
- [LibreChat](#librechat)
- [Windows GPU worker / remote Ollama](#windows-gpu-worker-remote-ollama)
- [Paperless-ngx](#paperless-ngx)
- [Mealie and Tandoor](#mealie-and-tandoor)
- [Recipe Forge](#recipe-forge)
- [Photo Stream](#photo-stream)
- [Reframe Journal](#reframe-journal)
- [Weight Tracker](#weight-tracker)
- [Circle Run / Homebox Arcade](#circle-run-homebox-arcade)
- [Helena's Run / Circle Run Lab](#helenas-run-circle-run-lab)
- [Bumper Dots](#bumper-dots)
- [Thomas Bear's Volley Clash](#thomas-bears-volley-clash)
- [Codex / GitHub workflow](#codex-github-workflow)
- [Backups and data protection](#backups-and-data-protection)
- [Common deployment patterns](#common-deployment-patterns)
- [Decision log](#decision-log)
- [Current gaps / next useful improvements](#current-gaps-next-useful-improvements)
- [Source files consolidated into this master](#source-files-consolidated-into-this-master)

## How to use this file

- Upload this file to the ChatGPT Project as the primary project context.
- When Homebox changes, update this file instead of creating another one-off status note.
- If this file conflicts with an older file, trust this file unless a live command on Homebox proves otherwise.
- This file is a consolidated project handoff, not a live scan of the server. For current runtime truth, verify with the commands in the verification sections.
- Do not add secrets, PINs, `.env` contents, database contents, uploaded files, private photos, raw recipe text, model logs containing private data, or journal content.

## Top priorities and mental model

Homebox is intended to be a boring, dependable private home server first, and a playground second.

Core philosophy:

1. Run Ubuntu Server on the Minisforum UM880 Plus as a single clean host.
2. Put services in Docker Compose under `/srv/containers/<service>`.
3. Put simple static/Caddy-served apps under `/srv/apps/<site>`.
4. Keep everything private to LAN/Tailscale unless there is a deliberate, secured public-exposure plan.
5. Prefer stable, understandable setups over clever homelab complexity.
6. Keep data under visible folders, usually `./data`, and keep runtime data out of GitHub.
7. Use Codex for code edits, but Tom should decide when to rebuild, test, commit, and push.

Important historical decision:

- The original plan intentionally chose Ubuntu Server + OpenSSH + Docker + Tailscale + optional Cockpit instead of Proxmox, Windows/WSL2, dual boot, or Ubuntu Desktop. The reason was long-term reliability, fewer layers, easier recovery after outages, and a cleaner mental model.
- Proxmox/Home Assistant OS/desktop Linux can be revisited later, but the base system should stay simple unless there is a strong reason to change.

## Privacy, safety, and security rules

Never commit or upload:

- `.env` files or secret keys
- PINs/passwords/API keys/JWT secrets
- SQLite/Postgres/MongoDB/MeiliSearch data
- uploaded photos, documents, recipes, images, or PDFs
- extracted/OCR text, generated model outputs, raw HTML captures
- journal entries, drawings, stroke JSON
- private family avatar images
- model logs that may include private content

Network rules:

- Keep services private to LAN/Tailscale.
- Do not open router ports for Homebox apps, Ollama, Open WebUI, LibreChat, Recipe Forge, or the Windows GPU worker unless explicitly planned and hardened.
- Use Tailscale for remote access.
- Ollama raw API on Homebox should remain bound to localhost unless there is a deliberate reason.
- The Windows GPU Ollama endpoint is LAN/private only.

Git rules:

- Do not commit `data/`, `.env`, databases, runtime uploads, private assets, or generated outputs.
- Prefer simple `.gitignore` patterns that catch private data before the first commit.
- Commit only after the app works in Docker and has been tested in the browser/phone when relevant.

## Machine and network identity

Homebox:

- Hostname/nickname: `homebox`
- Hardware: Minisforum UM880 Plus
- OS: Ubuntu Server
- Main LAN IP: `192.168.1.20`
- Tailscale IP: `100.78.92.57`
- Time zone: `America/New_York`
- SSH pattern:

```bash
ssh tom@192.168.1.20
ssh tom@homebox
```

Folder conventions:

```text
/srv/containers/<service>     Docker Compose services and app stacks
/srv/apps/<site>              static sites served by Caddy
/srv/docs                     optional local notes/documentation
```

Remote access:

- SSH works.
- VS Code Remote SSH works.
- Tailscale is available for access away from home.
- Cockpit is available for a browser-based server dashboard.
- VS Code integrated terminal in a Remote SSH session runs on Homebox, not on the Windows laptop.

## Current service and port map

Verify live state before relying on this table for changes. Some ports come from docs/status files rather than a fresh live scan.

| Service | Port / URL | Folder / notes |
|---|---:|---|
| SSH | `22` | Remote shell access. |
| Caddy / Homebox dashboard | `80` / `http://192.168.1.20` | Static dashboard at `/srv/apps/homebox/index.html`; Caddy stack in `/srv/containers/caddy`. |
| Pi-hole DNS | `192.168.1.20:53` TCP/UDP | DNS filtering for house network. |
| Pi-hole admin | `8081` / `http://192.168.1.20:8081/admin` | `/srv/containers/pihole`. |
| Cockpit | `9090` / `https://192.168.1.20:9090` | Self-signed certificate warning is expected. |
| Uptime Kuma | `3001` / `http://192.168.1.20:3001` | `/srv/containers/uptime-kuma`. |
| Open WebUI / TomGPT | `3002` / `http://192.168.1.20:3002` | `/srv/containers/llm`; local chat/model playground. |
| Photo Stream | `3003` / `http://192.168.1.20:3003` | `/srv/containers/photo-stream`; private photo sharing for Tom and Pharin. |
| Reframe Journal | `3004` / `http://192.168.1.20:3004` | `/srv/containers/reframe-journal`; private guided journal. |
| Mealie | `3005` / `http://192.168.1.20:3005` | `/srv/containers/mealie`; recipe manager being evaluated. |
| Recipe Forge | `3006` / `http://192.168.1.20:3006` | `/srv/containers/recipe-forge`; primary custom recipe/cocktail app. |
| Tandoor Recipes | `3007` / `http://192.168.1.20:3007` | `/srv/containers/tandoor`; recipe alternative being evaluated. |
| Circle Run / Homebox Arcade | `3008` / `http://192.168.1.20:3008` | `/srv/containers/homebox-arcade`; stable/family arcade game. |
| Helena's Run / Circle Run Lab | `3009` / `http://192.168.1.20:3009` | `/srv/containers/circle-run-lab`; experimental/lab maze game. |
| Bumper Dots | `3010` / `http://192.168.1.20:3010` | `/srv/containers/bumper-dots`; multiplayer phone arcade game. |
| Thomas Bear's Volley Clash | `3011` / `http://192.168.1.20:3011` | `/srv/containers/arcade-tennis`; multiplayer family volleyball/tennis game. |
| LibreChat | `3012` / `http://192.168.1.20:3012` | `/srv/containers/librechat`; newer ChatGPT-like frontend under testing. |
| Paperless-ngx | `8000` / `http://192.168.1.20:8000` | `/srv/containers/paperless-ngx`; document archive/search. |
| Weight Tracker | `8090` / `http://192.168.1.20:8090` | `/srv/containers/weight-tracker`; verify current port if needed. |
| Ollama raw API, Homebox | `127.0.0.1:11434` | In `/srv/containers/llm`; should generally stay localhost-bound. |
| Ollama raw API, Windows GPU worker | `192.168.1.35:11434` | Son's Windows gaming PC; private LAN endpoint, reachable when awake. |

Tailscale URLs usually use the same port with host `100.78.92.57`, for example `http://100.78.92.57:3006` for Recipe Forge.

## Verification commands

Show running containers and published ports:

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

Show listening host ports:

```bash
sudo ss -tulpn
```

Check a specific Compose app:

```bash
cd /srv/containers/<service>
sudo docker compose ps
sudo docker compose logs --tail=80
```

Search compose files for ports:

```bash
grep -R "3001\|3002\|3003\|3004\|3005\|3006\|3007\|3008\|3009\|3010\|3011\|3012\|8000\|8081\|8090" -n /srv/containers/*/compose*.y*ml
```

## Storage and reliability

Storage:

- Root filesystem was expanded from the original smaller logical volume to use the internal SSD allocation.
- Approximate post-resize root filesystem: about `914G` total mounted at `/`.
- `/srv` is part of the main root filesystem.
- Docker apps, Caddy, app data, uploads, and runtime data share this expanded root filesystem.

Useful storage checks:

```bash
df -hT /
findmnt -no SOURCE,FSTYPE /
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINTS
sudo vgs
sudo lvs
sudo du -sh /srv/containers /srv/apps
sudo docker system df
```

AC power recovery:

- Tom confirmed the Minisforum BIOS/UEFI setting for automatic power-on after power loss.
- The working BIOS path/label was similar to:

```text
Advanced -> AMD CBS -> FCH Common Options -> AC Power Loss Options -> AC Loss Control
```

Set AC Loss Control / Restore AC Power Loss / Power On After AC Loss to:

```text
Always On
```

or equivalent:

```text
Power On
```

Avoid `Always Off` / `Stay Off`. Avoid `Previous State` / `Last State` unless deliberately preferred.

Adjacent BIOS settings seen:

- Wake Up On Lan: Enabled
- Wake Up by RTC: Disabled
- PowerLimit Setting: Balance Mode

Wake-on-LAN is useful but is not the same as automatic power-on after an outage.

After an outage/reboot, check:

```bash
ssh tom@192.168.1.20
uptime
sudo docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
sudo systemctl status ssh
sudo systemctl status docker
```

Clean reboot/poweroff:

```bash
sudo reboot
sudo poweroff
```

Operational recommendations:

- Keep boot drive unencrypted if unattended recovery is the priority.
- Keep BIOS AC recovery on Always On / Power On.
- Use Docker restart policies where appropriate.
- Consider a small UPS for brief outages and cleaner shutdowns.
- If Homebox does not appear reachable after an outage, check power, BIOS setting, disk encryption/boot prompts, network/Tailscale startup, Docker restart policies, and Pi-hole/DNS effects.

## Dashboard and Caddy

Purpose:

- Local front door to Homebox services.
- Static dashboard served by Caddy on port 80.

Important paths:

```text
/srv/apps/homebox/index.html
/srv/containers/caddy
/srv/containers/caddy/conf/Caddyfile
```

Current Caddy behavior:

```caddyfile
:80 {
    root * /srv/apps/homebox
    file_server
}
```

Editing dashboard links:

- Usually edit only `/srv/apps/homebox/index.html`.
- Browser refresh is usually enough after saving dashboard HTML.
- Changing the Caddyfile requires validation/reload.

Useful commands:

```bash
cd /srv/containers/caddy
sudo docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile
sudo docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile
sudo docker compose logs --tail=80 caddy
```

Dashboard should include both LAN and Tailscale links for each service.

## Pi-hole

Purpose:

- DNS-level filtering for the home network.

Paths and ports:

```text
/srv/containers/pihole
DNS: 192.168.1.20:53 tcp/udp
Admin: http://192.168.1.20:8081/admin
```

Known state:

- Orbi/router has been configured so the house uses Pi-hole for DNS.
- iPhone manual DNS test worked before enabling Pi-hole network-wide.
- iCloud Private Relay warning appeared on iPhone.
- Decision: do not whitelist Apple Private Relay domains just to remove the warning. Keep `mask.icloud.com` and `mask-h2.icloud.com` blocked unless a specific Apple/iCloud feature breaks.
- Do not put public DNS such as `8.8.8.8` or `1.1.1.1` as secondary/fallback DNS in Orbi if the goal is to force Pi-hole filtering; clients may bypass Pi-hole. Public DNS is fine as Pi-hole upstream.

Useful commands:

```bash
cd /srv/containers/pihole
sudo docker compose ps
sudo docker compose up -d
sudo docker compose down
sudo docker compose up -d --force-recreate
sudo docker compose logs --tail=80
```

DNS recovery if the house breaks:

1. Set Orbi DNS back to automatic or a known-good upstream.
2. Reboot/renew leases on client devices.
3. Manually set laptop DNS temporarily to a public resolver if needed.
4. Debug Pi-hole after the house has internet again.

## Cockpit

Purpose:

- Browser dashboard for CPU/RAM/disk, logs, services, and terminal.

URLs:

```text
https://192.168.1.20:9090
https://100.78.92.57:9090
```

Self-signed certificate warnings are expected.

## Uptime Kuma

Purpose:

- Simple service status monitoring.

Path/URL:

```text
/srv/containers/uptime-kuma
http://192.168.1.20:3001
http://100.78.92.57:3001
```

Notes:

- Uses SQLite; this is appropriate for a small home monitor.
- Avoid Docker-socket/container monitoring for now because mounting Docker socket gives very broad Docker control.
- Monitor URLs should use correct scheme: Pi-hole is HTTP, Cockpit is HTTPS.

Commands:

```bash
cd /srv/containers/uptime-kuma
sudo docker compose ps
sudo docker compose up -d
sudo docker compose down
sudo docker compose logs --tail=80
```

## Local LLM stack: Ollama + Open WebUI / TomGPT

Path:

```text
/srv/containers/llm
```

URLs:

```text
Open WebUI/TomGPT LAN: http://192.168.1.20:3002
Open WebUI/TomGPT Tailscale: http://100.78.92.57:3002
Homebox Ollama host API: http://127.0.0.1:11434
Docker-internal Ollama URL: http://ollama:11434
```

Architecture:

- Ollama runs local models.
- Open WebUI provides TomGPT-style web chat.
- Open WebUI should reach Ollama using `OLLAMA_BASE_URL=http://ollama:11434` inside Docker.
- Ollama raw API should remain localhost-bound unless intentionally changed.
- Stable `WEBUI_SECRET_KEY` should live in `/srv/containers/llm/.env` and never be committed.

Common local models:

```text
gemma3:4b
qwen2.5:7b
qwen2.5vl:7b
qwen3:8b
```

Vulkan acceleration:

- Homebox local Ollama was tested with AMD Radeon 780M Vulkan acceleration.
- Reported result for `gemma3:4b`: about 14.5 output tokens/sec with Vulkan versus about 9.9 output tokens/sec without Vulkan.
- Longer-prompt tests were similar.
- Temperature observation: about 55 C with Vulkan versus about 66 C without Vulkan.
- Vulkan settings belong under the `ollama` service, not `open-webui`.
- Expected compose pattern includes:

```yaml
environment:
  OLLAMA_VULKAN: "1"
devices:
  - /dev/dri:/dev/dri
```

If `/dev/kfd` exists, it may also be passed through:

```yaml
  - /dev/kfd:/dev/kfd
```

Open WebUI lessons learned:

- Open WebUI is useful for model management and simple chat.
- Advanced model/tool/search settings were confusing and appeared to bleed across models.
- Native function calling broke non-tool-capable local models such as `gemma3:4b`, causing errors like `does not support tools`.
- Keep global Function Calling on Default/Off unless deliberately testing a tool-capable model.
- Keep web search off for casual local chat unless deliberately testing RAG/search.

Commands:

```bash
cd /srv/containers/llm
sudo docker compose up -d
sudo docker compose ps
sudo docker compose logs --tail=80
sudo docker compose logs --tail=80 ollama
sudo docker compose logs --tail=80 open-webui
sudo docker compose restart ollama
sudo docker compose restart open-webui
sudo docker compose exec ollama ollama list
curl http://127.0.0.1:11434/api/tags
```

Vulkan checks:

```bash
cd /srv/containers/llm
sudo docker compose config | grep -A60 -n "ollama:"
sudo docker compose exec ollama env | grep OLLAMA
sudo docker compose exec ollama sh -lc 'ls -l /dev/dri /dev/kfd 2>/dev/null || true'
sudo docker compose logs --tail=120 ollama
```

## LibreChat

Path:

```text
/srv/containers/librechat
```

URL:

```text
http://192.168.1.20:3012
http://100.78.92.57:3012
```

Purpose:

- New ChatGPT-like frontend being tested alongside Open WebUI.
- Works with both Homebox-local Ollama and the Windows GPU Ollama worker.
- Kept in its own folder rather than under `/srv/containers/llm` because it has separate compose/config/database/login state.

Known state:

- LibreChat is reachable on port `3012`.
- Homebox Ollama endpoint works through the Docker network.
- Windows GPU Ollama endpoint works when the GPU PC is awake/reachable.
- LibreChat can show manually listed models from `librechat.yaml` even if the model does not really exist in Ollama, especially if `fetch: false` is used for the GPU endpoint.

Endpoint intent:

```text
Ollama Homebox: http://ollama:11434/v1/
Ollama GPU:     http://192.168.1.35:11434/v1/
```

The Docker override mounts `librechat.yaml` into the API container and attaches the API service to the external `llm_default` network.

Startup note:

- First startup warned that `UID` and `GID` variables were unset. This was not immediately fatal.
- If needed, add UID/GID to `.env`:

```bash
cd /srv/containers/librechat
grep -q '^UID=' .env || echo "UID=$(id -u)" >> .env
grep -q '^GID=' .env || echo "GID=$(id -g)" >> .env
sudo docker compose up -d
```

Commands:

```bash
cd /srv/containers/librechat
sudo docker compose up -d
sudo docker compose ps
sudo docker compose logs --tail=80 api
sudo docker compose logs -f api
docker compose config
nano librechat.yaml
sudo docker compose up -d
```

Current recommendation:

- Keep Open WebUI on port `3002` as fallback/admin/model playground.
- Keep LibreChat on port `3012` as the main candidate for a more polished daily TomGPT-style frontend.
- Do not replace Open WebUI yet.
- Do not enable web search, agents, RAG, or tools in LibreChat until plain chat is solid.

## Windows GPU worker / remote Ollama

Machine:

- Reserved LAN IP: `192.168.1.35`
- Wi-Fi MAC for Wake-on-LAN: `E4-C7-67-06-2F-AC`
- Ollama API endpoint: `http://192.168.1.35:11434`
- GPU: son's Windows gaming PC with RTX 5070 Ti-class GPU
- Ollama configured on Windows with `OLLAMA_HOST=0.0.0.0:11434`

Wake-on-LAN:

- WOL over Wi-Fi was tested and works from Homebox.
- Monitor may stay dark even when the PC wakes.
- Ping may fail even when Ollama works; use `curl` to `/api/tags` as the real health check.
- Windows may go back to sleep quickly unless unattended sleep timeout/power settings are adjusted.

Wake command:

```bash
wakeonlan -i 192.168.1.255 -p 9 E4:C7:67:06:2F:AC
```

Wake and test:

```bash
wakeonlan -i 192.168.1.255 -p 9 E4:C7:67:06:2F:AC
sleep 15
curl --connect-timeout 10 http://192.168.1.35:11434/api/tags
```

Windows adapter settings tested:

- Wake on Magic Packet: Enabled
- Wake on Pattern Match: Disabled
- Allow this device to wake the computer: Checked
- Only allow a magic packet to wake the computer: Checked
- Allow the computer to turn off this device to save power: this needed to remain checked for wake to work reliably in Tom's test.

Windows unattended sleep commands suggested:

```powershell
powercfg -attributes SUB_SLEEP 7bc4a2f9-d8fc-4469-b07b-33eb785aaca0 -ATTRIB_HIDE
powercfg /setacvalueindex SCHEME_CURRENT SUB_SLEEP 7bc4a2f9-d8fc-4469-b07b-33eb785aaca0 900
powercfg /S SCHEME_CURRENT
```

GPU model aliases used/referenced:

```text
gpu-gemma4:12b
gpu-qwen2.5vl:7b
gpu-gemma3:4b
gpu-gemma4-chatgpt:12b   # may be configured-only; verify actual existence
```

Check actual GPU model list:

```bash
curl http://192.168.1.35:11434/api/tags
```

Remote Ollama commands:

```bash
curl http://192.168.1.35:11434/api/tags

curl -N http://192.168.1.35:11434/api/pull \
  -H "Content-Type: application/json" \
  -d '{"model":"MODEL_NAME"}'

curl http://192.168.1.35:11434/api/copy \
  -H "Content-Type: application/json" \
  -d '{"source":"SOURCE_MODEL","destination":"DESTINATION_MODEL"}'

curl http://192.168.1.35:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{"model":"MODEL_NAME","prompt":"Say hello from the Windows GPU worker.","stream":false}'
```

Performance observations:

- Homebox local `gemma3:4b` with Vulkan: about 14.5 tok/s.
- Homebox local `gemma3:4b` without Vulkan: about 9.9 tok/s.
- GPU worker `gpu-gemma3:4b`: about 180 tok/s.
- GPU worker `gpu-gemma4:12b`: about 79.6 tok/s.
- GPU worker Gemma 4 image transcription with `think:false`: about 65 tok/s, but quality was not good enough for Recipe Forge default OCR.

Recipe Forge GPU lesson:

- GPU direct connectivity from Recipe Forge worked.
- Gemma 4 is promising for text/reasoning but not reliable enough as the default image OCR/transcription model through Ollama.
- Qwen vision remains the better practical OCR/image transcription model.
- Daily Recipe Forge should remain homebox-local for simplicity unless there is a specific reason to wake/use the GPU.

Recommended future architecture:

```text
Recipe Forge -> Homebox LLM router -> Windows GPU if awake/reachable -> Homebox Ollama fallback
```

This would avoid hard-coding Recipe Forge directly to the Windows PC.

## Paperless-ngx

Path/URL:

```text
/srv/containers/paperless-ngx
http://192.168.1.20:8000
http://100.78.92.57:8000
```

Purpose:

- Private searchable document archive with OCR, tagging, document types, correspondents, and full-text search.

Known stack:

- Paperless web/app container
- PostgreSQL database container
- Redis broker container

Important notes:

- Recipe PDFs were uploaded as an early use case.
- Office document support may need Tika/Gotenberg later.
- `docker-compose.env` contains secret-looking values; never upload or commit it.
- Paperless can become a real personal records archive, so it needs a backup/export plan before important documents become irreplaceable.

Commands:

```bash
cd /srv/containers/paperless-ngx
sudo docker compose ps
sudo docker compose logs --tail=80
sudo docker compose up -d
sudo docker compose down
sudo docker compose pull
sudo docker compose up -d
```

## Mealie and Tandoor

Mealie:

```text
/srv/containers/mealie
http://192.168.1.20:3005
http://100.78.92.57:3005
host 3005 -> container 9000
```

Status:

- Installed and working.
- Initial recipe JSON imports worked technically.
- Natural-language ingredient parser is fussy and tends to require manual ingredient adjustment.
- Still may be useful for meal planning/shopping lists, but not the primary bulk import workflow.

Tandoor:

```text
/srv/containers/tandoor
http://192.168.1.20:3007
http://100.78.92.57:3007
host 3007 -> web container port 80
```

Status:

- Installed/loaded for evaluation.
- Uses Docker Compose stack with web and PostgreSQL containers.
- Initial setup included fixing a `.env` copy/paste issue where a shell heredoc line was accidentally written into `.env`.
- PDF/external-recipes workflow appeared more cumbersome than desired.
- Not adopted as the primary recipe workflow.

Decision:

- Recipe Forge is the primary custom recipe/cocktail workflow.
- Mealie and Tandoor can remain as references/alternatives unless they become clearly useful.

## Recipe Forge

Path/URLs:

```text
/srv/containers/recipe-forge
http://192.168.1.20:3006
http://100.78.92.57:3006
host port 3006 -> container port 5000
Durable data: /srv/containers/recipe-forge/data
```

Purpose:

- Private LAN/Tailscale-only AI-assisted recipe and cocktail library.
- Imports food recipes and cocktails from PDFs, pasted text, URLs, single images, and ordered multi-image sets.
- Every import goes through review-before-save.
- This is not a public SaaS app; it is a private household kitchen/bar notebook.

Stack:

- Python 3.12 slim container
- Flask 3.0.3
- Gunicorn 22.0.0
- SQLite
- PyMuPDF for PDF text extraction
- BeautifulSoup/requests for URL extraction
- Local Ollama for text and vision processing
- Plain HTML/CSS/JavaScript frontend
- No frontend build step

Deployment:

```bash
cd /srv/containers/recipe-forge
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
curl http://127.0.0.1:3006/api/health
```

Important environment defaults:

```text
OLLAMA_BASE_URL=http://ollama:11434
OLLAMA_MODEL=gemma3:4b
OLLAMA_TEXT_MODEL=
OLLAMA_VISION_MODEL=qwen2.5vl:7b
OLLAMA_API_STYLE=ollama
OLLAMA_TIMEOUT=600
OLLAMA_KEEP_ALIVE=30m
OLLAMA_TEXT_NUM_CTX=8192
OLLAMA_VISION_NUM_CTX=8192
MAX_UPLOAD_MB=100
MAX_TEXT_CHARS=45000
```

If vision imports fail with context-size errors, consider increasing `OLLAMA_VISION_NUM_CTX` to `12288` or `16384` if memory allows.

Hard rules:

- Do not change Docker ports unless explicitly asked.
- Do not expose publicly.
- Do not delete/move/rewrite/commit `./data`.
- Do not break existing imports, saved recipes, favorites, collections, shopping lists, cooking checklist behavior, scaling, or cocktails.
- Cocktails are a separate first-class module, not just recipe tags.
- Do not integrate cocktails into food shopping lists or food recipe collections unless explicitly requested.
- Schema changes require ordered SQL migrations in `migrations/`.

Current top-level pages:

- `/` and `/library` - food recipe library, search/sort/filter, favorites, collections, shopping basket, cooking/detail view.
- `/import` - Import Hub for food recipes: PDF, text, URL, image, multi-image.
- `/review` - import queue and review workspace.
- `/cocktails` - first-class cocktail library/import/review/detail/edit/favorite/filter/scale UI.
- `/shopping-lists` - food recipe shopping list management.
- `/collections` - food recipe collection management.

Visual design:

- Warm cookbook style: cream paper background, sage primary actions, warm orange secondary actions, soft borders, compact cards, gentle shadows, serif headings.
- Cocktails use the same visual system with a darker bar notebook accent.
- Avoid one-note palettes, oversized marketing layouts, nested cards, and heavy decorative backgrounds.

Food recipe capabilities:

- Upload one or more PDFs.
- Paste recipe text.
- Import recipe URL.
- Upload/drop/paste single image.
- Stage multiple images as one ordered recipe import.
- JPG, PNG, WebP supported.
- HEIC/HEIF intentionally rejected for now.
- Imports become persistent SQLite `import_jobs` and survive browser reloads.
- Running jobs reset to queued on app startup.
- Only one import job is claimed/running at a time to avoid overloading Ollama.
- Queued/running jobs can be canceled/dismissed.
- Ready/failed drafts can be retried.
- Drafts are editable before save.
- Saved recipes support favorites, collections, search/sort/filter, JSON download, ZIP download, deletion without deleting source files, checklist state, screen wake lock, and non-destructive scaling.

Image import behavior:

- OCR-first: vision model transcribes visible recipe/cocktail text.
- Extracted text is then sent to text parser.
- Multi-image imports transcribe images sequentially to reduce Ollama load/context problems.
- Review includes OCR/debug extracted text view.
- Old failed review cards keep old warning until Retry Import is clicked.

Ingredient parsing/scaling:

- `original_text` remains source of truth.
- Soft parsing extracts quantity, unit, item, note, scalable flag, confidence.
- Ambiguous lines stay unscalable or unchanged.
- Scaling is a cooking-view calculation and does not overwrite saved original.

Collections:

- Stored in SQLite.
- Recipes can belong to multiple collections.
- Collections can be created, renamed, deleted, and used as library filters.
- Deleting a collection does not delete recipes.

Shopping lists:

- Food recipes can be added to a shopping basket.
- Basket items can use original servings, desired servings, or scale factor.
- Lists support rename, archive, delete, regenerate, copy unchecked items, print, manual items, edit item text, check/uncheck, delete item, and clear checked.
- Latest user-requested improvement: when adding a recipe, the `+` action should ideally let the user choose which active/recent shopping list to add to, rather than always routing through a less-obvious basket flow.

Cocktails:

- Separate first-class module with separate tables/UI.
- Supports pasted text, URL, single image, and multi-image imports.
- Review-before-save uses cocktail-specific fields and validation.
- Library supports compact browsing, search/filtering, favorites, import cards, detail/bar card, create/edit, and scaling.
- Filters include All, Favorites, Base spirit, Method, Style, Tags.
- Scaling supports 1 drink, 2 drinks, 4 drinks, and custom batch.
- Cocktail-aware scaling behavior:
  - `normal`: scale confidently.
  - `cautious`: scale numeric values but show check-to-taste note.
  - `garnish`: preserve/adjust by eye.
  - `top-off`: preserve/add to taste or volume.
  - `do-not-scale`: preserve original.
- Bitters/dashes/drops/barspoons/egg white/mint/rinses are cautious.
- Garnishes/top-offs do not scale automatically.
- Original ingredient text is preserved.

Code map:

```text
compose.yaml                    Compose service, port, env defaults, Ollama network, data mount
Dockerfile                      Python image, requirements, app/migrations, Gunicorn
requirements.txt                Flask, Gunicorn, PyMuPDF, requests, BeautifulSoup
app/app.py                      Flask routes, APIs, import queue, worker, URL/image/PDF/text processing, cocktails
app/store.py                    SQLite helpers for recipes, imports, collections, shopping lists, cocktails
app/database.py                 data dir, SQLite connection, migrations, path helpers
app/llm.py                      Ollama native/OpenAI-compatible requests, text/vision, prompts, OCR
app/pdf_extract.py              PDF text extraction
app/recipes.py                  JSON cleanup/parsing/fallbacks/JSON-LD helpers
app/ingredient_parser.py         soft ingredient parser and scaling formatter
app/validation.py               validation helpers
app/templates/index.html         single app template with route-aware sections
app/static/app.js               client state/rendering/routing/import/review/library/cocktails/shopping UI
app/static/styles.css            visual system and responsive CSS
app/prompts/recipe_jsonld.txt    food recipe extraction prompt
app/prompts/cocktail_json.txt    cocktail extraction prompt
migrations/*.sql                ordered SQLite migrations
```

Data layout:

```text
data/uploads/                   original uploaded PDFs/images/multi-image assets
data/extracted/                 extracted PDF/image text and multi-image page text
data/raw_text/                  pasted text and readable URL text
data/raw_html/                  fetched URL HTML
data/generated/                 raw model outputs for review drafts
data/output/                    legacy JSON outputs
data/recipe_forge.sqlite        SQLite database
```

Migrations through current handoff:

```text
001_initial_schema.sql
002_import_jobs.sql
003_source_documents_ignored_at.sql
004_recipe_favorites.sql
005_source_document_import_sources.sql
006_recipe_collections.sql
007_shopping_lists.sql
008_source_document_assets.sql
009_cocktails.sql
```

Backup commands:

```bash
tar -czf recipe-forge-data-$(date +%Y%m%d-%H%M%S).tgz -C /srv/containers/recipe-forge data

sqlite3 /srv/containers/recipe-forge/data/recipe_forge.sqlite \
  ".backup '/srv/containers/recipe-forge/data/recipe_forge-backup.sqlite'"
```

Safe throwaway backend test:

```bash
cd /srv/containers/recipe-forge
tmpdir=$(mktemp -d)
RECIPE_FORGE_DATA_DIR="$tmpdir" python3 -m flask --app app.app run --port 5010
```

Useful checks:

```bash
python3 -m py_compile app/*.py
sqlite3 data/recipe_forge.sqlite "select * from schema_migrations;"
git diff --name-only
git diff
```

Known caveats:

- Code changes require Docker rebuild.
- Node is not installed on the host, so frontend syntax is usually checked by review/browser testing.
- HEIC/HEIF not supported yet.
- OCR quality depends heavily on image quality and local vision model.
- Tough two-photo iPhone imports may fail due to skew, crop, glare, tiny text, or multi-column source layout.
- Gemma 4 vision through Ollama is not reliable enough to be default OCR; use Qwen vision.

Likely future improvements:

1. Shopping-list selector when adding a recipe.
2. OCR correction workflow before parsing.
3. HEIC/HEIF conversion support, borrowing ideas from Photo Stream.
4. LLM router/fallback: GPU if awake, Homebox local otherwise.
5. Side-by-side model test tool for OCR outputs.
6. Better source-image cropping/rotation guidance for phone photos.
7. More polished cocktail batching/bar workflow if used often.

## Photo Stream

Path/URLs:

```text
/srv/containers/photo-stream
http://192.168.1.20:3003
http://100.78.92.57:3003
host port 3003 -> container port 5000
```

Purpose:

- Private photo-sharing app for Tom and Pharin.

Stack/features:

- Flask, Gunicorn, SQLite, Pillow, pillow-heif.
- Shared PIN login.
- Multi-photo upload.
- Captions/albums/timeline/favorites/comments/album filtering.
- Soft-delete and original downloads.
- PWA manifest/service worker.
- HEIF/HEIC-style iPhone uploads took work but now work.
- Supports many image formats and validates image bytes.

Data/privacy:

- Do not commit uploaded photos, app DB, `.env`, secrets, or user data.
- This is not a complete photo backup system; important photos need separate backup.

Typical commands:

```bash
cd /srv/containers/photo-stream
git status
git diff --name-only
git diff
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
```

## Reframe Journal

Path/URLs:

```text
/srv/containers/reframe-journal
http://192.168.1.20:3004
http://100.78.92.57:3004
host port 3004 -> container port 5000
```

Purpose:

- Private iPad-friendly guided journal for thought reframing, gratitude, mood check-ins, typed entries, and handwritten pages.

Current features:

- Typed guided fields.
- Apple Pencil/touch/mouse handwriting canvas.
- Pen tools, eraser, undo.
- Saves PNG handwriting and raw Apple Pencil/pointer stroke JSON.
- Browse by date, saved entry view, search across text fields/tags.

Data:

```text
./data/reframe_journal.sqlite3
./data/drawings/
./data/strokes/
```

Privacy:

- Treat as sensitive personal content.
- Never commit journal entries, drawings, stroke JSON, databases, PINs, `.env`, secrets.

UX direction discussed:

- Prefer a clearer entry-mode choice:
  - Typed guided entry
  - Handwritten journal page
- Handwritten page should be one large lined canvas with optional searchable summary.
- Future OCR/search could process handwriting images/strokes into text for search, manual correction, export, and local LLM theme analysis.

Development notes:

- Flask lives in Docker; host Python may not have dependencies.
- A per-app `.venv` or `scripts/check.sh` can be used for syntax/import checks.
- Docker Compose remains the real runtime test.

Commands:

```bash
cd /srv/containers/reframe-journal
git status
git diff --name-only
git diff
./scripts/check.sh
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
```

## Weight Tracker

Path/URL:

```text
/srv/containers/weight-tracker
http://192.168.1.20:8090
http://100.78.92.57:8090
```

Status:

- Works from iPhone and PC.
- Saves data on the server.
- Dockerized Flask/Gunicorn/SQLite app.
- Under local Git and private GitHub repo.
- Visual style: blue/silver.
- BMI tracking added with Codex.

BMI behavior:

- Weight entry remains main daily workflow.
- Height is stored separately in Profile/Settings.
- User does not manually enter BMI.
- BMI = `weight_lb * 703 / height_in^2`.
- BMI appears only if height has been saved.

Commands:

```bash
cd /srv/containers/weight-tracker
git status
git diff --name-only
git diff
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
```

## Circle Run / Homebox Arcade

Stable/family version:

```text
/srv/containers/homebox-arcade
http://192.168.1.20:3008
http://100.78.92.57:3008
host port 3008 -> nginx/container port 80
```

Purpose:

- Private LAN/Tailscale-only browser arcade game built with static HTML/CSS/JS and canvas.
- Began as Neon Maze Runner / Homebox Arcade and became Circle Run.

Current status:

- Stable/family version is intended as the working version to play with Nathan/the boys.
- Supports desktop keyboard controls and mobile touch controls.
- LocalStorage best score/high score.
- Neon/dark arcade look.
- Enemy movement/pathing improved to avoid freezing/stalling.
- Player death has dramatic visual effects.
- Private avatar assets can be used but should never be committed.

Privacy:

- `assets/private/` should be ignored by Git.
- Anyone with LAN/Tailscale access to the game may still be able to request private image URLs directly.

Commands:

```bash
cd /srv/containers/homebox-arcade
git status
git diff --name-only
git diff
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
```

## Helena's Run / Circle Run Lab

Path/URLs:

```text
/srv/containers/circle-run-lab
http://192.168.1.20:3009
http://100.78.92.57:3009
host port 3009 -> container port 80
```

Identity:

- Current game name: Helena's Run.
- Started as Neon Maze Runner: Touch Edition.
- Experimental/lab version with private avatars and background music.

Stack:

- Plain HTML/CSS/JavaScript canvas game.
- Tiny Node HTTP server for static files and `/api/leaderboard`.
- Docker Compose.
- Persistent shared leaderboard JSON at `data/leaderboard.json`.

Main files:

```text
index.html
styles.css
game.js
server.js
Dockerfile
compose.yaml
README.md
TODO.md
```

Implemented gameplay:

- Neon maze coin/star collection.
- Desktop controls: arrow keys/WASD.
- Mobile controls: swipe on arena plus bottom action bar.
- Score/lives/best/level HUD.
- Character select persists in localStorage.
- Private character files expected:

```text
assets/private/characters/tom.png
assets/private/characters/nathan.png
assets/private/characters/colin.png
assets/private/characters/pharin.png
```

- Selected character is player; other three render as enemies.
- Missing images fall back to neon/default rendering.
- Top-three leaderboard persists across devices.
- Turn assist and corridor centering.
- Enemy pathing improved with tile-center direction choices, dead-end reversal, and unstick fallback.
- 10 hand-authored/data-driven levels.
- Power-ups: Shield, Freeze, Speed, Magnet.
- `Power ?` legend overlay.
- Hidden developer mode by tapping/clicking title.

Planning notes:

- Procedural generation was discussed but deferred. Add more hand-authored levels first because procedural mazes can create fairness/pathing problems on mobile.
- Next: playtest all 10 levels on iPhone, tune power-ups/enemies/coin density, add real victory screen after Level 10, sound effects/mute, countdown, difficulty modes.

Deploy:

```bash
cd /srv/containers/circle-run-lab
sudo docker compose up -d --build
```

## Bumper Dots

Path/URLs:

```text
/srv/containers/bumper-dots
http://192.168.1.20:3010
http://100.78.92.57:3010
host port 3010 -> container port 3000
```

Purpose:

- Family-friendly real-time multiplayer arcade game for Tom and kids on iPhones.
- Private LAN/Tailscale only.
- No accounts, external services, paid services, or public runtime assumptions.

Do not touch:

- Helena's Run
- Circle Run
- Homebox Arcade

Stack:

- Node.js backend
- Express static serving
- Socket.IO real-time multiplayer
- Plain HTML/CSS/JavaScript frontend
- Canvas rendering
- Docker Compose
- Optional JSON persistence under `./data/leaderboard.json`

Important files:

```text
server.js                  Express, Socket.IO, game loop, collisions, scoring, power-ups, leaderboard
public/shared.js           constants, arena definitions, colors, power-ups, helpers
public/client.js           UI flow, socket events, HUD, sound, haptics, localStorage
public/renderer.js         canvas rendering, interpolation, camera, effects
public/input.js            mobile drag joystick and desktop controls
public/styles.css          mobile-first layout, safe-area handling
public/index.html          home/lobby/game/results/problem screens
compose.yaml               port 3010, data mount
Dockerfile                 production Node image
README.md                  run/deploy/test docs
TODO.md                    known limits/future ideas
```

Gameplay:

- Room codes.
- Host starts 60-second rounds.
- Host migrates gracefully when possible.
- Server owns canonical game state.
- Clients send input intent only.
- Players collect star coins, bump each other, use power-ups, and compete for score.
- Last 10 seconds are double points.
- Winner screen shows standings, achievements, and confetti.
- Arenas: Sunny Scramble, Wiggle Works, Popcorn Plaza.
- Power-ups: Zoom, Shield, Magnet, Pulse.
- Mobile drag-anywhere joystick; desktop WASD/arrows/Space.

Tuning:

- Gameplay constants live in `public/shared.js`.
- Current movement tuning: `baseSpeed: 440`, `acceleration: 16`, `speedBoostMultiplier: 1.58`.
- Tune in small increments and test on phones.

Deploy/test:

```bash
cd /srv/containers/bumper-dots
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
curl http://192.168.1.20:3010/health
npm run check
```

Known V1 limits:

- Rooms disappear after server restart.
- Reconnect depends on localStorage token while room still exists.
- Late joiners can enter active round but start at zero.
- Clipboard copy/haptics are browser/device dependent.

Future ideas:

- In-lobby leaderboard.
- Round length setting: 60/75/90 seconds.
- More arenas.
- Optional bot dots.
- Better reconnect/resume prompt.
- More stat callouts and close-finish messages.

## Thomas Bear's Volley Clash

Path/URLs:

```text
/srv/containers/arcade-tennis
http://192.168.1.20:3011
http://100.78.92.57:3011
host port 3011 -> container port 3000
```

Identity:

- App name: Thomas Bear's Volley Clash.
- Private family arcade tennis/volleyball-style game.
- Stack: Node.js, Express, Socket.IO, plain HTML/CSS/JS, canvas, Docker Compose.

Privacy guardrails:

- Do not touch other apps under `/srv`.
- Do not change host port `3011`.
- Do not add accounts, external APIs, CDNs, public internet assumptions, or service workers.
- Do not commit `data/`, `assets/private/`, `.env`, `node_modules/`, logs, DB files, runtime files.

Human avatars:

- Real-name private family avatars for human players only.
- Name matching:
  - `Tom` -> Tom avatar
  - `Colin` -> Colin avatar
  - `Nate` / `Nathan` -> Nathan/Nate avatar
  - `Pharin` -> Pharin avatar
- Display name stays as typed.
- Avatars are visual only; server-owned circular hitboxes remain.
- Bots must never use private family avatars.

CPU bots:

- CPU autofill always on.
- Humans replace CPU slots naturally.
- CPU Personalities default on.
- Generic CPU mode remains available.
- Current personalities:
  - Captain Lobster
  - Chaos Pickle
  - The Wall
  - Zippy McZoom
  - Professor Spin
  - Smashasaurus Rex
  - Grandma Pancake

Recent playability changes:

- Big Controls with per-device localStorage preference.
- iPhone/small touch devices default to Big Controls.
- Larger Swing/Lob/Smash buttons.
- Button flash and floating feedback labels.
- Easier joystick.
- Shot Assist: Off / Light / Family.
- Server-owned Smart Swing.
- Swing input buffering and weak reach shots.
- Rally Speed: Chill / Normal / Fast.
- Ball Visibility: Normal / High.
- Orientation hint on phone.
- Faster match-over flow with primary Rematch.
- Family Defaults button.

Latest adjustments:

- Removed `LAN family arcade tennis` eyebrow from title screen.
- CPU personalities on by default, including Family Defaults.
- Lobby setup less crowded, with advanced settings collapsed under `More Setup Options`.
- Lob has real tactical purpose: deep backcourt lobs can pass over net-crowders; net players can counter with Smash/overhead timing; near-net lobs become short/weak.
- Ball height has landing bounce/dampen.
- `staaank.png` flashes only for player/team that lost a point.
- `staaank.m4a` wired into audio and improved for iPhone reliability.
- Public generated PNG assets cleaned to RGBA/transparent edge mattes.
- Renderer keeps local player's team at bottom; Red-side local view rotates and inverts outgoing movement input.

Important files:

```text
server.js              authoritative rooms, match loop, scoring, bots, avatars, lob/net behavior
public/shared.js       constants, modes, courts, CPU profiles, asset map, playability constants
public/client.js       lobby/game UI, Socket.IO, settings, HUD, overlays
public/input.js        joystick/action buttons/keyboard input
public/renderer.js     canvas rendering, avatars, ball visibility, court, effects
public/audio.js        generated sound/music, iPhone audio fallback
public/styles.css      responsive layout and controls
public/index.html      home/lobby/game markup
HANDOFF.md             fuller engineering handoff
README.md              user/deployment docs
TODO.md                limitations/future ideas
```

Commands/tests:

```bash
cd /srv/containers/arcade-tennis
git diff --name-only
npm run check
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
```

Note: In some shell sessions, `npm`/`node` may not be available on the host. Docker build remains deployment path.

Recent user-desired improvement:

- Add a setup-screen button that explains controls.
- Keyboard controls should remain documented for desktop testing.
- Consider a match-intro use of animated/generated character assets, but avoid sound unless deliberately added and muteable.

## Codex / GitHub workflow

Core idea:

```text
Working files -> git add -> staging area -> git commit -> local Git history -> git push -> GitHub
```

GitHub stores a cloud copy of the project history. Codex edits code. Docker/Homebox runs the app.

Default division of labor:

- Codex may inspect files, propose plans, edit code, and run harmless non-sudo checks.
- Tom should manually run sudo Docker commands for now.
- Tom decides when to commit and push.

Existing app workflow:

```bash
cd /srv/containers/<app-name>
git status
# ask Codex for focused change
git diff --name-only
git diff
# optional quick check script if present
./scripts/check.sh
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
# test in browser/iPhone/iPad
# if it works:
git add .
git commit -m "Short meaningful message"
git push
```

New app workflow:

```bash
mkdir -p /srv/containers/<app-name>
cd /srv/containers/<app-name>
git init
git branch -M main
# create .gitignore
# create AGENTS.md
# ask Codex to scaffold after proposing a plan
sudo docker compose up -d --build
# test
# create empty private GitHub repo in browser
git remote add origin https://github.com/<user>/<repo>.git
git push -u origin main
```

Starter `.gitignore`:

```gitignore
data/
*.db
*.sqlite
*.sqlite3
.env
.env.*
__pycache__/
*.pyc
.venv/
venv/
node_modules/
dist/
.DS_Store
.vscode/
assets/private/
logs/
```

Starter `AGENTS.md`:

```markdown
# Codex instructions

This app runs on Tom's home server, homebox.

Environment:
- Ubuntu Server
- Docker Compose
- Apps live under /srv/containers/<app-name>
- Data should persist under ./data and should not be committed
- Remote access is LAN/Tailscale only
- Do not assume public internet exposure
- Do not change ports without explaining why
- Do not commit secrets, .env files, databases, private assets, or user data
- Prefer simple, readable code over clever frameworks
- Before large edits, explain the plan
- After edits, summarize changed files and test steps
```

Starter Codex prompt for a new Dockerized Flask app:

```text
You are working in /srv/containers/<app-name> on my Ubuntu home server.

Create a small Dockerized web app from scratch.

Requirements:
- Python Flask backend
- Simple HTML/CSS/JavaScript frontend
- SQLite persistence stored under ./data
- Dockerfile and compose.yaml included
- App should listen on internal port 5000
- Expose it on host port <choose-unused-port-and-explain>
- Mobile-friendly layout
- No external accounts required
- No public internet assumptions
- Runs on LAN/Tailscale only
- Include a README with run/test/deploy instructions
- Include .gitignore to avoid committing data, .env, databases, secrets, private assets, or user data

Before editing, propose the file structure and plan.
```

Useful Git commands:

```bash
git status
git diff --name-only
git diff
# exit diff pager with q
git add .
git commit -m "Describe the change"
git push
git pull
git log --oneline
```

Codex permissions guide:

| Request | Usually OK? | Notes |
|---|---|---|
| `python -m py_compile app.py` | Yes | Syntax check only. |
| `git status`, `git diff` | Yes | Read-only inspection. |
| `npm run check` | Usually | Fine if dependencies exist and no server-state change. |
| `docker compose run --rm ...` | Maybe | May need sudo; prefer manual until comfortable. |
| `sudo docker compose up -d --build` | Tom runs | Rebuilds/restarts running app. |
| `git commit` / `git push` | Tom runs | Saves/uploads changes. |
| `rm ...` / delete files | Be careful | Inspect first; can destroy data. |

Troubleshooting:

- Stuck in `git diff`: press `q`.
- `git push` says `src refspec main does not match any`: commit first.
- `git status` shows private data: run `git reset`, update `.gitignore`, then stage again.
- Flask not installed on Homebox host: normal; dependencies may live inside Docker.
- App works but URL is unknown: run `sudo docker compose ps` and inspect `PORTS`.
- Codex broke something and it is uncommitted: inspect `git diff --name-only`; use `git restore .` only if you really want to discard all uncommitted changes.
- Docker socket access errors in Codex are expected under current safety model; Tom should run Docker with sudo.

VS Code/GitHub auth socket issue:

If `git push` fails with a stale VS Code Git auth socket, such as:

```text
Missing or invalid credentials.
connect ECONNREFUSED /run/user/1000/vscode-git-....sock
fatal: Authentication failed
```

Fix:

1. Open VS Code Command Palette: `Ctrl+Shift+P`.
2. Run `Developer: Reload Window`.
3. Reopen a fresh Remote SSH terminal.
4. Return to app folder and retry `git push`.

If error says repository not found, check account and remote:

```bash
git remote -v
```

## Backups and data protection

Important runtime data locations include:

```text
/srv/containers/recipe-forge/data
/srv/containers/photo-stream/data
/srv/containers/reframe-journal/data
/srv/containers/weight-tracker/data
/srv/containers/paperless-ngx/data
/srv/containers/paperless-ngx/media
/srv/containers/paperless-ngx/export
/srv/containers/paperless-ngx/consume
/srv/containers/paperless-ngx/pgdata
/srv/containers/pihole/etc-pihole
/srv/containers/uptime-kuma/data
/srv/containers/librechat    # especially Mongo/Meili/uploads/config data, but not secrets in shared docs
```

Backup posture:

- Recipe Forge, Photo Stream, Reframe Journal, Paperless, and Pi-hole are the most important to back up once they contain real data.
- Immich was discussed in the original server guide but is not currently documented as an active service here.
- For family photos/documents, do not treat a single internal SSD as a backup.
- Preferred eventual strategy: live copy + local backup drive + offsite/cloud or another physical location.

## Common deployment patterns

Custom Flask/Gunicorn/SQLite apps:

```bash
cd /srv/containers/<app>
git status
git diff --name-only
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
```

Node/Socket.IO games:

```bash
cd /srv/containers/<game>
npm run check   # if node/npm available on host
sudo docker compose up -d --build
sudo docker compose ps
sudo docker compose logs --tail=80
```

Static/Caddy dashboard:

```bash
# edit /srv/apps/homebox/index.html
# refresh browser
# only reload Caddy if Caddyfile changed
cd /srv/containers/caddy
sudo docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile
sudo docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile
```

## Decision log

- Chose Ubuntu Server + Docker + SSH/Tailscale because it is reliable and simpler than Proxmox/Windows/desktop Linux for this use case.
- Use Orbi DHCP reservation rather than manually hard-coding static IP inside Linux when possible.
- Use Tailscale instead of router port forwarding.
- Keep Pi-hole router DNS without public fallback DNS if filtering is desired.
- Keep Homebox boot drive unencrypted to allow unattended restart after outages.
- AC Loss Control / Power On after AC Loss was found and tested successfully in BIOS.
- Use `/srv/containers/<service>` and `/srv/apps/<site>` as standard layout.
- Use Caddy for simple static dashboard, not as a complicated public reverse proxy yet.
- Use Recipe Forge as the primary custom recipe/cocktail workflow because Mealie/Tandoor imports were too fussy/cumbersome.
- Keep Recipe Forge imports review-before-save because LLM extraction can drift.
- Use Qwen vision for practical OCR; Gemma 4 is not reliable enough as OCR default yet.
- Keep Recipe Forge on homebox-local Ollama for daily reliability; use GPU worker for speed/experiments/TomGPT.
- Keep Open WebUI installed but treat advanced tool/search settings cautiously.
- Keep LibreChat installed separately as the likely better daily chat frontend candidate.
- Keep game stable/lab variants separate so experiments do not break family-playable versions.

## Current gaps / next useful improvements

Highest-value infrastructure:

1. Create a real backup plan and test restore for Recipe Forge, Photo Stream, Reframe Journal, Paperless, Pi-hole, and LibreChat.
2. Add/update dashboard entry for LibreChat if not already present.
3. Add Uptime Kuma monitors for LibreChat and key game apps.
4. Make a small `wake-gpu-and-check.sh` helper on Homebox.
5. Consider an LLM router/fallback layer so Recipe Forge/Open WebUI/LibreChat can prefer GPU when awake and fall back to local Ollama.
6. Document any actual Docker restart policies and fill gaps where services do not restart after reboot.

Recipe Forge:

1. Add a shopping-list selector when adding a recipe from Library/Cooking View.
2. Add OCR correction before parse for difficult phone photos.
3. Add HEIC/HEIF conversion support.
4. Add side-by-side OCR/model testing tool.
5. Improve multi-image phone-photo guidance/cropping.

LibreChat/TomGPT:

1. Keep testing plain chat with Homebox and GPU endpoints.
2. Verify actual GPU model aliases with `/api/tags` before listing models in `librechat.yaml`.
3. Do not enable tools/web search broadly until behavior is predictable.
4. Compare daily use against Open WebUI before replacing dashboard labels.

Games:

1. Volley Clash: add controls/help screen button in setup.
2. Volley Clash: continue tuning mobile controls, lob/smash behavior, bot personalities, and sound reliability.
3. Bumper Dots: add lobby leaderboard, round length setting, more arenas, and bot dots.
4. Helena's Run: add proper victory screen after Level 10, tune levels on iPhone, add sound/mute and difficulty modes.
5. Circle Run stable: keep stable; avoid merging lab changes until playtested.

Reframe Journal:

1. Add clearer Typed vs Handwritten entry modes.
2. Add optional searchable summary for handwritten entries.
3. Later: background OCR/search/theme extraction with local models.

Photo Stream:

1. Maintain HEIF/HEIC support when upload/image code changes.
2. Add backup/export clarity before relying on it as a long-term photo archive.

Paperless:

1. Understand backup/export workflow before adding important records.
2. Consider Tika/Gotenberg if Office document import becomes important.

## Source files consolidated into this master

This master was synthesized from the uploaded project files and current project chat context, including:

- `Beginner-friendly setup guide for your Minisforum UM880 Plus Linux home server.pdf`
- `Homebox_GitHub_Codex_App_Workflow_Guide.pdf`
- `recipe-forge-README - as of 5-15-2026.md`
- `status-as-of-5-19-2026.txt`
- `Recipe-forge-status as of 5-20-2026.txt`
- `CHATGPT_HANDOFF.md`
- `BUMPER_DOTS_CHATGPT_PROJECT.md`
- `homebox-status-5-29-2026.txt`
- `CHATGPT_PROJECT_UPDATE-VC-5-31-2026.md`
- `Recipe-forge-6-4-2026.txt`
- `homebox-gpu-worker-recipe-forge-update-2026-06-07.txt`
- `homebox-ac-power-recovery-update-2026-06-14.txt`
- `homebox-librechat-gpu-update-2026-06-15.md`
- recent project chat context through 2026-06-15

Future update rule:

- Update this file directly.
- Add a short dated note near the relevant section, not a whole new sidecar file.
- Keep it accurate, current, and free of secrets.
