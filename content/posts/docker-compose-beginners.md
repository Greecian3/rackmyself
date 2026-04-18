---
title: "Docker Compose for Beginners: Run Any App in Minutes"
date: 2026-04-18
description: "Learn Docker Compose from scratch. This beginner guide explains containers, docker-compose.yml, and how to run real self-hosted apps on your home server."
tags: ["docker", "docker compose", "self-hosting", "containers", "beginners"]
categories: ["guides"]
showToc: true
---

Docker Compose is the tool that makes self-hosting actually manageable. Instead of running long `docker run` commands with a dozen flags you'll forget tomorrow, Compose lets you define your entire app stack in a single readable file — then bring it up or down with one command.

If you've been intimidated by Docker, this guide is for you. We'll go from zero to running a real application, and explain every piece of the puzzle along the way.

## What Is Docker, Actually?

A container is a self-contained package that includes an application and everything it needs to run — the runtime, libraries, config. It runs isolated from your host system, which means:

- It won't conflict with other software on your server
- It's easy to delete and start fresh
- The same container runs identically on any machine

Think of it like a shipping container. The container itself is standardized; what's inside is whatever the shipper packed. Docker is the system that manages those containers.

## What Is Docker Compose?

Docker Compose is a tool for defining and running multi-container applications. Instead of managing containers one at a time, you describe the whole stack — app, database, reverse proxy — in a YAML file called `docker-compose.yml`. Then you run one command and everything starts together.

It also handles:
- Networks between containers
- Persistent volumes for data
- Environment variables
- Restart policies

## Installing Docker

On Ubuntu/Debian (the most common home server OS):

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

The last two commands let your current user run Docker without `sudo`. Verify the install:

```bash
docker --version
docker compose version
```

Docker Compose is now bundled with Docker (as `docker compose`, not the old `docker-compose` command). If you see version numbers, you're good.

## Your First docker-compose.yml

Let's deploy something real: Uptime Kuma, a self-hosted uptime monitoring tool with a clean UI.

Create a directory for it:

```bash
mkdir ~/uptime-kuma && cd ~/uptime-kuma
```

Create the compose file:

```bash
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

Visit `http://your-server-ip:3001` and you'll see the Uptime Kuma setup page. That's it.

## Breaking Down the Compose File

Let's read through what each line means:

```yaml
services:
```
Everything under `services` is a container you want to run. You can have one or twenty.

```yaml
  uptime-kuma:
```
This is the name you're giving the service. It's also the hostname other containers on the same network can use to reach this one.

```yaml
    image: louislam/uptime-kuma:1
```
The Docker image to use. `louislam/uptime-kuma` is the image name (from Docker Hub), and `1` is the tag (version). Using `latest` is fine for experimentation but pinning to a version is better for stability.

```yaml
    volumes:
      - ./data:/app/data
```
This maps a directory on your host (`./data`, relative to where the compose file lives) to a path inside the container (`/app/data`). This is how data persists — if you delete and recreate the container, the data is still there on disk.

```yaml
    ports:
      - "3001:3001"
```
Maps host port 3001 to container port 3001. Format is `host:container`. You can change the host port to anything — `"8080:3001"` would expose it on port 8080 locally.

```yaml
    restart: unless-stopped
```
The container restarts automatically if it crashes, or when your server reboots — unless you explicitly stop it yourself.

## A Real Multi-Container Example

Most real apps need a database. Here's how that works with Nextcloud:

```yaml
services:
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    ports:
      - "8080:80"
    volumes:
      - ./nextcloud_data:/var/www/html
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=changeme
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: mariadb:10.11
    container_name: nextcloud_db
    volumes:
      - ./db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=changeme_root
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=changeme
    restart: unless-stopped
```

Two things to notice here:

1. The `MYSQL_HOST=db` environment variable in the Nextcloud container uses `db` as the hostname — that's the name of the other service. Docker Compose automatically creates a network between services in the same file, so they can reach each other by service name.

2. `depends_on: - db` tells Compose to start the database before Nextcloud. It doesn't wait for the database to be *ready*, just for the container to exist — for most apps this is fine.

## Essential Docker Compose Commands

```bash
# Start everything in the background
docker compose up -d

# See running containers and their status
docker compose ps

# View logs (follow mode)
docker compose logs -f

# View logs for one service
docker compose logs -f nextcloud

# Stop everything (containers stay, data stays)
docker compose down

# Stop and delete volumes (data gone — careful!)
docker compose down -v

# Pull newer image versions
docker compose pull

# Restart a single service
docker compose restart nextcloud

# Run a command inside a running container
docker compose exec nextcloud bash
```

The `-d` flag means "detached" — runs in the background. Without it, logs stream to your terminal until you hit Ctrl+C.

## Where to Keep Your Compose Files

A common pattern that keeps things organized:

```
~/docker/
  uptime-kuma/
    docker-compose.yml
    data/
  nextcloud/
    docker-compose.yml
    nextcloud_data/
    db_data/
  pihole/
    docker-compose.yml
    pihole_data/
    dnsmasq_data/
```

Each app gets its own directory with its own compose file. Running `docker compose up -d` in any of those directories starts just that app. Clean and easy to manage.

## Using .env Files for Secrets

Hardcoding passwords in your compose file is a bad habit. Use an `.env` file instead:

Create `.env` in the same directory as your compose file:

```
MYSQL_PASSWORD=my_actual_password
MYSQL_ROOT_PASSWORD=my_root_password
```

Reference them in the compose file with `${VARIABLE_NAME}`:

```yaml
environment:
  - MYSQL_PASSWORD=${MYSQL_PASSWORD}
  - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
```

Docker Compose automatically reads `.env` from the current directory. Add `.env` to your `.gitignore` so you don't accidentally commit passwords.

## Updating Containers

When a new version of an app drops:

```bash
cd ~/docker/uptime-kuma
docker compose pull
docker compose up -d
```

Compose will detect the new image, stop the old container, and start a fresh one with the same volumes. Your data persists.

## Useful Hardware for Docker Hosting

Any mini PC or home server works, but some suggestions:

- [Beelink EQ12 Mini PC (Intel N100)](https://www.amazon.com/s?k=Beelink+EQ12&tag=YOURTAG-20) — quiet, low power, handles dozens of containers
- [Samsung 870 EVO 1TB SSD](https://www.amazon.com/dp/B08QBJ2YMG?tag=YOURTAG-20) — fast reliable storage for container volumes
- [TP-Link 8-Port Gigabit Switch](https://www.amazon.com/dp/B00A121WN6?tag=YOURTAG-20) — if you're wiring multiple devices

## Common Mistakes

**Using latest tags in production:** Fine for learning, annoying in practice when an update breaks something and you don't know what changed. Pin versions.

**Not using volumes:** If you don't map a volume for persistent data, it lives inside the container. When the container goes away, so does your data.

**Port conflicts:** Two services can't use the same host port. If something won't start, check if the port is in use: `ss -tulpn | grep :8080`.

**Forgetting -d:** Running `docker compose up` without `-d` ties the process to your terminal. Always use `-d` for background operation.

## What's Next

Once you're comfortable with Compose basics, the logical next step is a reverse proxy. Instead of accessing apps on weird port numbers (`192.168.1.100:3001`), a reverse proxy like Nginx Proxy Manager lets you use clean URLs (`uptime.home`) and handles HTTPS automatically.

After that, Cloudflare Tunnels let you expose specific apps to the internet securely — without opening ports on your router. We have guides on both.

Docker Compose is the foundation. Once you understand it, deploying new self-hosted apps takes minutes instead of hours.
