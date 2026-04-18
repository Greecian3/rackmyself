---
title: "The Beginner Homelab: What to Buy First and Why"
date: 2026-04-18
description: "Your first homelab doesn't need to be expensive or complicated. Here's exactly what to buy first, what to skip, and how to start running real services today."
tags: ["homelab", "beginners", "hardware", "getting started", "self-hosting"]
categories: ["guides"]
showToc: true
---

The homelab rabbit hole is deep. Look up any hardware recommendation and you'll be inundated with rack servers, 10GbE switches, UPS systems, and opinions from people running eight-drive ZFS arrays for fun. None of that is where you should start.

This guide is the opposite of that. It covers the minimum viable homelab — the stuff that actually matters for a beginner, what to buy first, and how to avoid the common trap of spending a lot of money before you know what you actually need.

## Start With a Question, Not Hardware

Before buying anything, decide what you want your homelab to actually do. The hardware requirements vary dramatically depending on the answer.

**Common starting goals:**

- Run Pi-hole or AdGuard Home (network-wide ad blocking)
- Self-host a password manager (Vaultwarden)
- Run Jellyfin for a media server
- Learn Docker and Linux
- Run Home Assistant for smart home
- Host local AI models

Some of these run fine on a $40 Raspberry Pi. Others need a real server. Knowing your goal saves you from either overbuying or hitting a wall immediately.

## The Starter Path: Mini PC First

For most beginners, the right first purchase is a mini PC. Not a Raspberry Pi (too weak for Docker-heavy setups), not a used rack server (loud, hot, expensive to run), not a NAS (limited flexibility).

A mini PC gives you:
- Low power draw (6–15W idle)
- Silent operation
- A real x86 CPU that runs everything Docker supports
- Enough RAM to run 10+ containers
- Small enough to stick behind your TV or on a shelf

### The Specific Recommendation

**[Beelink EQ12 with Intel N100](https://www.amazon.com/s?k=Beelink+EQ12&tag=YOURTAG-20)** — roughly $150–170 for the 16 GB / 500 GB model.

Why this one:
- Intel N100: 4 cores, efficient, runs everything you need for a starter homelab
- 16 GB RAM: enough for 15–20 containers, Home Assistant, and more
- 2.5 GbE: future-proofs your network connection
- 6W idle power: costs ~$8/year in electricity
- Silent
- Ships with Ubuntu-compatible hardware (everything works)

This is the correct first server for 90% of beginners. Buy this, not the $300 option, not a Raspberry Pi, not a used enterprise server.

If you have $50 more to spend, the **[Beelink EQR6 with AMD Ryzen 5 6600H](https://www.amazon.com/s?k=Beelink+EQR6&tag=YOURTAG-20)** is a meaningful step up — six real cores instead of four efficiency cores, handles more VMs and heavier workloads.

## Operating System: Ubuntu Server

Install Ubuntu Server 24.04 LTS. Not Ubuntu Desktop, not Proxmox (yet), not anything else.

Ubuntu Server reasons:
- Massive community, excellent documentation
- Docker support is first-class
- Snap packages for some things, apt for everything else
- LTS release means 5 years of security updates
- 95% of self-hosting guides assume Ubuntu/Debian

Proxmox (a hypervisor for running virtual machines) is great but adds complexity. Start with Ubuntu directly — you can migrate to Proxmox later if you need VMs.

**Installing Ubuntu:**
1. Download Ubuntu Server 24.04 LTS from ubuntu.com
2. Flash to a USB drive with Balena Etcher ([free download](https://www.balena.io/etcher/))
3. Boot the mini PC from USB
4. Follow the installer — select default storage, add your username and password, enable SSH when prompted
5. Remove USB when done, let it boot

Once it's up, SSH in from your main computer:
```bash
ssh username@your-server-ip
```

## What to Install First

### 1. Docker (Required)

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

This is your foundation. Docker lets you run almost any self-hosted app in minutes, isolated from your system.

### 2. Your First App: Portainer (Optional but Recommended)

Portainer is a web UI for managing Docker containers. Good for beginners who want a visual interface.

```bash
docker run -d \
  -p 9000:9000 \
  --name portainer \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Visit `http://your-server-ip:9000` to set it up.

### 3. Pi-hole or AdGuard Home (Network Ad Blocking)

This is the highest-impact low-effort self-hosted service. Set up Pi-hole as your DNS server and every device on your network blocks ads — no browser extensions, no per-device configuration.

AdGuard Home Docker Compose:

```yaml
services:
  adguardhome:
    image: adguard/adguardhome
    container_name: adguardhome
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "3000:3000/tcp"
    volumes:
      - ./work:/opt/adguardhome/work
      - ./conf:/opt/adguardhome/conf
    restart: unless-stopped
```

After setup, point your router's DNS settings to your server's IP. All devices on the network use your server for DNS, which filters ads at the network level.

### 4. Vaultwarden (Password Manager)

A self-hosted Bitwarden-compatible server. Install once, never pay for a password manager again.

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    ports:
      - "8080:80"
    volumes:
      - ./data:/data
    environment:
      - SIGNUPS_ALLOWED=true
    restart: unless-stopped
```

After first setup, set `SIGNUPS_ALLOWED=false` to prevent anyone else from creating accounts.

## Storage: Add a Drive Later

The EQ12 comes with 500 GB NVMe. That's enough to start. Once you know you're staying with this hobby and have more data to store, add storage:

- **For bulk storage:** A [2 TB 2.5" SATA SSD](https://www.amazon.com/s?k=2TB+2.5+SATA+SSD&tag=YOURTAG-20) (~$80) fits in the EQ12's second bay — plug and play
- **For media:** A USB 3.0 external drive works fine for Jellyfin
- **For serious storage:** A separate NAS is a later purchase

Don't buy a 4-bay NAS as your first homelab purchase. You don't know yet how much storage you actually need.

## Networking: What You Actually Need

**You don't need a managed switch.** The switch your router came with handles a beginner homelab fine.

**You don't need VLANs yet.** VLAN segmentation is a real security improvement but adds complexity. Learn the basics first.

**You do need a wired connection for your server.** WiFi introduces latency and reliability issues. Run a cable from your router to your server. If the server is in another room, a [flat Cat6 cable](https://www.amazon.com/s?k=flat+cat6+ethernet+cable+25ft&tag=YOURTAG-20) runs under doors and along baseboards invisibly.

**You should get a UPS.** Power blips corrupt drives and databases. An inexpensive UPS protects your investment.

[APC BE600M1 UPS](https://www.amazon.com/dp/B01FWYEOLY?tag=YOURTAG-20) — $55, protects against most common power events, gives your server 5–10 minutes of runtime to shut down cleanly.

## What Not to Buy First

**A rack:** Racks are for people with multiple 1U/2U servers. A single mini PC doesn't need a rack.

**A managed switch:** You don't need VLANs yet. A simple unmanaged gigabit switch is fine.

**A dedicated NAS:** Your mini PC can serve files. Add a NAS when you have data that justifies it.

**Enterprise hardware (Dell R720, HP DL380, etc.):** These are loud (70–80 dB), power-hungry (200–500W), and large. They're great at what they do but wrong for a home.

**More hardware than you can learn:** Buying everything at once means you're overwhelmed before you start. Add pieces as you have a specific reason.

## The 30-Day Plan

**Week 1:** Get the mini PC, install Ubuntu Server, SSH in, install Docker. Deploy Portainer. Get comfortable in the terminal.

**Week 2:** Deploy AdGuard Home, set your router to use it for DNS. Deploy Vaultwarden, migrate your passwords. Run both for a week without touching anything else.

**Week 3:** Deploy Jellyfin or Navidrome (music server). Move some media to your server. Learn how Docker volumes work.

**Week 4:** Set up Cloudflare Tunnels to access one service remotely. Understand what a reverse proxy does conceptually.

By day 30, you have a working homelab running real services you use daily, and you understand the fundamentals well enough to make intelligent decisions about what to add next.

## Total First Homelab Budget

| Item | Approx Cost |
|------|-------------|
| Beelink EQ12 Mini PC (16GB/500GB) | $165 |
| Flat Cat6 Ethernet cable | $12 |
| APC UPS BE600M1 | $55 |
| Domain name (optional) | $12/yr |

**Total: ~$232** plus a free afternoon.

That's it. No NAS, no rack, no managed switch, no enterprise hardware. Just a capable little server that runs quiet in the corner and runs dozens of services on $8/year of electricity.

## What Comes After

Once you've been running this for a few months and know what you actually want:

- **Proxmox:** If you want virtual machines and more isolation
- **A NAS (Synology or DIY TrueNAS):** When you have real storage needs
- **A managed switch:** When you want VLANs and network segmentation
- **A second server:** When you want high availability or Kubernetes
- **A rack:** When you have enough hardware to justify it

The homelab grows with your knowledge. Start simple, learn deliberately, and add complexity only when you have a specific reason to.

The biggest mistake in homelabs is spending $1,500 on hardware before you know what you're doing. The second biggest mistake is never starting because you think you need $1,500 of hardware. Start with $230, learn the fundamentals, and let the rest follow.
