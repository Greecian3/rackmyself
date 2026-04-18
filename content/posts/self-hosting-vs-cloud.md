---
title: "Self-Hosting vs. The Cloud: When It Actually Makes Sense"
date: 2026-04-18
description: "An honest comparison of self-hosting vs cloud services. When self-hosting saves money and gives you control, and when it's just more work for the same result."
tags: ["self-hosting", "cloud", "homelab", "cost analysis", "privacy"]
categories: ["opinion"]
showToc: true
---

Self-hosting is having a moment. The combination of rising SaaS prices, privacy concerns, and increasingly capable cheap hardware has pushed a lot of people toward running their own infrastructure. But the honest truth is: self-hosting isn't always the right answer. Sometimes the cloud wins, and knowing when to use each is more valuable than tribal loyalty to either.

This is an honest comparison — not a manifesto.

## The Case for the Cloud

Let's be real about what cloud services actually provide:

**No hardware to babysit.** Your files don't stop syncing because your home power went out. Your email doesn't bounce because you forgot to renew a certificate. The operational overhead of keeping a service running and available is real, and paying someone else to handle it has genuine value.

**Professional security and compliance.** Google's security team is much larger than yours. AWS's infrastructure handles DDoS attacks you'd never survive on residential bandwidth. For sensitive business data, regulated industries, or anything you can't afford to lose, professional cloud services have an advantage.

**Instant scale.** Cloud services handle your storage growing from 100 GB to 10 TB without any planning on your part. Self-hosted storage means buying drives, adding them to your array, and expanding your pool.

**Zero upfront cost.** A Dropbox subscription is $12/month. A comparable home server (even a cheap mini PC) is $150–300 plus drives, plus time. The break-even point on hardware is a real calculation.

## The Case for Self-Hosting

**Ongoing cost scales with usage.** The biggest difference. A cloud subscription costs the same whether you use it lightly or heavily. Your home server's electricity bill barely changes whether you store 1 TB or 20 TB, run 5 containers or 50.

Run the numbers: A home server running at $150/year in electricity, with $300 in hardware amortized over 5 years ($60/year), costs $210/year total. That includes hosting your own cloud storage, calendar, email, password manager, media server, home automation, and whatever else you run. Replace those with individual subscriptions and you're looking at $50–200/month.

**Privacy and data ownership.** Your data isn't mined, isn't subject to another company's privacy policy changes, and isn't handed to third parties. For personal photos, financial documents, notes, and health data, this matters.

**Learning value.** Running your own infrastructure teaches you networking, Linux, security, and operations in a way that reading about it never does. For people in tech or heading toward it, the skills built by running a homelab are directly applicable and worth time investment.

**Customization and control.** Cloud services give you what they give you. Self-hosted software can be configured, extended, and integrated in ways commercial products don't allow. Vaultwarden (self-hosted Bitwarden) lets you set your own policies, retain your data forever, and integrate with internal tooling. The hosted version doesn't.

**No account risk.** Cloud services get shut down, get acquired, change pricing, or suspend accounts. Google has killed dozens of products. Your self-hosted Nextcloud doesn't disappear because a business decision changed.

## Where Cloud Wins

### Email

Running your own mail server is genuinely hard. Deliverability — getting your email into inboxes instead of spam — requires a specific combination of DNS records (SPF, DKIM, DMARC), a good reputation for your sending IP, and ongoing maintenance. ISPs block port 25 on residential connections. Even if you get it working, you'll fight deliverability issues indefinitely.

**Verdict: Cloud wins.** Use Fastmail, Proton Mail, or even Gmail. The free tier of Proton Mail is private, reliable, and zero-maintenance. The cost of self-hosting email in time and frustration exceeds what you'd spend on a professional service for most people.

### Video Conferencing

This requires reliable uptime, low latency, good STUN/TURN infrastructure, and enough bandwidth to handle multiple simultaneous streams. Jitsi Meet works, but the quality gap versus Zoom or Google Meet is real, and the reliability requirements are demanding.

**Verdict: Cloud wins.** Unless you're running internal-only calls in a corporate context, the commercial options are better.

### Public-Facing Websites With High Traffic

A site getting millions of views benefits from CDN infrastructure, auto-scaling, and geographic distribution. Your home server's residential upload bandwidth (typically 20–50 Mbps) can't handle that. Static site hosting on Cloudflare Pages or Vercel is free, infinitely scalable, and better than what you can self-host.

**Verdict: Cloud wins** for anything with real traffic.

### Mobile Photo Sync

Google Photos and iCloud are deeply integrated with their respective platforms, have excellent apps, and provide a seamless experience. Nextcloud's photo app works but is slower and less polished. The built-in alternatives have features (People, Memories, sharing) that take real work to replicate.

**Verdict: Depends.** If privacy is the priority, Immich (self-hosted) is genuinely good now. If you want zero friction, Google Photos or iCloud is still better.

## Where Self-Hosting Wins

### Password Managers

[Vaultwarden](https://github.com/dani-garcia/vaultwarden) is a self-hosted Bitwarden-compatible server. The official Bitwarden personal plan is $10/year (great value), but the self-hosted version is free beyond your server's existing cost. It's stable, actively maintained, and the Bitwarden apps work identically against either.

**Verdict: Self-hosting wins**, especially if you're already running a server. Easy setup, huge ongoing savings, and you control your vault data.

### Media Serving

Jellyfin (self-hosted Plex alternative) is free and mature. Netflix costs $15–22/month. If you have a media library — movies, TV shows, home videos — Jellyfin serves them to any device, transcodes on the fly, and costs nothing beyond hardware you already own.

**Verdict: Self-hosting wins.** Jellyfin is excellent.

### Cloud Storage and File Sync

Nextcloud is the leading option here — calendar, contacts, file sync, office suite, all self-hosted. 2 TB of Dropbox is $120/year. Self-hosted on existing hardware, it's the electricity bill, which is already being paid.

**Verdict: Self-hosting wins at scale.** For someone who needs 50 GB, Dropbox free or Google One is easier. For someone with 2+ TB of files, self-hosting is dramatically cheaper.

### Home Automation

Home Assistant is the gold standard — open source, local processing, works without internet, integrates with virtually every smart home device. The commercial alternatives (SmartThings, Google Home) are cloud-dependent and vendor-locked. Local processing means your lights still work when the cloud is down.

**Verdict: Self-hosting wins.** Home Assistant specifically is a case where the self-hosted option is genuinely better than any commercial alternative.

### Development Infrastructure

CI/CD pipelines, internal tools, staging environments, private package repositories — these are ideal self-hosting candidates. GitHub Actions and similar have generous free tiers, but for teams or heavy usage, running your own Gitea + Woodpecker CI + private container registry is cheap and fast.

**Verdict: Self-hosting wins** for teams and power users.

## The Actual Cost Calculation

A realistic self-hosted setup:

| Item | Cost |
|------|------|
| Mini PC (Beelink EQ12) | $160 (one-time) |
| 2x 4TB NAS drives | $160 (one-time) |
| UPS | $80 (one-time) |
| Electricity (10W avg, $0.15/kWh) | $13/year |
| Domain name | $12/year |

**Total year 1:** ~$425  
**Total year 2+:** ~$25/year

Services replaced:
- Dropbox 2TB: $120/year
- Bitwarden premium: $10/year
- Plex Pass: $40/year
- Home Assistant Cloud: $65/year

**Cloud services replaced:** $235/year

Break-even: under 2 years. After that, you're ahead by $200+ annually, running substantially more than those services provide.

## The Honest Tradeoff

Self-hosting requires time and interest. If you enjoy the work — setting things up, learning, maintaining — it's rewarding and cost-effective. If you view it as chores, that friction erases the value.

You're also betting on your own reliability. If your server is down for a day and your family can't access their Nextcloud files, that's your problem to solve. Commercial services offer SLAs and support.

The right answer for most people: **self-host what you enjoy maintaining and where privacy or cost savings genuinely matter. Use the cloud for everything else.** There's no reason to be religious about it either direction.

Start with one or two things — maybe Jellyfin and Vaultwarden — and see if running your own infrastructure suits you before committing to a full homelab buildout.
