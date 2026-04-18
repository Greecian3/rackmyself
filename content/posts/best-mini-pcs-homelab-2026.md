---
title: "Best Mini PCs for a Homelab in 2026 (Under $300)"
date: 2026-04-18
description: "The best mini PCs for running a homelab on a budget. We cover top picks under $300 for Docker, Proxmox, NAS, and home server workloads in 2026."
tags: ["mini pc", "homelab", "hardware", "proxmox", "budget"]
categories: ["hardware"]
showToc: true
---

Mini PCs have quietly become the best starting point for a homelab. They're cheap to run, quiet enough for a living room shelf, and powerful enough to run Docker, Proxmox, a NAS, a VPN server, and a dozen containers at once. The days of needing a rack-mounted server to do serious self-hosting are over.

This guide covers the best options under $300 as of 2026, with real-world context on what each one is actually good for.

## What to Look For in a Homelab Mini PC

Before jumping to picks, here's what matters:

**RAM:** 16 GB minimum. 32 GB is better if you're running Proxmox with multiple VMs. Many mini PCs accept upgrades — check before you buy.

**Storage:** M.2 NVMe slots matter more than included storage. A fast SSD for the OS and containers is essential. Bonus points for a 2.5" SATA bay for bulk storage.

**Network:** Gigabit Ethernet is table stakes. 2.5 GbE is increasingly common and worth paying for if you move large files around your network.

**TDP/Power draw:** This is ongoing cost. A 15W TDP machine at $0.15/kWh costs about $20/year to run. A 65W machine costs $85/year. For always-on hardware, this adds up.

**CPU generation:** Older Intel N-series (N100, N305) chips punch far above their price point for low-power workloads. AMD Ryzen Embedded and recent Intel Core Ultra chips are better for compute-heavy tasks.

## The Picks

### Best Overall: Beelink EQR6 (AMD Ryzen 5 6600H)

The EQR6 hits the sweet spot between performance and price. The Ryzen 5 6600H is a real six-core chip — not an efficiency-focused Atom derivative — which means it handles Proxmox, multiple VMs, Jellyfin transcoding, and Docker containers without breaking a sweat.

- CPU: AMD Ryzen 5 6600H (6 cores / 12 threads)
- RAM: 16 GB DDR5 (upgradeable to 64 GB)
- Storage: 500 GB NVMe + M.2 slot + 2.5" SATA bay
- Networking: 2.5 GbE
- Power draw: ~15W idle, ~45W load
- Price: ~$260

[Beelink EQR6 on Amazon](https://www.amazon.com/s?k=Beelink+EQR6&tag=YOURTAG-20)

The 2.5 GbE port is a genuine advantage — if you have a 2.5G switch or NAS, you'll feel the difference. The extra SATA bay means you can add a cheap 2 TB drive for local storage without reaching for a separate NAS.

### Best Budget: Beelink EQ12 (Intel N100)

If you're starting out and don't need raw compute power, the EQ12 is hard to beat at under $170. The Intel N100 is an efficiency chip designed for exactly this workload — low power, surprisingly capable for I/O-heavy tasks like serving files, running containers, and managing home automation.

- CPU: Intel N100 (4 cores / 4 threads, ~6W TDP)
- RAM: 16 GB DDR4
- Storage: 500 GB NVMe
- Networking: 2.5 GbE
- Power draw: ~6W idle, ~15W load
- Price: ~$160

[Beelink EQ12 on Amazon](https://www.amazon.com/s?k=Beelink+EQ12&tag=YOURTAG-20)

The N100's power draw is almost insultingly low. Running 24/7, this thing costs around $8/year in electricity. It runs Home Assistant, Pi-hole, Tailscale, Jellyfin (software transcoding only), and a handful of other containers without complaint.

Where it falls short: anything that needs real CPU muscle. Large Docker Compose stacks, Proxmox with 3+ VMs, or any video transcoding that hits the CPU hard will expose its limits.

### Best for Proxmox/Virtualization: GMKtec NucBox G9 (Intel Core Ultra 5 125H)

If you want to run Proxmox with several actual VMs — not just LXC containers — you need more cores and a modern feature set. The NucBox G9 with the Core Ultra 5 125H delivers that without blowing the budget.

- CPU: Intel Core Ultra 5 125H (14 cores / 18 threads)
- RAM: 32 GB DDR5 (upgradeable to 96 GB)
- Storage: 1 TB NVMe
- Networking: 2.5 GbE x2 (useful for pfSense/OPNsense setups)
- Power draw: ~18W idle, ~60W load
- Price: ~$290

[GMKtec NucBox G9 on Amazon](https://www.amazon.com/s?k=GMKtec+NucBox+G9&tag=YOURTAG-20)

The dual 2.5 GbE ports make this genuinely interesting if you want to run a software router. You can assign one port as WAN and one as LAN and route your whole home network through OPNsense running as a VM on Proxmox. That's a legitimately advanced setup on a $290 box.

### Best for Low-Power NAS: Topton N5105 Mini ITX Board (DIY Build)

If your primary use case is a NAS with multiple drives, a purpose-built board designed for storage beats any consumer mini PC. The Topton N5105 board has 4x SATA ports, 2x M.2 slots, 4x 2.5 GbE ports, and runs the same efficient N5105 chip used in commercial NAS devices.

This is a DIY build — you supply the case, RAM, and drives — but the result is a custom NAS that runs TrueNAS SCALE or Unraid properly, with enough SATA ports to matter.

- CPU: Intel N5105 (4 cores, 10W TDP)
- Network: 4x 2.5 GbE
- Storage: 4x SATA III + 2x M.2
- RAM: Up to 32 GB DDR4 SODIMM
- Price: ~$130 for the board, add RAM + case + drives

[Topton N5105 Board on Amazon](https://www.amazon.com/s?k=Topton+N5105+mini+itx&tag=YOURTAG-20)

### Runner-Up: Minisforum UM773 Lite (AMD Ryzen 7 7735HS)

The UM773 Lite sits just under $300 and brings a Ryzen 7 7735HS — a proper 8-core mobile chip with a respectable iGPU that can handle hardware transcoding for Jellyfin without a discrete GPU.

- CPU: AMD Ryzen 7 7735HS (8 cores / 16 threads)
- RAM: 16 GB DDR5 (upgradeable to 64 GB)
- Storage: 512 GB NVMe
- Networking: 2.5 GbE
- Power draw: ~20W idle
- Price: ~$280

[Minisforum UM773 Lite on Amazon](https://www.amazon.com/s?k=Minisforum+UM773+Lite&tag=YOURTAG-20)

The integrated Radeon 680M GPU is notably capable — if you care about Jellyfin performance or want to dabble in local AI (small models only), this has a leg up over Intel iGPU options at this price.

## What About Used Hardware?

Buying used is a legitimate strategy. A used Intel NUC 11 or 12 with a Core i5/i7 can be found for $150–200 and delivers strong performance. The risk is no warranty and potential wear.

Used ThinkCentre Tiny and HP EliteDesk 800 Mini machines are also worth considering — they're enterprise-grade, widely available refurbished, and easy to service.

## RAM Upgrades: Always Check First

Most mini PCs ship with soldered RAM on one slot and an open SO-DIMM slot, or two SO-DIMM slots total. Before buying, verify:

1. Is the RAM soldered or socketed?
2. What's the max supported RAM?
3. What speed/generation is supported (DDR4 vs DDR5)?

Going from 16 GB to 32 GB is often a $40 upgrade and meaningfully changes what the machine can do under Proxmox.

## Recommended Accessories

A few things that make any mini PC homelab better:

- [TP-Link TL-SG108E 8-Port Managed Switch](https://www.amazon.com/dp/B00K4DS5KU?tag=YOURTAG-20) — VLANs, monitoring, cheap
- [APC BE600M1 UPS](https://www.amazon.com/dp/B01FWYEOLY?tag=YOURTAG-20) — protects against power blips and lets you shut down gracefully
- [Samsung 870 EVO 2TB SATA SSD](https://www.amazon.com/dp/B08QBJ2YMG?tag=YOURTAG-20) — reliable bulk storage for media or backups
- [Cat6 Flat Ethernet Cable](https://www.amazon.com/s?k=cat6+flat+ethernet&tag=YOURTAG-20) — route cables along baseboards cleanly

## The Bottom Line

For most people building their first homelab, the Beelink EQ12 (N100) is the right starting point — cheap, efficient, and more capable than it looks. If you already know you want Proxmox with real VMs, jump straight to the EQR6 or NucBox G9.

The best mini PC is the one you'll actually run 24/7. Keep the power draw in mind, pick hardware with upgrade headroom, and don't overbuy for a workload you haven't validated yet.
