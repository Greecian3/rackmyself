---
title: "Docker Compose for Beginners: Run Any Self-Hosted App in Minutes"
date: 2026-04-18
description: "Learn Docker Compose from scratch in 2026. No version field needed anymore. This beginner guide explains containers, docker-compose.yml, and how to deploy real apps on your home server."
tags: ["docker", "docker compose", "self-hosting", "containers", "beginners"]
categories: ["guides"]
showToc: true
---

Docker Compose is the tool that makes self-hosting manageable. Instead of running long `docker run` commands with a dozen flags you'll forget in a week, Compose lets you define your entire application stack in a single readable YAML file — and bring everything up or down with one command.

This guide goes from zero to a real running application, explains every piece of the syntax, and covers the 2026 changes that catch people off guard (the `version:` field is gone — more on that shortly).

## What Is Docker, and Why Does It Matter?

A Docker container is a self-contained package: an application plus everything it needs to run — runtime, libraries, config. It's isolated from your host system, which means:

- It doesn't conflict with other software on your server
- You can delete it and start fresh without affecting anything else
- The same container runs identically on any machine

Think of containers like appliances. Your server is the kitchen. Each container is a standalone appliance that plugs in, does its job, and doesn't interfere with the others.

**Docker Compose** is the tool for managing multiple containers together. A Nextcloud instance needs the app server *and* a database. A media stack needs Jellyfin *and* a VPN container *and* maybe a reverse proxy. Compose describes the whole stack in one file and manages it as a unit.

## Installing Docker in 2026

On Ubuntu or Debian (the most common home server OS), the official install script handles everything:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

The second command lets your user run Docker without `sudo`. The third applies the change to your current session without logging out.

Verify:

```bash
docker --version
docker compose version
```

You should see Docker 27+ and Docker Compose 2.x. Note: the modern command is `docker compose` (with a space) — not the legacy `docker-compose` (with a hyphen). The hyphen version is deprecated. If you find older guides using `docker-compose`, mentally swap it for `docker compose`.

## The 2026 Change You Need to Know

Older Docker Compose files started with a `version:` field:

```yaml
version: '3.8'   # ← This is obsolete. Don't write it.
services:
  ...
```

As of 2026, the `version` field is completely obsolete and ignored. Docker Compose now follows the open Compose Specification, which merged the old 2.x and 3.x formats. Just omit it. Modern Compose files look like this:

```yaml
services:
  my-app:
    image: some/image
    ...
```

Clean, no version needed. If you see tutorials still writing `version: '3'`, they're out of date.

## Your First docker-compose.yml

Let's deploy something genuinely useful: **Uptime Kuma**, a self-hosted status monitoring tool with a polished UI that shows you when your services go down.

Create a directory and a Compose file:

```bash
mkdir ~/docker/uptime-kuma && cd ~/docker/uptime-kuma
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

Visit `http://your-server-ip:3001` and you'll see the Uptime Kuma setup page. That's it — a real monitoring app running in under two minutes.

## Reading the Compose File Line by Line

Let's break down every line of that file:

```yaml
services:
```
Everything under `services` is a container. You can have one or twenty. Each service gets its own section.

```yaml
  uptime-kuma:
```
The name of this service. Also the hostname other containers on the same network use to reach it — important when containers talk to each other.

```yaml
    image: louislam/uptime-kuma:1
```
The Docker image to use. `louislam/uptime-kuma` is pulled from Docker Hub. `:1` is the version tag. Always pin to a specific version (or at least a major version like `:1`) rather than `:latest` — it prevents surprise breakage when the developer releases an incompatible update.

```yaml
    container_name: uptime-kuma
```
The name of the running container. Optional, but makes `docker logs uptime-kuma` and `docker exec -it uptime-kuma bash` much easier to type.

```yaml
    volumes:
      - ./data:/app/data
```
Maps a directory on your host (`./data`, relative to the Compose file) to a path inside the container (`/app/data`). This is how data persists. Without this, deleting the container deletes all your data. With it, the data lives on your host disk — recreate the container anytime and your data is still there.

```yaml
    ports:
      - "3001:3001"
```
Maps host port 3001 to container port 3001. Format: `"HOST:CONTAINER"`. Change the host port to anything — `"8080:3001"` exposes the app on port 8080 on your machine. The container port is fixed by the application; the host port is yours to choose.

```yaml
    restart: unless-stopped
```
The container restarts automatically after a crash or after your server reboots — unless you explicitly run `docker compose down`. This is what makes self-hosted apps stay up without babysitting.

## A Real Multi-Container Example: Nextcloud

Most serious apps need a database alongside the app itself. Here's how Compose handles that:

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

Two things to understand here:

**Container networking:** `MYSQL_HOST=db` works because Docker Compose automatically creates a private network for all services in the same file. Services can reach each other using their service name as a hostname. The Nextcloud container connects to the database at hostname `db` — no IP addresses needed, no manual network configuration.

**`depends_on`:** Tells Compose to start the `db` service before `nextcloud`. It doesn't wait for the database to be *ready* — just for the container to exist. For most setups this is fine; the app retries its database connection on startup.

## Using .env Files for Passwords

Never hardcode passwords in your Compose file. Use an `.env` file instead:

Create `.env` in the same directory as your `docker-compose.yml`:

```
MYSQL_PASSWORD=use_a_real_password_here
MYSQL_ROOT_PASSWORD=use_a_different_real_password_here
```

Reference them in the Compose file with `${VARIABLE_NAME}` — as shown in the Nextcloud example above. Compose automatically reads `.env` from the current directory.

Add `.env` to your `.gitignore` if you use git:

```bash
echo ".env" >> .gitignore
```

This prevents you from accidentally committing passwords to a GitHub repo — a common and embarrassing mistake.

## The Commands You'll Use Every Day

```bash
# Start all services in the background
docker compose up -d

# See what's running and their status
docker compose ps

# View logs for all services (follow mode)
docker compose logs -f

# View logs for one service only
docker compose logs -f nextcloud

# Stop everything (containers stop, data stays)
docker compose down

# Pull newer image versions
docker compose pull

# Restart one service
docker compose restart nextcloud

# Open a shell inside a running container
docker compose exec nextcloud bash

# Validate your Compose file before running
docker compose config
```

The last one — `docker compose config` — is new and useful. It validates your YAML and resolves all variables. Run it before `up` to catch typos and misconfigured environment variables before they cause a failed startup.

## How to Organize Your Files

A clean directory structure makes managing multiple apps straightforward:

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
  jellyfin/
    docker-compose.yml
    .env
    config/
    cache/
```

Each app gets its own directory with its own `docker-compose.yml`. Running `docker compose up -d` inside any directory starts just that app. `docker compose down` stops just that app. Clean isolation, easy to manage.

## Updating Containers

When a new version of an app releases:

```bash
cd ~/docker/uptime-kuma
docker compose pull          # download the new image
docker compose up -d         # recreate the container with the new image
```

Compose detects that the image changed, stops the old container, and starts a new one. Your volume data persists through the update — that's the whole point of using volumes.

## A Practical Starter Stack

If you're just getting started and want real useful apps running fast, here's a solid starting stack:

**AdGuard Home** — network-wide ad blocking

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

**Vaultwarden** — self-hosted password manager (Bitwarden-compatible)

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

**Portainer** — web UI for managing all your Docker containers

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

Deploy them in order. Within 30 minutes you have ad blocking, a password manager, and a Docker management UI — all self-hosted, all free.

## Hardware for Running Docker

Any Linux-capable mini PC or server works. The practical minimum:

- **CPU:** 4 cores (Intel N100 or better)
- **RAM:** 16 GB (32 GB if running many containers or VMs)
- **Storage:** 500 GB NVMe for the OS and containers; add a second drive for data

Specific picks:
- [Beelink EQ12 (Intel N100)](https://www.amazon.com/dp/B0C1G2K8ZJ?tag=YOURTAG-20) — ~$165, handles 20+ containers, 6W idle
- [Samsung 870 EVO 2TB SSD](https://www.amazon.com/dp/B08QB93S6R?tag=YOURTAG-20) — reliable bulk storage for container volumes
- [APC BE600M1 UPS](https://www.amazon.com/dp/B01FWAZEIU?tag=YOURTAG-20) — protect your server from power blips

## Common Mistakes to Avoid

**Using `:latest` image tags:** Convenient but risky. A developer can push a breaking change to `latest` and your next `docker compose pull` silently breaks your stack. Pin to a specific version like `:29` or `:1.5.2`.

**Not using volumes for persistent data:** If the container stores data internally and you don't map a volume, deleting or recreating the container means data loss. Always map a volume for anything you care about.

**Port conflicts:** Two services can't use the same host port. If something fails to start, check:

```bash
ss -tulpn | grep :8080
```

**Running `docker compose up` without `-d`:** Without detached mode, the process ties to your terminal and stops when you disconnect. Always use `-d` for services you want running persistently.

**Missing `.env` file:** If your Compose file references `${SOME_VARIABLE}` and the `.env` file doesn't exist or the variable isn't set, Docker Compose substitutes an empty string — which often causes subtle, confusing failures. Run `docker compose config` to catch this before it bites you.

## What Comes Next

Once you're comfortable with Docker Compose basics, the next step is a reverse proxy. Instead of accessing apps on `192.168.1.10:3001`, `192.168.1.10:8080`, and `192.168.1.10:9000`, a reverse proxy like Caddy or Nginx Proxy Manager lets you use clean URLs (`uptime.home`, `vault.home`) and handles HTTPS automatically.

After that, Cloudflare Tunnels let you expose specific apps to the internet securely — no open ports, no exposed home IP. Both are covered in other guides on this site.

Docker Compose is the foundation that everything else builds on. Once you understand it, deploying a new self-hosted app is a 5-minute job instead of an afternoon of Googling.
