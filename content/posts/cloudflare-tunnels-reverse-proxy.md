---
title: "How to Set Up a Reverse Proxy With Cloudflare Tunnels (No Port Forwarding)"
date: 2026-04-18
description: "Expose your self-hosted apps to the internet securely using Cloudflare Tunnels. No port forwarding, no exposed home IP, free SSL included."
tags: ["cloudflare", "reverse proxy", "tunnels", "self-hosting", "security"]
categories: ["networking"]
showToc: true
---

The classic problem with self-hosting is getting traffic in. The old-school answer — port forwarding on your router — works, but it exposes your home IP address, requires a static IP or DDNS, and creates a direct path to your home network from the internet. Cloudflare Tunnels solve all of this without opening a single port.

A Cloudflare Tunnel creates an outbound connection from your server to Cloudflare's network. Traffic flows inward through that tunnel — your router never needs to be touched, your home IP stays hidden, and Cloudflare handles SSL termination and DDoS protection automatically.

This guide gets you from nothing to securely exposed services in under an hour.

## How Cloudflare Tunnels Work

Normal self-hosting flow:
```
User → Internet → Your router (port forward) → Your server
```

Cloudflare Tunnel flow:
```
User → Cloudflare (your subdomain) → Cloudflare Tunnel → Your server
```

Your server runs `cloudflared`, a small daemon that maintains an outbound connection to Cloudflare. When traffic hits your subdomain on Cloudflare, it gets routed through that tunnel to your server. The connection is encrypted end-to-end, and from the outside world, the traffic appears to originate from Cloudflare's infrastructure — your home IP is never revealed.

## What You Need

- A domain managed by Cloudflare (even on the free plan)
- A home server or mini PC running Linux
- One or more services running locally (web apps, Docker containers, etc.)

If you don't have a domain yet, you can get one cheap at Namecheap or Cloudflare's own registrar, then add it to Cloudflare and point the nameservers.

## Step 1: Install cloudflared

Cloudflared is the tunnel daemon. Install it on your server:

**On Debian/Ubuntu:**
```bash
curl -L --output cloudflared.deb \
  https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb
cloudflared --version
```

The latest release as of April 2026 is 2026.3.0. The `latest` URL above always pulls the current version automatically.

**On ARM (e.g., Raspberry Pi):**
```bash
curl -L --output cloudflared.deb \
  https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb
sudo dpkg -i cloudflared.deb
```

## Step 2: Authenticate cloudflared

```bash
cloudflared tunnel login
```

This prints a URL. Open it in a browser, log into Cloudflare, and select the domain you want to use. Cloudflare saves a certificate to `~/.cloudflared/cert.pem` on your server. This authorizes your server to create tunnels for that domain.

## Step 3: Create a Tunnel

```bash
cloudflared tunnel create homelab
```

This creates a named tunnel called "homelab" and generates a credentials JSON file in `~/.cloudflared/`. The output shows a **tunnel ID** (a UUID). Note it — you'll need it in the next step.

List your tunnels anytime:
```bash
cloudflared tunnel list
```

## Step 4: Write the Config File

Create `~/.cloudflared/config.yml`:

```yaml
tunnel: YOUR_TUNNEL_ID
credentials-file: /home/YOUR_USERNAME/.cloudflared/YOUR_TUNNEL_ID.json

ingress:
  - hostname: nextcloud.yourdomain.com
    service: http://localhost:8080

  - hostname: uptime.yourdomain.com
    service: http://localhost:3001

  - hostname: ai.yourdomain.com
    service: http://localhost:3000

  - service: http_status:404
```

Replace:
- `YOUR_TUNNEL_ID` with the UUID from Step 3
- `YOUR_USERNAME` with your Linux username
- The hostnames and ports with your actual services

The last line (`service: http_status:404`) is a required catch-all — any request that doesn't match a hostname rule returns a 404.

You can expose as many services as you want in a single tunnel. Each gets its own hostname but they all share the same tunnel connection.

## Step 5: Create DNS Records

For each hostname you added, create a CNAME record pointing to your tunnel:

```bash
cloudflared tunnel route dns homelab nextcloud.yourdomain.com
cloudflared tunnel route dns homelab uptime.yourdomain.com
cloudflared tunnel route dns homelab ai.yourdomain.com
```

This creates the DNS records automatically in Cloudflare. They'll point to `YOUR_TUNNEL_ID.cfargotunnel.com`.

You can also do this in the Cloudflare dashboard under DNS → Records, adding CNAME records manually.

## Step 6: Run the Tunnel as a System Service

For the tunnel to survive reboots, install it as a systemd service:

```bash
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

Check status:
```bash
sudo systemctl status cloudflared
journalctl -u cloudflared -f
```

You should see `INF Connection registered connIndex=0` — that means your server is connected to Cloudflare's network and ready to receive traffic.

## Step 7: Test It

Visit one of your subdomains in a browser. It should load your app over HTTPS with a valid certificate — no SSL setup required on your end. Cloudflare handles the cert automatically.

From a browser on your phone (not connected to your home WiFi), test the same URL. If it works, your tunnel is fully operational.

## Adding Cloudflare Access (Authentication Gate)

If you're exposing sensitive services, you should put an auth gate in front of them. Cloudflare Access (free for up to 50 users) can require email verification before anyone reaches your app.

In the Cloudflare dashboard:
1. **Zero Trust → Access → Applications → Add an Application**
2. Choose **Self-hosted**
3. Application domain: `nextcloud.yourdomain.com`
4. Under **Policies**, create a policy:
   - Action: Allow
   - Include: Emails → `your@email.com`
5. Save

Now when anyone (including you) visits that subdomain, Cloudflare sends a one-time code to the allowed email addresses. Only verified users get through to your app.

This is particularly useful for:
- Home dashboards you don't want public
- Admin interfaces (Proxmox, TrueNAS, etc.)
- Development environments
- AI interfaces (like Open WebUI)

## Multiple Services, One Tunnel

One of the best features of Cloudflare Tunnels is that a single `cloudflared` process handles all your services. There's no need to run multiple tunnels or multiple daemon processes.

Example of a fully-loaded config:

```yaml
tunnel: YOUR_TUNNEL_ID
credentials-file: /home/USER/.cloudflared/TUNNEL_ID.json

ingress:
  # Public services (no Access gate)
  - hostname: blog.yourdomain.com
    service: http://localhost:1313

  # Internal services (protected by Cloudflare Access)
  - hostname: proxmox.yourdomain.com
    service: https://localhost:8006
    originRequest:
      noTLSVerify: true   # Proxmox uses a self-signed cert

  - hostname: nas.yourdomain.com
    service: http://localhost:5000

  - hostname: ai.yourdomain.com
    service: http://localhost:3000

  - service: http_status:404
```

Note `noTLSVerify: true` for Proxmox — it uses a self-signed certificate that cloudflared would otherwise reject. This tells cloudflared to skip certificate verification for that origin only. The connection from user to Cloudflare is still fully encrypted.

## Updating cloudflared

Cloudflare updates the daemon periodically. Update it the same way you installed it:

```bash
curl -L --output cloudflared.deb \
  https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared.deb
sudo systemctl restart cloudflared
```

Or set up automatic updates via a cron job.

## Limitations and Tradeoffs

**Websockets:** Work fine in most cases. Some apps (like some monitoring dashboards) need `websocketTimeout` set in the ingress config.

**Large file uploads:** Cloudflare has a 100 MB upload limit on the free plan. For Nextcloud or similar apps, you can increase this in Cloudflare's network settings or subscribe to a paid plan.

**Latency:** Traffic routes through Cloudflare's CDN. For most web apps this adds minimal latency. For latency-sensitive things (game servers, real-time audio), this isn't the right tool.

**Outages:** You're dependent on Cloudflare's network. For the vast majority of use cases this is fine — Cloudflare is extremely reliable — but it's worth knowing.

**Not for everything:** Some services (SSH, raw TCP, non-HTTP protocols) require cloudflared's `--url` mode or Cloudflare's WARP-to-Tunnel feature. HTTP/HTTPS is the primary use case.

## Hardware Recommendations

Any always-on Linux machine works. The cloudflared daemon itself is extremely lightweight — under 50 MB RAM and negligible CPU. Your limiting factor is the services you're exposing, not the tunnel itself.

- [Beelink EQ12 Mini PC](https://www.amazon.com/s?k=Beelink+EQ12&tag=YOURTAG-20) — ideal low-power always-on host
- [Raspberry Pi 5](https://www.amazon.com/s?k=Raspberry+Pi+5&tag=YOURTAG-20) — ARM-based, cloudflared has an ARM build
- [APC UPS BE600M1](https://www.amazon.com/dp/B01FWYEOLY?tag=YOURTAG-20) — protect your server from power interruptions

## Summary

Cloudflare Tunnels are the cleanest way to self-host services that need to be externally accessible. No port forwarding, no exposed home IP, automatic HTTPS, and built-in DDoS protection — all on Cloudflare's free tier. The setup is under an hour, and once the service is running you rarely need to touch it.

Combined with Cloudflare Access, it's a production-grade security setup that would cost real money to replicate with traditional infrastructure. For a homelab, it's one of the best free tools available.
