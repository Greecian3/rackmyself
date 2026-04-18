---
title: "How to Self-Host Open WebUI and Access It From Anywhere"
date: 2026-04-18
description: "Set up Open WebUI on your home server with Docker, connect it to Ollama, and expose it securely via Cloudflare Tunnels for access anywhere."
tags: ["open webui", "ollama", "self-hosting", "docker", "cloudflare tunnels"]
categories: ["guides"]
showToc: true
---

Open WebUI is the best front end for locally-running AI models. It looks and works like ChatGPT but runs entirely on your hardware, talking to Ollama (or other backends) on your local network. Once you've set it up, you get a polished chat interface for all your local models — and with a Cloudflare Tunnel, you can access it from your phone, laptop, or anywhere else, without punching holes in your firewall.

This guide covers the full stack: Ollama + Open WebUI via Docker, then exposing it securely with Cloudflare.

## Prerequisites

Before starting, you need:

1. **Ollama installed and running** on your server — see our [Ollama setup guide](/posts/run-ollama-home-server/)
2. **Docker installed** — if not, run: `curl -fsSL https://get.docker.com | sh`
3. **A Cloudflare account** (free tier works) with your domain added (optional, but needed for remote access)

## Understanding the Architecture

Here's what we're building:

```
Your phone/laptop → Cloudflare Tunnel → Your server → Open WebUI → Ollama
```

Ollama runs the actual AI models. Open WebUI is the web interface that talks to Ollama. Cloudflare Tunnel creates an encrypted connection from Cloudflare's network to your server — no open ports required.

## Step 1: Configure Ollama for Network Access

By default, Ollama only listens on `localhost`. Open WebUI (running in Docker) can't reach `localhost` on the host. We need to tell Ollama to listen on all interfaces.

Edit the systemd override:

```bash
sudo systemctl edit ollama
```

Add:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

Reload and restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Verify it's reachable:

```bash
curl http://localhost:11434/api/tags
```

You should see a JSON response listing your models.

## Step 2: Deploy Open WebUI With Docker Compose

Create a directory for Open WebUI:

```bash
mkdir ~/docker/open-webui && cd ~/docker/open-webui
```

Create `docker-compose.yml`:

```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3000:8080"
    volumes:
      - ./data:/app/backend/data
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      - OLLAMA_BASE_URL=http://host.docker.internal:11434
    restart: unless-stopped
```

The key piece here is `extra_hosts: host.docker.internal:host-gateway`. This is a Linux-specific fix — on Mac, `host.docker.internal` works automatically, but on Linux you need to add this to make the container able to reach the host machine's Ollama service.

Start it:

```bash
docker compose up -d
```

Check the logs to confirm it started:

```bash
docker compose logs -f
```

You should see output like `Application startup complete`. Visit `http://your-server-ip:3000` to confirm it's working.

## Step 3: First-Time Setup

On first load, Open WebUI prompts you to create an admin account. The first account registered becomes the admin — do this before anyone else can access it.

After logging in:

1. Click your username in the top right → **Settings**
2. Under **Connections**, verify the Ollama URL shows as connected (green indicator)
3. Go to **Models** — you should see your Ollama models listed

Start a new chat, select a model from the dropdown, and test it out. If the models show up and you can get responses, everything is wired correctly.

## Step 4: Pull Models From the UI

You don't have to use the command line to manage models anymore. In Open WebUI:

1. Go to **Settings → Models**
2. Under "Pull a model from Ollama.com", type the model name (e.g., `llama3.1:8b`)
3. Click the download icon

Open WebUI will pull the model through Ollama. Progress shows in the UI.

## Step 5: Set Up Cloudflare Tunnel for Remote Access

To access your Open WebUI from anywhere without opening ports, use cloudflared — the Cloudflare Tunnel daemon.

### Install cloudflared

```bash
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb
```

### Authenticate

```bash
cloudflared tunnel login
```

This opens a browser window. Log in to Cloudflare and select your domain. A certificate gets saved to `~/.cloudflared/`.

### Create a Tunnel

```bash
cloudflared tunnel create homelab
```

This creates a tunnel and saves a JSON credentials file. Note the tunnel ID from the output.

### Configure the Tunnel

Create `~/.cloudflared/config.yml`:

```yaml
tunnel: YOUR_TUNNEL_ID_HERE
credentials-file: /home/YOUR_USERNAME/.cloudflared/YOUR_TUNNEL_ID.json

ingress:
  - hostname: ai.yourdomain.com
    service: http://localhost:3000
  - service: http_status:404
```

Replace:
- `YOUR_TUNNEL_ID_HERE` with the ID from the previous step
- `YOUR_USERNAME` with your Linux username
- `ai.yourdomain.com` with your actual subdomain

### Create the DNS Record

```bash
cloudflared tunnel route dns homelab ai.yourdomain.com
```

This creates a CNAME record in Cloudflare DNS pointing to your tunnel.

### Run as a System Service

```bash
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

Check it's running:

```bash
sudo systemctl status cloudflared
```

Now visit `https://ai.yourdomain.com` from any device. It should load your Open WebUI over HTTPS, with a valid certificate provided by Cloudflare. No port forwarding, no exposed home IP.

## Step 6: Lock Down Access

If you're exposing this to the internet, you should add authentication. Cloudflare Access (free tier) can put an email-based login gate in front of your tunnel.

In the Cloudflare dashboard:
1. Go to **Zero Trust → Access → Applications**
2. Click **Add an Application → Self-hosted**
3. Set the subdomain to `ai.yourdomain.com`
4. Under **Policies**, add a policy that only allows your email address

Now anyone hitting `ai.yourdomain.com` has to authenticate via Cloudflare before they even reach Open WebUI. It's a simple and effective layer of protection.

Open WebUI also has its own user management — the admin account you created on first setup can add other users with restricted permissions.

## Optional: Add More Backends

Open WebUI isn't limited to Ollama. You can also connect:

- **OpenAI API** — if you want to use GPT-4o alongside local models
- **Anthropic Claude** — via API key
- **Any OpenAI-compatible API** — including hosted services that expose the same interface

In Settings → Connections, you can add API keys for these services. Once added, cloud models appear alongside local ones in the model dropdown. Useful for comparing outputs or falling back to the cloud when you need a larger model.

## Keeping Open WebUI Updated

Pull the latest image and restart:

```bash
cd ~/docker/open-webui
docker compose pull
docker compose up -d
```

Open WebUI is under active development and gets significant updates frequently. It's worth updating monthly at minimum.

## Hardware Notes

If you're running Open WebUI + Ollama on the same machine, the main bottleneck is inference speed. A GPU makes a real difference:

- [NVIDIA RTX 4060 8GB](https://www.amazon.com/s?k=RTX+4060+8GB&tag=YOURTAG-20) — affordable entry point for GPU inference, handles 7B models smoothly
- [NVIDIA RTX 3090 24GB](https://www.amazon.com/s?k=RTX+3090+24GB&tag=YOURTAG-20) — used prices have come down, excellent VRAM for larger models

Without a GPU, CPU inference on a modern 8-core mini PC is still usable for 3B–7B models, just slower.

## Troubleshooting

**Open WebUI can't connect to Ollama:** Double-check the `OLLAMA_HOST` env var is set and Ollama is listening on `0.0.0.0`. Run `ss -tulpn | grep 11434` on the host to confirm.

**Models not showing up:** Go to Settings → Connections in Open WebUI and check the Ollama connection indicator. Try clicking "Reset Connection".

**Cloudflare Tunnel not connecting:** Check `journalctl -u cloudflared -f` for errors. Common issues are wrong tunnel ID, wrong credentials file path, or DNS propagation delay (wait 5 minutes after creating the DNS record).

**Slow responses:** This is the model, not the setup. Try a smaller quantization or a smaller model. `phi3:mini` is surprisingly good and very fast.

The full stack — Ollama + Open WebUI + Cloudflare Tunnel — takes about 30 minutes to set up and runs reliably once it's in place. It's one of the most genuinely useful self-hosting setups you can build.
