---
title: "Docker Compose for Beginners: Run Any Self-Hosted App in Minutes"
date: 2026-04-18
description: "Learn Docker Compose from scratch. This 2026 guide covers the current syntax (no version field), real working examples, common mistakes, and a starter app stack you can deploy today."
tags: ["docker", "docker compose", "self-hosting", "containers", "beginners"]
categories: ["guides"]
showToc: true
---

Docker Compose is the tool that makes self-hosting manageable. Without it, running an app means remembering a long `docker run` command full of flags you'll forget by next week. With it, you define your entire stack — app, database, reverse proxy — in a single readable file and bring everything up or down with one command.

This guide is for complete beginners. By the end you'll understand what containers are, how Compose files work, and you'll have real apps running on your server. It's also updated for 2026 — the syntax has changed in a way that trips up anyone following older tutorials.

## The 2026 Syntax Change You Need to Know First

Older guides start their Compose files with this line:

```yaml
version: '3.8'
```

**Don't write this.** It's obsolete as of the current Compose Specification. Docker Compose V2 (the current standard, shipping with all modern Docker installs) ignores the `version` field entirely. Writing it won't break anything, but it marks your file as outdated and confuses newer features.

Modern Compose files start directly with `services:` — no version field, no preamble. Every example in this guide follows the current standard.

## What Is a Container, Actually?

A container is a self-contained package: an application bundled with everything it needs to run — the runtime, libraries, config files. It runs isolated from your host system, which means:

- It doesn't interfere with other software on your server
- You can delete and recreate it without touching the host
- The exact same container runs identically on any machine

The practical benefit: you can run a Nextcloud instance, a Pi-hole, a password manager, and a media server on the same machine without any of them conflicting — each in its own isolated bubble.

Docker Compose is the tool for running multiple containers together as a coordinated stack, defined in a single YAML file.

## Installing Docker

On Ubuntu or Debian (the most common home server OS):

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

Verify:

```bash
docker --version
docker compose version
```

Two things to note:
1. The modern command is `docker compose` (with a space) — not the legacy `docker-compose` (hyphenated). The hyphenated version is deprecated and may not be installed at all.
2. You should see Docker 27+ and Docker Compose 2.x. If you see older versions, your package repo may be outdated — the script above pulls from Docker's own repo and gives you the current version.

## Your First Compose File

Let's deploy something real: **Uptime Kuma**, a self-hosted monitoring tool that pings your services and alerts you when they go down. Polished UI, takes two minutes to set up.

Create a directory and the Compose file:

```bash
mkdir -p ~/docker/uptime-kuma && cd ~/docker/uptime-kuma
nano docker-compose.yml
```

Paste this:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./data:/app/data
    ports:
      - "3001:3001"
    restart: unless-stopped
```

Start it:

```bash
docker compose up -d
```

Visit `http://your-server-ip:3001`. You'll see the Uptime Kuma setup page. That's a running self-hosted app in about 90 seconds.

## Understanding the Compose File

Every line in that file does something specific. Here's what each part means:

```yaml
services:
```
Everything under `services:` is a container. One service = one container. You can define as many as you need.

```yaml
  uptime-kuma:
```
The name of this service. It's also the hostname other containers in the same Compose file use to reach this one — relevant when containers need to talk to each other.

```yaml
    image: louislam/uptime-kuma:1
```
Which Docker image to run. `louislam/uptime-kuma` is pulled from Docker Hub. `:1` is the version tag. Always pin to a specific version rather than `:latest` — if you use `latest`, the next `docker compose pull` might silently break your stack when the developer pushes a breaking update.

```yaml
    container_name: uptime-kuma
```
Optional, but makes debugging easier. Without it, Docker generates a random name. With it, `docker logs uptime-kuma` and `docker exec -it uptime-kuma bash` work cleanly.

```yaml
    volumes:
      - ./data:/app/data
```
This is the most important concept in Docker Compose. It maps a directory on your host machine (`./data`, relative to the Compose file) to a path inside the container (`/app/data`). Your data lives on the host. Deleting and recreating the container leaves the data untouched. Without a volume mapping, deleting the container deletes all your data.

```yaml
    ports:
      - "3001:3001"
```
Format: `"HOST_PORT:CONTAINER_PORT"`. The host port is what you access in the browser. The container port is fixed by the application. Change the host port to anything available — `"8080:3001"` would expose Uptime Kuma on port 8080 on your machine.

```yaml
    restart: unless-stopped
```
The container restarts automatically after a crash or server reboot — unless you explicitly stop it with `docker compose down`. This is what keeps your self-hosted apps up without babysitting.

## A Multi-Container Example

Real apps usually need a database. Here's Nextcloud with MariaDB — two containers working together:

```yaml
services:
  nextcloud:
    image: nextcloud:29
    container_name: nextcloud
    ports:
      - "8080:80"
    volumes:
      - ./data:/var/www/html
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: mariadb:11
    container_name: nextcloud_db
    volumes:
      - ./db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    restart: unless-stopped
```

Two things to understand:

**Container networking:** `MYSQL_HOST=db` works because Compose automatically creates a private network for all services in the same file. Containers reach each other by service name — `db` resolves to the database container's IP automatically. You never need to manage IP addresses.

**`depends_on`:** Starts the `db` container before `nextcloud`. Important: it doesn't wait for MariaDB to be *ready and accepting connections* — just for the container to exist. For most apps this is fine because the app retries its database connection on startup. If you need to wait for the database to be fully ready, you'd add a healthcheck — covered below.

## Secrets: Use a .env File

Never hardcode passwords in your Compose file. If you ever commit it to git or share it, your credentials are exposed.

Create `.env` in the same directory as `docker-compose.yml`:

```
MYSQL_PASSWORD=use_a_real_password_here
MYSQL_ROOT_PASSWORD=use_a_different_password_here
```

Compose automatically reads `.env` from the current directory. Reference variables with `${VARIABLE_NAME}` — as in the Nextcloud example above.

Add `.env` to `.gitignore`:

```bash
echo ".env" >> .gitignore
```

Run `docker compose config` before starting to verify variables resolved correctly:

```bash
docker compose config
```

This prints the final composed configuration with all variables substituted. If a variable shows as empty, your `.env` file has a typo or is in the wrong location.

## The Commands You'll Use Every Day

```bash
# Start everything in the background
docker compose up -d

# See what's running and status
docker compose ps

# View logs, all services
docker compose logs -f

# View logs, one service only
docker compose logs -f nextcloud

# Stop containers (data stays)
docker compose down

# Stop containers AND delete volumes (data gone — be careful)
docker compose down -v

# Pull newer image versions
docker compose pull

# Restart one service without touching others
docker compose restart nextcloud

# Open a shell inside a running container
docker compose exec nextcloud bash

# Validate your compose file before running
docker compose config
```

Run `docker compose config` before `up` when you've made changes — it catches YAML syntax errors and unresolved variables before they cause a confusing failed start.

## How to Organize Your Files

A clean structure makes managing many apps straightforward:

```
~/docker/
  uptime-kuma/
    docker-compose.yml
    .env
    data/
  nextcloud/
    docker-compose.yml
    .env
    data/
    db_data/
  vaultwarden/
    docker-compose.yml
    .env
    data/
```

Each app gets its own directory with its own Compose file. `docker compose up -d` inside any directory starts just that app. `docker compose down` stops just that app. Clean, isolated, easy to manage.

## A Practical Starter Stack

Three apps that are genuinely useful and easy to run:

**AdGuard Home** — network-wide ad blocking. Point your router's DNS at your server and every device blocks ads without any per-device configuration.

```yaml
services:
  adguardhome:
    image: adguard/adguardhome:latest
    container_name: adguardhome
    ports:
      - "53:53/udp"
      - "53:53/tcp"
      - "3000:3000/tcp"
    volumes:
      - ./work:/opt/adguardhome/work
      - ./conf:/opt/adguardhome/conf
    restart: unless-stopped
```

**Vaultwarden** — self-hosted Bitwarden-compatible password manager. Use the official Bitwarden apps with your own server.

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
      - SIGNUPS_ALLOWED=false
    restart: unless-stopped
```

Set `SIGNUPS_ALLOWED=false` after creating your account so nobody else can register.

**Portainer** — web UI for managing all your Docker containers. Good for visualizing what's running without living in the terminal.

```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data:/data
    restart: unless-stopped
```

## Updating Containers

When a new version of an app releases:

```bash
cd ~/docker/uptime-kuma
docker compose pull    # download the new image
docker compose up -d   # recreate with the new image
```

Compose detects the image changed, stops the old container, starts a new one. Your volume data persists through the update — that's the whole point of using volumes.

## Common Mistakes That Will Bite You

**Using `:latest` tags:** Fine for learning, dangerous in practice. Pin to a version like `:1` or `:29`. When you run `docker compose pull`, you get whatever `latest` means that day — if the developer pushed a breaking change, your app silently breaks on the next pull.

**Missing volumes for persistent data:** If you don't define a volume for data you care about, it lives inside the container. Delete or recreate the container and the data is gone. Always define volumes for anything that matters.

**Port conflicts:** Two services can't share a host port. If a service fails to start, check:

```bash
ss -tulpn | grep :8080
```

Something else is using that port. Change the host port in your Compose file.

**`depends_on` without health checks:** `depends_on` only ensures a container *exists* before another starts — it doesn't wait for the service inside to be ready. For databases, MariaDB or Postgres may take a few seconds to be ready to accept connections. Most app containers retry on startup and handle this gracefully, but if yours doesn't, add a healthcheck:

```yaml
  db:
    image: mariadb:11
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5
```

Then use `condition: service_healthy` in the dependent service:

```yaml
  nextcloud:
    depends_on:
      db:
        condition: service_healthy
```

**Running without `-d`:** `docker compose up` without the detached flag ties the process to your terminal. Close the terminal and everything stops. Always use `-d` for services you want running persistently.

## Hardware Recommendations

Any modern mini PC or Linux machine works. The practical minimum:

- **RAM:** 16 GB (every container needs some headroom)
- **CPU:** 4 cores (Intel N100 or better handles 20+ containers)
- **Storage:** 500 GB NVMe for OS and containers; add a second drive for data

Good options:
- [Beelink EQ12 (Intel N100)](https://www.amazon.com/s?k=Beelink+EQ12+N100+16GB+dual+2.5G&tag=YOURTAG-20) — ~$170, 6W idle, handles 20+ containers without complaint
- [Samsung 870 EVO 2TB SSD](https://www.amazon.com/dp/B08QB93S6R?tag=YOURTAG-20) — ~$110, reliable bulk storage for container volumes
- [APC BE600M1 UPS](https://www.amazon.com/dp/B01FWAZEIU?tag=YOURTAG-20) — ~$55, prevents data corruption from power blips

See our [homelab hardware guide](/posts/best-mini-pcs-homelab-2026/) for a full breakdown.

## Frequently Asked Questions

**Do I need to know Linux to use Docker Compose?**
You need to be comfortable with basic terminal commands — navigating directories, editing files with `nano`, running commands. Full Linux expertise isn't required.

**What's the difference between Docker and Docker Compose?**
Docker runs individual containers. Docker Compose orchestrates multiple containers together with a single config file. You always need Docker; Compose is the tool on top of it.

**Can I run Docker on a Raspberry Pi?**
Yes. Docker and Docker Compose run on ARM. Some images don't have ARM builds — check for `linux/arm64` in the image's supported platforms on Docker Hub before deploying.

**How do I back up my containers?**
Back up the data directories (the host-side of your volume mounts). The containers themselves don't need backing up — you can always recreate them from the Compose file. Back up `~/docker/` and you have everything you need to restore.

**What if an app doesn't have a Docker image?**
Most popular self-hosted apps have official or community Docker images. Check the project's GitHub README or Docker Hub. If there's no image, you'd need to build one yourself or install the app directly on the host — outside the scope of Compose.

**Is Docker Compose the same as Kubernetes?**
No. Compose is for single-machine deployments. Kubernetes manages containers across multiple machines. For a home server, Compose is the right tool. Kubernetes adds enormous complexity that doesn't pay off unless you're running a cluster.
